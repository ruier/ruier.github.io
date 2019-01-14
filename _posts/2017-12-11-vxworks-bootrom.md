---
title: VxWorks bootloader
categories: [blog, VxWorks]
description: 记录的是随心所欲
keywords: VxWorks, work, boot
---

VxWorks 操作系统自带 BootLoader， 类似于 u-boot，启动过程功能：

1. 初始化启动需要的外设
2. 设置好系统核心启动环境
3. 提供一个用户设置界面

其中用户界面同样跟 u-boot 类似，使用 command 命令来实现对应的相关操作。

## 启动界面 

VxWorks 的 bootrom 启动后界面如下：

```bash
boot device                       : ln
unit number                       : 0
processor number                  : 0
host name                         : mars
file name                         : c:\temp\vxWorks1
inet on ethernet (e)              : 90.0.0.50:ffffff00
inet on backplane (b)             :
host inet (h)                     : 90.0.0.1
gateway inet (g)                  :
user (u)                          : fred
ftp password (pw)(blank=use rsh)  :secret
flags (f)                         : 0x0
target name (tn)                  : phobos
startup script (s)                :
other (o)                         :
```

### boot device 

内置的启动设备器件，由于启动设备一般都是受限的，所以能够选择的启动设备也是有限的，可以通过 **h** 或者 **？** 命令查看当前支持的可启动设备

### unit number   

当前的启动设备编号，从 0 开始

### processor number  

backplane 多 target 系统上的核心号。一般都是 0 

### host name 

主机名

### file name 

主机上的启动文件名。 160-byte 限制

### inet on ethernet (e) 

IP 协议地址，以及 mask 的十六进制格式

### flags 

0x01 = Do not enable the system controller, even if the processor number is 0. (This option is board specific; refer to your target documentation)
0x02 = Load all VxWorks symbolsa , instead of just globals.
0x04 = Do not auto-boot.
0x08 = Auto-boot fast (short countdown).
0x20 = Disable login security.
0x40 = Use BOOTP to get boot parameters.
0x80 = Use TFTP to get boot image.
0x100 = Use proxy ARP.
0x200 = Use WDB agent.
0x400 = Set system to debug mode for the error detection and reporting
facility (depending on whether you are working on kernel modules
or user applications). 

## 命令详解 

| **Command**                 | **Description**                          |
| :-------------------------- | :--------------------------------------- |
| **h**                       | Help command—print a list of available boot commands. |
| **?**                       | Same as **h**.                           |
| **@**                       | Boot (load and execute file) using the current boot parameters. |
| **p**                       | Print the current boot parameter values. |
| **c**                       | Change the boot parameter values.        |
| **l**                       | Load the file using current boot parameters, but without executing. |
| **g**   *adrs*              | Go to (execute at) hex address adrs.     |
| **d**  *adrs*[, n]          | Display n words of memory starting at hex address adrs. If n is omitted, the default is 64. |
| **m**  *adrs*               | Modify memory at location adrs (hex). The system prompts for modifications to memory, starting at the specified address. It prints each address, and the current 16-bit value at that address, in turn. You can respond in one of several ways: <br> **ENTER**: Do not change that address, but continue prompting at the next address. <br> **number:** Set the 16-bit contents to number. <br> **.(dot):** Do not change that address, and quit. |
| **f** *adrs,nbytes,value*   | Fill nbytes of memory, starting at adrs with value. |
| **t**  *adrs1,adrs2,nbytes* | Copy nbytes of memory, starting at adrs1, to adrs2. |
| **s [ 0 \| 1 ]**            | Turn the CPU system controller ON (1) or OFF (0) (only on boards where the system controller can be enabled by software). |
| **e**                       | Display a synopsis of the last occurring VxWorks exception. |
| **v**                       | Display BSP and boot ROM version.        |
| **N**                       | Set Ethernet address.                    |

