title: 网站部署 Let's Encrypt SSL 证书
categories: 日志
tags: [SSL证书,Let's Encrypt]
date: 2024-04-08 12:26:00
---
[如何使用 Certbot 命令列工具建立免費的 TLS/SSL 頂層網域憑證][1]

## 安装 certbot

```
apt install certbot
```

## 生成证书

```
certbot certonly --manual --preferred-challenges http -m youremail@example.com -d www.zhouyuanchao.com
```

根据提示在网站服务器指定目录创建指定文件并包含指定内容，通过验证后会在本地生成证书相关文件：

- cert.pem
- chain.pem
- fullchain.pem
- privkey.pem

## 将证书部署到网站

以新网为例，要求上传三个文件：

- 证书链：chain.pem
- 公钥：cert.pem
- 私钥：privkey.pem

<!--more-->

## HTTPS 基本原理

数字证书由可信任的证书颁发机构颁发，包含证书所有者、颁发机构、有效期、公钥等信息。

数字证书相当于身份证，颁发机构相当于公安机关。

客户端向服务器发起连接请求，服务器向客户端发送数字证书，客户端验证数字证书的有效性。

客户端和服务器使用公钥和私钥加密通信协商一个会话密钥（客户端使用数字证书中服务器的公钥，服务器使用自己的私钥）。

非对称加密（使用公钥和私钥）速度慢用于协商会话密钥，对称加密（使用会话密钥）速度快用于数据的加密和解密。

  [1]: https://blog.miniasp.com/post/2021/02/11/Create-SSL-TLS-certificates-from-LetsEncrypt-using-Certbot