---
title: Linux下烧写树莓派系统
date: 2017-05-23 19:00:35
tags: 
  - 树莓派
  - linux
categories:
  - 工具使用
---

## 需求

在Windows系统下可以用Win32diskimager软件将镜像文件写入SD卡，但是如果SD卡中已经有了一个系统，Windows就无法识别其中的ext4分区，再次写入比较麻烦。在Linux下，这个就不是问题了。

## 识别设备号

首先识别SD卡的设备号。拔出SD卡，运行`df -h`命令显示已经挂载的设备，插入SD卡，再运行一次`df -h`，比较两次结果的差异，便得到SD卡的设备号，例如`/dev/mmcblk0`、`/dev/sdd`。如果之前有烧写过系统，可以看到诸如`/dev/mmcblk0p0`和`/dev/mmcblk0p1`的内容。

<!--more-->

## 写入镜像文件

运行`umount /dev/mmcblk0`卸载该设备，否则之后的写入操作无法进行。解压镜像包，得到后缀名为`.img`的文件，比如：`2015-02-16-raspbian-wheezy.img`，然后使用`dd`命令将镜像写入SD卡中：

```bash
sudo dd bs=4M if=2015-02-16-raspbian-wheezy.img of=/dev/mmcblk0
```

需要注意输出`of=`不要填`mmcblk0p1`之类的内容。

## 查看实时进度

写入需要一段较长的时间，可以运行`pkill -USR1 -n -x dd`来查看实时的进度。

## 参考

1. [Installing operating system images on Linux](https://www.raspberrypi.org/documentation/installation/installing-images/linux.md)
