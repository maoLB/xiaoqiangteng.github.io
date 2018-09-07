---
title: numpy和pandas教程
date: 2018-03-26 08:43:49
categories:
- 编程
- Python
tags:
- 编程
- Python
copyright:
---

# 安装 numpy和pandas

[numpy](http://www.numpy.org/) 和 [pandas](https://pandas.pydata.org/)

``` bash
sudo pip install numpy
sudo pip install pandas
```

# Numpy教程

## Numpy 属性

- ndim：维度
- shape：行数和列数
- size：元素个数

## Numpy 的创建 array

- array：创建数组
- dtype：指定数据类型
- zeros：创建数据全为0
- ones：创建数据全为1
- empty：创建数据接近0
- arrange：按指定范围创建数据
- linspace：创建线段

## Numpy 基础运算

```python
import numpy as np
a=np.array([10,20,30,40])   # array([10, 20, 30, 40])
b=np.arange(4)              # array([0, 1, 2, 3])
```

- 减法

```python
c=a-b  # array([10, 19, 28, 37])
```

- 加法

```python
c=a+b   # array([10, 21, 32, 43])
```

- 乘法

```python
c=a*b   # array([  0,  20,  60, 120])
```

有所不同的是，在Numpy中，想要求出矩阵中各个元素的乘方需要依赖双星符号 **

- 乘法

```python
c=b**2  # array([0, 1, 4, 9])
```

- 三角函数

```python
c=10*np.sin(a)  # cos tan
# array([-5.44021111,  9.12945251, -9.88031624,  7.4511316 ])
```

- 逻辑判断

```python
print(b<3)  
# array([ True,  True,  True, False], dtype=bool)
```

上述运算均是建立在一维矩阵，即只有一行的矩阵上面的计算，如果我们想要对多行多维度的矩阵进行操作，需要对开始的脚本进行一些修改：

```python
a=np.array([[1,1],[0,1]])
b=np.arange(4).reshape((2,2))

print(a)
# array([[1, 1],
#       [0, 1]])

print(b)
# array([[0, 1],
#       [2, 3]])
```

此时构造出来的矩阵a和b便是2行2列的，其中 reshape 操作是对矩阵的形状进行重构， 其重构的形状便是括号中给出的数字。

- 矩阵乘法运算

```python
c_dot = np.dot(a,b)
# array([[2, 4],
#       [2, 3]])
c_dot_2 = a.dot(b)
# array([[2, 4],
#       [2, 3]])
```

- 特定运算
	- 求和：np.sum(A)
	- 最小值：np.min(A)
	- 最大值：np.max(A)
	- 最小值索引：np.argmin(A)
	- 最大值索引：np.argmax(A)
	- 平均值：np.mean(A)
	- 中位数：np.medium(A)
	- 累加：np.cumsum(A)
	- 累差：np.diff(A)
	- 非零数：np.nonzero(A)
	- 排序：np.sort(A)
	- 矩阵反向（转置）：np.transpose(A)或A.T
	- 截断：np.clip(A,5,9) # 小于5为5，大于9为9

## Numpy 索引

###  一维索引

```python
import numpy as np
A = np.arange(3,15)

# array([3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14])
         
print(A[3])    # 6
```

### 二维索引

```python
print(A[1][1])      # 8
```

这一脚本中的flatten是一个展开性质的函数，将多维的矩阵进行展开成1行的数列。而flat是一个迭代器，本身是一个object属性。

```python
import numpy as np
A = np.arange(3,15).reshape((3,4))
         
print(A.flatten())   
# array([3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14])

for item in A.flat:
    print(item)
    
# 3
# 4
……
# 14
```

## Numpy array 合并

- 上下合并 - np.vstack()

```python
import numpy as np
A = np.array([1,1,1])
B = np.array([2,2,2])
         
print(np.vstack((A,B)))    # vertical stack
"""
[[1,1,1]
 [2,2,2]]
"""
```

- 左右合并 - np.hstack()

```python
D = np.hstack((A,B))       # horizontal stack

print(D)
# [1,1,1,2,2,2]

print(A.shape,D.shape)
# (3,) (6,)
```

- 转置操作 - np.newaxis()

```python
print(A[np.newaxis,:])
# [[1 1 1]]

print(A[np.newaxis,:].shape)
# (1,3)

print(A[:,np.newaxis])
"""
[[1]
[1]
[1]]
"""

print(A[:,np.newaxis].shape)
# (3,1)
```

此时我们便将具有3个元素的array转换为了1行3列以及3行1列的矩阵了。

```python
import numpy as np
A = np.array([1,1,1])[:,np.newaxis]
B = np.array([2,2,2])[:,np.newaxis]
         
C = np.vstack((A,B))   # vertical stack
D = np.hstack((A,B))   # horizontal stack

print(D)
"""
[[1 2]
[1 2]
[1 2]]
"""

print(A.shape,D.shape)
# (3,1) (3,2)
```

- 合并操作需要针对多个矩阵或序列 - np.concatenate()

当你的合并操作需要针对多个矩阵或序列时，借助concatenate函数可能会让你使用起来比前述的函数更加方便：

```python
C = np.concatenate((A,B,B,A),axis=0)

print(C)
"""
array([[1],
       [1],
       [1],
       [2],
       [2],
       [2],
       [2],
       [2],
       [2],
       [1],
       [1],
       [1]])
"""

D = np.concatenate((A,B,B,A),axis=1)

print(D)
"""
array([[1, 2, 2, 1],
       [1, 2, 2, 1],
       [1, 2, 2, 1]])
"""
```

axis参数很好的控制了矩阵的纵向或是横向打印，相比较vstack和hstack函数显得更加方便。

## Numpy array 分割

```python
A = np.arange(12).reshape((3, 4))
print(A)
"""
array([[ 0,  1,  2,  3],
    [ 4,  5,  6,  7],
    [ 8,  9, 10, 11]])
"""
```

- 纵向分割

```python
print(np.split(A, 2, axis=1))
"""
[array([[0, 1],
        [4, 5],
        [8, 9]]), array([[ 2,  3],
        [ 6,  7],
        [10, 11]])]
"""
```

- 横向分割

```python
print(np.split(A, 3, axis=0))

# [array([[0, 1, 2, 3]]), array([[4, 5, 6, 7]]), array([[ 8,  9, 10, 11]])]
```

- 错误的分割 

范例的Array只有4列，只能等量对分，因此输入以上程序代码后Python就会报错。

```python
print(np.split(A, 3, axis=1))

# ValueError: array split does not result in an equal division
```

- 不等量的分割

```python
print(np.array_split(A, 3, axis=1))
"""
[array([[0, 1],
        [4, 5],
        [8, 9]]), array([[ 2],
        [ 6],
        [10]]), array([[ 3],
        [ 7],
        [11]])]
"""
```

- 其他的分割方式

在Numpy里还有np.vsplit()与横np.hsplit()方式可用。

```python
print(np.vsplit(A, 3)) #等于 print(np.split(A, 3, axis=0))

# [array([[0, 1, 2, 3]]), array([[4, 5, 6, 7]]), array([[ 8,  9, 10, 11]])]


print(np.hsplit(A, 2)) #等于 print(np.split(A, 2, axis=1))
"""
[array([[0, 1],
       [4, 5],
       [8, 9]]), array([[ 2,  3],
        [ 6,  7],
        [10, 11]])]
"""
```

## Numpy copy & deep copy

- copy() 的赋值方式没有关联性

```python
b = a.copy()    # deep copy
print(b)        # array([11, 22, 33,  3])
a[3] = 44
print(a)        # array([11, 22, 33, 44])
print(b)        # array([11, 22, 33,  3])
```

# Pandas 教程

## Numpy 和 Pandas 有什么不同

如果用 python 的列表和字典来作比较, 那么可以说 Numpy 是列表形式的，没有数值标签，而 Pandas 就是字典形式。Pandas是基于Numpy构建的，让Numpy为中心的应用变得更加简单。

- Series

```python
import pandas as pd
import numpy as np
s = pd.Series([1,3,6,np.nan,44,1])

print(s)
"""
0     1.0
1     3.0
2     6.0
3     NaN
4    44.0
5     1.0
dtype: float64
"""
```

Series的字符串表现形式为：索引在左边，值在右边。由于我们没有为数据指定索引。于是会自动创建一个0到N-1（N为长度）的整数型索引。

- DataFrame

```python
dates = pd.date_range('20160101',periods=6)
df = pd.DataFrame(np.random.randn(6,4),index=dates,columns=['a','b','c','d'])

print(df)
"""
                   a         b         c         d
2016-01-01 -0.253065 -2.071051 -0.640515  0.613663
2016-01-02 -1.147178  1.532470  0.989255 -0.499761
2016-01-03  1.221656 -2.390171  1.862914  0.778070
2016-01-04  1.473877 -0.046419  0.610046  0.204672
2016-01-05 -1.584752 -0.700592  1.487264 -1.778293
2016-01-06  0.633675 -1.414157 -0.277066 -0.442545
"""
```

DataFrame是一个表格型的数据结构，它包含有一组有序的列，每列可以是不同的值类型（数值，字符串，布尔值等）。DataFrame既有行索引也有列索引， 它可以被看做由Series组成的大字典。

- DataFrame 的一些简单运用

```python
print(df['b'])

"""
2016-01-01   -2.071051
2016-01-02    1.532470
2016-01-03   -2.390171
2016-01-04   -0.046419
2016-01-05   -0.700592
2016-01-06   -1.414157
Freq: D, Name: b, dtype: float64
"""
```

我们在创建一组没有给定行标签和列标签的数据 df1:

```python
df1 = pd.DataFrame(np.arange(12).reshape((3,4)))
print(df1)

"""
   0  1   2   3
0  0  1   2   3
1  4  5   6   7
2  8  9  10  11
"""
```

默认的从0开始 index. 还有一种生成 df 的方法, 如下 df2:

```python
df2 = pd.DataFrame({'A' : 1.,
                    'B' : pd.Timestamp('20130102'),
                    'C' : pd.Series(1,index=list(range(4)),dtype='float32'),
                    'D' : np.array([3] * 4,dtype='int32'),
                    'E' : pd.Categorical(["test","train","test","train"]),
                    'F' : 'foo'})
                    
print(df2)

"""
     A          B    C  D      E    F
0  1.0 2013-01-02  1.0  3   test  foo
1  1.0 2013-01-02  1.0  3  train  foo
2  1.0 2013-01-02  1.0  3   test  foo
3  1.0 2013-01-02  1.0  3  train  foo
"""
```

这种方法能对每一列的数据进行特殊对待. 如果想要查看数据中的类型, 我们可以用 dtype 这个属性:

```python
print(df2.dtypes)

"""
df2.dtypes
A           float64
B    datetime64[ns]
C           float32
D             int32
E          category
F            object
dtype: object
"""
```

如果想看对列的序号:

```python
print(df2.index)

# Int64Index([0, 1, 2, 3], dtype='int64')
```

同样, 每种数据的名称也能看到:

```python
print(df2.columns)

# Index(['A', 'B', 'C', 'D', 'E', 'F'], dtype='object')
```

果只想看所有df2的值:

```python
print(df2.values)

"""
array([[1.0, Timestamp('2013-01-02 00:00:00'), 1.0, 3, 'test', 'foo'],
       [1.0, Timestamp('2013-01-02 00:00:00'), 1.0, 3, 'train', 'foo'],
       [1.0, Timestamp('2013-01-02 00:00:00'), 1.0, 3, 'test', 'foo'],
       [1.0, Timestamp('2013-01-02 00:00:00'), 1.0, 3, 'train', 'foo']], dtype=object)
"""
```

想知道数据的总结, 可以用 describe():

```python
df2.describe()

"""
         A    C    D
count  4.0  4.0  4.0
mean   1.0  1.0  3.0
std    0.0  0.0  0.0
min    1.0  1.0  3.0
25%    1.0  1.0  3.0
50%    1.0  1.0  3.0
75%    1.0  1.0  3.0
max    1.0  1.0  3.0
"""
```

如果想翻转数据, transpose:

```python
print(df2.T)

"""                   
0                    1                    2  \
A                    1                    1                    1   
B  2013-01-02 00:00:00  2013-01-02 00:00:00  2013-01-02 00:00:00   
C                    1                    1                    1   
D                    3                    3                    3   
E                 test                train                 test   
F                  foo                  foo                  foo   

                     3  
A                    1  
B  2013-01-02 00:00:00  
C                    1  
D                    3  
E                train  
F                  foo  

"""
```

如果想对数据的 index 进行排序并输出:

```python
print(df2.sort_index(axis=1, ascending=False))

"""
     F      E  D    C          B    A
0  foo   test  3  1.0 2013-01-02  1.0
1  foo  train  3  1.0 2013-01-02  1.0
2  foo   test  3  1.0 2013-01-02  1.0
3  foo  train  3  1.0 2013-01-02  1.0
"""

如果是对数据 值 排序输出:

```python
print(df2.sort_values(by='B'))

"""
     A          B    C  D      E    F
0  1.0 2013-01-02  1.0  3   test  foo
1  1.0 2013-01-02  1.0  3  train  foo
2  1.0 2013-01-02  1.0  3   test  foo
3  1.0 2013-01-02  1.0  3  train  foo
"""
```

参考：
- [Pandas 基本介绍](https://morvanzhou.github.io/tutorials/data-manipulation/np-pd/3-1-pd-intro/)



