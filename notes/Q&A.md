---
tags: [java/基础知识]
title: Q&A
created: '2020-04-22T05:53:56.719Z'
modified: '2020-04-22T06:33:27.502Z'
---

# Q&A
#### 什么是数据库事物？
数据库事物是指，做为一个逻辑单元的一系列操作，要么完全成功，要么完全失败。
#### 数据库事物的四大特性ACID
| -|- |- |- |
|---|---|---|---|
| A|Atomicity|原子性|所有操作不可分割要么都成功要么都失败|
| C|Consistency|一致性|事物执行前和执行后，数据库都应该处在一致性|
| I|isolation|隔离性|并发环境之中，事务应该互不影响|
| D|durability|持久性|事物一旦提交，那么对应的数据应该永久性的保存到数据库中。|

#### 数据库事物的隔离级别
| -|- |- |- |- |
|---|---|---|---|---|
Read_uncommited|读未提交|允许另外一个事物看到当前事物未提交数据|基本不会用|脏读
Read_commited|读取提交内容|一个事物只能看到其他事物已提交数据|大部分在用(除mysql)|不可重复读
repeatable_read|可重复读|一个事物多个实例并发读取会看到同样数据|mysql默认|幻读
Serializable|可串行化|每个读的数据上加上共享锁，使之串行|超时和锁竞争




