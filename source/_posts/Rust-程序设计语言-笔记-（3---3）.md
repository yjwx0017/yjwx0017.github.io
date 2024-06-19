title: Rust 程序设计语言 笔记 （3 / 3）
categories: 日志
tags: [Rust]
date: 2024-04-18 14:51:00
---
[Rust 程序设计语言 中文版][1]

## 智能指针

- Box<T>，用于在堆上分配值
- Rc<T>，一个引用计数类型，其数据可以有多个所有者
- Ref<T> 和 RefMut<T>，通过 RefCell<T> 访问

RefCell<T> 是一个在运行时而不是在编译时执行借用规则的类型。

## 使用 Box<T> 指向堆上的数据

box 允许你将一个值放在堆上而不是栈上。留在栈上的则是指向堆数据的指针。

``` rust
fn main() {
    let b = Box::new(5);
    println!("b = {}", b);
}
```

举例，使用box实现一个递归类型：

``` rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let list = Cons(1,
        Box::new(Cons(2,
            Box::new(Cons(3,
                Box::new(Nil))))));
}
```

Box<T> 类型是一个智能指针，因为它实现了 Deref trait，它允许 Box<T> 值被当作引用对待。

``` rust
fn main() {
    let x = 5;
    let y = Box::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

当 Box<T> 值离开作用域时，由于 Box<T> 类型 Drop trait 的实现，box 所指向的堆数据也会被清除。

<!--more-->

## Rc<T> 引用计数智能指针

以下代码不能通过编译：

``` rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let a = Cons(5,
        Box::new(Cons(10,
            Box::new(Nil))));
    let b = Cons(3, Box::new(a));
    let c = Cons(4, Box::new(a));
}
```

Cons 成员拥有其储存的数据，所以当创建 b 列表时，a 被移动进了 b 这样 b 就拥有了 a。接着当再次尝试使用 a 创建 c 时，这不被允许，因为 a 的所有权已经被移动。

使用 Rc<T> 修复代码：

``` rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    let b = Cons(3, Rc::clone(&a)); // 也可以调用 a.clone()
    let c = Cons(4, Rc::clone(&a));

    // 输出引用计数
    println!("{}", Rc::strong_count(&a));
}
```

## RefCell<T>

Rc<T> 允许相同数据有多个所有者；Box<T> 和 RefCell<T> 有单一所有者。

RefCell<T> 允许在运行时执行可变借用检查，所以我们可以在即便 RefCell<T> 自身是不可变的情况下修改其内部的值。

关于 Cell 和 RefCell ：https://zhuanlan.zhihu.com/p/659426524

## 循环引用

避免引用循环：将 Rc<T> 变为 Weak<T>。

## 并发编程

``` rust
use std::thread;
use std::time::Duration;

fn main() {
    // 使用 spawn 创建新线程
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    // 使用 join 等待所有线程结束
    handle.join().unwrap();
}
```

可以在参数列表前使用 move 关键字强制闭包获取其使用的环境值的所有权。这个技巧在创建新线程将值的所有权从一个线程移动到另一个线程时最为实用。

``` rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

以上代码如果不加 move 将无法通过编译，因为 Rust 会推断如何捕获 v，因为 println! 只需要 v 的引用，闭包尝试借用 v。然而这有一个问题：Rust 不知道这个新建线程会执行多久，所以无法知晓 v 的引用是否一直有效。

所以需要使用 move 强制转移所有权。

## 使用消息传递在线程间传送数据

一个日益流行的确保安全并发的方式是 消息传递（message passing），这里线程或 actor 通过发送包含数据的消息来相互沟通。这个思想来源于 Go 编程语言文档中 的口号：“不要通过共享内存来通讯；而是通过通讯来共享内存。”（“Do not communicate by sharing memory; instead, share memory by communicating.”）

Rust 中一个实现消息传递并发的主要工具是 通道（channel），Rust 标准库提供了其实现的编程概念。你可以将其想象为一个水流的通道，比如河流或小溪。如果你将诸如橡皮鸭或小船之类的东西放入其中，它们会顺流而下到达下游。

mpsc 是 多个生产者，单个消费者（multiple producer, single consumer）的缩写。

``` rust
use std::thread;
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        // 发送数据
        let val = String::from("hi");
        tx.send(val).unwrap();
        // val 的所有权被转移，无法再使用
    });

    // 接收数据
    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

send 函数获取其参数的所有权并移动这个值归接收者所有。这可以防止在发送后再次意外地使用这个值。

通过克隆发送者来创建多个生产者：

``` rust
let (tx, rx) = mpsc::channel();

let tx1 = tx.clone();
thread::spawn(move || {
    let vals = vec![
        String::from("hi"),
        String::from("from"),
        String::from("the"),
        String::from("thread"),
    ];

    for val in vals {
        tx1.send(val).unwrap();
        thread::sleep(Duration::from_secs(1));
    }
});

thread::spawn(move || {
    let vals = vec![
        String::from("more"),
        String::from("messages"),
        String::from("for"),
        String::from("you"),
    ];

    for val in vals {
        tx.send(val).unwrap();
        thread::sleep(Duration::from_secs(1));
    }
});

for received in rx {
    println!("Got: {}", received);
}
```

## Mutex

``` rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        let mut num = m.lock().unwrap();
        *num = 6;
    }

    println!("m = {:?}", m);
}
```

lock 调用 返回 一个叫做 MutexGuard 的智能指针。

MutexGuard 离开作用域时自动释放锁。

``` rust
use std::sync::{Mutex, Arc};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        // 增加引用计数，该局部变量的所有权将被转移到线程内
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

Rc<T> 并不能安全的在线程间共享，需要使用 Arc<T>。

counter 需要被多个线程共享，因此需要使用 Arc<T> 智能指针。

因为 counter 是不可变的，不过可以获取其内部值的可变引用；这意味着 Mutex<T> 提供了内部可变性，就像 Cell 系列类型那样。

使用 RefCell<T> 可以改变 Rc<T> 中的内容，同样的可以使用 Mutex<T> 来改变 Arc<T> 中的内容。

## 面向对象：封装

``` rust
pub struct AveragedCollection {
    list: Vec<i32>,
    average: f64,
}

impl AveragedCollection {
    pub fn add(&mut self, value: i32) {
        self.list.push(value);
        self.update_average();
    }

    pub fn remove(&mut self) -> Option<i32> {
        let result = self.list.pop();
        match result {
            Some(value) => {
                self.update_average();
                Some(value)
            },
            None => None,
        }
    }

    pub fn average(&self) -> f64 {
        self.average
    }

    fn update_average(&mut self) {
        let total: i32 = self.list.iter().sum();
        self.average = total as f64 / self.list.len() as f64;
    }
}
```

## 面向对象：多态

``` rust
// 定义 trait
trait Draw {
    fn draw(&self);
}

// Screen
struct Screen {
    components: Vec<Box<dyn Draw>>,
}

impl Screen {
    fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}

// Button 实现 trait
struct Button {
    width: u32,
    height: u32,
    label: String,
}

impl Draw for Button {
    fn draw(&self) {
        // 实际绘制按钮的代码
    }
}

// SelectBox 实现 trait
struct SelectBox {
    width: u32,
    height: u32,
    options: Vec<String>,
}

impl Draw for SelectBox {
    fn draw(&self) {
        // code to actually draw a select box
    }
}

fn main() {
    let screen = Screen {
        components: vec![
            Box::new(SelectBox {
                width: 75,
                height: 10,
                options: vec![
                    String::from("Yes"),
                    String::from("Maybe"),
                    String::from("No")
                ],
            }),
            Box::new(Button {
                width: 50,
                height: 10,
                label: String::from("OK"),
            }),
        ],
    };

    screen.run();
}
```

## 模式匹配 if let

``` rust
let favorite_color: Option<&str> = None;
let is_tuesday = false;
let age: Result<u8, _> = "34".parse();

if let Some(color) = favorite_color {
    println!("Using your favorite color, {}, as the background", color);
} else if is_tuesday {
    println!("Tuesday is green day!");
} else if let Ok(age) = age {
    if age > 30 {
        println!("Using purple as the background color");
    } else {
        println!("Using orange as the background color");
    }
} else {
    println!("Using blue as the background color");
}
```

## 模式匹配 while let

``` rust
let mut stack = Vec::new();

stack.push(1);
stack.push(2);
stack.push(3);

while let Some(top) = stack.pop() {
    println!("{}", top);
}
```

## 模式匹配 for

``` rust
let v = vec!['a', 'b', 'c'];

for (index, value) in v.iter().enumerate() {
    println!("{} is at index {}", value, index);
}
```

## 模式匹配 let

``` rust
let (x, y, z) = (1, 2, 3);
```

## 模式匹配 函数参数

``` rust
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({}, {})", x, y);
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
```

模式匹配的更多语法：https://rustwiki.org/zh-CN/book/ch18-03-pattern-syntax.html

## unsafe Rust

可以通过 unsafe 关键字来切换到不安全 Rust，接着可以开启一个新的存放不安全代码的块。

不安全 Rust 之所以存在，是因为静态分析本质上是保守的。

有时代码可能是合法的，但如果 Rust 编译器没有足够的信息来确定，它将拒绝该代码。

在这种情况下，可以使用不安全代码告诉编译器，“相信我，我知道我在干什么。”

这么做的缺点就是你只能靠自己了：如果不安全代码出错了，比如解引用空指针，可能会导致不安全的内存使用。

另一个 Rust 存在不安全一面的原因是：底层计算机硬件固有的不安全性。如果 Rust 不允许进行不安全操作，那么有些任务则根本完成不了。

使用不安全 Rust 可以做到以下操作：

- 解引用裸指针
- 调用不安全的函数或方法
- 访问或修改可变静态变量
- 实现不安全 trait
- 访问 union 的字段

## 解引用裸指针

``` rust
let mut num = 5;

let r1 = &num as *const i32;
let r2 = &mut num as *mut i32;

unsafe {
    println!("r1 is: {}", *r1);
    println!("r2 is: {}", *r2);
}
```

## 调用不安全函数或方法

``` rust
unsafe fn dangerous() {}

unsafe {
    dangerous();
}
```

## 使用 extern 函数调用外部代码

``` rust
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
```

## trait 中的关联类型

关联类型（associated types）是一个将类型占位符与 trait 相关联的方式，这样 trait 的方法签名中就可以使用这些占位符类型。

``` rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
}

impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        // --snip--
    }
}
```

重载 + 运算符：

``` rust
use std::ops::Add;

#[derive(Debug, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    // Add trait 有一个叫做 Output 的关联类型，它用来决定 add 方法的返回值类型。
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

fn main() {
    assert_eq!(Point { x: 1, y: 0 } + Point { x: 2, y: 3 },
               Point { x: 3, y: 3 });
}
```

## 完全限定语法与消歧义：调用相同名称的方法

``` rust
trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

struct Human;

impl Pilot for Human {
    fn fly(&self) {
        println!("This is your captain speaking.");
    }
}

impl Wizard for Human {
    fn fly(&self) {
        println!("Up!");
    }
}

impl Human {
    fn fly(&self) {
        println!("*waving arms furiously*");
    }
}

fn main() {
    let person = Human;
    Pilot::fly(&person);   // output: This is your captain speaking.
    Wizard::fly(&person);  // output: Up!
    person.fly();          // output: *waving arms furiously*
}
```

``` rust
trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String {
        String::from("Spot")
    }
}

impl Animal for Dog {
    fn baby_name() -> String {
        String::from("puppy")
    }
}

fn main() {
    println!("{}", Dog::baby_name());             // output: Spot
    println!("{}", <Dog as Animal>::baby_name()); // output: puppy
}

```

## 某个 trait 使用另一个 trait 的功能

``` rust
use std::fmt;

trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string();
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", output);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}
```

要想为某个类型实现 OutlinePrint trait，该类型必须也实现 Display trait。

## newtype 模式用于在外部类型上实现外部  trait

只要 trait 或类型对于当前 crate 是本地的话就可以在此类型上实现该 trait。

一个绕开这个限制的方法是使用 newtype 模式（newtype pattern）。

``` rust
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
```

## 类型别名

``` rust
type Kilometers = i32;

let x: i32 = 5;
let y: Kilometers = 5;

println!("x + y = {}", x + y);
```

类型别名的主要用途是减少重复。例如，可能会有很长的类型。

``` rust
type Thunk = Box<dyn Fn() + Send + 'static>;

let f: Thunk = Box::new(|| println!("hi"));

fn takes_long_type(f: Thunk) {
    // --snip--
}

fn returns_long_type() -> Thunk {
    // --snip--
}
```

std::io 有个类型别名声明：

``` rust
type Result<T> = std::result::Result<T, std::io::Error>;
```


``` rust
pub trait Write {
    fn write(&mut self, buf: &[u8]) -> Result<usize, Error>;
    fn flush(&mut self) -> Result<(), Error>;

    fn write_all(&mut self, buf: &[u8]) -> Result<(), Error>;
    fn write_fmt(&mut self, fmt: fmt::Arguments) -> Result<(), Error>;
}
```

以上代码等价于：

``` rust
pub trait Write {
    fn write(&mut self, buf: &[u8]) -> Result<usize>;
    fn flush(&mut self) -> Result<()>;

    fn write_all(&mut self, buf: &[u8]) -> Result<()>;
    fn write_fmt(&mut self, fmt: fmt::Arguments) -> Result<()>;
}
```

## 从不返回的 never type

Rust 有一个叫做 ! 的特殊类型。

``` rust
fn bar() -> ! {
    // --snip--
}
```

continue 的值是 !。

## 函数指针

``` rust
fn add_one(x: i32) -> i32 {
    x + 1
}

// 第一个参数为函数指针
fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}

fn main() {
    let answer = do_twice(add_one, 5);

    println!("The answer is: {}", answer);
}
```

使用 map 函数将一个数字 vector 转换为一个字符串 vector：

``` rust
// 使用闭包
let list_of_numbers = vec![1, 2, 3];
let list_of_strings: Vec<String> = list_of_numbers
    .iter()
    .map(|i| i.to_string())
    .collect();

// 使用函数
let list_of_numbers = vec![1, 2, 3];
let list_of_strings: Vec<String> = list_of_numbers
    .iter()
    .map(ToString::to_string)
    .collect();
```

## 返回闭包

``` rust
fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}
```

## 宏： 声明（Declarative）宏

从根本上来说，宏是一种为写其他代码而写代码的方式，即所谓的 元编程（metaprogramming）。

一个函数标签必须声明函数参数个数和类型。相比之下，宏能够接受不同数量的参数。

宏可以在编译器编译代码前展开。

``` rust
let v: Vec<u32> = vec![1, 2, 3];
```

vec! 宏的简化版本：

``` rust
#[macro_export]
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
```

`#[macro_export]` 标注说明，只要将定义了宏的 crate 引入作用域，宏就应当是可用的。

`$()` 捕获了符合括号内模式的值以用于替换后的代码。

`$x:expr` 匹配 Rust 的任意表达式，并将该表达式记作 `$x`。

`$()` 之后的逗号说明一个可有可无的逗号分隔符可以出现在 `$()` 所匹配的代码之后。紧随逗号之后的 `*` 说明该模式匹配零个或更多个 `*` 之前的任何模式。

当以 `vec![1, 2, 3];` 调用宏时，`$x` 模式与三个表达式 1、2 和 3 进行了三次匹配。

替换该宏调用所生成的代码会是下面这样：

``` rust
let mut temp_vec = Vec::new();
temp_vec.push(1);
temp_vec.push(2);
temp_vec.push(3);
temp_vec
```

## 用于从属性生成代码的过程宏

https://rustwiki.org/zh-CN/book/ch19-06-macros.html

## 最后的项目：构建多线程 Web 服务器

单线程版本：

``` rust
use std::io::prelude::*;
use std::net::TcpListener;
use std::net::TcpStream;
use std::fs;

fn main() {
    // 绑定
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    // 接受连接请求
    for stream in listener.incoming() {
        let stream = stream.unwrap();

        handle_connection(stream);
    }
}

fn handle_connection(mut stream: TcpStream) {
    // 接收数据
    let mut buffer = [0; 1024];
    stream.read(&mut buffer).unwrap();

    let get = b"GET / HTTP/1.1\r\n";

    let (status_line, filename) = if buffer.starts_with(get) {
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };

    // 读取文件内容
    let contents = fs::read_to_string(filename).unwrap();

    // 构建返回内容，格式化字符串
    let response = format!(
        "{}\r\nContent-Length: {}\r\n\r\n{}",
        status_line,
        contents.len(),
        contents
    );

    // 发送数据
    stream.write(response.as_bytes()).unwrap();
    stream.flush().unwrap();
}
```

改善吞吐量的方法：

- 线程池
- fork/join 模型
- 单线程异步 I/O 模型

我们会将池中线程限制为较少的数量，以防拒绝服务（Denial of Service， DoS）攻击；如果程序为每一个接收的请求都新建一个线程，某人向服务器发起千万级的请求时会耗尽服务器的资源并导致所有请求的处理都被终止。

使用线程池的多线程版本：

``` rust
// src/bin/main.rs
use hello::ThreadPool;

use std::io::prelude::*;
use std::net::TcpListener;
use std::net::TcpStream;
use std::fs;
use std::thread;
use std::time::Duration;

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        pool.execute(|| {
            handle_connection(stream);
        });
    }

    println!("Shutting down.");
}

fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 1024];
    stream.read(&mut buffer).unwrap();

    let get = b"GET / HTTP/1.1\r\n";
    let sleep = b"GET /sleep HTTP/1.1\r\n";

    let (status_line, filename) = if buffer.starts_with(get) {
        ("HTTP/1.1 200 OK\r\n\r\n", "hello.html")
    } else if buffer.starts_with(sleep) {
        thread::sleep(Duration::from_secs(5));
        ("HTTP/1.1 200 OK\r\n\r\n", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND\r\n\r\n", "404.html")
    };

    let contents = fs::read_to_string(filename).unwrap();

    let response = format!("{}{}", status_line, contents);

    stream.write(response.as_bytes()).unwrap();
    stream.flush().unwrap();
}
```

``` rust
// src/lib.rs
use std::thread;
use std::sync::mpsc;
use std::sync::Arc;
use std::sync::Mutex;

enum Message {
    NewJob(Job),
    Terminate,
}

pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Message>,
}

type Job = Box<dyn FnOnce() + Send + 'static>;

impl ThreadPool {
    /// 创建线程池。
    ///
    /// 线程池中线程的数量。
    ///
    /// # Panics
    ///
    /// `new` 函数在 size 为 0 时会 panic。
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        // 创建通道，用于线程池和线程之间的通信
        let (sender, receiver) = mpsc::channel();

        let receiver = Arc::new(Mutex::new(receiver));

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            // 创建线程，Worker::new 中线程被创建
            // 通道的接收端被传入线程
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }

        ThreadPool {
            workers,
            sender,
        }
    }

    // 在线程中执行函数f
    pub fn execute<F>(&self, f: F)
        where
            F: FnOnce() + Send + 'static
    {
        // 将函数作为消息发送到线程
        let job = Box::new(f);
        self.sender.send(Message::NewJob(job)).unwrap();
    }
}

// 销毁时清理
impl Drop for ThreadPool {
    fn drop(&mut self) {
        println!("Sending terminate message to all workers.");

        for _ in &mut self.workers {
            self.sender.send(Message::Terminate).unwrap();
        }

        println!("Shutting down all workers.");

        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);

            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}

struct Worker {
    id: usize,
    thread: Option<thread::JoinHandle<()>>,
}

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Message>>>) ->
        Worker {

        // 创建线程
        let thread = thread::spawn(move ||{
            loop {
                // 循环，从通道中接收消息，如果没有消息则阻塞
                let message = receiver.lock().unwrap().recv().unwrap();

                match message {
                    Message::NewJob(job) => {
                        println!("Worker {} got a job; executing.", id);

                        job();
                    },
                    Message::Terminate => {
                        println!("Worker {} was told to terminate.", id);

                        break;
                    },
                }
            }
        });

        Worker {
            id,
            thread: Some(thread),
        }
    }
}
```

  [1]: https://rustwiki.org/zh-CN/book/title-page.html