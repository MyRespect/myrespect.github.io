---
layout: post
title:  "Deep neural networks"
categories: Deep_Learning
tags: CTC
--- 

* content
{:toc}

In this post, I mainly make notes about some special networks.




#### **Convolution**
输出层的每一个像素是由输入层对应位置的F\*F的局部图片与一组相同F\*F的参数(权值)作内积(对应元素相乘再相加)，再经过非线性单元计算而来。比如，在一层神经网络里面有10个卷积核大小为3\*3\*3的卷积核，那么这一层有(27+1)\*10=280个参数。

#### **Residual**
(1) identity block, when input dimension=output dimension

(2) convolution block, when input and output dimension doesnot match.
The main difference between above two blocks is there is conv2D layer in shortcut path.

#### **Connectionist temporal classification**
CTC是一种计算损失值的方法，主要用于没有实现对齐的序列化数据的训练，在真实世界的序列学习任务中，需要从噪声和未格式化的数据上预测序列的标签．例如，在语言识别中，一个声音信号被转为words或sub-word单元．RNN要求预先分割训练数据，通过后处理将模型的转为label序列，应用受到限制，如何直接对未分割的序列上预测标签呢，一个比较好的组合是LSTM+CTC.　因为网络的输出可以看为在给定的输入下，所有可能对应的label可以看成序列上的一个概率分布，在给定这个分布，目标函数可以直接最大化正确label的概率．

RNN支持CTC模型所需要的输出表示，将网络输出转换为一个在label序列上的条件概率分布，之后对给定的输入，网络通过选择最可能的label分类，这些输出定义了将label序列对齐到输入序列的全部可能方法的概率．任何一个label序列的总概率，可以看做他的不同对齐形式对应的概率累加和．
