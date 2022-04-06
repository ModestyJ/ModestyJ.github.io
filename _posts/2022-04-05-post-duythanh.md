---
title: "[Architecture] FPGA로 구현한 YOLO CNN"
excerpt: "A High-Throughput and Power-Efficient FPGA Implementation of YOLO CNN for Object Detection"

categories:
  - ARCHITECTURE
tags:
  - [FPGA, YOLO, CNN, object detection]

permalink: /categories/arch/

date: 2022-04-05
last_modified_at: 2022-04-05
---
- [Abstract](#abstract)
- [Motivation](#motivation)
- [Proposed Architecture](#proposed-architecture)
  - [Quantization](#quantization)
  - [Data-path optimization](#data-path-optimization)
- [Result](#result)
- [Contributions](#contributions)

## Abstract
FPGA로 구현한 YOLO CNN 가속기
VC707 200Mhz에서 1.877TOPS
Key features : INT8
mAP 64.16%

## Motivation
기존 FPGA 구현 예를 보면 1) 중간 결과 feature map을 저장하기 위한 frequent off-chip access로 인한 performance degradation, 2) large buffer(e.g. interlayer double buffers) 사용으로 인한 resource 낭비의 trade-off 사이에서 최적화가 부족하다.

이 연구에서는 1) weight에 대해 1bit quantization 적용, 2) model에 따라 flexible한 low-bit activation(3~6bits)으로 retrain 하는 방법으로 적은 buffer 사용과 performance 사이에 적절한 최적화 YOLO CNN 모델을 FPGA로 구현했다.

## Proposed Architecture
### Quantization
- 기존에 알려진 1) Binary weight (1, -1), 2) low precision activation(3bit, 6bits)은 그대로 활용 -> model size를 30배, activation size 5.4배 줄일 수 있는 것으로 알려짐
- 기존 연구[8], [20]에 알려진 last fully connected layer가 low-precision에 민감하기 때문에 8bit fixed point quantization 채택
- 역시 low-precision에 민감한 batch normalization은 scale과 bias의 곱과 합 형태로 재정의 해서 16bit fixed-point로 활용
![Image1]({{site.url}}{{site.baseurl}}/assets/images/duythanh-image-1.jpg){: .align-center}
- Activation의 경우 shifter 구현을 위해 2^n step만 사용, bit 줄이기 위해 0은 제외한 symmetric quanzation 채택

### Data-path optimization
![Image2]({{site.url}}{{site.baseurl}}/assets/images/duythanh-image-2.png){: width="500px" : .align-center}
*Scheduling for streaming convolutional layer. (a) No weight reuse. (b) Fully weight reuse. (c) Proposed line-based weight reuse and input feature-maps fully reuse.*  
> N = Input channel size  
> M = Output channel size
> HxH = Input feature size  
> KxK = Kernel size  
> Ti = Input block depth  
> To = Output block depth  

- (a)는 partial sum이 output feature map에 계속 더해지는 형태로 weight는 reuse하지 않지만 accumulator를 위한 buffer가 필요없다. Input buffer size = K x N x H x Qa(input feature value bits)


## Result

## Contributions

