---
tags: [java/中间件]
title: Redis
created: '2020-05-08T06:43:40.776Z'
modified: '2020-05-31T14:25:15.120Z'
---

# Redis
### 什么是nosql
泛指非关系型数据库
关系型数据库并不适合存储文本类型的数据。所以发展出了非关系型数据库。
这些数据结构不不需要固定的模式，无需多余操作。
nosql数据模型更加灵活，无需事先为要存储的数据简历字段，随时可以存储自定义的数据格式。关系型数据库增删字段是非常麻烦的。
Nosql数据库有非常高的读写新能，（reids 1秒写8万，读11万）
高速缓存还是用memchache，功能更丰富，redis。更偏向数据存储，用mongodb。
#### 什么是BSON
JSON的二进制结构，支持内嵌文档对象和数组对象

#### redis是单线程的

#### ACID
- 原子性 Actomic
  要么都成功要么都失败
- 一致性 Consistency
  是指事物只能把数据库从一个有效的状态，转换成另外一个有效的状态，而不是某种临界状态，比如一个事物需要在A表某字段加100，B表某字段减100。如果只完成加100的时候，数据库的状态并不是一个有效的状态。
    事务可以不同程度的一致性：
    - 强一致性：读操作可以立即读到提交的更新操作
    - 弱一致性：提交的更新操作，不一定立即会被读操作读到，此种情况会存在一个不一致窗口，指的是读操作可以读到最新值的一段时间。
    - 最终一致性：是弱一致性的特例。事务更新一份数据，最终一致性保证在没有其他事务更新同样的值的话，最终所有的事务都会读到之前事务更新的最新值。如果没有错误发生，不一致窗口的大小依赖于：通信延迟，系统负载等。     
    其他一致性变体还有
    - 单调一致性：如果一个进程已经读到一个值，那么后续不会读到更早的值。
    - 会话一致性：保证客户端和服务器交互的会话过程中，读操作可以读到更新操作后的最新值。

- 独立性
  事物之间不互相影响，如果一个事物再查询的时候，另一个事物在修改，在修改提交之前，查询是查不到这些数据的。
- 持久性
  一旦事物提价，修改就会被永久保存。
#### CAP
Consistency 一致性
Availability 可用性
Partition tolerance 分区容错性
CAP 只能三选二。一个分布式系统不可能同事很好的满足一致性，可用性，和分区容错性。
CA 传统oracle数据库
CP redis，mongodb，hbase
AP couchDB，dynamoDB。访问数量统计，点赞统计。

#### BASE
为了解决关系型数据库强一致性引起的可用性降低而提出的解决方案
Basically Available 基本可用
Soft state 软状态
Eventually consistent 最终一致性 
通过降低一致性，把强一致性降级为最终一致性，来实现AP。

#### 分布式
不同的机器，部署不同的服务。
集群
不同的机器，部署相同的服务。

#### Redis的三个特点
1. 持久化，可以把数据持久化在硬盘上，重启不丢失。
2. 不仅仅支持简单的key-value类型，还提供list，set，zset，hash等数据结构
3. 支持数据的备份，master-slave模式。

#### 杂项
单进程，通过epoll包装之后，在大批量文件进行多路复用操作。
默认16个库，通过select切换库，select在cluster 模式不能使用
dbsize查看当前数据库的数量
flushdb 清空当前库
flushall 通杀全部库
统一密码管理，16个库都是一个密码
Redis索引是从0开始
默认端口是6379
#### 五大数据类型
String
  - String是二进制安全的，意思是redis的string可以包含任何类型，比如图片序列化后对象。最大是512M
hash
  - 类似java的 map。
list
  - 底层是个链表，可以从头部和尾部插入取出
set
  - string类型的无序集合，通过hashtable实现的
Zset
  - 有序集合

#### 常用命令
1. key
keys * 列出所有key
exists key  key是否存在
expire key 秒 过期时间
ttl key 还有多少秒过期
type key 查看key是什么类型
2. string 单值value
set get del append strlen
Incr decr 加减 incrby decrby 按照一定数量加减 （一定要是数字才能进行加减）
getrange  setrange 获取指定区间的值 0 -1 表示全部。 0 3 表示第0位到第3位。
setex带过期时间的设值  setnx不存在设置，存在返回0，并不覆盖。
3. list 单值多value
lpush rpush lrange
lpop rpop
lindex 按照索引下标获得元素，从上到下。
llen
lrem key n个 要删掉的值。 比如lrem list1 2 3  从list1 里面删掉2个3
ltrim key 开始index 结束index 截取指定范围值并且赋值给key
rpoplpush 源列表 目的列表
lset key index value 在指定下标设置值
linsert key before/after 值1 值2 。在值1之前或者之后插入值2   
>list是一个字符串链表，左右都可以插入，index是从最左边开始的，第一位是0。从头尾操作效率都很高，但是如果对中间进行操作，效率较低。

4. set 单值多value
sadd 添加，重复的值会自动去掉  smembers 列出所有值   sismember判断值是否存在
scard key 获取key中元素个数
srem key value 删除集合中元素
srandmember key n   随机出n个数
spop key  随机出栈
smove key1 key2 key1里面某个值。  作用是讲key1里面的某个值赋值给key2.
sdiff  key1 key2差集  在key1不在key2
sinter key1 key2交集  同时再key1和key2
sunion key1 key2并集  在key1 或者 key2

5. Hash    kv模式不变，但是v是一个键值对。
hset/hget    简单存取
hmset/hmget  多个键值对存取
hgetall  获取所有的键值对
hdel key 键值对的键   删掉key对应map中对应的键值对。
hlen 长度
hexisit key 键值对的键  判断map里面键值对知否存在
hkeys/hvals  所有的key和所有的value
hincrby/hincrbyfloat  key  map的key  值  。   对map的key对应的值进行增加操作
hsetnx   如果不存在就插入，如果存在就返回0

6. Zset 在set基础上增加一个score值。之前set是 k1 v1 v2 v3 现在zset是 k1 score1 v1 score2 v2 score3 v3. 其中score是排序值。
zadd/zrange 写入和读取
zrangebyscore key 开始score 结束score。(表示不包含  limit 开始下标 多少个。 在结果集里面再次截取
zrem key value值。 删除对应value
zcard 统计数量  zcount key score开始 score结束 按条件统计
zrank key 开始下标 结束下标   按照下标获取元素     zscore key value 获取对应的score
zrevrank key 开始下 标结束下标 获取逆序下标对应值
zrevrangebyscore key 结束score 开始scroe   逆序根据score获取值

#### 配置文件介绍
include 可以包含其他配置文件
daemonize yes 守护线程开启，允许后台运行。
pidfile pid文件位置，防止启动过个实例
port 6379 端口配置
Tcp-backlog 511  backlog是链接队列 总和=未完成三次握手+已完成三次握手队列总和。
bind 允许连进来的ip
tcp-keepalive 0   空闲多久断开链接，0表示不会断开
loglevel 日志级别 debug verbose notice warning
logfile 全路径（包含文件名）   在指定路径下记录日志。也可以留空，自动生成。
databases  数据库的数量。单机才有多个库，集群的话就没有库的概念了。
maxmemory redis最大内存
redis可以配置最大内存如果配置了最大内存，就需要配置过期策略。
volatile-lru  lru表示最近最少使用   volatile表示只针对设置有过期时间的key
allkeys-lru
volatile-random random表示随机
allkeys-random
volatile-ttl  ttl表示time to leave。最接近过期时间的key
noeviction 用不移除
save <seconds> <changes>  在多少秒之内有多少改动才会存储到数据库。 save ""或者不设置save 表示禁用 。redis默认配合是1分钟改了1万次  或者 5分钟改了10次 或者 15分钟改了1次。都可以触发。
dir ./ 数据库路径
maxclients  同一时间最大客户端连接数

#### 持久化
- rdb (redis database)
1. 将指定的时间段内，内存的数据快照写入到磁盘。也就是说snapshot快照，它恢复时是将快照文件直接读入内存
redis会单独创建一个子进程(fork)来进行持久化，所以先将数据写入到一个临时文件中，等待持久化过程结束了，再用这个临时文件替换上次持久化好的文件。整个过程，主进程是不尽兴任何IO操作的，保证了高性能。rdb缺点是最后一次持久化后的数据可能丢失。所以对于大规模回复，并且对于数据恢复完整性要求不高，rdb凡是比aof要好。
2. fork的作用是复制一个当前进程一样的进程。新进程的所有数据（变量，环境变量，程序计数器等）都和原进程一致。但是这个新的进程作为原进程的子进程。
3. rdb保存的是dump.rdb文件。
4. 通过save 开启 rdb. dbfilename 指定dump.rdb 文件位置，恢复数据时使用。
5. save命令可以立刻备份
6. Stop-writes-on-bgsave-error 默认yes（默认被配置成yes，不配置的话应该是没有）。 保存出错的时候停止写入
7. rdbcompression 默认yes。 内存快照要不要压缩。
8. rdbchecksum 默认yes。 存储快照后，让redis使用CRC64算法进行数据校验。会增加能耗，希望最大性能，可以关闭。
9. bgsave异步生成快照文件。备份过程中仍然允许主进程写入。save会阻塞。
10. flushall也会生成一个dump.rdb文件，但是这个文件为空。没有什么意义。
- aof  (append only file)
1. 以日志形式来记录每个写操作。将redis执行过的所有写指令记录下来，只允许追加文件，不允许修改文件。redis启动之初会读取该文件重新构建数据。
2. appendonly 默认是no。 这个用来是否开启aof。 会生成一个appendonly.aof文件
3. aof和dump文件可以同时存在。优先读取aof文件。 aof文件出错了，可以用redis-check-aof appendonly.aof来修复。
4. appendfilename 自定义aof文件名。
5. appendfsync 有几种配置。 always 同步支持化，每次变更都记录。 everysec 默认推荐，异步操作。每秒记录一下，有可能丢失一秒之内数据。No 表示redis不主动而是依赖系统操作，linxu操作系统，每30秒进行fsync。
6. rewrite。 aof文件越来越大的时候，新增的重写机制。当aof文件大小超过设定阈值，redis会自动进行压缩，值保留回复数据的最小指令集，可以使用命令bgrewriteaof。
- 官网建议
1. rdb 只用做备份，建议在slave上持久化rdb。15分钟一次就够了，只保留save 900 1这条规则。
2. 如果要用aof，要控制rewrite频率，aof默认的重写大小64太小了，可以设置到5g以上。默认超过原大小100%时重写可以改到适当的数值。
3. 如果不开启aof，仅仅靠master-slave replication实现高可用也是可以的，减少IO同事免去rewrite带来的波动，但是如果master/slave同时挂了，会丢失十几分钟数据，启动脚本也需要比较最新的rdb文件。载如比较新的那个。
#### 部分支持事物 
- redis可以一次执行多个命令，本质是一组命令的集合，一个事物中所有命令都会序列化，按照顺序的串行化执行而不会被其他命令插入，不允许加塞。
- 常用命令
  1. discard 取消事务，放弃执行事物块内的所有命令
  2. Exec 执行所有事物块的命令
  3. multi 标记一个事物的开始
  4. unwatch 取消watch 命令对所有key的监控
  5. watch key [key ...] 监视一个或者多个key，如果事物执行之前这些key被其他命令改动，那么事物被打断。
- 使用方式：multi返回ok，然后后续命令都会返回queued，表示入队。exec执行。
- 中断模式，如果一条命令在入队过程中就报错了，那么整个事物都会失败，如果入队过程不报错，执行报错，那么只有这条命令失败，其他命令不受影响。
- 悲观锁：每次修改数据都认为会冲突，所以每次都把要修改数据锁住。比如行锁，表锁。
- 乐观锁：每次修改数据都认为不会冲突，所以不需要加锁，但是为了防止冲突，修改前需要先判断是否有别人更新这个数据，比如可以使用版本号等级制。乐观锁适用于多度的应用类型，提高吞吐率。乐观锁策略：提交版本必须大于记录当前版本才能执行更新。
- 一旦开始watch之后，到exec执行之前，被watch 的key发生变动，muti的所有命令失败。并且一旦执行了exec，或者unwatch 之前加的监控锁都会被取消。
- 可以用lua脚本实现事务。
#### 发布订阅
有这个功能很少使用
1. subscribe c1 c2 c3 订阅一个或者多个
2. publish c2 hello-world 发布信息到c2, 订阅方会收到。
3. psuscribe new*  通过通配符批量订阅，以new开头的
4. publish new1 hello。发布到new1 会被3匹配上，所以hello可以被收到。
#### 主从复制
- 主机数据更新之后，根据配置和策略，自动同步到备机的master/slave机制，master以写为主，slave以读为主。
- 读写分离，容灾恢复
- 如何配置
  1. 配从不配主
  2. 从库配置：slaveof 主库ip 主库端口。每次与master断开都需要重新连接，除非配置在redis.conf文件
  3. info replication 查看自身主从
  4. 如果master 挂掉了，slave仍旧保持slave，这时候master启动，slave仍旧可以连接上来。没有配置的话，slave挂掉，再启动就跟master断开连接了。
  5. 上一个slaver可以作为下一个slaver的master。
  ```mermaid
  graph RL;
  slave2 --> slave1;
  slave1 --> master;
  ```
  6. 第四点说到master断掉之后，不做操作slave一直是slave。可以通过 slaveof no one 从slaver变成master。
  7. 当slave连接到master之后，会发送一个sync命令，master收到之后会启动存盘进程，同时收集数据修改命令，存盘完成之后将文件发送给slaver，slaver收到之后也存盘然后导入到自身（全量复制），然后在接收master发来的修改命令（增量复制），完成同步。每次slaver重连master都会触发一次全量复制。
  8. 哨兵模式，能够后台监控主机是否故障，如果故障了根据投票数自动将从库转为主库。
    - 新建sentinel.conf文件，文件名不能错。
    - sentinel monitor 被监控主机名字(自己起) ip 端口 数字。最后这个数字是指主机挂掉之后从机获得多少票数可以等级为主机
    - 启动哨兵：Redis-sentinel 配置文件。
    - 哨兵监控主机就可以了。
    - master中断之后，slave自动晋升为master，这时候如果master重新连接上，会自动变成slave。
    - 复制会有一定延迟，网络拥堵的时候会比较严重，slave越多延迟也越严重。

#### redis集群
实现了对redis的水平扩容，起送N个reids节点，将中各数据库分布存储在这N个节点中，每个节点存储总数居的1/N。
Redis 集群通过分区partition来提供一定程度的可用性。即使集群中有一部分节点失效或者无法通讯，集群也可以处理请求命令。
1. redis集群需要通过ruby实现，所以要先安装ruby
2. 拷贝redis-3.2.0.gem到/opt 目录下
3. 在opt目录下执行gem install --local redis-3.2.0.gem
4. 每台机器操作一遍。
5. cluster-enabled yes 打开集群模式
6. cluster-config-file 文件名  设定阶段配置文件名
7. cluster-node-timeout 15000 设定节点失联时间（15000毫秒），超过时间时候自动进行主从切换
8. 启动所有的机器，保证所有的node****.conf能自动生成
9. 在redis的src目录，执行命令
```
/redis-trib.rb create --replicas 1 ip:port [ip:port]
一个集群至少要有3个主节点
--replicas 1 表示为每个主节点至少创建一个从节点。
分配原则上要保证每个主库都是不同的ip，每个主库和从库在不同的ip上。
```
10. 一个redis集群包含16384个槽位 hash slot，采用一致性hash算法把不同的槽位安排到不同的服务器。
11. cluster nodes 查看集群信息
12. reids-cli 连接的时候每次插入和查询都会返回对应的槽位，如果不是客户端链接的服务器，会报错。通过加上-c 开启客户端集群连接模式可以解决
13. 通过{}来定义组的概念，从而使一组key分布在同一台服务器中。
```
set a{user} 1
set aa{user} 11
a和aa属于同一个组，会被发送到同一个服务器中
```
14. 查询槽位的命令
```
sluster keyslot aaa  查询aaa应该被放在哪个槽位
cluster countkeysinslot  123  查询槽位123中对应key的数量
cluster getkeysinslot 123 10 查询槽位123中的10个值
```
15. 主节点下线，从节点可以自动升级为主节点。主服务器再上线，会自动成为从服务器。某一段节点的主从全部挂掉，集群会down机。redis.conf中有个参数 cluster-require-full-coverage 表示所有16384个节点都正常的时候才能对外提供服务。


#### reids 集群优劣
- 优势
  1. 扩容
  2. 分摊压力
  3. 无中心配置相对简单
- 劣势
  1. 多建操作不支持
  2. 多键的redis事务是不支持的。lua脚本不支持。
  3. 集群方案出现的较晚，一般公司采用其他解决方案，迁移起来比较麻烦。  
  


