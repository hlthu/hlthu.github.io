---
layout: post
feature-img: "assets/img/pexels/computer.jpeg"
img: "assets/img/pexels/computer.jpeg"
title: 利用Kaldi做语言模型的自适应2
tags: [Kaldi,LM]
---

最近需要做一个语言模型的自适应，所谓自适应，是指我已经有了一个通用的语言模型，比如 librispeech 的 tgsmall，但是我现在的任务大部分是朗读一段内容，我希望提高语言模型在解码中的比重，于是想利用新的语料训练一个新的语言模型，然后和之前的通用的融合。

## 1. 生成一个 arpa 语言模型

将下面的语料存为 corpus.txt：

```
I'D LIKE TO GIVE YOU SOME ADVICE ON MAKING FRIENDS IN SCHOOLS
FIRST JOIN A CLUB
THERE ARE MANY STUDENT CLUBS
IF YOU LIKE READING YOU MAY JOIN A BOOK CLUB
YOU WILL FIND PEOPLE LOVE READING BOOKS JUST LIKE YOU THERE
IF YOU LIKE SPORTS LET'S SAY BASKETBALL YOU CAN JOIN A BASKETBALL CLUB
YOU PLAY BASKETBALL WITH THE MEMBERS AND MAKE FRIENDS WITH THEM
IT'S EASY TO MAKE FRIENDS WITH SAME HOBBIES IN THE CLUBS
SECOND TRY TO BE POLITE AND HELPFUL ALL THE TIME NO ONE LIKES RUDE PEOPLE
YOU'D BETTER SAY PLEASE AND THANK YOU WHEN YOU SHOULD
AND MAKE SURE NOT TO TALK BAD THINGS ABOUT PEOPLE BEHIND THEM
THIS BAD HABIT WILL TURN PEOPLE AWAY FROM YOU WHEN SOMEBODY NEEDS YOUR HELP
PLEASE TRY YOUR BEST TO HELP HIM
THIS WAY YOU WILL BE SURPRISED TO SEE HOW MANY FRIENDS YOU WILL MAKE
I'D LIKE TO GIVE YOU SOME ADVICE ON MAKING FRIENDS IN SCHOOLS FIRST JOIN A CLUB THERE ARE MANY STUDENT CLUBS
IF YOU LIKE READING YOU MAY JOIN A BOOK CLUB YOU WILL FIND PEOPLE LOVE READING BOOKS JUST LIKE YOU THERE
IF YOU LIKE SPORTS LET'S SAY BASKETBALL YOU CAN JOIN A BASKETBALL CLUB YOU PLAY BASKETBALL WITH THE MEMBERS AND MAKE FRIENDS WITH THEM
IT'S EASY TO MAKE FRIENDS WITH SAME HOBBIES IN THE CLUBS SECOND TRY TO BE POLITE AND HELPFUL ALL THE TIME
NO ONE LIKES RUDE PEOPLE YOU'D BETTER SAY PLEASE AND THANK YOU WHEN YOU SHOULD AND MAKE SURE NOT TO TALK BAD THINGS ABOUT PEOPLE BEHIND THEM
THIS BAD HABIT WILL TURN PEOPLE AWAY FROM YOU WHEN SOMEBODY NEEDS YOUR HELP PLEASE TRY YOUR BEST TO HELP HIM
THIS WAY YOU WILL BE SURPRISED TO SEE HOW MANY FRIENDS YOU WILL MAKE
I'D LIKE TO GIVE YOU SOME ADVICE ON MAKING FRIENDS IN SCHOOLS
FIRST JOIN A CLUB THERE ARE MANY STUDENT CLUBS IF YOU LIKE READING YOU MAY JOIN A BOOK CLUB
YOU WILL FIND PEOPLE LOVE READING BOOKS JUST LIKE YOU THERE IF YOU LIKE SPORTS LET'S SAY BASKETBALL YOU CAN JOIN A BASKETBALL CLUB
YOU PLAY BASKETBALL WITH THE MEMBERS AND MAKE FRIENDS WITH THEM IT'S EASY TO MAKE FRIENDS WITH SAME HOBBIES IN THE CLUBS
SECOND TRY TO BE POLITE AND HELPFUL ALL THE TIME NO ONE LIKES RUDE PEOPLE YOU'D BETTER SAY PLEASE AND THANK YOU WHEN YOU SHOULD
AND MAKE SURE NOT TO TALK BAD THINGS ABOUT PEOPLE BEHIND THEM THIS BAD HABIT WILL TURN PEOPLE AWAY FROM YOU
WHEN SOMEBODY NEEDS YOUR HELP PLEASE TRY YOUR BEST TO HELP HIM THIS WAY YOU WILL BE SURPRISED TO SEE HOW MANY FRIENDS YOU WILL MAKE
```

然后生成一个 arpa 语言模型，并将这个语言模型转成 FST。

```shell
lm_order=3
local=data/local
mkdir $local/tmp
ngram-count -order $lm_order -write-vocab $local/tmp/vocab-full.txt \
            -wbdiscount -text $local/corpus.txt -lm $local/tmp/lm.arpa
```

## 2. 语言模型融合

`lambda` 越大，表明原来的语言模型 `3-gram.pruned.3e-7.arpa` 的比重越大。

```shell
ngram -lm data/local/3-gram.pruned.3e-7.arpa -order 3 -mix-lm data/local/lm_reading/lm.arpa -lambda 0.5 -write-lm data/local/3-gram.pruned.3e-7_mix0.5.arpa 
ngram -lm data/local/3-gram.pruned.3e-7.arpa -order 3 -mix-lm data/local/lm_reading/lm.arpa -lambda 0.75 -write-lm data/local/3-gram.pruned.3e-7_mix0.75.arpa 
ngram -lm data/local/3-gram.pruned.3e-7.arpa -order 3 -mix-lm data/local/lm_reading/lm.arpa -lambda 0.9 -write-lm data/local/3-gram.pruned.3e-7_mix0.9.arpa 
ngram -lm data/local/3-gram.pruned.3e-7.arpa -order 3 -mix-lm data/local/lm_reading/lm.arpa -lambda 0.1 -write-lm data/local/3-gram.pruned.3e-7_mix0.1.arpa 
ngram -lm data/local/3-gram.pruned.3e-7.arpa -order 3 -mix-lm data/local/lm_reading/lm.arpa -lambda 0.25 -write-lm data/local/3-gram.pruned.3e-7_mix0.25.arpa
```

## 3. 生成解码图与解码

```shell
. cmd.sh 
. path.sh
cp -r lang lang_test_tgsmall_mix0.1
cp -r lang lang_test_tgsmall_mix0.25
cp -r lang lang_test_tgsmall_mix0.5
cp -r lang lang_test_tgsmall_mix0.75
cp -r lang lang_test_tgsmall_mix0.9
arpa2fst --disambig-symbol=#0 --read-symbol-table=lang/words.txt local/3-gram.pruned.3e-7_mix0.1.arpa lang_test_tgsmall_mix0.1/G.fst
arpa2fst --disambig-symbol=#0 --read-symbol-table=lang/words.txt local/3-gram.pruned.3e-7_mix0.25.arpa lang_test_tgsmall_mix0.25/G.fst
arpa2fst --disambig-symbol=#0 --read-symbol-table=lang/words.txt local/3-gram.pruned.3e-7_mix0.5.arpa lang_test_tgsmall_mix0.5/G.fst
arpa2fst --disambig-symbol=#0 --read-symbol-table=lang/words.txt local/3-gram.pruned.3e-7_mix0.75.arpa lang_test_tgsmall_mix0.75/G.fst
arpa2fst --disambig-symbol=#0 --read-symbol-table=lang/words.txt local/3-gram.pruned.3e-7_mix0.9.arpa lang_test_tgsmall_mix0.9/G.fst
utils/mkgraph.sh data/lang_test_tgsmall_mix0.1 exp/nnet3/tdnn exp/nnet3/tdnn/graph_tgsmall_mix0.1
utils/mkgraph.sh data/lang_test_tgsmall_mix0.25 exp/nnet3/tdnn exp/nnet3/tdnn/graph_tgsmall_mix0.25
utils/mkgraph.sh data/lang_test_tgsmall_mix0.5 exp/nnet3/tdnn exp/nnet3/tdnn/graph_tgsmall_mix0.5
utils/mkgraph.sh data/lang_test_tgsmall_mix0.75 exp/nnet3/tdnn exp/nnet3/tdnn/graph_tgsmall_mix0.75
utils/mkgraph.sh data/lang_test_tgsmall_mix0.9 exp/nnet3/tdnn exp/nnet3/tdnn/graph_tgsmall_mix0.9
nj=5
steps/nnet3/decode.sh --nj $nj --cmd "$decode_cmd"  exp/nnet3/tdnn/graph_tgsmall_mix0.1 data/mp3/test exp/nnet3/tdnn/decode_test_tgsmall_mix0.1 &
steps/nnet3/decode.sh --nj $nj --cmd "$decode_cmd"  exp/nnet3/tdnn/graph_tgsmall_mix0.25 data/mp3/test exp/nnet3/tdnn/decode_test_tgsmall_mix0.25 &
steps/nnet3/decode.sh --nj $nj --cmd "$decode_cmd"  exp/nnet3/tdnn/graph_tgsmall_mix0.5 data/mp3/test exp/nnet3/tdnn/decode_test_tgsmall_mix0.5 &
steps/nnet3/decode.sh --nj $nj --cmd "$decode_cmd"  exp/nnet3/tdnn/graph_tgsmall_mix0.75 data/mp3/test exp/nnet3/tdnn/decode_test_tgsmall_mix0.75 &
steps/nnet3/decode.sh --nj $nj --cmd "$decode_cmd"  exp/nnet3/tdnn/graph_tgsmall_mix0.9 data/mp3/test exp/nnet3/tdnn/decode_test_tgsmall_mix0.9 &
```
