---
tags: [java/中间件]
title: spring
created: '2020-11-26T08:56:31.626Z'
modified: '2021-01-21T08:18:15.767Z'
---

# spring

 能对spring源码进行二次扩展

 spring的初始化过程

 1. scann 扫描类  
 2. parse 转化为genericbeandefinition。
    把beandefinition放在一个map里面，key是beanname。同事把beanname放在一个list里面
 3. 调用扩展
 4. 创建对象


 - 实现BeanFactoryPostProcessor接口在spring启动的时候，会回调postProcessBeanFactory方法，该方法入参为ConfigurableListableBeanFactory，可以对beanDefinationMap进行操作。

- AbstractApplciationContext 里面的refresh方法是spring初始化的核心方法。
this.prepareRefresh();
this.prepareBeanFactory(beanFactory);
this.postProcessBeanFactory(beanFactory);
this.invokeBeanFactoryPostProcessors(beanFactory);在这里完成了扫描，生成了beanDeifnationMap
this.registerBeanPostProcessors(beanFactory);
this.initMessageSource();
this.initApplicationEventMulticaster();
this.onRefresh();
this.registerListeners();
this.finishBeanFactoryInitialization(beanFactory);在这里完成了实例化
this.finishRefresh();

- 在实例化过程中，会调用getBean->doGetBean方法。在这里面会调用两个getSingleton方法。如果第一个
    - getSingleton(String)先从singleObjects(一级缓存)里面尝试获取，有就返回，没有就继续。也就是说里面放的其实是完全完成了初始化，是可以使用的bean。没有的话会判断下是否正在创建（bean在singletonsCurrentlyInCreation中），由于正处在循环依赖中，所以肯定是正在创建（put的地方见@1）。这时候会尝试从earlySingleObject(二级缓存)获取，如果还没有，就从singleFactories（三级缓存）里面获取，这里面获取到的其实是ObjectFactory，然后再通过factory来getObject，如果能获取到，会把bean从三级缓移动到二级缓存。如果都获取不到就返回空回去。
    - 判断是不是单例对象，如果是单例的话会再次调用getSingleton（string,ObjectFactory）方法和上面那个并不是一个方法，而是一个重载
        - 该方法第二个参数是一个ObjectFactory的对象，在方法中会回调这个对象的getObject方法，而在getObject方法里面，会调用createBean方法。在这个getSingleton中会调用beforeSingletonCreation，把bean放在singletonsCurrentlyInCreation中（@1）。
        - 在finally里面会从singletonsCurrentlyInCreation中把beanName移除掉
        - 最终会把bean放入singletonObjects(一级缓存)，并且从二级和三级缓存中移除掉。
    - 在createBean方法里面会调用resolveBeforeInstantiation，对beanDefination中的属性进行处理。这是第一个后置处理器。
    - 下一步在doCreateBean里面
        - 如果是单例，从factoryBeanInstanceCache里面remove掉
        - 通过InstanceWapper.getWappedInstance实例化对象，只是实例化对象（通过构造函数反射调用），这时候还不是spring的bean。
        - 然后根据条件调用applyMergedBeanDefinitionPostProcessors。这里会进行一些注解的扫描。这里面会调用三个后置处理器。对注解进行扫描并做为element添加到BeanDefination里面，只是扫描并不进行注入。
            - CommonAnnotationBeanPostProcessor 扫描@PostContruct和@PreDestroy和@Resrouce
            - AutowiredAnnotationBeanPostProcessor 扫描@Autowired和@value
            - ApplicationListenerDetector 把类的isSingleton属性取出来，放到map里面singleNames
        - 判断是否允许提前暴露，有三个条件，是单例，允许循环依赖，单例正在创建中。
        - 是的话调用addSingletonFactory，把bean的ObjectFactory加入到三级缓存，并且从二级缓存删除。这时候Factory的getObject方法其实获取的是EarlyBeanReference,对bean进行封装。
        - 后面populateBean 自动注入属性，并且调用第五次和第六次后置处理器
            - 在 populateBean里面遍历所有的Processor
                - 在各自的processor之中完成注入，inject
                    - 在注入阶段获取之前扫描出来的element，然后进行注入，在注入过程中再次调用getBean方法。如果获取不到就去创建，循环上面的逻辑。
        - initializeBean 初始化spring，并且调用第七次和第八次后置处理器，执行bean的初始化方法。
            - applyBeanPostProcessorsBeforeInitialization 在这里面完成了aop。但是对于提前暴露的bean，
        - 然后再次尝试从一级二级缓存中获取bean如果获取不到返回，能获取到会进行循环以来的校验。


spring实例化全过程

- 实例化spirng容器
- 扫描类
- 解析类
- 实例化BeanDefination，并且存到map
- 调用Bean工厂的后置处理器
- 验证（比如类是不是dependsOn的，如果是的话就先不处理，判断是不是单例，单例才会实例化）
- 推断构造方法
- new对象，反射
- 缓存注解信息
- 提前暴露自己的工厂
- 判断是否需要完成属性注入
- 完成属性注入
- 调用生命周期回调方法
- 调用所有的Aware
- 完成代理-aop
- put容器
- 销毁对象


#### 散乱的知识点
1. @PostConstruct 调用完构造函数之后的回调方法。
2. 单例对象spring会实例化，原型对象spring并不会实例化。


##### 如何判断spring是否默认
在spring里面有一个属性isAllowCircularReferenes来判断是否开始循环依赖。默认是true。这个属性在AbstractAutowireCapableBeanFactory里面。


##### spring生命周期回调三种实现
1. 继承Initialization 并实现afterpropertiesSet。
2. xml里面配置init-method。
3. 方法上加@PostConstruct注解。
执行顺序是，3-->1-->2


#### refresh方法堆栈。





