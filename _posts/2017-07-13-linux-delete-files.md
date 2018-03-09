---
layout: post
title: 关于Linux误删文件
tags: [linux]
---

经常在朋友圈看到有朋友各种文件误删或者忘记保存，以前总是不屑，终于今天我把自己的一个30+G的工作成果给删了，主要是用Kaldi做识别的一些脚本、特征和模型。十分悔恨，尝试了各种方法，没有恢复。没办法，浪子回头。之后自己想了想，想把每次删除的文件先放到一个文件夹下，然后自己定期手动删除，毕竟误删文件在删除后五分钟内肯定是可以发现，至少我是这样的。



这里主要就是将`rm`命令用`mv`表示，这样就不会真正删除文件。



首先在`home`目录下创建隐藏目录`.trash`：

```shell
mkdir .trash
```

接着创建一个移动文件至上述目录的脚本`.trash.sh`:

```shell
#!/bin/bash                                                                                                                
datestr=$(date +%Y_%m_%d_%H_%M_%S)
mkdir -p ~/.trash/$datestr
mv $@ ~/.trash/$datestr/
```

在该目录里文件的命名方式是按时间来的，方便之后找回。

最后就是制作`rm`命令的替身，编辑`.bashrc`文件，加入：

```shell
alias rm=~/.trash.sh
```

最后就是使上述更改生效：

```shell
source .bashrc
```
