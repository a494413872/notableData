---
tags: [linux/myserver]
title: 手动安装ssr
created: '2020-04-16T03:59:17.911Z'
modified: '2020-04-26T06:57:36.728Z'
---

# 手动安装ssr
1. 输入下面命令自动开始安装，有需求的话设置一下密码端口号之类的
```
wget --no-check-certificate https://freed.ga/github/shadowsocksR.sh; bash shadowsocksR.sh
```
2. 自动安装成功之后输出信息
```
祝贺！ ShadowsocksR 已经配置成功!
服务器 IP:  104.224.129.106 
服务器 端口:  25565 
连接密码:  1.2.3 
加密:  aes-256-cfb 
协议:  auth_sha1_v4 
混淆:  http_simple 
```
3. https://github.com/shadowsocksrr/shadowsocksr-csharp/releases 下载ssr
4. 运行ShadowsocksR-dotnet4.0
5. 配置对应的server开始使用
启动命令如下
python /usr/local/shadowsocks/shadowsocks/server.py -c /etc/shadowsocks.json -d start

