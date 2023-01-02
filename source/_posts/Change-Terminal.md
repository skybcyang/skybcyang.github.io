---
title: Ubuntu18.04终端更换为深度终端
date: 2018-12-29 18:49:54
tags:
- deepin
- linux
---
之前从Deepin到Ubuntu18多多少少有点不适应，特别是终端，所以心血来潮把Ubuntu的终端换成深度终端。
<!-- more -->
### 深度终端安装
加入官方源，安装就完事了

    sudo add-apt-repository ppa:noobslab/deepin-sc
    sudo apt-get update
    sudo apt-get install deepin-terminal


### 改成默认终端

首先 安装`dconf-tools`

    apt-get install dconf-tools

打开`dconf`

![](/img/Change-Terminal/change_terminal_1.png)

搜索`exec`

![](/img/Change-Terminal/change_terminal_2.png)

关闭默认配置，在`Custom value`输入`deepin-terminal`

![](/img/Change-Terminal/change_terminal_3.png)

然后就可以`Ctrl+Alt+T`了

### 最后
祝使用深度终端愉快！