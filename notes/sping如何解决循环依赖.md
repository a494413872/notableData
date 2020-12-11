---
tags: [java/中间件]
title: sping如何解决循环依赖
created: '2020-04-16T06:00:08.389Z'
modified: '2020-12-02T09:17:05.469Z'
---

1. sping如何解决循环依赖

举个例子，A 依赖B ，B又依赖A。
- 三级缓存，三个map。singletonObjects一级，earlySingletonObjects二级，singletonFactory三级。
- 当A创建的时候，会先去一级缓存里面获取，这时候缓存里面是没有的所以取不到，然后判断下A是否存在于singletonsCurrentlyInCreation这个map中。这时候并不在，然后直接返回空。
- 下面需要去创建bean，在创建之前在singletonsCurrentlyInCreation中把A的beanName放进去。然后开始创建Bean，先创建一个Instance，如果A支持循环依赖（allowCircularReferences默认true），把A的factory放入三级缓存中，然后进行依赖注入，发现依赖B，然后去创建B。
- 后面的流程跟上面A的流程一样。
- 一直到发现A又依赖了B，这时候会再次去一级缓存里面获取，获取不到，去二级缓存里面获取，也获取不到，继续去三级缓存里面获取，可以获取到A对应的factory，调用这个factory的getObject方法，获取经过spring封装的可以提前暴露的对象A，同时把A从三级缓存移动到二级缓存，这时候A并不是一个完整的Bean里面有部分内容没有被注入。
- 这个时候A可以被注入给B，B经过注入完成初始化
- B完成初始化之后，把B从二级缓存移动到一级缓存。
- 然后再把B注入到A里面。这样A的初始化也完成了，而这时候B由于已经持有A的引用所以当A完整之后B里面的A自然也就完整了。这样就解决了循环依赖的问题。


3. @adptive  SPI

4. TCP和UDP区别。
网络七层结构，每一层是干什么的。



