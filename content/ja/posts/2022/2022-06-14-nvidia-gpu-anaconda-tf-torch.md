---
title: WSL2に機械学習環境を構築（NVIDIA GPU, Anaconda, TensorFlow, PyTorch）
date: 2022-06-14
draft: false
tags:
- ML
- Anaconda
- TensorFlow
- PyTorch
categories:
- ML
---

GPUありのWindowsマシンのWSL2に機械学習環境を立てた。

## WSL2
インストール済み前提。
```powershell
$ wsl -l -v
  NAME      STATE           VERSION
* Ubuntu    Stopped         2
$ wsl --status
既定の配布: Ubuntu
既定のバージョン: 2

Linux 用 Windows サブシステムの最終更新日: 2022/03/27
WSL の自動更新が有効になっています。

カーネル バージョン: 5.10.102.1
```

## NVIDIAドライバのインストール（on Windows）
WSL2ではなくてWindowsにインストールする。  
https://developer.nvidia.com/cuda-downloads?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_network

インストール完了後、`nvidia-smi`が実行できればOK。
```powershell
$ nvidia-smi
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 516.01       Driver Version: 516.01       CUDA Version: 11.7     |
|-------------------------------+----------------------+----------------------+
| GPU  Name            TCC/WDDM | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ... WDDM  | 00000000:01:00.0 Off |                  N/A |
| N/A   60C    P3    20W /  N/A |      0MiB /  6144MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```

## WSL2 Ubuntu 20.04 上に機械学習環境を構築

### CUDA 11.5
https://developer.nvidia.com/cuda-11-5-2-download-archive?target_os=Linux&target_arch=x86_64&Distribution=WSL-Ubuntu&target_version=2.0&target_type=deb_local
```bash
# install cuda
$ wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-wsl-ubuntu.pin
$ sudo mv cuda-wsl-ubuntu.pin /etc/apt/preferences.d/cuda-repository-pin-600
$ wget https://developer.download.nvidia.com/compute/cuda/11.5.2/local_installers/cuda-repo-wsl-ubuntu-11-5-local_11.5.2-1_amd64.deb
$ sudo dpkg -i cuda-repo-wsl-ubuntu-11-5-local_11.5.2-1_amd64.deb
$ sudo apt-key add /var/cuda-repo-wsl-ubuntu-11-5-local/7fa2af80.pub
$ sudo apt-get update
$ sudo apt-get -y install cuda

# set env
$ echo 'export CUDA_PATH=/usr/local/cuda-11.5' >> ${HOME}/.bashrc
$ echo 'export LD_LIBRARY_PATH=/usr/local/cuda-11.5/lib64:${LD_LIBRARY_PATH}' >> ${HOME}/.bashrc
$ echo 'export PATH=/usr/local/cuda-11.5/bin:${PATH}' >> ${HOME}/.bashrc
$ echo 'export CUDA_VISIBLE_DEVICES=0' >> ${HOME}/.bashrc
$ source ${HOME}/.bashrc

$ nvcc --version
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2021 NVIDIA Corporation
Built on Thu_Nov_18_09:45:30_PST_2021
Cuda compilation tools, release 11.5, V11.5.119
Build cuda_11.5.r11.5/compiler.30672275_0

$ cat /usr/local/cuda-11.5/version.json                                                             ❮   130
{
   "cuda" : {
      "name" : "CUDA SDK",
      "version" : "11.5.2"
   },
   "cuda_cudart" : {
      "name" : "CUDA Runtime (cudart)",
      "version" : "11.5.117"
   },
   "cuda_cuobjdump" : {
      "name" : "cuobjdump",
      "version" : "11.5.119"
   },
   "cuda_cupti" : {
      "name" : "CUPTI",
      "version" : "11.5.114"
   },
   "cuda_cuxxfilt" : {
      "name" : "CUDA cu++ filt",
      "version" : "11.5.119"
   },
   "cuda_demo_suite" : {
      "name" : "CUDA Demo Suite",
      "version" : "11.5.50"
   },
   "cuda_gdb" : {
      "name" : "CUDA GDB",
      "version" : "11.5.114"
   },
   "cuda_memcheck" : {
      "name" : "CUDA Memcheck",
      "version" : "11.5.114"
   },
   "cuda_nsight" : {
      "name" : "Nsight Eclipse Plugins",
      "version" : "11.5.114"
   },
   "cuda_nvcc" : {
      "name" : "CUDA NVCC",
      "version" : "11.5.119"
   },
   "cuda_nvdisasm" : {
      "name" : "CUDA nvdisasm",
      "version" : "11.5.119"
   },
   "cuda_nvml_dev" : {
      "name" : "CUDA NVML Headers",
      "version" : "11.5.50"
   },
   "cuda_nvprof" : {
      "name" : "CUDA nvprof",
      "version" : "11.5.114"
   },
   "cuda_nvprune" : {
      "name" : "CUDA nvprune",
      "version" : "11.5.119"
   },
   "cuda_nvrtc" : {
      "name" : "CUDA NVRTC",
      "version" : "11.5.119"
   },
   "cuda_nvtx" : {
      "name" : "CUDA NVTX",
      "version" : "11.5.114"
   },
   "cuda_nvvp" : {
      "name" : "CUDA NVVP",
      "version" : "11.5.126"
   },
   "cuda_samples" : {
      "name" : "CUDA Samples",
      "version" : "11.5.56"
   },
   "cuda_sanitizer_api" : {
      "name" : "CUDA Compute Sanitizer API",
      "version" : "11.5.114"
   },
   "cuda_thrust" : {
      "name" : "CUDA C++ Core Compute Libraries",
      "version" : "11.5.62"
   },
   "libcublas" : {
      "name" : "CUDA cuBLAS",
      "version" : "11.7.4.6"
   },
   "libcufft" : {
      "name" : "CUDA cuFFT",
      "version" : "10.6.0.107"
   },
   "libcufile" : {
      "name" : "GPUDirect Storage (cufile)",
      "version" : "1.1.1.25"
   },
   "libcurand" : {
      "name" : "CUDA cuRAND",
      "version" : "10.2.7.107"
   },
   "libcusolver" : {
      "name" : "CUDA cuSOLVER",
      "version" : "11.3.2.107"
   },
   "libcusparse" : {
      "name" : "CUDA cuSPARSE",
      "version" : "11.7.0.107"
   },
   "libnpp" : {
      "name" : "CUDA NPP",
      "version" : "11.5.1.107"
   },
   "libnvjpeg" : {
      "name" : "CUDA nvJPEG",
      "version" : "11.5.4.107"
   },
   "nsight_compute" : {
      "name" : "Nsight Compute",
      "version" : "2021.3.1.4"
   },
   "nsight_systems" : {
      "name" : "Nsight Systems",
      "version" : "2021.3.3.2"
   },
   "nvidia_fs" : {
      "name" : "NVIDIA file-system",
      "version" : "2.9.5"
   }
}

# Build sample program
$ sudo apt-get install -y g++ freeglut3-dev build-essential libx11-dev libxmu-dev libxi-dev libglu1-mesa libglu1-mesa-dev
$ sudo rm -rf cuda-samples > /dev/null 2>&1
$ sudo git clone https://github.com/NVIDIA/cuda-samples.git
$ cd cuda-samples/Samples/1_Utilities/deviceQuery/
$ sudo make
$ sudo ./deviceQuery
$ cd ../../../../cuda-samples/Samples/1_Utilities/bandwidthTest/
$ sudo make
$ sudo ./bandwidthTest
$ cd ../../../../cuda-samples/Samples/4_CUDA_Libraries/simpleCUBLAS
$ sudo make
$ sudo ./simpleCUBLAS
$ cd ../../../../cuda-samples/Samples/4_CUDA_Libraries/simpleCUFFT/
$ sudo make
$ sudo ./simpleCUFFT
```

### cuDNN 8.33
https://docs.nvidia.com/deeplearning/cudnn/install-guide/index.html#installlinux-deb
https://developer.nvidia.com/rdp/cudnn-archive
```bash
$ sudo dpkg -i cudnn-local-repo-ubuntu2004-8.3.3.40_1.0-1_amd64.deb
$ sudo cp /var/cudnn-local-repo-ubuntu2004-8.3.3.40/cuda-ubuntu2004.pin /usr/share/keyrings/
$ sudo apt-get -y update
$ apt-cache search cudnn
$ sudo apt-get -y install libcudnn8 libcudnn8-dev
```

### Anaconda
https://www.anaconda.com/
```bash
$ bash Anaconda3-2022.05-Linux-x86_64.sh
# 言われるがままに従う
```

### TensorFlow GPU
https://www.tensorflow.org/install/pip?hl=ja

TensorFlow 2以降はGPUもCPUも同じパッケージらしい。  
`conda install`でうまくいかなかったので、`pip install`した。
```bash
# ランタイムエラーになるので…
$ sudo ln -s /usr/local/cuda-11.5/lib64/libcusolver.so.11 /usr/local/cuda-11.5/lib64/libcusolver.so.10
$ conda create -n tf python=3.9
$ conda activate tf
(tf) $ pip install tensorflow==2.9.1
(tf) $ conda install jupyterlab -y
(tf) $ jupyter lab --no-browser
```
動作確認。
```python
import tensorflow as tf
from tensorflow.python.client import device_lib

print( tf.__version__ )
print(device_lib.list_local_devices())
```

### PyTorch
https://pytorch.org/get-started/locally/
```bash
$ conda deactivate
$ conda create -n torch python=3.9
$ conda activate torch
(torch) $ conda install pytorch torchvision torchaudio cudatoolkit=11.3 -c pytorch -y
(torch) $ conda install jupyterlab -y
(torch) $ jupyter lab --no-browser
```
動作確認。
```python
import torch
print(torch.__version__, torch.cuda.is_available())
x = torch.rand(5, 3)
print(x)
```