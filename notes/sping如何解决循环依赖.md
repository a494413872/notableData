---
tags: [java/中间件]
title: sping如何解决循环依赖
created: '2020-04-16T06:00:08.389Z'
modified: '2020-05-31T15:22:51.219Z'
---

1. sping如何解决循环依赖
通过三级缓存，三个map。singletonObjects，earlySingletonObjects，singletonFactory。
举个例子，A 依赖B ，B又依赖A。当创建到A的时候，A会调用构造函数创建对象成功，被放在singletonFactory中，然后进行初始化第二部步注入对象，这时候发现依赖B，就去创建B，B创建成功之后，去注入属性，发现依赖A，这时候会先去singletonObjects里面获取，没有再去early里面获取也没有，然后去singletonFactory里面获取，拿到之后赋值，并且把A移动到二级缓存,然后完成注入和初始化，并且把B放入一级缓存，并返回给A，而A就可以根据返回的B对象注入，注入，初始化完成之后也会从二级缓存移除，放入一级缓存。

3. @adptive  SPI

4. TCP和UDP区别。
网络七层结构，每一层是干什么的。



