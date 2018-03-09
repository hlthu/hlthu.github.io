---
layout: post
title: linux 杀死某一用户所有进程
tags: [linux]
---



在 linux 系统管理中，我们有时候需要 kill 掉某个用户的所有进程，初学者一般先查询出用户的所有 pid ，然后一条条 kill 掉，或者写好一个脚本，实际上方法都有现成的，这边有4种方法，我们以 kill 用户 `huanglu` 为例.

## 1. pkill方式

```
pkill -u huanglu
```

## 2.killall方式

```
killall -u huanglu
```

## 3.ps方式

ps 列出 huanglu 的 pid，然后依次 kill 掉，比较繁琐。

```
ps -ef | grep huanglu | awk '{ print $2 }' | sudo xargs kill -9
```

## 4.pgrep方式

pgrep -u 参数查出用户的所有 pid，然后依次 kill

```
pgrep -u huanglu | sudo xargs kill -9
```

