---
tags: [linux/myserver]
title: centos7 安装zk
created: '2020-04-16T04:00:51.863Z'
modified: '2020-04-26T06:58:05.035Z'
---

# centos7 安装zk

#### 从下面url获取下载地址
```
http://mirror.bit.edu.cn/apache/zookeeper/
```
然后下载
```
wget http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz
```
解压
```
tar zxf apache-zookeeper-3.6.0.tar.gz
```
去conf下面复制一份配置文件，并修改data路径为你想要的路径
```
cd conf/
cp zoo_sample.cfg zoo.cfg
vim zoo.cfg
```
启动zk
```
../bin/zkServer.sh start

```




#### 从下面url获取下载地址
```
http://mirror.bit.edu.cn/apache/zookeeper/
```
然后下载
```
wget http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz
```
解压
```
tar zxf apache-zookeeper-3.6.0.tar.gz
```
去conf下面复制一份配置文件，并修改data路径为你想要的路径
```
cd conf/
cp zoo_sample.cfg zoo.cfg
vim zoo.cfg
```
启动zk
```
../bin/zkServer.sh start

```





