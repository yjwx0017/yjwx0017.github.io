title: Python 实例
categories: 日志
tags: [Python,psutil,argparse]
date: 2021-08-06 15:14:00
---
## 实例网站

[https://www.programcreek.com/python/][1]

## 读写文件

### 写文件

``` python
#!/usr/bin/env python3

with open('test.txt', 'w') as f:
    f.write('hello\n')
    f.write('python')

```

### 读文件

``` python
#!/usr/bin/env python3

with open('test.txt', 'r') as f:
    for l in f.readlines():
        print(l, end='')
```

## 遍历文件夹下的所有文件

``` python
#!/usr/bin/env python3

import pathlib
import pprint

pprint.pprint(list(pathlib.Path().rglob("*")))
```


<!--more-->


## 遍历所有文件，gbk转码为utf-8

``` python
#!/usr/bin/env python3

import pathlib

for path in pathlib.Path().rglob("*"):
    file_name = str(path.resolve())
    if not file_name.endswith(".h") or not file_name.endswith("*.cpp"):
        continue

    f1 = open(file_name, "rb")
    content = str(f1.read(), "gbk")
    f1.close()

    f2 = open(file_name, "wb")
    f2.write(content.encode("utf8"))
    f2.close()
    print(file_name)
```

## 日期时间文本转为时间戳

``` python
import datetime

datetimestr = "2021-01-01 00:00:00"
dt = datetime.datetime.strptime(datetimestr, "%Y-%m-%d %H:%M:%S")
timestamp = dt.replace(tzinfo=datetime.timezone.utc).timestamp()
```

## RESTful 服务器

以下代码使用python标准库，推荐使用更好用的Flask-RESTful。

``` python
#!/usr/bin/env python3

from http import *
from http.server import *
from urllib.parse import urlparse
import json

class HTTPRequestHandler(BaseHTTPRequestHandler):
    def do_POST(self):
        url = urlparse(self.path)
        path = url.path
        if path == "/test":
            rep = json.dumps({"key1": "test"})
            self.send_response(HTTPStatus.OK)
            self.send_header("Content-Type", "application/json")
            self.send_header("Content-Length", str(len(rep)))
            self.end_headers()
            self.wfile.write(rep)

server_address = ("0.0.0.0", 8080)
print("Serving HTTP on " + str(server_address))

httpd = HTTPServer(server_address, HTTPRequestHandler)
httpd.serve_forever()
```

## 使用SQLite数据库

``` python
import sqlite3

connection = sqlite3.connect("test.db")
connection.row_factory = sqlite3.Row
cursor = connection.cursor

# SQL语句
sql = "..."

# 执行SQL语句，如果是修改数据库内容，必须调用commit
try:
    cursor.execute(sql)
    connection.commit()
except BaseException as e:
    print("error: " + str(e))

# 执行查询语句时，获取单个记录
try:
    cursor.execute(sql)
    row = cursor.fetchone()
    if row:
        keys = row.keys()  # 所有字段名
        vals = list(row)   # 所有字段值
        data = dict(zip(keys, vals)) # 构成一个字典 
except BaseException as e:
    print("error: " + str(e))

# 执行查询语句时，获取所有记录
try:
    for row in cursor.execute(sql):
        keys = row.keys()  # 所有字段名
        vals = list(row)   # 所有字段值
        data = dict(zip(keys, vals)) # 构成一个字典 
except BaseException as e:
    print("error: " + str(e))
```

## 自动关机

### Windows

在指定时间到达时关机：

``` python
import os
import time
import datetime

# 晚上9点
shutdown_time = datetime.time(21, 0, 0)

print('System will shutdown at " + str(shutdown_time))

while True:
    current_time = datetime.datetime.now().time()
    if current_time >= shutdown_time:
        print('Shutdown now...')
        os.system('shutdown /s /t 5') # 5秒后关机
        break
    else:
        time.sleep(1)
```

指定进程退出后关机：

``` python
import psutil
import time
import os

def proc_exist(process_name):
    pids = psutil.pids()
    for pid in pids:
        try:
            if psutil.Process(pid).name() == process_name:
                return True
        except:
            continue
    
    return False

while True:
    if proc_exist('vcpkg.exe'):
        print('vcpkg.exe is running')
        time.sleep(5)
    else:
        print('shutdown now')
        os.system('shutdown /s /t 5')
        break
```

## 处理命令行参数

``` python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import argparse

parser = argparse.ArgumentParser(description="程序描述")

# 互斥组（组内的各参数不能同时使用）
group = parser.add_mutually_exclusive_group()
group.add_argument("-a", "--aaa", help="a参数描述", action="store_true")
group.add_argument("-b", "--bbb", help="b参数描述", type=int)
group.add_argument("-c", "--ccc", help="c参数描述")
args = parser.parse_args()

if args.aaa:
    pass
elif args.bbb:
    pass
elif arg.ccc:
    pass
else:
    parser.print_help()
```

The Python Standard Library » Generic Operating System Services » argparse — Parser for command-line options, arguments and sub-commands

## 使用 psutil 查看进程信息

### 安装

``` shell
sudo dnf install gcc python3-devel
pip install psutil
```

### 使用

``` python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import psutil

# 遍历所有进程，输出进程ID，CPU占比，内存，进程名称
for pid in psutil.pids():
    p = psutil.Process(pid)
    print("{},{},{},{}".format(
        p.pid, p.cpu_percent(), p.memory_info().rss, p.name()));
```

### 文档

[https://psutil.readthedocs.io/en/latest][2]


  [1]: https://www.programcreek.com/python/
  [2]: https://psutil.readthedocs.io/en/latest