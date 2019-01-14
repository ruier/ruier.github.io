---
layout: post
title: VxWorks 控制台添加 shell 命令
categories: [blog, VxWorks, qemu]
description: 记录的是随心所欲
keywords: VxWorks, work, qemu
---


# 添加 VxWorks 控制台 shell 命令 

VxWorks 系统的控制台可以直接执行系统中函数，直接输入函数名就可以直接调用。这是一种常见的 kernel module 调试方式。

对于一些经常使用的功能，可以直接写成固定的功能模块提供给 shell 调用。

## usr shell 

VxWorks 的 usr shell 在 WindRiver\vxworks-6.6\target\src\usr 中 usrLib.c

我们可以编写自己的命令，然后添加进去：

```diff
diff --git a/usrLib.c b/usrLib.c
index ac9704d..57b6564 100644
--- a/usrLib.c
+++ b/usrLib.c
@@ -435,6 +435,7 @@ void help (void)
     "version                        Print VxWorks version info, and boot line",
     "shConfig  [\"config\"]           Display or set shell configuration variables",
     "strFree   [address]            Free strings allocated within the shell (-1=all)",
+	"usr_shell_cmd                  user defined command  -- added by royce"
     "",
     "NOTE:  Arguments specifying 'task' can be either task ID or name.",
     NULL
@@ -3971,4 +3972,8 @@ MODULE_ID reld
     else
         return (NULL);
     }
-	
\ No newline at end of file
+	
+	void usr_shell_cmd(void)
+	{
+		printf("add your code here, this is just a demo\n");
+	}
\ No newline at end of file

```

进入 workbench development shell，编译当前的库：执行 make。

## make usr code 

```bash
o usrShellHistLib.c
creating D:/vxworks/WindRiver/vxworks-6.6/target/lib/ppc/PPC604/common/libos.a
a - devSplit.o
a - ramDiskCbio.o
a - statTbl.o
a - tarLib.o
a - usrDosFsOld.o
a - usrFdiskPartLib.o
a - usrFsLib.o
a - usrLib.o
a - usrRtpLib.o
a - usrRtpStartup.o
a - usrShellHistLib.o
a - usrTransLib.o
rm D:/vxworks/WindRiver/vxworks-6.6/target/lib/ppc/PPC604/common/objos/statTbl.c

```

## 重新编译 VxWorks BSP 工程 

因为静态库是链接进去到核心的，所以需要 rebuild，根据当前的项目 BSP 编译需要的组件库：我的是 x86

```
make CPU=PENTIUM TOOL=gnu
```

然后 rebuild BSP 生成 bootrom.bin 和 vxWorks..st

## 下载运行 

我的系统是模拟 x86 的 qemu，所以还需要自己编译 vxWorks.st ，然后替换：

```bash
#!/bin/bash

IMAGE_BOOTROM=BOOTROM.IMG
BIN_BOOTROM=bootrom.bin

IMAGE_VXWORKS=floppy.img
BIN_VXWORKS=vxWorks.st

update_bootrom()
{
        mkdir rom
        mount $IMAGE_BOOTROM rom
        cp $BIN_BOOTROM rom/bootrom.sys
        umount rom
        rm -rf rom
        echo done
}

update_vxworks()
{
        mkdir vx
        mount $IMAGE_VXWORKS vx
        cp $BIN_VXWORKS vx/
        rm -rf vx/test
        umount vx
        rm -rf vx
        echo done
}

update_bootrom
update_vxworks
```

最后运行结果：

![查看](/images/blog/qemu_vx_cmd.png)

