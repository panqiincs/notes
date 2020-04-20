---
title: 在Emacs中预览Markdown
date: 2015-07-31 11:21:59
tags:
  - emacs
  - markdown
categories:
  - 工具使用
---

## 需求

[上一篇文章][lastpost]提到，在Emacs中可以预览Markdown文本得到的网页，但前提是，你的系统中要有将Markdown文本转化为HTML文本的Markdown命令。如果没有任何配置，直接使用预览的快捷键，Emacs将提示`/bin/bash: markdown: command not found`。我找到了两种办法来解决这个问题：

## markdown_py

Python有将Markdown转化为HTML的命令，只不过它的名字不叫Markdown。我通过如下的命令找到了它：

<!--more-->

```bash
$ su -c 'updatedb'
Password:
$ locate markdown | grep bin
/usr/bin/markdown_py
/usr/bin/markdown_py-2.7
```

找一个Markdown文本，尝试一下markdown_py命令的效果：

```bash
$ markdown_py filename.markdown > result
```

打开result文件，发现它就是一个HTML文本，所以这就是符合我们要求的命令。查看一下文件的信息：

```bash
$ ll /usr/bin | grep markdown
lrwxrwxrwx.   1 root  root          15 Mar 22 18:06 markdown_py -> markdown_py-2.7
-rwxr-xr-x.   1 root  root        1013 Feb  9  2012 markdown_py-2.7
```

原来`markdown_py`就是`markdown_py-2.7`的一个软链接。所以，我们再建立一个名为markdown的软链接指向`markdown_py-2.7`就可以解决问题了：

```bash
$ cd /usr/bin
$ su -c 'ln -s markdown_py-2.7 markdown'
Password: 
$ 
```

大功告成，敲下`C-c C-c p`看看效果，网页是乱码的。如果Markdown的内容是中文，那么转换出来的HTML在浏览器中打开就无法自动识别编码。这个时候，我们可以借助Markdown对HTML标记的支持，在Markdown文件开头中加入编码信息：

```bash
$ sed -i '1i\<meta http-equiv="content-type" content="text/html; charset=UTF-8">' *.markdown
```

我最开始使用的就是这个方法，虽然奏效了，但是在每个Markdown文本中都要加上这行代码，真是大煞风景！有没有更优雅的解决办法呢？于是，我找到了另外一个软件pandoc。


## pandoc

pandoc支持多种输出格式，我们现在只关注HTML格式。pandoc将Markdown转化为HTML并将结果定向到标准输出：

```bash
$ pandoc -f markdown -t html filename.markdown
```

为了防止中文乱码，要加上一个`--ascii`选项。pandoc还支持语法高亮和[LaTeX 数学公式][latexsupport]。Octopress默认使用pygments支持语法高亮的，因此加上`--highlight-stype pygments` 选项。加上`--mathjax`选项以支持LaTeX数学公式。得到的最终形式是：

```bash
$ pandoc -f markdown -t html -s --mathjax --highlight-style pygments filename.markdown
```

但是，如何让Emacs调用这个带很多选项的命令呢？这时候就要用到shell的知识了。在shell中，`$1`代表第一个命令行参数，我们将上述语句中最后的文件名替换成`$1`，然后把它封装成一个名为markdown的文本：

```bash
pandoc -f markdown -t html -s --mathjax --highlight-style pygments $1
```

将它移动到二进制文件目录中，并增加可执行权限:

```bash
$ su -c 'mv markdown /usr/bin'
Password:
$ su -c 'chmod a+x /usr/bin/markdown'
Password:
```

现在，执行`markdown filename.markdown`语句的效果与执行

```bash
$ pandoc -f markdown -t html -s --mathjax --highlight-style pygments filename.markdown
```

的效果是一样的。也就是说，我们的目的达到了。现在试一试预览效果，perfect！


## 参考

1. [如何在Linux下使用Markdown进行文档工作][linuxmarkdown]
2. [使用Pandoc转换markdown等文本文档][pandoc]
3. [Emacs配置Pandoc和Markdown][emacspandocmarkdown]
注：本文中提到在Emacs的Option中配置Markdown Command在我的机器上行不通


[lastpost]: https://panqiincs.me/2015/06/15/edit-markdown-with-emacs/
[latexsupport]: http://yanping.me/cn/blog/2012/03/10/octopress-with-latex/ 
[linuxmarkdown]: http://www.ituring.com.cn/article/10044
[pandoc]: http://higrid.net/c-art-pandoc.htm
[emacspandocmarkdown]: http://blog.sina.com.cn/s/blog_61f013b80101eo21.html
