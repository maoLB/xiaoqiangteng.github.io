---
title: colmap源码解读
date: 2018-04-03 21:10:31
categories:
- 编程
- colmap
tags:
- 编程
- colmap
copyright:
---

# feature_extractor.cc

## #include <memory>

1 auto_ptr

C++的auto_ptr所做的事情，就是动态分配对象以及当对象不再需要时自动执行清理。

使用std::auto_ptr，要#include <memory>。

double *p = new double;//为指针分配内存

std::auto_ptr<double> autop(p);

//继承性指针，必须依赖上面的指针p

//创建智能指针管理指针p指向的内存，可以自动释放内存，不用delete就可以自动删除

//搭配原生指针p使用，不用担心多delete或者少delete

//auto_ptr更多用于管理类和对象的内存

2 unique_ptr

unique_ptr是一种定义在<memory>中的智能指针(smart pointer)。它持有对对象的独有权——两个unique_ptr不能指向一个对象，不能进行复制操作只能进行移动操作。unique_ptr在超出作用域，即以下情况时它指向的对象会被摧毁：

unique_ptr指向的对象被破坏

对象通过operator=（）或reset（）被指定到另一个指针）

unique_ptr还可能没有对象，这种情况被称为empty。

//C++11新指针

//std::unique_ptr<指针指向的变量数据类型>指针变量名(new 指针指向的变量数据类型);

std::unique_ptr<double>pdb(new double);