---
title: TCP连接的建立和终止
date: 2015-08-10 18:14:24
tags:
  - tcp连接
  - tcp
categories:
  - 程序设计
---

## 理解协议

Richard Stevens先生在UNP2e的前言中写道：

> I have found when teaching network programming that about 80% of all network programming problems have nothing to do with network programming, per se. That is, the problems are not with the API functions such as accept and select, but the problems arise from a lack of understanding of the underlying network protocols. For example, I have found that once a student understands TCP’s three-way handshake and four-packet connection termination, many network programming problems are immediately understood.

<!--more-->

作者阐述了这么一个事实：大部分网络编程问题与API函数的使用无关，问题多是因对网络协议理解不够而导致的。所以，学好TCP/IP协议非常重要，首选之作当属Stevens先生的[TCP/IP Illustrated](https://book.douban.com/subject/1741925/)。我觉得[The TCP/IP Guide](https://book.douban.com/subject/2129076/)对新手更友好，讲解细致，插图精美，这本书可以[在线阅读](http://www.tcpipguide.com/free/index.htm)，也有免费的pdf下载。

## 连接建立和终止的过程

TCP连接的建立和终止过程就不用我蹩脚地重述一遍，The TCP/IP Guide的插图基本可以将全过程描述清楚。TCP连接建立的过程又称之为“三路握手”（Three-Way Handshake）（图片来源：[TCP Connection Establishment](http://www.tcpipguide.com/free/t_TCPConnectionEstablishmentProcessTheThreeWayHandsh-3.htm)）：

![TCP连接建立](https://images-1254088545.cos.ap-shanghai.myqcloud.com/blog/tcp_open.png)

TCP连接终止的过程又称之为“四路握手”（Four-Way Handshake）（图片来源：[TCP Connection Termination](http://www.tcpipguide.com/free/t_TCPConnectionTermination-2.htm)）：

![TCP连接终止](https://images-1254088545.cos.ap-shanghai.myqcloud.com/blog/tcp_close.png)

## 为什么是三路和四路

TCP必须保证通道双向都通畅，所以两端都要向对方发送数据并能收到对方的回复。握手的过程也要交流信息，双方都要知晓彼此的初始序列号（Initial Sequence Number）以及其它参数。这样来看，两端一共要传送四个分组。建立连接时，被动打开的一端可以将自己的SYN和回复给对方的ACK一起发送过去，所以只需要“三路握手”。终止连接时，被动关闭的一端收到FIN之后，要等待应用程序完成任务后才能关闭连接，所以只能先回复一个ACK，等待一段时间后再发送自己的FIN给对方，不能一起发送过去，所以需要“四路握手”。

## TIME_WAIT状态

主动关闭的一端发送ACK给对端后进入TIME_WAIT状态，然后才转到CLOSED状态。TIME_WAIT状态有两个存在的理由：

+ 可靠地终止TCP的全双工连接。保证有足够的时间使得被动关闭的一端能收到ACK，如果丢失了就重传
+ 在当前连接终止和接下来的连接之间保留缓冲时间，保证老的重复分组在网络中消逝，不然来自不同连接的分组有可能混杂

TIME_WAIT的持续时间是最长分节生命期（maximum segment lifetime, MSL）的两倍。MSL是任何IP数据报能够在网络中存活的最长时间。如果某个TCP连接关闭后，过一段时间在相同的IP地址和端口之间建立另一个连接，因为这两个连接的IP地址和端口号相同，后一个连接称之为前一个连接的化身。TCP必须防止来自某个连接的老的重复分组在该连接已终止后再现，以免这些分组被误解为属于新的化身。TIME_WAIT状态持续2MSL足以让某个方向上的分组最多存活MSL秒后即被丢弃，另一个方向上的应答最多存活MSL秒也被丢弃。通过这个规则，可以保证老的重复分组在网络中消逝。

## 参考

1. [谈一谈网络编程学习经验](https://github.com/downloads/chenshuo/documents/LearningNetworkProgramming.pdf)
2. [The TCP/IP Guide](http://www.tcpipguide.com/free/index.htm)
3. [Unix网络编程](https://book.douban.com/subject/1500149/)
