---
title: "[Architecture] FPGA로 구현한 YOLO CNN3"
excerpt: "ShortcutFusion: From Tensorflow to FPGA-Based Accelerator With a Reuse-Aware Memory Allocation for Shortcut Data"

categories:
  - ARCHITECTURE
tags:
  - [FPGA, YOLO, CNN, object detection]

permalink: /categories/arch/yolo_cnn_3

date: 2022-04-12
last_modified_at: 2022-04-12
---
D. T. Nguyen, H. Je, T. N. Nguyen, S. Ryu, K. Lee, and H.-J. Lee, “ShortcutFusion: From Tensorflow to FPGA-Based Accelerator With a Reuse-Aware Memory Allocation for Shortcut Data,” IEEE Transactions on Circuits and Systems I: Regular Papers, pp. 1–13, 2022, doi: 10.1109/TCSI.2022.3153288.

- [Overview](#overview)
- [Motivation](#motivation)
- [Shortcut Fusion](#shortcut-fusion)
- [Reuse-Aware Shortcut Optimizer](#reuse-aware-shortcut-optimizer)

## Overview

## Motivation
- Shallow layers : feature map size $\uparrow$ and weight size $\downarrow$
- Deep layers : feature map size $\downarrow$ and weight size $\uparrow$
![Image1](/assets/images/shortcut_fusion_2022/shortcut_fusion-image-1.png){: . width="500px" .align-center}  

(a) **frame-based weight reuse** : 사용하면 큰 weight에 대해 input feature 전체에 reuse 할 수 있으므로 KxKxTi만큼의 kernel buffer만 있으면 input feature 전체에 대해 해당 kernel 계산은 끝낼 수 있다.  
단, N/Ti개의 kernel에 대해 동일한 input을 sequential하게 처리해야 되므로 input feature를 N/Ti번 읽어야 한다. 즉, input feature(HxHxTi)가 on-chip buffer에 전부 들어가는 경우에 효율이 극대화 됨.  

(b) **row-based weight reuse** :  row line에 대해 To에 대한 계산을 마쳐야 하므로 1개 row에 대해서만 weight reuse가 가능해 H번 weight를 다시 읽어 와야 한다.  
대신 row line에 대한 input feature만큼만 on-chip buffer에 두고 모든 kernel에 대한 계산을 끝내므로, 해당 input feature는 다시 읽을 필요가 없다.
![Image2](/assets/images/shortcut_fusion_2022/shortcut_fusion-image-2.png){: . width="400px" .align-center}  

즉, (a)는 deep layer에 적합하고, (b)는 shallow layer에 적합하다.  
ShortcutFusion은 CNN HW accelerator가 효율적으로 연산을 수행할 수 있도록 target model을 static하게 분석해서 instruction을 만들어 전달한다.  

CNN HW accelerator는 여러 가지 DNN structure를 지원할 수 있도록 configurable 해야한다.
- **depth wise conv** vs **normal conv**
- **frame-based weight reuse** vs **row-based weight reuse**
- **normal conv** vs **conv+shortcut fused together**


## Shortcut Fusion
![Image3](/assets/images/shortcut_fusion_2022/shortcut_fusion-image-3.png){: .align-center}  
![Image4](/assets/images/shortcut_fusion_2022/shortcut_fusion-image-4.png){: .align-center}  
![Image5](/assets/images/shortcut_fusion_2022/shortcut_fusion-image-5.png){: .align-center}  

FPGA에서 MAC을 DSP48E2 primitive를 사용해 conv 동작에 맞춰 병렬연산을 할 수 있도록 입력을 재구성해서 shared MAC을 구현했다.  
8bit data를 가정하고, MAC 하나 당 입력이 3개 pair가 들어간다.  
출력은 Mult0, Mult1 두 개 이지만, depth-wise conv 인 경우는 Mult0 출력만 사용한다.  

Depthwise conv 인 경우는 2개의 32 shared MAC에 대해 각각 따로 계산해서 각각의 out buffer에 저장 함.  
Normal conv는 다른 weight $W_0$, $W_1$에 대해 32개씩 interleaved 형태로 입력 넣어주고 출력에서 동일 weight 해당하는 것들을 더해서 각각의 output buffer에 저장.  
_[Q] 왜 interleaved 형태로 넣어주는지?_  
![Image6](/assets/images/shortcut_fusion_2022/shortcut_fusion-image-6.png){: . width="500px" .align-center}  
FPN과 같은 shortcut이 있는 경우 conv, element-wise adder를 따로 계산하지 않고 buffer에 한 번만 읽어와서 계산을 마치고 출력을 내 보내도록 instruction을 구성하는 방식으로 shortcut data reuse.
![Image7](/assets/images/shortcut_fusion_2022/shortcut_fusion-image-7.png){: . width="500px" .align-center}  
![Image8](/assets/images/shortcut_fusion_2022/shortcut_fusion-image-8.png){: . width="400px" .align-center}  

## Reuse-Aware Shortcut Optimizer