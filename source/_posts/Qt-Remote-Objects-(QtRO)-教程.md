title: Qt Remote Objects (QtRO) 教程
categories: 日志
tags: [Qt,QtRO]
date: 2022-12-29 12:10:00
---
## 简介

Qt Remote Objects (QtRO) 是 Qt 的一个进程间通信模块。

## 术语

`Source` 是指提供服务或提供功能供其他程序使用的对象，是 RPC 中的被调用端。

`Replica` 是指 `Source` 对象的代理对象，用于 RPC 中的调用端，对 `Replica` 的调用请求将被转发给 `Source` 对象。

## 示例1：Direct Connection using a Static Source

### 创建接口定义文件

创建接口定义文件 simpleswitch.rep ：

```
class SimpleSwitch
{
  PROP(bool currState=false);
  SLOT(server_slot(bool clientState));
};
```

### 修改 .pro 文件

```
// 引入 QtRO 模块
QT += remoteobjects
// 引入接口定义文件
REPC_SOURCE = simpleswitch.rep
```

<!--more-->

Qt 将使用 repc 工具编译该接口定义文件生成 C++ 代码。

生成的文件：

- rep_simpleswitch_source.h
- rep_simpleswitch_replica.h

rep_simpleswitch_source.h 用于 `Source` 端，需要继承其中的接口类，实现其中的虚函数。

rep_simpleswitch_replica.h 用于 `Replica` 端，是 `Source` 对象的代理对象。

### `Source` 端

实现 rep_simpleswitch_source.h 中接口类的虚函数，作为服务对象。

创建服务对象，并设置为可远程访问：

``` c++
SimpleSwitch srcSwitch; // create simple switch

QRemoteObjectHost srcNode(QUrl(QStringLiteral("local:replica"))); // create host node without Registry
srcNode.enableRemoting(&srcSwitch); // enable remoting/Sharing
```

### `Replica` 端

连接到服务端：

``` c++
QSharedPointer<SimpleSwitchReplica> ptr;

QRemoteObjectNode repNode; // create remote object node
repNode.connectToNode(QUrl(QStringLiteral("local:replica"))); // connect with remote host node

ptr.reset(repNode.acquire<SimpleSwitchReplica>()); // acquire replica of source from host node
```

获取到 SimpleSwitchReplica 对象指针之后，就可以像使用普通 Qt 对象那样使用该对象，该对象拥有和服务对象相同的接口函数（信号函数、槽函数等）。

客户端也可以不使用 rep_simpleswitch_replica.h ，而是使用 QRemoteObjectDynamicReplica 类来动态地与服务对象交互。

## 示例2：Connections to Remote Nodes using a Registry

第一个示例是采用直接连接的方式，即代理对象直接连接到服务对象。

另一种方式是使用注册中心，此时服务对象将自己注册到服务中心，客户端连接到注册中心，然后获取指定服务对象的代理对象。

服务端：
```
// 注册中心，可以在一个单独的进程中
QRemoteObjectRegistryHost regNode(QUrl(QStringLiteral("local:registry"))); // create node that hosts registy

// 服务对象
SimpleSwitch srcSwitch; // create simple switch

// 在注册中心上注册服务对象
QRemoteObjectHost srcNode(QUrl(QStringLiteral("local:replica")), QUrl(QStringLiteral("local:registry"))); // create node that will host source and connect to registry
srcNode.enableRemoting(&srcSwitch); // enable remoting of source object
```

客户端：

``` c++
QSharedPointer<SimpleSwitchReplica> ptr;

QRemoteObjectNode repNode(QUrl(QStringLiteral("local:registry")));

ptr.reset(repNode.acquire<SimpleSwitchReplica>()); // acquire replica of source from host node
```