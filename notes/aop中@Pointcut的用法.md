---
tags: [java/中间件]
title: aop中@Pointcut的用法
created: '2020-11-17T06:06:51.085Z'
modified: '2020-11-17T06:58:26.636Z'
---

# aop中@Pointcut的用法
格式如下
execution(modifies-partten? ret-type-pattern declaring-type-pattern? name-parttern(param-pattern) throws-pattern?)
>带有问号的表示是可选项
- modifier-pattern? 修饰符匹配。例如public表示所有的公有方法
- ret-type-pattern 返回值匹配，*表示任何返回值，可以填写全路径类名
- declaring-type-pattern? 类路径匹配
- name-pattern(param-pattern) 方法名匹配，(param-patter) 参数匹配，(..)表示所有的参数()表示一个参数，(,String)代表第一个参数是任何值，第二个是String
- throws-pattern? 异常类型匹配

#### 通配符
- * 匹配任何字符
- .. 匹配任何数量字符的重复，如类型模式匹配任意数量子包，参数模式中匹配任何数量参数。
- +: 匹配指定类型的子型，仅能做为后缀放在类型模式后边
