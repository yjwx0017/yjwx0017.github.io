title: Linux 常用命令
categories: 日志
tags: [Linux,shell]
date: 2024-03-30 12:07:00
---
## 解包/重新打包 initrd 文件

动机：修改启动Logo

首先使用 binwalk 查看 initrd 文件的打包方式。

一种情况是整个文件是一个压缩文件，解压缩后是一个 cpio 归档文件。

另一种情况是文件分为两部分，开头是 cpio 归档文件，后面的部分是压缩文件（解压后也是 cpio 归档文件）。

压缩和解压缩根据压缩算法的类型使用相应的命令，此处以 lz4 为例。

### 情况一

``` bash
mkdir initrd_output
mkdir initrd_new

cd initrd_output

# 解压缩，并且解包 cpio
lz4 -dc ../initrd.img | cpio -id

# ... 修改解包出来的 initrd 文件系统

# 重新打包
find . | cpio -o -H newc | lz4 -l9 > ../initrd_new/initrd.img
```

### 情况二

使用 `binwalk initrd.img` 查看两部分内容的边界：

![截图][1]

本例中压缩文件的偏移值为 6906880 。

``` bash
mkdir initrd_output
mkdir initrd_new

cd initrd_output

# 解包第一部分 cpio 
cat ../initrd.img | cpio -id

# 解包第二部分
mkdir rootfs
cd rootfs
dd if=../../initrd.img bs=6906880 skip=1 | lz4 -dc | cpio -id --no-absolute-filenames

# ... 修改解包出来的 initrd 文件系统

# 打包第一部分
cd ..
find kernel/ | cpio -o -H newc > ../initrd_new/initrd.img

# 打包第二部分并附加
cd rootfs
find . | cpio -o -H newc | lz4 -l9 >> ../../initrd_new/initrd.img
```

<!--more-->

## 制作U盘启动盘

``` bash
fdisk -l                   # 确定U盘设备路径
umount /dev/sda*           # 取消挂载（以/dev/sda为例）
mkfs.vfat /dev/sda -I      # 格式化
dd if=xxx.iso of=/dev/sda  # 刻录镜像文件
```

## apt 查看安装包信息

``` bash
apt list --installed       # 查看已安装软件包
apt show xxx               # 显示软件包信息
apt-cache showpkg xxx      # 查看软件包依赖关系
dpkg -L xxx                # 查看软件包安装路径
```

## 解包/重新打包 deb 安装包

``` bash
dpkg -x xxx.deb output     # 解包安装内容
cd output
dpkg -e ../xxx.deb         # 解包控制信息
cd ..
dpkg -b output xxx.deb     # 重新打包
```

  [1]: http://www.zhouyuanchao.com/usr/uploads/2024/04/3068842815.png