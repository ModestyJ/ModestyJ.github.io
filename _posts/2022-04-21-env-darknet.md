---
title: "[ENV] darknet 사용을 위한 CUDA, OPENCV 환경"
excerpt: "Ubuntu GPU environment setup for darkent"

categories:
  - ENVIRONMENT
tags:
  - [darknet, cuda, opencv, cudnn]

permalink: /categories/env/darknet

date: 2022-04-21
last_modified_at: 2022-04-21
---

- [Overview](#overview)
- [Install Anaconda](#install-anaconda)
- [Install CUDA toolkit](#install-cuda-toolkit)
- [Install cuDNN](#install-cudnn)
- [Install OpenCV](#install-opencv)
  - [Errors](#errors)
- [Install darknet](#install-darknet)

## Overview
YOLO의 아버지 Joseph Redmon님께서 만드신 C language base의 darknet이라는 framework이 reference로 릴리즈 된다. 다른 framework(pytorch, tensorflow) porting 된 결과도 있지만 이번 기회에 한 번 사용해 보자.  

NVIDIA GPU를 사용해 darkent framework으로 training까지 해보고 싶다면 cuda toolkit, cuDNN library 설치가 필요하다.(opencv는 옵션)  

Ubuntu 에서 sudo 권한이 없을 때, cuda toolkit(driver 제외), cuDNN, opencv를 user local 설치하는 방법을 기록 해 둔다. 그러나 darknet 설치 시 GPU 환경 설정이 어려운 경우 cpu only로 설치하는 것이 정신건강에 좋을 듯 싶다. 특히 OpenCV는 root 권한이 없다면 설치하지 말자.

```bash
cat /etc/issue
Ubuntu 20.04 LTS
```

## Install Anaconda
```bash
wget https://repo.anaconda.com/archive/Anaconda3-2021.11-Linux-x86_64.sh
sh Anaconda3-2021.11-Linux-x86_64.sh
```
~/.bashrc에 환경변수 잡아주도록 이후에 추가 설치 'yes' 할 것.

```bash
conda create -n darknet python=3.9
conda install cmake
```

## Install CUDA toolkit
/home/jckim/tools/cuda 에 tookit 설치를 가정.
(단, 드라이버는 sudo 설치 해야 함)

`nvidia-smi`를 사용 가능한 상태였고, toolkit도 이미 설치 되어 있었지만 user specific toolkit이 필요한 경우 참조.

[cuda-toolkit](hhtps://developer.nvidia.com/cuda-downloads)에서 상황에 맞게 다운로드.
conda or pip install로 받는건 일부임.

![Image1](/assets/images/darknet/darknet-image-1.png){: . width='500px'}  
```bash
wget https://developer.download.nvidia.com/compute/cuda/11.6.2/local_installers/cuda_11.6.2_510.47.03_linux.run

./cuda_11.3.0_465.19.01_linux.run --toolkit --toolkitpath=/home/jckim/tools/cuda --samples --samplespath=/home/jckim/tools/
cuda/samples
```

![Image2](/assets/images/darknet/darknet-image-2.png){: . width='500px'}  
![Image3](/assets/images/darknet/darknet-image-3.png){: . width='500px'}  

```bash
~/.bashrc
export PATH=/home/jckim/tools/cuda/bin:$PATH
export LD_LIBRARY_PATH=/home/jckim/tools/cuda/lib64:$LD_LIBRARY_PATH
```

## Install cuDNN
[cudnn](https://developer.nvidia.com/rdp/cudnn-download) download after login. Toolkit이 잘 설치되어 있다면 복사만 해주면 된다.

![Image4](/assets/images/darknet/darknet-image-4.png){: . width='500px'}  

```bash
tar xvf cudnn-linux-x86_64-8.4.0.27_cuda11.6-archive.tar.xz
cp -rf cudnn-linux-x86_64-8.4.0.27_cuda11.6-archive/include/* ~/tools/cuda/include
cp -rf cudnn-linux-x86_64-8.4.0.27_cuda11.6-archive/lib/* ~/tools/cuda/lib64
```

## Install OpenCV

```bash
wget -O opencv_contrib.zip https://github.com/opencv/opencv/archive/4.4.0.zip
unzip opencv.zip
wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.4.0.zip
unzip opencv_contrib.zip

cd opencv-4.4.0
mkdir build
cd build
```

```bash
nvidia-smi -L
GPU 0: NVIDIA GeForce GTX TITAN X
```
[CUDA compute capability](https://en.wikipedia.org/wiki/CUDA) should be matched with **CUDA_ARCH_BIN**

```bash
cmake -D CMAKE_BUILD_TYPE=RELEASE \
  -D CMAKE_INSTALL_PREFIX=/home/jckim/tools/opencv \
  -D INSTALL_PYTHON_EXAMPLES=ON \
  -D INSTALL_C_EXAMPLES=ON \
  -D BUILD_DOCS=OFF \
  -D BUILD_PERF_TESTS=OFF \
  -D BUILD_TESTS=OFF \
  -D BUILD_PACKAGE=OFF \
  -D BUILD_EXAMPLES=OFF \
  -D WITH_TBB=ON \
  -D ENABLE_FAST_MATH=1 \
  -D CUDA_FAST_MATH=1 \
  -D CUDA_TOOLKIT_ROOT_DIR=/home/jckim/tools/cuda \
  -D WITH_CUDA=ON \
  -D WITH_CUBLAS=ON \
  -D WITH_CUFFT=ON \
  -D WITH_NVCUVID=ON \
  -D WITH_IPP=OFF \
  -D WITH_V4L=ON \
  -D WITH_1394=OFF \
  -D WITH_GTK=ON \
  -D WITH_QT=OFF \
  -D WITH_OPENGL=ON \
  -D WITH_EIGEN=ON \
  -D WITH_FFMPEG=ON \
  -D WITH_GSTREAMER=ON \
  -D BUILD_JAVA=OFF \
  -D BUILD_opencv_python3=ON \
  -D BUILD_opencv_python2=OFF \
  -D BUILD_NEW_PYTHON_SUPPORT=ON \
  -D OPENCV_SKIP_PYTHON_LOADER=ON \
  -D OPENCV_GENERATE_PKGCONFIG=ON \
  -D OPENCV_ENABLE_NONFREE=ON \
  -D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib-4.4.0/modules \
  -D WITH_CUDNN=ON \
  -D OPENCV_DNN_CUDA=ON \
  -D CUDA_ARCH_BIN=6.1 \
  -D CUDA_ARCH_PTX=6.1 \
  -D CUDNN_LIBRARY=/home/jckim/tools/cuda/lib64/libcudnn.so.8.4.0 \
  -D CUDNN_INCLUDE_DIR=/home/jckim/tools/cuda/include  ..
```

```bash
make -j6
make install
```

~/.bashrc
```bash
export LD_LIBRARY_PATH=/home/jckim/tools/opencv/lib:$LD_LIBRARY_PATH
```

### Errors  
conda에 설치되 것과 system default 사이에 충돌 때문인 듯.
```bash
conda uninstall libtiff
conda install numpy
```

## Install darknet
[yolo4 darket](https://github.com/AlexeyAB/darknet) 참조.
```bash
git clone https://github.com/AlexeyAB/darknet
cd darknet
mkdir build_release
cd build_release
cmake -D CUDNN_LIBRARY=/home/jckim/tools/cuda/lib64/libcudnn.so.8.4.0 -D CUDNN_INCLUDE_DIR=/home/jckim/tools/cuda/include ..
cmake --build . --target install --parallel 6
```


```bash
mkdir models
cd models
wget https://github.com/AlexeyAB/darknet/releases/download/darknet_yolo_v3_optimal/yolov4.weights

cd ..
./darknet detect cfg/yolov4.cfg models/yolov4.weights data/person.jpg
```
![Image5](/assets/images/darknet/darknet-image-5.png)