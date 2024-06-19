title: libcurl 笔记
categories: 日志
tags: [libcurl]
date: 2022-04-19 15:07:00
---
[https://curl.haxx.se/libcurl/c/libcurl-tutorial.html][1]

## 基础

初始化，参数指定要初始化的模块：

``` c++
curl_global_init(CURL_GLOBAL_ALL);
```

如果没有调用 `curl_global_init`，`curl_easy_perform` 会自动调用。

``` c++
CURL_GLOBAL_WIN32
CURL_GLOBAL_SSL
```

当不再使用 libcurl 时调用：

``` c++
curl_global_cleanup();
```

init 和 cleanup 避免重复调用，应只调用一次。

查询libcurl支持的特性：

``` c++
curl_version_info();
```

libcurl 提供两种接口：

 1. easy interface  - 函数以 curl_easy 为前缀，同步、阻塞调用。（let you do single transfers with a synchronous and blocking function call）
 2. multi interface - 支持在一个线程中多个传输任务，异步。（allows multiple simultaneous transfers in a single thread）

<!--more-->

### Easy Interface

首先获得一个句柄，该句柄不能被多个线程共享。

``` c++
easyhandle = curl_easy_init();
```

然后设置选项，用于接下来的传输。设置的选项将一直保持，直到下一次修改。

``` c++
curl_easy_setopt()
```

重置选项：

``` c++
curl_easy_reset()
```

复制句柄，连同其选项：

``` c++
curl_easy_duphandle()
```

常用的选项，比如 URL ：

``` c++
curl_easy_setopt(handle, CURLOPT_URL, "http://domain.com/");
```

大多数选项是字符串，libcurl内部会维护一个字符串的副本。

假设想要接收上面指定的链接收到的数据，可以设置自己的回调函数处理该数据，默认输出到 stdout 。

回调函数：

``` c++
size_t write_data(void* buffer, size_t size, size_t nmemb, void* userp);
```

设置回调函数：

``` c++
curl_easy_setopt(easyhandle, CURLOPT_WRITEFUNCTION, write_data);
```

收到数据时，write_data 函数被调用，如果不设置回调函数。

设置回调函数时，第四个参数接收的是自定义数据，比如可以将文件句柄传入第四个参数：

``` c++
curl_easy_setopt(easyhandle, CURLOPT_WRITEDATA, &internal_struct);
```

选项设置完毕，接下来就是执行任务：

``` c++
success = curl_easy_perform(easyhandle);
```

调用后 libcurl 将连接到远程地址，传输数据，当接收到数据时，之前设置的回调函数会被调用。回调函数会被调用一次或多次，回调函数应返回已处理的字节数，如果不等于传入的字节数 libcurl 将中止传输并返回错误代码。

传输完毕后，可继续使用同一个 handle 继续传输另一个文件，后台 libcurl 会重用连接。

对于某些协议，比如FTP，下载一个文件需要登录、设置传输模式、改变当前目录然后传输文件数据等一系列步骤，libcurl 会自动处理这些步骤，开发人员只需要提供这个需要下载的文件的URL。

libcurl 是线程安全的。

传输可能会因为某些原因失败。

调试用选项：

``` c++
CURLOPT_VERBOSE
CURLOPT_HEADER
CURLOPT_DEBUGFUNCTION
```

如果了解某些传输协议，学习协议的 RFC 文档，用利于更好地理解和使用 libcurl。

libcurl 试图使 api 的使用独立于具体的协议，比如使用 FTP 协议上传文件和使用 HTTP 的 PUT  请求上传到 HTTP 服务器使用的 api 调用是相似的。

如同下载，上传数据时，需要一个回调函数提供要上传的数据，调用函数返回0，表示上传结束。

回调函数格式：

``` c++
size_t read_function(char* bufptr, size_t size, size_t nitems, void* userp);
```

设置回调函数：

``` c++
curl_easy_setopt(easyhandle, CURLOPT_READFUNCTION, read_function);
```

设置回调函数第四个参数：

``` c++
curl_easy_setopt(easyhandle, CURLOPT_READDATA, &filedata);
```

设置 libcurl 表明要上传：

``` c++
curl_easy_setopt(easyhandle, CURLOPT_UPLOAD, 1L);
```

某些协议需要知道要上传文件的总大小，file_size 的变量类型必须为 curl_off_t：

``` c++
curl_easy_setopt(easyhandle, CURLOPT_INFILESIZE_LARGE, file_size);
```

当调用 `curl_easy_perform()` 时，libcurl执行传输。

许多协议需要提供用户名密码才能上传或下载。

libcurl 提供了几种方法来指定用户名密码：

在URL中指定用户名和密码：`protocol://user:password@example.com/path/`。

如果在用户名密码中包含奇怪的字符（URL只能包含ASCII字符），需要使用URL编码：%XX，XX为十六进制数字。

使用选项设置用户名密码：

``` c++
curl_easy_setopt(easyhandle, CURLOPT_USERPWD, "myname:thesecret")`;
```

设置代理（proxy）的用户名密码：

``` c++
curl_easy_setopt(easyhandle, CURLOPT_PROXYUSERPWD, "myname:thesecret");
```

在 Unix 下用户名密码可以保存在某个配置文件中 `$HOME/.netrc`。

可以设置选项使 libcurl 使用配置文件中的用户名和密码：

``` c++
curl_easy_setopt(easyhandle, CURLOPT_NETRC, 1L);
```

配置文件 .netrc 中的内容例如：

```
machine myhost.mydomain.com
login userlogin
password secretword
```

设置 SSL 私钥密码：

``` c++
curl_easy_setopt(easyhandle, CURLOPT_KEYPASSWD, "keypassword");
```

#### HTTP 认证（Authentication）

上文展示了如何设置用户名和密码，当使用 HTTP 协议时，有很多不同的方法提交认证，可以自由控制 libcurl 使用哪一种。

默认的认证方法是“Basic”：明文发送用户名密码，使用 base64 编码，这是不安全的。

目前 libcurl 可以使用以下方法：Basic、Digest、NTLM、Negotiate(SPNEGO)。

可以设置选项告诉 libcurl 使用哪一种：

``` c++
curl_easy_setopt(easyhandle, CURLOPT_HTTPAUTH, CURLAUTH_DIGEST);
```

代理的认证方法 `CURLOPT_PROXYAUTH`。

可以使用逻辑或运算符指定多个认证方法，libcurl 将自动选择最安全的一种：

``` c++
curl_easy_setopt(easyhandle, CURLOPT_HTTPAUTH, CURLAUTH_DIGEST | CURLAUTH_BASIC);
```

也可以使用 `CURLAUTH_ANY`。

#### HTTP POST请求

使用 libcurl 发送如同 <form> 表单中的 POST 请求，方法如下：

``` c++
char* data = "name=daniel&project=curl";
curl_easy_setopt(easyhandle, CURLOPT_POSTFIELDS, data);
curl_easy_setopt(easyhandle, CURLOPT_URL, "http://posthere.com/");
curl_easy_perform(easyhandle);  // 执行POST请求
```

POST 请求中使用二进制数据，需要指定二进制数据的长度，因为 libcurl 无法使用 strlen() 获得二进制数据的长度。

``` c++
// header
struct curl_slist* headers = NULL;
headers = curl_slist_append(headers, "Content-Type:text/xml");
// 二进制数据
curl_easy_setopt(easyhandle, CURLOPT_POSTFIELDS, binaryptr);
// 二进制数据的长度
curl_easy_setopt(easyhandle, CURLOPT_POSTFIELDSIZE, 23L);
// 设置自定义header
curl_easy_setopt(easyhandle, CURLOPT_HTTPHEADER, headers);
// 执行POST请求
curl_easy_perform(easyhandle);
// 释放内存
curl_slist_free_all(headers);
```

#### Muti-part formpost

用于 POST 请求中包含非常大的二进制数据时，二进制数据可以分多个部分添加。

用法见文首原文链接。

``` c++
curl_mime *multipart = curl_mime_init(easyhandle);
curl_mimepart *part = curl_mime_addpart(multipart);
curl_mime_name(part, "name");
curl_mime_data(part, "daniel", CURL_ZERO_TERMINATED);
part = curl_mime_addpart(multipart);
curl_mime_name(part, "project");
curl_mime_data(part, "curl", CURL_ZERO_TERMINATED);
part = curl_mime_addpart(multipart);
curl_mime_name(part, "logotype-image");
curl_mime_filedata(part, "curl.png");

/* Set the form info */
curl_easy_setopt(easyhandle, CURLOPT_MIMEPOST, multipart);

curl_easy_perform(easyhandle); /* post away! */

/* free the post data again */
curl_mime_free(multipart);
```

因为 easyhandle 会记住上次设置的选项。所以如果要回到 GET 请求模式，需要手动切换回来。

``` c++
curl_easy_setopt(easyhandle, CURLOPT_HTTPGET, 1L);
```

仅仅设置 `CURLOPT_POSTFIELDS` 为空字符串或 NULL 并不会退出POST模式，这样仅仅是不包含任何数据的 POST 请求。

#### 显示传输进度

libcurl 有一个内置的显示进度的功能，如果开启，将在终端窗口显示一个进度条。
设置选项开启：`CURLOPT_NOPROGRESS` 设为0为开启，默认值为1。

对于大多数应用程序来说，内置的进度条功能没什么用。

设置使用回调函数 `CURLOPT_PROGRESSFUNCTION`。

回调函数的格式为：

``` c++
int progress_callback(
	  void* clientp,   // 使用 CURLOPT_PROGRESSDATA 设置的自定义数据
    double dltotal,
    double dlnow,
    double ultotal,
    double ulnow);
```

### Multi Interface

有两种版本：`multi_socket`、`select()`。

首先创建 `easy handles`。

然后调用 `curl_multi_init` `curl_multi_add_handle` `curl_multi_perform`。

`curl_multi_perform` 是异步的，仅执行当前可以完成的操作，然后将控制权返回给调用者。该函数永远不会阻塞，你需要不断调用该函数直到所有的传输都已完成。

最好的方式是使用 `select()` 函数来判断何时需要调用 `curl_multi_perform` 。

相关函数：

``` c++
curl_multi_fdset()
curl_multi_remove_handle()
curl_multi_info_read()
```

## 示例

### ftpget

``` c++
#include <stdio.h>

#include <curl/curl.h>

/* <DESC>
 * Get a single file from an FTP server.
 * </DESC>
 */

struct FtpFile {
  const char *filename;
  FILE *stream;
};

static size_t my_fwrite(void *buffer, size_t size, size_t nmemb, void *stream)
{
  struct FtpFile *out = (struct FtpFile *)stream;
  if(!out->stream) {
    /* open file for writing */
    out->stream = fopen(out->filename, "wb");
    if(!out->stream)
      return -1; /* failure, can't open file to write */
  }
  return fwrite(buffer, size, nmemb, out->stream);
}


int main(void)
{
  CURL *curl;
  CURLcode res;
  struct FtpFile ftpfile = {
    "curl.tar.gz", /* name to store the file as if successful */
    NULL
  };

  curl_global_init(CURL_GLOBAL_DEFAULT);

  curl = curl_easy_init();
  if(curl) {
    /*
     * You better replace the URL with one that works!
     */
    curl_easy_setopt(curl, CURLOPT_URL,
                     "ftp://ftp.example.com/curl/curl-7.9.2.tar.gz");
    /* Define our callback to get called when there's data to be written */
    curl_easy_setopt(curl, CURLOPT_WRITEFUNCTION, my_fwrite);
    /* Set a pointer to our struct to pass to the callback */
    curl_easy_setopt(curl, CURLOPT_WRITEDATA, &ftpfile);

    /* Switch on full protocol/debug output */
    curl_easy_setopt(curl, CURLOPT_VERBOSE, 1L);

    res = curl_easy_perform(curl);

    /* always cleanup */
    curl_easy_cleanup(curl);

    if(CURLE_OK != res) {
      /* we failed */
      fprintf(stderr, "curl told us %d\n", res);
    }
  }

  if(ftpfile.stream)
    fclose(ftpfile.stream); /* close the local file */

  curl_global_cleanup();

  return 0;
}
```

### http-post

``` c++
/* <DESC>
 * simple HTTP POST using the easy interface
 * </DESC>
 */
#include <stdio.h>
#include <curl/curl.h>

int main(void)
{
  CURL *curl;
  CURLcode res;

  /* In windows, this will init the winsock stuff */
  curl_global_init(CURL_GLOBAL_ALL);

  /* get a curl handle */
  curl = curl_easy_init();
  if(curl) {
    /* First set the URL that is about to receive our POST. This URL can
       just as well be a https:// URL if that is what should receive the
       data. */
    curl_easy_setopt(curl, CURLOPT_URL, "http://postit.example.com/moo.cgi");
    /* Now specify the POST data */
    curl_easy_setopt(curl, CURLOPT_POSTFIELDS, "name=daniel&project=curl");

    /* Perform the request, res will get the return code */
    res = curl_easy_perform(curl);
    /* Check for errors */
    if(res != CURLE_OK)
      fprintf(stderr, "curl_easy_perform() failed: %s\n",
              curl_easy_strerror(res));

    /* always cleanup */
    curl_easy_cleanup(curl);
  }
  curl_global_cleanup();
  return 0;
}
```

### 10-at-a-time

``` c++
/* <DESC>
 * Download many files in parallel, in the same thread.
 * </DESC>
 */

#include <errno.h>
#include <stdlib.h>
#include <string.h>
#ifndef WIN32
#  include <unistd.h>
#endif
#include <curl/curl.h>

static const char *urls[] = {
  "https://www.microsoft.com",
  "https://opensource.org",
  "https://www.google.com",
  "https://www.yahoo.com",
  "https://www.ibm.com",
	/* ... */
};

#define MAX_PARALLEL 10 /* number of simultaneous transfers */
#define NUM_URLS sizeof(urls)/sizeof(char *)

static size_t write_cb(char *data, size_t n, size_t l, void *userp)
{
  /* take care of the data here, ignored in this example */
  (void)data;
  (void)userp;
  return n*l;
}

static void add_transfer(CURLM *cm, int i)
{
  CURL *eh = curl_easy_init();
  curl_easy_setopt(eh, CURLOPT_WRITEFUNCTION, write_cb);
  curl_easy_setopt(eh, CURLOPT_URL, urls[i]);
  curl_easy_setopt(eh, CURLOPT_PRIVATE, urls[i]);
  curl_multi_add_handle(cm, eh);
}

int main(void)
{
  CURLM *cm;
  CURLMsg *msg;
  unsigned int transfers = 0;
  int msgs_left = -1;
  int still_alive = 1;

  curl_global_init(CURL_GLOBAL_ALL);
  cm = curl_multi_init();

  /* Limit the amount of simultaneous connections curl should allow: */
  curl_multi_setopt(cm, CURLMOPT_MAXCONNECTS, (long)MAX_PARALLEL);

  for(transfers = 0; transfers < MAX_PARALLEL; transfers++)
    add_transfer(cm, transfers);

  do {
    curl_multi_perform(cm, &still_alive);

    while((msg = curl_multi_info_read(cm, &msgs_left))) {
      if(msg->msg == CURLMSG_DONE) {
        char *url;
        CURL *e = msg->easy_handle;
        curl_easy_getinfo(msg->easy_handle, CURLINFO_PRIVATE, &url);
        fprintf(stderr, "R: %d - %s <%s>\n",
                msg->data.result, curl_easy_strerror(msg->data.result), url);
        curl_multi_remove_handle(cm, e);
        curl_easy_cleanup(e);
      }
      else {
        fprintf(stderr, "E: CURLMsg (%d)\n", msg->msg);
      }
      if(transfers < NUM_URLS)
        add_transfer(cm, transfers++);
    }
    if(still_alive)
      curl_multi_wait(cm, NULL, 0, 1000, NULL);

  } while(still_alive || (transfers < NUM_URLS));

  curl_multi_cleanup(cm);
  curl_global_cleanup();

  return EXIT_SUCCESS;
}
```


  [1]: https://curl.haxx.se/libcurl/c/libcurl-tutorial.html