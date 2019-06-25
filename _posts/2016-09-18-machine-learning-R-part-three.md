---
layout: post
title:  "Machine Learning with R (3)"
categories: Machine_Learning
tags:  Regression R
--- 

* content
{:toc}  

As time goes by, I go to the part about regression. It goes without saying that the method is a crucial part in machine learning. In the meanwhile, I am watching Andrew Ng's Machine Learning online course, which also covers regression. I read the book like facing a black box but very useful. Andrew's vedio tells me why, and I can look into the methods more deeply.




#### **Regression**

我们可以回归方法预测数值型数据，主要关注因变量和自变量之间的关系，比较典型的有线性回归，逻辑回归和泊松回归。线性回归假设因变量的分布为正态分布，但这种分布是不理想的。它的作用主要有：
 
1. 用来估计一种方法对结果的影响和推断未来； 
2. 用于假设检验，包括数据能否表明原假设更可能是真还是假； 
3. 通过对关系强度和一致性的估计，用于评估结果是否是由于偶然性造成的。 

然后，照理说我们就应该介绍最小二乘法估计。这里呢，我先简单记录几个统计学名词：

对于y=a+bx, b=cov(x,y)/var(x,y)。 **残差**，y的预测值与y的真实值之间的垂直距离。**协方差**，用cov()函数计算每个数据点中x与其均值的偏差乘以y与其均值的偏差的乘积之和。  **相关系数**，用cor()函数计算，p=cov(x,y)/qx qy。 对于相关系数与相关性的理解要结合实际，一种是认为相关系数大于0.5便是强相关。 

在多元线性回归中，我们可以用矩阵进行计算系数向量：

```
# creating a simple multiple regression function
reg <- function(y, x) {
  x <- as.matrix(x)
  x <- cbind(Intercept = 1, x)
  solve(t(x) %*% x) %*% t(x) %*% y
}
```
as.matrix()将数据强制转换成矩阵形式， cbind() 用来将额外的一列添加到矩阵x中。题外话，通过比较，我发现了matlab在进行矩阵运算的方便之处。

**探索和准备数据:** 

使用cor()创建相关系数矩阵来探索特征之间的关系，使用pairs()创建散点矩阵来可视化特征之间的关系，改进后的散点矩阵，可以用psych包中的pairs.panels()函数创建。

```
# exploring relationships among features: correlation matrix
cor(insurance[c("age", "bmi", "children", "charges")])

# visualing relationships among features: scatterplot matrix
pairs(insurance[c("age", "bmi", "children", "charges")])
```
**基于数据训练模型:** 

使用lm()函数对数据拟合一个线性回归模型：

`
model <- lm(distress_ct ~ temperature + pressure + launch_id, data = launch)
`

**评估模型性能:**

summary()命令来分析预测所存储的回归模型：

`summary(model)`
 
**提高模型性能:**
 
1. 提高模型性能，添加非线性关系, 增加自变量的次数。 
2. 如果一个特征的影响是不累积的，而是当特征的取值达到一个给定的阈值后才产生影响，将一个数值型变量转换为一个二进制指标。
3. 如果某些特征之间相互作用，对因变量有综合的影响，那么加入相互作用的影响。
4. 全部放在一起建立一个改进的回归模型。

```
insurance$age2 <- insurance$age^2

# add an indicator for BMI >= 30
insurance$bmi30 <- ifelse(insurance$bmi >= 30, 1, 0)

# create final model
ins_model2 <- lm(charges ~ age + age2 + children + bmi + sex +                  bmi30*smoker + region, data = insurance)
```

**回归树(Regression tree)**，其并没有使用线性回归的方法，而是居于到达叶节点的案例的平均值做预测。 

**模型树(Model tree)**，在每个叶节点，根据到达该节点的案例建立多元线性回归模型。

将回归加入决策树，用于数值预测的巨册书的建立方式和用于分类的决策树的建立方式大致相同，一个常见的分割标准称为标准偏差检索，指从原始值的标准差减去分割后加权的标准差的减少量，减少量越多，一致性越大，都采用此次分割。

使用rpart()添加包中的函数rpart()可以实现分类回归决策树。
使用rpart.plot()添加包中的rpart.plot()函数可以实现可视化决策树。
使用平均绝对误差评估模型性能：

```
# function to calculate the mean absolute error
MAE <- function(actual, predicted) {
  mean(abs(actual - predicted))  
}
```
 
 为了提高模型性能， 我们尝试构建一棵模型树，使用RWeka()添加包的M5P()函数：
 

```
library(RWeka)
m.m5p <- M5P(quality ~ ., data = wine_train)

# display the tree
m.m5p

# get a summary of the model's performance
summary(m.m5p)

# generate predictions for the model
p.m5p <- predict(m.m5p, wine_test)

# summary statistics about the predictions
summary(p.m5p)

# correlation between the predicted and true values
cor(p.m5p, wine_test$quality)

# mean absolute error of predicted and true values
# (uses a custom function defined above)
MAE(wine_test$quality, p.m5p)
```

