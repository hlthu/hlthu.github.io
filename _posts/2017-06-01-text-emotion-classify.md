---
layout: post
title: Using LSTM for Text Emotion Classification
tags: [lstm]
---

之前简单介绍过 LSTM，[网址](https://hlthu.github.io/2017/05/30/understand-lstm/)，本文将利用 LSTM 来实现一个文本情感分类模型，是基于 Keras 和 Python 的。

# 概要

本文以词为单位，先分词，然后将每个句子截断为100词（不够则补空字符串），然后将句子以“词-词向量(embedding)”的矩阵形式输入到 LSTM 模型中进行学习分类。
本文使用的语料和参考来源于 [文本情感分类（三）：分词 OR 不分词](http://kexue.fm/archives/3863/).

# 数据预处理

## 数据读取与分词

```python
# 读取数据并分词
pos = pd.read_excel('pos.xls', header=None)
pos['label'] = 1
neg = pd.read_excel('neg.xls', header=None)
neg['label'] = 0
all_ = pos.append(neg, ignore_index=True)
all_['words'] = all_[0].apply(lambda s: list(jieba.cut(s))) #调用结巴分词
```

## 句子截断与构建词典

```python
maxlen = 100 #截断词数
min_count = 5 #出现次数少于该值的词扔掉

# 将所有分词结果放在一起
content = []
for i in all_['words']:
	content.extend(i)

# 计算每个分词出现的次数，构建词典
abc = pd.Series(content).value_counts()
abc = abc[abc >= min_count]
abc[:] = range(1, len(abc)+1)
abc[''] = 0 #添加空字符串用来补全
word_set = set(abc.index)

# 将词转成数字
def doc2num(s, maxlen):
    s = [i for i in s if i in word_set]
    s = s[:maxlen] + ['']*max(0, maxlen-len(s))
    return list(abc[s])

import keras
all_['doc2num'] = all_['words'].apply(lambda s: doc2num(s, maxlen))
```

## 打乱数据并生成 Keras 格式的数据

```python
# 手动打乱数据
idx = range(len(all_))
np.random.shuffle(idx)
all_ = all_.loc[idx]

# 按keras的输入要求来生成数据
x = np.array(list(all_['doc2num']))
y = np.array(list(all_['label']))
y = y.reshape((-1,1)) #调整标签形状
```

# 神经网络定义


```python
from keras.models import Sequential
from keras.layers import Dense, Activation, Dropout, Embedding
from keras.layers import LSTM

# 建立模型
model = Sequential()
model.add(Embedding(len(abc), 256, input_length=maxlen))
model.add(LSTM(128))
model.add(Dropout(0.5))
model.add(Dense(1))
model.add(Activation('sigmoid'))
model.compile(loss='binary_crossentropy',
              optimizer='adam',
              metrics=['accuracy'])
```


# LSTM 训练与测试

训练时使用前15000条语料，剩余的用于测试。本文是在 64 核的 CPU 服务器上运行的，一个 epoch 大约 75s.

```python
batch_size = 128
train_num = 15000
model.fit(x[:train_num], y[:train_num], batch_size = batch_size, epochs=30)
predict = model.predict_classes(x[train_num:]) #输出预测结果
acc = (predict == y[train_num:])
acc_rate = sum(acc) / float(len(acc))
print(acc_rate)
```

# 运行结果

最终在训练集上的正确率是 99.84%，在测试集上的准确率是 91.46%.

```
Building prefix dict from the default dictionary ...
Loading model from cache /tmp/jieba.cache
Loading model cost 0.347 seconds.
Prefix dict has been built succesfully.
Using TensorFlow backend.
Epoch 1/30
15000/15000 [==============================] - 70s - loss: 0.6314 - acc: 0.6470
Epoch 2/30
15000/15000 [==============================] - 71s - loss: 0.5524 - acc: 0.7257
Epoch 3/30
15000/15000 [==============================] - 72s - loss: 0.4716 - acc: 0.8085
Epoch 4/30
15000/15000 [==============================] - 73s - loss: 0.5120 - acc: 0.7481
Epoch 5/30
15000/15000 [==============================] - 73s - loss: 0.4216 - acc: 0.8057
..........
Epoch 26/30
15000/15000 [==============================] - 75s - loss: 0.0129 - acc: 0.9977
Epoch 27/30
15000/15000 [==============================] - 75s - loss: 0.0109 - acc: 0.9981
Epoch 28/30
15000/15000 [==============================] - 75s - loss: 0.0179 - acc: 0.9960
Epoch 29/30
15000/15000 [==============================] - 75s - loss: 0.0126 - acc: 0.9969
Epoch 30/30
15000/15000 [==============================] - 75s - loss: 0.0098 - acc: 0.9984
6105/6105 [==============================] - 14s
[ 0.91466011]
```