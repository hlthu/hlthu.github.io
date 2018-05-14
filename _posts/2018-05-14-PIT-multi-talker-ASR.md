---
layout: post
feature-img: "assets/img/pexels/computer.jpeg"
img: "assets/img/pexels/computer.jpeg"
title: 基于PIT的多说话人识别
tags: [PIT,ASR]
---

最近两天再看文章，主要是看了好几篇有关多说话人的文章，而其中主要看的是大佬 Dong Yu 的Permutation Invariant Training(PIT)。现在简单总结一下。

## 1、2017年ICASSP，首先提出PIT。

![paper](/assets/img/blog/PIT-1.png)

2018年ICASSP上，Dong Yu发表了一篇文章，讲的是将PIT用于语音分离。首先多说话人是指录音中含有多个说话人的语音，时域表达是$$y_t=\sum_{s=1}^{S}x_t^s$$
但是一般语音分离需要估计各个说话人的频谱$$|X_t^s(f)|$$，该文提出了一种结构用于估计该值，如下图为两个说话人的情况

![arch](/assets/img/blog/PIT-2.png)

损失函数为

![cost](/assets/img/blog/PIT-3.png)

## 2、2017年Trans上进一步改进

主要是将损失函数变到句子级别（而不是以前的帧级别），这样可以解决speaker tracing的问题。

（未完待续，最近比较忙，没写完。。。）

