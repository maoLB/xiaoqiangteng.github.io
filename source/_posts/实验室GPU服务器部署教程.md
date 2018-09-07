---
title: 实验室GPU服务器部署教程
date: 2018-03-29 22:21:46
categories:
- 编程
- 深度学习
- 环境配置
tags:
- 编程
- 深度学习
- 环境配置
copyright:
---

# [Docker](https://docs.docker.com/install/linux/ubuntu/) 安装

参考：https://docs.docker.com/install/linux/docker-ce/ubuntu/#set-up-the-repository

1. Prerequisites

	```bash
	To install Docker CE, you need the 64-bit version of one of these Ubuntu versions:
	
	Artful 17.10 (Docker CE 17.11 Edge and higher only)
	Xenial 16.04 (LTS)
	Trusty 14.04 (LTS)
	```

2. Uninstall old versions

	```bash
	sudo apt-get remove docker docker-engine docker.io
	```

3. Install Docker CE

	```bash
	$ sudo apt-get update
	# Install packages to allow apt to use a repository over HTTPS
	$ sudo apt-get install \
	    apt-transport-https \
	    ca-certificates \
	    curl \
	    software-properties-common
	# Add Docker’s official GPG key:
	$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
	$ sudo apt-key fingerprint 0EBFCD88
	# Use the following command to set up the stable repository.
	$ sudo add-apt-repository \
	   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
	   $(lsb_release -cs) \
	   stable"
	$ sudo apt-get update
	# Install the latest version of Docker CE
	$ sudo apt-get install docker-ce
	# Verify that Docker CE is installed correctly by running the hello-world image.
	$ sudo docker run hello-world 
	```
	
4. UPGRADE DOCKER CE

	```bash
	sudo apt-get update
	```
	
5. Uninstall Docker CE

	```bash
	$ sudo apt-get purge docker-ce
	$ sudo rm -rf /var/lib/docker
	```
	
# Docker 备份、恢复和迁移

## 备份

首先，为了备份Docker中的容器，我们会想看看我们想要备份的容器列表。要达成该目的，我们需要在我们运行着Docker引擎，并已创建了容器的Linux机器中运行 docker ps 命令。

```bash
# docker ps
```

在此之后，我们要选择我们想要备份的容器，然后去创建该容器的快照。我们可以使用 docker commit 命令来创建快照。

```bash
# docker commit -p 30b8f18f20b4 container-backup
```

该命令会生成一个作为Docker镜像的容器快照，我们可以通过运行 docker images 命令来查看Docker镜像，如下。

```bash
# docker images
```

正如我们所看见的，上面做的快照已经作为Docker镜像保存了。现在，为了备份该快照，我们有两个选择，一个是我们可以登录进Docker注册中心，并推送该镜像；另一个是我们可以将Docker镜像打包成tar包备份，以供今后使用。

如果我们想要在[Docker注册中心](https://hub.docker.com/)上传或备份镜像，我们只需要运行 docker login 命令来登录进Docker注册中心，然后推送所需的镜像即可。

```bash
# docker login
# docker tag a25ddfec4d2a arunpyasi/container-backup:test
# docker push arunpyasi/container-backup
```

如果我们不想备份到docker注册中心，而是想要将此镜像保存在本地机器中，以供日后使用，那么我们可以将其作为tar包备份。要完成该操作，我们需要运行以下 docker save 命令。

```bash
# docker save -o ~/container-backup.tar container-backup
```

要验证tar包是否已经生成，我们只需要在保存tar包的目录中运行 ls 命令即可。

## 恢复容器

接下来，在我们成功备份了我们的Docker容器后，我们现在来恢复这些制作了Docker镜像快照的容器。如果我们已经在注册中心推送了这些Docker镜像，那么我们仅仅需要把那个Docker镜像拖回并直接运行即可。

```bash
# docker pull arunpyasi/container-backup:test
```

但是，如果我们将这些Docker镜像作为tar包文件备份到了本地，那么我们只要使用 docker load 命令，后面加上tar包的备份路径，就可以加载该Docker镜像了。

```bash
# docker load -i ~/container-backup.tar
```

现在，为了确保这些Docker镜像已经加载成功，我们来运行 docker images 命令。

```bash
# docker images
```

在镜像被加载后，我们将用加载的镜像去运行Docker容器。

```bash
# docker run -d -p 80:80 container-backup
```

## 迁移Docker容器

迁移容器同时涉及到了上面两个操作，备份和恢复。我们可以将任何一个Docker容器从一台机器迁移到另一台机器。在迁移过程中，首先我们将把容器备份为Docker镜像快照。然后，该Docker镜像或者是被推送到了Docker注册中心，或者被作为tar包文件保存到了本地。如果我们将镜像推送到了Docker注册中心，我们简单地从任何我们想要的机器上使用 docker run 命令来恢复并运行该容器。但是，如果我们将镜像打包成tar包备份到了本地，我们只需要拷贝或移动该镜像到我们想要的机器上，加载该镜像并运行需要的容器即可。

# Docker SSH 访问

假设我们已经pull了一个docker 镜像，如下图所示的tensorflow/tensorflow。

## 启动容器

	```bash
	docker run --name my-tensorflow -it -p 8888:8888 -v ~/tensorflow:/test/data tensorflow/tensorflow
	# --name：创建的容器名，即my-tensorflow
	# -it：保留命令行运行
	# p 8888:8888：将本地的8888端口和http://localhost:8888/映射
	# -v ~/tensorflow:/test/data:将本地的~/tensorflow挂载到容器内的/# test/data下
	# tensorflow/tensorflow ：默认是tensorflow/tensorflow:latest,指定使用的镜像
	```
	
如：

```bash
docker run -it --name tf tensorflow/tensorflow /bin/bash
# 这样我们就创建了名为tf的容器
docker start tf
docker attach tf
```

## 修改容器的root密码

```bash
apt-get install vim -y
apt-get install openssh-server -y
apt-get install passwd
passwd root
```

## 修改ssh配置

```bash
vim /etc/ssh/sshd_config
# 修改PermitRootLogin yes  
UsePAM no
```

## 启动ssh服务

```bash
service ssh start
```

## 退出容器

```bash
exit
```

## 提交容器成为新的镜像

```bash
例如叫做ubuntu-ssh，输入docker commit 容器ID ubuntu-ssh
```

## 启动这个镜像的容器，并映射本地的一个闲置的端口

```bash
docker run -it -p 50001:22 tf-ssh /bin/bash
```

## ssh登录

```bash
ssh root@127.0.0.1 -p 50001
```

## Docker后台运行




# 阿里云加速器设置

由于官方Docker Hub网络速度较慢，这里使用阿里云提供的[Docker Hub](https://hub.docker.com/). 需要配置阿里云加速器，官方说明如下：

1. 针对Docker客户端版本大于1.10的用户： 

	```bash
	# 您可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器
	$ sudo mkdir -p /etc/docker
	$ sudo tee /etc/docker/daemon.json <<-‘EOF’ 
	  { 
	  “registry-mirrors”: [“https://fird1mfg.mirror.aliyuncs.com“] 
	  } 
	  EOF
	$ sudo systemctl daemon-reload
	$ sudo systemctl restart docker
	```
	
2. 针对Docker客户的版本小于等于1.10的用户或者想配置启动参数，可以使用下面的命令将配置添加到docker daemon的启动参数中.

	```bash
	# Ubuntu 12.04 14.04的用户:
	$ echo “DOCKER_OPTS=/”$DOCKER_OPTS –registry-mirror=https://fird1mfg.mirror.aliyuncs.com/”” | sudo tee -a /etc/default/docker
	$ sudo service docker restart
	
	# Ubuntu 15.04 16.04的用户
	$ sudo mkdir -p /etc/systemd/system/docker.service.d
	$ sudo tee /etc/systemd/system/docker.service.d/mirror.conf <<-‘EOF’ 
	[Service] 
	ExecStart=/usr/bin/docker daemon -H fd:// –registry-mirror=https://fird1mfg.mirror.aliyuncs.com 
	EOF
	$ sudo systemctl daemon-reload
	$ sudo systemctl restart docker
	```

# [NVIDIA-Docker](https://github.com/NVIDIA/nvidia-docker/wiki)安装

1. Prerequisties

	```bash
	GNU/Linux x86_64 with kernel version > 3.10 
	Docker >= 1.9 (official docker-engine, docker-ce or docker-ee only) 
	NVIDIA GPU with Architecture > Fermi (2.1) 
	NVIDIA drivers >= 340.29 with binary nvidia-modprobe (驱动版本与CUDA计算能力相关)
	```

2. CUDA与NVIDIA driver安装 [cuda](https://developer.nvidia.com/cuda-downloads)

	```bash
	处理NVIDIA-Docker依赖项 NVIDIA drivers >= 340.29 with binary nvidia-modprobe 要求. 
	根据显卡，下载对应版本的CUDA并进行安装.
	```

3. NVIDIA-Docker安装

	```bash
	#Install nvidia-docker and nvidia-docker-plugin
	
	wget -P /tmp https://github.com/NVIDIA/nvidia-docker/releases/download/v1.0.1/nvidia-docker_1.0.1-1_amd64.deb
	sudo dpkg -i /tmp/nvidia-docker*.deb && rm /tmp/nvidia-docker*.deb
	#Test nvidia-smi
	
	sudo nvidia-docker run –rm nvidia/cuda nvidia-smi
	```
	
4. 默认用nvdia-docker替代docker命令：

	```bash
	echo 'alias docker=nvidia-docker' >> ~/.bashrc
	bash
	```



# 参考
1. https://jingyan.baidu.com/article/a3aad71aa180e7b1fa009676.html
2. https://github.com/ufoym/deepo#Installation
3. https://hub.docker.com/r/ufoym/deepo/
4. https://github.com/fatedier/frp/blob/master/README_zh.md#frp-%E7%9A%84%E4%BD%9C%E7%94%A8
5. https://ranpox.github.io/2018/01/14/notification-of-gpu-server/