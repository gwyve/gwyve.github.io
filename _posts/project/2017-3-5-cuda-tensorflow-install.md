---
layout: post
title: cuda && tensorflow 安装
category: project
description: Ubuntu平台NVIDIA driver cuda cudnn以及tensorflow安装   
---


声明：本博客欢迎转发，但请保留原作者信息!      
作者：高伟毅    
博客：[https://gwyve.github.io/](https://gwyve.github.io/)    
微博：[http://weibo.com/u/3225437531/](http://weibo.com/u/3225437531/)    

## 引言      

使用NVIDIA GPU进行dnn目前已经成为了主流，年前就打算自行安装一遍，拖了这么长时间，到今天才弄得差不多了。本来觉得这个不打算写个东西的，后来怕忘了，还是写下来吧

## 设备介绍

主机: ThinkStation-P300       
CPU: Intel(R) Core(TM) i7-4790 CPU @ 3.60GHz              
GPU: Tesla K20c

## 所需软件

以下软件，在国内只有tensorflow需要翻墙，NVIDIA的都可以直接下载，速度还可以的

### Ubuntu

[Ubuntu 16.04.2 LTS](https://www.ubuntu.com/download/alternative-downloads)                   
[file_torrent](http://releases.ubuntu.com/16.04/ubuntu-16.04.2-desktop-amd64.iso.torrent?_ga=1.169319585.1810803403.1486517128)

### 驱动

[Nvidia-375.39](http://www.nvidia.cn/download/driverResults.aspx/115286/cn)                               
[file](http://cn.download.nvidia.com/XFree86/Linux-x86_64/375.39/NVIDIA-Linux-x86_64-375.39.run)

### cuda

[cuda-8.0](https://developer.nvidia.com/cuda-downloads)                          
[file](https://developer.nvidia.com/compute/cuda/8.0/Prod2/local_installers/cuda_8.0.61_375.26_linux-run)      

### cuDNN

[cuDNN_5.1](https://developer.nvidia.com/rdp/cudnn-download)                         
[file](https://developer.nvidia.com/compute/machine-learning/cudnn/secure/v5.1/prod_20161129/8.0/cudnn-8.0-linux-x64-v5.1-tgz)

### tensorflow 

需要翻墙

[tensorflow_1.0.0](https://www.tensorflow.org/install/install_linux#installing_with_native_pip)                                
[file_gpu_python2.7](https://storage.googleapis.com/tensorflow/linux/gpu/tensorflow_gpu-1.0.0-cp27-none-linux_x86_64.whl)

## 安装步骤

### 系统安装

安装Ubuntu的过程自行搜索吧，教程很多，在此不说了。

### 安装NVIDIA Driver

起初，我是选择使用直接安装cuda的，cuda中有driver，但是，cuda安装完了之后，重复出现登录界面，无法进入系统，所以，我选择单独安装driver，然后，再安装cuda。解决重复登录界面参考[参考1](http://blog.csdn.net/u012759136/article/details/53355781)

1.[选择](http://www.nvidia.cn/Download/index.aspx?lang=cn)机子所需要的驱动，并下载。                  
2.卸载原有驱动：
```bash
$ sudo apt-get remove –purge nvidia*
```                                      
3.关闭nouveau                                   
创建 /etc/modprobe.d/blacklist-nouveau.conf并包含                  
```bash
blacklist nouveau
options nouveau modeset=0
```
在terminal输入
```bash
$ sudo update-initramfs -u
```     
4.进入命令行模式                            
Ctrl+Alt+F1              
5.关闭lightdm服务                    
```bash
$ sudo service lightdm stop
```
6.运行驱动文件
改变权限       
```bash
$ sudo chmod a+x NVIDIA-Linux-x86_64-375.39.run
```
运行  **注意参数**
```bash
sudo ./NVIDIA-Linux-x86_64-375.39.run –no-x-check –no-nouveau-check –no-opengl-files
```  

- no-x-check 安装驱动时关闭X服务
- no-nouveau-check 安装驱动时禁用nouveau
- no-opengl-files 只安装驱动文件，不安装OpenGL文件

7.重启，不出现循环登录问题


### 安装cuda

本来是按照deb安装的，后来各种问题，就改成选择runfile的方式了。

这里主要参考[参考2](http://docs.nvidia.com/cuda/cuda-installation-guide-linux/#runfile-nouveau-ubuntu)，全是英文的，要是不想看英文的话，我觉得，那还是放弃做dnn吧，目前这个前沿领域中文文献比较少～

1.在运行.run文件之后，在选择是否安装驱动的位置选择no，剩下的都是yes。                       
2.添加环境变量                      
打开 ～/.bashrc在最后添加
```bash
export PATH=/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib:$LD_LIBRARY_PATH
```
在terminal输入  
```bash
$ source ~/.bashrc
```
3.测试cuda安装是否成功
```bash
$ nvcc -V
```
输入cuda的版本信息                       
4.测试samples                      
这部分参考[参考3](http://blog.csdn.net/u012235003/article/details/54575758)
           
进入 NVIDIA_CUDA-8.0-Samples/
```bash
$ make
```
运行NVIDIA_CUDA-8.0-Samples/bin/x86_64/linux/release/deviceQuery

![pass](/images/project/2017-3-5/pass.png)

显示最后出现“Resalt=PASS”，代表成功

### 安装cuDNN

这个都不应该叫做安装，就是一个创建链接的过程。这个主要参考[参考4](http://blog.csdn.net/jk123vip/article/details/50361951)

1.下载tar文件，解压                         
解压出一个叫做cuda的文件夹，以下操作都是在该文件夹下进行                  
2.复制文件                     
```bash
$ sudo cp include/cudnn.h /usr/local/include
$ sudo cp lib64/libcudnn.so /usr/local/lib
$ sudo cp lib64/libcudnn.so.5 /usr/local/lib
$ sudo cp lib64/libcudnn.so.5.1.10 /usr/local/lib
```
3.创建链接
```bash
$ sudo ln -sf /usr/local/lib/libcudnn.so.5.1.10 /usr/local/lib/libcudnn.so.5
$ sudo ln -sf /usr/local/lib/libcudnn.so.5 /usr/local/lib/libcudnn.so
$ sudo ldconfig -v
```

### 安装tensorflow

根据[参考5](https://www.tensorflow.org/install/install_linux)，可以用的方式很多，我是按照那个virtaulenv的方式，可用。这个也是英文的，还是那句话，目前这块中文文献很少。



## 参考

1.[【解决】Ubuntu安装NVIDIA驱动后桌面循环登录问题](http://blog.csdn.net/u012759136/article/details/53355781)                                
2.[NVIDIA CUDA Installation Guide for Linux](http://docs.nvidia.com/cuda/cuda-installation-guide-linux/#runfile-nouveau-ubuntu)         
3.[二、CUDA安装和测试](http://blog.csdn.net/u012235003/article/details/54575758)                
4.[import TensorFlow提示Unable to load cuDNN DSO](http://blog.csdn.net/jk123vip/article/details/50361951)                        
5.[Installing TensorFlow on Ubuntu](https://www.tensorflow.org/install/install_linux)