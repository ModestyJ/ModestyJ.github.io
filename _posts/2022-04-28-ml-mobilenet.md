---
title: "[ML] Mobilenet"
excerpt: "mobilenet"

categories:
  - MACHINE LEARNING
tags:
  - [mobilenet, CNN]

permalink: /categories/ml/mobilenet

date: 2022-04-22
last_modified_at: 2022-04-28
---
A. G. Howard et al., “MobileNets: Efficient Convolutional Neural Networks for Mobile Vision Applications,” arXiv:1704.04861 [cs], Apr. 2017, Accessed: Apr. 28, 2022. [Online]. Available: http://arxiv.org/abs/1704.04861

- [Overview](#overview)
- [Mobilenetv2](#mobilenetv2)
- [Mobilenetv3](#mobilenetv3)

## Overview
Depthwise Separable Convolution을 적용한 CNN architecture로 image feature map 추출을 적은 parameter, 연산량으로 할 수 있다. Object detection(SSD)을 포함한 다양한 vision task에 대해 좋은 성능을 확인 했음.


Depthwise Separable Convolution은 [여기](https://eli.thegreenplace.net/2018/depthwise-separable-convolutions-for-machine-learning) 잘 정리 됨.
[![Image0](https://eli.thegreenplace.net/images/2018/conv2d-depthwise-separable.svg){: . width="500px" .align-center}](https://eli.thegreenplace.net/2018/depthwise-separable-convolutions-for-machine-learning)

각 channel에 대해 depthwise convolutional filter를 적용하고(# channel N->N), 그 결과에 대해 1x1 pointwise convolution을 적용한다.(# channel N->M)
> M = Input channel size  
> N = Output channel size  
> $D_K$ = Kernel size  
> $D_F$ = Input feature size  

![Image2](/assets/images/mobilenet/mobilenet-image-2.png){: . width="500px" .align-center}  

기존 conv layer 대비 연산 수는 아래 만큼 줄어든다.
![Image1](/assets/images/mobilenet/mobilenet-image-1.png){: . width="500px" .align-center}  
![Image3](/assets/images/mobilenet/mobilenet-image-3.png){: . width="500px" .align-center}  

실제 layer 별 parameter 수를 계산 해 보면 다음과 같음. 8bits weight에 대해 약 3MB 필요.
일반적인 conv는 첫 번째 layer에만 사용 되고, pooling도 마지막에 한 번만 있다. 1x1 pointwise convolution parameter가 가장 많은 연산을 차지한다.
![Image4](/assets/images/mobilenet/mobilenet-image-4.png){: . width="600px" .align-center}  

## Mobilenetv2


## Mobilenetv3
