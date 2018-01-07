---
layout: post
title: 读文笔记 EigenROP        
category: blog
description: 阅读 Detecting ROP with Statistical Learning of Program Characteristics  笔记              
---


声明：本博客欢迎转发，但请保留原作者信息!      
作者：高伟毅    
博客：[https://gwyve.github.io/](https://gwyve.github.io/)    
微博：[http://weibo.com/u/3225437531/](http://weibo.com/u/3225437531/)    
  
## 引言    

之前的博客笔记都是在做Object Detection相关的论文笔记，这篇论文与Object Detection没有什么关系，因为自己对这个方向的了解仅有几篇中文综述的阅读。

这篇论文主要是使用统计学方法来处理ROP检测的问题。论文中提到Eigen这个词，我不认识，后来查才知道，他与线性代数中特征矩阵(eigenvector)和特征值(eigenvalue)有关。
                                      

## 发表位置  

- CODASAY‘17
- [ACM](https://dl.acm.org/citation.cfm?id=3029812)          
                     

## 研究背景介绍

作者讲到，目前ROP检测的方法主要有基于签名（signature-based）的检测和基于异常(anomaly-based)的检测。基于异常的检测主要是通过分析数据分布来识别正常行为和攻击行为。这些数据主要:
- __architectural characteristics__, which are dependent on the instruction set architecture (ISA), such as the number of load and store instructions retired. 
- __microarchitectural characteristics__, meaning characteristics that depend on the underlying
microarchitecture configurations, such as branches misprediction rate and cache misses. 

## Overview          

![overview](/images/blog/2018-1-6/EigenROP.png)

上图是整个的流程图，具体分为:                    
1. 一个程序，每隔着N个指令(instructions retired)生成一个镜像（snapshot），每个镜像生成d维的一个向量。                                  
2. 把这些矩阵映射到一个高维空间中。                         
3. 在学习阶段，EigenROP根据这些高维向量评估出一个方向，以及该方向的距离密度。                         
4. 在检测阶段，EigenROP计算输入值与学习阶段评估出方向的距离。                         

## 特征工程

作者根据之前工作<sub>[1][2][3]</sub>，测量了15个特征，将其分为了三类：Architectural [A], Microarchitecture-Independent[I], and Microarchitectural [M]。再根据Fisher Score，从15个特征选出10个特征。

15个特征如下：              

![overview](/images/blog/2018-1-6/characteristics.png)





### 学习过程

这一块用到了好多数学专有名词，看起来好吃力。

这里主要使用PCA核变换，把特征矩阵映射到高维空间中。并以clean之间距离最小为目标，学习变换过程的参数，并最终得到一个方向。

如果输入的测量值与训练过程得到方向距离超过阈值，就判别是异常(anomaly)。

## 实现 

### 特征收集

使用MICA<sub>[4]</sub>，一种PIN<sub>[5]</sub>工具，来收集特征。

### 机器学习

sklearn

## 数据

ROP数据(负样本):         
- OSVDB-ID:87289                 
- OSVDB-ID:72644                

数据具体信息：       
![overview](/images/blog/2018-1-6/data.png)            

__文章没有提到正负样本具体数量__

## 效果

这个具体数值有点吓人：

AUC81%,TPR80%,FPR0.8%

## 其他

1. 作者在文中提到，EigenROP是建立在程序特征统计学上集中的假设上的，如果分散的话，false positive rate就变高。                  
2. 文中提到EigenROP对ROP的变型也有效果。（个人感觉，作者使用数据的分布很重要）。                   
3. 文中使用三个理由指出，攻击者使用ROP的时候难以做到与正常程序出现相同的分布。            

   

## 个人想法

1. 这个是我之前没有接触过的领域，有部分涉及到硬件、指令的东西并没有了解；读起来比较吃力。         
2. 这篇论文就是把统计学方法应用在这个领域的论文，我觉得这个难点主要有：如何获得这些特征；数据量偏少；正负样本比例的问题。      
3. 作者对于这篇文章中使用数据的样本分布，没有说太多，怀疑他正样本都是类似的内容，仅仅是怀疑。          

## 引用

[1] K. Hoste and L. Eeckhout. Comparing benchmarks using key microarch.-independent characteristics. In Workload Characterization. IEEE, 2006.                                     
[2] C. Malone, M. Zahran, and R. Karri. Are hardware performance counters a cost effective way for integrity checking of programs. In 6th ACM workshop on Scalable Trusted Computing, 2011.                                      
[3] A. Tang, S. Sethumadhavan, and S. J. Stolfo.
Unsupervised anomaly-based malware detection using hardware features. In RAID. Springer, 2014.                                                 
[4] K. Hoste and L. Eeckhout. Microarchitecture-independent workload characterization. IEEE Micro, 3, 2007.                   
[5] C.-K. Luk et al. Building customized program analysis tools with dynamic instrumentation. In PLDI, 2005.





                          