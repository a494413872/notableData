---
tags: [java/中间件]
title: WebLogic
created: '2021-01-12T08:19:38.739Z'
modified: '2021-01-13T10:00:41.477Z'
---

# WebLogic

美国BEA公司的，后来被oracle收购了。
自身就是支持集群，不需要代码支持。可以直接就扩展，也能保证高可用。


1. 选择是否创建域，或者选择更新域
2. 选择从模板创建域
3. 设置管理员账户
4. 设置域模式（开发，生产）和jdk路径
5. 一些高级配置，是否开启服务器管理器，节点管理器等等。
6. 管理服务器，设置服务器名，监听端口等
7. 完成创建
8. 启动服务器，访问。

#### 什么是域
其实就是一组服务器资源，就比如一个tomcat可以部署多个项目。
weblogic的优势就在于域的管理。而这点是tomcat没有的。

#### 启动weblogic
使用startWeblogic.cmd来启动。
启动之后可以通过console来访问后台。
输入配置的时候的用户名密码。


#### myEclipse里面启动
1. 简单的空白项目
    - 新建一个web project,比如test
    - 把项目导出为war包，放到domain里面。具体是domain的autodeploy里面。
    - 访问的时候，也是127.0.0.1:7001/test/
    - 修改domain的conifg文件下的conifg.xml可以修改监听端口号等一些东西。

2. 稍微复杂的ssh项目。
    - 导出war包。
    - 放到autodeploy

#### 配置相关文件
- setDomainEnv.cmd
    >启动管理和被管理服务器的时候，参数信息记录在这个文件里面。位置在bin里面
- startManagedWebLogic.cmd
    >启动或者关闭受管理的服务器。也在bin里面
- startWebLogic.cmd/stopWebLogic.cmd
    >启动weiblogic，bin目录下，和根目录下的那个是一样的。
- config.xml        
    > 放在config文件夹下
    > 里面包含了一系列的xml元素，其中域元素是顶级元素，域中所有元素都是子元素。包括服务器，集群，应用等
    > 可以配置web程序也可以配置EJB
- weblogic.xml
    > 放在config文件夹下
    > 可以放在web.xml同步目录下，然后进行一些配置
    > 比如charset-params等，是对web.xml的补充。jsp-description->page-checkseconds用来判断是否重新编译jsp 
- logs文件夹，日志文件夹
    >域日志，base_domain.log
    >服务器日志，***Server.log
    >访问日志,access.log 


#### 配置weblogic数据源
1. 将mysql-connector-java-5.1.6-bin.jar拷贝到E:\Oracle\Middleware\Oracle_Home\wlserver\server\lib
2. 在setDomainEnv.cmd设置 set PRE_CLASSPATH = %WL_HOME%\server\lib\mysql-connector-java-5.1.6-bin.jar
3. 登录console，选择数据源，跳转到数据源页面。
4. 新建数据源。然后填写链接数据源需要的信息。测试成功点击完成就可以了。
5. 在数据库页面的目标标签里面选择服务器。

#### 链接weblogic数据源
修改项目的数据库配置文件，指向weblogic的数据源。


