---
layout: post
title: 读文笔记A-Fast-RCNN        
category: blog
description: 阅读 A-Fast-RCNN:Hard Positive Generation via Adversary for Object Detection 笔记              
---


声明：本博客欢迎转发，但请保留原作者信息!      
作者：高伟毅    
博客：[https://gwyve.github.io/](https://gwyve.github.io/)    
微博：[http://weibo.com/u/3225437531/](http://weibo.com/u/3225437531/)    
  
## 引言    

这篇论文是在Object Detection上使用GAN的一篇论文，在GAN这么火的时期，出现这么一篇也是理所应当，同时，也是大势所趋。重要的是，中了CVPR2017。       
                                      

## 发表位置  

- CVPR2017
- [PDF arXiv：1704.03414.pdf](https://arxiv.org/pdf/1704.03414.pdf)          
- [代码托管](https://github.com/xiaolonw/adversarial-frcnn)                      

## 研究趋势回顾

作者在相关工作中回顾了一下目前Object Detection主要趋势：

1. 更换Object Detection基础网络。（更深的网络不仅要在classification上有好的表现，在object Detection上也要有）                        
2. 使用上下文语义、其他任务和其他的机制来代表objcet。（有文献是使用Segmentation任务来提升objcet Detection）                          
3. 通过挖掘数据本身来提升

本文就是按照GAN的做法，沿着第三条路走的。


## 问题引入

沿着上面提到的第三个想法，作者，针对occulusions 和 deformations，通过对抗网络进行相关数据生成。

![od](/images/blog/2017-5-17/od.png)

 
          
## Object Detection 的对抗学习

### Object Detector的损失

![obj_loss](/images/blog/2017-5-17/obj_loss.png)

### 对抗网的损失

![a_loss](/images/blog/2017-5-17/a_loss.png)

这里的这个X是从图片上计算生成，这个损失跟detector的损失是相反数的关系。这就是，如果生成的特征容易被detector识别，那么损失就大；如果不易被Detector识别，那么损失就小。（典型对抗网络的思维）

## 对抗网络的设计

主要针对occlusion、deformation出了两个设计：          
- ASDN：Adversarial Spatial Dropout Network (occlusion)                 
- ASTN：Adversarial Spatial Transformer Network (deformation)

### Adversarial Spatial Dropout for Occlusion

![ASDN](/images/blog/2017-5-17/ASDN.png)

使用标准的FRCN，对抗网络与FRCN共享Conv和RoI-pooling；但是对抗网络与FRCN没有参数共享。

分阶段训练网络：           
1. 训练 FRCN without ASDN。                
2. 训练 ASDN without FRCN          
3. 联合训练

针对ASDN的初始化：把d x d的特征向量分成d/3 x d/3个滑动的窗口。对每一个滑动窗口都进行一次 drop out，并把这些向量输入到classification layer；在d/3 x d/3个向量中，选择一个classification loss最高的一个向量生成d x d个mask。根据这mask与向量再做运算，计算损失。

![a_int_loss](/images/blog/2017-5-17/a_int_loss.png)

## Adversarial Spatial Transformer Network

这个网络是建立在Spatial Transformer Network<sup>[1]</sup>基础之上的。这里网络ASTN是使用三个全连接层，并且主要注重图像的角度的转变。

实现细节：这里限制了旋转的角度变化，否则很容易就是上下翻转的那种。并且，把所有向量按照通道分为了四组，每组都有不同的旋转角度。

## 实验

![result](/images/blog/2017-5-17/result.png)

必然是效果好

按类分析：这里 ASTN 与 ASDN 有类似的结果。针对ASTN还好理解，这个ASDN就有点不知道为啥了


![category](/images/blog/2017-5-17/category.png)


## 结论                            

这个文章的立足点就是数据集中有long-tail分布，这就有一些数据很少见以至于难以被分类，对抗网就算是增强了这类数据的影响，从而提高了识别率。      

   

## 个人想法

在读这篇文章之前，有两篇文章最好了解一下：                  
1. Action recognition using visual attention<sup>[2]</sup>
2. Spatial transformer networks<sup>[1]</sup>

这篇文章很大程度上，是把这两篇文章的目的反过来，然后使用GAN的思路来解决这个问题，这个思路很妙～


## 注释           
[1] M. Jaderberg, K. Simonyan, A. Zisserman, and K. Kavukcuoglu. Spatial transformer networks. In NIPS, 2015.                      
[2] S. Sharma, R. Kiros, and R. Salakhutdinov. Action recognition using visual attention. In CoRR, 2016.


                          
