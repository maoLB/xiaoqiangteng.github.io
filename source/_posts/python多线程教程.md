---
title: python多线程教程
date: 2018-03-26 22:52:30
categories:
- 编程
- Python
- 多线程
tags:
- 编程
- Python
copyright:
---

# 什么多线程

多线程是加速程序计算的有效方式

## 添加线程 Thread

- 导入模块

```python
import threading
```

- 获取已激活的线程数

```python
threading.active_count()
```

- 查看所有线程信息

```python
threading.enumerate()
```

- 查看现在正在运行的线程

```python
threading.current_thread()
```

- 添加线程

threading.Thread()接收参数target代表这个线程要完成的任务，需自行定义

```python
def thread_job():
    print('This is a thread of %s' % threading.current_thread())

def main():
    thread = threading.Thread(target=thread_job,)   # 定义线程 
    thread.start()  # 让线程开始工作
    
if __name__ == '__main__':
    main()
```

## join 功能

```python
import threading
import time

def thread_job():
    print("T1 start\n")
    for i in range(10):
        time.sleep(0.1) # 任务间隔0.1s
    print("T1 finish\n")

added_thread = threading.Thread(target=thread_job, name='T1')
added_thread.start()
print("all done\n")
```

```python
thread_1.start() # start T1
thread_2.start() # start T2
thread_2.join() # join for T2
thread_1.join() # join for T1
print("all done\n")

"""
T1 start
T2 start
T2 finish
T1 finish
all done
"""
```

## 储存进程结果 Queue

- 导入线程,队列的标准模块

```python
import threading
import time
from queue import Queue
```

- 定义一个被多线程调用的函数

```python
def job(l,q):
    for i in range (len(l)):
        l[i] = l[i]**2
    q.put(l)   #多线程调用的函数不能用return返回值
```

完整的代码:

```python
import threading
import time

from queue import Queue

def job(l,q):
    for i in range (len(l)):
        l[i] = l[i]**2
    q.put(l)

def multithreading():
    q =Queue()
    threads = []
    data = [[1,2,3],[3,4,5],[4,4,4],[5,5,5]]
    for i in range(4):
        t = threading.Thread(target=job,args=(data[i],q))
        t.start()
        threads.append(t)
    for thread in threads:
        thread.join()
    results = []
    for _ in range(4):
        results.append(q.get())
    print(results)

if __name___=='__main__':
    multithreading()
```

## 线程锁 Lock

- 不使用 Lock 的情况

```python
import threading

def job1():
    global A
    for i in range(10):
        A+=1
        print('job1',A)

def job2():
    global A
    for i in range(10):
        A+=10
        print('job2',A)

if __name__== '__main__':
    lock=threading.Lock()
    A=0
    t1=threading.Thread(target=job1)
    t2=threading.Thread(target=job2)
    t1.start()
    t2.start()
    t1.join()
    t2.join()
```

运行结果:

```python
job1job2 11
job2 21
job2 31
job2 41
job2 51
job2 61
job2 71
job2 81
job2 91
job2 101
 1
job1 102
job1 103
job1 104
job1 105
job1 106
job1 107
job1 108
job1 109
job1 110
```

- 使用 Lock 的情况

```python
import threading

def job1():
    global A,lock
    lock.acquire()
    for i in range(10):
        A+=1
        print('job1',A)
    lock.release()

def job2():
    global A,lock
    lock.acquire()
    for i in range(10):
        A+=10
        print('job2',A)
    lock.release()

if __name__== '__main__':
    lock=threading.Lock()
    A=0
    t1=threading.Thread(target=job1)
    t2=threading.Thread(target=job2)
    t1.start()
    t2.start()
    t1.join()
    t2.join()
```

运行结果:

```python
job1 1
job1 2
job1 3
job1 4
job1 5
job1 6
job1 7
job1 8
job1 9
job1 10
job2 20
job2 30
job2 40
job2 50
job2 60
job2 70
job2 80
job2 90
job2 100
job2 110
```

# 参考

- https://morvanzhou.github.io/tutorials/python-basic/threading/