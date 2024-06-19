title: Qt for WebAssembly 在浏览器中运行Qt应用程序
categories: 日志
tags: [Qt for WebAssembly,Qt]
date: 2020-12-13 11:16:00
---
[https://wiki.qt.io/Qt_for_WebAssembly][1]

## 安装 Emscripten SDK

从GitHub下载emsdk

[https://github.com/emscripten-core/emsdk][2]

Qt版本和Emscripten版本的对应关系参考文首的链接。

安装1.38.27版本的Emscripten

``` shell
./emsdk install sdk-fastcomp-1.38.27-64bit
```

## 编译Qt源代码

下载Qt5.14源代码[http://download.qt.io/archive/qt/][3]

配置源代码

``` shell
./configure -xplatform wasm-emscripten -nomake examples -prefix $PWD/qtbase
```

<!--more-->

编译

``` shell
make module-qtbase module-qtdeclarative [other modules]
```

## 编译和运行应用程序

对自己编写的应用程序进行编译

``` shell
/path/to/qtbase/bin/qmake && make
```

编译后会生成以下文件

```
Name	        Description
appname.wasm	Main application binary
appname.js	Emscripten JavaScript runtime
appname.html	Html container
qtloader.js	Qt JavaScript runtime
```

启动Web服务器，比如`python3 -m http.server`。

然后在浏览器中输入Web服务器地址，就能访问Qt应用程序。

  [1]: https://wiki.qt.io/Qt_for_WebAssembly
  [2]: https://github.com/emscripten-core/emsdk
  [3]: http://download.qt.io/archive/qt/