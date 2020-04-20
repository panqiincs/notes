---
title: 套接字地址结构体
date: 2015-10-30 13:51:21
tags:
  - 结构体
categories:
  - 程序设计
---

大多数的套接字函数都需要一个指向套接字地址结构的指针作为参数。在使用中经常遇到两个结构体`sockaddr_in`和`sockaddr`，它们的区别是什么呢？

## POSIX

`sockaddr_in`是IPv4套接字地址结构体，它的定义在`<netinet/in.h>`头文件中，在POSIX中，它的定义如下：

<!--more-->

```c
struct in_addr {
    in_addr_t s_addr;                 /* 32-bit IPv4 address  */
                                      /* network byte ordered */
};

struct sockaddr_in {
    uint8_t          sin_len;         /* length of structure (16) */
    sa_family_t      sin_family;      /* AF_INET */
    in_port_t        sin_port;        /* 16-bit TCP or UDP port number */
    struct in_addr   sin_addr;        /* 32-bit IPv4 address */
                                      /* network byte ordered */
    char             sin_zero[8];     /* unused */
}
```

`sockaddr`是通用套接字地址结构体，套接字API函数的参数所使用的均是`sockaddr`结构体，这使得任何套接字函数都能够处理来自所支持的任何协议族的套接字地址结构体。例如，`bind`函数的原型如下：

```c
int bind(int, struct sockaddr *, socklen_t);
```

在调用函数时，必须要将指向特定于协议的套接字地址结构体的指针进行强制转换，变成指向某个通用套接字地址结构体的指针：

```c
struct sockaddr_in serv;       /* IPv4 socket address structure */

/* fill in serv{} */

bind(sockfd, (struct sockaddr *) &serv, sizeof(serv));
```

在POSIX中，`sockaddr`是这样定义的：

```c
struct sockaddr {
    unit8_t       sa_len;
    sa_family_t   sa_family;          /* address family: AF_xxx value */
    char          sa_data[14];        /* protocol-specific address */
};
```

可以看到，在POSIX中，`sockaddr_in`和`sockaddr`两个结构体的前两个成员是相同的，因此强制类型转换是有意义的。

## Linux

在Linux中，这两个结构体中的定义如下：

```c
struct in_addr {
    in_addr_t s_addr;
};

struct sockaddr_in {
    __SOCKADDR_COMMON (sin_);
    in_port_t sin_port;             /* Port number */
    struct in_addr sin_addr;        /* Internet address */
    /* Pad to size of 'struct sockaddr'. */
    unsigned char sin_zero[sizeof (struct sockaddr) -
               __SOCKADDR_COMMON_SIZE - 
               sizeof (in_port_t) - 
               sizeof (struct in_addr)];
};

struct sockaddr {
    sa_family_t   sa_family;
    char          sa_data[14];
}
```

`sockaddr_in`中有两个很奇怪的成员，一个是第一行的`__SOCKADDR_COMMON (sin_)`，另外一个是`sin_zero`数组括号中的`__SOCKADDR_COMMON_SIZE`，查询相关头文件找到它们的定义：

```c
#define __SOCKADDR_COMMON(sa_prefix) \
  sa_family_t sa_prefix##family

#define __SOCKADDR_COMMON_SIZE (sizeof (unsigned short int))
```

表达式`__SOCKADDR_COMMON (sin_)`是一个宏，展开后得到`sa_family_t sin_family`，因此，在Linux定义中，这两个结构体中并没有`sa_len`成员，而且第一个成员均为`sa_family`。`__SOCKADDR_COMMON_SIZE`是`sa_family_t`类型的大小，最后一个成员`sin_zero[]`的大小是通过一个计算式得来，使得两个结构具有相同的长度。

## 说明

实际上，POSIX规范只需要这个结构体的三个字段`sin_family`、`sin_addr`和`sin_port`，定义其他额外的字段是可以接受的。几乎所有的实现都加入了`sin_zero`字段，而且所有的套接字地址结构大小至少都为16字节。

有了ANSI C标准之后，可以使用`void *`通用指针类型。然而套接字函数是在ANSI C之前定义的，因此加入通用的套接字地址结构体`sockaddr`来解决类型转换问题。从应用程序开发人员的观点来看，这些通用套接字地址唯一的用途就是，对特定于协议的套接字地址结构体的指针执行类型强制转换。

## 参考

1. [Unix网络编程（卷1）](http://book.douban.com/subject/4859464/)
2. [stackoverflow](http://stackoverflow.com/questions/3829435/query-regarding-syntax-used-in-a-header-file-for-socket-programming)
