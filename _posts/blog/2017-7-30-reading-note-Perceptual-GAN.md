---
layout: post
title: 读文笔记 Perceptual GAN        
category: blog
description: 阅读 Perceptual Generative Adversarial Networks for Small Object Detection 笔记              
---


声明：本博客欢迎转发，但请保留原作者信息!      
作者：高伟毅    
博客：[https://gwyve.github.io/](https://gwyve.github.io/)    
微博：[http://weibo.com/u/3225437531/](http://weibo.com/u/3225437531/)    
  
## 引言    

之前的A-Fast-RCNN用的就是对抗网络，这个也是用的生成对抗网络。虽然也是object detection，但是，他所用的数据集并不是之前用的VOC、MS COCO，而是用于无人驾驶的图像数据集（交通信号检测、行人检测）。当然，这个也是中了CVPR2017的论文。
     
                                      

## 发表位置  

- CVPR2017
- [PDF arXiv：1706.05274.pdf](https://arxiv.org/pdf/1706.05274.pdf)          
                     

## 研究趋势回顾

作者主要是针对small object的目标检测，作者在introduction里面指出，目前所做的有两个方式：

- 增大输入图片尺寸：这就会造成计算量的扩大                    
- 改变网络结构：改变网络结构是一个黑盒的事情，不能保证结构正确


## 核心想法

将small object通过生成网络，生成高分辨率的输出，并且这个输出与真实large objcet相似的；生成网络就是通过根据small objcet 生成的 large objcet来欺骗判别网络中的对抗分支，判别网络的对抗分支提升自身的判别能力作为对抗（GAN的核心思想）。


## Perceptual GAN

### Overview

这个结构一共分为四个部分：

- 左侧上面的画的五个卷积，就是普通的VGG结构，在本文中，并没有特殊的作用。                          
- 左侧下面的是一个生成网络，负责根据small object生成类似于large object的representation，从而欺骗对抗网络。            
- 右侧上面是，判别网络的对抗分支，主要负责判别输入的网络结构是原始的large object的representation，还是根据small object生成的representation。        
- 右侧下面的网络是判别网络里面感知分支，主要负责分类和定位，类似于传统object Detection的最后的网络

![overview](/images/blog/2017-7-30/overview.png)


### 网络结构细节

![network_detail](/images/blog/2017-7-30/network_detail.png)

在VGG的基础上添加了以残差结构为基础的生成网络，最后添加了全连接网络为基础的对抗网络。

### 训练步骤细节

首先用large object 训练基础网络和感知网络分支，然后用small object来训练生成网络和判别网络，从而提升网络的判别能力。


## 实验结果

在small object上面确实有效果，毕竟中的CVPR

### Traffic-sign Detetion

![traffic_sign](/images/blog/2017-7-30/traffic_sign.png)

![traffic_sign_fig](/images/blog/2017-7-30/traffic_sign_fig.png)

### Pedestrian Detection

![pedestrian](/images/blog/2017-7-30/pedestrian.png)

## 实验分析           
 
### 生成的高分辨率特征的作用

![super_resolved_feature](/images/blog/2017-7-30/super_resolved_feature.png)

这里，

- Skip Pooling: 跳过池化层的，含义就是特征通过池化层，并没有减小。                             
- Large Scale Image: 输入的图片是2048x2048的图片，使用的Faster R-CNN。                 
-Multi—scale Input：使用Faster R-CNN，输入的有1120,1340,1600,1920,2300不同尺度的图片。

### 对抗训练的作用

![adversarial](/images/blog/2017-7-30/adversarial.png)

这里，baseline是没有经过对抗训练的。

### 生成网络放在不同层的效果

这里，说的意思是放在左上层的第几层之后的效果。

![lower](/images/blog/2017-7-30/lower.png)

## 结论                            

这篇论文是立足于small object进行解释的，通过实验分析，也确实看到该模型在small object确实有着不错的表现。     

   

## 个人想法

1. 这篇文章的主要数据集是使用的交通信号检测和行人检测的，这两个方面也确实是无人驾驶的一个研究方向。                  
2. 如果，刨除作者提到的单独拿small object进行比较，这篇文章比A-Fast-RCNN在对抗网络上多了一个部分，那就是在判别网络上专门设置了一个对抗分支，而非A-Fast-RCNN，直接将输出网络作为对抗网络。          
3. 这个实验是基于FRCN进行的，在VGG的基础上添加的残差模块结构。      




                          
