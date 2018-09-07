---
title: linux-深度学习环境配置-Centos
date: 2018-04-08 20:57:33
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

# Centos 7 环境安装

下载[Centos 7](https://www.centos.org/download/)安装镜像，制作启动优盘。

Install CentOS 7 安装CentOS 7。

- 第一步，配置日期、语言和键盘。

- 第二步，选择-系统-安装位置，进入磁盘分区界面。选择-其它存储选项-分区-我要配置分区，点左上角的“完成”，进入下面的界面:

```bash
# 分区前先规划好:
# swap #交换分区，一般设置为内存的2倍
# / #剩余所有空间
# 挂载点：swap, 期望容量：2048
```

点左上角的“完成”，接受更改。

- 第三步，在这步中，你可以通过选择列表中安全配置来设置你的系统“安全策略Security Policy”，点击选择配置按钮来选择你想要的安全配置并点击“应用安全策略Apply security policy”按钮到 On。点击“完成Done”按钮后继续安装流程。

- 第四步，点击“软件选择Software Selection”按钮来配置你的基础机器环境。左边的列表是你可以选择安装桌面环境（Gnome、KDE Plasma 或者创意工作站）或者安装一个服务器环境（Web 服务器、计算节点、虚拟化主机、基础设施服务器、带图形界面的服务器或者文件及打印服务器）或者执行一个最小化的安装。为了随后能自定义你的系统，选择最小化安装并附加兼容库，点击“完成Done”按钮继续。对于完整的 Gnome 或者 KDE 桌面环境。选择：

	- GNOM Applications
	- Internet Applications
	- Compatibility Libries
	- Compatibility Libries

- 第五步，设置你的主机名并启用网络服务。点击“网络和主机名Network & Hostname”，在主机名中输入你的 FQDN（完整限定网域名称），如果你在局域网中有一个 DHCP 服务器，将以太网按钮从 OFF 切换到 ON 来激活网络接口。为了静态配置你的网络接口，点击“配置Configure”按钮，添加 IP 设置，并点击“保存Save”按钮来应用更改。完成后，点击“完成Done”按钮来回到主安装菜单。

- 第六步，最后检查下所有到目前为止的配置，如果一切没问题，点击“开始安装Begin Installation”按钮开始安装

# 基础配置

## Centos 7 更换阿里源

备份

```bash
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

下载新的CentOS-Base.repo 到/etc/yum.repos.d/

```bash
# CentOS 5
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-5.repo

# CentOS 6
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo

# CentOS 7
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

之后运行yum makecache生成缓存.

## 安装常用工具

```bash
yum -y install nano vim wget curl net-tools lsof gcc gcc-c++ll
```

等待安装完成即可。如果提示有错可以执行：

```bash
yum makecache
```

重建缓存即可。


# NVIDIA显卡驱动安装

## 检查是否安装了GPU

```bash
lspci | grep -i nvidia
```

## 安装kernel-devel和kernel-headers

```bash
yum install kernel-devel  
yum install kernel-headers
```

## 修改/etc/modprobe.d/blacklist.conf 文件，以阻止 nouveau 模块的加载

方法： 添加blacklist nouveau，注释掉blacklist nvidiafb（如果存在）
blacklist.conf不存在时，执行下面的脚本

```bash
# echo -e "blacklist nouveau\noptions nouveau modeset=0" > /etc/modprobe.d/blacklist.conf
```

## 重新建立initramfs image文件

```bash
# mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bak
# dracut /boot/initramfs-$(uname -r).img $(uname -r)
```

## 安装驱动

```bash
Ctrl + Alt +F2    #纯文本命令模式  
登陆----获取root权限  
init 3  
切换至安装包文件夹  
./NVIDIA-Linux-x86_64-375.66.run   #根据提示安装  
cuda_8.0.61_375.26_linux.run   #根据提示安装
```

## 安装cuda

```bash
$ sudo rpm -i cuda-repo-rhel7-8-0-local-ga2-8.0.61-1.x86_64.rpm
$ sudo yum clean all
$ sudo yum install cuda
```

报错了：

```bash
Error: Package: 1:nvidia-kmod-375.26-2.el7.x86_64 (cuda-8-0-local-ga2)
           Requires: dkms
 You could try using --skip-broken to work around the problem
 You could try running: rpm -Va --nofiles --nodigest
```

缺少2个包，装第一个：

```bash
sudo vim /etc/yum.repos.d/linuxtech.testing.repo
```

输入：

```bash
[linuxtech-testing]
name=LinuxTECH Testing
baseurl=http://pkgrepo.linuxtech.net/el6/testing/
enabled=0
gpgcheck=1
gpgkey=http://pkgrepo.linuxtech.net/el6/release/RPM-GPG-KEY-LinuxTECH.NET
```

```bash
sudo yum --enablerepo=linuxtech-testing install libvdpau
```

第二个：

```bash
yum -y install epel-release
yum -y install --enablerepo=epel dkms
```

## 配置环境变量

```bash
gedit ~/.bashrc 
#写入bashrc文件保存  
#gpu driver  
export CUDA_HOME=/usr/local/cuda-8.0  
export PATH=/usr/local/cuda-8.0/bin:$PATH  
export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64:$LD_LIBRARY_PATH  
export LD_LIBRARY_PATH="/usr/local/cuda-8.0/lib:${LD_LIBRARY_PATH}" 
source ~/.bashrc
```

## 测试

```bash
nvidia-smi
```

# 参考

- https://my.oschina.net/u/2449787/blog/778145
