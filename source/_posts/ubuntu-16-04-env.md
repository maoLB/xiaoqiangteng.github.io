---
title: ubuntu_16.04_env
date: 2018-03-25 14:18:18
categories:
- 编程
- Linux
- 环境配置
tags:
- 编程
- 环境配置
copyright:
---

# Ubuntu 16.04 环境配置

## PCL1.8环境

### 第一步：安装依赖

``` bash
sudo apt-get update  
sudo apt-get install git build-essential linux-libc-dev  
sudo apt-get install cmake cmake-gui   
sudo apt-get install libusb-1.0-0-dev libusb-dev libudev-dev  
sudo apt-get install mpi-default-dev openmpi-bin openmpi-common    
sudo apt-get install libflann1.8 libflann-dev  
sudo apt-get install libeigen3-dev  
sudo apt-get install libboost-all-dev  
sudo apt-get install libvtk5.10-qt4 libvtk5.10 libvtk5-dev  
sudo apt-get install libqhull* libgtest-dev  
sudo apt-get install freeglut3-dev pkg-config  
sudo apt-get install libxmu-dev libxi-dev   
sudo apt-get install mono-complete  
sudo apt-get install qt-sdk openjdk-8-jdk openjdk-8-jre  
```

### 第二步：下载源码

``` bash
git clone https://github.com/PointCloudLibrary/pcl.git
```

### 第三步：编译源码

``` bash
cd pcl  
mkdir release  
cd release  
cmake -DCMAKE_BUILD_TYPE=None -DCMAKE_INSTALL_PREFIX=/usr \  
      -DBUILD_GPU=ON -DBUILD_apps=ON -DBUILD_examples=ON \  
      -DCMAKE_INSTALL_PREFIX=/usr ..  
make 
```

``` bash
sudo make install  
```

### 第四步（可选 and 建议）：如果需要PCLVisualizer

安装OpenNI、OpenNI2

``` bash
sudo apt-get install libopenni-dev   
sudo apt-get install libopenni2-dev
```

安装[ensensor](https://www.ensenso.com/support/sdk-download/?lang=de)

``` bash
sudo dpkg -i ensenso-sdk-2.0.147-x64.deb 
sudo dpkg -i codemeter_6.40.2402.501_amd64.deb
```

如缺少依赖：

``` bash
sudo apt-get -f install
```

### 参考：
1. https://blog.csdn.net/dantengc/article/details/78446600

## OpenCV环境

## Caffe 环境

