---
layout: post
title:  "Machine Learning with R (1)"
categories: Machine_Learning
tags:  R Data_Management
--- 

* content
{:toc}

In the following series of blogs, I will make some notes about the book "MACHINE LEARNING WITH R" written by Brett Lantz. In this blog, I will start with the introduction of R and Data Management.      





1. Machine learning algorithms are virtually a prerequisite for data mining but the opposite is not true.   

2. Trying to model the noise in data is the basis of a problem called overfitting.  

3. Collecting data -> Exploring and preparing the data -> Trainga model on the data -> Evaluating model performance -> Improving model performance.  


##### **Data Structure in R**

向量，一个向量包含的所有元素必须是一样的类型；NA 表示缺失值； NULL 表示没有任何值；其他类型包括 整型，数值型，字符型，逻辑型（TRUE, FALSE）；建立向量举例： subject_name <- c("Jason","Tom","Bob");  

列表, 列表类似向量，可以理解为结构体，允许收集不同类型的值；列表的创建使用 list() 函数；  

数据框， 结合了向量和列表的特点， 列代表的是属性，行代表的是案例， 使用data.frame() 函数； 

因子， 用类别数值代表特征的属性称为名义属性，R中使用因子表示这种属性数据，应用 factor() 函数；levels 指定数据可能取到的所有类别；  

矩阵，只能包含单一类型的数据； 建立矩阵举例： m <- matrix(c('a','b','c','d'), nrow=2);  

##### **Exploring the Structure of Data** 

I just noted some main key functions in R for exploring the structure of Data, and the most important things are learning to read the manual and using Google.  
 
1. 函数 str() 提供了一个显示数据框结构的方法；  

2. 函数 summary() 提供了汇总统计量，min, 1st Qu, Median, Mean, 3rd Qu, Max;    

3.  range() 返回最大值最小值， diff() 求差值， IQR 求四分位距； quantile() 求任意分位数； seq() 产生间距大小的相同向量；  

4. table() 产生一个分类变量的统计一元表；检验变量之间的关系用双向交叉表，CrossTable() 观察一个变量的值是如何随另一个的变化而变化的;

5. 箱图，boxplot(); 直方图，hist(); 散点图， plot(); 标准差， var(); 方差， sd().
