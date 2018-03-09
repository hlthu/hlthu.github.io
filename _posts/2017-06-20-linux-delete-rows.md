---
layout: post
title: Linux Shell 删除文件中含特定字符串的行
tags: [linux]
---
 
* 删除`a.txt`中含`abc`的行，但不改变`a.txt`文件本身，操作之后的结果在终端显示

```
sed -e '/abc/d'  a.txt
```
 
* 删除`a.txt`中含`abc`的行，将操作之后的结果保存到`a.log`

```
sed -e '/abc/d'  a.txt  > a.log
```
* 删除含字符串`abc`或`efg`的行，将结果保存到`a.log`
 
```
sed '/abc/d;/efg/d' a.txt > a.log
```

其中，`abc`也可以用正则表达式来代替。