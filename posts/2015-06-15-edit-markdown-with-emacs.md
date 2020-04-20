---
title: 用Emacs编辑Markdown
date: 2015-06-15 08:52:45
tags:
  - emacs
  - markdown
categories:
  - 工具使用
---

## 安装

Emacs默认并没有Markdown模式。网络的智慧是无穷的，果然，我找到了[Emacs Markdown Mode](http://jblevins.org/projects/markdown-mode/)。接下来便是本文的主题，我只是一个蹩脚的翻译。将[markdown-mode.el](http://jblevins.org/git/markdown-mode.git/plain/markdown-mode.el)放到Emacs的`load-path`中，并在`.emacs`配置文件中添加如下内容，以支持后缀名为`.markdown`和`.md`的文件:

<!--more-->

```el
(autoload 'markdown-mode "markdown-mode"
    "Major mode for editing Markdown files" t)
(add-to-list 'auto-mode-alist '("\\.markdown\\'". markdown-mode))
(add-to-list 'auto-mode-alist '("\\.md\\'". markdown-mode))
```

至此，大功告成，效果如下：

![编辑markdown](https://images-1254088545.cos.ap-shanghai.myqcloud.com/blog/markdown_edit.png)

## 使用

现在用Emacs写博文真是一种享受。会Emacs的人会发现，下面提到的快捷键大多和Emacs的基础快捷键相关，我只列出常用的：

### 在Emacs中预览效果

使用该功能的前提是系统中有将Markdown文本转化为HTML的**Markdown命令**。配置方法请看[这里](https://panqiincs.me/2015/07/31/preview-markdown-with-emacs/)。命令如下：

1. `C-c C-c m` 转化为HTML，在另一个buffer中预览HTML文件，个人觉得没太大意义
2. `C-c C-c p` 转化为HTML，在浏览器中预览
3. `C-c C-c e` 转化为HTML，保存为文件
4. `C-c C-c v` 转化为HTML，保存为文件，并在浏览器中预览

### 插入超链接

插入超链接的命令是**C-c C-a**。`C-c C-a l`插入`[]()`形式的链接，`C-c C-a L`插入`[LinkText][Label]`形式的链接。在这种形式下，如果光标附近有文字或是`Active Region`，会自动被选择当作`LinkText`。后一种形式会提示你在`Minibuffer`中输入`LinkText`、`LinkLabel`和可选的`LinkTitle`。

### 插入图片

插入图片用命令`C-c C-i i`和`C-c C-i I`，两者的区别和超链接的类似。

### 插入样式

插入样式的命令是**C-a C-s**：

1. `C-c C-s e` *插入斜体字*（e表示emphasis）
2. `C-c C-s s` **插入粗体字**（s表示strong）
3. `C-c C-s c` 插入代码框

### 插入标题

插入标题的命令是：**C-c C-t**

1. 我最常用的`C-c C-t n`，数字n的范围从1到6，表示各级标题。比如`C-c C-t 3`得到`### Heading ###`
2. `C-c C-t h`根据前面的标题自动选择标题级别。`C-c C-t H`类似，区别在于它得到的是带下划线的标题

### 快捷键 

其它一些快捷键：

1. `C-c -` 插入水平线
2. `C-c C-o` 如果该点是一个链接(hyperlink)，就会在浏览器中打开它的URL，如果该点是**维基百科**链接(wikilink)，就会在另一个buffer中打开
3. `C-c C-j` 如果该点所在的位置是一个hyperlink，按下此快捷键就会在`Link Text`和`Link URL`之间跳转。同样也适用于脚注(footnote)等其它类似目标
4. `C-c C--`和`C-c C-=` 升级（promotion）和降级（demotion）。例如，在`### ###`附近按下`C-c C--`会使它变成`## ##`，按下`C-c C-=`会使它变成`#### ####`。前者让heading升级，后者让heading降级
5. `C-c C-k` 将该点的目标kill掉，并将其内容送到kill ring中，适用于以下目标：inline code, headings, horizonal rules, links, images, email address等
6. `C-c C-n`, `C-c C-p`, `C-c C-f`, `C-c C-b`, `C-c C-u` 在heading之间移动
7. `M-{`, `M-}`, `C-M-a`, `C-M-e`, `C-M-h` 快速跳转，和Emacs基础快捷键操作一样
