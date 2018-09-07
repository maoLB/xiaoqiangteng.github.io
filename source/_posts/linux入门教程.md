---
title: linux入门教程
date: 2018-03-25 15:44:04
categories:
- 编程
- Linux
- 入门
tags:
- 编程
- Linux
copyright:
---

# Linux入门教程

## SSH 远程连接 Linux

### Linux端配置

``` bash
sudo apt-get install openssh-server
```

查看IP地址：
``` bash
ifconfig
```

### Mac端配置

``` bash
ssh 用户名@IP地址 #输入密码确认
```

### 免密登录

Mac端：

``` bash
ssh-keygen -t rsa -f test -C "test-key" # 一直回车
cat test.pub # 查看公钥内容
```

配置公钥到服务器：将公钥内容添加到服务器的~/.ssh/authorized_keys 文件中.

例子：

```bash
scp   /home/yourname/.ssh/authorized_keys yourname@192.168.38.58:/home/yourname/.ssh/  
```

alias 实现命令快速登陆：做好配置之后，通过ssh可以直接登录了。对经常登录的服务器，可以将ssh登录命令的alias加到 ~/.bash_profile文件中。

``` bash
$ cat ~/.bash_profile | grep 101
alias to-101='ssh huqiu@192.168.154.101'
```

登录的时候:

``` bash
$ to-101
```

无法登录一般的原因：

- 客户端的私钥和公钥文件位置必须位于 ~/.ssh 下。

- 确保双方 ~/.ssh 目录，父目录，公钥私钥，authorized_keys 文件的权限对当前用户至少要有执行权限，对其他用户最多只能有执行权限。

- 注意git登录，要求对公钥和私钥以及config文件，其他用户不能有任何权限。

- 服务器端 ~/.ssh/authorized_keys 文件名确保没错 :).

ssh-copy-id：ssh-copy-id 是一个小脚本，你可以用这个小脚本完成以上工作。这个脚本在linux系统上一般都有。

[ssh-keygen 基本用法](https://www.liaohuqiu.net/cn/posts/ssh-keygen-abc/)

1) 使用 ssh-keygen 时，请先进入到 ~/.ssh 目录，不存在的话，请先创建。并且保证 ~/.ssh 以及所有父目录的权限不能大于 711

2) 使用 ssh-kengen 会在~/.ssh/目录下生成两个文件，不指定文件名和密钥类型的时候，默认生成的两个文件是：

- id_rsa
- id_rsa.pub

第一个是私钥文件，第二个是公钥文件。

生成ssh key的时候，可以通过 -f 选项指定生成文件的文件名，如下:

``` bash
ssh-keygen -f test   -C "test key" # test - 文件名，"test key" - 备注
```

### SSH config

```bash
Host example                       # 关键词
HostName example.com           # 主机地址
User root                      # 用户名
IdentityFile ~/.ssh/id_ecdsa # 认证文件
Port 22                      # 指定端口
ControlMaster auto
ControlPath /tmp/%r@%h:%p
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

cmake时ippicv_linux_20151201.tgz总是不能成功下载，故cmake之前将./downloads/linux-808b791a6eac9ed78d32a7666804320e 文件拷贝至./opencv-3.1.0/3rdparty/ippicv/ 路径下(先执行一次cmake 命令生成文件路径，在将ippicv_linux_20151201.tgz复制进去) ippicv_linux_20151201.tgz下载链接:链接: https://pan.baidu.com/s/1jBBPxXX_NqCodS5bAln4-g 密码: x4sn

然后开始cmake，这里需要注意几个cmake的参数，比较重要。

```bash
sudo cmake -D CMAKE_BUILD_TYPE=Release -D CMAKE_INSTALL_PREFIX=/usr/local WITH_LIBV4L=ON ..
```

切记最后’..’两个点之前要加空格！！

## 把代码编译成可执行文件

这里官方推荐使用多进程编译，推荐七个进程：

```bash
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

# 参考

1. https://www.liaohuqiu.net/cn/posts/ssh-public-key-auto-login/
2. https://blog.csdn.net/asukasmallriver/article/details/72927860
3. https://blog.csdn.net/u011440558/article/details/78358447