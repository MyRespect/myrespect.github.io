---
layout: post
title:  "Deep neural networks"
categories: Deep_Learning
tags: CTC
--- 

* content
{:toc}

In this post, I mainly make notes about some special networks and a guide to an efficient way to build neural network architectures about hyper-parameter selection and tuning, which is refered to posts by Shashank Ramesh in towardsdatascience.




#### **Convolution**
Every pixel of the output layer is computed by F\*F part of the input layer Inner product the F\*F filter, and then applied by the activation function, for example, if there are 10 filters which is 3\*3\*3, then we have (27+1)\*10=280 parameters.

#### **Residual**
(1) identity block, when input dimension=output dimension

(2) convolution block, when input and output dimension doesnot match.
The main difference between above two blocks is there is conv2D layer in shortcut path.

#### **Alchemy**
1.	Hyperas which enables us to take advantage of GPU acceleration enabling you you train models atleast 10x faster and is also an easy way to approach hyper-parameter tuning. Or you can use GridSearchCV, which do not make use of GPU.

2.	 sparse categorical cross-entropy is used when output is NOT one-hot encoded(1,2,3 etc.) and categorical cross-entropy when output is one-hot encoded( [1,0,0] or [0,1,0] ). 

3.	It can be inferred from the graph that the data-set is balanced as we have nearly the same amount of data-points in every class. This check is necessary for choosing Accuracy as a metric. Other alternatives could be F1-score.

4.	gradient exploding or vanishing: learning_rate, batch_norm, RELU, LSTM, residual network,  train per layer and then fine-tunning, gradient clipping, the number of layers.

5.	saddle points and local optima: learning rate, optimizer: Adam or RMSProp.

6.	No convergence: adaptive learning rate optimizers lilke Adam or use decay in our optimizers.

7.	Make sure you make the best out of  your simple architecture and slowly increase complexity and components if required.

8.	if number of layers is high, it might introduce problems like over-fitting and vanishing and exploding gradient problems. lower number may cause high bias. Also, number of hidden units per layer depends on the data size.

9.	sigmoid and tanh are only used for shallow network, RELU is pretty good.

10.	The problem with SGD is that due to the frequent updates and fluctuations it ultimately complicates the convergence to the exact minimum and will keep overshooting due to the frequent fluctuations. Mini Batch Gradient Descentn can educes the variance in the parameter updates , which can ultimately lead us to a much better and stable convergence. We can use these good optimizers: Momentum, Adagrad, RMSProp, Adam.

11.	learning rate: for SGD, 0.1 is generally works well, for Adam-0.001/0.01 is good. You can try in powers of 10 from 0.001 to 1. You can use the decay parameter to reduce your learning with number of  iterations. Generally it is better to use adaptive learning rate algorithms like Adam than using a decaying learning rate.

12.	Initialization, batch size, number of epoches, dropout, L1/L2 regularization.

13.	Use train to learn various patterns in data, validation to learn values for the hyperparameters and test to see if the model generalizes well.

14.	Kernel/Filter size: Smaller filters collect as much local information as possible, bigger filters represent more golbal, high-level and representative information. If you think that differentiates objects are some small and local features you should use small filters. In general, we use filters with odd sizes.

15.	padding: padding is generally used to add columns and rows of zeroes to keep the spatial sizes constant after convolution, doing this might improve performance as it retains the information as the borders.

16.	The more the number of channels, more the number of filters used, more are the features learnt, and more is the chances to over-fit and vice-versa.

17.	Batch normalization: generally in deep neural network architectures the normalized input after passing through various adjustments in intermediate layers becomes too big or too small, which causes a problem of internal co-variate shift which impacts learning. The batch normalization layer should be placed in the architecture after passing it through the layer containing activation function and before the dropout layer(if any). An exception is for the sigmoid activation functino wherein you need to place the batch normalization layer before the activation to ensure that the values lie in linear region of sigmoid.

18.	keep the feature space wide and shallow in the initial stages of the network, and the make it narrower and deeper towards the end.

19.	Always start by using smaller filters is to collect as much local information as possible, and then gradually increase the filter width to reduce the generated feature space width to represent more global, high-level and representative information. The number of filters is increased to increase the depth of the feature space thus helping in learning more levels of global abstract structures. One more utility of making the feature space deeper and narrower is to shrink the feature space for input to the dense networks.

#### **Connectionist temporal classification**

CTC是一种计算损失值的方法，主要用于没有实现对齐的序列化数据的训练，在真实世界的序列学习任务中，需要从噪声和未格式化的数据上预测序列的标签．例如，在语言识别中，一个声音信号被转为words或sub-word单元．RNN要求预先分割训练数据，通过后处理将模型的转为label序列，应用受到限制，如何直接对未分割的序列上预测标签呢，一个比较好的组合是LSTM+CTC.　因为网络的输出可以看为在给定的输入下，所有可能对应的label可以看成序列上的一个概率分布，在给定这个分布，目标函数可以直接最大化正确label的概率．

RNN支持CTC模型所需要的输出表示，将网络输出转换为一个在label序列上的条件概率分布，之后对给定的输入，网络通过选择最可能的label分类，这些输出定义了将label序列对齐到输入序列的全部可能方法的概率．任何一个label序列的总概率，可以看做他的不同对齐形式对应的概率累加和．
