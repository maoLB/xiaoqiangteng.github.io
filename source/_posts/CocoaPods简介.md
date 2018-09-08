---
title: CocoaPods简介
date: 2018-09-08 9:56:10
categories:
- 编程
- IOS
tags:
- 编程
- IOS
copyright:
---

# CocoaPods简介

CocoaPods是iOS的包管理工具。在开发iOS项目时，经常会使用第三方开源库，手动引入流程复杂，并且库之间还存在依赖关系，更增加了手动管理的难度。开源库如果升级了，你也想用最新版本，还需要重新手动导入，这大大增加了工作量。但用了CocoaPods后，安装和升级都只是一句命令的事情，让你可以专于业务本身。

# CocoaPods安装

## 安装Ruby

CocoaPods基于Ruby语言开发而成，因此安装CocoaPods前需要安装Ruby环境。幸运的是Mac系统默认自带Ruby环境，如果没有请自行查找安装。检测是否安装Ruby：

```bash
$ gem -v
2.0.14
```

安装则会提示当前Ruby版本。gem介绍：gem是一个管理Ruby库和程序的标准包，它通过Ruby Gem（如 http://rubygems.org/ ）源来查找、安装、升级和卸载软件包，非常的便捷。

## 更换gem源

因为国内网络的问题导致gem源间歇性中断，原因你懂的。因此我们需要更换gem源，使用淘宝的gem源https://ruby.taobao.org/。

第一步：移动默认的源:

```bash
gem sources --remove https://rubygems.org/
```

第二步：指定淘宝的源:

```bash
gem sources -a https://ruby.taobao.org/
```

第三步：查看指定的源是不是淘宝源:

```bash
$ gem sources -l
*** CURRENT SOURCES ***

https://ruby.taobao.org/
```

如果是https://ruby.taobao.org/，则更换成功。

## 安装CocoaPods

确认改成淘宝源后执行以下命令进行安装：

```bash
sudo gem install cocoapods
```

稍等片刻即可安装完成，输入以下命令检测是否安装成功：

```bash
$ pod --version
0.39.0
```

成功则会提示CocoaPods版本。

# CocoaPods使用实例

首先新建一个iOS工程MyDemo，在该工程中演示CocoaPods的使用。

1. 进入工程的根目录，创建Podfile文件.

2. 根据需要，我们可以在Podfile文件中写入我们需要的第三方库，这里以AFNetworking和MJRefresh为例，Podfile内容如下：

	```pod
	platform :ios, '7.0'
	pod 'AFNetworking', '~> 3.0'
	pod 'MJRefresh','~> 3.1'
	# 这段代码的意思是，当前类库支持的iOS最低版本是iOS 7.0, 要下载的两个类库的版本分别为 3.0、3.1。
```

3. 这时候，你就可以利用CocoPods下载AFNetworking和MJRefresh类库了。在终端中进入工程根目录，运行以下命令：

```bash
$ cd /Users/myl/Desktop/IOS/MyDemo
$ pod install
Analyzing dependencies
Downloading dependencies
Installing AFNetworking (3.0.4)
Installing MJRefresh (3.1.0)
Generating Pods project
Integrating client project

[!] Please close any current Xcode sessions and use `MyDemo.xcworkspace` for this project from now on.
Sending stats
```

