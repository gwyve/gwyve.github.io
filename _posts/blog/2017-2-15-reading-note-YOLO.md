---
layout: post
title: 读文笔记YOLO        
category: blog
description: 阅读 You Only Look Once:Unified,Real-Time Object Detection 笔记
---


声明：本博客欢迎转发，但请保留原作者信息!      
作者：高伟毅    
博客：[https://gwyve.github.io/](https://gwyve.github.io/)    
微博：[http://weibo.com/u/3225437531/](http://weibo.com/u/3225437531/)    
  
## 引言    
这是在Faster R-CNN之后的一篇object detection的文章，他的想法跟R-CNN区别还有有的，联系也有，不过不大。


## 发表位置  
- CVPR 2016
- [PDF.v5 arXiv：1506.02640.pdf](https://arxiv.org/pdf/1506.02640.pdf)      
- [项目主页](http://pjreddie.com/darknet/yolov1/)    
- [代码托管](https://github.com/pjreddie/darknet)    

## 问题引入
这篇文章都是在按照题目说的“你只用看一次”，作者主要发现了目前art-of-state的object detection的方法都是先找到一个推荐区域，然后对这个区域进行确定是不是有object同时分类，然后再fine-tune推荐的算法。作者发现这个object detection跟常人的自然认知不同，人在看到一副图片，看一下在就能够知道目标在哪里，目标是什么。所以，作者把推荐（定位）和分类（是否存在）两个步骤合并为一个步骤，从而把两个网络合并为一个网络。所以YOLO就是一个CNNs+fc就这么简单。  

## YOLO核心思想              
整张图片作为输入，之后在输出层回归bounding box的位置和类别。     

## YOLO优势   

1. 快：只用一个网络，想想也是应该快           
2. 全局意识：分类的时候，是把整个图片输入到网络中，而不是想R-CNN那样输入图片的一部分          
3. 有目标的归纳代表性（YOLO learns generalizable representations of objects.）     
4. 可以做到实时（快到一定程度）        

## YOLO网络结构                  

![architecture](/images/blog/2017-2-15/architecture.png)

这里一共是用了24层的Conv后面跟着两个全连接层。基本思想很大程度上参考了GoogleNet。          
__这里这个通道值的变化我至今也没有看明白，源码没有深入看，有空了要看一看。__

## 检测过程

![detection](/images/blog/2017-2-15/detection.png)

把一张图片分成S×S个cell，每个cell都会预测出BGE边界框（bounding boxes）。给这个box赋一个可信（confidence）的值，用来表示是否有object的中心在这个cell中，即Pr(Object)*IOU。每个边界框都有五个值x,y,w,h,confidence，其中（x,y）表示box的中心。        
每个cell有个分类概率值，即Pr(Class<sub>i</sub>|Object)。只有cell预测这个值，与boxes无关。        
在预测的时候，对于每个box有这样的函数：
![predict_fucntion](/images/blog/2017-2-15/predict_function.png) 

## 训练过程             

### 预训练                           
在开始正式开始训练之前，对整个模型的参数使用已经训练完成的，这些参数是ImageNet训练成的。                     

### 输入               
这里输入是固定大小，不是224×224，而是因为要发现目标，大了一倍是448×448。              

### 激活函数        
最后一层用的是线性激活函数，其他层用的是leaky rectified linear activation:  
![fucntion](/images/blog/2017-2-15/function1.png)              

### 损失函数<sup>[1]<sup>
![loss_fucntion](/images/blog/2017-2-15/loss_function.png)     

- 只有当某个网格中有object的时候才对classification error进行惩罚。
- 只有当某个box predictor对某个ground truth box负责的时候，才会对box的coordinate error进行惩罚，而对哪个ground truth box负责就看其预测值和ground truth box的IoU是不是在那个cell的所有box中最大。

## YOLO的局限                 
- 每个cell产生的bounding boxes比较少，一个cell里面如果有多个object的话，就无法Detection。比如鸟群。                  
- 每张图片固定输入，图像走形。这个SPPnet<sup>[2]</sup>有个解决方法。SPPnet就是解决R-CNN的这个问题。             
- IOU的损失对于大目标和小目标的影响没有考虑。             

## 实验       
这篇文章，还提出了一个叫做Fast YOLO（9layers）的模型。YOLO用的Googlenet，其他的用的VGG-16，作者也用VGG-16做了一个YOLO展示出来。                       

### 实验结果
![result](/images/blog/2017-2-15/result.png)            

### 错误分析           
![analysis](/images/blog/2017-2-15/analysis.png)         

### 非学术数据集实验    
作者都说自己是实时的，那就不用学术的数据集。这次用了Picasso Dataset 和 People-Art Dataset；另外直接在野外应用YOLO。            
![result2](/images/blog/2017-2-15/result2.png)     


## 个人总结
这篇文章就彻底摆脱了目标检测中的推荐过程。这个直接就是定位分类全是一个网络来完成。
   

## 个人想法
文章写到的局限那个图像走形，可以试试用SPPnet的思路来试试。看完YOLO9000要考虑这个事儿。


## 注释           
[1][论文阅读笔记：You Only Look Once: Unified, Real-Time Object Detection](http://blog.csdn.net/tangwei2014/article/details/50915317)         
[2]K. He, X. Zhang, S. Ren, and J. Sun. Spatial pyramid pooling in deep convolutional networks for visual recognition. In ECCV, 2014. 1
