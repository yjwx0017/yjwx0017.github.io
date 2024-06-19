title: 编译Linux内核源码的过程中发生的故事
categories: 日志
tags: [Linux]
date: 2020-06-20 12:45:00
---
## 目的

构建环境，学习 《Linux设备驱动程序》 《Linux内核设计与实现》

## Linux环境

Fedora 24，内核版本4.5.5。

`Linux 4.5.5-300.fc24.x86_64`

## 下载内核源码

从[https://www.kernel.org][1]下载了4.19.128版本。

## 内核配置

使用`make help`查看所有make选项。

可以使用不同的方法进行配置，比如`make config`、`make menuconfig`、`make xconfig`等。


<!--more-->


`make xconfig`是基于Qt的一个配置工具。

执行`make xconfig`，按照错误提示安装依赖库。

``` shell
sudo install gcc-c++ qt qt-devel bison flex
make xconfig
```

最终在根目录生成`.config`文件。

## 编译遇到问题

执行`qmake`，提示编译器版本太低。

最终决定将Fedora从24升级到30。

## 升级Fedora

[https://www.cnblogs.com/suodingsuiji/p/6288064.html][2]

``` shell
dnf install dnf-plugin-system-upgrade
dnf system-upgrade download --releasever=25
dnf system-upgrade reboot
```

一开始使用`--releasever=30`，提示错误，遂决定逐版本升级。

从24升级到25需要下载大量的软件包，运行了一夜，早上发现提示错误：

```
警告：/var/lib/dnf/system-upgrade/python2-cssselect-0.9.2-1.fc25.noarch.rpm: 头V3 RSA/SHA256 Signature, 密钥 ID fdb19c98: NOKEY
Curl error (37): Couldn't read a file:// file for file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-x86_64 [Couldn't open file /etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-x86_64]
```

[https://blog.csdn.net/xdzfli/article/details/20225459][3]

根据这篇文章从[https://rpmfusion.org/keys][4]上下载25的key，结果不能使用，因为公钥不匹配。网上几番查询，百度、Bing轮番上阵（别问为什么不用Google），终于找到了所需的公钥文件。

[https://github.com/facebookincubator/rpm-backports/blob/master/rpms/mkosi/sources/RPM-GPG-KEY-fedora-25-x86_64][5]

执行`dnf system-upgrade reboot`，重启后又是漫长的升级过程，终于系统由24成功升级为25。

升级到26之后，发现可以编译内核了。

当前的版本：Fedora 26，内核版本4.16.11。

`Linux 4.16.11-100.fc26.x86_64`

## 编译内核

直接make有错误发生，按照提示安装了几个软件包。

``` shell
sudo dnf install elfutils-libelf-devel
sudo dnf install openssl-devel
make
sudo make modules_install
sudo make install
```

重启后，启动选项中新加了一项内核4.19.128的启动项，启动提示错误，无法进入桌面环境：

```
/dev/fedora/swap does not exist
/dev/mapper/fedora-root does not exist
```

网上查询一番，无果，以失败告终。

## 再一次尝试

在网上搜到一篇文章，说是在Fedora系统中有一个编译内核用的配置文件，可以用来替换.config。

[https://www.cnblogs.com/lufee/archive/2011/08/30/2160099.html][6]

`cp /boot/config-4.16.11-100.fc26.x86_64 .config`

执行`make oldconfig`和`make`，这次编译的时间非常漫长，最后导致硬盘空间不足了，导致编译失败。需要扩充虚拟机的硬盘空间，然后建立新分区、挂载至一个临时目录、将home文件夹的内容拷贝至临时目录、再将新分区卸载并挂载至home文件夹。

[https://zhidao.baidu.com/question/1865322402933792867.html][7]

解决了空间不足问题，继续`make`。

自此一切顺利，系统正常启动。看来以后得多研究研究内核配置。

有图为证：

![uname][8]

  [1]: https://www.kernel.org
  [2]: https://www.cnblogs.com/suodingsuiji/p/6288064.html
  [3]: https://blog.csdn.net/xdzfli/article/details/20225459
  [4]: https://rpmfusion.org/keys
  [5]: https://github.com/facebookincubator/rpm-backports/blob/master/rpms/mkosi/sources/RPM-GPG-KEY-fedora-25-x86_64
  [6]: https://www.cnblogs.com/lufee/archive/2011/08/30/2160099.html
  [7]: https://zhidao.baidu.com/question/1865322402933792867.html
  [8]: /usr/uploads/2020/06/2482608851.jpg