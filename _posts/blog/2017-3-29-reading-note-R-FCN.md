---
layout: post
title: 读文笔记R-FCN        
category: blog
description: 阅读 R-FCN： Object Detection via Region-based Fully Convolutional Networks 笔记              
---


声明：本博客欢迎转发，但请保留原作者信息!      
作者：高伟毅    
博客：[https://gwyve.github.io/](https://gwyve.github.io/)    
微博：[http://weibo.com/u/3225437531/](http://weibo.com/u/3225437531/)    
  
## 引言    

这个是使用残差网络的在object Detection上的论文，读这篇论文让我感觉最厉害的地方在于作者阐明了一个整个图片共享网络和RoI范围的卷积的关系，以及作者在各种用词和解释上，确实值得学习。
           
这篇论文在NIPS刚开的时候就打印了，当时因为看到了“Region-based”，所以就没有看，后来觉得NIPS上的应该看看～

## 发表位置  

- [NIPS 2016](https://nips.cc/Conferences/2016/AcceptedPapers)
- [PDF arXiv：1605.06409.pdf](https://arxiv.org/pdf/1605.06409.pdf)          
- [代码托管](https://github.com/daijifeng001/R-FCN)                 

   

## 问题引入

作者开篇描述了这样一个问题，在object Detection上的深度网络，被最后一个RoI池化层分为了两部分：1、共享计算的全卷积层；2、RoI范围的子网络，不共享计算。这一现象是因为object Detection的方法都是根据图像分类的方法（AlexNet、VGG）改的，这些图像分类的方法就是这样的结构，所以，object Detection也是这样的方法。

但是，新出的googlenet、残差网络本身就是全卷积的，如果按照之前的添加一个RoI池化层的话，效果并不好。为此ResNet，添加了一个不能共享计算的RPN，这样就慢了很多。

作者把这个不自然的设计归为，增强图像分类的平移不变形和尊重目标检测的平移可变性（这个，我实在是翻译不出来，原文：We argue that the aforementioned unnatural design is caused by a dilemma of increasing translation invariance for image classification vs. respecting translation variance for object detection. ）。图像分类对object平移不敏感，目标检测敏感。 

下图说的R-FCN中全部都是共享计算

![shared_conv.png](/images/blog/2017-3-29/shared_conv.png)


## R-FCN核心思想   
           
### pipeline

只用基于推荐区域的全卷积网络。通过把从共享卷积（ResNet）提取的特征和RPN推荐上来bbox的位置，通过投票（Vote）来确定这个bbox是否可用，以及对bbox进行微调。             

![pipeline.png](/images/blog/2017-3-29/pipeline.png)

### 主要结构

这个的主要结构就是ResNet，把ResNet-101的最后的fc改成1x1x1024的conv，然后在使用k<sub>2</sub>(C+1)通道conv。

具体网络（100层之后的）见图：

![R-FCN.png](/images/blog/2017-3-29/R-FCN.png)

注：这个是使用[NetScope](http://ethereon.github.io/netscope/quickstart.html)，根据[py-R-FCN](https://github.com/Orpine/py-R-FCN)中的[models/pascal_voc/ResNet-101/rfcn_end2end/train_agnostic.prototxt](https://raw.githubusercontent.com/Orpine/py-R-FCN/master/models/pascal_voc/ResNet-101/rfcn_end2end/train_agnostic.prototxt)绘制的。


### Position-sensitive score maps & Position-sensitive RoI pooling

![key_idea.png](/images/blog/2017-3-29/key_idea.png)

从Feature map出来一个k<sub>2</sub>(C+1)的卷积特征。对于每一个RoI，在预测分类的时候，就变成（C+1）维的向量，进行softmax计算。

除了k<sub>2</sub>(C+1)的卷积向量之外，还生成一个4k<sub>2</sub>维的向量，用于定位微调：把4k<sub>2</sub>合成一个4k的向量，用来表示bbox。

## 训练

### loss

这里用的box regression的loss跟Fast R-CNN一样的，RPN跟Faster R-CNN一样的。

### OHEM

这个用的[另一篇](https://arxiv.org/pdf/1604.03540.pdf)论文的东西，我还不是太懂，待读了这篇论文再整理。

### À trous and stride.

这个里面借鉴了语义分割的东西，我也不是特别明白。先放在这里。

## 实验与分析

### 与Faster R—CNN对比

mAP掉一点，速度提升得多

![result.png](/images/blog/2017-3-29/result.png)

### 网络深度影响

![depth_impact.png](/images/blog/2017-3-29/depth_impact.png)

### 推荐方法影响

![proposal_impact.png](/images/blog/2017-3-29/proposal_impact.png)

### coco

![coco.png](/images/blog/2017-3-29/coco.png)

## 结论                            

R-FCN很牛，之后要用在语义分割上
   

## 个人想法

我认为这个论文讲的重要的地方就是把这个识别和检测共享计算的事儿说了，明白了，把两个网络合在一起的地方越多，正确率就越高的问题。

这个是在ResNet上做的事情，他没有说在VGG效果如何，他效果提升，究竟是在ResNet的识别能力，还是他提出位置敏感的问题，无从知晓。
                        