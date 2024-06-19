title: Docker初探
categories: 日志
tags: [Docker]
date: 2016-11-18 13:43:00
---
![第一本Docker书][1]

## 简介

Docker是一个能够把开发的应用程序自动部署到容器的开源引擎。

Docker容器拥有很高的性能，同时同一台宿主机中也可以运行更多的容器，使用户可以尽可能充分地利用系统资源。

Docker设计的目的就是要加强开发人员写代码的开发环境与应用程序要部署的生产环境的一致性，从而降低那种“开发时一切都正常，肯定是运维的问题”的风险。

Docker的目标之一就是缩短代码从开发、测试到部署、上线运行的周期，让你的应用程序具备可移植性，易于构建，并易于协作。

容器是镜像的实例。容器是基于镜像启动起来的。镜像是Docker生命周期中的构建或打包阶段，而容器则是启动或执行阶段。


<!--more-->


## 安装

Fedora中安装Docker：

``` shell
sudo dnf install docker
```

当Docker软件包安装完毕后，默认会立即启动Docker守护进程。守护进程监听/var/run/docker.sock这个Unix套接字文件，来获取来自客户端的Docker请求。

Fedora中启动/停止Docker守护进程：

``` shell
sudo service docker start
sudo service docker stop
```

## 常用操作

### 运行容器

``` shell
sudo docker run -i -t ubuntu /bin/bash
```

`-i`标志保证容器中STDIN是开启的，持久的标准输入时交互式shell的“半边天”。

`-t`标志则是另外的“半边天”，它告诉Docker为要创建的容器分配一个伪tty终端。

总之如果要创建一个shell与容器进行交互就要加`-i -t`参数。

Docker首先检查本地是否存在ubuntu镜像，如果不存在则从 Docker Hub 远程下载该镜像。

### 容器命名

``` shell
sudo docker run --name bob_the_container -i -t ubuntu /bin/bash
```

### 查看当前系统中容器的列表

``` shell
sudo docker ps -a
```

### 停止正在运行的容器

``` shell
sudo docker stop bob_the_container
```

### 重新启动已经停止的容器

``` shell
sudo docker start bob_the_container 
sudo docker restart bob_the_container 
```

### 删除容器

``` shell
sudo docker rm bob_the_container 
```

### 附着到容器

``` shell
sudo docker attach bob_the_container 
```

### 创建守护式容器

``` shell
sudo docker run --name daemon_dave -d ubuntu /bin/sh -c \
  "while true; do echo hello world; sleep 1; done"
```

`-d` 参数会将容器放到后台运行。

获取容器的日志：

``` shell
sudo docker logs daemon_dave
```

### 查看容器内的进程

``` shell
sudo docker top daemon_dave
```

### Docker统计信息

``` shell
sudo docker stats daemon_dave daemon_kate
```

### 在容器内部运行进程

在容器中运行`touch /etc/new_config_file`命令：

``` shell
sudo docker exec -d daemon_dave touch /etc/new_config_file
```

在容器内运行交互命令：

``` shell
sudo docker exec -t -i daemon_dave /bin/bash
```

### 自动重启容器

``` shell
sudo docker run --restart=always --name daemon_dave -d ubuntu /bin/sh -c \
  "while true; do echo hello world; sleep 1; done"
```

`--restart=always`参数表示，无论容器的退出代码是什么，Docker都会自动重启该容器。

## 扩展阅读

[我眼中的 Docker（一）docker、vm、lxc][2]
[https://www.runoob.com/docker/docker-command-manual.html][3]
[http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html][4]


  [1]: /usr/uploads/2020/06/16329531.jpg
  [2]: http://blog.csdn.net/jcjc918/article/details/46486655
  [3]: https://www.runoob.com/docker/docker-command-manual.html
  [4]: http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html