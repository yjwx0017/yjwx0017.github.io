title: C++11 Lambda表达式
categories: 日志
tags: [C++11,Lambda]
date: 2021-05-09 16:00:31
---
[https://www.cnblogs.com/jimodetiantang/p/9016826.html][1]
[https://blog.csdn.net/qq_26079093/article/details/90759175][2]

## 语法

[capture](parameters) mutable -> return-type { statement }

## 示例

``` c++
[] (int x, int y) { return x + y; } 
[] (int x, int y) -> int { int z = x + y; return z; }
```

## capture

```
[空]        没有任何函数对象参数。
[=]         函数体内可以使用 Lambda 所在范围内所有可见的局部变量（包括 Lambda 所在类的 this），并且是值传递方式（相当于编译器自动为我们按值传递了所有局部变量）。
[&]         函数体内可以使用 Lambda 所在范围内所有可见的局部变量（包括 Lambda 所在类的 this），并且是引用传递方式（相当于是编译器自动为我们按引用传递了所有局部变量）。
[this]      函数体内可以使用 Lambda 所在类中的成员变量。
[a]         将 a 按值进行传递。按值进行传递时，函数体内不能修改传递进来的 a 的拷贝，因为默认情况下函数是 const 的，要修改传递进来的拷贝，可以添加 mutable 修饰符。
[&a]        将 a 按引用进行传递。
[a, &b]     将 a 按值传递，b 按引用进行传递。
[=, &a, &b] 除 a 和 b 按引用进行传递外，其他参数都按值进行传递。
[&, a, b]   除 a 和 b 按值进行传递外，其他参数都按引用进行传递。
```


  [1]: https://www.cnblogs.com/jimodetiantang/p/9016826.html
  [2]: https://blog.csdn.net/qq_26079093/article/details/90759175