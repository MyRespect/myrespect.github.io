---
layout: post
title:  "Machine Learning with R (6)"
categories: Machine_Learning 
tags:  Evaluation R
--- 

* content
{:toc}  

Today I am going to introduce some methods of evaluation models. I will provide you with some performance measures that reasonably reflect a model's ability to predict or forecast unseen data. And we will see how to use R to apply those useful measures and methods to the predictive models.  




#### **Evaluations**

**Measuring performance for classification:**

为什么预测准确度不足以度量性能？ 主要原因是类不平衡问题，当数据中有很大一部分记录属于同一类时就会产生问题。研究内部预测概率对评估模型的性能非常有用。在R中可以使用probability, posterior 等得到实际的预测概率，具体的参数使用方法在介绍每个模型中的语法中。 

然后介绍一个混淆矩阵，可以使用gmodels添加包中的CrossTable()函数。除了准确度之外，其他的常见性能评价指标有哪些呢？让我们来看看分类和回归训练添加包caret中包含的一些函数，其中confusionMatrix()是另一个创建混淆矩阵的函数。 

1. Kappa统计量，最大值是1，说明预测值和真实值之间的一致程度。在可视化分类数据添加包 (Visualizing Categorical Data,vcd)中也包含了Kappa()函数；评定者可信度(Inter-Rater Realizbility,irr)添加包中的kappa2()函数可以直接使用数据框中的预测值向量和实际分类向量计算Kappa值。注意，R中内置的kappa()函数与此处介绍的Kappa统计量无关。

2. 灵敏度，特异性，精确度，回归率，F度量。可以用caret包中的sensitivity(),specificity(),posPredValue()等函数计算。其中灵敏度和回归率的计算公式是一样的，TP/(TP+FN); 特异性=TN/(TN+FP); 精确度=TP/(TP+FP)。

性能权衡的可视化，可以使用ROCR包。使用ROCR创建可视化图形需要两个数据向量，第一个是包含预测的类别值，第二个是包含阳性类别的估计概率。 ROC(Receiver Operating Characteristic)曲线常用来检查在找出真阳性和避免假阳性之间的权衡。如何来绘制它呢？举个例子：

```
#生成预测对象 
pred <- prediction(predictions=sms_results$prob_spam, labels=sms_results$actual_type)

#构建性能对象
pref <- performance(pred, measure="tpr",x.measure="fpr")

#画图
plot(pref,main="ROC curve for SMS spam filter", col="blue",lwd=3)

#使用ROC曲线下面积来度量距离完美分类器的接近程度
pref.auc<-performance(pred, measure="auc")
```

**评估未来数据的性能**

我们常用的几种做法是:

1. 数据集分为 训练集，验证集，测试集。使用分层随机抽样可以避免不同类别数量过大或过小的问题。

2. k折交叉验证，将数据分为k份，机器学习模型每次使用1折作为评估，k-1折用来训练模型，重复k次，然后取所有折的平均性能指标。可以使用carcet添加包中的createFolds()创建交叉验证数据集。举个例子：

```
folds<-createFolds(credit$default,k=10)
cv_results<-lapply(folds,function(x){
	credit_train<-credit[x,]
	credit_test<-credit[-x,]
	credit_model<-C5.0(default~., data=credit_train)
	credit_pred<-predict(credit_model, credit_test)
	credit_actual<-credit_test$default
	kappa<-kappa2(data.frame(credit_actual, credit_pred))$value
	return(kappa)
})
```
