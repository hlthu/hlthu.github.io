---
layout: post
title: 178小时开源中文语料库
tags: [ASR]
---

最近一直做实验，选择的都是TEDLIUM、AMI等开源的英文语料库，以及Switchboard语料库，而在中文上目前开源的也只有我们清华王东老师THCHS-30，之前也在上面做过实验，但是数据集只有30小时，还是不怎么痛快。今天看微信，偶然发现【北京希尔贝壳科技有限公司】开源了一个178小时的中文语料库。


# 简介

178小时中文普通话开源语音数据（AISHELL-ASR0009-OS1）包含400位来自中国不同口音区域的发言人。录音文本包含财经、科技、体育、娱乐等领域。录制过程在安静室内环境中，使用高保真麦克风和录音机进行。此数据库经过专业语音校对人员转写标注，并通过严格质量检验，文本正确率在95%以上。

178-hour Chinese Mandarin Open Source Speech Corpus (AISHELL-ASR0009-OS1), including 400 speakers from different accent area in China. The recordings contain financial, technical, sports, entertainment, and other fields. The recording is done in quiet indoor environment using high fidelity microphone and recorder. The manual transcription accuracy is above 95%, through professional speech annotation and strict quality inspection.


[官网](http://www.aishelltech.com/kysjcp).

# 下载

[OpenSLR](http://www.openslr.org/33/).


# 引用

```
@inproceedings{aishell_2017,
  title={AIShell-1: An Open-Source Mandarin Speech Corpus and A Speech Recognition Baseline},
  author={Hui Bu, Jiayu Du, Xingyu Na, Bengu Wu, Hao Zheng},
  booktitle={Oriental COCOSDA 2017},
  pages={Submitted},
  year={2017}
}
```