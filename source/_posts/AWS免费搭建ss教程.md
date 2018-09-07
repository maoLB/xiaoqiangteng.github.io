---
title: AWS免费搭建ss教程
date: 2018-06-11 21:35:34
categories:
- 编程
- AWS
- SS
tags:
- 编程
- AWS
- ss
copyright:
---

# 注册免费的AWS服务器

注册地址：[ec2 免费试用全球云服务-亚马逊AWS
](https://aws.amazon.com/cn/ec2/pricing/?sc_channel=PS&sc_campaign=acquisition_CN&sc_publisher=baidu&sc_category=pc&sc_medium=ec2_b&sc_content=ec2_e&sc_detail=ec2&sc_segment=100009040&sc_matchtype=phrase&sc_country=CN&s_kwcid=AL!4422!88!5666095499!!17798814222&ef_id=WjtnMQAAAdinHin@:20180611133737:s)

填写注册信息、输入信用卡（成功后扣除1美元）、电话语音确认。

上述过程成功后，即可进入EC2创建服务器实例。如下图所示：

![](http://p55se4hrx.bkt.clouddn.com/images/programming/AWS_SS1.jpeg)

# 创建服务器实例

选择Ubuntu16.04 LTS系统：

![](http://p55se4hrx.bkt.clouddn.com/images/programming/AWS_SS2.jpeg)

选择实例类型：

![](http://p55se4hrx.bkt.clouddn.com/images/programming/AWS_SS3.jpeg)

根据自己的需求添加存储，免费用户最大30GB存储：

![](http://p55se4hrx.bkt.clouddn.com/images/programming/AWS_SS4.jpeg)

创建秘钥对：

![](http://p55se4hrx.bkt.clouddn.com/images/programming/AWS_SS5.jpeg)

编辑安全组规则：

![](http://p55se4hrx.bkt.clouddn.com/images/programming/AWS_SS6.jpeg)


# 登录实例并配置ss环境

```bash
$ sudo chmod 400 XXX.pem # 更改秘钥权限
$ ssh -i XXX.pem ubuntu@[IP] # 利用公有IP登录实例
```

安装Shadowsocks:

```bash
# 获取root权限
sudo -s
# 更新apt-get
apt-get update
# 安装python包管理工具
apt-get install python-setuptools
apt-get install python-pip
# 安装shadowsocks
pip install shadowsocks
```

创建配置文件:

```bash
mkdir /etc/shadowsocks
vim /etc/shadowsocks/ss.json
```

配置文件内容：

```bash
{
    "server":"0.0.0.0",
    "server_port":443, //ss连接服务器的端口
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"abcd1234", // 设置ss连接时的密码
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open":false,
    "workers": 1
}
```

启动Shadowsocks:

```bash
启动：sudo /usr/local/bin/ssserver -c /etc/shadowsocks/ss.json -d start

停止：sudo /usr/local/bin/ssserver -c /etc/shadowsocks/ss.json -d stop  
重启：sudo /usr/local/bin/ssserver -c /etc/shadowsocks/ss.json -d restart
```

# 配置自动切换模式

配置好 ss 情景模式后虽然可以使用 Chrome 浏览器科学上网了，但是这样的话无论你访问什么网站都会走代理，有时候访问国内的一些网站反而会很慢，这时候自动切换模式就解决了这个问题。下面介绍一下如何配置自动切换模式。

点击左侧的 自动切换，或者自己新建情景模式，类型选择第二个 自动切换模式。然后做如下配置：

![](http://p55se4hrx.bkt.clouddn.com/images/programming/AWS_SS7.jpeg)

导入在线规则列表，类型选择AutoProxy，可以选择导入gfwlist - https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt 或者自己自定义的AutoProxy文件。

保存设置并更新情景模式，若更新失败则开启全局代理后更新。

设置规则匹配则使用代理模式，否则直接连接。保存退出。

# 参考：

- https://blog.csdn.net/miracleswang/article/details/78959305
- https://blog.csdn.net/kntanchao/article/details/79191149


