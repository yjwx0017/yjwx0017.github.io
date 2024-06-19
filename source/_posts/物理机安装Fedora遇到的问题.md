title: 物理机安装Fedora遇到的问题
categories: 日志
tags: [Linux,Fedora]
date: 2016-10-27 12:44:00
---
前几天为电脑新加了块硬盘，并且将Fedora安装到了新硬盘上。以下是安装使用过程中遇到的问题，在此记录一下。

## 新加硬盘后Windows无法启动

网上搜了之后，据说是数据线的问题，于是换了根数据线，谢天谢地，好像是很久以前组装电脑买的主板带了两根数据线。换了另一根数据线后成功开机了，1T的硬盘分了500G给Windows，剩下的留给Linux。

## 安装Fedora时提示自动分区失败

原因是从光盘启动时，启动项选择的是UEFI模式，不使用UEFI模式则自动分区成功，安装一切顺利。安装光盘是使用Fedora官网下载的Fedora-24-Xfce-Live镜像刻录的。


<!--more-->


## 进入Windows后感觉特别卡甚至死机

百度一番后，说是电源问题。但是我的电源怎么会突然就出问题呢。后来试着换了一根电源线接到硬盘上，结果问题就解决了，估计是接口接触不良吧。

## locate命令

Windows下习惯了Everything搜索的舒爽体验。据说Linux下`locate`命令也能达到这样的效果。使用之前，记得先执行一下`updatedb`命令。

## 无线上网问题

台式机上装的是一个USB接口的无线网卡，Fedora下遇到以下问题：网速奇慢无比，但是绝大多数情况连网卡都识别不出来，只有一次速度是正常。估计是Fedora对这个无线网卡的驱动支持不太好。所以在淘宝上搜了一个明确支持Linux的无线网卡，到货后，问题解决，上网一切正常。

## 查询软件包的安装位置

查询dnf命令安装的软件包的安装位置：

`rpm -ql <软件包名>`

## Fedora安装VirtualBox虚拟机

从VirtualBox官网下载rpm安装包，安装之后，创建虚拟机会提示错误，大体意思是还没有安装VirtualBox的内核驱动，按要求执行`/sbin/vboxconfig`。但执行会出现错误。需要先安装编译内核驱动所需的一些软件包，如下：

```
dnf -y update  # 更新所有软件包，包括内核版本
dnf install kernel
dnf install kernel-devel
dnf install kernel-headers
dnf install gcc
```

## Linux版VirtualBox中安装Ghost版Windows 7

新建虚拟机时，硬盘选择固定空间而不是动态增长的，否则安装完系统后无法启动。

## Chromium安装flash插件

打开优酷视频某个视频，会提示没有安装插件并且显示一个“安装”链接，点进去打开Flash插件的下载页面，将下载的.tar.gz包内的.json和.so文件拷贝至`/usr/lib64/chromium-browser/PepperFlash`文件夹下，注意.json文件也是必不可少的。