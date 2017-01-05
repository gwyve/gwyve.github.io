---
layout: post
title: 读文笔记Faster R-CNN
category: blog
description: 阅读 Faster R-CNN Towards Real-Time Object Detection with Region Proposal Networks笔记
---


声明：本博客欢迎转发，但请保留原作者信息!      
作者：高伟毅    
博客：[https://gwyve.github.io/](https://gwyve.github.io/)    
微博：[http://weibo.com/u/3225437531/](http://weibo.com/u/3225437531/)    
  
## 引言    
这是在Fast R-CNN之后的一篇文章，推荐阅读Fast R-CNN之后再阅读这篇文章。


## 发表位置  
- NIPS 2015
- [PDF.v3](https://arxiv.org/pdf/1506.01497v3.pdf)

## 问题引入
整篇文章，只是说明了一件事，就是之前的fast R-CNN中，在进入RoI pooling层之前的推荐区域是通过SS（Selective Search）产生的。SS在产生推荐区域的这个过程花费时间太多。作者引入了RPN（Region Proposal Networks）这个模型方法来替代SS，节省时间。也可以说，Faster R-CNN = RPN + FRCN。     

## Faster R-CNN结构和训练过程   

1. 整张图片输入Conv层生成Feature map           
2. 把Feature Map输入RPN生成一个要推荐区域的位置坐标          
3. 根据步骤2推荐出的区域坐标，把Feature map中相应的区域输入RoI pooling层，生成固定长度的向量。     
4. 第3步产生的固定长度向量输入全连接（fc）层，fc层分成两个sibling output layer：一个生成K+1个类的softmax，另一个生成bounding-position的位置。（与我之前说的FRCN的第四步一样的）        

![architecture](/images/blog/2017-1-5/architecture.png)

这个意思就是在Fast R-CNN之前不再使用SS，而加上RPN生成Proposals。

我根据作者的想法，我画了一个流程，不过有个地方没有没有画明白。

![architecture_ve](/images/blog/2017-1-5/architecture_ve.png)

__这里，Proposal并不是直接输入到RoI Pooling层，而是根据这个Proposal产生的坐标，或者说是个框框，在Feature Map找出相应的区域，把这个Feature Map中的这一块输入到RoI Pooling层。__


## RPN：Region Proposal Networks

### RPN 结构
![RPN_architecture](/images/blog/2017-1-5/RPN_architecture.png)

在Feature map上有一个n×n的滑窗（文章中用n=3），这个滑窗是一个n×n的convolutional layer，如果用
ZF模型，则通过该滑窗生成256维的向量。这个向量分别通过两个1×1的convolutional layer，生成一个是否有object的分数和这个框框的位置。生成的这两个量并非跟真正的作对比，而是跟一个被称为Anchors的东西作对比。

### Anchors
在文章中，作者是这么说的“The k proposals are parameterizedrelative to k reference boxes, which we call anchors.”作者还提到是不同的scale和长宽比（本文中用到了分别是（128<sup>2</sup>,256<sup>2</sup>,512<sup>2</sup>）（2:1,1:1,2:1），一共是九种anchors）。    

这个Anchor是在滑窗当前位置下，以当前窗为中心，变动的一个区域。这个Anchor具体如何用，在接下来的损失函数讲。

### 损失函数
![loss_fucntion](/images/blog/2017-1-5/loss_function1.png)     
具体过程：     
1. 给anchor标一个分数： 与真实的框框IoU最高的标为正数；与真实的框框IoU超过0.7的标为正数；与真实框框IoU小于0.3的标为负数。
2. Pi<sup>*</sup>是anchor在步骤1中标的值。
3. 这个t表示的是预测出的框框位置，t<sup>*</sup>表示Anchor的位置，具体如图                  
![loss_fucntion2](/images/blog/2017-1-5/loss_function2.png)      
__到这里我才能明白Anchor是干啥用的，就是用来在真实框框跟预测框框之间的一个东西，仔细想想叫做“锚”这个词还是有道理的。__   

### Faster R-CNN 训练过程

4步交替训练
1. 不管Fast R-CNN，只是训练RPN。            
2. 不改变RPN，只是使用RPN输入的框框训练Fast R-CNN。            
3. 用Fast R-CNN初始化RPN，不改变Convolutional layers，只是微调RPN。                 
4. 不改变Convolutional layers，微调Fast R-CNN。          

## 实验、各种测评
### mAP
![experiment1](/images/blog/2017-1-5/experiment1.png)     
这个是PASCAL VOC 2007的结果，在具体实现的时候，使用了RPN推荐的区域里面排名前300的区域（这样做是否有道理，后面会实验证明）。

ablation experiments的第一个为了证明，RPN与fast R-CNN共享卷积层有用。第二个为了证明，使用推荐区域的前多少个合适。第三、四为了证明cls、reg两层有用。

### 时间
![experiment_time](/images/blog/2017-1-5/experiment2.png)

可以看出，在conv层的耗时基本差不多，在选择推荐区域的时候RPN比SS更省时间，在“Region-wise”上Faster R-CNN更省时间（只输入推荐区域的一部分，数据少了，时间必然少了）。总之，整个目标发现系统使用的时间少了。

### Anchor && λ
就是选择具体参数的问题           
![experiment3](/images/blog/2017-1-5/experiment3.png)    
### Analysis of Recall-to-IoU
这一部分也说明了RPN可以使用推荐的top-300这个方法。作者原话“ The plots show that the RPN method behaves gracefully when the number of proposals drops from 2000 to 300. This explains why the RPN has a good ultimate detection mAP when using as few as 300 proposals.”
### One-Stage Detection vs. Two-Stage Proposal + Detection
![one vs two](/images/blog/2017-1-5/experiment4.png)   
这个主要证明了，RPN只是判定区域里面有没有对象，没有判断里面的对象的分类，这件事是有作用的。  

## 个人总结
我觉得这个Faster R-CNN主要贡献就是RPN     
### 提升准确率的
- RPN和Fast R-CNN共享Conv layers    
- Anchor的存在，尺度上不必有变化    

### 提升速度的
- RPN没有之前的那种滑动窗口或者过滤器，Anchor的使用。   
- RPN没有像SS推荐的那么多的区域。   
- RPN推荐的区域，可以使用排名靠前的一部分，而不是全部。    

## 个人想法
有的几个地方，还是不太懂，比如在第一个alation experiments的时候，为什么训练的时候用SS；One-Stage Detection vs Two-Stage Proposal + Detection这一部分，为什么这么设计实验？

我觉得这些问题可以等等看的论文多了，抑或看看Faster R-CNN的代码了，会有更好的理解。

