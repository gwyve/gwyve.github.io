---
layout: post       
title: caffe添加新layer
category: blog
description: 在caffe上添加新类型的layer       
---


声明：本博客欢迎转发，但请保留原作者信息!      
作者：高伟毅    
博客：[https://gwyve.github.io/](https://gwyve.github.io/)    
微博：[http://weibo.com/u/3225437531/](http://weibo.com/u/3225437531/)    


## 引言

使用caffe有一段时间了，可是，目前使用的都是caffe自带的layer，随着自己对各种模型的熟悉，添加或修改现有layer的需求越来越大。

## 难度分析  

我认为对caffe的layer的修改，按照难度，可以分成以下几个阶段：                             
1. 只修改layer在cpu里的计算过程，同时不修改parameter，相当于单纯新添加一个layer                      
2. 修改layer在cpu计算过程，同时修改parameter                               
3. 修改layer在cudnn的计算过程                      
4. 修改layer在cuda的计算过程              

## 难度1

说明：                      
1. 这里使用最简单的mnist的例子，首先，保证caffe能够运行mnist的训练模型，即，运行./$CAFFE_ROOT/examples/mnist/train_lenet.sh能够出现下图的内容。
2. 这里添加的layer命名为VeLayer，主要是根据Conv换个名字

![first](/images/blog/2017-3-27/first.png)

### 1.复制conv_layer.cpp命名为ve_layer.cpp 以及 conv_layer.hpp -> ve_layer.hpp

```bush
$ cp src/caffe/layers/conv_layer.cpp src/caffe/layers/ve_layer.cpp
$ cp include/caffe/layers/conv_layer.hpp include/caffe/layers/ve_layer.hpp
```

### 2.修改ve_layer.hpp内容

开头的位置：#ifndef CAFFE_CONV_LAYER_HPP_ ->  #ifndef CAFFE_VE_LAYER_HPP_                      #define CAFFE_CONV_LAYER_HPP_ -> #define CAFFE_VE_LAYER_HPP_

类声明、构造函数声明：ConvolutionLayer -> VeLayer

type（）返回："Convolution" -> "Ve" 这个值改不改的意义不大，都可以运行


### 3.修改ve_layer.cpp内容

include : "caffe/layers/conv_layer.hpp" -> "caffe/layers/ve_layer.hpp"

函数属于类： ConvolutionLayer -> VeLayer

STUP_GPU: ConvolutionLayer -> VeLayer

INSTANTIATE: ConvolutionLayer -> VeLayer


### 4.在src/caffe/layer_factory.cpp修改

添加 #include "caffe/layers/ve_layer.hpp"

在namespace caffe 下添加:

```cpp
// Get Ve layer according to engine.
template <typename Dtype>
shared_ptr<Layer<Dtype> > GetVeLayer(
    const LayerParameter& param) {
  ConvolutionParameter conv_param = param.convolution_param();
  ConvolutionParameter_Engine engine = conv_param.engine();
#ifdef USE_CUDNN
  bool use_dilation = false;
  for (int i = 0; i < conv_param.dilation_size(); ++i) {
    if (conv_param.dilation(i) > 1) {
      use_dilation = true;
    }
  }
#endif
  if (engine == ConvolutionParameter_Engine_DEFAULT) {
    engine = ConvolutionParameter_Engine_CAFFE;
#ifdef USE_CUDNN
    if (!use_dilation) {
      engine = ConvolutionParameter_Engine_CUDNN;
    }
#endif
  }
  if (engine == ConvolutionParameter_Engine_CAFFE) {
    return shared_ptr<Layer<Dtype> >(new VeLayer<Dtype>(param));
#ifdef USE_CUDNN
  } else if (engine == ConvolutionParameter_Engine_CUDNN) {
    if (use_dilation) {
      LOG(FATAL) << "CuDNN doesn't support the dilated VE at Layer "
                 << param.name();
    }
    return shared_ptr<Layer<Dtype> >(new CuDNNConvolutionLayer<Dtype>(param));
#endif
  } else {
    LOG(FATAL) << "Layer " << param.name() << "  VEVEVEVE has unknown engine.";
    throw;  // Avoids missing return warning
  }
}

REGISTER_LAYER_CREATOR(Ve, GetVeLayer);
```

return shared_ptr<Layer<Dtype> >(new VeLayer<Dtype>(param));

这里REGISTER_LAYER_CREATOR里面的第一个参数是prototxt里面type的参数



### 5. 编译

make 

### 6. 修改examples/mnist/lenet_train_test.prototxt

把第二个conv层，改成Ve

type: "Convolution" -> "Ve"  这个“VE”是跟第4步的REGISTER_LAYER_CREATOR相呼应的。

### 7. 运行

./$CAFFE_ROOT/examples/mnist/train_lenet.sh 结果如图

![second](/images/blog/2017-3-27/second.png)

## 难度2

这里依照难度1说的VeLayer，添加新的parameter

### 1. make clean

清除难度1编译成功的文件

### 2. 修改proto文件


1. 在src/caffe/proto/caffe.proto的message V1LayerParameter {的enum LayerType {添加

VE = 40;  //这个具体的ID根据实际更改的,并有觉得这一步有什么用，具体什么用，后面再看

2. 在src/caffe/proto/caffe.proto的message LayerParameter { 添加

optional VeParameter ve_param = 147;

3. 在src/caffe/proto/caffe.proto添加

```proto

message VeParameter {
  optional uint32 num_output = 1; // The number of outputs for the layer
  optional bool bias_term = 2 [default = true]; // whether to have bias terms

  // Pad, kernel size, and stride are all given as a single value for equal
  // dimensions in all spatial dimensions, or once per spatial dimension.
  repeated uint32 pad = 3; // The padding size; defaults to 0
  repeated uint32 kernel_size = 4; // The kernel size
  repeated uint32 stride = 6; // The stride; defaults to 1
  // Factor used to dilate the kernel, (implicitly) zero-filling the resulting
  // holes. (Kernel dilation is sometimes referred to by its use in the
  // algorithme à trous from Holschneider et al. 1987.)
  repeated uint32 dilation = 18; // The dilation; defaults to 1

  // For 2D convolution only, the *_h and *_w versions may also be used to
  // specify both spatial dimensions.
  optional uint32 pad_h = 9 [default = 0]; // The padding height (2D only)
  optional uint32 pad_w = 10 [default = 0]; // The padding width (2D only)
  optional uint32 kernel_h = 11; // The kernel height (2D only)
  optional uint32 kernel_w = 12; // The kernel width (2D only)
  optional uint32 stride_h = 13; // The stride height (2D only)
  optional uint32 stride_w = 14; // The stride width (2D only)

  optional uint32 group = 5 [default = 1]; // The group size for group conv

  optional FillerParameter weight_filler = 7; // The filler for the weight
  optional FillerParameter bias_filler = 8; // The filler for the bias
  enum Engine {
    DEFAULT = 0;
    CAFFE = 1;
    CUDNN = 2;
  }
  optional Engine engine = 15 [default = DEFAULT];

  // The axis to interpret as "channels" when performing convolution.
  // Preceding dimensions are treated as independent inputs;
  // succeeding dimensions are treated as "spatial".
  // With (N, C, H, W) inputs, and axis == 1 (the default), we perform
  // N independent 2D convolutions, sliding C-channel (or (C/g)-channels, for
  // groups g>1) filters across the spatial axes (H, W) of the input.
  // With (N, C, D, H, W) inputs, and axis == 1, we perform
  // N independent 3D convolutions, sliding (C/g)-channels
  // filters across the spatial axes (D, H, W) of the input.
  optional int32 axis = 16 [default = 1];

  // Whether to force use of the general ND convolution, even if a specific
  // implementation for blobs of the appropriate number of spatial dimensions
  // is available. (Currently, there is only a 2D-specific convolution
  // implementation; for input blobs with num_axes != 2, this option is
  // ignored and the ND implementation will be used.)
  optional bool force_nd_im2col = 17 [default = false];
}

```


### 3. 修改layer_factory.cpp

将难度1添加的内容改成

``` cpp
// Get Ve layer according to engine.
template <typename Dtype>
shared_ptr<Layer<Dtype> > GetVeLayer(
    const LayerParameter& param) {
  VeParameter ve_param = param.ve_param();
  VeParameter_Engine engine = ve_param.engine();
#ifdef USE_CUDNN
  bool use_dilation = false;
  for (int i = 0; i < ve_param.dilation_size(); ++i) {
    if (ve_param.dilation(i) > 1) {
      use_dilation = true;
    }
  }
#endif
  if (engine == VeParameter_Engine_DEFAULT) {
    engine = VeParameter_Engine_CAFFE;
#ifdef USE_CUDNN
    if (!use_dilation) {
      engine = ConvolutionParameter_Engine_CUDNN;
    }
#endif
  }
  if (engine == ConvolutionParameter_Engine_CAFFE) {
    return shared_ptr<Layer<Dtype> >(new VeLayer<Dtype>(param));
#ifdef USE_CUDNN
  } else if (engine == ConvolutionParameter_Engine_CUDNN) {
    if (use_dilation) {
      LOG(FATAL) << "CuDNN doesn't support the dilated VE at Layer "
                 << param.name();
    }
    return shared_ptr<Layer<Dtype> >(new CuDNNConvolutionLayer<Dtype>(param));
#endif
  } else {
    LOG(FATAL) << "Layer " << param.name() << "  VEVEVEVE has unknown engine.";
    throw;  // Avoids missing return warning
  }
}

REGISTER_LAYER_CREATOR(Ve, GetVeLayer);
```

### 4. 编译

make

### 5. 运行

./$CAFFE_ROOT/examples/mnist/train_lenet.sh 结果如图

![third](/images/blog/2017-3-27/third.png)


## 难度3   待补充



