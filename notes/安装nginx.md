---
tags: [linux/myserver]
title: 安装nginx
created: '2020-04-16T09:01:16.237Z'
modified: '2020-04-26T06:57:50.380Z'
---

# 安装nginx

创建目录
```
mkdir nginx
```
下载
```
wget http://nginx.org/download/nginx-1.6.2.tar.gz
```
解压
```
tar -zxvf nginx-1.6.2.tar.giz
```
安装编译环境
```
yum install pcre
yum install pcre-devel
yum install zlib
yum install zlib-devel
yum install gcc-c++
```
修改配置文件指定编译文件以及配置文件的位置
```
 cd /usr/nginx/nginx-1.6.2   && ./configure --prefix=/usr/nginx
```
编译安装
```
make && make install
```
启动nginx
cd 到/usr/nginx目录下。执行ls，可以看到四个目录
conf----配置文件  html----网页文件  logs-----日志文件  sbin------主要二进制程序
```
/usr/ngnix/sbin/nginx    (无参数) 启动    （-s  stop）关闭    （-s reload）重启
```



