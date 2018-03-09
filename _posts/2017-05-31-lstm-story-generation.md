---
layout: post
title: Using LSTM to Generate a Story
tags: [lstm]
---

之前简单介绍过 LSTM，[网址](https://hlthu.github.io/2017/05/30/understand-lstm.html)，本文将利用 LSTM 构建一个故事生成的模型。

# 数据及其预处理

## 数据源

我们使用的数据是 [Aesop’s Fables](http://www.taleswithmorals.com/) 里的一部分内容，如下：

> long ago , the mice had a general council to consider what measures they could take to outwit their common enemy , the cat . some said this , and some said that but at last a young mouse got up and said he had a proposal to make , which he thought would meet the case . you will all agree , said he , that our chief danger consists in the sly and treacherous manner in which the enemy approaches us . now , if we could receive some signal of her approach , we could easily escape from her . i venture , therefore , to propose that a small bell be procured , and attached by a ribbon round the neck of the cat . by this means we should always know when she was about , and could easily retire while she was in the neighbourhood . this proposal met with general applause , until an old mouse got up and said that is all very well , but who is to bell the cat ? the mice looked at one another and nobody spoke . then the old mouse said it is easy to propose impossible remedies .


## 数据读取

我们将改段文字保存在 `story.txt` 中，使用下面的代码读取，代码中有注释，不再解释。

```python
## 读取训练数据
fname = './story.txt'
with open(fname) as f:                                # 读取所有行
	words = f.readlines()
words = [x.strip() for x in words]                    # 删除前导和后缀字符
words = [words[i].split() for i in range(len(words))] # 将句子分割成词， 以空格、TAB、回车等为分隔符
words = np.array(words)                               # 转成向量表示
words = np.reshape(words, [-1, ])
```

## 建立词到数字的映射

```python
## 建立“词典”，实现词到数字的转换
def build_dict(words):
	cnt = collections.Counter(words).most_common() # 计算每个词出现的次数并排序
	dictionary = dict()
	for word, _ in cnt:
		dictionary[word] = len(dictionary)         # 字典的key是词，value是依次增加的序号。
	dictionary_reverse = dict(zip(dictionary.values(), dictionary.keys()))
	return dictionary, dictionary_reverse
## 构建训练数据的字典
dictionary, dictionary_reverse = build_dict(words)
num_words = len(dictionary)
```

# 神经网络定义

## 定义流图输入

```python
## Tensorflow日志与流图输入
log_dir = '/tmp/tf_log'
writer = tf.summary.FileWriter(log_dir)
x = tf.placeholder('float', [None, n_input, 1])
y = tf.placeholder('float', [None, num_words])

weights = {
    'out': tf.Variable(tf.random_normal([n_hidden, num_words]))
}
biases = {
    'out': tf.Variable(tf.random_normal([num_words]))
}
```

## 定义 RNN

```python
def RNN(x, weights, biases):
	x = tf.reshape(x, [-1, n_input])
	x = tf.split(x, n_input, 1)
	rnn_cell = rnn.MultiRNNCell([rnn.BasicLSTMCell(n_hidden),rnn.BasicLSTMCell(n_hidden)])
	outputs, states = rnn.static_rnn(rnn_cell, x, dtype=tf.float32)
	return tf.matmul(outputs[-1], weights['out']) + biases['out']
```

## RNN 前向计算和优化

```python
# RNN前向计算
pred = RNN(x, weights, biases)

# 定义loss(交叉熵)和优化器
cost = tf.reduce_mean(tf.nn.softmax_cross_entropy_with_logits(logits=pred, labels=y))
optimizer = tf.train.RMSPropOptimizer(learning_rate=lr).minimize(cost)

# 定义正确率
acc_pred = tf.equal(tf.argmax(pred,1), tf.argmax(y,1))
acc_rate = tf.reduce_mean(tf.cast(acc_pred, tf.float32))

# 初始化变量
init = tf.global_variables_initializer()
```

# LSTM 训练与测试

```python
## LSTM训练的相关参数
lr = 0.001
train_iters = 50000
display_step = 1000
n_input = 3
n_hidden = 512
# 启动流图
with tf.Session() as session:
	session.run(init)
	# 定义初始的参数
	step = 0
	offset = random.randint(0, n_input+1)
	end_offset = n_input + 1
	acc_total = 0
	loss_total = 0
	writer.add_graph(session.graph)
	
	# 循环迭代
	while step < train_iters:
		if offset > (len(words) - end_offset):
			offset = random.randint(0, n_input+1)
		# 获取输入的三个单词对应的数字
		symbols_in_keys = [ [dictionary[str(words[i])]] for i in range(offset, offset+n_input) ]
		symbols_in_keys = np.reshape(np.array(symbols_in_keys), [-1, n_input, 1])
		# 获取输出对应的单词数字，同时将LSTM概率输出对应的位置置为1
		symbols_out_onehot = np.zeros([num_words], dtype=float)
		symbols_out_onehot[dictionary[str(words[offset+n_input])]] = 1.0
		symbols_out_onehot = np.reshape(symbols_out_onehot,[1,-1])
		# 神经网络前向计算与反向回传
		_ , acc, loss, onehot_pred = session.run([optimizer, acc_rate, cost, pred], feed_dict={x: symbols_in_keys, y: symbols_out_onehot})
		# loss 和 acc 累加
		loss_total += loss
		acc_total += acc
		# 每隔 display_step 打印一次 loss 和 acc
		if (step+1) % display_step == 0:
			print("Iter= " + str(step+1) + ", Average Loss= " + \
                  "{:.6f}".format(loss_total/display_step) + ", Average Accuracy= " + \
                  "{:.2f}%".format(100*acc_total/display_step))
			# 打印当前输入与预测输出，预测输出是概率最大的那一个
			symbols_in = [words[i] for i in range(offset, offset+n_input)]
			symbols_out = words[offset+n_input]
			symbols_out_pred = dictionary_reverse[int(tf.argmax(onehot_pred, 1).eval())]
			print("%s - [%s] vs [%s]" % (symbols_in,symbols_out,symbols_out_pred))
			# 将 loss 重新置为 0
			acc_total = 0
			loss_total = 0
		step += 1
		offset += (n_input+1)
		
	# 测试给定输入求出的32个输出
	while True:
		# 获取输入并处理，和之前读文件类似
		prompt = "%s words: " % n_input
		sentence = input(prompt)
		sentence = sentence.strip()
		sentence = sentence.split()
		sentence = np.array(sentence)
		sentence = np.reshape(sentence, [-1, ])
		if len(sentence) != n_input:
			continue
		try:
			symbols_in_keys = [dictionary[str(sentence[i])] for i in range(len(sentence))]
			for i in range(32):
				keys = np.reshape(np.array(symbols_in_keys), [-1, n_input, 1])
				# 进行一次前向计算，求出预测
				onehot_pred = session.run(pred, feed_dict={x: keys})
				onehot_pred_index = int(tf.argmax(onehot_pred, 1).eval())
				sentence = "%s %s" % (sentence,dictionary_reverse[onehot_pred_index])
				# 将预测结果加到末尾，重新循环预测
				symbols_in_keys = symbols_in_keys[1:]
				symbols_in_keys.append(onehot_pred_index)
			print(sentence)
		except:
			print("Word not in dictionary")
```

# 运行结果

![](https://raw.githubusercontent.com/hlthu/lstm/master/01-story_generation/1.PNG)
