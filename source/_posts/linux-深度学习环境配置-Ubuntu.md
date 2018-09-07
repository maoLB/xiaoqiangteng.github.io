---
title: linux-深度学习环境配置-Ubuntu
date: 2018-03-31 21:17:38
categories:
- 编程
- Linux
- 环境配置
tags:
- 编程
- Linux
- 环境配置
copyright:
---

# Ubuntu16.04 cuda8.0 GPU驱动配置

- ubuntu 16.04 64bit 
- 显卡：NVIDIA Tesla k40m + 集成显卡

## 更换阿里源

更换之前要先备份之前的源：

```bash
sudo cp /etc/apt/source.list /etc/apt/source.list.bak
```

编辑源列表文件:

```bash
sudo vim/etc/apt/sources.list
```

原来的列表删除，替换：

```bash
# deb cdrom:[Ubuntu 16.04 LTS _Xenial Xerus_ - Release amd64 (20160420.1)]/ xenial main restricted
deb-src http://archive.ubuntu.com/ubuntu xenial main restricted #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb http://mirrors.aliyun.com/ubuntu/ xenial multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse #Added by software-properties
deb http://archive.canonical.com/ubuntu xenial partner
deb-src http://archive.canonical.com/ubuntu xenial partner
deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial-security universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-security multiverse
```

更新：

```bash
sudo apt-get update
```

## Ubuntu16.04挂载新硬盘并格式化硬盘

查看硬盘:

```bash
sudo fdisk -l
```

新建分区:

```bash
$ sudo fdisk /dev/sdb
```

之后进入command状态，大概是这么操作的：

- 输入 m 查看帮助
- 输入 p 查看 /dev/sdb 分区的状态
- 输入 n 创建sdb这块硬盘的分区
- 选 p primary =>输入　p
- Partition number =>分一个区所以输入　1
- 其他的默认回车即可
- 最后输入 w 保存并退出 Command 状态。

操作示例:

```bash
Command (m for help): n
# n创建分区
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
# p(primary主分区） e(extended拓展分区)
Partition number (1-4, default 1): 1
# 分区号
First sector (2048-83886079, default 2048): 
# 默认
Last sector, +sectors or +size{K,M,G,T,P} (2048-83886079, default 83886079): 
# 大小，可自定义，保持默认
Created a new partition 1 of type 'Linux' and of size 40 GiB.

Command (m for help): p
# 查看分区情况
Disk /dev/sdb: 40 GiB, 42949672960 bytes, 83886080 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xbb6c1792

Device     Boot Start      End  Sectors Size Id Type
/dev/sdb1        2048 83886079 83884032  40G 83 Linux

Command (m for help): w
# 保存
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

在通过查看命令即可查看，新增的硬盘.

格式化:

```bash
$ sudo mkfs.ext4 /dev/sdb1 # ext4为分区格式
```

挂载:

```bash
sudo mkdir /home/data
sudo mount /dev/sdb1 /home/data
```

开机自动挂载:

```bash
$ sudo blkid
```

添加UUID到/etc/fstab 添加UUID=63295b70-daec-4253-b659-821f51200be9 /home/data ext4 defaults,errors=remount-ro 0 1到/etc/fstab 其中UUID后面跟sdb1的UUID 重启。

## 安装必要的软件

```bash
sudo apt-get install vim git openssh-server
```

## 检查是否正确识别显卡

```bash
lspci | grep -i nvidia
```

## 查看是否已有安装的NVIDIA驱动

```bash
lsmod | grep nvidia
```

## 查看集显驱动

```bash
lsmod | grep nouveau
```

## 禁用nouveau驱动和相关的驱动包

用编辑器打开blacklist.conf配置文件

```bash
sudo gedit /etc/modprobe.d/blacklist.conf
```

在文件的最后一行加入下面的命令，屏蔽有影响的驱动包（这里有的博客添加了blacklist amd76x_edac，但是经测试后不加也是可以安装成功的）

```bash
blacklist rivafb
blacklist vga16fb
blacklist nouveau
blacklist nvidiafb
blacklist rivatv
options nouveau modeset=0
alias nouveau off
alias lbm-nouveau off
```

禁用 nouveau 内核模块:

```bash
$echo options nouveau modeset=0 | sudo tee -a /etc/modprobe.d/nouveau-kms.conf
$sudo update-initramfs -u
```

## 卸载所有安装的nvidia驱动

```bash
sudo apt-get remove –purge nvidia*
```

## 重启

```bash
sudo reboot
```

## GPU驱动配置

根据GPU型号从相应网站下载驱动，例如使用NVIDIA Tesla M60，从[NVIDIA网站](http://www.nvidia.cn/Download/index.aspx?lang=cn)选择对应的型号和操作系统，CUDA Toolkit版本，下载驱动文件，如NVIDIA-Linux-x86_64-375.66.run，运行驱动文件，根据提示安装：


### 安装驱动

安装驱动可能需要的依赖(可选):

```bash
$sudo apt-get update
$sudo apt-get install dkms build-essential linux-headers-generic
$sudo gedit ~/.bashrc
	export LD_LIBRARY_PATH=/usr/lib/x86_64-linux-gnu:$LD_LIBRARY_PATH
	export LD_LIBRARY_PATH=/lib/x86_64-linux-gnu:$LD_LIBRARY_PATH
$source ~/.bashrc
```

进入命令行界面

```bash
Ctrl-Alt+F1 
sudo /etc/init.d/lightdm stop #关闭当前图形环境令
```

关闭桌面服务

```bash
sudo service lightdm stop
```

给驱动run文件赋予执行权限

```bash
sudo chmod a+x NVIDIA-Linux-x86_64-384.66.run
```

安装: 注意下面参数

```bash
sudo ./NVIDIA-Linux-x86_64-384.66.run –no-x-check –no-nouveau-check –no-opengl-files 
# –no-x-check安装驱动时关闭X服务 
# –no-nouveau-check 安装驱动时禁用nouveau 
# –no-opengl-files 只安装驱动文件，不安装OpenGL文件
```

注意：安装CUDA时一定使用runfile文件，这样可以进行选择。不再选择安装驱动，以及在弹出xorg.conf时选择NO.不要使用ubuntu设置中附加驱动中驱动

报错：

（1）ERROR: Unable to load the kernel module 'nvidia.ko'. This happens most frequently when this kernel module was built against the wrong or improperly configured kernel sources, with a version of gcc that differs from the one used to build the target kernel, or if a driver such as rivafb, nvidiafb, or nouveau is present and prevents the NVIDIA kernel module from obtaining ownership of the NVIDIA graphics device(s), or no NVIDIA GPU installed in this system is supported by this NVIDIA Linux graphics driver release.

解决方法：

- 禁用nouveau驱动和相关的驱动
- 首先，Ctrl+Alt+F1进入命令提示符界面 
- 然后，输入对应的username和passwd进入命令行. 
- 最后，使用指令sudo service lightdm stop 关闭图形界面，再利用cd指令进入下载好的驱动目录

```bash
sudo chmod 755 NVIDIA-Linux-x86_64-384.111.run  #修改权限（否则没有访问权限，无法进行指令安装）
sudo ./NVIDIA-Linux-x86_64-384.111.run –no-x-check –no-nouveau-check –no-opengl-files #安装驱动
#–no-x-check 关闭X服务
#–no-nouveau-check 禁用nouveau
#–no-opengl-files 不安装OpenGL文件
#...安装完成后
sudo update-initramfs -u
sudo reboot
```

(2) WARNING: Unable to find a suitable destination to install 32-bit compatibility libraries. Your system may not be set up for 32-bit compatibility. 32-bit compatibility files will not be installed; if you wish to install them, re-run the installation and set a valid directory with the --compat32-libdir option.

解决方法：

运行命令：

```bash
sudo aptitude install ia32-libs
```

## 安装[cuda](https://developer.nvidia.com/cuda-80-ga2-download-archive)

注意这里下载的是cuda8.0的runfile（local）文件。 

这里是nvidia给出的官方安装指南（遇到问题时可以查阅) 

下载完cuda8.0后，执行如下语句，运行runfile文件： 

```bash
sudo sh cuda_8.0.27_linux.run
```

因为驱动之前已经安装，这里就不要选择安装驱动。其余的都直接默认或者选择是即可。

使用：

```bash
sudo gedit /etc/profile
```

打开“profile”文件，在末尾处添加（注意不要有空格，不然会报错):

```bash
export PATH=/usr/local/cuda-8.0/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64$LD_LIBRARY_PATH
```

重启电脑：

```bash
sudo reboot
```

测试cuda的Samples:

```bash
cd /usr/local/cuda-8.0/samples/1_Utilities/deviceQuery
sudo make
./deviceQuery
```

# OpenCV3.1配置

## 安装依赖库

```bash
sudo apt-get install build-essential
# 必须的，gcc编译环境

sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
# 必须的,包括cmake等工具

sudo apt-get install python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libjasper-dev libdc1394-22-dev
# 可选的，看个人需要，总共5M左右

sudo apt-get install libv4l-dev
```

## 下载 源码

[OpenCV](https://opencv.org/releases.html)

或着用git clone：

```bash
cd ~/opencv310
    # opencv310为自己建的，源码将放在这里
git clone https://github.com/Itseez/opencv.git
git clone https://github.com/Itseez/opencv_contrib.git
```

## CMake Opencv源码

建立一个编译目录（例如：/build）把cmake后的文件都放在这里边。

```bash
cd ~/opencv
mkdir build  //建立一个build目录，把cmake的文件都放着里边
cd build　　　//进入build目录
```

cmake时ippicv_linux_20151201.tgz总是不能成功下载，故cmake之前将./downloads/linux-808b791a6eac9ed78d32a7666804320e 文件拷贝至./opencv-3.1.0/3rdparty/ippicv/ 路径下(先执行一次cmake 命令生成文件路径，在将ippicv_linux_20151201.tgz复制进去) 首先，手动下载[ippicv](https://raw.githubusercontent.com/Itseez/opencv_3rdparty/81a676001ca8075ada498583e4166079e5744668/ippicv/ippicv_linux_20151201.tgz)

然后开始cmake，这里需要注意几个cmake的参数，比较重要。

```bash
sudo cmake -D CMAKE_BUILD_TYPE=Release -D CMAKE_INSTALL_PREFIX=/usr/local WITH_LIBV4L=ON ..
```

切记最后’..’两个点之前要加空格！！

## 把代码编译成可执行文件

这里官方推荐使用多进程编译，推荐七个进程：

```bash
# 报错：
# modules/cudalegacy/src/graphcuts.cpp:120:54: error: 
# ‘NppiGraphcutState’ has not been declared
# typedef NppStatus (*init_func_t)(NppiSize oSize, 
# NppiGraphcutState** ppState, Npp8u* pDeviceMem);
# 这是因为opecv3.0与cuda8.0不兼容导致的。解决办法： 
# 修改 ～/opencv/modules/cudalegacy/src/graphcuts.cpp文件内容
# 将  
# #if !defined (HAVE_CUDA) || defined (CUDA_DISABLER)   
# 改为  
# #if !defined (HAVE_CUDA) || defined (CUDA_DISABLER) || (CUDART_VERSION >= 8000) 
make -j7 # 并行运行七个jobs，这一步也在build目录中进行
```

## 安装

```bash
sudo make install

如果你要在python下运行opencv库的情况下，那就必须安装安装python-opencv
sudo apt-get install python-opencv
```

## 配置库文件路径

```bash
/bin/bash -c 'echo "/usr/local/lib" > /etc/ld.so.conf.d/opencv.conf'
#或者直接打开/etc/ld.so.conf.d/opencv.conf，添加/usr/local/lib
#使配置生效
sudo  ldconfig(重要)
```

## 设置环境变量

```bash
sudo vim/etc/bash.bashrc   
#在最后加入以下两行代码
PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/local/lib/pkgconfig 
export PKG_CONFIG_PATH  
#使配置生效
sudo source /etc/bash.bashrc 
（该步骤可能会报错找不到命令，原因是source为root命令
su（进入root权限） 
```

## 查看

```bash
pkg-config --modversion opencv
pkg-config --cflags opencv
```

## 运行测试

我是用python+opencv的，我这里直接运行opencv自带的python的例子程序，TX1自带摄像头不能用，需要使用外接USB摄像头，插入USB接口即可，无需安装驱动，也无需改动测试代码。

```bash
cd <opencv3.1.0_dir>/samples/python/
python video.py
python edge.py
python facedetect.py
```

# [colmap](https://colmap.github.io/tutorial.html)配置

clone colmap源码到本地：

```bash
git clone https://github.com/colmap/colmap
```

安装依赖：

```bash
sudo apt-get install cmake build-essential libboost-all-dev libeigen3-dev libsuitesparse-dev libfreeimage-dev libgoogle-glog-dev libgflags-dev libglew-dev qtbase5-dev libqt5opengl5-dev
```

配置Ceres Solver:

```bash
sudo apt-get install libatlas-base-dev libsuitesparse-dev
git clone https://ceres-solver.googlesource.com/ceres-solver
cd ceres-solver
mkdir build
cd build
cmake .. -DBUILD_TESTING=OFF -DBUILD_EXAMPLES=OFF
make
sudo make install
# 注：如果该安装包无法下载，请离线下载安装，需翻墙
```

配置和编译colmap:

```bash
cd path/to/colmap
mkdir build
cd build
cmake ..
make -j8
sudo make install
```

运行colmap:

```bash
colmap -h
colmap gui
```

# Caffe配置

安装依赖：

```bash
sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libhdf5-serial-dev protobuf-compiler
sudo apt-get install --no-install-recommends libboost-all-dev
sudo apt-get install libopenblas-dev liblapack-dev libatlas-base-dev
sudo apt-get install libgflags-dev libgoogle-glog-dev liblmdb-dev
sudo apt-get install git cmake build-essential
```

有一定几率安装失败而导致后续步骤出现问题，所以要确保以上依赖包都已安装成功，验证方法就是重新运行安装命令，如验证 git cmake build-essential是否安装成功共则再次运行以下命令：

```bash
sudo apt-get install git cmake build-essential 
```

安装的路径下 clone ：

```bash
git clone https://github.com/BVLC/caffe.git
```

进入 caffe ，将 Makefile.config.example 文件复制一份并更名为 Makefile.config ，也可以在 caffe 目录下直接调用以下命令完成复制操作 ：

```bash
sudo cp Makefile.config.example Makefile.config
```

复制一份的原因是编译 caffe 时需要的是 Makefile.config 文件，而Makefile.config.example 只是caffe 给出的配置文件例子，不能用来编译 caffe。

参考我的Makefile.config：

```bash
## Refer to http://caffe.berkeleyvision.org/installation.html
# Contributions simplifying and improving our build system are welcome!

# cuDNN acceleration switch (uncomment to build with cuDNN).
# USE_CUDNN := 1

# CPU-only switch (uncomment to build without GPU support).
# CPU_ONLY := 1

# uncomment to disable IO dependencies and corresponding data layers
# USE_OPENCV := 0
# USE_LEVELDB := 0
# USE_LMDB := 0

# uncomment to allow MDB_NOLOCK when reading LMDB files (only if necessary)
#	You should not set this flag if you will be reading LMDBs with any
#	possibility of simultaneous read and write
# ALLOW_LMDB_NOLOCK := 1

# Uncomment if you're using OpenCV 3
OPENCV_VERSION := 3

# To customize your choice of compiler, uncomment and set the following.
# N.B. the default for Linux is g++ and the default for OSX is clang++
# CUSTOM_CXX := g++

# CUDA directory contains bin/ and lib/ directories that we need.
CUDA_DIR := /usr/local/cuda
# On Ubuntu 14.04, if cuda tools are installed via
# "sudo apt-get install nvidia-cuda-toolkit" then use this instead:
# CUDA_DIR := /usr

# CUDA architecture setting: going with all of them.
# For CUDA < 6.0, comment the *_50 lines for compatibility.
CUDA_ARCH := -gencode arch=compute_20,code=sm_20 \
		-gencode arch=compute_20,code=sm_21 \
		-gencode arch=compute_30,code=sm_30 \
		-gencode arch=compute_35,code=sm_35 \
		-gencode arch=compute_50,code=sm_50 \
		-gencode arch=compute_50,code=compute_50

# BLAS choice:
# atlas for ATLAS (default)
# mkl for MKL
# open for OpenBlas
BLAS := open
# Custom (MKL/ATLAS/OpenBLAS) include and lib directories.
# Leave commented to accept the defaults for your choice of BLAS
# (which should work)!
# BLAS_INCLUDE := /path/to/your/blas
# BLAS_LIB := /path/to/your/blas

# Homebrew puts openblas in a directory that is not on the standard search path
# BLAS_INCLUDE := $(shell brew --prefix openblas)/include
# BLAS_LIB := $(shell brew --prefix openblas)/lib

# This is required only if you will compile the matlab interface.
# MATLAB directory should contain the mex binary in /bin.
# MATLAB_DIR := /usr/local
# MATLAB_DIR := /Applications/MATLAB_R2012b.app

# NOTE: this is required only if you will compile the python interface.
# We need to be able to find Python.h and numpy/arrayobject.h.
PYTHON_INCLUDE := /usr/include/python2.7 \
		/usr/lib64/python2.7/site-packages/numpy/core/include
# Anaconda Python distribution is quite popular. Include path:
# Verify anaconda location, sometimes it's in root.
# ANACONDA_HOME := $(HOME)/anaconda
# PYTHON_INCLUDE := $(ANACONDA_HOME)/include \
		# $(ANACONDA_HOME)/include/python2.7 \
		# $(ANACONDA_HOME)/lib/python2.7/site-packages/numpy/core/include \

# We need to be able to find libpythonX.X.so or .dylib.
PYTHON_LIB := /usr/lib
# PYTHON_LIB := $(ANACONDA_HOME)/lib

# Homebrew installs numpy in a non standard path (keg only)
# PYTHON_INCLUDE += $(dir $(shell python -c 'import numpy.core; print(numpy.core.__file__)'))/include
# PYTHON_LIB += $(shell brew --prefix numpy)/lib

# Uncomment to support layers written in Python (will link against Python libs)
WITH_PYTHON_LAYER := 1

# Whatever else you find you need goes here.
INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial/
LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /usr/lib/x86_64-linux-gnu /usr/lib/x86_64-linux-gnu/hdf5/serial

# If Homebrew is installed at a non standard location (for example your home directory) and you use it for general dependencies
# INCLUDE_DIRS += $(shell brew --prefix)/include
# LIBRARY_DIRS += $(shell brew --prefix)/lib

# Uncomment to use `pkg-config` to specify OpenCV library paths.
# (Usually not necessary -- OpenCV libraries are normally installed in one of the above $LIBRARY_DIRS.)
# USE_PKG_CONFIG := 1

BUILD_DIR := build
DISTRIBUTE_DIR := distribute

# Uncomment for debugging. Does not work on OSX due to https://github.com/BVLC/caffe/issues/171
# DEBUG := 1

# The ID of the GPU that 'make runtest' will use to run unit tests.
TEST_GPUID := 0

# enable pretty build (comment to see full commands)
Q ?= @
```

可以开始编译了，在 caffe 目录下执行 ：

```bash
make all -j8
sudo make runtest -j8
sudo make pycaffe -j8 # 安装 pycaffe notebook 接口环境
```

# [darknet](https://pjreddie.com/darknet/yolo/)配置

环境: Ubuntu16.04 + Titan X + Cuda8.0 + OpenCV3.1 + Python2.7

请参考前文。

```bash
git clone https://github.com/pjreddie/darknet
cd darknet
# 配置Makefile
# GPU=1
# CUDNN=0
# OPENCV=1
# DEBUG=0
make
```

可能出现的报错：

（1）error:/usr/bin/ld: 找不到 -lippicv</br >
collect2: error: ld returned 1 exit status </br >
Makefile:82: recipe for target 'libdarknet.so' failed </br >

解决方法：找到-lippicv对应的库（libippicv.a），该库位于 安装目录./opencv-3.1.0/3rdparty/ippicv/unpack/ippicv_lnx/lib/intel64文件夹下 ，进入该文件夹下执行

```bash
sudo cp sudo cp libippicv.a /usr/local/lib/
```   

继续执行make 即可。

（2）找不到nvcc

解决方法：修改darknet下的Makefile文件，将其中的NVCC=nvcc改为/usr/local/cuda-*/bin/nvcc即安装的cuda版本信息

保存  继续执行make 即可。

下载权重测试:

```bash
wget http://pjreddie.com/media/files/yolo.weights  
./darknet yolo test cfg/yolo.cfg yolo.weights data/dog.jpg   
./darknet detect cfg/yolo.cfg yolo.weights data/dog.jpg
```

- [YOLOv2训练自己的数据集（VOC格式）](https://blog.csdn.net/ch_liu23/article/details/53558549)
- [YOLOv2目标检测_单目标_训练自己数据全过程（自用）](https://blog.csdn.net/jozeeh/article/details/79087311)

# 参考

1. https://www.liaohuqiu.net/cn/posts/ssh-public-key-auto-login/
2. https://blog.csdn.net/asukasmallriver/article/details/72927860
3. https://blog.csdn.net/u011440558/article/details/78358447
4. https://www.mtyun.com/library/how-to-install-caffe-on-centos7
5. https://blog.csdn.net/qq_28413479/article/details/76377184
6. https://blog.csdn.net/yhaolpz/article/details/71375762
7. https://www.jianshu.com/p/10ed332caf07