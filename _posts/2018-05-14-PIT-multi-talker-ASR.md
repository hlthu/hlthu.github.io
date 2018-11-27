---
layout: post
feature-img: "assets/img/pexels/computer.jpeg"
img: "assets/img/pexels/computer.jpeg"
title: 基于PIT的单通道多说话人分离与识别
tags: [PIT,ASR]
---

最近两天再看文章，主要是看了好几篇有关单通道多说话人分离和识别 (Single-Channel Multi-Talker Speech Separation and Recognition) 的文章，而其中主要看的是大佬 Dong Yu 的 Permutation Invariant Training (PIT)。现在简单总结一下。

## 0. 背景

首先多说话人是指录音中含有多个说话人的语音，时域表达是 $$y_t=\sum_{s=1}^{S}x_t^s$$，其中 $$x_t^s$$ 是第 $$s$$ 个说话人的语音，$$S$$ 是总共说话人数。之前的工作多事是基于时频特征去做，也就是时域信号做短时傅里叶变换 (STFT) ，然后从混合信号的 STFT $$Y_t(f)$$ 中分理出需要估计各个说话人的 STFT $$X_t^s(f)$$。

PS：目前有 Columbia University 的研究者 Yi Luo等实现了基于时域的分离方法，而且目前的效果要好于基于频域的方法，请搜索 TasNet (Time-domain Audio Separation
Network)，之后有时间再介绍一下这几篇文章。

## 1. 2017 年 ICASSP，首先提出 PIT。

![paper](/assets/img/blog/PIT-1.png)

2018年ICASSP上，Dong Yu发表了一篇文章，讲的是将 PIT 用于语音分离。该文提出了一种结构用于估计各个说话人的 STFT，如下图为两个说话人的情况

![arch](/assets/img/blog/PIT-2.png)

损失函数为

![cost](/assets/img/blog/PIT-3.png)

从损失函数来看，似乎和普通的分离网络没什么区别，但是请注意，在图中有一个“Pairwise Scores”。这是什么意思呢，就是不考虑输出的两个端口的顺序，分别考虑第一个输出端口为第一个说话人和第二个说话人的情况，然后取使得 loss 最小的那个 assignment。这样的话，这种模式是不要对输出做任何假设，自然也是一种说话人无关的分离技术。

## 2. 2017 年 Trans 上进一步改进，提出 uPIT。

![paper2](/assets/img/blog/PIT-4.png)

主要是将损失函数变到句子级别 (而不是以前的帧级别，以前在每一帧都考虑这种 assignment 的交叉与否，现在放在整句级别，也就是全局要不都是第一个输出端口全是第一个人的，要么全是第二个人的)，这样还可以解决 speaker tracing 的问题。另外为了解决这种时序上的连续性，作者提出了将 LSTM 和 BLSTM 用于分离，取得了十分不错的效果。Loss 见下：

![cost-upit1](/assets/img/blog/PIT-5.png)

其中 $$B=T\times F \times S$$ 是对所有帧、频点和说话人的归一化系数，而 $$\hat{\mathbf{M}_s}$$ 则是估计到的第 $$s$$ 个人的 Mask，注意这个 Mask 是个 $$T\times F$$ 的矩阵，覆盖了所有帧和频点；而 $$\phi^*$$ 则是使得匹配的 loss 最小的那个 assignment：

![cost-upit2](/assets/img/blog/PIT-6.png)

## 3. 2017 年 Interspeech 上直接应用到识别上。

之前的工作都是在分离上进行的，一个直观的想法就是将两个输出换成识别系统中声学模型的输出，然后用交叉熵 (Cross Entropy, CE) 取代上面的最小均方误差 (Mean Square Error, MSE)。没错，Yu 老师就是这么想的，这篇论文是：

![pit-asr1](/assets/img/blog/PIT-7.png)

这时候把结构图换了一下：

![pit-asr2](/assets/img/blog/PIT-8.png)

Loss 函数也换成了基于 CE 的：

![pit-asr3](/assets/img/blog/PIT-9.png)

其中 $$\textbf{O}_t^s$$ 是第 $$s$$ 个人 $$t$$ 时刻的神经网络输出，而 $$l_t^{s^,}$$ 则是使得误差最小的 permutation 对应的标注。同样这个 loss 是整句算一个 loss，然后回传，因此对于一句话，每个时刻的 permutation 被强制约束为一致的。

## 4. 2018 年以来的一系列改进

至此，单通道多说话人识别的基础结构已经很清晰了，之后的工作都是在上面基础上做的小修小改，主要是提升性能。

### 4.1 Zhehuai Chen 的 Progressive Joint Modeling

![pit-asr4](/assets/img/blog/PIT-10.png)

这篇 Trans 文章的信息在于提出 Progressive 的训练方式，即先别训练帧级别的 PIT MSE，再训练句子级别的 PIT MSE；与此同时训练单通道单说话人的识别系统；最后再把两部分结合起来联合训练。并且，这篇文章也提出了利用干净语音进行 Transform Learning，此外还提出了一些序列化的方法。看这张图就好了。

![pit-asr5](/assets/img/blog/PIT-11.png)

### 4.2 Tian Tan 的 Transform Learning

![pit-asr6](/assets/img/blog/PIT-12.png)

其实 Yu 老师也想到了 Transform Learning，并发表在 ICASSP 2018 上。结构如下，很清晰易懂。

![pit-asr7](/assets/img/blog/PIT-13.png)

### 4.3 Xuankai Chang 的 Adaptation with Auxiliary Information

![pit-asr8](/assets/img/blog/PIT-14.png)

这也是 Yu 老师的作品，主要是在 PIT CE 系统中加入了说话人的特征 (pitch, i-vector)，还实现自适应和性能的提升，同时增加了一个 Task 用于预测当前混合的说话人的性别对信息 (男男，女女，男女)，结构如下。

![pit-asr9](/assets/img/blog/PIT-15.png)


### 4.4 Zhehuai Chen 的 SEQUENCE MODELING

这是 Zhehuai Chen 做的序列化工作，也是在 ICASSP 2018 上，笔者表示看不懂。

![pit-asr10](/assets/img/blog/PIT-16.png)

### 4.5 Yanmin Qian 的 Speech Com. 长文

![pit-asr11](/assets/img/blog/PIT-17.png)

这篇是个了解 PIT ASR 的期刊，但是文章新意已经不多，比较了四种结构。

### 4.6 Lianwu Chen 的 PIT-ASR + GAN

![pit-asr12](/assets/img/blog/PIT-18.png)

这篇文章将 GAN 和 PIT MSE 结合，Idea 很好，但是性能一般般。

### 4.7 Xuankai Chang 的 PIT-ASR + Attention

![pit-asr13](/assets/img/blog/PIT-19.png)

在 PIT CE ASR 基础上引入了 Attention 机制，取得了一定的效果。

## 5. PIT + End-to-End for ASR

最后还有一个很容易想到的点，将基于 CE 的识别换成基于 End-to-End 的识别。没错，笔者也想到了，但是没有来得及做出来，手还是慢了，这里也有几篇文章，主要是上交大的 Qian 和 JHU 的 Shinji 等的工作。不打算仔细介绍了，小伙伴们看论文吧，其中第一篇是 ACL 长文。


![pit-asr13](/assets/img/blog/PIT-20.png)

![pit-asr13](/assets/img/blog/PIT-21.png)

![pit-asr13](/assets/img/blog/PIT-22.png)
