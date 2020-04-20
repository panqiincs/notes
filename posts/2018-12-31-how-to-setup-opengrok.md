---
title: OpenGrok安装和配置
date: 2018-12-31 20:12:29
tags:
  - opengrok
categories:
  - 工具使用
---

OpenGrok是跨平台、开源免费的源码阅读工具。OpenGrok生成源码的索引，Apache Tomcat提供Web服务，用户即可在浏览器中阅读代码，使用体验可以去看看[这里](http://bxr.su/FreeBSD/)。Linux下好用的源码阅读工具极少，OpenGrok可以一试。

本文是一篇没有技术含量的笔记，之所以贴出来是因为我找到的其它教程都失效了。

<!--more-->

部署OpenGrok之前需安装如下软件：

  + JDK 1.8或以上
  + universal-ctags
  + Tomcat 8，请参考[这里](https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-8-on-ubuntu-16-04)，**非常重要**的步骤，请认真执行每一步，一定要执行配置systemd那步

在OpenGrok的[Github项目主页](https://github.com/oracle/opengrok)下载二进制安装包，当前最新的稳定版本为1.1。解压安装包到`/opt/opengrok`目录下即可。OpenGrok安装目录下的`source.war`文件很重要，Tomcat检测到该配置文件后部署应用。如果遵从默认配置，不需修改，直接拷贝到Tomcat的安装目录下：

```bash
sudo cp /opt/opengrok/lib/source.war /opt/tomcat/webapps
```

源码目录和索引目录默认存放在`/var/opengrok`下，建立如下文件夹：

```bash
mkdir -p /var/opengrok/data
mkdir -p /var/opengrok/src
mkdir -p /var/opengrok/etc
```

将源码文件夹都放在`src`目录下，运行下面的命令生成索引：

```bash
java -Djava.util.logging.config.file=/var/opengrok/logging.properties \
     -jar /opt/opengrok/lib/opengrok.jar \
     -c /usr/bin/universal-ctags \
     -s /var/opengrok/src -d /var/opengrok/data -H -P -S -G \
     -W /var/opengrok/etc/configuration.xml -U http://localhost:8080 
```

可以看到`data`目录下有文件生成，即为索引文件。在浏览器中打开`http://localhost:8080/source`，应该就能阅读源码了。当`src`目录发生变化时，需重新运行上面的命令更新索引。
