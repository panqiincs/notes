---
title: 大小端字节序
date: 2015-05-19 17:36:10
tags:
  - 字节序
  - 联合体
categories:
  - 程序设计
---

大多数计算机使用字节（byte）作为最小的可寻址的内存单位。对于跨越多字节的对象，我们需要知道该对象的地址以及内存中是如何排列这些字节的。在几乎所有的机器上，多字节对象存储在连续的字节序列上，对象的地址即为所使用字节中最小的地址。

那么，字节存储的顺序就成了一个问题。有的机器在内存中按照从最低有效字节（Least Significant Byte, LSB）到最高有效字节(Most Significant Byte, MSB)的顺序存储对象，而另一些机器则按照从最高有效字节到最低有效字节的顺序存储，分别称之为小端（little endian）存储和大端（big endian）存储：

<!--more-->

+ 小端序：最低有效字节在低内存地址
+ 大端序：最低有效字节在高内存地址

我们修改[整数的表示](https://panqiincs.me/2015/03/21/integer-representations/)一文中提到的`show_bytes`函数，同时打印内存地址和数据：

```c
#include <stdio.h>

typedef unsigned char *byte_pointer;

void show_bytes(byte_pointer start, int len)
{
    // 从内存低地址到高地址打印每个字节的内容
    for (int i = 0; i < len; i++) {
        printf("%p:", &(start[i]));
        printf(" 0x%.2x", start[i]);
        printf("\n");
    }
}

int main()
{
    int x = 0x12345678;  // int型大小为4字节
    show_bytes((byte_pointer)&x, sizeof(x));    
}
```

在我的机器上，程序运行的结果是：

```bash
0x7fff809bc2fc: 0x78
0x7fff809bc2fd: 0x56
0x7fff809bc2fe: 0x34
0x7fff809bc2ff: 0x12
```

整数`0x12345678`从最低有效字节到最高有效字节依次是：`0x78`，`0x56`，`0x34`和`0x12`。根据运行结果，多字节对象是按照从低地址到高地址的顺序存储的，所以我的机器使用的是小端存储方式。还可以用系统命令`lscpu`确认如下：

```bash
$ lscpu
Architecture:        x86_64
CPU op-mode(s):      32-bit, 64-bit
Byte Order:          Little Endian
...
Model name:          Intel(R) Core(TM) i5-4430 CPU @ 3.00GHz
...
```

通过上述的`show_bytes`函数即可以判断机器的字节序，不过面试官一般将下面的方法作为标准答案，要用到C语言的联合体（union）：

```c
#include <stdio.h>

int main()
{
    union {
        short s;     // 2字节
        char  c[2];  
    } un;
    
    un.s = 0x0102;
    if (un.c[0] == 1 && un.c[1] == 2)
        printf("big-endian\n");
    else if (un.c[0] == 2 && un.c[1] == 1)
        printf("little-endian\n");
    else 
        printf("unknown\n");
}
```

使用大端字节序的有SPARC、PowerPC、MIPS等架构的CPU。Intel x86系列的CPU都使用小端序，因此小端序更常见。无论是在大端序还是小端序的机器上运行程序，结果都是一样的。

但是，在不同字节序的机器之间通过网络传播二进制数据会带来问题，数据里的字节会反序。网络协议使用大端字节序来传送多字节对象，因此，大端序也称之为**网络字节序**（network byte order）。某个给定系统所用的字节序称之为**主机字节序**（host byte order）。套接字地址结构中的某些字段就必须按照网络字节序进行维护，操作系统提供了函数进行主机字节序和网络字节序之间的相互转换：

```c
#include <netinet/in.h>

uint16_t htons(uint16_t hostshort);  // host to net, short
uint32_t htonl(uint32_t hostlong);   // host to net, long
uint16_t ntohs(uint16_t netshort);   // net to host, short
uint32_t ntohl(uint32_t netlong);    // net to host, long
```

在那些与网络协议使用字节序（大端）相同的系统中，这四个函数通常被定义为空宏。
