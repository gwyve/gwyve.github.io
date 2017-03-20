---
layout: post
title: 读文笔记ResNet        
category: blog
description: 阅读 Deep Residual Learning for Image Recognition 笔记              
---


声明：本博客欢迎转发，但请保留原作者信息!      
作者：高伟毅    
博客：[https://gwyve.github.io/](https://gwyve.github.io/)    
微博：[http://weibo.com/u/3225437531/](http://weibo.com/u/3225437531/)    
  
## 引言    
这是CVPR2016上的best paper，这篇文章确实厉害，不服不行。主要就是为了解决随着网络深度增加，出现网络退化问题（degradation）。这个文章提出的ResNet也确实厉害的不行。
                                      

## 发表位置  

- CVPR 2016 best paper
- [PDF arXiv：1512.03385.pdf](https://arxiv.org/pdf/1512.03385.pdf)          
- [非官方翻译](http://www.leiphone.com/news/201606/BhcC5LV32tdot6DD.html)                      

   

## 问题引入

深度神经网络变深成为了一种趋势，但是，随着深度增加，遇到了梯度爆炸、消失的问题，这个问题用normalization initialization有了改观。可是，一个叫做退化(degradation)的问题又出现了，这个并不是因为过拟合产生的。退化问题：随着网络层数增加，正确率开始饱和，然后快速降低。退化问题使得求最佳变得困难。
  
![degradation](/images/blog/2017-3-20/degradation.png) 


## ResNet核心思想   
           
### 参差(Residual)             

这个文章就是讲的残差网络，核心思想必然就是残差。我理解的残差：我有一个模型是y = H(x)，我整个训练过程就是为了求H(x)的样子，现在我不求H(x)了，我重新定义一个模型F(x) = H(x)-x。那么训练出来的这个F(x)，跟这个H(x)只差了一个输入值x，就把这个H(x)，称作残差。

### Identity Mapping by Shortcuts

我看到有的博客，把这个identity翻译成了“身份”，我觉得这个词在这里应该当做“恒等”讲，这个意思就是直接减去输入值。下图中，两个权层就是上面讲H(x)，那个标注identity的箭头，就表示直接减去输入值，这个图的全部就是H(x)的残差F(x)。

![Identity](/images/blog/2017-3-20/identity.png) 


### plain network

从这篇文章中，我并没有想到这个结构有多大的意义，作者也并没有说太多为什么要设计这样一个网络结构。本人私以为，这个就是为了增加网络深度用的，在具体实现上，可以明显看到，这个结构根据VGG-19相比，在channel上的数据量减少，但是，在网络深度上增加了。虽然深度增加了，但是，因为channel数据量的减少，导致计算复杂度还下降了。     

![plain_network](/images/blog/2017-3-20/plain_newwork.png) 


## 实现细节      
                       
- 不用dropout，而是使用的batch normalization(BN)                
- 训练了60x10^4 iterations                     
- 学习率从0.1开始，遇到停滞（plateaus），被分10               
- 权衰减0.0001，冲量0.9


## 实验与分析

### plain network && residual network

使用plain network，18层比34层的好，是因为退化的问题。添加上残差，情况就反过来了。

![plain_residual](/images/blog/2017-3-20/plain_residual.png) 

### Identity vs. Projection Shortcuts

注：parameter-free，我的理解是不使用参数

使用恒定捷径的时候有助于训练，使用映射捷径的时候，分成了三类：
- A:有维度变化的残差，输入值直接往输出上怼，没有的边缘，补0                  
- B:维度增加的，在输入值上乘上一个矩阵转换维度，维度不变的，用恒定捷径                     
- C:全部都在输入值上乘一个矩阵怼到输出值上

**这里是我不太明白的地方，主要是不明白怎么怼的问题，他说这个乘以一方阵（square），我怎么也不理解是怎么做的。**



## 结论                            

这个文章，说了一个残差网络，解决了退化的问题，在image recognition、 object detection、 localization获得应用，并取得不错的效果。

这个文章，不愧是best paper，从内容上讲明白了东西，读完了，就觉得这个文章有所收获，不只是因为有干货，写的也是牛，真是一个字，服。

这个文章说的这个残差，我觉得，我目前用不上了，就这个显卡，没啥希望。看这个文章，自己扩展知识面罢了。

   

## 个人想法

这篇文章不愧是                         