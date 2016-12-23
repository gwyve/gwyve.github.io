---
layout: post
title: 读书笔记SPPnet
category: blog
description: 阅读Spatial Pyramid Pooling in Deep Convolutional Networks for Visual Recognition 笔记
---




声明：本博客欢迎转发，但请保留原作者信息!      
作者：高伟毅       
博客：[https://gwyve.github.io/](https://gwyve.github.io/)    
微博：[http://weibo.com/u/3225437531/](http://weibo.com/u/3225437531/)    


## 引言
最近在看R-CNN<sup>[1]</sup>、fast R-CNN<sup>[2]</sup>、faster R-CNN<sup>[3]</sup>这几篇文章，在fast和faster中都提到了SPPnet的相关东西，可是在cs231这么课程中偏偏没有提到这篇文章，而自己在月fast和faster的之后总觉得不爽，然后就读了一下这一篇文章。

## 发表位置
- [ECCV 2014](http://www.cvpapers.com/eccv2014.html)   
- [PDF](https://arxiv.org/pdf/1406.4729v4.pdf)

## 问题引入
这个文章是R-CNN出来之后的改进文章，在fast C-NN和faster C-NN之前。本文主要是发现了R-CNN的主要问题：R-CNN的输入需要固定的尺寸，针对不同尺寸的图片R-CNN采取crop和warp这两种方法（见图1）。而之所以必须输入固定尺寸的原因是fc层需要固定输入。所以作者就是希望在fc层之前添加另外一种结构来使整个网络结构在fc层之前达到固定输入结构。图2上是RCNN的结构，下是
SPPnet的结构。         
![crop-warp](/images/blog/2016-12-23/crop-warp.png)      
<center>图1</center>      
crop会导致无法输入整个目标，warp会导致图像失真     

## SPPnet
根据R-CNN中的问题，作者提出了
SPPnet的这一种结构，就是在conv之后使用空间金字塔池化层，具体实现是把cnn的最后一pooling层换成SPP。。整体结构如图2
![architecture](/images/blog/2016-12-23/architeture.png)     
<center>图2</center>

## SPP层
作者提出的主要是这个空间金字塔层，他就是依次把一张图分成1份、4份、16份，然后在每一份中提取一个256维的向量（本文用的max的方法）。使用spp之前，只是在固定份数下提取数据，spp是在不同层次使用max pooling。具体如图3     
![SPP-Layer](/images/blog/2016-12-23/SPP-layer.png)    
<center>图3</center>      
图片上分出来的每一份是使用一个滑动的窗口，如果需要输入的是a*a的，需要分成n*n份，那么步长就是a/n。

## 优势
- SPP可以生成固定大小的向量，以此适应fc层要求固定输入
- 多个pooling窗口，可以取出不同层次的特征
- 可以不再限制输入图片的尺寸
- __对于每张图片只需跑一遍CNN(这个在目标检索的时候提速明显)__

## 图像分类实验
实验做了不少，各种优势基本就是
- 可以从多个pooling取特征，就相当于从多种层次提取特征
- 可以使用多种尺寸的照片，降低错误率
- 输入图片不需要切割，这个画面会降低错误率       

## 目标检索    
R-CNN中，先在一种图片里面通过Selective Search选出n个推荐区域，每个区域跑一遍Conv层，这就要跑n遍conv层。在SPPnet中，只需要对一张图片跑一遍CNN，根据整张图片生成一个feature map，在这个feature map中使用Selective Search选出n个推荐区域，n个区域依次输入到SPP层和fc层。可以看出SPPnet对于一张图片只用跑一遍CNN。图4，图5说明了具体实验结果。    
![VOC2007-1](/images/blog/2016-12-23/VOC2007-1.png)         
<center>图4</center>           
![VOC2007-2](/images/blog/2016-12-23/VOC2007-2.png)             
<center>图5</center>        
图4是SPPnet、R-NN使用不同的模型的对比，SPPnet快了64倍，图5是使用同一模型时，SPPnet快了102倍。   


## 模型联合
文章最后写了一下两个模型一起用的可以提高正确率。他这里还说了，这个联合提高正确率跟Conv无关。       
![combination](/images/blog/2016-12-23/combination.png)    
<center>图6</center>


## 个人想法
这篇论文确实是一种启发性质的，随后的fast、faster都可以看到SPPnet的某些想法的影子。其实SPP层就Pooling的一种变形，使用了多种层次的想法。      
整篇文章，感觉真的严谨，考虑很多地方。主要表现在各种对比上，就是真的控制变量法。



## 相关
[1]R. Girshick, J. Donahue, T. Darrell, and J. Malik, “Rich feature
hierarchies for accurate object detection and semantic segmentation,”
in CVPR, 2014.    
[2]R. Girshick, "Fast R-CNN", ICCV, 2015      
[3] Shaoqing Ren Kaiming He Ross Girshick Jian Sun. Faster R-CNN: Towards Real-Time Object Detection with Region Proposal Networks.arXiv:1506.01497,2015               