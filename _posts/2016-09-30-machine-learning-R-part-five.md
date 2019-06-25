---
layout: post
title:  "Machine Learning with R (5)"
categories: Machine_Learning 
tags:  Asocciation-rules Apriori K-means Cluster R
--- 

* content
{:toc}  

In this blog, I would like to share some knowledge about market basket analysis. The blog will cover machine learning methods for identifying associations among items in transactional data. You will learn the start-to-finish steps needed for using association rules to perform a market basket analysis on real-world data.    




#### **Apriori Algorithm**

关联规则是解决大数据问题的一种方法，作为一种无监督的学习算法，它能够从没有任何关于模式的先验知识的大型数据库中提取知识。 

首先，我们来理解一下关联规则和事务型数据： 

{花生酱，果冻} ——> {面包}。  

**Apriori property**: 一个频繁项集的所有子集也必须是频繁的。 

度量关联规则的兴趣度： 

**support and confidence**

如果一个规则同时具有高支持度和置信度，则称为强规则。支持度是指项集在数据中出现的频率,support(A,B)与P(A,B)是一样的;置信度表示交易中项或者项集X的出现导致项或者项集Y出现的比例，confidence(A->B)与P(B\|A)是一样的。

**提升度**(lift)，假设知道一类商品已经被购买，提升都就是用来度量一类商品相对于它的一般购买率，lift(x->y)=confidence(x->y)/support(x)。

**Aprior算法**

创建规则主要分两步。第一步，识别所有满足最小支持度阈值的项集；第二步，根据满足最小置信度阈值的这些项来创建规则。

首先是数据准备，为交易数据创建一个**稀疏矩阵**，稀疏矩阵的每一行表示一次交易，用列表示可能出现在购物者购物篮的每一件商品。稀疏矩阵实际在内存中没有存储完整的矩阵，知识存储了一个商品所占用的单元，这样的话该结构的内存效率比一个大小相当的矩阵或者数据框的内存效率更高。 

下面介绍一下R语言中的关联规则添加包 arules中的几个函数 ：read.transactions()函数可以产生用于事务性数据的稀疏矩阵； 
inspect()函数可以用来查看交易数据，检验关联规则； 
itemFrequency()函数查看商品频率比例； 
itemFrequencyPlot()函数可视化商品的支持度； 
image()通过绘制稀疏矩阵来可视化交易数据； 
sample()函数可以对数据进行随机抽样； 
sort()可以用来对规则列表重新排序； 
subset()函数提供了一种用来寻找交易或者规则子集的方法，关键词items与出现在规则任何位置的项相匹配：

```
berryrules <- subset(groceryrules, items %in% "berries")
```
好了，现在我们来看arules添加包中的关联规则函数apriori()的用法：

```
myrules <- apriori(data=mydata, parameter=list(support=0.1,confidence=0.8,minlen=1))
#minlen:the least number of rules
```

#### **K-Means Cluster**

聚类的原则是一个组内的记录，彼此必须非常相似，而与组外的记录截然不同。 

顺便插一句， 什么叫半监督学习？如果你一开始从无标签的数据入手，那么你首先可以使用聚类来创建分类标签，然后，你可以应用一个有监督学习算法，如决策树，寻找类中重要的预测指标，这就叫semi-supervised learning。 

k均值算法涉及将n个案例中的每一个案例分配到k个类中的一个，k是一个事先定义好的数。该算法使用一个可以找到局部最优解的启发式过程，它从类分配的最初猜测开始，然后对分配稍加修正以查看该变化是否提升了类内部的同质性。K均值将特征值作为一个多位特征空间的坐标，首先在特征空间中选择k个点作为类中心，这些中心是促使剩余的案例落入特征空间的催化剂，计算距离，如欧式距离。

数据准备，对于缺失值的处理：
* 一种简单的方法是排除具有缺失值的记录；
* 像性别这样的变量，另一种方法是将缺失值作为一个单独的类别。进行虚拟编码，将名义变量如性别，转换成可以用来距离计算的数值形式变量。
* 像年龄这样的变量，我们使用查补法来根据可能的真实值的猜测来填补缺失值，一般根据相应值的平均水平。 

可以使用stats包中的kmeans()函数来训练模型，aggregate()函数可以为数据的子组计算统计量。

1. 使用距离来分配和更新类，将初始的类中心转移到一个新的位置，成为质心；
2. 选择适当的聚类数，肘部法(elbow method)用来度量对于不同的k值，类内部的同质性是如何变化的。
