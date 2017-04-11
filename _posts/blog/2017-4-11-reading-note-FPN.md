---
layout: post
title: 读文笔记FPN         
category: blog
description: 阅读 Feature Pyramid Networks for Object Detection 笔记              
---


声明：本博客欢迎转发，但请保留原作者信息!      
作者：高伟毅    
博客：[https://gwyve.github.io/](https://gwyve.github.io/)    
微博：[http://weibo.com/u/3225437531/](http://weibo.com/u/3225437531/)    
  
## 引言    

这个还是rbg这个R-CNN开山鼻祖弄得东西，只不过是他从MicroSoft跳到Facebook之后发的东西，同时，还看到了ResNet的Kaiming.He。文章是一个朋友推荐的，内容读起来并不费劲。

## 发表位置  

- [PDF arXiv：1612.03144](https://arxiv.org/abs/1612.03144)          
               

   

## 问题引入

通过多个尺度上预测效果会好，可是，如果这些多个尺度的特征都相互没有关联的预测就会失去利用较高特征图的机会。所以，需要让这些不同尺度的特征之间，相互发生一定联系。不同尺度可以看做是一个金字塔，产生联系，就是在不同层之间产生联系。

![pyramid.png](/images/blog/2017-4-11/pyramid.png)


## FPN核心思想   

这个FPN有两个思想，但是，并不是一脉相承的关系，而是对比的关系。虽然两个都是FPN的思想，但是作者明确指出他的FPN就是Top-down and lateral connections。

### Bottom-up pathway

这个就是上图c的想法。这个实现起来比较简单。作者对这个想法的意义并没有做过多的解释，因为这个实现其实就是在主网络外再加上一个predict相关金字塔网络。

### Top-down and lateral connections

作者在这里指出高层的特征具有更强的语义（semantical）含义，虽然越高层就越粗糙。所以，从横向过来的底层的特征与高层的结合进行预测输入。

作者在这里并没有说明upsample的具体方法（作者说这个方法不是这个文章的重点，并指出有很多方法可以用）。但是，作者提到了，底层特征与高层特征结合之后需要对其进行一个卷积，目的是为了郊区upsample的混淆（aliasing）影响。 

![Top_down.png](/images/blog/2017-4-11/Top_down.png)          


## 实验

作者分别对RPN、Faster R-CNN 和Fast R-CNN做实验

这里RPN主要是针对不同金字塔层的特征进行不知类的预测位置；而Faster/Fast R-CNN都是根据已经知道框的位置，微调上预测。

800像素图片输入，8张GPU训练8个小时

数据集COCO

## 分析

![RPN.png](/images/blog/2017-4-11/RPN.png) 

### top-down有用

上图c和d的对比，说明top-down有用。d的bottom-up中large object好，作者推测是因为bottom-up的不同层之间的语义鸿沟大。

### latera connections 有用

上图c和e说明了此

### 金字塔每层都预测有用

f是下图的上面的一个

![topPredict.png](/images/blog/2017-4-11/topPredict.png) 

### Faster/Fast R-CNN的结果

![2result.png](/images/blog/2017-4-11/2result.png) 

### 针对其他牛的方法使用FPN的效果

![3result.png](/images/blog/2017-4-11/3result.png) 

### 在ResNet共享特征

![shareFeature.png](/images/blog/2017-4-11/shareFeature.png)

### 运行时间

- NVIDIA M40 GPU
- FPN-based Faster R-CNN Resnet-50   per image  0.20s
- FPN-based Faster R-CNN Resnet-101  per image  0.24s
- single-scale ResNet-50             per image  0.32s 

   

## 个人想法

这个FPN跟之前看的那个hyperNet有些类似的地方，hyperNet是特征提取过程进行多层之间关联，这个是在预测的时候进行关联。

现在大有不在VOC上玩的趋势，都开始看coco，coco很大，跑起来对硬件要求太高了。作者8张GPU，那简直就不是一般人能玩的来的，趁着这个民用级别的还能玩，赶紧弄，这是个关键。

他既然可以在predict的时候有关联，就是类似这个金字塔的网络，我就要在这个上面搞一下。
                        