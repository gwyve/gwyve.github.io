---
layout: post
title: 读文笔记SSD        
category: blog
description: 阅读 SSD:single shot multibox detector 笔记              
---


声明：本博客欢迎转发，但请保留原作者信息!      
作者：高伟毅    
博客：[https://gwyve.github.io/](https://gwyve.github.io/)    
微博：[http://weibo.com/u/3225437531/](http://weibo.com/u/3225437531/)    
  
## 引言    
这篇文章是在YOLO<sub>[1]</sub>之后的一篇文章，这篇文章目前是一篇the-art-of-state的方法。这篇文章可以看到很多前人的影子，我所感受到这篇文章主要借鉴前人的有DeepNultiBox<sub>[2]</sub>、Faster R-CNN<sub>[3]</sub>、YOLO<sub>[1]</sub>、VGG16<sub>[4]</sub>等。        
                                      

## 发表位置  

- ECCV 2016
- [PDF.v5 arXiv：1512.02325.pdf](https://arxiv.org/pdf/1512.02325.pdf)          
- [代码托管](https://github.com/weiliu89/caffe/tree/ssd)                      

   

## 问题引入

对于object detection，之前的方法都没有特别快，虽然YOLO很快，但是在准确性上下降了较多。作者就提出了一个速度很好，准确性还好的方法——SSD。

  

## SSD核心思想   
           
整个图片做输入，之后在输出层回归出box的位置和object的类别。与YOLO不同的是，SSD在输出层只用conv layer，而不是全链接层。     

## 模型    
          
![model](/images/blog/2017-3-1/model.png)                       
这个模型，是以VGG16进行修改的，把VGG16的fc层换成conv层，中间还夹杂了maxpooling层。每个额外添加的conv层都输出一个Feature map，并以此作为预测一个输入，换句话说，这些不同scale的Feature map都作为预测的输入，以此来达到不同scale。作者举了一个例子，如果一个Feature map是m×n×p（width×height×channel）的，那么用3×3×p的kernel去产生value，产生了m×n个value。

为了方便读者理解这个网络结构，我把VGG<sub>[4]</sub>论文中的图粘贴过来了，作者用的是D这个模型进行修改的。
![VGG](/images/blog/2017-3-1/VGG.png)     


## 训练      
                       
### default boxes  
           
作者在这里提到的default boxes与faster R-CNN中的Anchor很类似，这里是按照YOLO中的想法，把每个Feature map（不同scale的）分成m×n个cell，每个cell有默认出k个default boxes，最后的predict box与default box有4个offset，并为每个predict box计算c个类的值。最后产生了（c+4）kmn个值。这里与faster R-CNN Anchor最大的不同就是，faster R-CNN是通过改变Anchor的大小来实现scalable的，SSD是改变Feature map大小来实现的。
![frameword](/images/blog/2017-3-1/framework.png) 

### 训练目标

损失函数：这个与Faster R-CNN中的RPN是一样的，不过RPN是预测box里面有object或者没有，所以，没有分类，SSD直接用的softmax分类。location的损失，还是一样，都是用predict box和default box/Anchor的差 与 ground truth box和default box/Anchor的差 进行对比，求损失。                        
![loss](/images/blog/2017-3-1/loss.png)                      

以下，是我根据我的理解画的一张简单示意图。这个default box存在求损失，就是尽可能把X与Y的数值做到差值最小。

![loss](/images/blog/2017-3-1/loss_.png)

### 为default box选择尺度（scale）和长宽比   
                         
如果我们用m个Feature maps进行预测，那么尺度最大的是s<sup>max</sup>，最小的是s<sup>min</sup>，那么第k个Feature map的default boxes的尺度s<sup>k</sup>：
![scale](/images/blog/2017-3-1/scale.png)                        
使用不同的长宽比，a<sup>r</sup>∈{1,2,3,1/2,1/3}，长度（w<sup>r</sup> = s<sup>k</sup> × squr(a<sup>r</sup>)），高度（h<sup>r</sup> = s<sup>k</sup> / squr(a<sup>r</sup>)）。除此之外，对于a<sup>r</sup>=1，还有一个情况s<sup>k</sup>‘= squr(s<sup>k</sup> × s<sup>k+1</sup>)。

### negative数据处理       

这种方法会产生很多个negative的数据，使positive与negative失去平衡，作者把negative：positive控制在3:1。

### 数据扩大      
![data_augmentation](/images/blog/2017-3-1/data_augmentation.png)                                     


## 实验结果                               
              
### PASCAL VOC2007

![VOC2007](/images/blog/2017-3-1/VOC2007.png)                                     

### PASCAL VOC2012

![VOC2012](/images/blog/2017-3-1/VOC2012.png)                                     

### COCO

![COCO](/images/blog/2017-3-1/COCO.png)                                     
             

## 模型分析

### 数据扩大很重要

![data_augmentation_effect](/images/blog/2017-3-1/data_augmentation_effect.png)  

### 多个default box 尺度更好

### atrous 更快

目前，我还没有明白atrous是什么。

### 多个Feature map用于预测更好

![multiple_layers](/images/blog/2017-3-1/multiple_layers.png)  

### SSD定位错误少

因为他把定位和分类放在一步里解决。

![analysis](/images/blog/2017-3-1/analysis.png)

### 小目标效果不好

因为在top layer中，获得小目标的信息不多，扩大图片大小输入，会有改善。通过数据扩大也会有改善。

![smallobject](/images/blog/2017-3-1/smallobject.png)

## 速度

使用 Titan X 、 cuDNN v4 、 Intel Xeon E5-2667v3@3.20GHz。

![speed](/images/blog/2017-3-1/speed.png)




## 结论                            

- SSD的各种特定和优点。
- SSD将会获得更为广泛的应用。       

   

## 个人想法

这篇文章在很早之前就想拜读，但是，到今天才扣完。之前读的object Detection文章基本都是在为了这篇做基础。这篇文章看完了，想法有很多，现在简单叙述一下把。

### 巨人肩膀

- **DeepMultiBox:**这篇文章主要讲的多个scale的问题，利用图片放缩的方法进行不同scale多个框预测。文章在最后提到把定位分类放到one-shot的网络里面，我觉得作者可能受到这个的启发。                                                    
- **YOLO:**YOLO这篇文章就是把定位和分类放在一个网络里面，同时彻底拜托了之前位置推荐的方式。SSD中的default box的生成借鉴了YOLO中cell的想法。SSD在很多方面都是在于YOLO做对比。毕竟YOLO是发表在CVPR2016上的成果。                                                          
- **Faster R-CNN:**这篇文章主要借鉴了Faster R-CNN中anchor的想法。正如作者在提到“similar to Faster R-CNN”，SSD里面提到的“default bounding box”直接按照anchor理解的话，便容易理解多了，SSD与Faster R-CNN中关于Anchor的使用区别主要在Scalable上。                                                
- **VGG16:**这个主要说的是个网络结构，SSD主要就是根据VGG16改的，VGG16中抛弃了之前用的5×5、7×7、11×11等大的卷积核，而是全用3×3的卷积核（VGG文章中提到了具体原因，大主要有1.更多个non-linear rectification layers.）。  

### 作者提到的改进空间              

- “How to design the optimal tiling is an open question as well”，这是作者在提到default box的尺度问题时候说的。                              
- 针对small object的问题，这个跟YOLO是一致的，我觉得这很可能是这样划分cell的问题。


## 注释           
[1]Redmon, J., Divvala, S., Girshick, R., Farhadi, A.: You only look once: Unified, real-time object detection. In: CVPR. （2016)                     
[2]Erhan, D., Szegedy, C., Toshev, A., Anguelov, D.: Scalable object detection using deep neural networks. In: CVPR. (2014)                      
[3]Zhang, L., Lin, L., Liang, X., He, K.: Is faster r-cnn doing well for pedestrian detection. In:ECCV. (2016)                                 
[4]Simonyan, K., Zisserman, A.: Very deep convolutional networks for large-scale image recognition.In: NIPS. (2015)                            