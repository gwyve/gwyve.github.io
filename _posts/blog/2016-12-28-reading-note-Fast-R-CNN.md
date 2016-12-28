---
layout: post
title: 读文笔记Fast R-CNN
category: blog
description: 阅读Fast R-CNN笔记
---




声明：本博客欢迎转发，但请保留原作者信息!      
作者：高伟毅       
博客：[https://gwyve.github.io/](https://gwyve.github.io/)    
微博：[http://weibo.com/u/3225437531/](http://weibo.com/u/3225437531/)    


## 引言

这篇文章属于R-CNN之后的一篇，基本是在R-CNN<sup>1</sup>和SPPnet<sup>2</sup>基础之上的，推荐阅读R-CNN和SPPnet两篇再阅读该文章（我也是先看的这一篇，但是不得已又看了SPPnet）。    

## 发表位置
- [ICCV 2015](http://www.cvpapers.com/iccv2015.html)   
- [PDF.v2](https://arxiv.org/pdf/1504.08083v2.pdf)

## 问题引入

整篇文章是在R-CNN和SPPnet基础之上的改进，自然就是在与R-CNN和SPPnet进行对比，通过发现R-CNN和SPPnet的缺点来提升，从而形成Fast R-CNN（简称FRCN）。    

### R-CNN  
- 多阶段训练（Training is a multi-stage pipeline）：1、object proposal(SS) 2、object detector(SVM) 3、bounding-box regressor
- 空间时间上花费大：SVM、bounding-box regressor的训练过程需要把特征写于硬盘。
- 目标检索的时候慢：对每一个object proposal都跑一遍CNN（SPP就是发现该问题提升的速度）     

### SPPnet
- 多阶段训练
- 在SPP层之前的CNN不随着训练数据更新


## Fast R-CNN结构和训练过程

具体过程如下步骤：          
1. 整张图片输入Cnov层和pooling层生成conv feature map               
2. 从Feature map推荐生成一定数量的RoI     
3. 对每个RoI进入RoI pooling层，生成固定长度的向量。                  
4. 第3步产生的输入全连接（fc）层，fc层分成两个sibling output layer：一个生成K+1个类的softmax，另一个生成bounding-position的位置。               

![architecture](/images/blog/2016-12-28/architecture.png)

### RoI pooling layer
这个pooling层跟SPPnet里面的SPP层很类似，只是SPP层是分为多个层次的max pooling（1,4,16....），这里的RoI pooling layer只有一个层次的取样（作者也提到这是一个特殊的SPP）。如果要输出H×W，输入是h×w，那么每个小pooling块的大小就是h/H × w/W。

### 多任务损失
这里从fc直接出来两个损失，是在联合训练。    
![multi-task loss](/images/blog/2016-12-28/multi-task-loss.png)   
第一项是跟某个目标对象进行比对，第二项是位置损失函数。     
作者在这里说明之前的方法（OverFeat<sup>3</sup>，R-CNN，SPPnet）都是用的stage-wise的方法，Fast R-CNN是用的比他们更优。

### 小批量取样
这里确实是提升速度的一个方法，之前的方法都是用全部数据把模型训练完的，会频繁的访问硬盘，这个小批量只用把这小批量的数据放在内存就可以。这里2张图128RoI。

### 透过RoI pooling前馈传播
他说的这个意思，max pooling只能影响一部分（即生成最大数的这一部分），但是，可以通过这一部分对Conv层的filter产生影响。    
我觉得，既然他这个RoI pooling作为特殊的SPP可以传播，那SPP应该也可以传播。

### 截短的SVD
这一块我打算再看一下SVD之后再自己研究

## 实验结果
他使用三个实验模型：S、M、L。

### mAP
在mAP上与R-CNN、SPPnet对比，就像挤牙膏一样。     
![experiment result](/images/blog/2016-12-28/experiment_result.png)

### 时间
时间上提升很明显的，毕竟说的是fast嘛    
![time result](/images/blog/2016-12-28/time_result.png)   

## 设计测评
这个就是各种实验，然后亮出数据表，一看有用。这个方法是写论文必用的。     
### 截短的SVD
使用该方法，mAP下降0.3%，时间减少30%     
![SVD](/images/blog/2016-12-28/SVD.png)

### fine-tune哪一层
fine-tune conv层有用，但并非所有的都有用，而且还可能花费更多的训练时间。     
![fine-tuen](/images/blog/2016-12-28/fine-tune.png)

### 多任务训练有用吗？
有用    
![multi-task test](/images/blog/2016-12-28/Multi-task_experiment.png)  

### 尺度不变
这里在考虑是使用多种尺度还是单一尺度，结果是发现多种尺度提升mAP效果一般，速度变慢。所以，作者认为单一尺度是一种速度和准确的权衡，特别是对于很深的模型。    
![scale](/images/blog/2016-12-28/scale.png)   

### 需要更多训练数据吗？
训练数据越多mAP越大，那么结果显然

### SVM or softmax
R-CNN、SPPnet使用的SVM，FRCN使用softmax。    
![svm_soft](/images/blog/2016-12-28/svm_softmax.png)      
虽然svm比softmax稍微好一点，但是它证明了，跟多阶段训练比较，一下子的微调是足够的。softmax引入了各个类的竞争。     
我觉得，作者仍然选择使用softmax很可能跟多任务损失有关。

### 更多的proposals有用吗？
一定程度有用，太多了就可能有害了。            
![map](/images/blog/2016-12-28/map_ap.png)
又提到AR，但是又说AP要慎重考虑，也确实是这样的。

## 个人总结

### 提升准确率的
- RoI pooling layer可以前馈传播，可以更新Conv的filter
- fc输入两个损失函数，联合训练

### 提升速度的
- 小批量训练：避免数据集频繁访问硬盘
- 截短的SVD

## 个人想法
整片论文确实严谨，不过，在有个地方确实不理解，那就是为什么RoI Pooling layer使用的单一层次的max pooling，如果使用SPP层那样的会怎么样，这里本文作者也并没有指出。     
本文有两个版本，强烈建议直接阅读第二个版本，第一个版本不建议读，在第一个版本读到Back-propagation through RoI pooling layers，就彻底懵了。


## 相关
[1]R. Girshick, J. Donahue, T. Darrell, and J. Malik, “Rich feature
hierarchies for accurate object detection and semantic segmentation,”
in CVPR, 2014.    
[2]K. He, X. Zhang, S. Ren, and J. Sun. Spatial pyramid pooling
in deep convolutional networks for visual recognition. In
ECCV, 2014. 1  
[3]P. Sermanet, D. Eigen, X. Zhang, M. Mathieu, R. Fergus,
and Y. LeCun. OverFeat: Integrated Recognition, Localization and Detection using Convolutional Networks. In ICLR, 2014. 1, 3        
              
