---
title: "[Architecture] EYERISS"
excerpt: "Efficient Processing of Deep Neural Networks: A Tutorial and Survey"

categories:
  - ARCHITECTURE
tags:
  - [DATAFLOW, Row-stationary, CNN, accelerator]

permalink: /categories/arch/eyeriss

date: 2022-09-26
last_modified_at: 2022-09-26
---
- [Overview](#overview)
- [ROW-STATIONARY DATAFLOW](#row-stationary-dataflow)
  - [A.	Configurable Dataflow to Maximize the Reuse for the Energy Efficiency](#aconfigurable-dataflow-to-maximize-the-reuse-for-the-energy-efficiency)
  - [B. Implementation of the Row-Stationary in Eyeriss](#b-implementation-of-the-row-stationary-in-eyeriss)
  - [C. Energy Efficiency](#c-energy-efficiency)
- [DISCUSSION](#discussion)
  - [A. Criticism about the Row-Stationary Dataflow](#a-criticism-about-the-row-stationary-dataflow)
    - [1.	Control logic is complicated compared to other dataflows and distributed across PEs.](#1control-logic-is-complicated-compared-to-other-dataflows-and-distributed-across-pes)
    - [2.	Additional On-chip network design overhead is required to support row-stationary dataflow efficiently.](#2additional-on-chip-network-design-overhead-is-required-to-support-row-stationary-dataflow-efficiently)
    - [3.	It is tightly optimized only for CNN workloads.](#3it-is-tightly-optimized-only-for-cnn-workloads)
    - [4.	It is difficult to analyze dataflow with analytic model](#4it-is-difficult-to-analyze-dataflow-with-analytic-model)
    - [5.	A lack of consideration for global buffer level reuse (i.e., temporal reuse).](#5a-lack-of-consideration-for-global-buffer-level-reuse-ie-temporal-reuse)
  - [B. Comparing dataflows](#b-comparing-dataflows)
    - [1.	The row-stationary is more flexible and configurable.](#1the-row-stationary-is-more-flexible-and-configurable)
    - [2.	The row-stationary is highly customized to CNN workload and design of PE](#2the-row-stationary-is-highly-customized-to-cnn-workload-and-design-of-pe)
- [CONCLUSION](#conclusion)
- [REFERENCES](#references)
  

# Overview
The Eyeriss project [1] is a well-known study about an energy-efficient deep convolutional neural network (CNN) acceleration architecture. It was published by the energy-efficient multimedia systems (EEMS) group at MIT in 2016 and proven by chip fabrication in [2], [3]. They contribute comprehensively explaining the characteristics of CNN computation and hardware optimization. In particular, the paper [4] is a well-organized survey work to understand the state-of-the-art DNN accelerators. It covers a wide range from the history of DNNs to various design considerations for efficient accelerators. In this summary report, dataflow-centric design considerations pointed out in [4] will be summarized for the efficient processing of DNNs. The novel dataflow called row-stationary (RS) is mainly described in section II. In addition, criticism about the row-stationary dataflow and other design considerations introduced in [1] are discussed in section III.

# ROW-STATIONARY DATAFLOW
## A.	Configurable Dataflow to Maximize the Reuse for the Energy Efficiency

The row-stationary dataflow is a major contribution of the Eyeriss project in terms of energy efficiency. The key idea of the row-stationary dataflow is as follows.
Any 2-D convolution can be expressed with aggregations of 1-D convolutions.

Based on this hypothesis, it is possible that row-unit reuse can be maximized by residing any type of data such as weights, input activations, and partial sums at the register file (RF) inside the PE. It is different from other dataflow such as weight-stationary (WS), output-stationary (OS), and input-stationary (IS) in terms of flexibility because resided data type is fixed in the case of other dataflows. For a single PE, reusable filter weights reside inside of the RF like WS and they are reused to calculate partial sums with streamed input activations for a row. As a result, calculated partial sums are accumulated. This is the first advantage of the row-stationary dataflow in terms of data reuse.

Second advantage of the dataflow is spatial data reuse in 2-D PE array. Figure 1 shows how spatial reuse works for each data type. Firstly, a pair of weights and input activations for a row should be provided to each PE. To exploit the systolic array style on-chip network, weights are passed from left to right through the PE array. Meanwhile, input activations coupled with the weights are passed diagonal direction through the PE array as well. Lastly, the results of 1-D convolutions (i.e., the partial sums) in each PE are accumulated from south to north direction through the PE array. As a result, three PEs in each column of the PE array generate an output of 1-D convolution for a row and three 1-D convolutions from each column are aggregated to complete 2-D convolution. These spatial reuses reduce the number of accesses to a global on-chip buffer shared by all PEs. Because access to a global buffer is more expensive than a local RF in a PE, spatial reuses have an effect on overall energy efficiency.

![Image1](/assets/images/eyeriss/eyeriss-image-1.png){: . width="500px" .align-center}
*Figure 1 Spatial reuse for 2-D systolic array for row-stationary dataﬂow*

In addition to the 2-D convolution reuses, multiple rows can be mapped to same PE to calculate real CNN works including multiple features (i.e., batches), multiple filters for outputs, and multiple channels. It makes room to optimize efficient dataflow for CNN works. Unlike other fixed dataflows such as WS, OS, and IS, the row-stationary can select optimal mapping depending on the characteristics of network layer by static analysis like a compiler. Figure 2 shows that three mapping examples onto a single PE. The first one is that multiple input feature maps are mapped to a PE in batch mode. In this case, the same weights can be reused for each input row for different input feature maps, and then separate partial sums are calculated sequentially. The second one is that an input feature map is reused for output channel weights. The last one is for exploiting reuse of output partial sum. Pairs of multiple weights and input for input channel wise generate a partial sum. In this case, an accumulated partial sum can be resided an RF and reused.
 

![Image2](/assets/images/eyeriss/eyeriss-image-2.png){: . width="500px" .align-center}
*Figure 2 Mapping examples to same PE for additional reuse in the row-stationary dataﬂow [1]*

## B. Implementation of the Row-Stationary in Eyeriss

 
![Image3](/assets/images/eyeriss/eyeriss-image-3.png){: . width="500px" .align-center}
*Figure 3 Architecture of Eyeriss DNN accelerator [2]*

The test chip of Eyeriss shows in Figure 3 was implemented in 65nm CMOS. In the view of dataflow, an important design consideration is the size of PE array. Because the chip consists of a fixed size 14x12 PE array, one challenge is to map real DNN workload to the PE array. Eyeriss adopts the strategy called replication and folding for workload adaptive mapping. Figure 4 shows an example of AlexNet workload. Layers 3-5 in AlexNet consists of 3x3 weights and 13x13 input activations. In this case, PE arrays are divided into four 3x13 arrays to map the workload as a replication way. In the case of Layer 2, weights and input activations are 5x5 and 27x27 respectively. Because size of input activations is larger than width of PE array, PE arrays with 5x14 and 5x13 are used as a folding way.

 
![Image4](/assets/images/eyeriss/eyeriss-image-4.png){: . width="500px" .align-center}
*Figure 4 Mapping Example for AlexNet to maximized utilization of PE array [2]*

## C. Energy Efficiency

Considering advantages of the row-stationary dataflow, it is expected optimizing local RF level data reuse and overall energy efficiency. Figure 5 shows experimental results to analyze energy efficiency for AlexNet. In terms of off-chip DRAM access (i.e., most driving factor for power consumption), the no local reuse (NLR) case is best. However, it requires a large size of global on-chip SRAM buffer and heavy accesses. In the case of the row-stationary, the majority of energy consumption occurs at the RF level, because the local RF level data reuse is maximized. As a result, overall energy efficiency is best in the row-stationary case as the expectations. The row-stationary dataflow is configurable for the data types to reuse, so energy breakdown across data types could be different depending on static optimization like a compiler.
 	 
![Image5](/assets/images/eyeriss/eyeriss-image-5.png){: . width="500px" .align-center}
*Figure 5 Comparison of energy efficiency between dataflows in the convolution layers of AlexNet with a batch size of 16 [1]*

# DISCUSSION

## A. Criticism about the Row-Stationary Dataflow

Even though the row-stationary dataflow is very energy efficient for CNN computations, there is room for improvement. Followings are five critiques about intrinsic drawbacks of the dataflow and things that were not considered at the time of publication for original Eyeriss.
### 1.	Control logic is complicated compared to other dataflows and distributed across PEs.  
 
Configurability to select reuse data type (i.e., weight, input activation, partial sum) is an advantage for the row-stationary. However, control logics distributed across all PEs are required to support that configuration. That makes it difficult to minimize chip area. Furthermore, a compiler should map the workload to each PEs and generate all instructions to control all PEs individually. It is quite complicated compared to SIMD based accelerators.

### 2.	Additional On-chip network design overhead is required to support row-stationary dataflow efficiently.  
Even though a systolic array structure has advantages to implementing routing friendly network, row-stationary which supports various data access patterns has design overhead to implementing on-chip network. Because data access patterns are varying depends on which data type will be reused in the row-stationary, on-chip network should provide target data depending on the reuse data type. For example, Figure 6 is the global input network (GIN) to arbitrate to identify data traffics. They adopted multi-cast network on chip (NoC) with additional arbitration logic like GIN. If the data access pattern is fixed and simple like other dataflows, that kind of additional logic is not required. EyerissV2 [5] proposed some enhancements for exploiting the sparsity and scalable and flexible NoC. In particular, the flexible NoC helps to alleviate the memory bandwidth problem in the on-chip network level. (c.f., exploiting sparsity has effect on reducing required bandwidth it self)
 
![Image6](/assets/images/eyeriss/eyeriss-image-6.png){: . width="500px" .align-center}
*Figure 6 Architecture of the global input network (GIN) in Eyeriss [3]*

### 3.	It is tightly optimized only for CNN workloads.  
As described in section II-A, the key idea of the row-stationary dataflow is that any 2-D convolution can be expressed with aggregations of 1-D convolutions. In other words, it’s intrinsically optimized for convolutional computation. However, nowadays, fully connected (FC) based recommender systems and the transformer-based natural language processing (NLP) are popular DNN workloads. In the case of FC, Figure 7 shows the characteristics of FC computation. There is no chance to reuse in weights and the majority of energy consumption is by off-chip DRAM access. It means memory bandwidth requirements could be higher than CNN. If reuse chance is low like FC, complex control logic described in 1 and NoC level overhead described in 2 are just waste of resources.
 	 
![Image7](/assets/images/eyeriss/eyeriss-image-7.png){: . width="500px" .align-center}
*Figure 7 Comparison of energy efficiency in FC layers (left), Energy breakdown across layers of the AlexNet (right). [1]*

### 4.	It is difficult to analyze dataflow with analytic model  
Even though the row-stationary dataflow is energy-efficient, it is not widely used in recent accelerators. One reason why it has not been used is that the exploration of design space by analytic models is not that simple for the row-stationary dataflow. Almost all systolic array-based cycle-accurate simulators only support other dataflows such as WS, IS, and OS. For example, SCALE-Sim [6] just supports WS, IS, and OS because the row-stationary is tightly coupled with the PE design. For example, it is possible to design exploration for each dataflow and PE array size demonstrated in Figure 8. Using that kind of analytic model, various design considerations could be decided with fewer resources in a short time.
 
![Image8](/assets/images/eyeriss/eyeriss-image-8.png){: . width="500px" .align-center}
*Figure 8 Design Exploration for Dataflows and Array Shape of the Systolic Array in BERT4Rec using SCALE-Sim*

### 5.	A lack of consideration for global buffer level reuse (i.e., temporal reuse).  
The row-stationary dataflow is well optimized for the local RF level reuse. In contrast, global buffer level temporal reuse is rarely considered. For example, ShortcutFusion [7] is well optimized for global buffer level reuse. In particular, shortcut connection which is widely used in recent DNNs similar to ResNet occupies a large portion of off-chip DRAM accesses. Shortcut Mining [8] pointed out that the shortcut connection accounts for 40% of the feature maps access in Resnet-152. If the shortcut data resided in the global on-chip buffer by the next consuming layer, the amount of shortcut data for one write and one read to access off-chip DRAM could be saved.

## B. Comparing dataflows
Main difference between the row-stationary and other dataflows such as WS, IS, and OS is flexibility.
### 1.	The row-stationary is more flexible and configurable.  
As described in section II-A, the row-stationary computes 1-D convolution and aggregates the results. For the computation of 1-D convolution, the row-stationary can decide to reuse data type among weights, input activations, and output partial sum. On the other hand, WS, IS, and OS has fixed target reuse data. Thanks to this flexibility, the row-stationary can efficiently map special layers like depth-wise convolutional layers of MobileNet without underutilization issues. Even though the fact that the total amount of computation is small for the depth-wise layers is an advantage, channel wise parallelization is quite low. Because of this reason, other dataflows suffer from underutilization issues in the depth-wise convolutional layers.
### 2.	The row-stationary is highly customized to CNN workload and design of PE  
As described in section III-A, additional design is required to support the row-stationary efficiently such as control logic inside each PE, the global input network (GIN), and so on. This is a disadvantage for area optimization. Furthermore, the concept that aggregation of 1-D convolutions is not suitable to support recent memory-intensive workloads like recommender system, NLP workload.  

# CONCLUSION  
In this survey report, the row-stationary dataflow adopted in the Eyeriss project was mainly reviewed in terms of energy efficiency. The row-stationary dataflow is well optimized for CNN workload by aggregation of 1-D convolution to maximize data reuse in the local register file level. Because reuse data types can be selected depending on the characteristics of the target layer, there is more room to optimize data reuse than in other dataflows. It is very energy efficient due to the data reuse, on the other hand, additional design overhead exists and it is overly dependent on PE design and convolutional workloads.

# REFERENCES  
[1]	Y.-H. Chen, J. Emer, and V. Sze, “Eyeriss: A Spatial Architecture for Energy-Efficient Dataflow for Convolutional Neural Networks,” in 2016 ACM/IEEE 43rd Annual International Symposium on Computer Architecture (ISCA), Jun. 2016, pp. 367–379. doi: 10.1109/ISCA.2016.40.  
[2]	Y.-H. Chen, T. Krishna, J. Emer, and V. Sze, “14.5 Eyeriss: An energy-efficient reconfigurable accelerator for deep convolutional neural networks,” in 2016 IEEE International Solid-State Circuits Conference (ISSCC), Jan. 2016, pp. 262–263. doi: 10.1109/ISSCC.2016.7418007.  
[3]	Y.-H. Chen, T. Krishna, J. S. Emer, and V. Sze, “Eyeriss: An Energy-Efficient Reconfigurable Accelerator for Deep Convolutional Neural Networks,” IEEE J. Solid-State Circuits, vol. 52, no. 1, pp. 127–138, Jan. 2017, doi: 10.1109/JSSC.2016.2616357.  
[4]	V. Sze, Y.-H. Chen, T.-J. Yang, and J. Emer, “Efficient Processing of Deep Neural Networks: A Tutorial and Survey.” arXiv, Aug. 13, 2017. doi: 10.48550/arXiv.1703.09039.  
[5]	Y.-H. Chen, T.-J. Yang, J. Emer, and V. Sze, “Eyeriss v2: A Flexible Accelerator for Emerging Deep Neural Networks on Mobile Devices,” IEEE J. Emerg. Sel. Top. Circuits Syst., vol. 9, no. 2, pp. 292–308, Jun. 2019, doi: 10.1109/JETCAS.2019.2910232.  
[6]	A. Samajdar, Y. Zhu, P. Whatmough, M. Mattina, and T. Krishna, “SCALE-Sim: Systolic CNN Accelerator Simulator,” ArXiv181102883 Cs, Feb. 2019, Accessed: May 07, 2022. [Online]. Available: http://arxiv.org/abs/1811.02883  
[7]	D. T. Nguyen, H. Je, T. N. Nguyen, S. Ryu, K. Lee, and H.-J. Lee, “ShortcutFusion: From Tensorflow to FPGA-Based Accelerator With a Reuse-Aware Memory Allocation for Shortcut Data,” IEEE Trans. Circuits Syst. Regul. Pap., pp. 1–13, 2022, doi: 10.1109/TCSI.2022.3153288.  
[8]	A. Azizimazreah and L. Chen, “Shortcut Mining: Exploiting Cross-Layer Shortcut Reuse in DCNN Accelerators,” in 2019 IEEE International Symposium on High Performance Computer Architecture (HPCA), Feb. 2019, pp. 94–105. doi: 10.1109/HPCA.2019.00030.  

