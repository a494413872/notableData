---
tags: [linux/myserver]
title: 安装mysql
created: '2020-07-29T06:16:47.434Z'
modified: '2020-07-29T08:25:46.925Z'
---

# 安装mysql

1. 从mysql官网上下载自己适合的mysql版本https://dev.mysql.com/downloads/mysql/5.6.html#downloads
获取下载链接
https://dev.mysql.com/get/Downloads/MySQL-5.6/mysql-5.6.49-linux-glibc2.12-x86_64.tar.gz

2. 通过wget 下载
```
[root@104 mysql]# wget https://dev.mysql.com/get/Downloads/MySQL-5.6/mysql-5.6.49-linux-glibc2.12-x86_64.tar.gz
```

3. 解压
tar -zxvf mysql-5.6.49-linux-glibc2.12-x86_64.tar.gz

4. 进入到mysql目录，执行添加MySQL配置的操作
```
cp support-files/my-default.cnf /etc/my.cnf
```
是否覆盖？按y 回车

5. 编辑/etc/my.cnf文件添加上默认配置
```
explicit_defaults_for_timestamp=true

# These are commonly set, remove the # and set as required.
basedir = /usr/local/mysql
datadir = /usr/local/mysql/data
port = 3306
# server_id = .....
socket = /tmp/mysql.sock
character-set-server = utf8
skip-name-resolve
log_error = /usr/local/mysql/data/error.log
pid-file = /usr/local/mysql/data/mysql.pid
```

6. 初始化数据（在mysql/bin或者mysql/scripts下有个 mysql_install_db 可执行文件初始化数据库），进入mysql/bin或者mysql/scripts目录下，执行下面命令
```
./mysql_install_db --verbose --user=root --defaults-file=/etc/my.cnf --datadir=/usr/mysql/mysql-5.6.49-linux-glibc2.12-x86_64/data --basedir=/usr/mysql/mysql-5.6.49-linux-glibc2.12-x86_64 --pid-file=//usr/mysql/mysql-5.6.49-linux-glibc2.12-x86_64/data/mysql.pid --tmpdir=/tmp
```
报错
```
FATAL ERROR: please install the following Perl modules before executing ./mysql_install_db:
Data::Dumper
```
解决方案
```
 yum -y install autoconf
```
再次报错
```
Installing MySQL system tables.../usr/mysql/mysql-5.6.49-linux-glibc2.12-x86_64/bin/mysqld: error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory
```
这是因为缺少libaio  导致的安装之
```
yum install -y libaio
```
安装成功输出如下
```
PLEASE REMEMBER TO SET A PASSWORD FOR THE MySQL root USER !
To do so, start the server, then issue the following commands:

  /usr/mysql/mysql-5.6.49-linux-glibc2.12-x86_64/bin/mysqladmin -u root password 'new-password'
  /usr/mysql/mysql-5.6.49-linux-glibc2.12-x86_64/bin/mysqladmin -u root -h 104.224.129.106 password 'new-password'

Alternatively you can run:

  /usr/mysql/mysql-5.6.49-linux-glibc2.12-x86_64/bin/mysql_secure_installation

which will also give you the option of removing the test
databases and anonymous user created by default.  This is
strongly recommended for production servers.

See the manual for more instructions.

You can start the MySQL daemon with:

  cd . ; /usr/mysql/mysql-5.6.49-linux-glibc2.12-x86_64/bin/mysqld_safe &

You can test the MySQL daemon with mysql-test-run.pl

  cd mysql-test ; perl mysql-test-run.pl

Please report any problems at http://bugs.mysql.com/

```

7. 配置开机启动
```
cp /usr/mysql/mysql-5.6.49-linux-glibc2.12-x86_64/support-files/mysql.server /etc/rc.d/init.d/mysqld

chmod 700 /etc/init.d/mysql
chkconfig --add mysqld
chkconfig --level 2345 mysqld on
```

8. 重启linux查看mysql状态
```
reboot
service mysqld status
```
发现启动失败，经过查询发现是因为没有指定用户

修改mysqld文件
```
$bindir/mysqld_safe --datadir="$datadir" --pid-file="$mysqld_pid_file_path" $other_args >/dev/null &

$bindir/mysqld_safe --datadir="$datadir" --pid-file="$mysqld_pid_file_path" $other_args >/dev/null &

```
启动成功
9. 配置 mysql_secure_installation
```
/usr/mysql/mysql-5.6.49-linux-glibc2.12-x86_64/bin/mysql_secure_installation
```
报错
```
Can't find a 'mysql' client in PATH or ./bin
Cleaning up...
Warning: Could not unlink .my.cnf.5004: 没有那个文件或目录
Warning: Could not unlink .mysql.5004: 没有那个文件或目录

```
配置环境变量
```
vim /etv/profile  #修改文件
# 添加以下内容
export MYSQL_HOME=/usr/local/mysql
export PATH=$MYSQL_HOME/bin:$PATH
# 刷新环境变量
source /etc/profile 
```
继续配置
第一次提示直接回车，会让你输入root密码，设置为Eudcsvc66
然后问你是否移除anonymous，选择移除
然后问题你是否限制root只能访问LocalHost。设置为Y
然后问你是否删除Test数据库，选择删除
然后问你是否马上生效，也选择是
```
All done!  If you've completed all of the above steps, your MySQL
installation should now be secure.

Thanks for using MySQL!

```

然后开始导入数据
```
mysql> create database smiledb;
Query OK, 1 row affected (0.00 sec)

mysql> use smiledb;
Database changed

mysql> set names utf8;
Query OK, 0 rows affected (0.00 sec)

mysql> source /usr/smiledb.sql
```    

