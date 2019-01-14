---
title: Linux C 语言之 Hello World 详解
tags: Linux
---

# Linux C 语言之 Hello World 详解

[TOC] 

## 第一个 C 语言程序
学习 C 语言，大多数接触的第一个 C 语言程序便是经典的 Hello World 程序，程序的功能是在当前终端上打印 “Hello World” 字符串！
该程序的实现代码如下：
```
#include <stdio.h>

void main()
{
  printf("Hello World\n");
}
```

在 GNU/Linux 系统中，使用 gcc 编译器，编译并执行 helloworld 程序的指令为：

1.    通过 vi 编辑器编写上面代码，并保存为 helloworld.c
2.    使用 gcc 编译器编译源代码生成可执行文件 helloworld： gcc -o helloworld helloworld.c 
3.    执行当前目录中的 helloworld 程序：./helloworld

当前终端屏幕就会打印 Hello World，如下图：
![](https://img2018.cnblogs.com/blog/1457710/201809/1457710-20180914102701186-1408422976.png)

## 程序运行原理
GNU/Linux 系统中可执行程序都是 elf 格式二进制文件，该文件跟 Windows 系统的 exe 文件类似，通过 Linux 的 Shell 比如 Bash 加载到内存，由操作系统启动
新线程，然后开始执行。我们可以通过 file 命令查看目标文件的格式：
:~$ file helloworld
helloworld: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=203388067920d237ab234e8eb97714f56919799f, not stripped
### 编译，链接
从源代码生成可执行文件，需要很多步骤，最主要的步骤就是编译和链接。在我们上述的过程中，编译和链接都是由 gcc 程序完成的。
当然我们也可以分开来执行编译和链接过程：

gcc -c helloworld.c 
ld -o helloworld helloworld.o -dynamic-linker /lib64/ld-linux-x86-64.so.2 /usr/lib/x86_64-linux-gnu/crt1.o /usr/lib/x86_64-linux-gnu/crti.o /usr/lib/x86_64-linux-gnu/crtn.o -lc

可以看到，简单的 helloworld 程序依赖了大量的系统文件，其中主要的是程序运行环境相关的 crt （C RunTime Library）和 系统 c 语言库 glibc。
当然不同的平台这个步骤可能不同，可以在 gcc 命令中添加 -v 参数，查看编译和链接的完整步骤。
### 运行时
我们从代码可见的程序起始是 main 函数，但是编译器在编译链接的过程中，在我们的程序中添加了运行时代码，所以程序的起始并不是 main 函数了，可以通过 nm 查看我们的程序的地址和符号：
$ nm helloworld
```
0000000000600734 D __bss_start
0000000000600730 D __data_start
0000000000600730 W data_start
0000000000600570 d _DYNAMIC
0000000000600734 D _edata
0000000000600738 D _end
0000000000400464 T _fini
0000000000600708 d _GLOBAL_OFFSET_TABLE_
                 w __gmon_start__
0000000000400340 T _init
0000000000600570 d __init_array_end
0000000000600570 d __init_array_start
000000000040047c R _IO_stdin_used
0000000000400460 T __libc_csu_fini
00000000004003f0 T __libc_csu_init
                 U __libc_start_main@@GLIBC_2.2.5
00000000004003a0 T main
                 U puts@@GLIBC_2.2.5
00000000004003c0 T _start
```
可以看到 main 函数已经不是在程序的代码段开头了。可以通过对 gcc 添加 -Map 参数，来生成程序的 map 文件，方便我们查看程序的代码段，数据段等信息：
gcc -o helloworld helloworld.c -Wl,-Map,helloworld.map
通过 helloworld.map 可以清晰的看到 main 函数所在的 text 段，和相关的地址信息。

### 链接库
gcc 默认动态库的搜索路径搜索的先后顺序是：
  
1.    编译目标代码时指定的动态库搜索路径；
2.    环境变量LD_LIBRARY_PATH指定的动态库搜索路径；
3.    配置文件/etc/ld.so.conf中指定的动态库搜索路径；
4.    默认的动态库搜索路径/lib、/usr/lib。
所以指定目标库的时候需要使用 -rpath 参数传递路径给 gcc。
我们这里只是使用了标准 c 库，版本为 ldd 展示的 /lib/x86_64-linux-gnu/libc.so.6  GLIBC_2.2.5
 ldd helloworld
        linux-vdso.so.1 =>  (0x00007ffd493f3000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f5f12756000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f5f12b20000)

### 编译器优化
我们显示调用的 c 库函数是 printf，在 c 语言库中 stdio.h 中定义：
```
/* Write formatted output to stdout.

   This function is a possible cancellation point and therefore not
   marked with __THROW.  */
extern int printf (const char *__restrict __format, ...);
```
但是实际上，我们通过 nm 命令看到可执行文件中调用的 c 库的 puts， 通过汇编更能清晰的看到这个调用的详细情况：
 gcc -S helloworld.c            
 cat helloworld.s 
```
        .file   "helloworld.c"
        .section        .rodata
.LC0:
        .string "Hello World"
        .text
        .globl  main
        .type   main, @function
main:
.LFB0:
        .cfi_startproc
        pushq   %rbp
        .cfi_def_cfa_offset 16
        .cfi_offset 6, -16
        movq    %rsp, %rbp
        .cfi_def_cfa_register 6
        movl    $.LC0, %edi
        call    puts
        nop
        popq    %rbp
        .cfi_def_cfa 7, 8
        ret
        .cfi_endproc
.LFE0:
        .size   main, .-main
        .ident  "GCC: (Ubuntu 5.4.0-6ubuntu1~16.04.9) 5.4.0 20160609"
        .section        .note.GNU-stack,"",@progbits
```
当打印的全部是字符串，即没有需要转为字符串的操作的时候， gcc 会把 printf 优化成 puts。所以对于编译器的优化对程序员来说有时候是透明的。
我们需要仔细的检查编译器是否对我们的代码进行了优化。

## Hello World 打印原理

从上面的分析，我们知道，我们的 helloworld 程序主要是调用了 puts 函数进行打印，puts 在 glibc 中的实现如下：
```
/* Write the string in S and a newline to stdout.  */
int
puts (const char *s)
{
  return fputs (s, stdout) || putchar ('\n') == EOF ? EOF : 0;
}

```
该函数主要是调用 fputs 将字符串送到 stdout （标注输出），并送出一个换行符！换行符同样是送到 stdout ：
```
/* Write the character C on stdout.  */
int
putchar (int c)
{
  return __putc (c, stdout);
}

```

### stdout, stdin 和 stderr

那么 stdout 是什么，glibc 是如何通过 stdout 将我们的终端相连接的呢？
stdout 在 glibc 中是 FILE 类型的指针：

```
/* Standard streams.  */
extern FILE *stdin, *stdout, *stderr;
#ifdef __STRICT_ANSI__
/* ANSI says these are macros; satisfy pedants.  */
#define	stdin	stdin
#define	stdout	stdout
#define	stderr	stderr
#endif
```
这 3 个指针分别是对应 fd 号为 0,1,2 的 3 个 标准 fd 的封装：
```
/* Standard streams.  */
#define	READ		1, 0
#define	WRITE		0, 1
#define	BUFFERED	0
#define	UNBUFFERED	1
#define	stdstream(name, next, fd, readwrite, unbuffered)		      \
    {									      \
      _IOMAGIC,								      \
      NULL, NULL, NULL, NULL, 0,					      \
      (void *) fd,							      \
      { readwrite, /* ... */ },						      \
      { NULL, NULL, NULL, NULL, NULL },					      \
      { NULL, NULL },							      \
      -1, -1,								      \
      (next),								      \
      NULL, '\0', 0,							      \
      0, 0, unbuffered, 0, 0, 0, 0					      \
    }
static FILE stdstreams[3] =
  {
    stdstream (&stdstreams[0], &stdstreams[1], STDIN_FILENO, READ, BUFFERED),
    stdstream (&stdstreams[1], &stdstreams[2], STDOUT_FILENO, WRITE, BUFFERED),
    stdstream (&stdstreams[2], NULL, STDERR_FILENO, WRITE, UNBUFFERED),
  };
FILE *stdin = &stdstreams[0];
FILE *stdout = &stdstreams[1];
FILE *stderr = &stdstreams[2];
```

其中可以明确的知道：
1.    只有 stderr 是不缓冲的，stdin 和 stdout 都是缓冲的，那么输出到 stdout 的字符可能不会立即显示
2.    stdin 是只读的， stdout 和 stderr 是只能写的，其他的操作，比如读 stdout 是不可预知的。
3.    fd 是显示直接强制赋值的，就是说 0,1,2 应该是已经打开的描述符，否则会出现输入输出错误。

那么是在何时打开的标准描述符呢？

### stdio 与 tty

stdio 是与 tty 对应的，一个系统中可以有很多用户，或者一个用户打开了多个终端，但是 printf 等输出都是在当前终端上。
stdio 是与 tty 一一对应。从 glibc 的代码我们可以找到打开标准描述符 0,1,2 的位置：
login_tty.c
```
int
login_tty(fd)
	int fd;
{
	(void) setsid();
#ifdef TIOCSCTTY
	if (ioctl(fd, TIOCSCTTY, (char *)NULL) == -1)
		return (-1);
#else
	{
	  /* This might work.  */
	  char *fdname = ttyname (fd);
	  int newfd;
	  if (fdname)
	    {
	      if (fd != 0)
		(void) close (0);
	      if (fd != 1)
		(void) close (1);
	      if (fd != 2)
		(void) close (2);
	      newfd = open (fdname, O_RDWR);
	      (void) close (newfd);
	    }
	}
#endif
	(void) dup2(fd, 0);
	(void) dup2(fd, 1);
	(void) dup2(fd, 2);
	if (fd > 2)
		(void) close(fd);
	return (0);
}
```
每次登陆的时候，系统会将当前的 login 程序传入的 fb， dump 出来 3 份，分别的 fb 值就是 0,1,2 
因此， stdin、stdout、stderr 其实对应的是同一个文件，这个文件就是当前 login 使用的 tty 。

## 从内存到设备
我们的 helloworld 程序被 shell 加载到内存， “Hello World” 字符串也是在内存的位置，如何输出到 tty 设备呢？
我们 tty 设备是虚拟的设备，可能是 LCD 显示器，可能是串口，也可能是 LED 显示器。其中的对应和输出流，
那就是要牵涉到具体的设备驱动，那又是另一个领域才能讲清楚的了。大概的数据流就是：
1.    输出设备和 tty 是绑定的，输出到 tty 就会把数据传递给显示设备驱动程序
2.    设备驱动程序会把字符串数据最后通过 DMA 或者其他总线方式发给设备
3.    最终的设备会显示我们需要看到的字符串 “Hello World”
