---
tags: [linux/myserver]
title: 安装tomcat
created: '2020-07-29T06:01:48.439Z'
modified: '2020-07-29T09:25:26.417Z'
---

# 安装tomcat
1. 首先去官网下载tar.gz文件并上传到服务器
然后执行解压命令
tar -zxvf apache-tomcat-7.0.105.tar.gz

2. 改变环境变量位置。
进入tomcat安装目录的conf目录，编辑server.xml文件
cd /usr/local/apache-tomcat-7.0.73/
vim server.xml
找到配置8080端口的位置，在节点末尾添加URIEncoding="UTF-8"

3. 去到tomcat目录里面启动
./bin/startup.sh



