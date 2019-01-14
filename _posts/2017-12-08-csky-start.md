---
layout: post
title: csky 开发环境搭建
categories: [blog, buildroot, csky]
description: 记录的是随心所欲
keywords: blog, buildroot, csky
---
# csky 开发环境搭建

## csky 介绍

https://c-sky.github.io 一块国产开发板，使用国产处理器，必须支持一下

购买了开发板后，板子上默认集成 4MB SPI Flash 用于存放 bootloader 和播放器程序，所以开机后就是执行 BootLoader 和播放器程序，启动界面如下 串口打印 ：

```bash
XNts_port=2 not support, please input 1.
Partition Version :  102
Partition Count   :  7
Write Protect     :  TRUE
CRC32 Enable      :  TRUE
Table CRC32       :  646B2E8D
==============================================================================================
ID NAME    FS      CRC32     START    TOTAL_SIZE   MAIN_SIZE  USED_SIZE      Use%  RES_SIZE
==============================================================================================
0  BOOT    RAW     0FC882F2  00000000     128 KB     128 KB   75332  B      57%       0 MB
1  TABLE   RAW               00020000     512  B     512  B     512  B     100%       0 MB
2  LOGO    RAW     4CEDB02C  00020200   65024  B   65024  B   29801  B      45%       0 MB
3  KERNEL  RAW     C2246841  00030000    2176 KB    2176 KB    2168 KB      99%       0 MB
4  ROOT    CRAMFS  4046AEB9  00250000     512 KB     512 KB       8 KB       1%       0 MB
5  THEME   CRAMFS  00BCA597  002d0000     832 KB     832 KB     792 KB      95%       0 MB
6  DATA    MINIFS  FF000000  003a0000     384 KB     384 KB       1  B       0%       0 MB
----------------------------------------------------------------------------------------------


GxLoader v1.9 20140509 

cpu family      : CSKY
chip model      : gx6605s
board type      : 6605s
memory size     : 64 MB
Flash type      : w25q32
Flash size      : 4 MB
error: can't support 60HZ with PAL.
warning: board-init not call function enable_dac, will open cvbs & ypbpr default.
Hit any key to stop autoboot: 2 

boot> 
```

注意使用官方提供的 usb 线，我用其他的线，出现无法识别的问题。

官方提供的 usb 镜像 https://gitlab.com/c-sky/buildroot/-/jobs/24633630/artifacts/raw/output/images/usb.img  是 HDMI 的输出， 串口无法使用，所以搭建环境自己编译串口镜像：

| csky_gx6605s_defconfig       | 使用串口作为终端   |
| ---------------------------- | ---------- |
| csky_gx6605s_fbcon_defconfig | 使用HDMI作为终端 |

## 准备编译环境

docker + debian

1. ### 安装 docker

```bash
sudo apt install docker.io
```

2. ### 获取 debian 镜像

```bash
sudo docker run -it debian:stable
```

按照 c sky 的 wiki 配置环境：

```bash
image: debian:stable
before_script:
    - dpkg --add-architecture i386
    # The container has no package lists, so need to update first
    - apt-get update -qq
    - apt-get install -y -qq --no-install-recommends
        build-essential locales bc ca-certificates file rsync gcc-multilib
        git bzr cvs mercurial subversion libc6:i386 unzip wget cpio
    # To be able to generate a toolchain with locales, enable one UTF-8 locale
    - sed -i 's/# \(en_US.UTF-8\)/\1/' /etc/locale.gen
    - /usr/sbin/locale-gen
```

## 下载代码并编译

```bash
git clone https://github.com/c-sky/buildroot.git
cd buildroot
make csky_gx6605s_defconfig
make
```

编译完成后会生成 usb.img

```bash
chmod a+x /home/royce/buildroot/output/build/_fakeroot.fs
PATH="/home/royce/buildroot/output/host/bin:/home/royce/buildroot/output/host/sbin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin" /home/royce/buildroot/output/host/bin/fakeroot -- /home/royce/buildroot/output/build/_fakeroot.fs
rootdir=/home/royce/buildroot/output/target
table='/home/royce/buildroot/output/build/_device_table.txt'
/usr/bin/install -m 0644 support/misc/target-dir-warning.txt /home/royce/buildroot/output/target/THIS_IS_NOT_YOUR_ROOT_FILESYSTEM
>>>   Executing post-image script board/nationalchip/gx66xx/post-image.sh
vfat(boot.vfat): adding file 'uImage' as 'uImage' ...
vfat(boot.vfat): adding file 'gx6605s.dtb' as 'gx6605s.dtb' ...
hdimage(usb.img): adding partition 'boot' (in MBR) from 'boot.vfat' ...
hdimage(usb.img): adding partition 'rootfs' (in MBR) from 'rootfs.ext4' ...
hdimage(usb.img): writing MBR
root@ade85b049363:/home/royce/buildroot#
```

```bash
root@ade85b049363:/home/royce/buildroot# ls output/images/
boot.vfat  gx6605s.dtb  rootfs.ext2  rootfs.ext4  rootfs.tar  uImage  usb.img
```

## 烧写运行 

使用 dd 命令烧写到 usb，插上板子开机运行，效果如图：

![启动完成](/images/blog/c_sky_booted.png)



