---
layout: post
title: Linux 下批量重命名
tags: [linux]
---

我们做机器学习的，首先需要对数据进行处理，但是一般我们拿到的数据的命名常常不是符合自己的需求的，因此通常需要重命名，一个两个文件还好，一堆成千上万自己手动改可能就不好了，这里简单介绍一下如何在 Linux 下完成对数据的批量重命名。

假设有这么一批数据

```
huanglu@DeepNet1:~/test$ ls
I5019_N11.jpg   I5160_N119.jpg  I5213_N68.jpg   I5679_N13.jpg
I5057_N75.jpg   I5174_N96.jpg   I5343_N120.jpg  I5733_N52.jpg
I5148_N103.jpg  I5192_N81.jpg   I5415_N29.jpg
```

我们需要命名为 `pos_序号_train.jpg`的形式，可以使用下面的命令：

```
huanglu@DeepNet1:~/test$ i=1
huanglu@DeepNet1:~/test$ for x in ./*.jpg; do mv "$x" "pos_${i}_train.jpg"; i=`expr $i + 1`; done
```

注意 `expr $i + 1` 里的空格必须要有。重命名完的数据如下：

```
huanglu@DeepNet1:~/test$ ls
pos_10_train.jpg  pos_2_train.jpg  pos_5_train.jpg  pos_8_train.jpg
pos_11_train.jpg  pos_3_train.jpg  pos_6_train.jpg  pos_9_train.jpg
pos_1_train.jpg   pos_4_train.jpg  pos_7_train.jpg
```