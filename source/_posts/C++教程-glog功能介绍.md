---
title: C++教程-glog功能介绍
date: 2018-04-03 20:51:26
categories:
- 编程
- C++
- glog
tags:
- 编程
- C++
copyright:
---

# 概述

Google glog是一个基于程序级记录日志信息的c++库，编程使用方式与c++的stream操作类似，例：

       LOG(INFO) << "Found " << num_cookies << " cookies";
       
“LOG”宏为日志输出关键字，“INFO”为严重性程度。

主要支持功能：

- 参数设置，以命令行参数的方式设置标志参数来控制日志记录行为；
- 严重性分级，根据日志严重性分级记录日志；
- 可有条件地记录日志信息；
- 条件中止程序。丰富的条件判定宏，可预设程序终止条件；
- 异常信号处理。程序异常情况，可自定义异常处理过程；
- 支持debug功能。可只用于debug模式；
- 自定义日志信息；
- 线程安全日志记录方式；
- 系统级日志记录；
- google perror风格日志信息；
- 精简日志字符串信息。

# 安装

最新版本：0.3.1  http://code.google.com/p/google-glog/

安装只需：

```bash
./configure
make
make install
```

简单示例:

```C++
#include <iostream>
#include "glog/logging.h"   // glog 头文件

using namespace std;

int main(int argc, char** argv) {
  google::InitGoogleLogging(argv[0]);    // 初始化
  // FLAGS_log_dir=".";   
  LOG(INFO) << "hello glog";     // 打印log：“hello glog.  类似于C++ stream。
}
```

Makefile:

LIB=$(HOME)/install/glog/lib    #glog 安装路径

INCLUDE=$(HOME)/install/glog/include

test_glog : main.o

        g++ -o $@ $^ -L$(LIB) -lglog –lpthread   #-lpthread 因为glog在多线程中需要一些锁机制。

main.o: main.cpp

        g++ -c -o $@ $^ -I$(INCLUDE)

说明：

glog 默认对log分为4级： INFO,  WARNING,  ERROR,  FATAL.  打印log语句类似于C++中的stream，实际上LOG(INFO) 宏返回的是一个继承自std::ostrstream类的对象。

编译运行上述demo， glog默认会在/tmp/目录下生成log日志文件：test_glog.search-x2.username.log.INFO.20111003-161341.2083

文件名各字段对应含义为：

<program name>.<hostname>.<user name>.log.<severity level>.<date>.<time>.<pid>

其中：

1），<program name> 其实对应google::InitGoogleLogging(argv[0])；中的argv[0]，即通过改变google::InitGoogleLogging的参数可以修改日志文件的名称。

2），每个级别的日志会输出到不同的文件中。并且高级别日志文件会同样输入到低级别的日志文件中。 即：FATAL的信息会同时记录在INFO，WARNING，ERROR，FATAL日志文件中。默认情况下，glog还会将会将FATAL的日志发送到stderr中。

现在的问题是：log总不能都打印到/tmp/目录下吧。

参数设置：

不同于log4系列的日志系统通过配置文件的方式， glog采用命令的方式来来配置参数。在glog的官方文档里，提到如下两种方式来配置参数（以修改日志目录为例：）

1），gflags：

./your_application --log_dir=.

（gflags 我还没有使用过）

2），export 修改环境变量，如下所示：修改GLOG_log_dir为上层目录

3）以上两种方法都需要使用命令行，除此之外，还可以直接在程序中指定（官方文档中没有提到， glog源代码中也不鼓励这么用，但确实是可行的）：

在glog/logging.h 头文件287---350行，有诸如“GLOG_log_dir”等变量的宏定义， 则其GLOG_log_dir实际为FLAGS_log_dir,  因此只需要在程序中设置FLAGS_log_dir的值即可。其他变量类似。取消main.cpp中的注释行“// FLAGS_log_dir="."; ” 试试吧

# 参考

- http://blog.51cto.com/mengjh/546766
- http://www.cnblogs.com/foreveryl/archive/2011/10/14/2212265.html