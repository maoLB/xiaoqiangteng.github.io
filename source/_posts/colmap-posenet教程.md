---
title: colmap-posenet教程
date: 2018-05-17 19:58:35
categories:
- 编程
- colmap-posenet
tags:
- 编程
- colmap-posenet
copyright:
---

# Colmap 教程

# Posenet 教程

## 转换colmap数据至文本文件

```bash
$ ./colmap-path/build/src/exe/model_converter --input_path ./ --output_path ./ --ouput_type TXT
```

## 根据TXT文件生成posenet的训练数据集和测试数据集

```python
#!/usr/bin/python
#coding=utf-8
'''
Created on 2017年11月26日

@author: teng
'''
import os
from numpy import*
import numpy as np
import random

def ReadLog(log_file):
    lines = []
    with open(log_file, 'r') as fin:
        lines=fin.readlines()
    return lines

if __name__ == '__main__':
    imagesFile = []
    q = []
    loc = []
    count = 0
    count_data = 0
    lines = ReadLog('images.txt')
    for line in lines[4:]:
        if count % 2 == 0:
            data = line.strip().split()
            imagesFile.append(str(data[9]))
            q.append([float(data[1]), float(data[2]), float(data[3]), float(data[4])])
            loc.append([float(data[5]), float(data[6]), float(data[7])])
            count_data += 1
        count += 1
    
    randomTest = []
    randomVal = []
    for i in range(int(count_data/10)):
        randomTest.append(random.randint(0, count_data-1))
        randomVal.append(random.randint(0, count_data-1))
     
    output = open('train.txt', 'w')
    output1 = open('test.txt', 'w')
    output2 = open('val.txt', 'w')
    for i in range(len(loc)):
        if i in randomTest:
            output1.write("images/" + str(imagesFile[i]) + " " + str(loc[i][0]) + " " + \
            str(loc[i][1]) + " " + str(loc[i][2]) + " " + \
                                str(q[i][0]) + " " + str(q[i][1]) + " " + \
                                str(q[i][2]) + " " + str(q[i][3]) + "\n")
        elif i in randomVal:
            output2.write("images/" + str(imagesFile[i]) + " " + str(loc[i][0]) + " " + \
            str(loc[i][1]) + " " + str(loc[i][2]) + " " + \
                                str(q[i][0]) + " " + str(q[i][1]) + " " + \
                                str(q[i][2]) + " " + str(q[i][3]) + "\n")
        else:
            output.write("images/" + str(imagesFile[i]) + " " + str(loc[i][0]) + " " + \
            str(loc[i][1]) + " " + str(loc[i][2]) + " " + \
                                str(q[i][0]) + " " + str(q[i][1]) + " " + \
                                str(q[i][2]) + " " + str(q[i][3]) + "\n")
    output.close()
    output1.close()
    output2.close()
```

# posenet 教程

## 生成lmdb文件

```python
caffe_root = '/home/teng/programmings/semap/caffe-posenet/'  # Change to your directory to caffe-posenet
import sys
sys.path.insert(0, caffe_root + 'python')

import numpy as np
import lmdb
import caffe
import random
import cv2

directory = '/home/teng/programmings/semap/caffe-posenet/posenet/models/611/'
dataset = 'train.txt'

poses = []
images = []

with open(directory+dataset) as f:
    for line in f:
        fname, p0,p1,p2,p3,p4,p5,p6 = line.split()
        p0 = float(p0)
        p1 = float(p1)
        p2 = float(p2)
        p3 = float(p3)
        p4 = float(p4)
        p5 = float(p5)
        p6 = float(p6)
        poses.append((p0,p1,p2,p3,p4,p5,p6))
        images.append(directory+fname)

r = list(range(len(images)))
print r
print 'r = \n'
random.shuffle(r)
print r

print 'Creating PoseNet Dataset.'
env = lmdb.open('train_lmdb', map_size=int(1e12))

count = 0

for i in r:
    print('i = {}, r = {}').format(i, len(r))
    if (count+1) % 100 == 0:
        print 'Saving image: ', count+1
    X = cv2.imread(images[i])
    print('images[i] = \n X = {}').format(images[i], X)
    X = cv2.resize(X, (455,256))    # to reproduce PoseNet results, please resize the images so that the shortest side is 256 pixels
    X = np.transpose(X,(2,0,1))
    im_dat = caffe.io.array_to_datum(np.array(X).astype(np.uint8))
    im_dat.float_data.extend(poses[i])
    str_id = '{:0>10d}'.format(count)
    with env.begin(write=True) as txn:
        txn.put(str_id, im_dat.SerializeToString())
    count = count+1

env.close()
```

### test文件

```python
caffe_root = '/home/teng/programmings/semap/caffe-posenet/'  # Change to your directory to caffe-posenet
import sys
sys.path.insert(0, caffe_root + 'python')

import numpy as np
import lmdb
import caffe
import random
import cv2

directory = '/home/teng/programmings/semap/caffe-posenet/posenet/models/611/'
dataset = 'test.txt'

poses = []
images = []

with open(directory+dataset) as f:
    for line in f:
        fname, p0,p1,p2,p3,p4,p5,p6 = line.split()
        p0 = float(p0)
        p1 = float(p1)
        p2 = float(p2)
        p3 = float(p3)
        p4 = float(p4)
        p5 = float(p5)
        p6 = float(p6)
        poses.append((p0,p1,p2,p3,p4,p5,p6))
        images.append(directory+fname)

r = list(range(len(images)))
print r
print 'r = \n'
random.shuffle(r)
print r

print 'Creating PoseNet Dataset.'
env = lmdb.open('test_lmdb', map_size=int(1e12))

count = 0

for i in r:
    print('i = {}, r = {}').format(i, len(r))
    if (count+1) % 100 == 0:
        print 'Saving image: ', count+1
    X = cv2.imread(images[i])
    print('images[i] = \n X = {}').format(images[i], X)
    X = cv2.resize(X, (455,256))    # to reproduce PoseNet results, please resize the images so that the shortest side is 256 pixels
    X = np.transpose(X,(2,0,1))
    im_dat = caffe.io.array_to_datum(np.array(X).astype(np.uint8))
    im_dat.float_data.extend(poses[i])
    str_id = '{:0>10d}'.format(count)
    with env.begin(write=True) as txn:
        txn.put(str_id, im_dat.SerializeToString())
    count = count+1

env.close()
```

## 计算均值文件

```bash
$ ./caffe-posenet-path/build/tools/compute_image_mean train_lmdb/ train.binaryproto
$ ./caffe-posenet-path/biuld/tools/compute_image_mean test_lmdb/ test.binaryproto
```

## 修改训练配置文件

### 修改 train_bayesian_posenet.prototxt

```bash
name: "GoogLeNet"
layers {
  top: "data"
  top: "label"
  name: "data"
  type: DATA
  data_param {
    source: "/home/teng/programmings/semap/caffe-posenet/posenet/models/611/lmdb/train_lmdb/" # revised
    batch_size: 64
    backend: LMDB
  }
  include {
    phase: TRAIN
  }
  transform_param {
    mirror: false
    crop_size: 224
    mean_file: "/home/teng/programmings/semap/caffe-posenet/posenet/models/611/lmdb/train.binaryproto" # revised
  }
}
layers {
  top: "data"
  top: "label"
  name: "data"
  type: DATA
  data_param {
    source: "/home/teng/programmings/semap/caffe-posenet/posenet/models/611/lmdb/test_lmdb" # revised
    batch_size: 32
    backend: LMDB
  }
  include {
    phase: TEST
  }
  transform_param {
    mirror: false
    crop_size: 224
    mean_file: "/home/teng/programmings/semap/caffe-posenet/posenet/models/611/lmdb/test.binaryproto" # revised
  }
}
layers {
    name: "slice_label"
    type: SLICE
    bottom: "label"
    top: "label_xyz"
    top: "label_wpqr"
    slice_param {
        slice_dim: 1
        slice_point: 3
    }
}
...
...

```

### 修改 solver_bayesian_posenet.prototxt

```bash
net: "/home/teng/programmings/semap/caffe-posenet/posenet/models/611/train/ train_bayesian_posenet.prototxt" #训练或者测试配置文件
test_initialization: false
test_iter: 7 #完成一次测试需要的迭代次数
test_interval: 36 #测试间隔
base_lr: 0.00001 #基础学习率
lr_policy: "step" #学习率变化规律
gamma: 0.9 #学习率变化指数
stepsize: 20000 #学习率变化频率
momentum: 0.9 #动量
display: 20 #屏幕显示间隔
max_iter: 120000 #最大迭代次数
solver_type: SGD
weight_decay: 0.005 #权重衰减
snapshot: 5000 #保存模型间隔
snapshot_prefix: "/home/teng/programmings/semap/caffe-posenet/posenet/models/611/train/models/posenet" #权重衰减
solver_mode: GPU #是否使用GPU
```

解释：

训练样本

总共:2280个

batch_szie:64

将所有样本处理完一次（称为一代，即epoch)需要：2280/64=36 次迭代才能完成

所以这里将test_interval设置为36，即处理完一次所有的训练数据后，才去进行测试。所以这个数要大于等于36.
如果想训练100代，则最大迭代次数为3600；


测试样本

同理，如果有212个测试样本，batch_size为32，那么需要7次才能完整的测试一次。 所以test_iter为7；这个数要大于等于7.

学习率

学习率变化规律我们设置为随着迭代次数的增加，慢慢变低。总共迭代120000次，我们将变化6次，所以stepsize设置为120000/6=20000，即每迭代20000次，我们就降低一次学习率。

## 训练

```bash
$ ./caffe-posenet-path/build/tools/caffe train -solver=examples/mnist/lenet_solver.prototxt 2>&1 | tee log
# 从中断点的 snapshot 继续训练
$ ./caffe-posenet-path/build/tools/caffe train -solver examples/mnist/lenet_solver.prototxt -snapshot examples/mnist/lenet_iter_5000.solverstate
# 观察各个阶段的运行时间可以使用
$ ./caffe-posenet-path/build/tools/caffe time -model examples/mnist/lenet_train_test.prototxt -iterations 10
# 使用已有模型提取特征
$ ./caffe-posenet-path/build/tools/extract_features.bin models/bvlc_reference_caffenet/bvlc_reference_caffenet.caffemodel examples/feature_extraction/train_val.prototxt fc7 examples/temp_features 10 lmdb
```

1） fc7表示提取全连接第七层特征，conv5表示提取第五个卷积层的特征， examples/temp_features表示存放结果的目录（目录不需要提前构建）

2.）10：输入的包的数量，我们test时的batchsize是50，这里输入10，表示会提取50*10=500张图片的特征

3.）imageNet网络有很多层（data conv1 conv2 conv3 conv4 conv5 fc6 fc7 fc8 prob），我们可以选取任意一层；fc7是最后一层特征，fc8输出的就是softmax的输出了，所以提取fc7层

4.）lmdb：输出的数据格式是lmdb，还可以是leveldb