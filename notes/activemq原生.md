---
tags: [java/中间件]
title: activemq原生
created: '2020-04-17T07:09:32.228Z'
modified: '2020-04-26T07:40:39.940Z'
---

# activemq原生
发送/接收消息（队列）

1.创建一个connection factory
2.通过connection factory来创建JMS connection
3.启动JMS connection
4.通过connection 创建JMS session
5.通过session创建JMS destionation
6.创建JMS producer或者创建JMS message并设置desitination
7.创建JMS consumer或者注册一个JMS message listeners
8.发送或者接收JMS messages
9.关闭所有JMS资源

发送/接收消息（主题）

1.创建一个connection factory
2.通过connection factory来创建JMS connection
3.启动JMS connection
4.通过connection 创建JMS session
5.<font color='red'>通过session创建Topic</font>
6.创建JMS producer或者创建JMS message并设置<font color='red'>Topic</font>
7.创建JMS consumer或者注册一个JMS message listener
8.发送或者接收JMS messages
9.关闭所有JMS资源

activemq 队列的默认负载均衡是轮询
比如10条消息，两个消费者监听，那么会每个消费者收到两个。

#### 队列
1. 生产者把消息发送到queue之中，每个消息一个消费者，属于1：1关系
2. 生产者和消费者之间无时间相关性，消费者可以消费自己启动之前生产者发送的消息

#### 主题
1. 生产者把消息发送到topic中，每个消息可以有多个消费者，属于1：N关系
2. 生产者和消费者之间有时间相关性，订阅某个主题的消费者，只能消费自它订阅之后发布的消息
3. 要先启动消费者订阅，然后再启动生产者，如果没有订阅者的话，生产的消息属于废消息

#### JAVAEE
一套使用java进行企业级应用开发的，大家一致遵循的13个核心规范工业标准。javaEE平台提供了一个基于组件的方法来加快设计，开发，装配以及部署企业应用程序
1. JDBC 数据库连接
2. JNDI java的命名和目录接口
3. EJB Enterprise JavaBean
4. RMI 远程调用
5. Java IDL/CORBA 接口定义语言/公用对象请求代理程序体系结构
6. JSP
7. Servlet
8. XML 可扩展字标记语言
9. JMS java消息服务
10. JTA java事物API
11. JTS java事物服务
12. JavaMail
13. JAF JavaBean Activition Framework

#### JMS
java消息服务是指两个应用程序之间进行异步通信的API
两个程序使用jms通信，他们之间不直接相连，通过共同的消息收发服务关联起来达到，<font color='red'>解耦，异步，削峰</font>的目的

#### 消息组成
- 消息头由很多属性，常用几个
1. deliverMode 持久非持久。
2. timeToLive(JMSExpiration) 过期时间，默认永不过期。
3. priority 优先级。总共0到9，其中0到4是普通消息，5到9是加急消息。不严格按照优先级排序，但是会保证加急消息先于普通消息送达。
4. messageId消息的唯一识别编号。
- 消息体有五种格式
1. TextMessage
2. MapMessage
3. BytesMessage
4. StreamMessage
5. ObjectMessage
- 消息属性
如果需要消息头意外的标识，可以增加消息属性。
通过StringProperties，BooleanBroperties等等。

#### JMS可靠性
1. 持久性
    > deliveryMode 持久化模式，如果是非持久化，重启消息会丢失，如果是持久化，消息会被保存在硬盘，重启会重新加载
    > 队列是默认持久化
    > topic持久化，需要手动设置。发布的时候一定要在Producer上先设置持久化，然后再打开连接。消费的时候，需要使用DurableSubscriber进行订阅
2. 事物
    > 在createSession可以设置事物，如果设置false，send就可以把消息发送到服务器，如果是true一定要手动commit才能提交到服务器。反之消费者也一样，如果true，也需要手动commit，服务器的消息才会被消费掉。
3. 签收
事物大于签收，如果开启事物了，一般以事物为准.所以签收一般是指非事物情况下才会生效

 -|- |-
---|---|-
自动签收(默认)|auto_acknowledge|消费接收消息之后，消息就被消费掉了
手动签收|client_acknowledge|手动acknowledge之后，消息才会算被消费掉
允许重复消息|DUPS_OK_ACKNOWLEDGE|允许重复签收，不一定会出现重复现象。

#### broker
其实就是实现了代码的形式启动activeMq，将mq嵌入到java代码中，使用的时候随时去启动。相当于服务器实例
具体操作如下
```
<!--activemq broker需要引入的-->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.9.6</version>
            <optional>true</optional> <!-- 这个需要为 true 热部署才有效 -->
        </dependency>
//手动启动broker

```
#### activemq 传输协议
MQ支持的通信协议有：TCP，NIO，UDP，SSL，https，VM等
在activemq 安装目录的conf/activemq.xml中的<transportconnectors>标签之内
NIO 性能要更好。下面是例子
```
<transportConnector name="openwire" uri="tcp://0.0.0.0:61616?maximumConnections=3000&amp;wireFormat.maxFrameSize=104857600"/>
<transportConnector name="amqp" uri="amqp://0.0.0.0:5672?maximumConnections=3000&amp;wireFormat.maxFrameSize=104857600"/>
```
为什么叫做openwire是因为，网络传输数据前，必须序列化数据，消息是通过wire protocol的来序列化成字节流，所以用openwire指代wire protocol。
TCP的URI形式如 tcp://hostname:port?key=value&key=value 参数是可选的。
TCP：
- 可靠性高稳定性强。
- 高效性：字节流传递，效率高
- 有效性，可用性：应用广泛，支持任何平台。
NIO：
- 和tcp类似，但是更侧重底层访问操作，允许开发人员对同一资源有个多的client调用和服务端有更多的负载
- 当有大量client连接到broker上，NIO更好。
- 因为NIO性能更好，所以当网络迟钝的情况下可以用NIO。
- 客户端使用nio://hostname:port?......


如果您不特别指定ActiveMQ的网络监听端口，那么这些端口都将使用BIO网络IO模型
如果想用NIO。<transportConnector name="nio" uri="nio://0.0.0.0:61618?maximumConnections=1000"/> 表示TCP协议为基础的NIO网络IO模型。相当于openwire采用NIO
如果想用别的协议加NIO网络模型，activemq支持用+配置
```
<transportConnector name="stomp+nio" uri="stomp+nio://0.0.0.0:61613?transport.transformer=jms"/>
// 表示这个端口使用NIO模型支持Stomp协议
<transportConnector name="amqp+ssl" uri="amqp+ssl://localhost:5671"/>
// 表示这个端口支持amqp和ssl密文传输
```
配置协议的时候也可以使用auto，auto表示根据请求自动识别相应的协议。

我们系统中客户端参数是
failover://(tcp://******?wireFormat.maxInactivityDuration=0)?randomize=false
其中，failover表示断线重连，如果当前连接断掉，客户端会不断从括号中取url尝试重新连接。自动重连的线程是deamon的，可以手动设置为false
wireFormat.maxInactivityDuration=0 这样设置表示永远不合服务器断开连接。就算长时间没有消息接收或者发送仍然处在连接中。
randomize=false 使客户端首先连接到主节点，并在主节点不可用时只连接到辅助备份代理
#### activemq消息存储和持久化
#### activemq多节点集群
#### activemq高级特性和大厂常考重点








