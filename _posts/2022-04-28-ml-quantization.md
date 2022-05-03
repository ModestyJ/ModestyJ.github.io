---
title: "[ML] Quantization"
excerpt: "quantization"

categories:
  - MACHINE LEARNING
tags:
  - [quantization]

permalink: /categories/ml/quantization

date: 2022-04-29
last_modified_at: 2022-04-29
---

- [Overview](#overview)
- [Background](#background)
  - [Quantization Mapping](#quantization-mapping)
- [Dynamic Quantization](#dynamic-quantization)
- [Static Post Training Quantization](#static-post-training-quantization)

## Overview

|   |Quantization|Dataset Requirements|Works Best For|Accuracy|Notes|
|:---:|:---:|:---:|:---:|:---:|:---:|
|Dynamic Quantization  |weights only(both fp16 and int8)|None|LTSMs, MLPs, Transformers|good|Suitable for dynamic models(LTSMs), CLose to static post training quant when performance is compute bound or memory bound due to weights|
|Static Post Training Quantization|weights and activations(8bit)|calibration|CNNs|good|Suitable for static models, provides best perf|
|Static Quantization Aware Training|weights and activations(8bit)|fine-tuning|CNNs|best|Requires fine tuning of model, currently supported only for static quantization|

## Background
### Quantization Mapping
a floating point value : $x \in [\alpha, \beta]$  
a b-bit integer : $x_q \in [\alpha_q, \beta_q]$  

the de-quantization process is defined as

$x = c(x_q + d)$  
{: .text-center }  

the quantization process is defined as  

$\displaystyle x_q = round(\frac{1}{c}x - d)$
{: .text-center }  

$\beta = c(\beta_q + d)$  
$\alpha = c(\alpha_q + d)$  
{: .text-center }  

$\displaystyle c = \frac{\beta - \alpha}{\beta_q - \alpha_q}$  
$\displaystyle d = \frac{\alpha\beta_q - \beta\alpha_q}{\beta - \alpha}$  
{: .text-center }  

일반적으로 floating point 0은 integer 0임을 이용,

$\displaystyle x_q = round(\frac{1}{c}0 - d)$  
$= round(-d)$  
$= -round(d)$  
$= -d$  
{: .text-center }  

$d = round(d)$  
$\displaystyle = round(\frac{\alpha\beta_q - \beta\alpha_q}{\beta - \alpha})$  
{: .text-center }  

$c$를 scale $s$로 정의하고, $-d$를 zero pint $z$로 정의하면,

the de-quantization process is defined as

$x = s(x_q - z)$  
{: .text-center }  

the quantization process is defined as  

$\displaystyle x_q = round(\frac{1}{s}x + z)$
{: .text-center }  

$\displaystyle s = \frac{\beta - \alpha}{\beta_q - \alpha_q}$  
$\displaystyle z = round(\frac{\alpha\beta_q - \beta\alpha_q}{\beta - \alpha})$  
{: .text-center }  

$z$는 integer, $s$는 positive floating point number.


## Dynamic Quantization


## Static Post Training Quantization
