---
attachments: [Clipboard_2020-09-07-11-28-30.png]
tags: [java/杂记]
title: mysql知识碎片
created: '2020-08-26T08:42:49.607Z'
modified: '2020-09-07T03:28:30.209Z'
---

# mysql知识碎片

#### char 和 varchar的区别
char的长度是固定的，varchar的长度是变化的。比如char(10),就算只存1个字符进去，占用的长度也是10，而varchar是变长的，如果根据存储的数据分配长度。相应的，char的处理速度要比varchar快

#### varchar的长度问题
在mysql 5.0.3之前，varchar的长度只有255，现在不再限制了，但是这个表所有列的长度是65535，可以再找个限制只下灵活的分配varchar。

#### int和bigint
在mysql中，int(num),bigint(num) 这个num并不是表示的字段长度，而是当前字段的展示长度，只有当字段补零的时候可以看出区别。比如int(10)和int(11) 里面存储的是666，展示出来分别是0000000666，和00000000666。而int能保存的最大数据只有42亿左右，换算成长度也就是如果长度超过10（包含10），就会存在超限的风险。所以需要换成bigint，而bigint，当长度超过20（包含20），也存在超标风险。

#### mysql的limit
limit可以接受两个参数，如果只传一个参数n,表示前n行。如果有两个参数 limit m,n 则表示从m开始的前n行。要注意的是行数是从0开始的，比如limit 0，5 就是从前五行，如果需要再取五行，需要limit4，5。

#### MySQL DATE_FORMAT() 函数
DATE_FORMAT() 函数用于以不同的格式显示日期/时间数据。
DATE_FORMAT(date,format)
date 参数是合法的日期。format 规定日期/时间的输出格式。
%d	月的天，数值(00-31)
%e	月的天，数值(0-31)
%M	月名
%m	月，数值(00-12)
%Y	年，4 位
%y	年，2 位
%H	小时 (00-23)
%h	小时 (01-12)
%I	小时 (01-12)
%i	分钟，数值(00-59)
%S	秒(00-59)
%s	秒(00-59)

#### Waiting for table metadata lock
出现上面报错的原因是有一条线程获取锁没有释放，所以其他线程就陷入等待。下面是如何分析问题所在。
1. 通过show processlist 查看所有正在运行的线程。如果有waiting的先杀掉，
2. 到information_schema库下面，查看下面这几张表： 
innodb_trx ## 当前运行的所有事务 
innodb_locks ## 当前出现的锁 
innodb_lock_waits ## 锁等待的对应关系
看下是否有存在的事物或者锁。
3. 从performance_schema.events_statements_current表中可以查到失败的语句。因为对于已经失败的语句，本身也会获取锁。同样的获取到线程id然后杀掉。
4. 如果线程本身就已经处在killed状态。


#### mysql的索引长度。
Specified key was too long; max key length is 3072 bytes 
报错原因是添加索引的字段太长。这个报错的节点有两个一个是 767  一个是3072。
mysql默认情况下单个列的索引不能超过767位
可以启用innodb_large_prefix选项，将约束项扩展至3072byte
通过SHOW variables like 'innodb_large_prefix'
如果不是On就通过下面命令打开
SET GLOBAL INNODB_LARGE_PREFIX = ON;
执行完了 之后 还得查看当前的innodb_file_format引擎格式类型是不是BARRACUDA
执行 SHOW variables like 'innodb_file_format'
如果不是的话则需要修改 SET GLOBAL innodb_file_format = BARRACUDA;
如果上面两个都确认了，就修改表结构
alter table 表名 row_format=dynamic;
这样就把index长度扩展至3072.
如果字段长度唱过3072byte，注意单位。而字段又不能缩短，可以通过对部分长度简历索引来解决。
alter table table_neme add index `索引名称` (`列名`(1024))

#### mysql效率
![](@attachment/Clipboard_2020-09-07-11-28-30.png)
