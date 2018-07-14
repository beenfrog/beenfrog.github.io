---
layout: post
title: Build pytorch from source
tags:
- linux
- pytorch
categories:
- code
comments: true
mathjax: false
date: 2018-07-14 17:08:22 +0800
---

When I wanted to install the lastest version of pytorch via conda, it is OK on my PC. However it could not work on Server with OS of CentOS 6.x due to the version of GLIBC. So I decided to build and install pytorch from source. The commands are recorded as follows.

### 1. Clone the source from github
```bash
git clone --recursive https://github.com/pytorch/pytorch  # new clone
git pull && git submodule update --init --recursive       # or update
```

### 2. Install dependencies
```bash
conda install numpy pyyaml mkl mkl-include setuptools cmake cffi typing
conda install -c pytorch magma-cuda90  #[on cluster] 
conda install -c pytorch magma-cuda91  #[on local PC]   
```

### 3. Set environment variables
#### cluster
```bash
export NO_MKLDNN=1 # disable due to GLIBC problem
export NO_SYSTEM_NCCL=1
export CUDNN_LIB_DIR="/data1/****/local/cudnn9.0-v7/lib64"
export CUDNN_INCLUDE_DIR="/data1/****/local/cudnn9.0-v7/include"
export CMAKE_PREFIX_PATH="$(dirname $(which conda))/../"
```
#### local pc
```bash
export NO_MKLDNN=1
export NO_SYSTEM_NCCL=1
export CUDNN_LIB_DIR="/usr/local/cuda-9.1/lib64"
export CUDNN_INCLUDE_DIR="/usr/local/cuda-9.1/include"
export CMAKE_PREFIX_PATH="$(dirname $(which conda))/../"
```

### 4. Install
```bash
python setup.py install     # build and install
python setup.py clean --all # clean the build
```

### 5. Others
```bash
pip install git+https://github.com/pytorch/vision.git@master
pip install tensorboardX  #[the loggor]
pip install tensorboard   #[the web server for display]
pip install tensorflow    #[need by tensorboard]
```
