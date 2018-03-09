---
layout: post
title: kaldi 笔记：数据准备
tags: [kaldi, asr]
---

本文主要参考的是 [kaldi-asr.org](http://kaldi-asr.org/doc/data_prep.html)，主要介绍我们在使用 kaldi 的时候可能用到的数据处理的脚本以及相关文件的信息。

# 简介

在运行完 kaldi 提供的例子之后，也许你想用自己的数据来建立一个系统，本节主要介绍如何准备自己的数据。请确保你使用的是例程脚本是最新的脚本。在本文中，你可以参考这些例子中有关数据准备的脚本。在每一个例子的根目录有一个 `run.sh`，该文件一般在开头会有若干行和数据准备有关的操作。比如在 RM 的例子中有：

```
local/rm_data_prep.sh /export/corpora5/LDC/LDC93S3A/rm_comp || exit 1;
utils/prepare_lang.sh data/local/dict '!SIL' data/local/lang data/lang || exit 1;
local/rm_prepare_grammar.sh || exit 1;
```

而在 TIMIT 的例子中有下面几句：

```
timit=/home/huanglu/timit 
local/timit_data_prep.sh $timit || exit 1
local/timit_prepare_dict.sh
utils/prepare_lang.sh --sil-prob 0.0 --position-dependent-phones false \
    --num-sil-states 3 data/local/dict "sil" data/local/lang_tmp data/lang
local/timit_format_data.sh
```

这些数据准备的命令成功运行后，会产生两类数据，一类是关于数据的（或者说声学的），比如 `data/train`；另一类是关于语言的，比如 `data/lang`。数据部分和你的具体录音相关，而语言部分则是和录音的语言（中午、英文等）相关的，比如字典、音素集等。如果你想利用一个现存的语言模型来解码，那么你只需要处理数据部分就可以了。


# 数据准备——数据部分

作为一个例子，你可以查看 `data/train` 目录下的文件，此外例如 `data/eval2000` 和 `data/test` 或者 `data/dev` 都是可以的，如果有的话。这里我们查看的是 `egs/swbd/s5` 目录下的 SwitchBoard 的例子。

```
$ ls data/train
cmvn.scp  feats.scp  reco2file_and_channel  segments  spk2utt  text  utt2spk  wav.scp
```

并不是所有文件都是同等重要的。在初步开始的的时候，一般是没有分段(segmentation)信息的，也就是一段话对应一个录音文件。只需要你自己手动创建的文件包括 `utt2spk`、`text` 和 `wav.scp`，可能还会有 `segments` 和 `reco2file_and_channel`，另外剩余的都可以使用标准的脚本自动创建。


我们会逐一阐述这个目录下的文件，首先从需要你自己创建的文件开始。

## 需要手动创建的文件

文件 `text` 包含了每个话语（一段语音）的转录（标注），比如：

```
$ head -3 data/train/text
sw02001-A_000098-001156 hi um yeah i'd like to talk about how you dress for work and and um what do you normally what type of outfit do you normally have to wear
sw02001-A_001980-002131 um-hum
sw02001-A_002736-002893 and is
```

每行的第 1 个元素是 `utterance-id`，即话语的编号，它可以是任意字符串，但是不能重复；如果你的设置中有说话人的信息，你应该把说话人的编号（speaker-id）作为话语编号的前缀。这对于文件内的排序是非常重要的。每一行的剩下部分是每个话语的转录。你并不需要确保这个文件李所有的词都在你的词汇表（vocabulary，称为词典可能好些）里，不在词汇表里的词会被映射到 `data/lang/oov.txt` 文件里的特定词。



它需要是这样的情况，当你排序 `utt2spk` 和 `spk2utt` 文件，顺序必须一致，例如，从 `utt2spk` 文件中提取的 `speaker-id` 列表与字符串排序顺序相同。一个更简单的方法就是把说话人的编号（speaker-id）作为话语编号的前缀。在这个特定的例子中，我们使用下划线 `_` 来连接 `utterance-id` 的“speaker”和“utterance”部分。但是一般来说，使用连接号（`-`）可能更安全，因为它的 ASCII 值更小。因为如果说话人 ID 在长度上变化， 还是使用下划线 `_` 来连接的话，当使用标准 C 语言排序的时候，speaker-id 和它们对应的话语的顺序可能是不一样的，这将导致一个崩溃。


另一个重要的文件是 `wav.scp`，在 SWBD 的例子里

```
$ head -3 data/train/wav.scp
sw02001-A /mnt/b/huanglu/kaldi/egs/swbd/s5c/../../../tools/sph2pipe_v2.5/sph2pipe -f wav -p -c 1 /mnt/b/huanglu/.swbd/SWBD1R2/cd1/swb1/sw02001.sph |
sw02001-B /mnt/b/huanglu/kaldi/egs/swbd/s5c/../../../tools/sph2pipe_v2.5/sph2pipe -f wav -p -c 2 /mnt/b/huanglu/.swbd/SWBD1R2/cd1/swb1/sw02001.sph |
sw02005-A /mnt/b/huanglu/kaldi/egs/swbd/s5c/../../../tools/sph2pipe_v2.5/sph2pipe -f wav -p -c 1 /mnt/b/huanglu/.swbd/SWBD1R2/cd1/swb1/sw02005.sph |
```

该文件的格式是

```
<recording-id> <extended-filename>
```

其中 `<extended-filename>` 可以是一个实际的文件名，也可以是是一个提取 wav 格式文件的命令（本例中正是）。文件名后面的管道符号指明他将被解释为管道。我们之后会解释 `<recording-id>`，但是首先我们必须指出，如果不存在 `segments` 文件，那么 `wav.scp` 文件每行的第一个就是 `utterance-id`。`wav.scp` 文件里的录音文件必须是单通道的(单声道)，如果底层 wav 有多个通道，必须要有一个 `sox` 命令来提取一个特定的通道。 

在 SWBD 里，有 `segments` 文件，下面我们将讨论这个文件

```
$ head -3 data/train/segments
sw02001-A_000098-001156 sw02001-A 0.98 11.56
sw02001-A_001980-002131 sw02001-A 19.8 21.31
sw02001-A_002736-002893 sw02001-A 27.36 28.93
```

`segments` 文件的格式是

```
<utterance-id> <recording-id> <segment-begin> <segment-end>
```

其中 `<segment-begin>` 和 `<segment-end>` 是以秒为单位计量的，这些指定到录音的时间偏移。`<recording-id>` 和 `wav.scp` 文件里的一样，同样也是一个任意的字符串。


文件 `reco2file_and_channel` 仅在使用 NIST 的“sclite”工具进行评分（测量错误率）时使用：

```
$ head -3 data/train/reco2file_and_channel
sw02001-A sw02001 A
sw02001-B sw02001 B
sw02005-A sw02005 A
```

该文件的格式是

```
<recording-id> <filename> <recording-side (A or B)>
```

文件名通常是 `.sph` 文件的名称，没有后缀，但通常它可以是你在 `stm` 文件中具有的任意标识符。`recording-side` 是一个关于有两个通道的电话对话的概念，如果没有两个通道，使用 `A` 应该是安全的。如果你没有 `stm` 文件，或者你对这个不甚了解，那么你就不需要文件 `reco2file_and_channel`

最后一个需要手动创建的文件是 `utt2spk`，该文件描述了每一个话语的说话人 id。

```
$ head -3 data/train/utt2spk
sw02001-A_000098-001156 2001-A
sw02001-A_001980-002131 2001-A
sw02001-A_002736-002893 2001-A
```

文件的格式是

```
<utterance-id> <speaker-id>
```

请注意 `speaker-id` 并不需要和精确的真是说话人呢的姓名对应，这只是一个合理的标识。在本例中，我们假设电话两边的每个说话人只有一个人。这可能不完全正确，因为有时候一个人可能会让另一个人来听电话，或者一个人可能在多个电话中说话。但是我们这样的假设对于我们的目标已经足够好了。**如果你么有关于所有说话人身份的信息，你可以简单地把 `speaker-id` 和 `utterance-id` 等同**，从而这个格式可能变成了：

```
<utterance-id> <utterance-id>
```

我们把上面一句加粗，因为我们遇到过只创建一个全局的 `speaker-id` 的人，千万不要那么做。这是一个非常愚蠢的做法，因为这样会导致在训练时倒谱均值归一化（epstral mean normalization）无效（因为它是全局应用的）。而且当你使用 `utils/split_data_dir.sh` 来将你的数据分成若干部分时会产生问题。


在其他的离子的数据准备过程中可能会产生另一个文件，但是它只是在 kaldi 系统中偶尔使用，在 rm 的例子中有这样一个文件 `spk2gender`。在我的上一篇文章[构建一个简单的英文数字串识别系统](/2017/02/22/kaldi-numbers-asr/)也有，其文件的内容是

```
1 m
2 m
3 m
4 m
5 m
6 f
7 f
8 f
```

该文件把说话人的 ID 映射为说话人的性别（Male or Female）。


上面的所有文件都应该排序好，否则的话，在运行脚本的时候可能会出错，在 [The Table concept](http://kaldi-asr.org/doc/io.html#io_sec_tables)一文中我们阐述了原因。它与 IO 的框架有关，而终极原因是排序可以使一些相当于在一些不支持 `fseek()` 的流上的随机查找的命令可用，比如管道命令。

很多 kaldi 的程序都从其他 kaldi 命令读取很多管道，读取不同类型的对象，也在一些不同的输入上做一些大致相当于合并排序（merge-sort）的事情。当然，merge-sort 需要输入时有序的。请注意，当你排序的时候，请把 shell 变量 `LC_ALL` 定义为 `C`，比如在终端直接输入：

```
export LC_ALL=C
```

如果你不做的话，那么就会得到一个不同于用 C++ 排序的结果，这会导致 kaldi 崩溃。我们已经提醒你了！

如果你的数据集包括了一个来自于 NIST 的测试集的话，有 `stm` 文件和 `glm` 文件的话，那么你就可以测量 WER，你需要把这些文件放进数据的目录，用 `stm` 和 `glm` 命名。请注意，我们把打分的脚本写在 `local/score.sh` 里。这意味着它是特定的设置。并不是所有例子里的打分脚本会识别出 `stm` 和 `glm` 文件。打分脚本使用这些文件的一个例子是 SWBD，即 `egs/swbd/s5/local/score_sclite.sh`，其被更高层的打分脚本 `egs/swbd/s5/local/score.sh` 调用，如果它检测到有 `stm` 和 `glm` 文件的话。


## 不需要手动创建的文件

数据目录下另外一部是可以利用你提供的文件自动产生的文件。比如你可以创建 `spk2utt`，仅仅使用下面的命令（存在于 `egs/rm/s5/local/rm_data_prep.sh`）：

```
utils/utt2spk_to_spk2utt.pl data/train/utt2spk > data/train/spk2utt
```

这当然是肯能的，因为这两个文件保存着同样的信息，都是说话人编号和话语编号的对应关系。文件 `spk2utt` 的格式为

```
<speaker-id> <utterance-id1> <utterance-id2> ....
```

现在我们来看文件 `feats.scp`

```
$ head -3 data/train/feats.scp
sw02001-A_000098-001156 /home/dpovey/kaldi-trunk/egs/swbd/s5/mfcc/raw_mfcc_train.1.ark:24
sw02001-A_001980-002131 /home/dpovey/kaldi-trunk/egs/swbd/s5/mfcc/raw_mfcc_train.1.ark:54975
sw02001-A_002736-002893 /home/dpovey/kaldi-trunk/egs/swbd/s5/mfcc/raw_mfcc_train.1.ark:62762
```

这个文建指向了很多我们提取到的特征的原文件，而这也正是我们在一些脚本使用的。该文件的格式是

```
<utterance-id> <extended-filename-of-features>
```

其中 extended filename `/home/dpovey/kaldi-trunk/egs/swbd/s5/mfcc/raw_mfcc_train.1.ark:24` 表示从这个文件的第24个位置开始读（使用 `fseek()` 函数）。

文件 `feats.scp` 是用下面的命令创建的。

```
steps/make_mfcc.sh --nj 20 --cmd "$train_cmd" data/train exp/make_mfcc/train $mfccdir
```

最后一个文件是 `cmvn.scp`，其保存了倒谱归一化均值和方差（cepstral mean and variance normalization）的统计信息.

```
$ head -3 data/train/cmvn.scp
2001-A /home/dpovey/kaldi-trunk/egs/swbd/s5/mfcc/cmvn_train.ark:7
2001-B /home/dpovey/kaldi-trunk/egs/swbd/s5/mfcc/cmvn_train.ark:253
2005-A /home/dpovey/kaldi-trunk/egs/swbd/s5/mfcc/cmvn_train.ark:499
```

该文件格式是

```
<speaker-id> <extended-filename-of-cmvn>
```

可以用下面的命令创建：

```
steps/compute_cmvn_stats.sh data/train exp/make_mfcc/train $mfccdir
```

由于数据中可能存在错误，我们可以用下面的命令来 check 数据的格式。

```
utils/validate_data_dir.sh data/train
```

也可以尝试用下面的命令来修正一些错误，比如排序错误等。

# 数据准备——语言部分

现在来看 `data/lang` 目录

```
$ ls data/lang
L.fst  L_disambig.fst  oov.int	oov.txt  phones  phones.txt  topo  words.txt
```

其中 `phones` 是一个目录

```
$ ls data/lang/phones
context_indep.csl  disambig.txt         nonsilence.txt        roots.txt    silence.txt
context_indep.int  extra_questions.int  optional_silence.csl  sets.int     word_boundary.int
context_indep.txt  extra_questions.txt  optional_silence.int  sets.txt     word_boundary.txt
disambig.csl       nonsilence.csl       optional_silence.txt  silence.csl
```

这个目录包含了音素集的一些信息，里面的文件，不同格式的文件包含相同的信息。幸运的是，你并需要手动去创建这里所有的文件，因为我们有一个脚本可以使用。

```
utils/prepare_lang.sh
```

这个脚本只需要一些简单的输入就可以完成上面大部分文件的创建。



# `lang` 目录的组成

首先来看 `phones.txt` 和 `words.txt`。

```
$ head -3 data/lang/phones.txt
<eps> 0
SIL 1
SIL_B 2
s5# head -3 data/lang/words.txt
<eps> 0
!SIL 1
-'S 2
```

这两个文件是 kaldi 用来在整数和文本形式之间来回映射的。他们可能会被下面的两个脚本使用。

```
utils/int2sym.pl
utils/sym2int.pl
```

`L.fst` 是有限状态机形式的词典，用音素符号输入，得到词的符号输出。`L_disambig.fst` 是上面提及的词典，但是包含了歧义的符号 ` #1, #2`等。

文件 `data/lang/oov.txt` 仅仅包含一行。

```
$ grep -w UNK data/local/dict/lexicon.txt
<UNK> SPN
```

文件 `oov.int` 包含了这个符号的整数形式。

```
$ cat data/lang/oov.txt
<UNK>
$ cat data/lang/oov.int 
2
```

文件 `data/lang/topo` 包含了一些数据

```
$ cat data/lang/topo
<Topology>
<TopologyEntry>
<ForPhones>
21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69 70 71 72 73 74 75 76 77 78 79 80 81 82 83 84 85 86 87 88 89 90 91 92 93 94 95 96 97 98 99 100 101 102 103 104 105 106 107 108 109 110 111 112 113 114 115 116 117 118 119 120 121 122 123 124 125 126 127 128 129 130 131 132 133 134 135 136 137 138 139 140 141 142 143 144 145 146 147 148 149 150 151 152 153 154 155 156 157 158 159 160 161 162 163 164 165 166 167 168 169 170 171 172 173 174 175 176 177 178 179 180 181 182 183 184 185 186 187 188
</ForPhones>
<State> 0 <PdfClass> 0 <Transition> 0 0.75 <Transition> 1 0.25 </State>
<State> 1 <PdfClass> 1 <Transition> 1 0.75 <Transition> 2 0.25 </State>
<State> 2 <PdfClass> 2 <Transition> 2 0.75 <Transition> 3 0.25 </State>
<State> 3 </State>
</TopologyEntry>
<TopologyEntry>
<ForPhones>
1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20
</ForPhones>
<State> 0 <PdfClass> 0 <Transition> 0 0.25 <Transition> 1 0.25 <Transition> 2 0.25 <Transition> 3 0.25 </State>
<State> 1 <PdfClass> 1 <Transition> 1 0.25 <Transition> 2 0.25 <Transition> 3 0.25 <Transition> 4 0.25 </State>
<State> 2 <PdfClass> 2 <Transition> 1 0.25 <Transition> 2 0.25 <Transition> 3 0.25 <Transition> 4 0.25 </State>
<State> 3 <PdfClass> 3 <Transition> 1 0.25 <Transition> 2 0.25 <Transition> 3 0.25 <Transition> 4 0.25 </State>
<State> 4 <PdfClass> 4 <Transition> 4 0.75 <Transition> 5 0.25 </State>
<State> 5 </State>
</TopologyEntry>
</Topology>
```

它定义了我们所要使用的 HMMs 的拓扑。“真实”音素包含具有标准 3 状态从左到右拓扑的“发射模式”的三种发射状态。

在目录 ` data/lang/phones/` 下有很多关于音素集的文件，这些文件一般有有三种类型 `txt`、`csl` 和 `int`。

```
$ head -3 data/lang/phones/context_indep.txt
SIL
SIL_B
SIL_E
$ head -3 data/lang/phones/context_indep.int
1
2
3
$ cat data/lang/phones/context_indep.csl
1:2:3:4:5:6:7:8:9:10:11:12:13:14:15:16:17:18:19:20
```

他们包含了相同的信息。下面主要关注 `.txt`格式的。文件 `context_indep.txt` 包含了我们建立上下文无关的模型时的音素。也就是对于这些音素，我们不需要用决策树来询问上下文的信息（左右两边）。实际上，我们建立了一个小的决策树来询问中间音素和 HMMs 的状态。

文件 `context_indep.txt` 包括了所有不是“真音素”的音素，比如：silence (SIL), spoken noise (SPN), non-spoken noise (NSN), and laughter (LAU)。

```
$ cat data/lang/phones/context_indep.txt
SIL
SIL_B
SIL_E
SIL_I
SIL_S
SPN
SPN_B
SPN_E
SPN_I
SPN_S
NSN
NSN_B
NSN_E
NSN_I
NSN_S
LAU
LAU_B
LAU_E
LAU_I
LAU_S
```

由于词的位置的依赖，这里有很多这些音素的变体。

> Here, SIL would be the silence that gets optionally inserted by the lexicon (not part of a word), SIL_B would be a silence phone at the beginning of a word (which should never exist), SIL_I word-internal silence (unlikely to exist), SIL_E word-ending silence (should never exist), and SIL_S would be silence as a "singleton word"

文件 `silence.txt` 和 `nonsilence.txt` 分别包括了静音音素和非静音音素，它们互不相交，但是合起来包括了所有的音素。在这个特例中，`silence.txt` 和 `context_indep.txt` 的内容是一样的。非静音音素是那些我们打算估计各种线性变换的音素。

```
$ head -3 data/lang/phones/silence.txt
SIL
SIL_B
SIL_E
$ head -3 data/lang/phones/nonsilence.txt
IY_B
IY_E
IY_I
```

文件 `disambig.txt` 包含了消歧符号的列表。

```
$ head -3 data/lang/phones/disambig.txt
#0
#1
#2
```

这些符号存在于文件 `phones.txt`，似乎他们也是音素一样。


文件 ` optional_silence.txt` 只包含了音素，他可以在两个词之间出现。

```
$ cat data/lang/phones/optional_silence.txt
SIL
```

文件 `sets.txt` 包括了我们聚类在一起的音素，在聚类时我们认为他们是相同的。

```
$ head -3 data/lang/phones/sets.txt
SIL SIL_B SIL_E SIL_I SIL_S
SPN SPN_B SPN_E SPN_I SPN_S
NSN NSN_B NSN_E NSN_I NSN_S
```

文件 `extra_questions.txt` 包括了一些额外的问题，在我们聚类的时候产生问题时使用到。

```
$ cat data/lang/phones/extra_questions.txt
IY_B B_B D_B F_B G_B K_B SH_B L_B M_B N_B OW_B AA_B TH_B P_B OY_B R_B UH_B AE_B S_B T_B AH_B V_B W_B Y_B Z_B CH_B AO_B DH_B UW_B ZH_B EH_B AW_B AX_B EL_B AY_B EN_B HH_B ER_B IH_B JH_B EY_B NG_B
IY_E B_E D_E F_E G_E K_E SH_E L_E M_E N_E OW_E AA_E TH_E P_E OY_E R_E UH_E AE_E S_E T_E AH_E V_E W_E Y_E Z_E CH_E AO_E DH_E UW_E ZH_E EH_E AW_E AX_E EL_E AY_E EN_E HH_E ER_E IH_E JH_E EY_E NG_E
IY_I B_I D_I F_I G_I K_I SH_I L_I M_I N_I OW_I AA_I TH_I P_I OY_I R_I UH_I AE_I S_I T_I AH_I V_I W_I Y_I Z_I CH_I AO_I DH_I UW_I ZH_I EH_I AW_I AX_I EL_I AY_I EN_I HH_I ER_I IH_I JH_I EY_I NG_I
IY_S B_S D_S F_S G_S K_S SH_S L_S M_S N_S OW_S AA_S TH_S P_S OY_S R_S UH_S AE_S S_S T_S AH_S V_S W_S Y_S Z_S CH_S AO_S DH_S UW_S ZH_S EH_S AW_S AX_S EL_S AY_S EN_S HH_S ER_S IH_S JH_S EY_S NG_S
SIL SPN NSN LAU
SIL_B SPN_B NSN_B LAU_B
SIL_E SPN_E NSN_E LAU_E
SIL_I SPN_I NSN_I LAU_I
SIL_S SPN_S NSN_S LAU_S
```

文件 `word_boundary.txt` 包括了每个词的边界信息，如

```
$ head  data/lang/phones/word_boundary.txt
SIL nonword
SIL_B begin
SIL_E end
SIL_I internal
SIL_S singleton
SPN nonword
SPN_B begin
```

文件 `roots.txt` 包括了我们在建立音素上下文决策树时需要的信息。

```
$ head data/lang/phones/roots.txt
shared split SIL SIL_B SIL_E SIL_I SIL_S
shared split SPN SPN_B SPN_E SPN_I SPN_S
shared split NSN NSN_B NSN_E NSN_I NSN_S
shared split LAU LAU_B LAU_E LAU_I LAU_S
...
shared split B_B B_E B_I B_S
```

## `lang` 目录的创建

可以用下面的简单命令：

```
data/local/dict "<UNK>" data/local/lang data/lang
```

其中 `data/local/dict` 是输入目录，`<UNK>` 是 `data/lang/oov.txt` 的内容，`data/local/lang/` 是一个临时目录，而 `data/lang/` 是真正的输出目录。

所以你需要创建目录 `data/local/dict`。

```
$ ls data/local/dict
extra_questions.txt  lexicon.txt nonsilence_phones.txt  optional_silence.txt  silence_phones.txt
```

```
$ head -3 data/local/dict/nonsilence_phones.txt
IY
B
D
$ cat data/local/dict/silence_phones.txt
SIL
SPN
NSN
LAU
$ head -3 data/local/dict/nonsilence_phones.txt
S
UW UW0 UW1 UW2
T
$ cat data/local/dict/extra_questions.txt
$ head -5 data/local/dict/lexicon.txt
!SIL SIL
-'S S
-'S Z
-'T K UH D EN T
-1K W AH N K EY
```

我们只是列出了非静音音素，静音音素和可选的静音音素，以及一个词典，`extra_questions.txt` 是空的。词典 `lexicon.txt` 的格式是：

```
<word> <phone1> <phone2> ...
```

注意，如果一个字有不同发音的话，该文件会包含同一个字的重复条目。如果你想使用发音概率，需要创建另外一个文件 `lexiconp.txt`。它在 `lexicon.txt` 加上了发音概率，格式如下：

```
<word> <prob> <phone1> <phone2> ...
```

如：

```
$ head data/local/dict/lexiconp.txt 
!sil 1 sil
-'s 0.285714 z
-'s 1 s
-'t 1 k uh d en t
-1k 1 w ah n k ey
-able 1 ax b ax l
-ains 1 t ih n
-an 1 ae n
-arious 1 ae r iy ih s
-as 0.125 ah z
```

我们还需要介绍 `utils/prepare_lang.sh` 的使用方法。

```
usage: utils/prepare_lang.sh <dict-src-dir> <oov-dict-entry> <tmp-dir> <lang-dir>
e.g.: utils/prepare_lang.sh data/local/dict <SPOKEN_NOISE> data/local/lang data/lang
<dict-src-dir> should contain the following files:
 extra_questions.txt  lexicon.txt nonsilence_phones.txt  optional_silence.txt  silence_phones.txt
See http://kaldi-asr.org/doc/data_prep.html#data_prep_lang_creating for more info.
options: 
     --num-sil-states <number of states>             # default: 5, #states in silence models.
     --num-nonsil-states <number of states>          # default: 3, #states in non-silence models.
     --position-dependent-phones (true|false)        # default: true; if true, use _B, _E, _S & _I
                                                     # markers on phones to indicate word-internal positions. 
     --share-silence-phones (true|false)             # default: false; if true, share pdfs of 
                                                     # all non-silence phones. 
     --sil-prob <probability of silence>             # default: 0.5 [must have 0 <= silprob < 1]
     --phone-symbol-table <filename>                 # default: ""; if not empty, use the provided 
                                                     # phones.txt as phone symbol table. This is useful 
                                                     # if you use a new dictionary for the existing setup.
     --unk-fst <text-fst>                            # default: none.  e.g. exp/make_unk_lm/unk_fst.txt.
                                                     # This is for if you want to model the unknown word
                                                     # via a phone-level LM rather than a special phone
                                                     # (this should be more useful for test-time than train-time).
     --extra-word-disambig-syms <filename>           # default: ""; if not empty, add disambiguation symbols
                                                     # from this file (one per line) to phones/disambig.txt,
                                                     # phones/wdisambig.txt and words.txt
```

一个很重要的选项是 ` –share-silence-phones`，另一个是 `--sil-prob`。


## 创建语言模型或文法

在一些例子中，我们会有很多 `lang` 目录，比如在 WSJ 的例子中

```
$ echo data/lang*
data/lang data/lang_test_bd_fg data/lang_test_bd_tg data/lang_test_bd_tgpr data/lang_test_bg \
 data/lang_test_bg_5k data/lang_test_tg data/lang_test_tg_5k data/lang_test_tgpr data/lang_test_tgpr_5k
```


`G.fst` 文件会随着是否使用统计语言模型和采用某种元法有关，在 RM 中使用的 bigram grammar。这个在文件 `local/rm_data_prep.sh` 中声明：

```
local/make_rm_lm.pl $RMROOT/rm1_audio1/rm1/doc/wp_gram.txt  > $tmpdir/G.txt || exit 1;
```

文件 `local/make_rm_lm.pl` 创建一个 FST 格式的文法，其包括类似这样的行：

```
$ head data/local/tmp/G.txt
0    1    ADD    ADD    5.19849703126583
0    2    AJAX+S    AJAX+S    5.19849703126583
0    3    APALACHICOLA+S    APALACHICOLA+S    5.19849703126583
```

脚本 `local/rm_prepare_grammar.sh` 包含了命令将 `G.txt` 转成 `G.fst`。

```
fstcompile --isymbols=data/lang/words.txt --osymbols=data/lang/words.txt --keep_isymbols=false \
    --keep_osymbols=false $tmpdir/G.txt > data/lang/G.fst
```

脚本 ` utils/format_lm.sh ` 会 ARPA 格式的语言模型转换成 FST 格式。

```
Usage: utils/format_lm.sh <lang_dir> <arpa-LM> <lexicon> <out_dir>
E.g.: utils/format_lm.sh data/lang data/local/lm/foo.kn.gz data/local/dict/lexicon.txt data/lang_test
Convert ARPA-format language models to FSTs.
```

该文件中重要的一句是

```
gunzip -c $lm \
  | arpa2fst --disambig-symbol=#0 \
             --read-symbol-table=$out_dir/words.txt - $out_dir/G.fst
```

其中 `arpa2fst` 是最主要的。



