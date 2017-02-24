---
layout: post
title: 读文笔记DeepMultiBox        
category: blog
description: 阅读 Scalable Object Detection using Deep Neural Networks 笔记
---


声明：本博客欢迎转发，但请保留原作者信息!      
作者：高伟毅    
博客：[https://gwyve.github.io/](https://gwyve.github.io/)    
微博：[http://weibo.com/u/3225437531/](http://weibo.com/u/3225437531/)    
  
## 引言    
这是在SSD之前的一篇文章，在SSD<sup>[1]</sup>中多次提到这篇文章，源于对SSD不是很懂，故此精读这篇论文。这篇论文很早就打印出来，并泛读过，这几天重新精读，故此笔记。因为是很久之前的文章，现在看来并没有太大的新意，很多地方就省略记录了。               
这篇论文在题目上就是Scalable的问题，跟SPPnet<sup>[2]</sup>在某种程度上有解决类似的问题。


## 发表位置  
- CVPR 2014
- [PDF.v1 arXiv：1312.2249.pdf](https://arxiv.org/pdf/1312.2249.pdf)      

   

## 问题引入
之前的object detection是根据图像的底层特征进行彻底详尽得搜索，每一个对象种类都有一个用于预测的模式，（DPM<SUP>[3]</SUP>的方法）。随着种类的提升，这种做法就变得越来越困难了。因此，作者提出一种叫做“DeepMultiBox”，使用深度神经网络预测出一个框，这个框并不管是什么种类，只负责确定里面有object，之后再进行分类。

  

## DeepMultiBox核心思想              
整张图片的不同尺度的Max-center square输入到dnn中，判断里面是否有object，但是不分类。从而把object Detection的发现部分解决，后面交给分类器解决。     


## 训练                             
整个图片作为输入，损失函数分为两部分：框重复匹配损失、置信损失。

![loss_match](/images/blog/2017-2-24/loss_match.png)                   
![loss_confidence](/images/blog/2017-2-24/loss_confidence.png)            
![loss](/images/blog/2017-2-24/loss.png)                        
i-th是预测的box，j-th是真实的box，只有i-th预测对应了j-th个的时候x<sub>i,j</sub>才为1，其他的为0。l<sub>i</sub>是预测box的位置，g<sub></sub>是真实box的位置。c<sub>i</sub>是预测的box中有object（不知道是哪一类）的置信度。α是平衡两个损失的超参数。


### 训练细节                        
- 这里先把ground truth boxes聚类，通过与具有代表性的聚类进行回归，以此来节省训练时间。
- 在4.3中，作者提到了这个多个尺度的问题，具体我没有看懂。但是跟朋友猜测，就是一个中心最大的crop和几个不同尺度的中心最大的crop输入到网络中，尺度越多效果越好。具体语句，见图。         
![4.3](/images/blog/2017-2-24/4.3.png)                  
![figure1](/images/blog/2017-2-24/figure1.png)             


## 结论与分析                            
- 总结了DeepMultiBox方法。
- 作者提到了未来要做的就是把定位和识别放到一个网络中。        

   

## 个人想法
这篇文章是很早的文章了，2013年的，当时，还是基于HoG的DPM大行其道的时候，这样一篇文章现在看起不起眼，但是确实有很大的作用。而且，后来的很多地方都跟这个是有关系的，都能发现这篇文章贡献的影子。


## 注释           
[1]W. Liu, D. Anguelov, D. Erhan, C. Szegedy, and S. E. Reed.SSD: single shot multibox detector. CoRR, abs/1512.02325,2015. 4, 5, 6                 
[2]K. He, X. Zhang, S. Ren, and J. Sun. Spatial pyramid pooling in deep convolutional networks for visual recognition. In ECCV, 2014. 1                