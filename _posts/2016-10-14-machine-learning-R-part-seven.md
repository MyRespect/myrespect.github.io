---
layout: post
title:  "Machine Learning with R (7)"
categories: Machine_Learning
tags:  Fine-Tune Ensemble-Learning R
--- 

* content
{:toc}  

In this blog, I will introduce a set of techniques for improving the predictive performance of machine learners, such as how to fine-tune the performance by searching for the optimal set of trainging conditions. What's more, I will record some useful techniques for certain types of work. 




**第一个问题，如何通过寻找训练条件的优化集合来调整机器学习模型的性能呢？** 

我们可以使用caret添加包进行自动参数调整，caret提供了一个train()函数作为标准接口，为分类和回归任务训练[150种](https://caret.r-forge.r-project.org/modelList.html)不同的机器学习模型。 我们需要考虑以下几点： 

1. 使用哪个机器学习模型？
2. 哪些模型参数是可以调整的，调整的范围是多大？
3. 使用什么评价标准来找到最优的候选者呢？

我们用一个例子来说明如何通过调整多个模型来提高性能： 

**首先，我们来创建简单的调整模型**

```
library(caret)
set.seed(300)
m<-train(default~.,data=credit,method="C5.0")
p<-predict(m,credit，type="prob")
```


在这里，train()函数使用最佳模型中的参数对所有数据进行建模，并存储在m$finModel对象中。在大多数情况下，我们无需直接操作finalModel对象，而是使用predict()函数通过m对象进行预测。这样，train()函数应用到数据中的任何数据预处理方法都会应用到预测的数据中，predict()函数提供了标准的接口来得到预测的类别值和概率。

set.seed()函数用来初始化R的随机数发生器，通过设定seed参数，可以使随机数遵循一个预先设定的序列，可以使得train()这样使用随机抽样的模拟方法能够在重复运行中得到相同的结果，如果分享代码并尝试得到之前相同的结果，是非常有用的。

**然后，我们来定制调整过程**

我们使用trainControl()函数来创建一系列的配置选项，可以使用?trainControl命令来详细查看。然后我们创建用来优化参数的表格。expand.grid()函数利用所有的值的组合创建数据框。

```
ctrl<-trainControl(method="cv",number=10,selectionFunction="oneSE")
grid <- expand.grid(.model="tree",.trials=c(1,5,10,15,20,25,30,35),.winnow="FALSE")
m<-train(default~.,data=credit,method="C5.0",metric="Kappa",trControl=ctrl,tuneGrid=grid)
```
这样，我们已经定制好了我们的train()实验模型了，我们使用了selectionFunction函数参数在各个候选模型中选择最佳的模型，这里我们使用了oneSE函数，表示选择了最好性能标准差之内的最简单的候选者，其他的还有best, Tolerance函数参数。


**第二个问题，如何使用元学习来提高模型的性能呢？**

元学习方法，简单来讲是将多个模型合并组成一个更强的组。它可以包含从通过自动迭代设计决策来提升性能的简单算法到借鉴了进化生物学的自修改和自适应的学习方式的复杂算法。 

**集成学习** (Ensemble learning) 基于结合多个很弱的学习器来创建一个很强的学习器。首先输入训练数据用来建立多个模型，分配函数决定每个模型接收完整的训练数据还是某个抽样的样本（可以改变训练的数据或者算法）。创建模型后它可以产生一系列的预测，可以用组合函数对预测中的不一致进行调解。有些集成学习甚至可以使用另一个模型从各种预测的组合中学习一个组合函数，得到一个仲裁模型，我们将这个过程称为堆叠(stacking)。

集成学习方法之一的技术就是自助汇聚法，称为**bagging**方法。bagging对原始数据使用自助法抽样的方法产生很多个训练数据集，这些数据集使用单一的机器学习算法产生多个模型，然后使用投票（分类问题）或平均（数值预测）的方法组合预测值。bagging倾向使用不稳定学习器（随着数据发生很小的变化产生差别很大的模型），因为可以确保当自助法的数据集之间的差异很小时集成学习也具有很好的多样性。基于这个原因，bagging经常与决策树一起使用，因为决策树倾向于随着数据的微小变化而发生较大的改变。

举个例子， ipred添加包中提供了bagging决策树的经典实现，其中nbagg参数控制用来投票的决策树的数目。

```
library(ipred)
mybag<-bagging(default~.,data=credit,nbagg=25)
credit_pred<-predict(mybag,credit)
```
下面来测试模型在未来的性能表现如何,其中ipred添加包中的bagging树的函数是treebag：

```
library(caret)
set.seed(300)
ctrl<-trainControl(method="cv",number=10)
train(default~.,data=credit,method="treebag",trControl=ctrl)
```
我们还可以通过使用caret添加包的bag()函数来配置bagging过程，需要指定用来拟合的模型函数，一个用来预测的函数，一个用来聚集投票的函数。我们来学习一个例子，然后我们使用相同的格式来创建自己的函数，就可以使用bagging来实现任意的机器学习算法。

```
str(svmBag)
svmBag$fit
bagctrl<-bagControl(fit=svmBag$fit,predict=svmBag$pred, aggregate=svmBag$aggregate)
svmbag<-train(default~.,data=credit,"bag",trControl=ctrl,bagControl=bagctrl)
```
caret添加包中还包含朴素贝叶斯模型(nbBag),决策树(ctreeBag),支持向量机(svmBag)和神经网络(nnetBag)的例子。

另一种基于集成学习的方法是**boosting**,被称为自适应boosting。该算法产生弱分类器来迭代的学习训练集中很大比例难以分类的样本，对经常分错的样本给予更大的权重。因为它增加弱学习器的性能来获得强学习器的性能，通过简单的添加更多弱学习器的方式将性能提升，boosting中重抽样数据集是专门构建用来产生互补的模型，而且所有的选票也不是同等重要，会根据性能进行加权。

另一种集成学习方法称为**random forests**，它只关注决策树的集成学习，可以处理非常大量的数据，而大数据中所谓的“维数灾难”常常会让其他模型失败。我们使用randomForest添加包来训练随机森林。

```
m<-randomForest(train,class,ntree=500,mtry=sqrt(p))
p<-predict(m,test,type="response")
```
train是包含训练数据集的数据框，class是一个因子向量，代表训练集中每一行的类别，ntree指定树的数目，mtry代表每次划分中随机选择的特征的数目。

最后，我们介绍一些用于机器学习的R添加包，需要使用时可以查询相关包的使用方法: 

*使用[RCurl添加包](http://ww.omegahat.org/RCurl)从网上抓取数据；
*使用[XML添加包](http://www.omegahat.org/RSXML)读写XML格式的数据；
*使用rjson添加包读写JSON
*使用xlsx添加包读写Excel电子表格
*用data.table添加包使得数据框运算的更快
*用ff添加包构建基于磁盘的数据框
*用bigmemory添加包使用大矩阵
*用[foreach添加包](http://www.revolutionanalytics.com/)来处理并行问题
*用MapReduce和Hadoop添加包进行并行云计算
*并行处理的一个选择是使用计算机图形处理单元GPU增强数学计算速度