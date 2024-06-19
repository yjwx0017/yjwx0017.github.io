title: boost.asio 笔记
categories: 日志
tags: [boost,asio,boost.asio]
date: 2021-05-09 11:00:00
---
## 《Boost.Asio C++ Network Programming》

libtorrent 使用了 Boost.Asio

### 支持

network
com serial ports
files

### 实现同步/异步输入输出

``` c++
read(stream, buffer)
async_read(stream, buffer)
write(stream, buffer)
async_write(stream, buffer)
```

### 协议

`TCP` `UDP` `IMCP`

可以根据自己的需要扩展使其支持自己的协议

### 同步和异步

如果使用同步模式，因为是阻塞模式，所以服务器客户端往往使用多线程

编程前尽早决定使用同步还是异步模式

<!--more-->

异步模式调试更困难

 - 同步多线程
 - 异步单线程

### 同步客户端

```c++
using boost::asio;
io_service service;
ip::tcp::endpoint ep(ip::address::from_string("127.0.0.1", 2001);
ip::tcp::socket(service);
sock.connect(ep);
```

Boost.Asio uses io_service to talk to the operating system's input/output services.

### 同步服务器

```c++
using boost::asio;
typedef boost::shared_ptr<ip::tcp::socket> socket_ptr;
io_service service;
ip::tcp::endpoint ep(ip::tcp::v4(), 2001)); // listen on 2001
ip::tcp::acceptor acc(service, ep);
while (true)
{
    socket_ptr sock(new ip::tcp::socket(service));
    acc.accept(*sockt); // 接受一个连接请求
    boost::thread(boost::bind(client_session, sock);
}
void client_session(socket_ptr sock)
{
    while (true)
    {
        char data[512];
        size_t len = sock->read_some(buffer(data));
        if (len > 0)
            write(*sock, buffer("ok", 2));
    }
}
```

### 异步客户端

```c++
using boost::asio;
io_service service;
ip::tcp::endpoint ep(ip::address::from_string("127.0.0.1", 2001);
ip::tcp::socket sock(service);
sock.async_connect(ep, connect_handler);  // 使用回调函数
service.run();

void connect_handler(const boost::system::error_code& ec)
{
    // here we known we connected successfully
	// if ec indicates success
}

// service.run() 直到所有异步操作完成才退出
```

### 异步服务器

```c++ 
using boost::asio;
typedef boost::shared_ptr<ip::tcp::socket> socket_ptr;
io_service service;
ip::tcp::endpoint ep(ip::tcp::v4(), 2001);
ip::tcp::acceptor acc(service, ep);
socket_ptr sock(new ip::tcp::socket(service));
start_accept(sock);
service.run();

void start_accept(socket_ptr sock)
{
	acc.async_accept(*sock, boost::bind(handle_accept, sock, _1));
}

void handle_accept(socket_ptr sock, const boost::system::error_code& err)
{
    if (err) 
	    return;
	
	// at this point, you can read/write to the socket
	
	socket_ptr sock(new ip::tcp::socket(service));
	start_accept(sock);
}
```

### 异常处理

```c++
using boost::asio;
ip::tcp::endpoint ep;
ip::tcp::socket sock(service);
// 抛出异常 boost::system::system_error
try
{
    sock.connect(ep);
} catch (boost::system::system_error& e)
{ std::cout << e.code() << std::endl; }
boost::system::error_code err;
// 返回错误代码
sock.connect(ep, err);
if (err)
    std::cout << err << std::endl;
```
	
异步模式始终返回错误代码，异步函数不会抛出异常。

```c++
char data[512];
boost::system::error_code error;
size_t length = sock.read_some(buffer(data), error);
if (error == error::eof)
    return; // Connection closed
```

boost/asio/error.hpp

### 线程安全

#### io_service

io_service 是线程安全的。

多个线程可以调用io_service::run()

回调函数会在任意一个调用run()的线程中执行。

#### socket

socket类不是线程安全的

应该避免一个线程读另一个线程写同一个socket

#### utility

utility 用在多个线程中是没有意义的，所以不是线程安全的。

Most of them are meant to just be used for a short time, then
go out of scope.

### 除了网络之外

#### 信号

```c++
void signal_handler(const boost::system::error_code & err, int signal)
{
// log this, and terminate application
}
boost::asio::signal_set sig(service, SIGINT, SIGTERM);
sig.async_wait(signal_handler);
```

#### 串口

```c++
// Using Boost.Asio, you can easily connect to a serial port. 
// The port name is COM7 on Windows, or /dev/ttyS0 on POSIX platforms:
io_service service;
serial_port sp(service, "COM7");
serial_port::baud_rate rate(9600);
sp.set_option(rate);
```

其他详见Boost.Asio C++ Network Programming.pdf 14页

#### 计时器

```c++
sock.async_read_some(buffer(data, 512));
deadline_timer t(service, boost::posix_time::milliseconds(100));
t.async_wait(&deadline_handler);
service.run();
```

同步计时器和sleep等价

```c++
boost::this_thread::sleep(500);
// -or-
deadline_timer t(service, boost::posix_time::milliseconds(500));
t.wait();
```

### io_service

以下代码没有意义，多个service实例运行在同一个线程中
因为service_[1].run()需要等待service_[0].run()完成。

```c++
for ( int i = 0; i < 2; ++i)
    service_[i].run();
```

#### 一个线程和一个io_service实例

线程1 s.run()

所有回调函数的调用在该线程中是同步的

#### 多个线程和一个io_service实例

线程1 s.run()
线程2 s.run()

所有回调函数会被分配到多个线程执行，多线程间并发执行

#### 多个线程和多个io_service实例

线程1 s1.run()
线程2 s2.run()


#### 使 run() 函数始终运行的方法

方法一，在回调函数中执行新的异步操作

方法二，使用如下代码：

``` c++ 
typedef boost::shared_ptr<io_service::work> work_ptr;
work_ptr dummy_work(new io_service::work(service_));
```

以上代码可以使 `service_.run()` 始终保持运行，直到使用 `service_.stop()` 或 `dummy_work.reset(0)` 终止。

#### post 和 dispatch

```c++
// service.post()
io_service::strand strand_one(service);
service.post(strand_one.wrap(boost::bind(func, i)));
```

post() 是立即返回，回调函数被加入 io_service 处理队列。

```c++
boost::function<void()> func = service.wrap(func_2);
service.dispatch(func_2)
```

dispatch() 如果是在 run() 所在线程中调用（比如在某个回调函数中），则直接调用函数然后返回，否则和 post() 一样。

erase 配合 remove_if

### thread_group

```c++
boost::thread_group threads;
threads.create_thread(func1);
threads.create_thread(func2);
threads.join_all();
```

### remove_if

```c++
bool isZero(int num) { return num == 0; }
vector<int> v {0, 1, 3, 4};
v.erase(remove_if(v.begin(), v.end(), isZero), v.end());
// v 结果为 1 3 4
// remove_if 不改变容器的大小，返回移除后的后边界
// 然后需要配合 erase 清除无用数据
```

## 官方文档

[https://www.boost.org/doc/libs/1_60_0/doc/html/boost_asio.html][1]

### Basic Skills

#### Timer.1 - Using a timer synchronously

``` c++
#include <iostream>
#include <boost/asio.hpp>
#include <boost/date_time/posix_time/posix_time.hpp>

int main()
{
  boost::asio::io_service io;

  boost::asio::deadline_timer t(io, boost::posix_time::seconds(5));
  t.wait();

  std::cout << "Hello, world!" << std::endl;

  return 0;
}
```

#### Timer.2 - Using a timer asynchronously

The asio library provides a guarantee that callback handlers will only be called from threads that are currently calling io_service::run(). Therefore unless the io_service::run() function is called the callback for the asynchronous wait completion will never be invoked.


<!--more-->


``` c++
#include <iostream>
#include <boost/asio.hpp>
#include <boost/date_time/posix_time/posix_time.hpp>

void print(const boost::system::error_code& /*e*/)
{
  std::cout << "Hello, world!" << std::endl;
}

int main()
{
  boost::asio::io_service io;

  boost::asio::deadline_timer t(io, boost::posix_time::seconds(5));
  t.async_wait(&print);

  io.run();

  return 0;
}
```

#### Timer.3 - Binding arguments to a handler

``` c++
#include <iostream>
#include <boost/asio.hpp>
#include <boost/bind.hpp>
#include <boost/date_time/posix_time/posix_time.hpp>

void print(const boost::system::error_code& /*e*/,
    boost::asio::deadline_timer* t, int* count)
{
  if (*count < 5)
  {
    std::cout << *count << std::endl;
    ++(*count);

    t->expires_at(t->expires_at() + boost::posix_time::seconds(1));
    t->async_wait(boost::bind(print,
          boost::asio::placeholders::error, t, count));
  }
}

int main()
{
  boost::asio::io_service io;

  int count = 0;
  boost::asio::deadline_timer t(io, boost::posix_time::seconds(1));
  t.async_wait(boost::bind(print,
        boost::asio::placeholders::error, &t, &count));

  io.run();

  std::cout << "Final count is " << count << std::endl;

  return 0;
}
```

#### Timer.4 - Using a member function as a handler

``` c++
#include <iostream>
#include <boost/asio.hpp>
#include <boost/bind.hpp>
#include <boost/date_time/posix_time/posix_time.hpp>

class printer
{
public:
  printer(boost::asio::io_service& io)
    : timer_(io, boost::posix_time::seconds(1)),
      count_(0)
  {
    timer_.async_wait(boost::bind(&printer::print, this));
  }

  ~printer()
  {
    std::cout << "Final count is " << count_ << std::endl;
  }

  void print()
  {
    if (count_ < 5)
    {
      std::cout << count_ << std::endl;
      ++count_;

      timer_.expires_at(timer_.expires_at() + boost::posix_time::seconds(1));
      timer_.async_wait(boost::bind(&printer::print, this));
    }
  }

private:
  boost::asio::deadline_timer timer_;
  int count_;
};

int main()
{
  boost::asio::io_service io;
  printer p(io);
  io.run();

  return 0;
}
```

#### Timer.5 - Synchronising handlers in multithreaded programs

An boost::asio::strand guarantees that, for those handlers that are dispatched through it, an executing handler will be allowed to complete before the next one is started. This is guaranteed irrespective of the number of threads that are calling io_service::run(). Of course, the handlers may still execute concurrently with other handlers that were not dispatched through an boost::asio::strand, or were dispatched through a different boost::asio::strand object.

The io_service::strand class provides the ability to post and dispatch handlers with the guarantee that none of those handlers will execute concurrently.

``` c++
#include <iostream>
#include <boost/asio.hpp>
#include <boost/thread/thread.hpp>
#include <boost/bind.hpp>
#include <boost/date_time/posix_time/posix_time.hpp>

class printer
{
public:
  printer(boost::asio::io_service& io)
    : strand_(io),
      timer1_(io, boost::posix_time::seconds(1)),
      timer2_(io, boost::posix_time::seconds(1)),
      count_(0)
  {
    timer1_.async_wait(strand_.wrap(boost::bind(&printer::print1, this)));
    timer2_.async_wait(strand_.wrap(boost::bind(&printer::print2, this)));
  }

  ~printer()
  {
    std::cout << "Final count is " << count_ << std::endl;
  }

  void print1()
  {
    if (count_ < 10)
    {
      std::cout << "Timer 1: " << count_ << std::endl;
      ++count_;

      timer1_.expires_at(timer1_.expires_at() + boost::posix_time::seconds(1));
      timer1_.async_wait(strand_.wrap(boost::bind(&printer::print1, this)));
    }
  }

  void print2()
  {
    if (count_ < 10)
    {
      std::cout << "Timer 2: " << count_ << std::endl;
      ++count_;

      timer2_.expires_at(timer2_.expires_at() + boost::posix_time::seconds(1));
      timer2_.async_wait(strand_.wrap(boost::bind(&printer::print2, this)));
    }
  }

private:
  boost::asio::io_service::strand strand_;
  boost::asio::deadline_timer timer1_;
  boost::asio::deadline_timer timer2_;
  int count_;
};

int main()
{
  boost::asio::io_service io;
  printer p(io);
  boost::thread t(boost::bind(&boost::asio::io_service::run, &io));
  io.run();
  t.join();

  return 0;
}
```

### Introduction to Sockets

#### Daytime.1 - A synchronous TCP daytime client

``` c++
#include <iostream>
#include <boost/array.hpp>
#include <boost/asio.hpp>

using boost::asio::ip::tcp;

int main(int argc, char* argv[])
{
  try
  {
    if (argc != 2)
    {
      std::cerr << "Usage: client <host>" << std::endl;
      return 1;
    }

    boost::asio::io_service io_service;

    // resolver配合query使用，query第一个参数为IP，第二个参数为端口或服务名（比如http）
    // 通过resolver可以得到endpoint，endpoint包含ip和端口信息，表示一个网络节点
    // https://blog.csdn.net/zyd_15221378768/article/details/79722888
    tcp::resolver resolver(io_service);
    tcp::resolver::query query(argv[1], "daytime");
    tcp::resolver::iterator endpoint_iterator = resolver.resolve(query);

    // 连接到服务端
    tcp::socket socket(io_service);
    boost::asio::connect(socket, endpoint_iterator);

    for (;;)
    {
      boost::array<char, 128> buf;
      boost::system::error_code error;

      size_t len = socket.read_some(boost::asio::buffer(buf), error);

      if (error == boost::asio::error::eof)
        break; // Connection closed cleanly by peer.
      else if (error)
        throw boost::system::system_error(error); // Some other error.

      std::cout.write(buf.data(), len);
    }
  }
  catch (std::exception& e)
  {
    std::cerr << e.what() << std::endl;
  }

  return 0;
}
```

#### Daytime.2 - A synchronous TCP daytime server

``` c++
#include <ctime>
#include <iostream>
#include <string>
#include <boost/asio.hpp>

using boost::asio::ip::tcp;

std::string make_daytime_string()
{
  using namespace std; // For time_t, time and ctime;
  time_t now = time(0);
  return ctime(&now);
}

int main()
{
  try
  {
    boost::asio::io_service io_service;

    // 在13端口监听
    tcp::acceptor acceptor(io_service, tcp::endpoint(tcp::v4(), 13));

    for (;;)
    {
      // 接受一个连接请求
      tcp::socket socket(io_service);
      acceptor.accept(socket);

      std::string message = make_daytime_string();

      boost::system::error_code ignored_error;
      boost::asio::write(socket, boost::asio::buffer(message), ignored_error);
    }
  }
  catch (std::exception& e)
  {
    std::cerr << e.what() << std::endl;
  }

  return 0;
}
```

#### Daytime.3 - An asynchronous TCP daytime server

``` c++
#include <ctime>
#include <iostream>
#include <string>
#include <boost/bind.hpp>
#include <boost/shared_ptr.hpp>
#include <boost/enable_shared_from_this.hpp>
#include <boost/asio.hpp>

using boost::asio::ip::tcp;

std::string make_daytime_string()
{
  using namespace std; // For time_t, time and ctime;
  time_t now = time(0);
  return ctime(&now);
}

class tcp_connection
  : public boost::enable_shared_from_this<tcp_connection>
{
public:
  typedef boost::shared_ptr<tcp_connection> pointer;

  static pointer create(boost::asio::io_service& io_service)
  {
    return pointer(new tcp_connection(io_service));
  }

  tcp::socket& socket()
  {
    return socket_;
  }

  void start()
  {
    message_ = make_daytime_string();

    boost::asio::async_write(socket_, boost::asio::buffer(message_),
        boost::bind(&tcp_connection::handle_write, shared_from_this(),
          boost::asio::placeholders::error,
          boost::asio::placeholders::bytes_transferred));
  }

private:
  tcp_connection(boost::asio::io_service& io_service)
    : socket_(io_service)
  {
  }

  void handle_write(const boost::system::error_code& /*error*/,
      size_t /*bytes_transferred*/)
  {
  }

  tcp::socket socket_;
  std::string message_;
};

class tcp_server
{
public:
  tcp_server(boost::asio::io_service& io_service)
    : acceptor_(io_service, tcp::endpoint(tcp::v4(), 13))
  {
    start_accept();
  }

private:
  void start_accept()
  {
    tcp_connection::pointer new_connection =
      tcp_connection::create(acceptor_.get_io_service());

    // 接受连接，指定回调函数
    acceptor_.async_accept(new_connection->socket(),
        boost::bind(&tcp_server::handle_accept, this, new_connection,
          boost::asio::placeholders::error));
  }

  void handle_accept(tcp_connection::pointer new_connection,
      const boost::system::error_code& error)
  {
    if (!error)
    {
      // 处理请求
      new_connection->start();
    }

    // 接受下一个连接
    start_accept();
  }

  tcp::acceptor acceptor_;
};

int main()
{
  try
  {
    boost::asio::io_service io_service;
    tcp_server server(io_service);
    io_service.run();
  }
  catch (std::exception& e)
  {
    std::cerr << e.what() << std::endl;
  }

  return 0;
}
```

#### Daytime.4 - A synchronous UDP daytime client

``` c++
#include <iostream>
#include <boost/array.hpp>
#include <boost/asio.hpp>

using boost::asio::ip::udp;

int main(int argc, char* argv[])
{
  try
  {
    if (argc != 2)
    {
      std::cerr << "Usage: client <host>" << std::endl;
      return 1;
    }

    boost::asio::io_service io_service;

    udp::resolver resolver(io_service);
    udp::resolver::query query(udp::v4(), argv[1], "daytime");
    udp::endpoint receiver_endpoint = *resolver.resolve(query);

    udp::socket socket(io_service);
    socket.open(udp::v4());

    boost::array<char, 1> send_buf  = {{ 0 }};
    socket.send_to(boost::asio::buffer(send_buf), receiver_endpoint);

    boost::array<char, 128> recv_buf;
    udp::endpoint sender_endpoint;
    size_t len = socket.receive_from(
        boost::asio::buffer(recv_buf), sender_endpoint);

    std::cout.write(recv_buf.data(), len);
  }
  catch (std::exception& e)
  {
    std::cerr << e.what() << std::endl;
  }

  return 0;
}
```

#### Daytime.5 - A synchronous UDP daytime server

``` c++
#include <ctime>
#include <iostream>
#include <string>
#include <boost/array.hpp>
#include <boost/asio.hpp>

using boost::asio::ip::udp;

std::string make_daytime_string()
{
  using namespace std; // For time_t, time and ctime;
  time_t now = time(0);
  return ctime(&now);
}

int main()
{
  try
  {
    boost::asio::io_service io_service;

    udp::socket socket(io_service, udp::endpoint(udp::v4(), 13));

    for (;;)
    {
      boost::array<char, 1> recv_buf;
      udp::endpoint remote_endpoint;
      boost::system::error_code error;
      socket.receive_from(boost::asio::buffer(recv_buf),
          remote_endpoint, 0, error);

      if (error && error != boost::asio::error::message_size)
        throw boost::system::system_error(error);

      std::string message = make_daytime_string();

      boost::system::error_code ignored_error;
      socket.send_to(boost::asio::buffer(message),
          remote_endpoint, 0, ignored_error);
    }
  }
  catch (std::exception& e)
  {
    std::cerr << e.what() << std::endl;
  }

  return 0;
}
```

#### Daytime.6 - An asynchronous UDP daytime server

``` c++
#include <ctime>
#include <iostream>
#include <string>
#include <boost/array.hpp>
#include <boost/bind.hpp>
#include <boost/shared_ptr.hpp>
#include <boost/asio.hpp>

using boost::asio::ip::udp;

std::string make_daytime_string()
{
  using namespace std; // For time_t, time and ctime;
  time_t now = time(0);
  return ctime(&now);
}

class udp_server
{
public:
  udp_server(boost::asio::io_service& io_service)
    : socket_(io_service, udp::endpoint(udp::v4(), 13))
  {
    start_receive();
  }

private:
  void start_receive()
  {
    socket_.async_receive_from(
        boost::asio::buffer(recv_buffer_), remote_endpoint_,
        boost::bind(&udp_server::handle_receive, this,
          boost::asio::placeholders::error,
          boost::asio::placeholders::bytes_transferred));
  }

  void handle_receive(const boost::system::error_code& error,
      std::size_t /*bytes_transferred*/)
  {
    if (!error || error == boost::asio::error::message_size)
    {
      boost::shared_ptr<std::string> message(
          new std::string(make_daytime_string()));

      socket_.async_send_to(boost::asio::buffer(*message), remote_endpoint_,
          boost::bind(&udp_server::handle_send, this, message,
            boost::asio::placeholders::error,
            boost::asio::placeholders::bytes_transferred));

      start_receive();
    }
  }

  void handle_send(boost::shared_ptr<std::string> /*message*/,
      const boost::system::error_code& /*error*/,
      std::size_t /*bytes_transferred*/)
  {
  }

  udp::socket socket_;
  udp::endpoint remote_endpoint_;
  boost::array<char, 1> recv_buffer_;
};

int main()
{
  try
  {
    boost::asio::io_service io_service;
    udp_server server(io_service);
    io_service.run();
  }
  catch (std::exception& e)
  {
    std::cerr << e.what() << std::endl;
  }

  return 0;
}
```

###　Daytime.7 - A combined TCP/UDP asynchronous server

``` c++
#include <ctime>
#include <iostream>
#include <string>
#include <boost/array.hpp>
#include <boost/bind.hpp>
#include <boost/shared_ptr.hpp>
#include <boost/enable_shared_from_this.hpp>
#include <boost/asio.hpp>

using boost::asio::ip::tcp;
using boost::asio::ip::udp;

std::string make_daytime_string()
{
  using namespace std; // For time_t, time and ctime;
  time_t now = time(0);
  return ctime(&now);
}

class tcp_connection
  : public boost::enable_shared_from_this<tcp_connection>
{
public:
  typedef boost::shared_ptr<tcp_connection> pointer;

  static pointer create(boost::asio::io_service& io_service)
  {
    return pointer(new tcp_connection(io_service));
  }

  tcp::socket& socket()
  {
    return socket_;
  }

  void start()
  {
    message_ = make_daytime_string();

    boost::asio::async_write(socket_, boost::asio::buffer(message_),
        boost::bind(&tcp_connection::handle_write, shared_from_this()));
  }

private:
  tcp_connection(boost::asio::io_service& io_service)
    : socket_(io_service)
  {
  }

  void handle_write()
  {
  }

  tcp::socket socket_;
  std::string message_;
};

class tcp_server
{
public:
  tcp_server(boost::asio::io_service& io_service)
    : acceptor_(io_service, tcp::endpoint(tcp::v4(), 13))
  {
    start_accept();
  }

private:
  void start_accept()
  {
    tcp_connection::pointer new_connection =
      tcp_connection::create(acceptor_.get_io_service());

    acceptor_.async_accept(new_connection->socket(),
        boost::bind(&tcp_server::handle_accept, this, new_connection,
          boost::asio::placeholders::error));
  }

  void handle_accept(tcp_connection::pointer new_connection,
      const boost::system::error_code& error)
  {
    if (!error)
    {
      new_connection->start();
    }

    start_accept();
  }

  tcp::acceptor acceptor_;
};

class udp_server
{
public:
  udp_server(boost::asio::io_service& io_service)
    : socket_(io_service, udp::endpoint(udp::v4(), 13))
  {
    start_receive();
  }

private:
  void start_receive()
  {
    socket_.async_receive_from(
        boost::asio::buffer(recv_buffer_), remote_endpoint_,
        boost::bind(&udp_server::handle_receive, this,
          boost::asio::placeholders::error));
  }

  void handle_receive(const boost::system::error_code& error)
  {
    if (!error || error == boost::asio::error::message_size)
    {
      boost::shared_ptr<std::string> message(
          new std::string(make_daytime_string()));

      socket_.async_send_to(boost::asio::buffer(*message), remote_endpoint_,
          boost::bind(&udp_server::handle_send, this, message));

      start_receive();
    }
  }

  void handle_send(boost::shared_ptr<std::string> /*message*/)
  {
  }

  udp::socket socket_;
  udp::endpoint remote_endpoint_;
  boost::array<char, 1> recv_buffer_;
};

int main()
{
  try
  {
    boost::asio::io_service io_service;
    tcp_server server1(io_service);
    udp_server server2(io_service);
    io_service.run();
  }
  catch (std::exception& e)
  {
    std::cerr << e.what() << std::endl;
  }

  return 0;
}
```

  [1]: https://www.boost.org/doc/libs/1_60_0/doc/html/boost_asio.html