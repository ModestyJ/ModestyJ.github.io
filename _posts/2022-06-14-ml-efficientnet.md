---
title: "[ML] EfficientNet"
excerpt: "efficientnet"

categories:
  - MACHINE LEARNING
tags:
  - [efficientnet, CNN, compound scaling]

permalink: /categories/ml/efficientnet

date: 2022-06-14
last_modified_at: 2022-06-14
---
M. Tan and Q. V. Le, “EfficientNet: Rethinking Model Scaling for Convolutional Neural Networks,” arXiv, arXiv:1905.11946, Sep. 2020. doi: 10.48550/arXiv.1905.11946.

- [Overview](#overview)
- [EfficientDet](#efficientdet)
- [EfficientNetV2](#efficientnetv2)

## Overview
Accuracy와 같은 모델 performance 관점에서 DNN 모델들의 발전사를 보면 Resnet의 skip-connection과 같이 기존과 다른 형태도 있을 수 있지만, 큰 흐름은 모델 사이즈가 커진다는 공통점이 있었다.  
크기를 키워서 성능 확보 -> Accuracy drop 최소화 하면서 연산량 줄이기 위한 노력 이런 식의 반복이지 않았나 싶다.  

EfficientNet은 애초에 모델 크기를 키울 때 효율적으로 키우는 방법을 제시하는데, 결과가 아주 좋아서 처음 공개됐을 때 Network Architecture Search (NAS)쪽에 굉장히 좋은 인사이트를 주지 않았을까 싶다. 물론 모델 사이즈가 HW resource에 limit 된다는 점에서 HW accelerator 설계 관점에서도 중요한 내용이기도 하다.    

모델을 키우는 방법으로 key idea인 compound scaling을 제안했다. 기존에 알려진 모델을 block 단위로 반복한다고 할 때, depth, width, input resoultion의 세 가지로 한정해서 적절한 비율로 동시에 키울 때 좋은 성능을 보이더라는 내용.  

아래 그래프를 보면 imagenet accuracy를 기준으로 기존 SOTA model에 비해 크기는 월등히 작고 accuracy는 높더라는 결과를 보여준다.  
![Image1](/assets/images/efficientnet/efficientnet-image-1.png){: . width="500px" .align-center}  

Compound scailing은 width, depth, resolution을 동시에 키우는 방법이다. 이 때 어려운 점은 어느 것을 얼마나 키워야 하는가이다.  
![Image2](/assets/images/efficientnet/efficientnet-image-2.png){: . width="500px" .align-center}  


![Image3](/assets/images/efficientnet/efficientnet-image-3.png){: . width="500px" .align-center}  
![Image4](/assets/images/efficientnet/efficientnet-image-4.png){: . width="500px" .align-center}  
![Image5_1](/assets/images/efficientnet/efficientnet-image-5_1.png){: . width="500px" .align-center}  
![Image5_2](/assets/images/efficientnet/efficientnet-image-5_2.png){: . width="500px" .align-center}  
![Image6](/assets/images/efficientnet/efficientnet-image-6.png){: . width="500px" .align-center}  
![Image7](/assets/images/efficientnet/efficientnet-image-7.png){: . width="500px" .align-center}  
![Image8](/assets/images/efficientnet/efficientnet-image-8.png){: . width="500px" .align-center}  
![Image9](/assets/images/efficientnet/efficientnet-image-9.png){: . width="500px" .align-center}  
![Image10](/assets/images/efficientnet/efficientnet-image-10.png){: . width="500px" .align-center}  
![Image11](/assets/images/efficientnet/efficientnet-image-11.png){: . width="500px" .align-center}  


## EfficientDet


## EfficientNetV2
