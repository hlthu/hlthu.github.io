---
layout: post
title: 深度学习与语音识别：语音识别的基本结构
date: 2017-02-08 18:19:30 +08:00
category: "ASR"
tags: [ASR]
---


最近开始阅读《解析深度学习：语音识别实践》，将会把文中的一些总结放到这里和大家分享。今天的这篇文章主要介绍语音识别的基本结构。

下图中展示的是语音识别系统的典型结构，语音识别系统主要由图中的四部分组成：信号处理和特征提取、声学模型（AM）、语言模型（LM）和解码搜索部分。

![](https://raw.githubusercontent.com/hlthu/hlthu.github.io/master/images/posts/asr/2017-02-08/1.jpg)

信号处理和特征提取部分以音频信号为输入，通过消除噪声和信道失真对语音进行增强，将信号从时域转化到频域，并为后面的声学模型提取合适的有代表性的特征向量。声学模型将声学和发音学（phonetics）的知识进行整合，以特征提取部分生成的特征为输入，并为可变长特征序列生成声学模型分数。语言模型估计通过从训练语料（通常是文本形式）学习词之间的相互关系，来估计假设词序列的可能性，又叫语言模型分数。如果了解领域或任务相关的先验知识，语言模型分数通常可以估计得更准确。解码搜索对给定的特征向量序列和若干假设词序列计算声学模型分数和语言模型分数，将总体输出分数最高的词序列当作识别结果。在本书中，我们主要关注声学模型。

关于声学模型，有两个主要问题，分别是特征向量序列的可变长和音频信号的丰富变化性。可变长特征向量序列的问题在学术上通常由如动态时间规整（dynamictime warping，DTW）方法和将在第3章描述的隐马尔可夫模型（HMM）方法来解决。音频信号的丰富变化性（variable）是由说话人的各种复杂的特性（如性别、健康状况或紧张程度）交织，或是说话风格与速度、环境噪声、周围人声（side talk）、信道扭曲（channel distortion）（如麦克风间的差异）、方言差异、非母语口音（non-nativeaccent）引起的。一个成功的语音识别系统必须能够应付所有这类声音的变化因素。

在过去，最流行的语音识别系统通常使用梅尔倒谱系数（mel-frequency cepstral coefficient，MFCC）或者“相对频谱变换–感知线性预测”（perceptual linear prediction，PLP）作为特征向量，使用混合高斯模型–隐马尔可夫模型（Gaussianmixture model-HMM，GMM-HMM）作为声学模型。在20 世纪90年代的时候，最大似然准则（maximum likelihood，ML）被用来训练这些GMM-HMM声学模型。到了21世纪，序列鉴别性训练算法（sequence discriminative training algorithm）如最小分类错误（minimum classification error，MCE）和最小音素错误（minimum phone error，MPE）等准则被提了出来，并进一步提高了语音识别的准确率。

在近些年中，分层鉴别性模型（discriminative hierarchical model）如深度神经网络（deep neural network，DNN）依靠不断增长的计算力、大规模数据集的出现和人们对模型本身更好的理解，变得可行起来。它们显著地减小了错误率。举例来说，上下文相关的深度神经网络–隐马尔可夫模型（context-dependent DNN-HMM，CD-DNN-HMM）与传统的使用序列鉴别准则（sequence discriminative criteria）训练的GMM-HMM系统相比，在Switchboard对话任务上错误率降低了三分之一。

在本书中，我们将介绍这些分层鉴别性模型的最新研究进展，包括深度神经网络（DNN）、卷积神经网络（convolutional neural network，CNN）和循环神经网络（recurrent neural network，RNN）。我们将讨论这些模型的理论基础和使得系统能够正常工作的实践技巧。由于我们对自己所做的工作比较熟悉，本书主要着眼于我们自己的工作，当然，在需要的时候也会涉及其他研究者的相关研究。

