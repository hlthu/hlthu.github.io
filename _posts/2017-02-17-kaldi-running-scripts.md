---
layout: post
title: kaldi 笔记：运行 timit 例程脚本(部分)
tags: [kaldi, timit]
---

本文将以 kaldi 中 timit 的例程来看整个 run.sh 脚本的执行过程。本文来自于[Running the example scripts (40 minutes)](http://kaldi-asr.org/doc/tutorial_running.html)

# 数据准备

请先进入 `kaldi\egs\timit\s5\` 这个目录。

## 运行环境

由于 kaldi 可以在本地运行，也可以在 Oracle GridEngine 上运行，因此，请修改 `cmd.sh`。

如果你是在本地运行，请输入

```
export train_cmd="run.pl --max-jobs-run 10"
export decode_cmd="run.pl --max-jobs-run 10"
export cuda_cmd="run.pl --max-jobs-run 2"
export mkgraph_cmd="run.pl --max-jobs-run 10"
```

其中 `max-jobs-run` 限定了最大的运行线程数，如果你的机器CPU不是特别多，请一定加上这个条件，防止内存被耗尽。

如果你是在 Oracle GridEngine 上运行，那么请修改成类似的内容

```
export train_cmd="queue.pl --mem 4G"
export decode_cmd="queue.pl --mem 4G"
export mkgraph_cmd="queue.pl --mem 8G"
export cuda_cmd="queue.pl --gpu 1"
```

## 准备数据

timit 下的 `run.sh` 前面 5 条命令都是和数据准备有关的。

```
# 数据路径
timit=/mnt/b/huanglu/timit

# 准备数据
local/timit_data_prep.sh $timit || exit 1

# 准备词典
local/timit_prepare_dict.sh

# 准备语言文件
utils/prepare_lang.sh --sil-prob 0.0 --position-dependent-phones false --num-sil-states 3 \
 data/local/dict "sil" data/local/lang_tmp data/lang

# 准备训练集、开发集合测试集
local/timit_format_data.sh
```

在执行完上面的脚本之后，进入 `data` 目录，会看到下面一些文件夹

| 文件夹          | 内容              |
| ------------ | --------------- |
| dev          | 开发集数据           |
| lang         | 包含语言文件          |
| lang_test_bg | 用于测试的语言文件       |
| local        | 包含了原始数据的信息，以及词典 |
| test         | 测试集             |
| train        | 训练集             |

### 词典

```
$ cd data/local/dict
$ head lexicon.txt
$ head nonsilence_phones.txt
$ head silence_phones.txt
```

| 文件                    | 内容            |
| --------------------- | ------------- |
| lexicon.txt           | 词典            |
| nonsilence_phones.txt | 包含哪些音素是静音的信息  |
| silence_phones.txt    | 包含哪些音素不是静音的信息 |


### 训练（测试、开发）数据

```
$ cd data/train
$ head text
$ head spk2gender
$ head spk2utt
$ head utt2spk
$ head wav.scp
```

| 文件         | 内容                  |
| ---------- | ------------------- |
| spk2gender | 包含说话人的性别信息          |
| spk2utt    | 包含说话人编号和说话人的语音编号的信息 |
| text       | 包含语音和语音编号之间的关系      |
| utt2spk    | 语音编号和说话人编号之间的关系     |
| wav.scp    | 包含了原始语音的路径信息等       |

### 语言文件

```
$ cd data/lang
$ head words.txt
$ head phones.txt
$ cd phones
$ head silence.csl
$ head nonsilence.csl
$ cd ..
$ ls -hl
```

其中 `words.txt` 和 `phones.txt` 是 OpenFst 格式的符号表，表示音素到整数的映射。

而 `*.csl` 文件则则包括了静音、非静音和音素的整数 id  列表。

文件 `L.fst` 是FST 格式的编译词典。为了查看其代表的信息，可以执行

```
fstprint --isymbols=data/lang/phones.txt --osymbols=data/lang/words.txt data/lang/L.fst | head
```

如果显示 `No command 'fstprint' found`，则可以执行

```
. ./path.sh
```

输出结果如下：

```
0	0	aa	aa
0	0	ae	ae
0	0	ah	ah
0	0	ao	ao
0	0	aw	aw
0	0	ax	ax
0	0	ay	ay
0	0	b	b
0	0	ch	ch
0	0	cl	cl
```

# 特征提取

有关特征提取的代码主要是下面几行：

```shell
mfccdir=mfcc
for x in train dev test; do 
  steps/make_mfcc.sh --cmd "$train_cmd" --nj $feats_nj data/$x exp/make_mfcc/$x $mfccdir
  steps/compute_cmvn_stats.sh data/$x exp/make_mfcc/$x $mfccdir
done
```

执行这段代码之前需要执行

``` shell
. ./cmd.sh
feats_nj=4
train_nj=4
decode_nj=1
```

执行前面的代码，可以生成了两个文件夹：`mfcc` 和 `exp/make_mfcc`，其中 `mfcc` 里主要保存了提取的特征，而 `exp/make_mfcc` 里保存了日志，即 `.log` 文件。

在 `steps/make_mfcc.sh` 里用到的最主要的命令就是 `compute-mfcc-feats` 和 `copy-feats`，其在 `src` 里编译好的。

`mfcc` 目录里主要是 `.ark` 和 `.scp` 文件，其中 `.scp` 文件里的内容是语音段和特征对应，而真正的特征保存在 `.ark` 文件里。用下面的命令可以看清楚

```
$ head raw_mfcc_train.1.scp
faem0_si1392 /home/huanglu/kaldi/egs/timit/s5/mfcc/raw_mfcc_train.1.ark:13
faem0_si2022 /home/huanglu/kaldi/egs/timit/s5/mfcc/raw_mfcc_train.1.ark:6313
faem0_si762 /home/huanglu/kaldi/egs/timit/s5/mfcc/raw_mfcc_train.1.ark:9349
faem0_sx132 /home/huanglu/kaldi/egs/timit/s5/mfcc/raw_mfcc_train.1.ark:13048
faem0_sx222 /home/huanglu/kaldi/egs/timit/s5/mfcc/raw_mfcc_train.1.ark:16916
faem0_sx312 /home/huanglu/kaldi/egs/timit/s5/mfcc/raw_mfcc_train.1.ark:20537
faem0_sx402 /home/huanglu/kaldi/egs/timit/s5/mfcc/raw_mfcc_train.1.ark:25549
faem0_sx42 /home/huanglu/kaldi/egs/timit/s5/mfcc/raw_mfcc_train.1.ark:29767
fajw0_si1263 /home/huanglu/kaldi/egs/timit/s5/mfcc/raw_mfcc_train.1.ark:32804
fajw0_si1893 /home/huanglu/kaldi/egs/timit/s5/mfcc/raw_mfcc_train.1.ark:39403
```

`scripts` 和 `archives` 的底层是 `Table` 的概念。一个 `Table` 基本上是由唯一字符串（例如话语标识符）索引的项目（例如特征文件）的有序集合。`.scp` 格式是一个纯文本格式，每一行有一个 `key`，然后一个“扩展文件名”，告诉 Kaldi 在哪里可以找到数据。`archive` 格式可以是文本或二进制。 格式是：`key`（例如 utterance id），然后一个空格，然后是对象数据。

除了 MFCC 特征以外，文件夹 `mfcc` 里还有 CMVN 特征。

关于 `scripts` 和 `archives` 两种格式，有以下论述

>1. A string that specifies how to read a Table (archive or script) is called an rspecifier; for example "ark:gunzip -c my/dir/foo.ark.gzl".
>2. A string that specifies how to write a Table (archive or script) is called a wspecifier; for example "ark,t:foo.ark".
>3. Archives can be concatenated together and still be valid archives (there is no "central index" in them).
>4. The code can read both scripts and archives either sequentially or via random access. The user-level code only knows whether it's iterating or doing lookup; it doesn't know whether it's accessing a script or an archive.
>5. Kaldi doesn't attempt to represent the object type in the archive; you have to know the object type in advance.
>6. Archives and script files can't contain mixtures of types.
>7. Reading archives via random access can be memory-inefficient as the code may have to cache the objects in memory.
>8. For efficient random access to an archive, you can write out a corresponding script file using the "ark,scp" writing mechanism (e.g., used in writing the mfcc features to disk). You would then access it via the scp file.
>9. Another way to avoid the code having to cache a bunch of stuff in memory when doing random access on archives is to tell the code that the archive is sorted and will be called in sorted order (e.g. "ark,s,cs:-").
>10. Types that read and write archives are templated on a Holder type, which is a type that "knows how" to read and write the object in question.

至于如何通过管道使用 `scripts` 和 `archives` 两种格式的文件，可以参考下面的命令

```
$ head -1 mfcc/raw_mfcc_train.1.scp | copy-feats scp:- ark:- | copy-feats ark:- ark,t:- | head -n 2
# 示例输出
copy-feats ark:- ark,t:-
copy-feats scp:- ark:-
LOG (copy-feats:main():copy-feats.cc:120) Copied 1 feature matrices.
faem0_si1392  [
  38.3508 -31.04888 -10.46318 -9.166654 -11.21832 -1.998698 18.50948 3.447642 6.247809 -5.24096 -4.613473 -14.82091 -12.65494
  38.3508 -29.88511 -2.339327 -2.430964 -3.292726 2.997352 5.760813 4.516098 14.78676 7.313382 -6.224341 -16.88169 -8.839561
```

# 单音素训练和解码

## 单音素训练

下面的命令可以训练单音素模型(monophone model)

```
steps/train_mono.sh  --nj "$train_nj" --cmd "$train_cmd" data/train data/lang exp/mono
```

训练过程的输出大致如下

```
steps/train_mono.sh --nj 4 --cmd run.pl --max-jobs-run 10 data/train data/lang exp/mono
steps/train_mono.sh: Initializing monophone system.
steps/train_mono.sh: Compiling training graphs
steps/train_mono.sh: Aligning data equally (pass 0)
steps/train_mono.sh: Pass 1
steps/train_mono.sh: Aligning data
························
steps/train_mono.sh: Aligning data
steps/train_mono.sh: Pass 36
steps/train_mono.sh: Pass 37
steps/train_mono.sh: Pass 38
steps/train_mono.sh: Aligning data
steps/train_mono.sh: Pass 39
steps/diagnostic/analyze_alignments.sh --cmd run.pl --max-jobs-run 10 data/lang exp/mono
steps/diagnostic/analyze_alignments.sh: see stats in exp/mono/log/analyze_alignments.log
2 warnings in exp/mono/log/align.*.*.log
exp/mono: nj=4 align prob=-99.15 over 3.12h [retry=0.0%, fail=0.0%] states=144 gauss=985
```

之后会在 `exp` 文件夹下产生一个 `mono` 的目录，里面以 `.mdl` 结尾的就保存了模型的参数。使用下面的命令可以查看模型的内容。

```
$ gmm-copy --binary=false exp/mono/0.mdl - | less
# 示例输出
<TransitionModel>
<Topology>
<TopologyEntry>
<ForPhones>
2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48
</ForPhones>
<State> 0 <PdfClass> 0 <Transition> 0 0.75 <Transition> 1 0.25 </State>
<State> 1 <PdfClass> 1 <Transition> 1 0.75 <Transition> 2 0.25 </State>
<State> 2 <PdfClass> 2 <Transition> 2 0.75 <Transition> 3 0.25 </State>
<State> 3 </State>
</TopologyEntry>
<TopologyEntry>
<ForPhones>
1
</ForPhones>
<State> 0 <PdfClass> 0 <Transition> 0 0.5 <Transition> 1 0.5 </State>
<State> 1 <PdfClass> 1 <Transition> 1 0.5 <Transition> 2 0.5 </State>
<State> 2 <PdfClass> 2 <Transition> 2 0.75 <Transition> 3 0.25 </State>
<State> 3 </State>
</TopologyEntry>
</Topology>
<Triples> 144
1 0 0
1 1 1
1 2 2
2 0 3
2 1 4
2 2 5
3 0 6
3 1 7
3 2 8
4 0 9
············
```

有关上下文的信息可以查看 `tree` 文件

```
$ copy-tree --binary=false exp/mono/tree - | less
# 示例输出
copy-tree --binary=false exp/mono/tree -
LOG (copy-tree:main():copy-tree.cc:55) Copied tree
ContextDependency 1 0 ToPdf TE 0 49 ( NULL TE -1 3 ( CE 0 CE 1 CE 2 )
TE -1 3 ( CE 3 CE 4 CE 5 )
TE -1 3 ( CE 6 CE 7 CE 8 )
TE -1 3 ( CE 9 CE 10 CE 11 )
TE -1 3 ( CE 12 CE 13 CE 14 )
TE -1 3 ( CE 15 CE 16 CE 17 )
TE -1 3 ( CE 18 CE 19 CE 20 )
TE -1 3 ( CE 21 CE 22 CE 23 )
TE -1 3 ( CE 24 CE 25 CE 26 )
TE -1 3 ( CE 27 CE 28 CE 29 )
TE -1 3 ( CE 30 CE 31 CE 32 )
TE -1 3 ( CE 33 CE 34 CE 35 )
···················
EndContextDependency
```

下面查看训练数据 Veterbi 对齐，`ali.*.gz` 的每一行是对应于每一个训练文件的。


```
$ copy-int-vector "ark:gunzip -c exp/mono/ali.1.gz|" ark,t:- | head -n 2
# 示例输出
faem0_si1392 2 4 3 3 3 3 3 3 6 5 5 5 5 5 38 37 37 40 42 218 217 217 217 217 217 217 217 217 217 217 217 220 219 222 221 221 221 248 247 247 247 247 247 250 252 176 175 178 177 177 177 177 177 180 122 121 121 121 121 121 121 121 124 123 123 126 125 26 28 27 30 29 29 212 211 214 216 215 146 148 150 260 262 264 263 263 278 277 277 277 280 282 281 281 281 14 13 13 13 13 16 15 18 17 176 175 178 180 62 64 66 206 208 210 242 241 244 246 170 169 169 169 169 172 171 174 173 173 173 173 173 38 37 37 37 40 42 41 218 217 217 217 217 217 217 217 217 217 220 219 222 221 146 145 148 150 62 64 66 56 55 55 55 55 55 55 58 60 59 59 248 250 249 252 251 251 251 251 251 116 115 115 115 115 118 117 117 120 119 224 223 223 223 223 223 223 226 225 225 228 98 97 97 100 99 102 101 266 265 265 265 265 268 267 267 267 270 269 269 86 88 90 110 109 112 111 114 122 121 121 121 121 121 121 121 121 121 124 123 126 125 8 7 7 7 7 7 7 7 7 10 9 9 12 212 214 216 176 175 175 178 177 180 134 133 133 133 136 135 138 137 86 88 90 89 278 277 277 277 280 282 281 281 38 37 40 42 62 64 66 65 206 208 207 207 207 207 207 210 209 14 13 13 16 15 15 18 17 17 17 62 61 64 66 164 166 168 167 167 152 154 156 188 190 189 189 192 191 191 224 223 223 223 223 223 223 223 223 223 226 225 225 228 227 86 85 85 85 85 85 85 85 85 85 85 88 87 90 260 259 262 264 263 68 70 72 71 71 71 2 4 3 3 6 5 5 5 5 5 5 14 13 13 16 15 15 15 15 15 18 17 17 182 181 184 186 260 262 264 68 70 72 122 121 121 121 121 121 121 124 123 123 126 152 151 151 151 151 154 153 153 153 156 155 155 155 155 155 155 170 169 169 169 169 169 169 172 171 174 173 260 259 262 261 264 263 263 263 218 217 217 217 217 217 217 217 217 217 220 219 222 221 221 2 1 4 3 3 3 3 3 3 3 6
faem0_si2022 2 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 4 3 3 3 3 3 6 5 5 5 266 265 265 265 265 265 265 268 267 270 269 269 20 22 24 80 82 81 84 83 32 31 34 33 33 33 33 36 35 35 35 35 35 35 122 121 121 121 121 121 121 121 121 124 123 126 125 146 145 148 147 150 62 64 63 66 65 65 65 65 65 65 68 70 72 71 242 241 244 246 245 224 223 223 223 223 223 223 223 223 223 223 223 226 225 225 228 152 154 153 156 260 262 264 263 68 70 72 71 212 211 211 211 214 213 213 216 215 215 215 215 44 43 46 45 45 45 48 47 47 254 256 258 122 121 121 121 121 121 121 121 121 121 121 124 123 126 125 125 26 25 25 28 27 27 27 27 27 30 29 29 29 2 1 1 1 1 4 3 3 3 3 3 3 6 5
```

可以使用 `show-transitions` 命令来查看有关 transition-id 的信息。

```
$ show-transitions data/lang/phones.txt exp/mono/0.mdl
# 示例输出
Transition-state 1: phone = sil hmm-state = 0 pdf = 0
 Transition-id = 1 p = 0.5 [self-loop]
 Transition-id = 2 p = 0.5 [0 -> 1]
Transition-state 2: phone = sil hmm-state = 1 pdf = 1
 Transition-id = 3 p = 0.5 [self-loop]
 Transition-id = 4 p = 0.5 [1 -> 2]
···········
```

## 构建解码图

在解码之前，我们需要构建解码图。输入下面的命令

```
$ utils/mkgraph.sh --mono data/lang_test_bg exp/mono exp/mono/graph
# 示例输出
tree-info exp/mono/tree
tree-info exp/mono/tree
fsttablecompose data/lang_test_bg/L_disambig.fst data/lang_test_bg/G.fst
fstdeterminizestar --use-log=true
fstpushspecial
fstminimizeencoded
fstisstochastic data/lang_test_bg/tmp/LG.fst
-0.00841335 -0.00928529
fstcomposecontext --context-size=1 --central-position=0 --read-disambig-syms=data/lang_test_bg/phones/disambig.int --write-disambig-syms=data/lang_test_bg/tmp/disambig_ilabels_1_0.int data/lang_test_bg/tmp/ilabels_1_0
fstisstochastic data/lang_test_bg/tmp/CLG_1_0.fst
-0.00841335 -0.00928515
make-h-transducer --disambig-syms-out=exp/mono/graph/disambig_tid.int --transition-scale=1.0 data/lang_test_bg/tmp/ilabels_1_0 exp/mono/tree exp/mono/final.mdl
fstminimizeencoded
fstdeterminizestar --use-log=true
fsttablecompose exp/mono/graph/Ha.fst data/lang_test_bg/tmp/CLG_1_0.fst
fstrmsymbols exp/mono/graph/disambig_tid.int
fstrmepslocal
fstisstochastic exp/mono/graph/HCLGa.fst
0.000381767 -0.00951546
add-self-loops --self-loop-scale=0.1 --reorder=true exp/mono/final.mdl
```

## 解码

接下来我们就可以解码了，分别针对开发集和测试集解码。

```
steps/decode.sh --nj "$decode_nj" --cmd "$decode_cmd" \
 exp/mono/graph data/dev exp/mono/decode_dev
 
steps/decode.sh --nj "$decode_nj" --cmd "$decode_cmd" \
 exp/mono/graph data/test exp/mono/decode_test
```

解码的日志会保存在 `exp/mono/decode_dev/log` 和 `exp/mono/decode_test/log` 里。下面是开发集的解码输出。

```
steps/decode.sh --nj 1 --cmd run.pl --max-jobs-run 10 exp/mono/graph data/dev exp/mono/decode_dev
decode.sh: feature type is delta
steps/diagnostic/analyze_lats.sh --cmd run.pl --max-jobs-run 10 exp/mono/graph exp/mono/decode_dev
steps/diagnostic/analyze_lats.sh: see stats in exp/mono/decode_dev/log/analyze_alignments.log
Overall, lattice depth (10,50,90-percentile)=(5,26,120) and mean=61.6
steps/diagnostic/analyze_lats.sh: see stats in exp/mono/decode_dev/log/analyze_lattice_depth_stats.log
```

紧接着你可以完成其他训练和解码。

# 查看结果

输入下面的命令来查看结果

```
#!/bin/bash
for x in exp/{mono,tri,sgmm,dnn,combine}*/decode*; do [ -d $x ] && echo $x | grep "${1:-.*}" >/dev/null && grep WER $x/wer_* 2>/dev/null | utils/best_wer.sh; done
for x in exp/{mono,tri,sgmm,dnn,combine}*/decode*; do [ -d $x ] && echo $x | grep "${1:-.*}" >/dev/null && grep Sum $x/score_*/*.sys 2>/dev/null | utils/best_wer.sh; done
```

我的解码结果如下：

```
%WER 31.8 | 400 15057 | 71.7 19.4 8.9 3.5 31.8 100.0 | -0.496 | exp/mono/decode_dev/score_5/ctm_39phn.filt.sys
%WER 32.2 | 192 7215 | 70.5 19.4 10.0 2.8 32.2 100.0 | -0.211 | exp/mono/decode_test/score_6/ctm_39phn.filt.sys
```

开发集和测试集的词错率(WER)分别是 31.8% 和 32.2%。

# 参考

* [Running the example scripts (40 minutes)](http://kaldi-asr.org/doc/tutorial_running.html)