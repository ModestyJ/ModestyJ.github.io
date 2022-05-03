---
title: "[ENV] torchrec CUDA 환경 설정"
excerpt: "Ubuntu GPU environment setup for torchrec"

categories:
  - ENVIRONMENT
tags:
  - [torchrec, dlrm, recommendation system]

permalink: /categories/env/torchrec

date: 2022-05-03
last_modified_at: 2022-05-03
---

- [Overview](#overview)
- [Install torch](#install-torch)
- [Install torchrec](#install-torchrec)

## Overview
Recommendation system 중 meta의 [DLRM](https://github.com/facebookresearch/dlrm) model, 대규모 recommender system 학습 시 효율적인 workload distribution을 위한 [Torchrec](https://pytorch.org/torchrec) 사용을 위해 GPU 상황에 따라 manual compile이 필요한 경우가 있어 정리해 둔다.  
`pip install torchrec` 으로 설치는 잘 되는데 다음과 같은 CUDA runtime error가 발생하는 경우에 해당 함.
```
RuntimeError: CUDA error: no kernel image is available for execution on the device
```

## Install torch
anaconda 환경에서 pytorch cuda 버젼을 사용할 경우에는 local cuda toolkit이 이미 설치되어 있어도 동일(또는 더 낮은) 버젼의 cuda toolkit을 같이 설치해야 했다.  
nightly 버젼은 stable 보장하지 않는 최신 소스코드 기반 버젼.
```bash
conda install pytorch cudatoolkit=11.3 -c pytorch-nightly
```

## Install torchrec
NVIDIA GPU [Compute capability](https://developer.nvidia.com/cuda-gpus)에 맞는 fbgemm(matrix multiplication library) 설치 필요.  
[FBGEMM github](https://github.com/pytorch/FBGEMM/tree/e236401ee2dcb262c1556709993f71cd08e0d834/fbgemm_gpu)에 따르면 2022.05.03 현재 `pip install torchrec` 에서는 capability 7.0, 8.0만 지원 함.  

현재 사용중인 서버 GPU capability 확인은 아래와 같이 device 찾아서 [CUDA compute capability](https://developer.nvidia.com/cuda-gpus)(또는 [wiki](https://en.wikipedia.org/wiki/CUDA))에서 확인.  

```bash
nvidia-smi -L
GPU 0: NVIDIA GeForce GTX TITAN X
```

GTX TITAN X는 5.2이므로 setup.py의 **TORCH_CUDA_ARCH_LIST** 에 5.2 입력.
```bash
# install fbgemm separatly
git clone --recursive https://github.com/pytorch/torchrec
cd torchrec/third_party/fbgemm/fbgemm_gpu

# Specify CUDA version to use
export CUDA_BIN_PATH=/usr/local/cuda-11.3/
export CUDACXX=/usr/local/cuda-11.3/bin/nvcc

# if using CUDA 10 or earliers set the location to the CUB installation directory
export CUB_DIR=${CUB_DIR}
# in fbgemm_gpu folder
# build for the CUDA architecture supported by current system (or all architectures if no CUDA device present)
python setup.py install
# or build it for specific CUDA architectures (see PyTorch documentation for usage of TORCH_CUDA_ARCH_LIST)
python setup.py install -DTORCH_CUDA_ARCH_LIST="5.2"
```

다시 torchrec repository 받았던 directory로 돌아가서,
```bash
# install torchrec without fbgemm
python setup.py install develop --skip_fbgemm
```