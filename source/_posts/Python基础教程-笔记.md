title: Python基础教程 笔记
categories: 日志
tags: [Python]
date: 2021-08-05 12:50:00
---
《Python基础教程》笔记​

Python具有丰富和强大的库

## 数学运算

``` python
2**3      # 幂运算符
pow(2, 3) # 幂运算

abs(-10) # 绝对值

round(1.0 / 2.0) # 四舍五入到整数

import math
math.floor(32.9) # -> 32

from math import floor
floor(32.9)
```

## 变量

``` python
x = 3
print("Hello World!")
x = input("prompt:")
```

## 转换为字符串

str 类型

repr 函数

``` python
str(1000L) # -> 1000
repr(1000L) # -> 1000L
```

## 拼接字符串

``` python
"Hello " + "World!"
temp = 100
s = "Hello" + str(temp)

print ("Hello, \
    World") # -> "Hello, World"
```

## 长字符串

保留换行，和特殊字符比如'"：

``` python
""" string """
''' string '''
```

原始字符串:

``` python
r'Hello,\nWorld!'
```

Unicode字符串：

``` python
u'Hello World'
```

<!--more-->

## 基本类型

str int long

## 序列

``` python
a1 = ['Hello', 100]
str = 'Hello'
str[0]  # -> 'H'
str[-1] # -> 'o'

# 分片 第一二个参数分别是元素索引[index1, index2)
a1 = [1, 2, 3, 4, 5]
a1[0:3] # -> [1,2,3]
a1[0:]  # -> 0到结尾
a1[:]   # -> 所有元素
a2[0:3:1] # ->第三个参数步长，默认为1

# 序列连接
[1,2,3] + [4,5,6]
'Hello ' + 'World'

# Error，相同类型的序列才能相加
# [1,2,3] + 'Hello' 

# 序列乘以数字n : 序列重复n次
'python' * 4  # -> pythonpythonpythonpython

# 空序列
[]
[None] * 10 # None表示什么都没有 类似c++中的NULL

# 判断是否在序列中
permissions = 'rw'
'w' in permissions # -> True

# 内建函数 len min max
# len 返回序列中元素个数
min(2,4,6)

# 列表、元组属于序列
# 列表是可变序列
# 元组是不可变序列

# 元素赋值
x = [1, 1, 1]
x[1] = 2 # -> [1, 2, 1]

# 删除元素
names = ['Alice', 'Beth', 'Cecil']
del names[2]

# 分片赋值
names[1:] = ['a', 'b']

list('Hello') # 转换为可变序列
names.append('Hello')
names.count('Alice') # 元素出现的次数

# list tuple str 类型
a.extend(b)    # 拼接，修改a
a.index('who') # 查找元素 返回索引，找不到将引发异常
a.insert(index, value)
a.pop(index)   # 移除，如果不指定index，删除最后一个
a.remove(value)
a.reverse()
a.sort()
sorted(a) # 返回排序副本

# 内建比较函数 cmp()
cmp(100, 200) # -> -1
```

## 元组

元组不能修改

``` python
1,2,3   # -> (1,2,3)
(1,2,3) # -> (1,2,3)
()      # 空元组
(42,)   # 包含一个值的元组

# tuple 类型，非函数 
# 以一个序列作为参数并把它转换为元组
tuple([1,2,3]) # -> (1,2,3)
tuple('abc')   # -> ('a','b','c')
tuple((1,2,3)) # -> (1,2,3)
```

## 字符串方法

``` python
# find 没有找到返回-1
"With a moo-moo here, and a moo-moo there.".find('moo')
title = "Monty python"
title.find('Monty')
table = maketrans('ABC', 'abc')
word = 'ABC'
word.translate(table)
```

## 字典

字典是引用

``` python
phonebook = {'Alice':'2341', 'Beth':'9120}

# 空字典 
{}

# dict 类型
items = [('name','Gumby'), ('age',42)]
d = dict(items)
d = dict(name='Gumby', age=42)

# 基本字典操作
len(d)
d[k]
d[k] = v
del d[k]
k in d
```

## 导入模块

几种导入方式：

``` python
import module
module.function()

from module import function
function()

import math as foobar
foobar.sqrt(100)

from math import sqrt as foobar
foobar(100)
```

## 比较运算符

``` python
0<age<100
```

== 值相等 可以比较序列的值

is 引用相等 （避免用于比较类似数值和字符串这类不可变值）

## 逻辑运算符

and or not

## 三元运算符

类似c++中的?:

``` python
a = b and c or d
c if b else d
```

## 循环

``` python
# while循环
x=1
while x<=100:
    print x
    x+=1

# for 循环
words=['this','is','an','ex','parrot']
for word in words:
    print word

range(0, 10) # 结果是一个序列
```

## zip 函数

``` python
names = ['anne', 'beth', 'george', 'damon']
ages = [12, 45, 32, 102]
zip(names, ages)
# -> [('anne', 12), ('beth', 45), ('george', 32), ('damon', 102)]

for index, string in enumerate(strings):
    if 'xxx' in string:
        strings[index] = '[censored]'
```

函数 reversed sorted

循环的else子句 没有调用break时执行

``` python
for n in range(99, 81, -1)
    root = sqrt(n)
    if root == int(root):
        print n
        break
else:
    print "Didn't find it!"
```

## 列表推导式

``` python
[x*x for x in range(10)]
# -> [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```

## pass 语句

什么都不做的语句

## 删除名称

``` python
x = ["Hello", "World]
del x
```

## 执行字符串中的 Python 代码

``` python
exec("print('Hello, world!')") #Python3.0
```

## 命名空间

``` python
from math import sqrt
scope = {}
exec 'sqrt = 1' in scope
sqrt(4) # -> 2.0
scope['sqrt'] # -> 1
```

## eval

eval 计算表达式值

## 函数是否可调用

``` python
callable(x)
hasattr(func, __call__)  # Python3.0
```

## 定义函数

``` python
def hello(name):
    return 'Hello. ' + name + '!'

## 文档字符串

def语句后面、模块或者类的开头

``` python
def square(x):
    'Calculates the square of the number x.'
    return x * x

square.__doc__  # 输出 'Calculates ... '
help(square)

def func(greeting, name):
    print '%s, %s' % (greeting, name)

# 非位置参数（与位置无关），调用函数时，指明参数名
func(greeting='Hello', name='world')

# 参数默认值
def func(greeting='Hello', name='world'):
    pass

# *收集参数到元组
def func(*params):
    print params

func(1, 2, 3) # 输出(1, 2, 3)

# **收集关键字参数到字典
# 在调用函数时使用 * ** 将元组或字典解包成参数
```

vars() 返回当前作用域字典

globals() 返回全局作用域字典

locals()

## 给全局变量赋值

``` python
x = 1
def func():
    global x
    x = x + 1
```

## 其他

支持闭包

lambda 数学中表示匿名函数

一切用递归实现的功能都可以用循环实现，但有时递归更易读。

递归的一个局限，有耗尽栈空间的风险。

``` python
map(func, seq)    # 对序列中的每个元素应用函数
filter(func, seq) # 返回其函数为真的元素的列表

isinstance(object, tuple)
isinstance(object, dict)
```

type

isinstance

issubclass

## 为变量随机赋值

``` python
from random import choice
x = choice(['Hello, world!', [1,2,'e','e',4]])
```

## repr函数

输出变量文本表示

``` python
x = 'Fnord'
print repr(x)
```

## 创建类

``` python
__metaclass__ = type # 确定使用新式类，需要在模块或脚本开始的地方放置，python3不需要
class Person:
    def setName(self, name):
        self.name = name

    def getName(self):
        return self.name
    
    def greet(self):
        print "Hello, world! i'm %s." % self.name

# 类中的变量和self.变量不是同一个，前者所有实例共享，后者是实例的私有变量。
foo = Person() # 创建实例
foo.setName('Hello')
foo.greet()
Person.setName(foo, 'Hello')
```

私有实现方式：

前面加__，但仍然可以访问，s._类名__函数名()

指定超类：

``` python
class Filter:
# ...
class SPAMFilter(Filter):
# ...
issubclass(SPAMFilter, Filter) # -> True

# 获取基类
SPAMFilter.__bases__
# 一个对象属于哪个类
s.__class__
type(s) # 使用__metaclass__=type 或从object继承

# tc是否包含talk特性
hasattr(tc, 'talk')
# 是否可调用
callable(getattr(tc, 'talk', None))
hasattr(x, '__call__') # Python3.0
```

## 术语

特性 - 成员变量

方法 - 成员函数

## 异常处理

异常类 从Exception继承

引发异常 raise Exception 会自动创建实例

``` python
raise Exception('hyperdrive overload')
```

内建异常 exceptions模块

dir函数 列出模块的内容

``` python
import exceptions
dir(exceptions)
```

最重要的内建异常类：146页

捕获异常：

``` python
try:
    x = 12
    y = 0
    print x / y
except ZeroDivisionError:
    print "The second number can't be zero!"
except TypeError:
    print "That wasn't a number, was it?"
// 多个异常作为元组列出
except (ZeroDivisionError, TypeError, NameError):
    print 'Your number were bogus...'
```

重新抛出捕获的异常：

``` python
raise # 无参数
```

获得异常实例：

``` python
except (ZeroDivisionError, TypeError), e:
    print e

# Python3.0
except (ZeroDivisionError, TypeError) as e:
```

捕获所有异常：

``` python
except:
    print 'Something wrong happened...'
else:
    pass # 没有异常发生时才会执行
finally:
    pass # 不管是否发生异常，都会执行
```

## 特殊方法

__future__

两边带下划线的被称为魔法或特殊方法

实现这些方法，这些方法会在特殊情况下被Python调用

## 构造函数

``` python
def __init__(self):
    self.somevar = 42
```

析构函数：

__del__

因为有垃圾回收机制，所以避免使用__del__

调用父类的构造函数：

``` python
# 方法1
class SongBird(Bird):
    def __init__(self):
        Bird.__init__(self)
        self.sound = 'Squawk!'

# 方法2
class SongBird(Bird):
    def __init__(self):
        super(SongBird, self).__init__()
        self.sound = 'Squawk!'
```

## 静态方法和类成员方法

``` python
def smeth():
    pass
smeth = staticmethod(smeth)

def cmeth():
    pass
cmeth = classmethod(cmeth)

@staticmethod
def smeth():
    pass

@classmethod
def cmeth(cls):
    pass

MyClass.smeth()
MyClass.cmeth()
```

## 可迭代

一个实现了__iter__方法的对象是可迭代的，一个实现了next方法的对象则是迭代器。

内建函数iter可以从可迭代的对象中获得迭代器：

``` python
it = iter([1,2,3])
it.next() # -> 1
it.next() # -> 2

# 将迭代器或迭代对象转换为序列
ti = TestIterator()
list(ti) # -> [...]

## 生成器

任何包含yield语句的函数称为生成器。

生成器是逐渐产生结果的复杂递归算法的理想实现工具。

``` python
nested = [[1, 2], [3, 4], [5]]
def flatten(nested):
    for sublist in nested:
        for element in sublist:
            yield element

for num in flatten(nested):
    print num
```

## 充电时刻

``` python
# 告诉解释器除了从默认的目录中寻找之外，还需要从目录c:/python中寻找模块
import sys
sys.path.append('c:/python')
```

.py .pyw (windows系统)

.pyc 平台无关的，经过处理（编译）的，已经转换成Python能够更加有效地处理的文件。

导入模块的时候，其中的代码被执行。重复导入，不会重复执行。

.py 文件导入后文件名就是模块名作用域

主程序 __name__ -> '__main__'

import __name__ -> '__模块名__'

模块中的测试代码可以使用__name__判断是否执行测试代码。

pprint模块中pprint函数 提供更加智能的打印输出，比如列表分行输出。

sys.path

PYTHONPATH 环境变量

包 - 目录，目录中包含__init__.py文件

查看模块包含的内容可以使用dir函数，他会将对象（以及模块的所有函数、类、变量等）的所有特性列出

``` python
dir(sys) # -> 列表['a', 'b', ...]
from copy import *
copy.__all__ # 所有导出的名称
# 如果没有设定__all__ 用 import * 语句默认导出所有不以下划线开头的全局名称。
help(copy.copy) # copy模块中的copy函数
print copy.copy.__doc__

# 模块的源代码
print copy.__file__
```

标准库

```
-sys
argv
exit([arg])
modules
path
platform
stdin
stdout
stderr
-os
environ
system(command)
sep
pathsep
linesep
urandom(n)

webbrowser模块

-fileinput
input
filename()
lineno()
filelineno()
isfirstline()
isstdin()
nextfile()
close()
-collections
-heapq #堆
-time
-random
-shelve #序列化 保存到文件
-re #正则表达式
```

open 默认读模式

- + 参数可以用到其他任何模式中，指明读和写都是允许的。比如r+能在打开一个文件用来读写时使用。
- r 读模式
- w 写模式
- a 追加模式
- b 二进制模式 （可添加到其他模式中使用）
- rb 读取二进制文件

``` python
f = open(r'c:\file.txt', 'w')
f.write('0112345')
f.seek(5)
f.close()
f.read()

file.readline() # 读取一行 （包含换行符）
file.readlines()

# Open your file here
try:
# Write data to your file
finally:
    file.close()

# Python2.5 from __future__ import with_statement
# 自动close
with open("somefile.txt") as somefile:
```

with 上下文管理 __enter__ __exit__ contextlib模块

当到达文件末尾时，read方法返回一个空的字符串。

``` python
import fileinput
for line in fileinput.input(filename):
    pass
```

数据库

Python DB API的模块特性

- apilevel 所使用的Python DB API版本
- threadsafety 模块的线程安全等级
- paramstyle 在SQL查询中使用的参数风格

``` python
import sqlite3
conn = sqlite3.connect('somedatabase.db')
curs = conn.cursor()
conn.commit()
conn.close()
```

socket模块

``` python
# 服务器
import socket
s = socket.socket()
host = socket.gethostname()
port = 1234
s.bind((host, port))
s.listen(5)
while True:
    c.addr = s.accept()
    print 'Got connection from' , addr
    c.send('Thank you for connecting')
    c.close()

# 客户端
import socket
s = socket.socket()
host = socket.gethostname()
port = 1234
s.connect((host, port))
print s.recv(1024)
```

urllib urllib2模块

能让通过网络访问文件，就像那些文件存在于你的电脑上一样。通过一个简单的函数调用，几乎可以把任何URL所指向的东西用作程序的输入。

如果需要使用HTTP验证或cookie或者要为自己的协议写扩展程序的话，urllib2是个好的选择。

标准库中一些与网络相关的模块

服务器框架

- SocketServer
- BaseHTTPServer
- SimpleHTTPServer
- CGIHTTPServer
- SimpleXMLRPCServer
- DocXMLRPCServer

基于SocketServer的服务器

``` python
from SocketServer import TCPServer, StreamRequestHandler
class Handler(StreamRequestHandler):
    def handle(self):
        addr = self.request.getpeername()
        print 'Got connection from', addr
        self.wfile.write('Thank you for connection')

server = TCPServer(('', 1234), Handler)
server.serve_forever()
```

Twisted 一个非常强大的异步网络编程框架

``` python
import fileinput
for line in fileinput.input(filename):
    process(line)
    inplace=1 # 标准输出会被重定向到打开文件

import fileinput
for line in fileinput.input(inplace = 1):
    line = line.rstrip()
    num = fileinput.lineno()
    print '%-40s # %2i' % (line, num)
```

文件是一个可迭代对象

``` python
f = open(filename)
for line in f:
    process(line)
    f.close()

# 使用默认浏览器打开某个网址
import webbrowser
webbrowser.open('www.baidu.com')
```

集合 set

```
set([0, 1, 2, 3])
set会剔除重复元素
set的元素顺序是随意的
a = set([1, 2, 3])
b = set([2, 3, 4])
并集 a.union(b) a | b
交集 a.intersection(b) b & b
a.issuperset(b)
a.issubset()
a.difference(b) a - b
a.symmetric_difference(b) a ^ b
a.copy()

reduce(func, [a, b, c])
reduce执行以下步骤
tmp = func(a, b)
tmp = func(tmp, c)
```

time 模块

- time() 当前时间 - 1970至今的秒数
- localtime() 将秒数转换为日期元组，本地时间
- asctime() 将日期元组转换为字符串
- mktime() 将日期元组转换为本地时间（秒数） 与localtime()功能相反

sleep()

datetime 模块

支持日期和时间的算法

timeit 模块

帮助开发人员对代码段的执行时间进行计时

random 模块

- random() 0<= n < 1
- randrange([start], stop, [step]) [start, stop) 随机整数
- uniform(a, b) [a, b] 随机实数
- choice() 从给定序列中选择随机元素
- shuffle() 将给定（可变）序列的元素进行随机移位
- sample() 从给定序列中 选择给定数目的元素，同时确保元素互不相同

re 正则表达式模块

- compile(pattern, [flags]) 根据包含正则表达式的字符串创建模式对象
- search(pattern, string, [flags]) 在字符串中寻找模式，返回第一个匹配的MatchObject
- match(pattern, string, [flags]) 在字符串的开始处匹配模式，返回MatchObject
- split(pattern, string, [maxsplit=0]) 根据模式的匹配项来分割字符串，返回列表
- findall(pattern, string) 列出字符串中模式的所有匹配项，返回列表
- sub(pat, repl, string, [count=0])将字符串中所有pat匹配项用repl替换
- escape(string) 将字符串中所有特殊正则表达式字符转义

``` python
pattern = ...
re.sub(pattern, r'<em>\1</em>', 'Hello, *world*!')
```

\1 引用模式匹配中的组

模式匹配默认是贪婪模式

所有的重复运算符都可以通过在其后面加上一个问号变成非贪婪版本

string模块中的模板系统 template类

functools 通过部分参数来使用某个函数，稍后再为剩下的参数提供数值

difflib 这个库让你可以计算两个序列的相似程度，还能让你在一些序列中找出和提供的原始序列最像的那个。

可以用于创建简单的搜索程序

hashlib 如果为两个不同的字符串计算出了签名。几乎可以确保这两个签名完全不同。

加密和安全性 见 md5 sha 模块

csv 读写csv文件

timeit profile trace

itertools

logging

getopt optparse

cmd

项目3 万能的XML

``` python
from xml.sax.handler import ContentHandler
from xml.sax import parse

class MyHandler(ContentHandler):
parse('website.xml', MyHandler)

str.capitalize()
getattr()
callable()
os.makedirs('foo/bar/baz')
os.path.isdir()
a = ['hello']
*a + ['boys'] -> * (a + ['boys'])
```

项目4 新闻聚合

nntplib

NNTP(Network News Transfer Protocal,网络新闻组传输协议)

urllib

strftime('%y%m%d') # 年月日

strftime('%H%M%S') # 时分秒

``` python
servername = 'news.foo.bar' # 服务器地址
group = 'comp.lang.python.announce'
server = NNTP(servername)
ids = server.newnews(group, date, hour)[1] # 文章的ID号

head = server.head(id)[3]
for line in head:
    if line.lower().startswith('subject:'):
        subject = line[9:]
        break
body = server.body(id)[3] # 返回一个字符串列表
```

打印完所有文章后 调用server.quit()

list.extend(list)

list.append(object)

BBC新闻网页的HTML页面布局可能会变，如果这样的话就需要重写正则表达式。

在使用其他页面的时候也要注意这样的情况。要查看HTML源代码然后试着去找到适用的匹配模式。

```
file
str
unicode
zlib
gzip
bz2
zipfile
tarfile
shutil
```
