---
layout: post
title: 双电源供电
category: blog
description: 针对电源功率不足，双电源供电
---


## 引言

自己搞DNN在CV上的实验，需要显卡，实验室只有两张Tesla K20，这个4GB显存的显卡，在目前大数据集下，显得有些捉襟见肘。逼得急了，在TB上搜到了[租1080Ti](https://item.taobao.com/item.htm?spm=a1z09.2.0.0.GyZ5V5&id=547034476533&_u=259gaih0075),当时，感觉就是柳暗花明又一村，那个舒爽程度，类似于吃了槟榔顺气丸。

可是，想到供电问题，菊花又是一紧，以下来记录下我填过的各种坑。

## 材料

CPU：6700K        
主板：Z170-A             
显卡：1080Ti，两张                
电源：400w一块，600w一块               
双电源启动线：1根                            

## 步骤

### 1. 租显卡

穷人没有办法只能自己租显卡来搞，推荐各位想做机器学习或者深度学习的小伙伴们，如果自己没有打游戏的需求，那就果断选择租卡吧。推荐TB上搜“租显卡”，目前只有[租去](https://shop202168812.taobao.com/?spm=2013.1.0.0.cR7TqO)这一家，老板人挺好，大家可以试一试～

### 2. 买双电源启动线

目前这个在TB上挺多的，直接输入“双电源启动线”就有，价格也比较便宜，十几块钱就拿下了；据说这个之前都是手工制作的，现在便宜多了。

### 3. 组装机器<sub>[1](http://itbbs.pconline.com.cn/diy/14392370.html?nsukey=nn9k6dKZTfdurPMOSihAdPI4DYHB42hTn6MZwvEhRl8x34cwMTg0c9SdKHoaMMq7063KFT2akHMWZS5YDClXuwahMmZJTWXOKXSp8c%2B1j0aEZCBmW5DFpnzTuzRoPAkYPZV5TLqkE8F%2B%2Bvev%2BnE007Eze2VDd9UIyfZhgUvoqQ5y7KTFPGc8bD%2BeqNEc1hCR)</sub>

双电源启动线的公头插在主板上，线多的那个母头插在400w电源上，线少的那个插在600w的电源上：400w电源负责主板、CPU、硬盘供电；600w电源负责两张1080Ti的供电。**一定要这样做，否则各种坑，下面给大家讲一下哈**

### 4.大功告成

![done](/images/blog/2017-5-2/done.jpg) 

## 坑坑坑们

### 坑1：单电源供电

如果使用两张显卡，妄想使用两张1080Ti，那就趁早放弃，每次模型一开始跑，直接自动关机，重启！！！

### 坑2：600w负责一张GPU、CPU、主板；400w负责另一张GPU

最好不要使用GPU和CPU使用一路供电，很容易自动关机然后重启。个人推测，显卡某一个时刻的瞬时功率特别大，直接导致供电不足，CPU掉电。很多个夜晚的某时刻停下，浪费了很多个晚上。

### 坑3：使用300w电源给一张GPU供电

300w的电源，理论上完全可以带起一张250w的1080Ti，可是，实际上，这个根本就不成！不一定某个时刻，就可以听到另一张显卡巨大的风扇声，nvidia-smi打印出“GPU is lost”。个人推测，这个是其中一个GPU直接掉电。然后一个GPU失联就无法训练了，cudaerror是30。

### 坑4：电源没有插紧

这个问题，确实有，也比较常见～弄完了检查一下吧～


## 附录

[1][XX青年教你怎么用双电源带动GTX570,花60元300W电源拖GTX570显卡](http://itbbs.pconline.com.cn/diy/14392370.html?nsukey=nn9k6dKZTfdurPMOSihAdPI4DYHB42hTn6MZwvEhRl8x34cwMTg0c9SdKHoaMMq7063KFT2akHMWZS5YDClXuwahMmZJTWXOKXSp8c%2B1j0aEZCBmW5DFpnzTuzRoPAkYPZV5TLqkE8F%2B%2Bvev%2BnE007Eze2VDd9UIyfZhgUvoqQ5y7KTFPGc8bD%2BeqNEc1hCR)                     
[2] [TB租显卡](https://item.taobao.com/item.htm?spm=a1z09.2.0.0.GyZ5V5&id=547034476533&_u=259gaih0075)