title: 实现一个FTP服务端（Qt）
categories: 日志
tags: [Qt]
date: 2024-05-16 13:56:00
---
## 源代码

[https://gitee.com/yjwx0017/ftpserver][1]

## 参考

- [RFC 959][2]
- [vsftpd][3]

## FTP 协议简介

命令连接负责传送命令，数据连接负责传送数据。

服务端启动后在21端口监听，接受客户端连接。

命令格式,例如：

```
client --> server: USER username\r\n
client <-- server: 331 User name okay, need password.\r\n
```

命令交互方式为：客户端发送一个命令，服务端返回响应（返回代码 + 描述）。

部分命令：

- USER命令 指定用户名
- PASS命令 指定密码
- PORT命令 主动模式，服务端将连接到客户端提供的IP地址和端口（数据连接）
- PASV命令 被动模式，服务端启动监听，接受客户端的连接（数据连接）
- TYPE命令 指定二进制模式还是ASCII模式
- CWD 命令 更改当前目录
- CDUP命令 切换到上级目录
- QUIT命令 断开连接
- DELE命令 删除文件
- RMD 命令 删除目录
- MKD 命令 创建目录
- PWD 命令 输出当前目录
- RNFR命令 指定重命名的源（Rename From）
- RNTO命令 指定重命名的目标（Rename To）
- LIST命令 列出当前目录下的所有内容
- NLST命令 列出当前目录下的所有内容（只包含名称）
- RETR命令 下载文件（Retrieve）
- STOR命令 上传文件（Store）
- STOU命令 上传文件（不覆盖重名文件， Store Unique）
- APPE命令 上传文件（以附加的方式， Append）
- REST命令 指定断点续传的位置（Restart）
- FEAT命令 查询支持的扩展特性（比如 UTF8 ）
- OPTS命令 启用禁用某特性（比如 OPTS UTF8 ON）

  [1]: https://gitee.com/yjwx0017/ftpserver
  [2]: https://datatracker.ietf.org/doc/html/rfc959
  [3]: https://security.appspot.com/vsftpd.html