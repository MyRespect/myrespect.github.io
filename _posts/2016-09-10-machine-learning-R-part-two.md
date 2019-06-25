---
layout: post
title:  "Machine Learning with R (2)"
categories: Machine_Learning
tags:  KNN Naive_Bayes Decision_Tree R
--- 

* content
{:toc}  

In this blog, I will introduce three main marchine learning models and its realizations in R, K-Nearest Neighbors, Naive Bayes and Decision tree. I am not going to go into those methods seriously now, but I hope I will have a better understanding of those methods after I watching Android Ng's online course and reading Zhihua Zhou's book, and then I will replenish the content. 




#### **KNN**

KNN是懒惰的，基于示例的学习算法，没有需要学习的数据参数。K-Nearest Neighbors 算法将特征处理为一个多维特征空间内的坐标，实例间的相似性用距离函数来衡量，如欧氏距离。用K来指定近邻数，然后进行投票表决再归类。K的选择非常重要，选择一个大的K会减少噪声数据对模型的影响，但是会使分类器产生偏差，反之也不好。常见做法，设置K等于训练集中案例数量的平方根；或者基于各种测试数据集测试多个K值，选择最佳的。      

使用数据前，我们一般将数据“归一化”处理，对数值特征有如 min-max normalization, z-score standardization (可以由scale()函数实现)方法；对于名义数据，为了计算名义特征之间的距离，我们需要将他们转化成数值型的格式，典型方案“哑变量编码”。其中1表示一个类别，0表示其他类别。一个具有n个类别的名义特征可以对特征的(n-1)个水平创建二元指示变量进行编码。

记录R实现过程中的几个函数，lapply() 函数可以输入一个列表，然后把一个函数应用到列表的每一个元素。as.data.frame() 把列表转换为数据框。 `wbcd_n <- as.data.frame(lapply(wbcd[2:31]),normalize)`, 注意mormalize为用户自定义函数。  

 class添加包中的knn()函数提供了一个标准的KNN算法的实现。 `p<-knn(train,test, class,k)` 参数含义分别是训练数据的数据框，测试数据的数据框，训练数据的分类类别的因子向量，最近邻的数目。    

#### **Naive Bayes**

基于贝叶斯方法的分类器是利用训练数据并根据特征的取值来计算每个类别被观察到的概率。朴素贝叶斯假设数据集的算有特征都具有相同的重要性和独立性，所以比较naive嘛。 拉普拉斯估计(Lapace estimator) 本质上是给频率表上的每个计数都加上一个较小的数，保证了每个特征发生的概率是非零的。     

词云，以前经常见却不知道怎么做出来的，可以用 wordcloud包实现，如`wordcloud(spam$text,max.words=40,scale=c(3,0.5))`  

文本挖掘添加包tm，`sms_corpus <- Corpus(VectorSource(sms_raw\$text))`, 函数Corpus()创建了一个R对象来存储文本文档，VectorSource()指示函数Corpus()使用向量sms_train\$text的信息。然后我们把所有字母变成小写字母，去除所有数字，去除停等词，去除标点符号，去除额外空格:    

    corpus_clean <- tm_map(sms_corpus,tolower)    
    corpus_clean <- tm_map(corpus_clean, removeNumbers)   
    corpus_clean <- tm_map(corpus_clean, removeWords,stopwords())  
    corpus_clean <- tm_map(corpus_clean, removePunctuation)  
    corpus_clean <- tm_map(corpus_clean, stripWhitespace)
   

查找频繁出现的单词可以用findFreqTerm()函数，该函数输入一个文档-单词矩阵，并返回一个字符向量，该向量中包含出现次数不少于制定次数的单词，用来剔除过滤低于某个频次的单词，减少数据特征。 函数DocumentTermMatrix()将一个语料库作为输入，创建一个稀疏矩阵，行表示一篇文档，列表示单词，每个单元存储一个数字，表示由列标识的单词在由行标识的文档中出现的次数。   


值得注意的是，tm包使用应该是很广泛的，而且内容也在不断更新，有些函数用法可能会改变设置被删除，使用函数时一定要搞清楚函数要求的数据类型是什么，当前数据类型是什么，学会阅读帮助文档和网上求教。  

定义一个函数，将计数转换成因子：

```
covert_counts <- function(x){
	x <- ifelse(x>0,1,0)
	x <- factor(x,levels=c(0,1),labels=c("No","Yes"))
	return (x)
}
```

应用e1071添加包中的函数naiveBayes(), `m<-naiveBayes(train, class, lapace=0)`, train 表示数据框或者训练数据矩阵，class 表示训练数据每一行分类的类别因子向量,laplace控制拉普拉斯估计值；使用predict()函数进行预测,用 CrossTable进行评估,然后要想办法提升模型性能。

```
sms_classifier <- naiveBayes(sms_train,sms_raw_train$type)
sms_test_pred <- predict(sms_classifier, sms_test)
CrossTable(sms_test_pred, sms_raw_test$type, prop.chisq=FALSE, prop.t=FALSE,dnn=c('predicted','actual'))
```


#### **Decision tree**

决策树像是流程图，从代表整个数据集的根节点开始，算法选择最能预测目标类的特征，然后，这些案例将被划分到这一特征的不同值的组中，这一决定形成了第一组树枝，然后该算法集训分而治之其他节点，每次选择最佳的候选特征，知道达到停止的标准。决策树算法使用轴平行分割，每次只考虑一个特征。      

我们可以使用runif()函数，默认情况下产生[0,1]之间的随机树序列，order()函数返回一个数值向量，该向量记录了排序后的数载原来序列中的位置

C5.0算法已经成为生成决策树的行业标准，容易解释和部署。`m <- C5.0(train,class,trials=1,costs=NULL)`, train一个包含训练数据的数据框，class包含训练数据每一行的分类类别的因子向量，costs用于给出课中类型错误相对应的成本，trial用于控制自助法循环的次数。
`p <- predict(m,test,type="class")`, m表示由函数C5.0训练的一个模型，test表示一个包含测试数据的数据框，type取值为class表示预测是最可能的类别，type取值为prob表示得到原始的预测概率值。

分类规则算法是根据示例的相应特征规则进行类别划分。可以使用RWeka添加包中的函数OneR()或者JRip()，格式相似，`m <- JRip(class~predictors,data=mydata)`,class是mydata数据框中需要预测的那一列，predictors是一个R公式，用来指定mydata数据框中用来预测的特征规则组合，data为包含class和predictions所要求的数据框。