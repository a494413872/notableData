---
attachments: [Clipboard_2019-11-27-11-15-09.png, Clipboard_2019-11-27-11-16-30.png, Clipboard_2019-11-27-11-16-36.png, Clipboard_2019-11-27-11-17-27.png, Clipboard_2019-11-27-11-17-37.png, Clipboard_2019-11-27-11-17-46.png, Clipboard_2019-11-27-11-17-55.png, Clipboard_2019-11-27-11-18-01.png, Clipboard_2019-11-27-11-18-06.png, Clipboard_2019-11-27-11-18-30.png, Clipboard_2019-11-27-11-18-37.png, Clipboard_2019-11-27-11-19-53.png, Clipboard_2019-11-27-11-20-13.png, Clipboard_2019-11-27-11-20-20.png, Clipboard_2019-11-27-11-20-29.png, Clipboard_2019-11-27-11-20-36.png, Clipboard_2019-11-27-11-20-45.png, Clipboard_2019-11-27-11-20-58.png, Clipboard_2019-11-27-11-21-05.png, Clipboard_2019-11-27-11-21-40.png, Clipboard_2019-11-27-11-22-17.png, Clipboard_2019-11-27-11-22-22.png, Clipboard_2019-11-27-11-22-30.png, Clipboard_2019-11-27-11-22-40.png, Clipboard_2019-11-27-11-22-54.png, Clipboard_2019-11-27-11-23-39.png, Clipboard_2019-11-27-11-24-13.png, Clipboard_2019-11-27-11-24-19.png]
tags: [java/中间件]
title: activemq
created: '2019-11-27T02:46:13.998Z'
modified: '2019-12-05T05:47:16.443Z'
---

# activemq
本文分四个部分：
1.Jms简介
2.spring整合activeMq
3.我们系统的消息
4.新消息设计

## JMS 简介
JMS的全称是Java Message Service，即Java消息服务。
它主要用于在生产者和消费者之间进行消息传递，生产者负责产生消息，而消费者负责接收消息。
对于消息的传递有两种类型：
1.	是点对点的，即一个生产者和一个消费者一一对应
 ![](@attachment/Clipboard_2019-11-27-11-16-30.png)![](@attachment/Clipboard_2019-11-27-11-16-30.png)
2.	是发布/订阅模式，即一个生产者产生消息并进行发送后，可以由多个消费者进行接收。
 ![](@attachment/Clipboard_2019-11-27-11-16-36.png)![](@attachment/Clipboard_2019-11-27-11-16-36.png)
Spring整合JMS
JMS是一个标准，下面是有不同的实现的。这里只是简单介绍下，ActiveMq的实现是如何整合到Spring里面的。

## ActiveMQ准备
既然是使用的apache的activeMQ作为JMS的实现，那么首先我们应该到apache官网上下载activeMQ（http://activemq.apache.org/download.html），进行解压后运行其bin目录下面的activemq.bat文件启动activeMQ。
里面还有很多配置细节，这里就不一一介绍了。

配置connectionFactory
Spring对于JMS提供了三种connection Factory:
1.	SingleConnectionFactory 
最基础的connecton factory。它将在所有的createConnection()调用中返回同一个相同的共享连接对象， 并且忽略Connection.close()和stop()的调用。根据JMS连接模型，这是完全线程安全的。这个共享连接能够在出现异常时自动恢复创建一个新的共享连接。可以通过SingleConnectionFactory的构造函数中传入Connection对象或者 ConnectionFactory对象，用来创建被代理的连接对象。由于我们使用的是ActiveMQ所以，我们要把ActiveMQConnectionFactory给注入到里面，这样返回的就是ActiveMQ的Connection了。
2.	PoolConnectionFactory 
为因JmsTemlate每次发送消息时都会重新创建连接，创建connection，session，创建productor。这是一个非常耗性能的地方，特别是大数据量的情况下。因此出现了PooledConnectionFactory。这个类只会缓存connection，session和productor，不会缓存consumer。因此只适合于生产者发送消息
3.	CachingConnectionFactory
扩展自SingleConnectionFactory，主要用于提供缓存JMS资源功能。具体包括MessageProducer、MessageConsumer和Session的缓存功能

Spring中发送消息的核心是JmsTemplate，然而Jmstemplate的问题是在每次调用时都要打开/关闭session和producter，效率很低，所以引申出了PooledConnectionFactory连接池，用于缓存session和producter。然而这还不是最好的。从spring2.5.3版本后，Spring又提供了CachingConnectionFactory，这才是首选的方案。然而CachingConnectionFactory有一个问题必须指出，默认情况下，CachingConnectionFactory只缓存一个session，在它的JavaDoc中，它声明对于低并发情况下这是足够的。与之相反，PooledConnectionFactory的默认值是500。

我们这里就用SingleConnectionFactory，然后传入ActiveMQConnectionFactory，用来产生connection。
ActiveMQConnectionFactory有一个重要的属性，brokerURL，这里配置的就是jms服务器的地址。我们的具体地址配置在properties文件里面，这里放上属性名就好了。
![](@attachment/Clipboard_2019-11-27-11-17-27.png)![](@attachment/Clipboard_2019-11-27-11-17-27.png)
![](@attachment/Clipboard_2019-11-27-11-17-37.png)![](@attachment/Clipboard_2019-11-27-11-17-37.png)
发送消息
我们想要发送消息的话，一般是使用工具类JmsTemplate。这个工具类会帮我们完成发消息的细节。但是既然是发消息到ActiveMQ，JmsTemplate首先需要获取到一个链接到ActiveMQ的链接。这时候就需要我们把之前配置的connectionFactory给传进去。
![](@attachment/Clipboard_2019-11-27-11-17-46.png)![](@attachment/Clipboard_2019-11-27-11-17-46.png)
当JmsTemplate获取链接之后，我们就可以发消息了，但是这个消息要发送到哪里，这个Destination也是需要我们配置的。Destination有两种，1.一种是queue，2.另一种是topic。这里我们以queue作为例子。
![](@attachment/Clipboard_2019-11-27-11-17-55.png)![](@attachment/Clipboard_2019-11-27-11-17-55.png) 
时候我们的准备条件都已经齐备了，我们可以书写自己的生产者了。然后再把JmsTemplate和Destination给注入进去，然后就可以发消息了。
![](@attachment/Clipboard_2019-11-27-11-18-01.png)![](@attachment/Clipboard_2019-11-27-11-18-01.png) 
![](@attachment/Clipboard_2019-11-27-11-18-06.png)![](@attachment/Clipboard_2019-11-27-11-18-06.png)
发消息写的差不多了，我们写一个测试类测试一下。
![](@attachment/Clipboard_2019-11-27-11-18-30.png)![](@attachment/Clipboard_2019-11-27-11-18-30.png) 
输出如下：
![](@attachment/Clipboard_2019-11-27-11-19-53.png)![](@attachment/Clipboard_2019-11-27-11-19-53.png) 
程序执行正常，但是是否正确发送出去了，这就需要我们写一个消费者来接收下消息了。
接收消息
接收消息的话，spring提供了两个消息监听容器，1. SimpleMessageListenerContainer和2. DefaultMessageListenerContainer。1.的话接近于原生JMS规范，不太兼容JavaEE，所以不管它。我们用2。

像发送的时候一样，我们需要提供connection这样程序才能连接到ActiveMQ，我们需要告诉它Destination，这样程序才能知道监听那个Queue，还有我们需要提供一个MessageListener来对消息进行处理。
![](@attachment/Clipboard_2019-11-27-11-20-13.png)![](@attachment/Clipboard_2019-11-27-11-20-13.png) 
MessageListener也是有三种的
1.	就是我们例子中的，MessageListener。是最原始的监听器，我们自己的实现类MessageConsumerTest需要实现MessageListener接口，其中定义了一个用于处理接收到的消息的onMessage方法，该方法只接收一个Message参数。Java代码如下。
![](@attachment/Clipboard_2019-11-27-11-20-20.png)![](@attachment/Clipboard_2019-11-27-11-20-20.png)      
2.	SessionAwareMessageListener，算是MessageListener的增强版，它的onMessage方法有两个参数(message,session)，这样的话，如果我们接收到消息之后需要发送另外一个消息的情况，比如要发送一个收到的消息给生产者。这时候我们就可以直接从Session来创建producer而不用从新获取connection和session了。
同样的，具体监听器的java代码上面也需要实现SessionAwareMessageListener这个接口。
3.	MessageListenerAdapter。这个类主要功能是，可以把消息转化为普通对象，并且交给一个普通的java类来处理。不需要类再继承什么东西。并且可以把对应的消息对象转换为对应的对象，比如TextMessage转换为String等等。改造后的配置文件和java类如下。
![](@attachment/Clipboard_2019-11-27-11-20-29.png)![](@attachment/Clipboard_2019-11-27-11-20-29.png)
![](@attachment/Clipboard_2019-11-27-11-20-36.png)![](@attachment/Clipboard_2019-11-27-11-20-36.png)
![](@attachment/Clipboard_2019-11-27-11-20-45.png)![](@attachment/Clipboard_2019-11-27-11-20-45.png)
##我们系统中的消息
有了上面的基础，我们再来看我们系统中的具体实现。
我们以Scenic_back作为例子。我们先看Scenic的配置文件。
![](@attachment/Clipboard_2019-11-27-11-20-58.png)![](@attachment/Clipboard_2019-11-27-11-20-58.png) 
然后再看看vst_back的配置文件。
![](@attachment/Clipboard_2019-11-27-11-21-05.png)![](@attachment/Clipboard_2019-11-27-11-21-05.png) 
可以看出，首先scenic_back会继承scenic_common的配置文件。并且注释掉了***processer.xml，这个配置文件,这个配置文件是接收消息的配置文件，所以从这里可以看出来实际上scenic_back是不接收消息的，不过我们也会介绍一下，如果不注释掉，接收消息的时候是应该怎么做。
既然scenic_back继承scenic_common的配置文件所以我们也看下scenic_common的配置文件。
![](@attachment/Clipboard_2019-11-27-11-21-40.png)![](@attachment/Clipboard_2019-11-27-11-21-40.png) 
由于我们的配置文件引入了activeMQ的Schema，所以我们可以采用更简便的方式来配置我们所需要的东西。
就如上面讲的一样，要发消息的话，要通过JmsTemplate，而JmsTemplate需要我们传入ConnectionFactory，ConnectionFactory又要我们制定具体的ActiveMq的ConnectionFactory。
![](@attachment/Clipboard_2019-11-27-11-22-17.png)![](@attachment/Clipboard_2019-11-27-11-22-17.png) 
![](@attachment/Clipboard_2019-11-27-11-22-22.png)![](@attachment/Clipboard_2019-11-27-11-22-22.png) 
然后我们就可以使用JmsTemplate，我们以申码消息为例子。
![](@attachment/Clipboard_2019-11-27-11-22-30.png)![](@attachment/Clipboard_2019-11-27-11-22-30.png) 
这里可以看出来真正发消息是调用的vst_comm的TopicMessageProducer。代码如下
![](@attachment/Clipboard_2019-11-27-11-22-40.png)![](@attachment/Clipboard_2019-11-27-11-22-40.png) 
这个类的名字虽然叫做TopicMessageProducer，但是并不是发送topic类型的消息，而是发送queue类型的消息，只是通过预先定义好的数组来遍历的发送消息到不同的queue来实现类似于topic的效果。遍历的数组来自于MessageProtector。比如我们的例子里面，最终要发送到的ActiveMQ.VST_CODE_PASSPORT.passport这个queue里面。
![](@attachment/Clipboard_2019-11-27-11-22-54.png)![](@attachment/Clipboard_2019-11-27-11-22-54.png) 
上面的TopicMessageProducer类里面还有一点要说明一下。
和我们的例子不同的的是，它发送的是一个自定义Message对象，这时候JsmTemplate的convertAndSend方法，是可以把我们自定的Message对象给转化为jms的标准消息对象然后发送出去的方法。
下面我们再说一下，消费者，虽然现在Scenic里面暂时不启用，但是为了以后改造方便也记录下。我们以下面这个为例子。
![](@attachment/Clipboard_2019-11-27-11-23-39.png)![](@attachment/Clipboard_2019-11-27-11-23-39.png)
从配置文件可以看出，我们使用的MessageListenerAdapter，然后我们自定义的处理类是TopicMessageConsumer。这个类也是模拟了一个Topic的机制，通过自己配置processorList来实现多个processor循环来处理这个message，相当于一个message被多个consumer给消费了。这里面有两点需要说明，1.首先这个TopicMessageConsumer通过配置文件已经可以看出是迁移到子系统了，但是里面引用的Message和MessageProcessor并没有迁移过来，还是用的vst的。2.监听的queue是这里配置的<property name="destination" ref="VST_CONTROL_TOPIC" />而不是，ActiveMQ.VST_CONTROL.${jms_node}。从代码也可以看出来后者并没有什么用。而前者是引用的Scenic_common的配置 
![](@attachment/Clipboard_2019-11-27-11-24-13.png)![](@attachment/Clipboard_2019-11-27-11-24-13.png) 
虽然在consumer的配置里两个值是一样的，但是意义完全不同。
TopicMessageConsumer具体代码如下：
![](@attachment/Clipboard_2019-11-27-11-24-19.png)![](@attachment/Clipboard_2019-11-27-11-24-19.png) 

## 本次改动
现有消息类属性如下，我认为完全符合我们新消息的需求。所以不需要做出改动。
 
业务相关字段对应关系如下：
ObjectIdsupp_goods_id
Objectypesupp_goods,time_price（待定）
eventTypeupdate,add,…..
addition变更字段名，用逗号分隔。(非必填)

在ScenicCommn配置文件里面添加生产者。数量待定
 

修改vst_common在MessageProtector里面配置对应的消息队列。

剩下的就是在对应的修改点，调用我们的生产者的Send方法，把拼装好的消息发送出去。

本文分四个部分：
1.Jms简介
2.spring整合activeMq
3.我们系统的消息
4.新消息设计





































JMS 简介
JMS的全称是Java Message Service，即Java消息服务。
它主要用于在生产者和消费者之间进行消息传递，生产者负责产生消息，而消费者负责接收消息。
对于消息的传递有两种类型：
1.	是点对点的，即一个生产者和一个消费者一一对应
 
2.	是发布/订阅模式，即一个生产者产生消息并进行发送后，可以由多个消费者进行接收。
 
Spring整合JMS
JMS是一个标准，下面是有不同的实现的。这里只是简单介绍下，ActiveMq的实现是如何整合到Spring里面的。



ActiveMQ准备
既然是使用的apache的activeMQ作为JMS的实现，那么首先我们应该到apache官网上下载activeMQ（http://activemq.apache.org/download.html），进行解压后运行其bin目录下面的activemq.bat文件启动activeMQ。
里面还有很多配置细节，这里就不一一介绍了。

配置connectionFactory
Spring对于JMS提供了三种connection Factory:
1.	SingleConnectionFactory 
最基础的connecton factory。它将在所有的createConnection()调用中返回同一个相同的共享连接对象， 并且忽略Connection.close()和stop()的调用。根据JMS连接模型，这是完全线程安全的。这个共享连接能够在出现异常时自动恢复创建一个新的共享连接。可以通过SingleConnectionFactory的构造函数中传入Connection对象或者 ConnectionFactory对象，用来创建被代理的连接对象。由于我们使用的是ActiveMQ所以，我们要把ActiveMQConnectionFactory给注入到里面，这样返回的就是ActiveMQ的Connection了。
2.	PoolConnectionFactory 
为因JmsTemlate每次发送消息时都会重新创建连接，创建connection，session，创建productor。这是一个非常耗性能的地方，特别是大数据量的情况下。因此出现了PooledConnectionFactory。这个类只会缓存connection，session和productor，不会缓存consumer。因此只适合于生产者发送消息
3.	CachingConnectionFactory
扩展自SingleConnectionFactory，主要用于提供缓存JMS资源功能。具体包括MessageProducer、MessageConsumer和Session的缓存功能

Spring中发送消息的核心是JmsTemplate，然而Jmstemplate的问题是在每次调用时都要打开/关闭session和producter，效率很低，所以引申出了PooledConnectionFactory连接池，用于缓存session和producter。然而这还不是最好的。从spring2.5.3版本后，Spring又提供了CachingConnectionFactory，这才是首选的方案。然而CachingConnectionFactory有一个问题必须指出，默认情况下，CachingConnectionFactory只缓存一个session，在它的JavaDoc中，它声明对于低并发情况下这是足够的。与之相反，PooledConnectionFactory的默认值是500。

我们这里就用SingleConnectionFactory，然后传入ActiveMQConnectionFactory，用来产生connection。
ActiveMQConnectionFactory有一个重要的属性，brokerURL，这里配置的就是jms服务器的地址。我们的具体地址配置在properties文件里面，这里放上属性名就好了。
 
 

发送消息
我们想要发送消息的话，一般是使用工具类JmsTemplate。这个工具类会帮我们完成发消息的细节。但是既然是发消息到ActiveMQ，JmsTemplate首先需要获取到一个链接到ActiveMQ的链接。这时候就需要我们把之前配置的connectionFactory给传进去。
 
当JmsTemplate获取链接之后，我们就可以发消息了，但是这个消息要发送到哪里，这个Destination也是需要我们配置的。Destination有两种，1.一种是queue，2.另一种是topic。这里我们以queue作为例子。
 
时候我们的准备条件都已经齐备了，我们可以书写自己的生产者了。然后再把JmsTemplate和Destination给注入进去，然后就可以发消息了。
 
 
发消息写的差不多了，我们写一个测试类测试一下。
 
输出如下：
 
程序执行正常，但是是否正确发送出去了，这就需要我们写一个消费者来接收下消息了。
接收消息
接收消息的话，spring提供了两个消息监听容器，1. SimpleMessageListenerContainer和2. DefaultMessageListenerContainer。1.的话接近于原生JMS规范，不太兼容JavaEE，所以不管它。我们用2。

像发送的时候一样，我们需要提供connection这样程序才能连接到ActiveMQ，我们需要告诉它Destination，这样程序才能知道监听那个Queue，还有我们需要提供一个MessageListener来对消息进行处理。
 
MessageListener也是有三种的
1.	就是我们例子中的，MessageListener。是最原始的监听器，我们自己的实现类MessageConsumerTest需要实现MessageListener接口，其中定义了一个用于处理接收到的消息的onMessage方法，该方法只接收一个Message参数。Java代码如下。
      
2.	SessionAwareMessageListener，算是MessageListener的增强版，它的onMessage方法有两个参数(message,session)，这样的话，如果我们接收到消息之后需要发送另外一个消息的情况，比如要发送一个收到的消息给生产者。这时候我们就可以直接从Session来创建producer而不用从新获取connection和session了。
同样的，具体监听器的java代码上面也需要实现SessionAwareMessageListener这个接口。
3.	MessageListenerAdapter。这个类主要功能是，可以把消息转化为普通对象，并且交给一个普通的java类来处理。不需要类再继承什么东西。并且可以把对应的消息对象转换为对应的对象，比如TextMessage转换为String等等。改造后的配置文件和java类如下。
 
 
 

我们系统中的消息
有了上面的基础，我们再来看我们系统中的具体实现。
我们以Scenic_back作为例子。我们先看Scenic的配置文件。
 
然后再看看vst_back的配置文件。
 
可以看出，首先scenic_back会继承scenic_common的配置文件。并且注释掉了***processer.xml，这个配置文件,这个配置文件是接收消息的配置文件，所以从这里可以看出来实际上scenic_back是不接收消息的，不过我们也会介绍一下，如果不注释掉，接收消息的时候是应该怎么做。
既然scenic_back继承scenic_common的配置文件所以我们也看下scenic_common的配置文件。
 
由于我们的配置文件引入了activeMQ的Schema，所以我们可以采用更简便的方式来配置我们所需要的东西。
就如上面讲的一样，要发消息的话，要通过JmsTemplate，而JmsTemplate需要我们传入ConnectionFactory，ConnectionFactory又要我们制定具体的ActiveMq的ConnectionFactory。
 
 
然后我们就可以使用JmsTemplate，我们以申码消息为例子。
 
这里可以看出来真正发消息是调用的vst_comm的TopicMessageProducer。代码如下
 
这个类的名字虽然叫做TopicMessageProducer，但是并不是发送topic类型的消息，而是发送queue类型的消息，只是通过预先定义好的数组来遍历的发送消息到不同的queue来实现类似于topic的效果。遍历的数组来自于MessageProtector。比如我们的例子里面，最终要发送到的ActiveMQ.VST_CODE_PASSPORT.passport这个queue里面。
 
上面的TopicMessageProducer类里面还有一点要说明一下。
和我们的例子不同的的是，它发送的是一个自定义Message对象，这时候JsmTemplate的convertAndSend方法，是可以把我们自定的Message对象给转化为jms的标准消息对象然后发送出去的方法。

下面我们再说一下，消费者，虽然现在Scenic里面暂时不启用，但是为了以后改造方便也记录下。我们以下面这个为例子。
 
从配置文件可以看出，我们使用的MessageListenerAdapter，然后我们自定义的处理类是TopicMessageConsumer。这个类也是模拟了一个Topic的机制，通过自己配置processorList来实现多个processor循环来处理这个message，相当于一个message被多个consumer给消费了。这里面有两点需要说明，1.首先这个TopicMessageConsumer通过配置文件已经可以看出是迁移到子系统了，但是里面引用的Message和MessageProcessor并没有迁移过来，还是用的vst的。2.监听的queue是这里配置的<property name="destination" ref="VST_CONTROL_TOPIC" />而不是，ActiveMQ.VST_CONTROL.${jms_node}。从代码也可以看出来后者并没有什么用。而前者是引用的Scenic_common的配置 
 
虽然在consumer的配置里两个值是一样的，但是意义完全不同。
TopicMessageConsumer具体代码如下：
 

本次改动
现有消息类属性如下，我认为完全符合我们新消息的需求。所以不需要做出改动。
 
业务相关字段对应关系如下：
ObjectIdsupp_goods_id
Objectypesupp_goods,time_price（待定）
eventTypeupdate,add,…..
addition变更字段名，用逗号分隔。(非必填)

在ScenicCommn配置文件里面添加生产者。数量待定
 

修改vst_common在MessageProtector里面配置对应的消息队列。

剩下的就是在对应的修改点，调用我们的生产者的Send方法，把拼装好的消息发送出去。



