---
title: 大小端以及转换
categories: [embeded]
description: big or little endian
keywords: endian
---

## 字节序

在各种计算机体系结构中，对于字节、字等的存储机制有所不同，因而引发了计算机通信领域中一个很重要的问题，即通信双方交流的信息单元（比特、字节、字、双字等等）应该以什么样的顺序进行传送。如果不达成一致的规则，通信双方将无法进行正确的编/译码从而导致通信失败。

目前在各种体系的计算机中，通常采用big-endian和little-endian两种字节存储机制描述在多字节数中各个字节的存储顺序[^1]。

## 字节序转换

```c
/* Swap bytes in 16 bit value.  */
#define __bswap_constant_16(x) \
     ((unsigned short int) ((((x) >> 8) & 0xff) | (((x) & 0xff) << 8)))

/* Swap bytes in 32 bit value.  */
#define __bswap_constant_32(x) \
     ((((x) & 0xff000000) >> 24) | (((x) & 0x00ff0000) >>  8) |               \
      (((x) & 0x0000ff00) <<  8) | (((x) & 0x000000ff) << 24))

```

[^1]: Endian 的由来 <http://www.eygle.com/digest/2007/01/whats_mean_endian.html>