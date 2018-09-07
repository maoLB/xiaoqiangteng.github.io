---
title: C++教程-cmake
date: 2018-04-03 22:14:54
categories:
- 编程
- C++
- cmake
tags:
- 编程
- C++
copyright:
---

# Cmake入门

## CMake编译原理

CMake是一种跨平台编译工具，比make更为高级，使用起来要方便得多。CMake主要是编写CMakeLists.txt文件，然后用cmake命令将CMakeLists.txt文件转化为make所需要的makefile文件，最后用make命令编译源码生成可执行程序或共享库（so(shared object)）。因此CMake的编译基本就两个步骤：

```bash
cmake ..
make
```

cmake  指向CMakeLists.txt所在的目录，例如cmake .. 表示CMakeLists.txt在当前目录的上一级目录。cmake后会生成很多编译的中间文件以及makefile文件，所以一般建议新建一个新的目录，专门用来编译，例如:

```bash
mkdir build
cd build
cmake ..
make
```

make根据生成makefile文件，编译程序。

## 使用Cmake编译程序

### 源码文件介绍

本文以一个例子入手介绍，即开平方。

```bash
├── CMakeLists.txt
└── src
    ├── main.cpp
    ├── test_math.cpp
    └── test_math.h
```

其中，src目录存放所有的源代码，即test_math.cpp、test_math.h和main.cpp。每个源代码文件内容如下：

test_math.h:

```c++
#ifndef TEST4_TEST_MATH_H
#define TEST4_TEST_MATH_H

#include <math.h>

double cal_sqrt(double value);

#endif //TEST4_TEST_MATH_H
```

test_math.cpp:

```c++
#include "test_math.h"

double cal_sqrt(double value)
{
    return sqrt(value);
}
```

main.cpp:

```c++
#include <iostream>
#include "test_math.h"

int main() {
    double a = 49.0;
    double b = 0.0;
    b = cal_sqrt(a);
    printf("sqrt result:%f\n",b);
    return 0;
}
```

### 编写CMakeLists.txt

CMakeLists.txt文件，如下所示：

```c++
#1.cmake verson，指定cmake版本
cmake_minimum_required(VERSION 3.8)

#2.project name，指定项目的名称，一般和项目的文件夹名称对应
project(test4)

#3.head file path，头文件目录
#INCLUDE_DIRECTORIES(include)

#4.source directory，源文件目录
AUX_SOURCE_DIRECTORY(src DIR_SRCS)

#5.set environment variable，设置环境变量，编译用到的源文件全部都要放到这里，否则编译能够通过，但是执行的时候会出现各种问题，比如"symbol lookup error xxxxx , undefined symbol"
SET(TEST_MATH ${DIR_SRCS})

#7.add link library，添加可执行文件所需要的库，比如我们用到了libm.so（命名规则：lib+name+.so），就添加该库的名称
#TARGET_LINK_LIBRARIES(${PROJECT_NAME} m)

#6.add executable file，添加要编译的可执行文件
ADD_EXECUTABLE(${PROJECT_NAME} ${TEST_MATH})

set(CMAKE_CXX_STANDARD 11)

```

### 编译和运行程序

由于编译中出现许多中间的文件，因此最好新建一个独立的目录build，在该目录下进行编译，编译步骤如下所示：

```c++
mkdir build
cd build
cmake ..
make
```

build下生成的目录结构如下：

```bash
├── CMakeLists.txt
├── cmake-build-debug
│   ├── CMakeCache.txt
│   ├── CMakeFiles
│   │   ├── 3.8.2
│   │   │   ├── CMakeCCompiler.cmake
│   │   │   ├── CMakeCXXCompiler.cmake
│   │   │   ├── CMakeDetermineCompilerABI_C.bin
│   │   │   ├── CMakeDetermineCompilerABI_CXX.bin
│   │   │   ├── CMakeSystem.cmake
│   │   │   ├── CompilerIdC
│   │   │   │   ├── CMakeCCompilerId.c
│   │   │   │   ├── a.out
│   │   │   │   └── tmp
│   │   │   └── CompilerIdCXX
│   │   │       ├── CMakeCXXCompilerId.cpp
│   │   │       ├── a.out
│   │   │       └── tmp
│   │   ├── CMakeDirectoryInformation.cmake
│   │   ├── CMakeOutput.log
│   │   ├── CMakeTmp
│   │   ├── Makefile.cmake
│   │   ├── Makefile2
│   │   ├── TargetDirectories.txt
│   │   ├── clion-environment.txt
│   │   ├── clion-log.txt
│   │   ├── cmake.check_cache
│   │   ├── feature_tests.bin
│   │   ├── feature_tests.c
│   │   ├── feature_tests.cxx
│   │   ├── progress.marks
│   │   └── test4.dir
│   │       ├── CXX.includecache
│   │       ├── DependInfo.cmake
│   │       ├── build.make
│   │       ├── cmake_clean.cmake
│   │       ├── depend.internal
│   │       ├── depend.make
│   │       ├── flags.make
│   │       ├── link.txt
│   │       ├── progress.make
│   │       └── src
│   │           ├── main.cpp.o
│   │           └── test_math.cpp.o
│   ├── Makefile
│   ├── cmake_install.cmake
│   ├── test4
│   └── test4.cbp
└── src
    ├── main.cpp
    ├── test_math.cpp
    └── test_math.h
```

# 参考
- http://www.cnblogs.com/cv-pr/p/6206921.html