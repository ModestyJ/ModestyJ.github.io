---
title: "[Architecture] FPGA로 구현한 YOLO CNN"
excerpt: "A High-Throughput and Power-Efficient FPGA Implementation of YOLO CNN for Object Detection"

categories:
  - ARCHITECTURE
tags:
  - [FPGA, YOLO, CNN, object detection]

permalink: /categories/arch/yolo_cnn

date: 2022-04-05
last_modified_at: 2022-04-09
---
D. T. Nguyen, T. N. Nguyen, H. Kim, and H.-J. Lee, “A High-Throughput and Power-Efficient FPGA Implementation of YOLO CNN for Object Detection,” IEEE Transactions on Very Large Scale Integration (VLSI) Systems, vol. 27, no. 8, pp. 1861–1873, Aug. 2019, doi: 10.1109/TVLSI.2019.2905242.

- [Overview](#overview)
- [Motivation](#motivation)
- [Algorithmic Optimization for the Proposed Streaming Architecture](#algorithmic-optimization-for-the-proposed-streaming-architecture)
  - [Quantization](#quantization)
  - [Data-path optimization](#data-path-optimization)
- [Proposed Architecture](#proposed-architecture)
  - [Streaming desing of the convolutional layer](#streaming-desing-of-the-convolutional-layer)
- [Result](#result)

## Overview
실시간 object detection을 위해 TPU와 같은 일반적인 systolic array 구조로 MAC을 펼치는게 아니라, layer by layer pipelining을 위한 streaming architecture를 제안했다.

Image width line 전체에 대해 output feature 계산을 마치고 한 줄 한 줄 내려가는 data flow 및 이를 지원하기 위한 cycle buffer, line buffer에 대한 구현상 디테일을 확인할 수 있다.
Binary weight, low precision activation과 같은 quantization feature를 채택해서 **batch norm 부분과 last FC layer를 제외하고는 multiplier가 필요 없다**.

**각 layer의 line 단위로 최종 output 계산이 끝나기 때문에 layer by layer pipeline이 가능**하고 한 line에 대한 layer buffer를 두어서 input과 output buffer가 overlap 될 수 있다.
**output buffer가 곧 다음 layer의 input buffer로 사용**되기 때문에 feature에 대한 dram access가 없으면서도 전체 image에 대한 기존 double buffer 방식이 아니기 때문에 상대적으로 memory 사용량도 적다.

VC707 FPGA로 구현해서 200Mhz frequency로  sim-YOLO-v2 network에 대해 batch를 최대한 활용한 경우 약 1.8TOPS, batch 없이 1TOPS 정도 성능을 확인했다. Power는 각각 11.11W, 18.29W라고 함.
mAP accuracy는 Full precision YOLO-v2 75.88%, Sim-YOLO-v2 72%일 때 Sim-YOLO-v2에서 2 layer 제거하고 binary weight, 4-6 bit activation 적용한 net에 대해 64.16%라고 함.

## Motivation
기존 FPGA 구현 예를 보면 1) 중간 결과 feature map을 저장하기 위한 frequent off-chip access로 인한 performance degradation, 2) large buffer(e.g. interlayer double buffers) 사용으로 인한 resource 낭비의 trade-off 사이에서 최적화가 부족하다.

이 연구에서는 1) weight에 대해 1bit quantization 적용, 2) model에 따라 flexible한 low-bit activation(3~6bits)으로 retrain 하는 기존 quantization feature를 영끌해서 weight 크기 자체를 줄여서 weight reuse는 적당히(line 단위) 해도 부담이 안된다고 전제 했음.

key feature는 image line 단위로 전체 channel에 대해 연산을 수행하고, 그 결과를 layer to layer overlap 가능하도록 buffer를 활용해 상대적으로 적은 interlayer buffer size로 pipeline을 가능하게 했다는 것이다.

## Algorithmic Optimization for the Proposed Streaming Architecture
### Quantization
- 기존에 알려진 1) Binary weight (1, -1), 2) low precision activation(3bit, 6bits)은 그대로 활용 -> model size를 30배, activation size 5.4배 줄일 수 있는 것으로 알려짐
- 기존 연구[8], [20]에 알려진 last fully connected layer가 low-precision에 민감하기 때문에 8bit fixed point quantization 채택
- 역시 low-precision에 민감한 batch normalization은 scale과 bias의 곱과 합 형태로 재정의 해서 16bit fixed-point로 활용  
![Image1](/assets/images/yolo_cnn_vlsi_2019/duythanh-image-1.jpg){: .align-center}  
- Activation의 경우 shifter 구현을 위해 $2^n$ step만 사용, bit 줄이기 위해 0은 제외한 symmetric quanzation 채택

### Data-path optimization
![Image2](/assets/images/yolo_cnn_vlsi_2019/duythanh-image-2.png){: width="500px" : .align-center}  
*Scheduling for streaming convolutional layer. (a) No weight reuse. (b) Fully weight reuse. (c) Proposed line-based weight reuse and input feature-maps fully reuse.*  
> N = Input channel size  
> M = Output channel size
> HxH = Input feature size  
> KxK = Kernel size  
> Ti = Input block depth  
> To = Output block depth  

- **(a) Input feature reuse:** 특정 Output feature 채널을 계산하는데 필요한 input channel wise convolution 계산을 마침, 대신 input feature은 buffer에 저장해 두고, weight는 필요할 때 마다 읽어옴
  * Input buffer size = $K * N * H * Qa$(bits for input feature value)
  * Weight read = $H^2$
- **(b) Weight reuse:** 특정 weight가 필요한 특정 채널의 input feature 전체에 대한 계산을 마침, 대신 output channel에 대한 계산이 아직 끝나지 않았으므로 partial sum을 위한 output buffer가 필요, input feature map을 M/To times 읽어올 수는 없으므로 이를 buffer에 저장할 필요가 있음
  * Output buffer size(doubled for pipeline) = $2 * To * H^2 * Qs$(bits for accumulation before quantization)
  * Input buffer size(doubled for pipeline) = $2 * H^2 * N * Qa$(bits for input feature value)
- **(c) Proposed(weight partially reuse + input feature fully reuse):** input feature의 row에 대해서만 weight reuse, input feature map을 fully reuse
  * Line buffer for entire weight blocks: H times read, but latency could be hided by prefetch the next line  
  -> parallel 연산하는 Ti, To 단위 Block에 대해 미리 SRAM에 prefetch
  * Input buffer size = $(K+1) * N * H * Qa$  
  -> K+1에서 1은 이전 layer output이 쌓이는 공간임
  * Temporarily accumulation buffer size = $To * H * Qs$  
  -> To에 대해서는 input channel 전체에 대한 연산이 끝남, Ti/N번 계산 된 결과가 누적해서 더해져야 하기에 버퍼 필요

(a)는 weight read 가 너무 빈번하고, (b)는 input/output buffer size가 너무 크다.

_[Q] batch mode에서 (b) buffer linear하게 증가, (c) hardware resource 증가 없다고 했는데 (b)에서 buffer가 linear하게 증가하는 이유가 무엇인지?_  
_[A] (b)는 layer 단위로 image feature 전체 연산이 끝나야 다음으로 넘어간다는 전제가 있는 듯._

## Proposed Architecture
### Streaming desing of the convolutional layer
![Image3](/assets/images/yolo_cnn_vlsi_2019/duythanh-image-3.png){: width="500px" : .align-center}  
![Image4](/assets/images/yolo_cnn_vlsi_2019/duythanh-image-4.png){: width="500px" : .align-center}  
![Image5](/assets/images/yolo_cnn_vlsi_2019/duythanh-image-5.jpg){: width="600px" : .align-center}  
![Image6](/assets/images/yolo_cnn_vlsi_2019/duythanh-image-6.jpg){: width="600px" : .align-center}  
![Image7](/assets/images/yolo_cnn_vlsi_2019/duythanh-image-7.png){: width="500px" : .align-center}  
![Image8](/assets/images/yolo_cnn_vlsi_2019/duythanh-image-8.png){: width="500px" : .align-center}  

## Result
![Image9](/assets/images/yolo_cnn_vlsi_2019/duythanh-image-9.png){: width="500px" : .align-center}  
![Image10](/assets/images/yolo_cnn_vlsi_2019/duythanh-image-10.png){: width="500px" : .align-center}  
![Image11](/assets/images/yolo_cnn_vlsi_2019/duythanh-image-11.png){: width="500px" : .align-center}  
![Image12](/assets/images/yolo_cnn_vlsi_2019/duythanh-image-12.png){: width="500px" : .align-center}  

_[Q] power, TOPS는 어떻게 측정?_
