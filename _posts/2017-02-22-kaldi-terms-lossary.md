---
layout: post
title: kaldi 笔记：部分术语词汇表
tags: [kaldi, ASR]
---

本文主要参考的是 [kaldi-asr.org](http://kaldi-asr.org/doc/glossary.html)，主要介绍我们在使用 kaldi 的时候可能想要了解的术语词汇表。当然这里介绍的只是一部分例子，相信不久 [kaldi-asr.org](http://kaldi-asr.org/doc/glossary.html) 就会增加新的内容。

# acoustic scale

可以翻译为声学尺度吧，是在解码时使用的。在 C++ 程序中经常被写成 `–acoustic-scale`，而在其他程序中可能被写作 `–acwt`。这是对声学对数概率的描述，是一个在 HMM-GMM 和 HMM-DNN 系统中通常使用的系统，以说明帧与帧之间的相关性。其值都成被设置为 0.1，这意味着声学对数概率比语言模型的对数概率具有更低的权重。在打分(score)的脚本中，经常会看到语言模型的权重被搜索的范围(range)。这可以解释为声学尺度的逆(inverse，这里理解成1/n，也就是倒数可能比较合适)，因为这个 range 的定义是语言模型对数概率和声学对数概率的比值。

# (forced) alignment

是由一句话的Viterbi(最佳路径)对齐取得的 HMM 状态的序列的表示，在 kaldi 中，alignment 和 transition-id 的序列是一样的，大多数时候，alignment 是由一句话的参考标注通过对其得到的，也通常被称为强制对齐(forced alignment)。网格(lattices)还包含了网格中每个词序列的 transition-id 的序列信息。

# lattice

表示一段话的可选择的转录，当然还有对齐和成本信息(alignment and cost information)。

# transition-id

一个基于 1 的索引，其编码了概率密度函数的id(pdf-id，也就是聚类后的上下文相关的 HMM 状态)、音素表达，和一些关于我们在 HMM 中是否采用 self-loop 和 前向转录(forward transition)的信息。在网格、解码图和对齐的时候出现。

# transition model

对象 TransitionModel 编码了 HMMs 的转录状态，以及其他一些重要的各类集成图。这个对象经常是在模型文件的开始处写入。

# G.fst

在脚本中，语法 FST G.fst 存在于 `data/lang` 目录内，其代表了语言模型中有限的状态转换格式。对于大部分读者，这意味着在 arcs 上输入和输出符号是相同的，但是对于使用 backoff 的统计语言模型，backoff arcs 只在输入有"disambiguation symbol" #0。







