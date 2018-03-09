---
layout: post
title: Understanding LSTM Networks
tags: [lstm]
---


This blog is reprinted from [colah's blog](http://colah.github.io/posts/2015-08-Understanding-LSTMs/) and some changes are added by myself.

# About RNN

Humans don’t start their thinking from scratch every second. And traditional neural networks have a major shortcoming, and  they cannot learn from the previous information. Recurrent neural networks (RNN) address this issue.

RNN are networks with loops in them to allow information to persist, shown as the follwing figure (left).

![RNN](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/RNN-unrolled.png)



The right part of the above figure is the form if unroll the loop. In RNN, the cell A's value $$h_t$$ is determined by both the current input $$x_t$$ and the previous cell's state $$h_{t-1}$$. That is

$$
h_t=\sigma(W_{xh}x_t+W_{hh}h_{t-1}+b_h)
$$

In the last few years, there have been incredible success applying RNNs to a variety of problems: speech recognition, language modeling, translation, image captioning. And the essential to these successes is the use of “LSTMs,” a very special kind of recurrent neural network which works, for many tasks, much much better than the standard version.


# Long-Term Dependencies


It seems that RNNs might be able to connect previous information to the present task, such as using previous video frames might inform the understanding of the present frame. But always they can not.

For example, consider a language model trying to predict the next word based on the previous ones. If we are trying to predict the last word in “the clouds are in the *sky*,” we don’t need any further context – it’s pretty obvious the next word is going to be sky. In such cases, where the gap between the relevant information and the place that it’s needed is small, RNNs can learn to use the past information.

![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/RNN-shorttermdepdencies.png)

But there are also cases where we need more context. Consider trying to predict the last word in the text “I grew up in France… I speak fluent *French*.” Recent information suggests that the next word is probably the name of a language, but if we want to narrow down which language, we need the context of France, from further back. It’s entirely possible for the gap between the relevant information and the point where it is needed to become very large.

Unfortunately, as that gap grows, RNNs become unable to learn to connect the information.

![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/RNN-longtermdependencies.png)

In theory, RNNs are absolutely capable of handling such “long-term dependencies.” A human could carefully pick parameters for them to solve toy problems of this form. Sadly, in practice, RNNs don’t seem to be able to learn them. Thankfully, LSTMs don’t have this problem!

# LSTM

Long Short Term Memory (LSTM) networks are a special kind of RNN, capable of learning long-term dependencies. They work tremendously well on a large variety of problems, and are now widely used.

LSTMs are explicitly designed to avoid the long-term dependency problem. Remembering information for long periods of time is practically their default behavior, not something they struggle to learn!

All recurrent neural networks have the form of a chain of repeating modules of neural network. In standard RNNs, this repeating module will have a very simple structure, such as a single tanh layer.

![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-SimpleRNN.png)

LSTMs also have this chain like structure, but the repeating module has a different structure. Instead of having a single neural network layer, there are four, interacting in a very special way.

![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-chain.png)

![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM2-notation.png)

The processed input:

$$
g_t=\text{tanh}(W_{xg}x_t+W_{hg}h_{t-1}+b_g)
$$

And the input gate:

$$
i_t=\sigma(W_{xi}x_t+W_{hi}h_{t-1}+W_{ci}c_{t-1}+b_i)
$$

The foget gate:

$$
f_t=\sigma(W_{xf}x_t+W_{hf}h_{t-1}+W_{cf}c_{t-1}+b_f)
$$

And the cell state: 

$$
c_t=i_t\odot g_t + f_t\odot c_{t-1}
$$

The output gate:

$$
o_t=\sigma(W_{xo}x_t+W_{ho}h_{t-1}+W_{co}c_{t}+b_o)
$$


The cell's output:

$$
h_t=\text{tanh}(c_t)\odot o_t
$$

# Conclusion

LSTMs were a big step in what we can accomplish with RNNs. It’s natural to wonder: is there another big step? A common opinion among researchers is: “Yes! There is a next step and it’s attention!” The idea is to let every step of an RNN pick information to look at from some larger collection of information. For example, if you are using an RNN to create a caption describing an image, it might pick a part of the image to look at for every word it outputs. In fact, Xu, et al. (2015) do exactly this – it might be a fun starting point if you want to explore attention! There’s been a number of really exciting results using attention, and it seems like a lot more are around the corner…

