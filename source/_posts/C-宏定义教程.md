---
title: C++宏定义教程
date: 2018-09-14 10:46:04
categories:
- 编程
- C++
tags:
- 编程
- C++
copyright:
---

<script type="text/javascript"
   src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

# #define基本用法

\#define命令是C语言中的一个宏定义命令，它用来将一个标识符(宏名)定义为一个字符串，该标识符被称为宏名，被定义的字符串称为替换文本。程序编译之前，编译的时候所有的宏名都会被定义的字符串替换，这便是宏替换。

理解宏定义的关键在于 “替换”。

该命令有两种格式：一种是简单的宏定义，另一种是带参数的宏定义。

## 简单的宏定义：

```C++
#define PI 3.14
float pi2 = PI * 2;//pi2 = 6.28
```

## 带参数的宏定义 

```C++
#define <宏名> (<参数表>) <宏体>
#define AddOne(x) (x+1) 
float pi2 = PI * 2;//pi2 = 6.28
pi2 = AddOne(pi2);// pi2 = 7.28
```

# 宏定义的特点

1）宏名一般用大写，且末尾不加分号。

（2）宏定义的参数是无类型的，不做语法检查，不做表达式求解，只做替换。

（3）宏定义通常在文件的最开头，可以使用\#undef 宏名 命令终止宏定义的作用域。

(4）宏定义可以嵌套，但字符串” “中永远不包含宏。

（5）宏展开使源程序变长，函数调用不会；宏展开不占运行时间，只占编译时间，函数调用占运行时间（分配内存、保留现场、值传递、返回值）。

（6）函数调用在编译后程序运行时进行，并且分配内存。宏替换在编译前进行，不分配内存。

（7）使用宏可提高程序的通用性和易读性，减少不一致性，减少输入错误和便于修改。例如：数组大小常用宏定义，常量pi常用宏定义。

# 宏替换发生的时机

为了能够真正理解#define的作用，让我们来了解一下对C语言源程序的处理过程。当我们在一个集成的开发环境如Turbo C中将编写好的源程序进行编译时，实际经过了预处理、编译、汇编和连接几个过程。其中预处理器产生编译器的输出，它实现以下的功能：

（1）文件包含

可以把源程序中的#include 扩展为文件正文，即把包含的.h文件找到并展开到#include 所在处。

（2）条件编译

预处理器根据#if和#ifdef等编译命令及其后的条件，将源程序中的某部分包含进来或排除在外，通常把排除在外的语句转换成空行。

（3）宏展开

预处理器将源程序文件中出现的对宏的引用展开成相应的宏定义，即本文所说的#define的功能，由预处理器来完成。经过预处理器处理的源程序与之前的源程序有所有不同，在这个阶段所进行的工作只是纯粹的替换与展开，没有任何计算功能，所以在学习#define命令时只要能真正理解这一点，这样才不会对此命令引起误解并误用。

# 宏替换错误举例

只要严格遵守“直接替换”，就不会出现下面的问题

```C++
#define Square(x) x*x

float temp = Square(3+3);
//程序的本意可能是要计算6*6=36，但由于宏定义执行的是直接替换，本身并不做计算，因此实际的结果为 3+3*3+3=15
//想要避免这个问题，只需要修改如下：
#define Square(x) ((x)*(x))
```

# define中的三个特殊符号：##，#，#@

```C++
/*x连接y，例如：int n = Conn(123,456); 结果就是n=123456;char* str = Conn("asdf", "adf"); /*结果就是 str = "asdfadf";*/
#define Conn(x,y) x##y

/*给x加上单引号，结果返回是一个const char。例如：char a = ToChar(1);结果就是a='1';做个越界试验char a = ToChar(123);结果就错了;但是如果你的参数超过四个字符，编译器就给给你报错了！error C2015: too many characters in constant   ：P */

#define ToChar(x) #@x


// x加双引号,例如：char* str = ToString(123132);就成了str="123132";
#define ToString(x) #x
```

# 常用宏定义

(1) 防止一个头文件被重复包含

```C++
#ifndef BODYDEF_H 

#define BODYDEF_H 

 //头文件内容 
#endif
```

(2) 得到指定地址上的一个字节或字

```C++
#define MEM_B( x ) ( *( (byte *) (x) ) ) 

#define MEM_W( x ) ( *( (word *) (x) ) )

//例如：
int bTest = 0x123456;

byte m = MEM_B((&bTest));/*m=0x56*/
int n = MEM_W((&bTest));/*n=0x3456*/
```

(3) 得到一个field在结构体(struct)中的偏移量

```C++
#define OFFSETOF( type, field ) ( (size_t) &(( type *) 0)-> field )
```

(4) 得到一个结构体中field所占用的字节数

```C++
#define FSIZ( type, field ) sizeof( ((type *) 0)->field )
```

(5) 得到一个变量的地址（word宽度）

```C++
#define B_PTR( var ) ( (byte *) (void *) &(var) ) 

#define W_PTR( var ) ( (word *) (void *) &(var) )
```

(6) 将一个字母转换为大写

```C++
#define UPCASE( c ) ( ((c) >= ''a'' && (c) <= ''z'') ? ((c) - 0x20) : (c) )
```

(7) 防止溢出的一个方法

```C++
#define INC_SAT( val ) (val = ((val)+1 > (val)) ? (val)+1 : (val))
```

(8) 返回数组元素的个数

```C++
#define ARR_SIZE( a ) ( sizeof( (a) ) / sizeof( (a[0]) ) )
```

(9) 使用一些宏跟踪调试

ANSI标准说明了五个预定义的宏名。它们是：

__LINE__：在源代码中插入当前源代码行号；

__FILE__：在源文件中插入当前源文件名；

__DATE__：在源文件中插入当前的编译日期

__TIME__：在源文件中插入当前编译时间；

__STDC__：当要求程序严格遵循ANSI C标准时该标识被赋值为1；

__cplusplus：当编写C++程序时该标识符被定义

# 参考

- [C++ 宏定义](https://blog.csdn.net/shuzfan/article/details/52860664)

