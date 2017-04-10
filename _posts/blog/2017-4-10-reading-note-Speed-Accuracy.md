---
layout: post
title: 读文笔记Speed/Accuracy trade-offs        
category: blog
description: 阅读 Speed/accuracy trade-offs for modern convolutional object detectors 笔记              
---


声明：本博客欢迎转发，但请保留原作者信息!      
作者：高伟毅    
博客：[https://gwyve.github.io/](https://gwyve.github.io/)    
微博：[http://weibo.com/u/3225437531/](http://weibo.com/u/3225437531/)    
  
## 引言    

这个文章应该是对最近三个object Detection模型（SSD、Faster R-CNN、R-FCN）的一个总结，作者主要用了同一个框架（Tensorflow）重写了三个模型，并结合不同特征提取器和各种参数，做了一个对比。数据集使用2016 COCO。目前，还没有发现作者公布代码托管。        
                                      

## 发表位置  

- [PDF arXiv：1611.10012.pdf](https://arxiv.org/abs/1611.10012)          
                     
   

## 问题引入

目前object Detection的方法有很多，在选择上存在较难，急需有个针对这些模型的对比。针对这一问题，作者使用tensorflow把这些模型全部实现了一遍，并有针对性地修改某些参数，进行对比。

  

## Meta-atchitectures   
           
三个模型（SSD、Faster R-CNN、R-FCN）都使用了“Anchor”的想法，作者指出，有的时候，这个被叫做“priors”、“default boxes”，所包含的意思都是，提前出一个框，然后再微调。

这个Anchor的选择上，最开始的MultiBox<sub>[1]</sub>，是用真框聚类。最近的工作都是tiling出的。

这个损失函数：

![LOSS](/images/blog/2017-4-10/loss.png)     
    


三个模型的对比，我之前都总结了：

[SSD](http://gwyve.github.io/blog/2017/03/01/reading-note-SSD.html)

[Faster R-CNN](http://gwyve.github.io/blog/2017/01/05/reading-note-Faster-R-CNN.html)

[R-FCN](http://gwyve.github.io/blog/2017/03/29/reading-note-R-FCN.html)

![meta-arch](/images/blog/2017-4-10/meta-arch.png)

  

## Feature extractors    

用了六个特征提取网络，其中mobileNet我并没有找到；这个待之后总结。

这几个object Detection模型使用的具体情况如下：          

![Feature_extractors](/images/blog/2017-4-10/Feature_extractors.png)

这网络在分类上的效果如下：   
  
![classi](/images/blog/2017-4-10/classi.png)



          
### VGG-16<sub>[2]</sub>

![VGG](/images/blog/2017-3-1/VGG.png)

### Resnet-101<sub>[3]</sub>

![ResNet](/images/blog/2017-4-10/ResNet.png)

### Inception v2<sub>[4]</sub>

添加BN（Batch Normalization）的Inception

### Inception v3<sub>[5]</sub>

为了提高速度把3x3换成1x3和3x1

### Inception Resnet v2<sub>[6]</sub>

在Resnet上应用Inception

### MobileNet<sub>[7]</sub>

没有找到相关资料，坐等CVPR 2017之后

## 训练和调超参

- 异步梯度更新，使用distributed cluster<sub>[8]</sub>
- Faster R-CNN、R-FCN用的是end-to-end，而不是4步训练
- 因为内存问题，作者有时用的batch size不是32

## 硬件

- 32GB RAM
- Intel Xeon E5-1650 v2 
- Nvidia GeForce GTK Titan X


## 分析

### 正确率vs时间

SSD、R-FCN速度快过Faster R-CNN

![accuracy_time](/images/blog/2017-4-10/accuracy_time.png)

### 重要点的模型

![CriticalPoint](/images/blog/2017-4-10/CriticalPoint.png)


### 特征提取（网络结构）的影响

与直觉相同，在分类上好的mAP高

![netVSmap](/images/blog/2017-4-10/netVSmap.png)

### objcet 大小影响

在大objcet都挺好，SSD在小object上很不好。

![objcetSize](/images/blog/2017-4-10/objcetSize.png)

### 推荐区域数量的影响

Faster R-CNN、R-FCN是从RPN里推荐出数量的。这里只对比这两个模型，

![proposal](/images/blog/2017-4-10/proposal.png)


### FLOPs分析

这个要表达的意思，我还是不太了解。

![GPUtime](/images/blog/2017-4-10/GPUtime.png)


### 内存分析

SSD使用最少

![Memory](/images/blog/2017-4-10/Memory.png)

![MemoryVStime](/images/blog/2017-4-10/MemoryVStime.png)
                         
## 个人总结

虽然SSD在mAP和small object上有欠缺，但是，他在内存使用和时间上，都表现较好，这个跟他single stage网络结构和default box的数量有关吧。

残差表现的很好，可是没有公布的训练好的模型参数，或许是我没有找到，使用ResNet或者Inception v4应该有不错的表现。
   

## 个人想法

如果把这篇文章看做是一种综述，那作者所做的确实不只是调研；论文的另一种写法把。可以说，这个文章背后的工程量一点都不小，可惜，我还没有看到他把代码托管在哪里，或许作者分享了他的代码比分享这个文章更让人称赞吧。

Tensorflow是一个大的趋势，有Google这样的大妈，逐渐会越来越多人使用，可能会更贴近与工业界吧。



## 引用               

[1]D. Erhan, C. Szegedy, A. Toshev, and D. Anguelov. Scalable object detection using deep neural networks. In Proceedings of the IEEE Conference on Computer Vision and
Pattern Recognition, pages 2147–2154, 2014. 2            
[2]K. Simonyan and A. Zisserman. Very deep convolutional networks for large-scale image recognition. arXiv preprint arXiv:1409.1556, 2014. 4, 6, 7                  
[3]K. He, X. Zhang, S. Ren, and J. Sun. Deep residual learning for image recognition. arXiv preprint arXiv:1512.03385, 2015. 3, 4, 5, 6, 7, 13, 14                 
[4]S. Ioffe and C. Szegedy. Batch normalization: Accelerating deep network training by reducing internal covariate shift. arXiv preprint arXiv:1502.03167, 2015. 4, 6, 7                         
[5]C. Szegedy, V. Vanhoucke, S. Ioffe, J. Shlens, and Z. Wojna. Rethinking the inception architecture for computer vision. arXiv preprint arXiv:1512.00567, 2015. 4, 6                  
[6]C. Szegedy, S. Ioffe, and V. Vanhoucke. Inception-v4, inception-resnet and the impact of residual connections on learning. arXiv preprint arXiv:1602.07261, 2016. 4, 6, 7                         
[7]Anonymous. Mobilenets: Efficient convolutional neural networks for mobile vision applications. Submitted to CVPR
2017, 2016. 4, 6, 7                         
[8]J. Dean, G. Corrado, R. Monga, K. Chen, M. Devin, M. Mao, A. Senior, P. Tucker, K. Yang, Q. V. Le, et al. Large scale distributed deep networks. In Advances in neural information processing systems, pages 1223–1231, 2012. 3, 5    