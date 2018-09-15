---
title: C++回调机制教程
date: 2018-09-09 11:48:32
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

# 引言

在 C++ 开发过程中，经常会遇到需要一个类在遇到某种情况下（比如触发了一个行为等等）需要驱使另一类去做某些行为的需求（也就是回调机制）。本文参考如下博文：[C++简单实现回调机制](https://blog.csdn.net/u012814856/article/details/73294124)进行讲解。

# Demo

A和B的打招呼程序。

## 需求

> 现在有两个人，一位是 A 先生，另一位是 B 先生。 
现在要求当 A 先生给 B 先生打了招呼之后，B 先生立马回复 A，向 A 问好。

## 抽象

这个过程抽象为三个类进行模拟：

角色 | 功能
---- | ---
A | 向B打招呼
B |  当A向自己打招呼时，向A打招呼
main | 创造A和B对象和控制A和B的行为

当我们看到上述的分析后，应该想到这么三个内容：

A 的作用：A 在这里干了什么呢？他是触发一次回调行为的触发点，也就是因为 A 进行了打招呼的行为，才有了 B 的回复的行为。也就是说：A 是触发方

B 的作用：B 在这里干的就多了。他在得知 A 向自己打招呼了之后，进行了回复。也就是说：B 是驱使方

回复事件：这是我们的回调行为本身，当 A 进行了打招呼的行为，B 进行了回调事件指定的行为

主过程：它掌控着时空，提供上述所有行为的必要条件

那么分析至此，我们已经拥有了写出这个程序的一切条件，接下来让我们一步一步分析一步一步实现吧。

# 实现：A、B、Event、main

首先，我们需要定义一个抽象的事件类，用来定义回复事件：

```C++
// event class
class CEvent {
public:
    virtual void hiReply() = 0;
};
```

这个类是这个方法的核心：

> 这个事件类是一个抽象的类： 
1.驱使方应该继承这个类并实现其内的具体功能 
2.触发方应该将此类包含为一个私有变量，当遇到触发事件的时候，通过此指针（父类指针）调用其具体实现的方法（由驱使方实现） 
3.还需要注意的一点是，为了方便调用，这个类中的方法需要全部声明为公有的，也就是需要在前面加 public 修饰

然后，我们需要实现 A 类，也就是事件触发方：

```C++
// person A class , call an event
class A {
public:
    A() : m_pEvent(NULL) {}
    void sayHi() {
        cout << "A: hello B" << endl;
        if (nullptr != m_pEvent) {
            m_pEvent->hiReply();
        }
    }
    void setEvent(CEvent *event) {
        m_pEvent = event;
    }
private:
    CEvent *m_pEvent;
};
```

这个类是触发方：

- 将事件类作为其私有变量：用来接收 B （事件类的子类）的指针，用于实现多态方法调用

- 实现了 sayHi() 方法：用来触发 B 的回复行为。可以看到在这个函数里，我们使用了事件类对象调用其抽象方法进行了向 B 的事件通知

- 实现了 setEvent() 方法：用来传递 B 的指针，将其赋值给事件类的指针（父类指针），方便在 sayHi() 方法中多态调用回调方法

接下来，我们需要实现 B 类，也就是驱使方：

```C++
// person B class , implement the event function
class B : CEvent {
public:
    void sayHi() {
        cout << "B: hello A" << endl;
    }
    void hiReply() {
        cout << "B: I'm fine, thanks, and you ?" << endl;
    }
};
```

B 类中，我们继承了事件类，并且实现了里面的方法。

最后，让我们看看世界的主宰–主过程里干了什么：

```C++
int main()
{
    A a;
    B b;
    a.setEvent((CEvent*)&b);
    a.sayHi();
    system("pause");
    return 0;
}
```

可以看到，主过程里我们干了这几件事：

绑定事件关系：将 b 的指针（事件类子类）传递给了 a 的私有变量 m_pEvent（事件类父类），具有了多态调用的必要条件；这里需要注意的是我们子类向父类的强制转换的写法 (CEvent*)&b，不这么写的话，编译器会报错；想要详细了解 C++ 中父类和子类的转换的可以点击这里 C++中子类和父类之间的相互转化，这篇博客质量还是可以的

触发事件：让 a 向 b 打招呼

让我们运行下这个程序，看看运行结果：

```C++
A: hello B
B: I am fine.
```

# 总结

```C++
#include <iostream>
#include <cstdlib>

using std::cout;
using std::endl;

// event class
class CEvent {
public:
	virtual void hiReply() = 0;
};

// person A class , call an event
class A {
public:
	A() : m_pEvent(NULL) {}
	void sayHi() {
		cout << "A: hello B" << endl;
		if (nullptr != m_pEvent) {
			m_pEvent->hiReply();
		}
	}
	void setEvent(CEvent *event) {
		m_pEvent = event;
	}
private:
	CEvent *m_pEvent;
};

// person B class , implement the event function
class B : CEvent {
public:
	void hiReply() {
		cout << "B: I'm fine, thanks, and you ?" << endl;
	}
};

int main()
{
	A a;
	B b;
	a.setEvent((CEvent*)&b);
	a.sayHi();
	system("pause");
	return 0;
}
```

# 参考

- [C++简单实现回调机制](https://blog.csdn.net/u012814856/article/details/73294124)