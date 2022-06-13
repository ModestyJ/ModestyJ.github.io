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

| Type | Layer   | \# params | Stride | in\_ch | in\_act\_width | \# mul    | \# add     | \# mul+add |
| ---- | ------- | --------- | ------ | ------ | -------------- | --------- | ---------- | ---------- |
| conv | 0       | 864       | 2      | 32     | 224            | 10838016  | 9633792    | 20471808   |
| dw   | 1       | 288       | 1      | 32     | 112            | 3612672   | 3211264    | 6823936    |
| pw   | 2       | 2048      | 1      | 32     | 112            | 25690112  | 24887296   | 50577408   |
| dw   | 3       | 576       | 2      | 64     | 112            | 1806336   | 1605632    | 3411968    |
| pw   | 4       | 8192      | 1      | 64     | 56             | 25690112  | 25288704   | 50978816   |
| dw   | 5       | 1152      | 1      | 128    | 56             | 3612672   | 3211264    | 6823936    |
| pw   | 6       | 16384     | 1      | 128    | 56             | 51380224  | 50978816   | 102359040  |
| dw   | 7       | 1152      | 2      | 128    | 56             | 903168    | 802816     | 1705984    |
| pw   | 8       | 32768     | 1      | 128    | 28             | 25690112  | 25489408   | 51179520   |
| dw   | 9       | 2304      | 1      | 256    | 28             | 1806336   | 1605632    | 3411968    |
| pw   | 10      | 65536     | 1      | 256    | 28             | 51380224  | 51179520   | 102559744  |
| dw   | 11      | 2304      | 2      | 256    | 28             | 451584    | 401408     | 852992     |
| pw   | 12      | 131072    | 1      | 256    | 14             | 25690112  | 25589760   | 51279872   |
| dw   | 13      | 4608      | 1      | 512    | 14             | 903168    | 802816     | 1705984    |
| pw   | 14      | 262144    | 1      | 512    | 14             | 51380224  | 51279872   | 102660096  |
| dw   | 15      | 4608      | 1      | 512    | 14             | 903168    | 802816     | 1705984    |
| pw   | 16      | 262144    | 1      | 512    | 14             | 51380224  | 51279872   | 102660096  |
| dw   | 17      | 4608      | 1      | 512    | 14             | 903168    | 802816     | 1705984    |
| pw   | 18      | 262144    | 1      | 512    | 14             | 51380224  | 51279872   | 102660096  |
| dw   | 19      | 4608      | 1      | 512    | 14             | 903168    | 802816     | 1705984    |
| pw   | 20      | 262144    | 1      | 512    | 14             | 51380224  | 51279872   | 102660096  |
| dw   | 21      | 4608      | 1      | 512    | 14             | 903168    | 802816     | 1705984    |
| pw   | 22      | 262144    | 1      | 512    | 14             | 51380224  | 51279872   | 102660096  |
| dw   | 23      | 4608      | 2      | 512    | 14             | 225792    | 200704     | 426496     |
| pw   | 24      | 524288    | 1      | 512    | 7              | 25690112  | 25639936   | 51330048   |
| dw   | 25      | 9216      | 2      | 1024   | 7              | 112896    | 100352     | 213248     |
| pw   | 26      | 1048576   | 1      | 1024   | 7              | 51380224  | 51330048   | 102710272  |
| FC   | 27      | 1024000   | \-     | 1024   | \-             | 1024000   | 1023000    | 2047000    |
| sum  | 4209088 | \-        |        | \-     | 568401664      | 562592792 | 1130994456 |

## Mobilenetv2


## Mobilenetv3
