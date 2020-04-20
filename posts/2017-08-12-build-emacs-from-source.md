---
title: 从源码编译安装Emacs
date: 2017-08-12 11:32:18
tags:
  - emacs
categories:
  - 工具使用
---

## 版本要求

最近想尝试一下酷眩的[spacemacs](http://spacemacs.org/)配置，但是它在Emacs 24.4或者更高的版本上才能展现所有的特性，而软件仓库中最新的版本为24.3，因此只能从源码编译安装了。

## 编译安装

首先在官方网站上下载源码，文件名为`emacs-VERSION.tar.gz`，我下载的VERSION为24.4，解压文件：

<!--more-->

```bash
tar -zxvf emacs-VERSION.tar.gz
```

然后进入到文件夹中编译、安装：

```bash
cd emacs-VERSION
./configure             # 提升缺失一些库时，需要手动安装
make
sudo make install       # 默认安装到/usr/local
```

此时应该可以通过在命令行中输入emacs来启动软件了。但是对于桌面版的Linux，还需要做如下配置才能够在菜单中搜索到Emacs、显示对应图标、并能通过点击图标启动Emacs。

## 额外配置

安装过程中会将`.desktop`文件放到`/usr/share/applications`中，图标（icon）文件放到`/usr/share/icon/hicolor`中。如果对应路径中找不到文件，需要手动操作一下（我安装时，desktop文件未拷贝到对应路径）：

```bash
sudo cp emacs-VERSION/etc/emacs.desktop /usr/share/applications
```

然后更新一下cache：

```
sudo gtk-update-icon-cache -t -f --include-image-data /usr/share/icons/hicolor
sudo update-desktop-database
```

这时，就可以在Linux桌面系统中通过菜单中的图标启动Emacs了。

## 参考

1. [Installing GNU](https://www.gnu.org/software/emacs/manual/html_node/efaq/Installing-Emacs.html)
2. [Emacs-25.2](http://www.linuxfromscratch.org/blfs/view/cvs/postlfs/emacs.html)
3. [Desktop entries](https://wiki.archlinux.org/index.php/Desktop_entries)
