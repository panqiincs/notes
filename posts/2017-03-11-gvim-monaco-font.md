---
title: 配置GVim使用Monaco字体
date: 2017-03-11 11:06:03
tags:
  - vim
  - 字体
categories:
  - 工具使用
---

## 效果

Mac系统下的Monaco字体非常适合编程。GVim（Ubuntu系统）使用Monaco字体的效果如下：

<!--more-->

![字体显示效果](https://images-1254088545.cos.ap-shanghai.myqcloud.com/blog/gvim_font_monaco.png)

## 安装字体

首先在[Github](https://github.com/cstrap/monaco-font)上下载Monaco字体。解压后有三个安装脚本，分别对应不同的系统：`install-font-centos.sh`，`install-font-centos.sh`，`install-font-centos.sh`，运行相应的脚本即可完成字体的安装。

## 配置字体

Vim下配置字体需要设置`guifont`变量的值。在Vim的命令模式下输入`:set guifont=*`后回车，弹出如下的对话框：

![字体选择对话框](https://images-1254088545.cos.ap-shanghai.myqcloud.com/blog/gvim_font_select.png)

选择Monaco字体即可。然后输入`:set guifont?`后回车，GVim会返回`guifont=Manoca 11`，因此，配置语句如下：

```bash
if has("gui_running")
    set guifont=Monaco\ 11
endif
```

注意空格之前要加上一个转义字符`\`。将上面的配置语句加到`.vimrc`文件中即可看到效果。

## 参考

1. [Ask Ubuntu](https://askubuntu.com/questions/333409/how-to-install-the-monaco-font)
2. [Stack Overflow](https://stackoverflow.com/questions/16882696/settings-default-font-in-gvim)
