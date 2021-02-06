---
attachments: [Clipboard_2020-07-13-21-17-46.png, Clipboard_2020-07-13-21-24-33.png, Clipboard_2020-07-13-21-25-31.png, Clipboard_2020-07-13-21-47-00.png, Clipboard_2020-07-13-21-48-55.png, Clipboard_2020-07-13-21-50-23.png, Clipboard_2020-07-13-21-53-10.png, Clipboard_2020-07-13-21-54-33.png, Clipboard_2020-07-13-21-55-28.png, Clipboard_2020-07-13-22-02-30.png, Clipboard_2020-07-13-22-04-19.png, Clipboard_2020-07-13-22-10-46.png, Clipboard_2020-07-13-22-11-36.png]
tags: [java/数据库]
title: windos安装mysql
created: '2020-07-13T13:15:02.312Z'
modified: '2021-01-30T09:12:34.061Z'
---

# windos安装mysql

先下载
![](@attachment/Clipboard_2020-07-13-21-17-46.png)
然后解压到D:\mysql-5.7.24-winx64

配置环境变量
![](@attachment/Clipboard_2020-07-13-21-24-33.png)

配置完环境变量在 D:\mysql-5.7.24-winx64 添加 mysql.ini 并且新建data目录
```
[mysql]

# 设置mysql客户端默认字符集
default-character-set=utf8 

[mysqld]

#设置3306端口
port = 3306 

# 设置mysql的安装目录
basedir=F:\mysql\mysql-5.7.24-winx64\mysql-5.7.24-winx64

# 设置mysql数据库的数据的存放目录
datadir=F:\mysql\mysql-5.7.24-winx64\mysql-5.7.24-winx64\data

# 允许最大连接数
max_connections=200

# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8

# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB

```
![](@attachment/Clipboard_2020-07-13-21-53-10.png)

打开cmd，不需要进入安装目录（∵之前配置过环境变量），输入下面命令，回车，没有反应
mysqld --initialize-insecure --user=mysql
>注：在初始化时如果加上 –initial-insecure，则会创建空密码的 root@localhost 账号，否则会创建带密码的 root@localhost 账号，密码直接写在日志文件中（datadir/hostname.err）

这里报错
![](@attachment/Clipboard_2020-07-13-21-47-00.png)
然后使用管理员用户直接运行cmd，删除datas文件里面的内容。 
![](@attachment/Clipboard_2020-07-13-21-48-55.png)
不再报错
![](@attachment/Clipboard_2020-07-13-21-50-23.png)
然后 安装mysql
![](@attachment/Clipboard_2020-07-13-21-55-28.png)
启动服务，输入如下命令，回车，启动成功后如下图
![](@attachment/Clipboard_2020-07-13-22-02-30.png)
命令执行完一直正在启动，回车之后才出现了启动成功。

服务启动成功之后，需要登录的时候输入命令（第一次登录没有密码，直接按回车过
mysql -u root -p
![](@attachment/Clipboard_2020-07-13-22-04-19.png)
修改密码（必须先启动mysql），执行如下命令回车，enter password也回车，密码一般设置为root，方便记忆
![](@attachment/Clipboard_2020-07-13-22-10-46.png)

退出exit 就行了，记住直接关闭cmd窗口是没有退出的，要输入exit才会退出啊
![](@attachment/Clipboard_2020-07-13-22-11-36.png)




