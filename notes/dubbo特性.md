---
tags: [java/中间件]
title: dubbo特性
created: '2020-04-16T04:00:21.664Z'
modified: '2020-11-02T08:45:24.560Z'
---

# dubbo特性
- 面向接口的高性能RPC调用
> 服务端只需要对消费端提供接口就行，屏蔽了调用细节
- 智能负载均衡
>内置多种负载均衡策略，均匀分摊流量压力
- 服务注册与发现
>支持多种注册中心，服务上线下线及时感知
- 高度可扩展能力
> 支持第三方实现
- 运行期流量调度
> 内置条件，脚本路由等策略，通过配置不同路由规则，轻松实现灰度发布（先发布部分服务器）。
- 可视化服务治理与运维
> 可以通过可视化界面看到服务状态，调用情况

#### dubbo服务端配置
- 指定当前服务名称dubbo:application
- 指定注册中心位置dubbo:registry
- 指定服务端提供的通信协议dubbo:protocol
- 暴漏服务dubbo:service
#### dubbo消费端配置
- 指定当前服务名称dubbo:application
- 指定注册中心位置dubbo:registry
- 引用服务 dubbo:reference
 



dubbo:reference 的 check属性表示是否在启动时检查服务是否存在也可以通过dubbo:consumer check 统一为消费端配置是否检查

dubbo:reference 的timeout表示多长时间调用超时，还可以通过method标签为某一个方法配置超时时间。
dubbo:service 也可以配置timeout。
优先级总结来说，精确优先，相同级别消费者优先

dubbo:provider 统一配置提供方属性

retries 重试次数，不包含第一次调用
重试会换服务器
幂等方法可以充实，非幂等方法不能重试

dubbo可以同一个服务存在多个版本

stub属性可以实现本地存根功能。可以在服务端先拿到远程代理对象，本地处理一些逻辑，然后判断是否调用远程代理对象。
stub对应类要实现BarService接口。

springboot整合dubbo
1. 导入dubbo-starter，在application.properties配置属性，使用@service（dubbo的）暴露服务使用@Reference引用服务。使用@EnableDubbo开启dubbo扫描
2. 保留dubbo的xml配置文件。不使用@EnableDubbo，而使用@importResource导入配置文件。
3. 使用注解api的方式每一标签都通过代码方式配置。配置类注解@Configuration
4. 
#### 高可用--减少系统不能提供服务的时间
- 注册中心down机
    - 仍然可以通过本地缓存实现通信
    - 没有注册中心的时候，也可以通过dubbo本地直连实现通信@Reference(ip:端口)

- 集群模式下负载均衡策略
    - RandomLoadBalance 基于权重的随机负载
    - RoundRobinLoadBalance 基于权重的轮询
    - LeasetActiveLoadBalance 最少活跃数（上一次请求最快的）
    - ConsistentHashLoadBalance 一致性哈希（方法名参数名都一样就是一台机器）

#### 服务降级
对某些服务有策略的不处理，或者更换简单的处理方式。来保证系统的稳定
- mock-false:return+null  消费放对服务方调用都直接返回空，并不进行远程调用
- mock=fail:return+null 当消费调用失败的时候返回null不抛出异常。来容忍不重要服务抛出异常
> 上面两个在dubbo admin里面对应屏蔽和容错


#### 集群容错
- dubbo:service 或者dubbo:reference 配置cluster 
- 默认的cluster是failover 就是失败重试，重试次数有retries来决定

实际操作中会整合hystrix。有更丰富的处理方式


#### dubbo 原理
1. 标签设计
    - DubboNameSpaceHandler 注册各种parser
    - DubboBeanDefinitionParser读取xml标签并且注入到对应的config文件或者serviceBean，referenceBean文件。
2. 服务暴漏流程
    - serviceBean实现了InitializingBean和ApplicationListener 等等
    - 实现了InitializingBean的类会在spring实例化之后调用afterPropertiesSet方法
    - 实现了ApplicatonListener<ContextRefreshContext>的类，在ioc容器刷新完成之后会调用onApplicationEvent方法。
    - afterPropertiesSet 里面会把之前识别的配置信息都保存起来
    - onApplicatonEvent里面会调用export方法
    - 暴漏的时候，先去registryProtocal然后调用DubboProtcal，然后在dubboProtocal里面openServer。这个其实就是个nettyserver。然后再在registryProtocal里面注册provider，其实就是维护一个map，key是url，value是封装实现类的invoker。最后再去注册中心把url注册上去
3. 引用服务流程
    - referenceBean实现了factoryBean。实现了这个接口的类，在ioc初始化获得对象的时候会调用getObject方法
    - referenceBean继承了referenceConfig，所以调用referenceConfig的createCreateProxy方法创建代理
    - 然后现在registryProtocol里面，先根据registry获得注册中心的信息。然后获取注册中的url，封装成invoker。invoker里面包含nettyClient(在dubboProtocol里面封装的)。
4. 调用服务流程
    - 上面引用服务会创建一个代理对象Involer
    - 要执行invoke方法的时候，会封装一个rpcInvocation对象到invoke 方法里面
    - 最终会是在dubboinvoker进行远程调用，调用只传输数据。
    
#### dubbo 调用过程总结
- 暴漏服务
    1. spring通过SPI调用dubboNameSpaceHandler来解析dubbo的配置文件，生成一个serviceBean
    2. 由于serviceBean实现了initialingBean，如果该服务没有配置delay属性或者delay属性配置的是0，会在springBean创建成功的时候，回调了afterPropertiesSet方法，进行暴漏，如果大于0会在等待指定时间之后调用。如果配置了delay=-1，由于serviceBean实现了applicationListener接口，所以会在spring启动成功之后回调onApplicationEvent方法进行暴漏。
    3. 在serviceConfig中先封装一个Invoker，该invoker代理了service具体实现类。然后通过dubbo的spi继续暴漏
    4. ProtocolFilterWapper里面把invoker封装成带filter调用链的invoker。
    5. registryProtocol，连接zk，并且把服务URL注册到zk上，然后调用dubboProtocol,最后再把invoker封装成exportor
    6. dubboProtocol,开启nettyServer，并且把封装了filter的invoker封装成exporter。并放入exprotermap，key是服务:端口号，value是exportor
- 消费端
    1. spring通过SPI调用dubboNameSpaceHandler来解析dubbo的配置文件，生成一个referenceBean
    2. 由于referenceBean实现了factoryBean接口，所以会调用getObject方法创建该对象。并且由于也实现了initializingBean接口，也会回调afterPropertiesSet，而该方法也会调用getObject方法。
    3. 在DubboProtocol里面refer方法，new一个invoker把接口，url，nettyclient封装进去
    4. 在ProtocolFilterWapper里面把dubboProtocol封装的invoker拼上装成带filter链的invoker。
    5. 在registryProtocol里面，先去zk订阅所有服务的RUL，然后再把带filter的invoker（3，4）作为服务放到本地缓存中
    6. 在referenceConfig里面把服务对应的路径重新封装成一个FailOverClusterinvoker，并做为代理对象使用
 - 服务调用
    - 消费端
        1. 消费端拿到的是一个代理对象，所有调用会被代理到FailOverClusterinvoker的invoke方法。
        2. FailOverClusterinvoker的invoke方法中，先拿到配置的负载均衡策略，然后根据负载均衡策略去本地缓存map中取到带filter链的Invoker。
        3. 由于filter链的末端是dubboInvoker所以请求最终被代理给了dubboInvoker的invoke方法。
        4. 在dubboInvoker的invoke方法里面，拿到之前封装的nettyClient，然后向服务器发送请求。参数是RpcInvocation。包含服务名，参数
    - 服务端
        1. 服务端收到请求会回调dubboProtocol的reply方法
        2. reply方法接收到rpcInvocation之后，通过请求参数去exportorMap里面获取exporter
        3. 从exprorter中取出带filter的invoker，依次调用invoke方法
        4. 最终代理到springBean的具体实现类。通过反射调用实现类的方法。

### 2020年11月2日 杂记
- spring 初始化的时候会调用AbstractApplicationContext#refresh方法。在这个方法中。obtainFreshBeanFactory 会去加载beanDefination，这里暂时只关注dubbo相关加载。
- spring解析dubbo的配置文件把配置转换为对应的beanDefination。这个beanDefination是一个RootBeanDefination里面会有对应的beanClass，比如一个registry节点对应的beanclass就是class com.alibaba.dubbo.config.RegistryConfig.
- 上面的beanDefination会注册到一个map里面。注册方式是，ParserContext.getRootContext.getRegistry.registerBeanDefinition.最终是调用DefaultListableBeanFactory#registerBeanDefinition，因为这个类实现了registry接口。
- AbstractApplicationContext#finishBeanFactoryInitialization 里面会实例化所有的单例。
  - 初始化单例的时候先调用构造函数创建出来对象
  - 对象会被放到（map）singletonFactories，（LinkList）registeredSingletons
  - 创建出来对象之后，会根据bean上实现的接口调用对应的方法进行一些初始化工作。



