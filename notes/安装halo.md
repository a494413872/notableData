---
tags: [linux/myserver]
title: 安装halo
created: '2020-04-23T02:19:22.141Z'
modified: '2020-05-11T05:47:10.641Z'
---

# 安装halo
1. 请确保服务器的软件包已经是最新的。
```
sudo yum update -y
```
2. 安装java运行环境
```
sudo yum install java-1.8.0-openjdk -y
java -version
```
3.下载配置文件
考虑到部分用户的需要，可能需要自定义比如端口等设置项，我们提供了公共的配置文件，并且该配置文件是完全独立于安装包的。当然，你也可以使用安装包内的默认配置文件，但是安装包内的配置文件是不可修改的。请注意：配置文件的路径为 ~/.halo/application.yaml
```
# 下载配置文件到 ~/.halo 目录
curl -o ~/.halo/application.yaml --create-dirs https://dl.halo.run/config/application-template.yaml
```
4. 修改配置文件
```
# 使用 Vim 工具修改配置文件
vim ~/.halo/application.yaml
```
> 如果需要自定义端口，修改 server 节点下的 port 即可。
默认使用的是 H2 Database 数据库，这是一种嵌入式的数据库，使用起来非常方便。需要注意的是，默认的用户名和密码为 admin 和 123456，这个是自定义的，最好将其修改，并妥善保存。
如果需要使用 MySQL 数据库，需要将 H2 Database 的所有相关配置都注释掉，并取消 MySQL 的相关配置。另外，MySQL 的默认数据库名为 halodb，请自行配置 MySQL 并创建数据库，以及修改配置文件中的用户名和密码。
h2 节点为 H2 Database 的控制台配置，默认是关闭的，如需使用请将 h2.console.settings.web-allow-others 和 h2.console.enabled 设置为 true。控制台地址即为 域名/h2-console。注意：非紧急情况，不建议开启该配置。
server.compression.enabled 为 Gzip 功能配置，如有需要请设置为 true，需要注意的是，如果你使用 Nginx 或者 Caddy 进行反向代理的话，默认是有开启 Gzip 的，所以这里可以保持默认。
halo.admin-path 为后台管理的根路径，默认为 admin，如果你害怕别人猜出来默认的 admin（就算猜出来，对方什么都做不了），请自行设置。仅支持一级，且前后不带 /。
halo.cache 为系统缓存形式的配置，可选 memory 和 level，默认为 memory，将数据缓存到内存，使用该方式的话，重启应用会导致缓存清空。如果选择 level，则会将数据缓存到磁盘，重启不会清空缓存。如不知道如何选择，建议默认。
5. 运行hallo
>Halo 的整个应用程序只有一个 Jar 包，且不包含用户的任何配置，它放在任何目录都是可行的。需要注意的是，Halo 的整个额外文件全部存放在 ~/.halo 目录下，包括 application.yaml（用户配置文件），template/themes（主题目录），upload（附件上传目录），halo.db.mv（数据库文件）。一定要保证 ~/.halo 的存在，你博客的所有资料可都存在里面。所以你完全不需要担心安装包的安危，它仅仅是个服务而已。

```
# 下载最新的 Halo 安装包，{{version}} 为版本号，不带 v，更多下载地址请访问 https://halo.run/archives/download.html
wget https://dl.halo.run/release/halo-{{version}}.jar -O halo-latest.jar

# 启动测试
java -jar halo-latest.jar

如看到以下日志输出，则代表启动成功.

run.halo.app.listener.StartedListener    : Halo started at         http://127.0.0.1:8090
run.halo.app.listener.StartedListener    : Halo admin started at   http://127.0.0.1:8090/admin
run.halo.app.listener.StartedListener    : Halo has started successfully!
```
6.上面只是测试是否成功，如果想持续运行还需要下面配置
上面我们已经完成了 Halo 的整个配置和安装过程，接下来我们对其进行更完善的配置，比如：需要开机自启？，更简单的启动方式？
实现以上功能我们只需要新增一个配置文件即可，也就是使用 Systemd 来完成这些工作
```
# 下载 Halo 官方的 halo.service 模板
sudo curl -o /usr/lib/systemd/system/halo.service --create-dirs https://dl.halo.run/config/halo.service
```
7. 下载完成之后，我们还需要对其进行修改。
```
# 修改 halo.service
sudo vim /usr/lib/systemd/system/halo.service
```
8. 进去之后你会看到如下配置
```
[Unit]
Description=Halo Service
Documentation=https://halo.run
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/usr/bin/java -server -Xms256m -Xmx256m -jar YOUR_JAR_PATH
ExecStop=/bin/kill -s QUIT $MAINPID
Restart=always
StandOutput=syslog

StandError=inherit

[Install]
WantedBy=multi-user.target
```
>-Xms256m：为 JVM 启动时分配的内存，请按照服务器的内存做适当调整，512 M 内存的服务器推荐设置为 128，1G 内存的服务器推荐设置为 256，默认为 256。
-Xmx256m：为 JVM 运行过程中分配的最大内存，配置同上。
YOUR_JAR_PATH：Halo 安装包的绝对路径，例如 /www/wwwroot/halo-latest.jar。
9. 启动服务命令
```
# 修改 service 文件之后需要刷新 Systemd
sudo systemctl daemon-reload

# 使 Halo 开机自启
sudo systemctl enable halo

# 启动 Halo
sudo service halo start

# 重启 Halo
sudo service halo restart

# 停止 Halo
sudo service halo stop

# 查看 Halo 的运行状态
sudo service halo status
```
10. 由于之前nginx已经安装好了，所以直接开始配置
```
vi /usr/nginx/conf/nginx.conf
#修改为
server {
    listen 80;

    server_name goodmemory.tk www.goodmemory.tk;

    location / {
        proxy_set_header HOST $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_pass http://127.0.0.1:8090/;
    }
}
#测试配置文件
{{目录}}nginx -t
# 带配置文件启动
./nginx -c /usr/nignx/conf/nginx.conf
```
11. 手动安装SSL证书
goodadmin_luo_9899@sian.cn


