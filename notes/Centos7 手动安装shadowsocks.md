---
tags: [linux/myserver]
title: Centos7 手动安装shadowsocks
created: '2020-04-16T03:58:18.515Z'
modified: '2020-04-26T06:58:01.642Z'
---

# Centos7 手动安装shadowsocks

1. 安装Python和Pip环境
```
yum install python-setuptools && easy_install pip
```
2. 安装Shadowsocks

```
pip install shadowsocks
```
3. 在/etc下添加文件shadowsock.json
```
{
  "server": "0.0.0.0",//这里不用改，全0代表地服务器监所有可用网络。
  "server_port": 6356,//服务器端口号，1025到65535任选一。
  "password": "mima123321",//设置登录密码。
  "method": "rc4-md5"//加密方式。
}
```
4. 通过json启动shadowsock
```
ssserver -c /etc/shadowsocks.json -d start
```
5.停止shadowsock
```
ssserver -d stop
```
6.配置自动启动
在/etc/rc.d/rc.local追加启动命令

### 卸载
1. 先去/etc/rc.d/rc.local 删除自动启动脚本
2. 执行ssserver -d stop停止服务
3. 卸载pip uninstall shadowsocks
4. 删除配置文件pip uninstall shadowsocks



