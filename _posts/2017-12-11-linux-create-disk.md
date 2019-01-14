---
layout: post
title: Linux 下制作磁盘镜像
categories: [blog, linux]
description: 记录的是随心所欲
keywords: linux, work, qemu
---

# Linux 下制作虚拟软盘镜像

转载：http://wenix.blog.51cto.com/874806/364816

3.5寸1.44M软盘结构：

- ​    2面、80道/面、18扇区/道、512字节/扇区
- ​    扇区总数=2面 X  80道/面 X  18扇区/道  =  2880扇区
- ​    存储容量= 512字节/扇区X  2880扇区 =  1440 KB =1474560B

1. 创建虚拟软盘镜像文件

- ​    dd if=/dev/zero of=floppy.img bs=1474560 count=1


- ​    dd if=/dev/zero of=floppy.img bs=512 count=2880


- ​    dd if=/dev/zero of=floppy.img bs=1024 count=1440

2. 在软盘镜像文件上建立文件系统

​    下面两条命令中的任意一个可在软盘镜像上建立文件系统，可根据需要选择相应的文件系统：

- ​    mkfs.vfat floppy.img              /*建格式化为vfat文件系统*/


- ​    mkfs.ext2 floppy.img               /*建格式化为ext2文件系统*/

​    建立ext2文件系统时回询问： floppy.img is not a block special device. Proceed anyway? (y,n) y， 选y，回车。

3. 读写建立的软盘镜像

​    首先将软盘镜像挂载在一个文件夹中，用下列命令建立一个文件夹floppy：

- ​    mkdir floppy

   用下列命令将软盘镜像挂载到floppy文件夹：

- ​    mount floppy.img floppy -o loop

​    如果所用的系统不会自动识别文件系统的话
 mount 命令要加上 -t 选项：

- ​    mount floppy.img floppy -o loop -t vfat        /*如果软盘镜像为vfat文件系统*/


- ​    mount floppy.img floppy -o loop -t ext2        /*如果软盘镜像为ext2文件系统*/

​    然后就可以像操作普通文件夹那样对floppy文件夹进行操作了，如将 "kernel" 文件复制到里面：

- ​    cp kernel floppy    

​    查看其中的文件：

- ​    ls floppy                                      /*  输出 "kernel" */

​    操作完以后用下列命令将其卸载：

- ​    umount floppy.img


# linux 制作 flash image

http://bbs.chinaunix.net/thread-3667938-1-1.html

我在QEMU上加了flash emulation support, 然后模拟开发版 boot 全过程，先制作flash file:
文件制作如下：

```bash
dd if=/dev/zero of=flash.img bs=256k count=256
dd if=u-boot.bin of=flash.img bs=256k conv=notrunc
dd if=uImage of=flash.img bs=256k seek=2 conv=notrunc
dd if=roofs.img.gz of=flash.img bs=256K seek=8 conv=notrunc
```



运行后出现问题：
qemu-system-arm -M versatilepb -m 200 -nographic -pflash flash.img

qemu: fatal: Trying to execute code outside RAM or ROM at 0xffff0844

R00=fffcdf70 R01=ffff0000 R02=00000000 R03=00028250
R04=ffff0000 R05=fffcdf70 R06=ffff0000 R07=00000000
R08=008fff78 R09=fffe0000 R10=00028250 R11=00000000
R12=008ffff0 R13=fffcdf60 R14=ffff0844 R15=ffff0844
PSR=600001d3 -ZC- A svc32
Aborted

如果单独运行没问题：
qemu-system-arm -M versatilepb -m 200 -nographic -kernel u-boot.bin

好像uImage在flash文件里没放对，u-boot 时间延迟不起作用。有没有做过类似的flash文件，请帮帮忙！万分感谢