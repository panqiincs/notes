---
title: Linux下安装TL-WN823N无线网卡驱动
date: 2016-08-28 15:55:20
tags: 
  - 网卡驱动
  - linux
categories:
  - 工具使用
---

请注意，本文的安装方法针对的3.x版本的内核，对4.x版本内核可能不起作用，4.x内核下的安装步骤请参考下文中给出的Github页面。

`TP-LINK TL-WN823N`无线网卡在我的Linux（内核版本3.18）系统下无法正常使用，因为没有对应的驱动。官方网站上也很难找到可用的驱动，需要手动从源代码编译安装。

首先查看网卡信息，运行`lsusb`得到如下信息：

<!--more-->

```bash
Bus 003 Device 002: ID 0bda:818b Realtek Semiconductor Corp.
```

Google一下发现网卡使用的是`Realtek RTL8192EU`芯片：

```bash
Realtek RTL8192EU ID 0BDA:818B
```

在[Github](https://github.com/Mange/rtl8192eu-linux-driver)上找到了驱动的源代码，下载到本地：

```bash
git clone https://github.com/Mange/rtl8192eu-linux-driver
```

编译之前要安装`kernel-devel`组件，否则会编译报错。在openSUSE运行下面的命令安装`kernel-devel`：

```bash
sudo zypper install kernel-devel
```

最后编译安装驱动程序：

```bash
cd rtl8192eu-linux-driver/
make
sudo make install
```

升级内核之后，网卡驱动可能会失效，需要在新的内核下重新编译一次才能使用。
