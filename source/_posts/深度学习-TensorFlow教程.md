---
title: 深度学习-TensorFlow教程
date: 2018-03-30 20:11:54
categories:
- 编程
- 深度学习
- TF
tags:
- 编程
- 深度学习
- TF
copyright:
---

# TensorFlow简介

TensorFlow是Google开发的一款神经网络的Python外部的结构包, 也是一个采用数据流图来进行数值计算的开源软件库.TensorFlow 让我们可以先绘制计算结构图, 也可以称是一系列可人机交互的计算操作, 然后把编辑好的Python文件 转换成 更高效的C++, 并在后端进行计算.

# TensorFlow安装

## Docker安装

### Dock安装

Docker安装请参考[实验室GPU服务器部署教程](http://mapstec.com/2018/03/29/%E5%AE%9E%E9%AA%8C%E5%AE%A4GPU%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%83%A8%E7%BD%B2%E6%95%99%E7%A8%8B/)

Docker 需要用户具有 sudo 权限，为了避免每次命令都输入sudo，可以把用户加入 Docker 用户组，参考：[docker docs](https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user)

```bash
sudo usermod -aG docker $USER
```

### 安装NVIDIA-Docker

安装完成docker并检查安装正确（能跑出来hello-world）后，如果需要docker容器中有gpu支持，需要再安装NVIDIA-Docker，同样找到并打开该项目的主页：

```bash
NVIDIA/nvidia-docker: Build and run Docker containers leveraging NVIDIA GPUs
https://github.com/NVIDIA/nvidia-docker
```

可以看到在Quick start小节，根据系统版本执行命令：

Ubuntu 14.04/16.04/18.04, Debian Jessie/Stretch：

```bash
# If you have nvidia-docker 1.0 installed: we need to remove it and all existing GPU containers
docker volume ls -q -f driver=nvidia-docker | xargs -r -I{} -n1 docker ps -q -a -f volume={} | xargs -r docker rm -f
sudo apt-get purge -y nvidia-docker

# Add the package repositories
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
  sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update

# Install nvidia-docker2 and reload the Docker daemon configuration
sudo apt-get install -y nvidia-docker2
sudo pkill -SIGHUP dockerd

# Test nvidia-smi with the latest official CUDA image
docker run --runtime=nvidia --rm nvidia/cuda nvidia-smi
```

CentOS 7 (docker-ce), RHEL 7.4/7.5 (docker-ce), Amazon Linux 1/2:

If you are not using the official docker-ce package on CentOS/RHEL, use the next section.

```bash
# If you have nvidia-docker 1.0 installed: we need to remove it and all existing GPU containers
docker volume ls -q -f driver=nvidia-docker | xargs -r -I{} -n1 docker ps -q -a -f volume={} | xargs -r docker rm -f
sudo yum remove nvidia-docker

# Add the package repositories
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.repo | \
  sudo tee /etc/yum.repos.d/nvidia-docker.repo

# Install nvidia-docker2 and reload the Docker daemon configuration
sudo yum install -y nvidia-docker2
sudo pkill -SIGHUP dockerd

# Test nvidia-smi with the latest official CUDA image
docker run --runtime=nvidia --rm nvidia/cuda nvidia-smi
```

If yum reports a conflict on /etc/docker/daemon.json with the docker package, you need to use the next section instead.

CentOS 7 (docker), RHEL 7.4/7.5 (docker):

```bash
# If you have nvidia-docker 1.0 installed: we need to remove it and all existing GPU containers
docker volume ls -q -f driver=nvidia-docker | xargs -r -I{} -n1 docker ps -q -a -f volume={} | xargs -r docker rm -f
sudo yum remove nvidia-docker

# Add the package repositories
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-container-runtime/$distribution/nvidia-container-runtime.repo | \
  sudo tee /etc/yum.repos.d/nvidia-container-runtime.repo

# Install the nvidia runtime hook
sudo yum install -y nvidia-container-runtime-hook
sudo mkdir -p /usr/libexec/oci/hooks.d
echo -e '#!/bin/sh\nPATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin" exec nvidia-container-runtime-hook "$@"' | \
  sudo tee /usr/libexec/oci/hooks.d/nvidia
sudo chmod +x /usr/libexec/oci/hooks.d/nvidia

# Test nvidia-smi with the latest official CUDA image
# You can't use `--runtime=nvidia` with this setup.
docker run --rm nvidia/cuda nvidia-smi
```

上面最后一条命令是检查是否安装成功，安装成功，则会显示关于GPU的信息。

然后在执行下面这句，默认用nvdia-docker替代docker命令：

```bash
echo 'alias docker=nvidia-docker' >> ~/.bashrc
bash
```

### 下载使用TensorFlow镜像

打开dockerhub关于tensorflow的页面：

```bash
tensorflow/tensorflow – Docker Hub
https://hub.docker.com/r/tensorflow/tensorflow/
```

根据需要的版本下载tensorflow镜像并开启tensorflow容器：

CPU版本

```bash
docker run -it -p 8888:8888 tensorflow/tensorflow
```

GPU版本

```bash
nvidia-docker run -it -p 8888:8888 tensorflow/tensorflow:latest-gpu
```

如何使用,执行以上命令的结果类似如下：

```bash
$ nvidia-docker run -it -p 8888:8888 tensorflow/tensorflow:latest-gpu
[I 02:51:21.230 NotebookApp] Writing notebook server cookie secret to /root/.local/share/jupyter/runtime/notebook_cookie_secret
[W 02:51:21.242 NotebookApp] WARNING: The notebook server is listening on all IP addresses and not using encryption. This is not recommended.
[I 02:51:21.249 NotebookApp] Serving notebooks from local directory: /notebooks
[I 02:51:21.249 NotebookApp] 0 active kernels 
[I 02:51:21.249 NotebookApp] The Jupyter Notebook is running at: http://[all ip addresses on your system]:8888/?token=8f90cc7b9ad6ccc4f36f53f347c7a314220bbcb82dd416ea
[I 02:51:21.249 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[C 02:51:21.249 NotebookApp] 
    
    Copy/paste this URL into your browser when you connect for the first time,
    to login with a token:
        http://localhost:8888/?token=8f90cc7b9ad6ccc4f36f53f347c7a314220bbcb82dd416ea
[I 02:51:31.832 NotebookApp] 302 GET / (172.17.0.1) 0.74ms
[I 02:51:31.943 NotebookApp] 302 GET /tree? (172.17.0.1) 1.44ms
```

其中看到有个网址：

```bash
http://localhost:8888/?token=8f90cc7b9ad6ccc4f36f53f347c7a314220bbcb82dd416ea
```

每个人的网址在token=后面的内容是不一样的，现在我们打开浏览器，输入网址：

```bash
http://localhost:8888/
```

输入刚刚token后面的值后,点击第一个1_hello_tensorflow.ipynb，然后可以选择执行所有代码.

常用命令：

```bash
docker image pull library/hello-world
```

上面代码中，docker image pull是抓取 image 文件的命令。library/hello-world是 image 文件在仓库里面的位置，其中library是 image 文件所在的组，hello-world是 image 文件的名字。

```bash
docker image ls
```

抓取成功以后，就可以在本机看到这个 image 文件了。运行这个 image 文件:

```bash
docker container run hello-world
```

docker container run命令会从 image 文件，生成一个正在运行的容器实例。

注意，docker container run命令具有自动抓取 image 文件的功能。如果发现本地没有指定的 image 文件，就会从仓库自动抓取。因此，前面的docker image pull命令并不是必需的步骤。

有些容器不会自动终止，因为提供的是服务。比如，安装运行 Ubuntu 的 image，就可以在命令行体验 Ubuntu 系统。

```bash
docker container run -it ubuntu bash
```

对于那些不会自动终止的容器，必须使用docker container kill 命令手动终止。

```bash
docker container kill [containID]
```

image 文件生成的容器实例，本身也是一个文件，称为容器文件。也就是说，一旦容器生成，就会同时存在两个文件： image 文件和容器文件。而且关闭容器并不会删除容器文件，只是容器停止运行而已。

```bash
# 列出本机正在运行的容器
$ docker container ls

# 列出本机所有容器，包括终止运行的容器
$ docker container ls --all
```

上面命令的输出结果之中，包括容器的 ID。很多地方都需要提供这个 ID，比如上一节终止容器运行的docker container kill命令。

终止运行的容器文件，依然会占据硬盘空间，可以使用docker container rm命令删除。

```bash
docker container rm [containerID]
```

运行上面的命令之后，再使用docker container ls --all命令，就会发现被删除的容器文件已经消失了。

创建tensorflow docker容器：

```bash
docker container run --name [name] -it -p 8888:8888 tensorflow/tensorflow:latest-gpu /bin/bash
# [name]-- 容器的名字
# -it -- 保留命令行运行
# -p 8888:8888 —— 将本地的8888端口和http://localhost:8888/映射
# tensorflow/tensorflow:latest-gpu ：默认是tensorflow/tensorflow:latest,指定使用的镜像
```

启动docker：

```bash
docker start [name]
```

进入docker:

```bash
docker attach [name]
```

重命名docker:

```bash
docker rename  old_name new_name
```
### 如何进入正在执行的 docker container

#### docker attach

这个是官方提供的一种方法。

测试，首先启动一个container:

```bash
$ docker run -i -t ubuntu bash
root@4556f5ad6067:/#
```

不要退出，打开另一个终端：

```bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
4556f5ad6067        ubuntu:14.04        "bash"              45 seconds ago      Up 43 seconds                           jolly_ardinghelli

$ docker attach 4556f5ad6067
root@4556f5ad6067:/#
```

这样就连接进去了。这时候如果我们输入一些命令，就能看到在两个终端都有显示和输出。这种方式有比较大的局限性，如果知道了entrypoint或者有程序正在执行，通过docker attach进入之后是不能执行操作的，一个终端退出之后整个container就终止了。不推荐使用这种方式。

#### lxc-attach

如果使用这种方式，首先要保证docker是以lxc方式启动的，具体可以这样做：

修改/etc/default/docker增加DOCKER_OPTS="-e lxc"

重启docker服务sudo service docker restart

启动container的方式和之前一样：

```bash
$ docker run -i -t ubuntu bash
root@e7f01f0ff598:/#
```

进入container可以这样：

```bash

$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
e7f01f0ff598        ubuntu:14.04        "bash"              17 seconds ago      Up 15 seconds                           grave_jones

$ ps aux | grep e7f01f0ff598
root     23691  0.0  0.0  43140  1876 pts/9    Ss   21:47   0:00 lxc-start -n e7f01f0ff598c80d70a996135c98fbaeddc6daa61436bbbfa735233e8b6f8ebe -f /var/lib/docker/containers/e7f01f0ff598c80d70a996135c98fbaeddc6daa61436bbbfa735233e8b6f8ebe/config.lxc -- /.dockerinit -g 172.17.42.1 -i 172.17.0.3/16 -mtu 1500 -- bash
ma6174   23756  0.0  0.0  13428   928 pts/12   S+   21:47   0:00 grep --color=auto e7f01f0ff598

$ sudo lxc-attach -n e7f01f0ff598c80d70a996135c98fbaeddc6daa61436bbbfa735233e8b6f8ebe
root@e7f01f0ff598:/#
```

这种方式还是很方便的。前提是需要重启docker服务以lxc的方式执行，进入container之后会有一个终端可以执行命令，不影响正在执行的程序。

#### nsenter

如果docker不是以lxc方式启动的，这时候还想进入一个正在执行的container的话，可以考虑使用nsenter

这个程序的安装方式很独特，使用docker进行安装：

```bash
$ docker run --rm -v /usr/local/bin:/target jpetazzo/nsenter
```

使用方法也很简单，首先你要进入的container的PID：

```bash
$ PID=$(docker inspect --format {{.State.Pid}} <container_name_or_ID>)
```

然后就可以用这个命令进入container了：

```bash
$ nsenter --target $PID --mount --uts --ipc --net --pid
```

为了使用方便可以写一个脚本自动完成：

```bash
$ cat /bin/docker_enter
#!/bin/bash
sudo nsenter --target `docker inspect --format {{.State.Pid}} $1` --mount --uts --ipc --net --pid bash
```

这样每次要进入某个container只需要执行docker_enter <container_name_or_ID>就可以了。

#### ssh

这个原理也很简单，在container里面启动ssh服务，然后通过ssh的方式去登陆到container里面，不推荐这种方式，主要是配置ssh登陆比较繁琐，开启ssh服务也会耗费资源，完全没有必要。

### TensorFlow安装方式一

1. 下载镜像

	```bash
	docker pull tensorflow/tensorflow
	```

2. 创建Tensorflow容器

	```bash
	docker run --name my-tensorflow -it -p 8888:8888 -v ~/tensorflow:/test/data tensorflow/tensorflow
	# --name：创建的容器名，即my-tensorflow
	# -it：保留命令行运行
	# p 8888:8888：将本地的8888端口和http://localhost:8888/映射
	# -v ~/tensorflow:/test/data:将本地的~/tensorflow挂载到容器内的/# test/data下
	# tensorflow/tensorflow ：默认是tensorflow/tensorflow:latest,指定使用的镜像
	```

3. 拷贝带token的URL在浏览器打开

	```bash
	http://[all ip addresses on your system]:8888/?token=649d7cab1734e01db75b6c2b476ea87aa0b24dde56662a27
	```

4. 显示Jupyter Notebook，Jupyter Notebook（此前被称为 IPython notebook）是一个交互式笔记本。示例中已经显示了Tensorflow的入门教程，点开一个可以看见。

5. 关闭容器

	```bash
	docker stop my-tensortflow
	```

6. 再次打开

	```bash
	docker start my-tensortflow
	```

### TensorFlow安装方式二

1. 下载镜像

	```bash
	docker pull tensorflow/tensorflow
	```

2. 创建Tensorflow容器

	```bash
	docker run -it --name bash_tensorflow tensorflow/tensorflow /bin/bash
	# 这样我们就创建了名为bash_tensorflow的容器
	```
	
3. start命令启动容器

	```bash
	docker start bash_tensorflow
	```

4. 再连接上容器

	```bash
	docker attach bash_tensorflow
	# 可以看到我们用终端连接上了容器，和操作Linux一样了。
	```
	
## Pip安装

### Linux 和 MacOS

```bash
# Ubuntu/Linux 64-位 系统的执行代码:
$ sudo apt-get install python-pip python-dev

# Mac OS X 系统的执行代码:
$ sudo easy_install --upgrade pip
$ sudo easy_install --upgrade six
```

1. CPU 版

	```bash
	# python 2+ 的用户:
	$ pip install tensorflow
	
	# python 3+ 的用户:
	$ pip3 install tensorflow
	```

2. GPU 版

	```bash
	$ sudo apt-get install libcupti-dev
	$ sudo apt-get install python-pip python-dev   # for Python 2.7
	$ sudo apt-get install python3-pip python3-dev # for Python 3.n
	$ pip install tensorflow      # Python 2.7; CPU support (no GPU support)
	$ pip3 install tensorflow     # Python 3.n; CPU support (no GPU support)
	$ pip install tensorflow-gpu  # Python 2.7;  GPU support
	$ pip3 install tensorflow-gpu # Python 3.n; GPU support
	```
	
3. 测试

	```python
	import tensorflow
	```
	
# TensorFlow 教程

## Session 会话控制

参考：

- https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/2-3-session/

Session 是 Tensorflow 为了控制,和输出文件的执行的语句. 运行 session.run() 可以获得你要得知的运算结果, 或者是你所要运算的部分.

例子讲解：建立两个 matrix ,输出两个 matrix 矩阵相乘的结果。

```python
import tensorflow as tf

# create two matrixes

matrix1 = tf.constant([[3,3]])
matrix2 = tf.constant([[2],
                       [2]])
product = tf.matmul(matrix1,matrix2)
```

因为 product 不是直接计算的步骤, 所以我们会要使用 Session 来激活 product 并得到计算结果. 有两种形式使用会话控制 Session 。

```python
# method 1
sess = tf.Session()
result = sess.run(product)
print(result)
sess.close()
# [[12]]

# method 2
with tf.Session() as sess:
    result2 = sess.run(product)
    print(result2)
# [[12]]
```

## Variable 变量

参考：

- https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/2-4-variable/

在 Tensorflow 中，定义了某字符串是变量，它才是变量。定义语法： state = tf.Variable()

```python
import tensorflow as tf

state = tf.Variable(0, name='counter')

# 定义常量 one
one = tf.constant(1)

# 定义加法步骤 (注: 此步并没有直接计算)
new_value = tf.add(state, one)

# 将 State 更新成 new_value
update = tf.assign(state, new_value)
```

如果你在 Tensorflow 中设定了变量，那么初始化变量是最重要的！！所以定义了变量以后, 一定要定义 init = tf.initialize_all_variables() .

到这里变量还是没有被激活，需要再在 sess 里, sess.run(init) , 激活 init 这一步.

```python
# 如果定义 Variable, 就一定要 initialize
# init = tf.initialize_all_variables() # tf 马上就要废弃这种写法
init = tf.global_variables_initializer()  # 替换成这样就好
 
# 使用 Session
with tf.Session() as sess:
    sess.run(init)
    for _ in range(3):
        sess.run(update)
        print(sess.run(state))
```

注意：直接 print(state) 不起作用！！

一定要把 sess 的指针指向 state 再进行 print 才能得到想要的结果！

## Placeholder 传入值

参考：

- https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/2-5-placeholde/

placeholder 是 Tensorflow 中的占位符，暂时储存变量.

Tensorflow 如果想要从外部传入data, 那就需要用到 tf.placeholder(), 然后以这种形式传输数据 sess.run(***, feed_dict={input: **}).

示例：

```python
import tensorflow as tf

#在 Tensorflow 中需要定义 placeholder 的 type ，一般为 float32 形式
input1 = tf.placeholder(tf.float32)
input2 = tf.placeholder(tf.float32)

# mul = multiply 是将input1和input2 做乘法运算，并输出为 output 
ouput = tf.multiply(input1, input2)
```

接下来, 传值的工作交给了 sess.run() , 需要传入的值放在了feed_dict={} 并一一对应每一个 input. placeholder 与 feed_dict={} 是绑定在一起出现的。

```python
with tf.Session() as sess:
    print(sess.run(ouput, feed_dict={input1: [7.], input2: [2.]}))
# [ 14.]
```

## 建造神经网络

参考：

- https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/3-2-create-NN/

### add_layer 功能

```python
import tensorflow as tf
import numpy as np
def add_layer(inputs, in_size, out_size, activation_function=None):
    Weights = tf.Variable(tf.random_normal([in_size, out_size]))
    biases = tf.Variable(tf.zeros([1, out_size]) + 0.1)
    Wx_plus_b = tf.matmul(inputs, Weights) + biases
    if activation_function is None:
        outputs = Wx_plus_b
    else:
        outputs = activation_function(Wx_plus_b)
    return outputs
```

### 导入数据

构建所需的数据。 这里的x_data和y_data并不是严格的一元二次函数的关系，因为我们多加了一个noise,这样看起来会更像真实情况。

```python
x_data = np.linspace(-1,1,300, dtype=np.float32)[:, np.newaxis]
noise = np.random.normal(0, 0.05, x_data.shape).astype(np.float32)
y_data = np.square(x_data) - 0.5 + noise
```

利用占位符定义我们所需的神经网络的输入。 tf.placeholder()就是代表占位符，这里的None代表无论输入有多少都可以，因为输入只有一个特征，所以这里是1。

```python
xs = tf.placeholder(tf.float32, [None, 1])
ys = tf.placeholder(tf.float32, [None, 1])
```

接下来，我们就可以开始定义神经层了。 通常神经层都包括输入层、隐藏层和输出层。这里的输入层只有一个属性， 所以我们就只有一个输入；隐藏层我们可以自己假设，这里我们假设隐藏层有10个神经元； 输出层和输入层的结构是一样的，所以我们的输出层也是只有一层。 所以，我们构建的是——输入层1个、隐藏层10个、输出层1个的神经网络。

### 搭建网络

利用之前的add_layer()函数，这里使用 Tensorflow 自带的激励函数tf.nn.relu。

```python
l1 = add_layer(xs, 1, 10, activation_function=tf.nn.relu)
```

接着，定义输出层。此时的输入就是隐藏层的输出——l1，输入有10层（隐藏层的输出层），输出有1层。

```python
prediction = add_layer(l1, 10, 1, activation_function=None)
```

计算预测值prediction和真实值的误差，对二者差的平方求和再取平均。

```python
loss = tf.reduce_mean(tf.reduce_sum(tf.square(ys - prediction),
                     reduction_indices=[1]))
```

接下来，是很关键的一步，如何让机器学习提升它的准确率。tf.train.GradientDescentOptimizer()中的值通常都小于1，这里取的是0.1，代表以0.1的效率来最小化误差loss。

```python
train_step = tf.train.GradientDescentOptimizer(0.1).minimize(loss)
```

使用变量时，都要对它进行初始化，这是必不可少的。

```python
# init = tf.initialize_all_variables() # tf 马上就要废弃这种写法
init = tf.global_variables_initializer()  # 替换成这样就好
```

定义Session，并用 Session 来执行 init 初始化步骤。 （注意：在tensorflow中，只有session.run()才会执行我们定义的运算。）

```python
sess = tf.Session()
sess.run(init)
```

### 训练

机器学习的内容是train_step, 用 Session 来 run 每一次 training 的数据，逐步提升神经网络的预测准确性。 (注意：当运算要用到placeholder时，就需要feed_dict这个字典来指定输入。)

```python
for i in range(1000):
    # training
    sess.run(train_step, feed_dict={xs: x_data, ys: y_data})
```

每50步我们输出一下机器学习的误差。

```python
    if i % 50 == 0:
        # to see the step improvement
        print(sess.run(loss, feed_dict={xs: x_data, ys: y_data}))
```

```python
0.021204619
0.009980676
0.007174721
0.006633012
0.00622975
0.005894037
0.005621146
0.0053801737
0.00519997
0.005050111
0.004922069
0.0048095705
0.0047140927
0.0046234317
0.0045334958
0.0044504963
0.004378309
0.0043256846
0.0042802156
0.0042369063
```

