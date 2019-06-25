---
layout: post
title:  "Machine Learning with R (4)"
categories: Machine_Learning 
tags:  ANN SVM R
--- 

* content
{:toc}  

Now I start to learn neural network and SVM. Actually, I am a bit more familiar with those two parts than other machine learning methods if I just use them like black boxes. However, the mathematics behind SVM or ANN  is some kind of complicated. I am not sure I should prob into the math or not. Those days I have so many things want to do, maybe I should subtract some from my busy work.  




这两个是具有巨大潜能的机器学习方法，相对来说也比较复杂，在此处我不打算展开细节，简单介绍一下他们在R中的应用吧。

#### **Neural Network**

哈哈，首先普及一点生物知识。生物神经元细胞的树突通过一个生化过程（神经冲动根据其相对重要性或者频率加权的生化过程）来接受输入信号。那么这个生化过程是什么呢？ 细胞体开始积累输入信号，达到一个阈值后， 细胞便会充满活力击破，然后输出信号通过一个电化过程传送到轴突，在轴突末端，该电信信号在此被处理成一种化学信号，穿过成为突触的一个微小间隙传递到相邻的神经元。

神经网络的要点，激活函数，网络拓扑。下面描述一下BP神经网络的过程，在向前阶段，神经元从输入层到输出层的序列被激活，沿途应用每一个神经元权重和激活函数，一旦到达最后一层，就产生一个输出信号。在向后阶段，由向前阶段产生的网络输出信号与训练数据的真是目标值进行比较，将他们之间的误差向后传播，从而修正神经元之间的连接权重，并减少将来产生的误差。

R中神经网络的添加包有nnet添加包，RSNNS添加包，这里我们使用neuralnet添加包中的neuralnet()函数：

```
m<-neuralnet(target ~ predicetions, data=mydata, hidden=1);
p<-compute(m,test);
```

#### **SVMs**
支持向量，就是每个类中最接近最大间隔超平面的点。

线性核函数（也可以说没有用核函数），S型核函数，高斯核函数（Gaussian RBF kernel）；

e1071添加包中提供一个关于LIBSVM库的R接口，kernlab添加包中的ksvm()函数。

```
m<-ksvm(target~predictions, data=mydata, kernel="rbfdot",c=1); p<-predict(m, test, type="response")
```