title: C++/CLI 教程
categories: 日志
tags: [C++/CLI]
date: 2021-08-29 10:13:00
---
《Visual C++/CLI 从入门到精通》 笔记

## 什么是C++/CLI

C++/CLI是C++和 .Net Framework 平台版本，它对标准C++进行了一些修改。标准C++允许的一些操作在C++/CLI中是不允许的（例如不能从多个基类继承），并进行了一些修改以支持.NET功能（例如接口和属性）以及兼容.Net运行时。

选择C++/CLI有两个重要理由。首先是互操作性，C++/CLI简化了将标准C++代码集成到.NET项目的步骤。其次是现在有了C++标准模板库（STL）的.NET版本，熟悉STL编码的人很容易迁移到.NET。

## 定义托管类

``` c++
ref class Animal
{
public:
    int legs;
};
```

ref关键字用于简化与 .NET Framework 组件的交互。在class关键字前添加ref，类就成了托管类。实例化对象时，会在“公共语言运行时”（CLR）堆上创建对象。对象的生存期由 .NET Framework 的垃圾回收器管理。

也可以定义非托管类，即不加ref关键字。

## 创建托管类对象

``` c++
Animal cat, dog;
```

## 句柄

在C++/CLI中，是“运行时”帮你管理内存，C++/CLI没有了传统的“指针概念”。相反是用句柄来包含变量的地址。使用gcnew操作符动态创建对象。

定义一个指向Animal对象的句柄：

``` c++
Animal ^dog = gcnew Animal;
dog->legs = 4;
```


<!--more-->


## 数组

``` c++
array<int> ^arr = gcnew array<int>(10); 
arr[0] = 23;
int x = arr[0];
```

## String 类型

String类不是像int或long那样的内建类型，它是 .NET Framework 的一部分，在System命名空间中。为了使用String，要在源代码文件顶部添加这一行语句：

``` c++
using namespace System;
```

String类的一些成员函数（比如Insert和Replace）表面上能修改字符串，但实际上是返回一个新String对象，其中包含修改好的内容。如需反复修改字符串，应该使用StringBuilder类（添加using namespace System::Text;来使用）。

## 使用DLL中的定义的托管类数据

``` c++
#using <mscorlib.dll>
```

## 跟踪引用

``` c++
int i = 5;
int %ri = i; // ri 是跟踪引用，即变量的别名
```

## 继承

``` c++
ref class MyDerivedClass : MyBaseClass
{
};
```

C++/CLI（和其他所有.NET语言）只支持公共继承，所以不需要使用public关键字。用了不会出错，但要记住它是多余的。

如果不指定基类，默认从System::Object派生。

子类重写基类函数必须使用override关键字。

## 使用非托管代码

``` c++
ref class ManagedClass
{
    UnmanagedClass *puc; // 内存的管理需要自己维护，使用delete释放
};
```

``` c++
ref class ManagedClass
{
    UnmanagedClass obj; // 错误，不支持混合类型
};
```

