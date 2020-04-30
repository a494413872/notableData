---
tags: [java/基础知识]
title: Java基础
created: '2019-11-27T02:44:21.273Z'
modified: '2020-04-27T04:45:32.398Z'
---

# Java基础
1. hashmap结构；什么对象能做为key
不可变对象才能作为key，所谓的可变对象，指的是对象在创建之后它的哈希值（由类的hashCode（）方法可以得出哈希值）是可以改变的。

2. hashMap,concurrentHashMap,hashtable比较
Map是定义了存储key-value元素对的接口，具体实现是链表加数组的形式。
HashMap实现将唯一键映射到特定值上，允许一个null的key和多个null的value。
jdk 8 之后，优化了下，链表长度超过8之后，就会转化为红黑树时间复杂度从O(n)变为O(logN)
HashTable同步版的HashMap，通过synchronize锁住整个table，所以速度较慢。同时不允许null的key和null的value
concurrentHashMap，通过segment进行分段锁，其实每一个segement就相当一个小的hashTable。结构是数组-数组-链表


3. 二叉树和红黑树。
二叉树指的是一种数据结构，左子树一定小于等于根节点，右子树一定大于等于根节点。
二叉树会导致长度不均。所以就有了红黑树。
红黑树是一种自平衡二叉树。通过改变颜色以及左旋和右旋调节自身平衡。红黑树有5个规则
1.节点是红色或黑色。
2.根节点是黑色。
3.每个叶子节点都是黑色的空节点（NIL节点）。
4 每个红色节点的两个子节点都是黑色。(从每个叶子到根的所有路径上不能有两个连续的红色节点)
5.从任一节点到其每个叶子的所有路径都包含相同数目的黑色节点。


4. HashMap扩容
当元素个数>数组*负载因子的时候。就会重新扩容。
扩容为原容量两倍，并且重新计算索引转移到新的数组里面。


6. String,StringBuilder,StringBuffer
在这方面运行速度快慢为：StringBuilder > StringBuffer > String
String为字符串常量，而StringBuilder和StringBuffer均为字符串变量，即String对象一旦创建之后该对象是不可更改的，但后两者的对象是变量，是可以更改的
在线程安全上，StringBuilder是线程不安全的，而StringBuffer是线程安全的

5. CAS算法
synchronized加锁属于悲观锁，我们认为一旦并发就一定会发生冲突。
CAS属于乐观锁，我们认为不太可能发生冲突。而一旦冲突我们就重复之前的动作。
比较交换（CAS Compare And Swap）
CAS算法的过程是这样：它包含三个参数 CAS（V,O,N）。
V内存地址存（要更新的变量），O表示预期的值（旧值），N表示新值。仅当V值等于O值时，才会将V的值设置成N，否则什么都不做。
CAS实现类有很多，可以说是支撑起整个concurrency包的实现，在Lock实现中会有CAS改变state变量，在atomic包中的实现类也几乎都是用CAS实现.
CSA的问题
1.ABA问题
如果一个A变为B再变为A，这时候单纯的CAS算法，就会认为这个值没有变化，还是A但是实际上已经改变了。这时候可以加一个版本号。ABA就变成了A1B2A3这样。
2.自旋时间过长
使用CSA就因为这个不会挂起，会自旋（死循环），进行下次尝试。如果不加控制，消耗很大。
3.只能保证一个共享变量的原子操作。
如果是多个共享变量，就不能保证原子性。但是可以通过把多个变量封装为一个对象来解决


6. 对象的深浅复制
浅复制只复制值，如果有对象复制引用。
深复制，连对象也复制。所有对象树都进行浅复制就是深复制了。

7. 多线程：
wait,sleep分别是谁的方法，区别
1.sleep是Thread的静态类方法，谁调用的谁去睡觉，即使在a线程里调用了b的sleep方法，实际上还是a去睡觉，要让b线程睡觉要在b的代码中调用sleep
2.最主要是sleep方法没有释放锁，而wait方法释放了锁，使得其他线程可以使用同步控制块或者方法。Thread.Sleep(0)的作用是“触发操作系统立刻重新进行一次CPU竞争”
3.使用范围：wait，notify和notifyAll只能在同步控制方法或者同步控制块里面使用，而sleep可以在任何地方使用
4.sleep必须捕获异常，而wait，notify和notifyAll不需要捕获异常
countLatch的await方法是否安全，怎么改造
线程池参数，整个流程描述
new ThreadPoolExecutor(1, 3, 5, TimeUnit.SECONDS, blockingQueue, new ThreadPoolExecutor.CallerRunsPolicy())
1, 核心线程数，核心线程除非特别设定poolExecutor.allowCoreThreadTimeOut(true)是不会受到存活时间的影响。
3, 最大线程数, 最大线程-核心线程=存活时间之后需要回收的线程,
5, 线程存活时间, 如果不设定poolExecutor.allowCoreThreadTimeOut(true)，那么五秒后，回收两个线程，剩余一个核心线程不会回收
TimeUnit.SECONDS,时间单位。就是前面数字的单位
blockingQueue, 阻塞队列的长度，是当线程池慢了之后后面可以排队的线程的线程的长度
new ThreadPoolExecutor.CallerRunsPolicy()，当线程池满了，并且队列也满了之后的处理策略.总共有4种，实际使用中，也就这种比较靠谱了。
背后的底层原理aqs，cas
先启动核心线程，然后再放入队列，最后再启动max，最后走策略。


8. 还有Java的锁，内置锁，显示锁，各种容器
及锁优化：锁消除，锁粗化，锁偏向，轻量级锁

9. web方面：
servlet是否线程安全，如何改造
servlet在tomcat之中是单例多线程的。此时如果Servlet中定义了实例变量或静态变量，那么可能会发生线程安全问题

10. session与cookie的区别
由于客户端和服务端之间的会话是无状态的机制，所以Session可以用来关联访问。
当第一次访问js和servlet的时候tomcat会自动生成一个session(当访问html，image的时候不会生成，可以通过request.getSession(true)强制生成)。同时生成sessionid。
tomcat的ManagerBase类提供创建sessionid的方法：随机数+时间+jvmid。
sessionid生成之后，会被放在response的header里面，叫做Set-Cookie用来通知浏览器把sessionId存储下来，在tomcat和jetty服务器中，这个cookie的name叫jsessionid。
当第二次访问的时候，tomcat就会根据cookie中的jsession在服务器端查到对应的session，如果能查找的到，那么两次请求就相当于关联起来了。
如果在java代码中，调用session.invalidate()方法，会注销掉当前session，但是不代表当前就无session了，因为tomcat会马上生成一个新的session并且，当请求返回时通过Set-Cookie方式通知服务器存储新的jseesionid
session是存在于服务器的，cookie是存在于客户机的
11. get和post区别
get和post本质上都是tcp请求。一般情况下，get的参数是放在URL后面的，而post请求是放在request body里面的。当然这并不是绝对的
get请求也可以在request body里面传递参数，有些服务器会解析有些不会。post和get还有一个重大区别，get产生一个tcp数据包，post产生两个tcp数据包。
对于get请求，浏览器会把httpheader和data发过去服务器返回200，而post请求会先返回100continue，然后在发送data过去，服务器才会返回200
get的话，一般认为有长度限制2k，而post数据放在httpbody就无限制了。所以上传文件一般会用post
tcp3次握手，打开连接
（1）第一次握手：建立连接时，客户端A发送SYN包（SYN=j）到服务器B，并进入SYN_SEND状态，等待服务器B确认。
（2）第二次握手：服务器B收到SYN包，必须确认客户A的SYN（ACK=j+1），同时自己也发送一个SYN包（SYN=k），即SYN+ACK包，此时服务器B进入SYN_RECV状态。
（3）第三次握手：客户端A收到服务器B的SYN＋ACK包，向服务器B发送确认包ACK（ACK=k+1），此包发送完毕，客户端A和服务器B进入ESTABLISHED状态，完成三次握手。
tcp4次握手，关闭连接
1.服务器读通道关闭
2.客户机写通道关闭
3.客户机读通道关闭
4.服务器写通道关闭



12. 如何防止表单重复提交
1.通过js限制。
2.在服务器端生成一个唯一的随机标识号，Token(令牌)，同时在当前用户的Session域中保存这个Token。然后将Token发送到客户端的Form表单中，在Form表单中使用隐藏域来存储这个Token，表单提交的时候连同这个Token一起提交到服务器端，然后在服务器端判断客户端提交上来的Token与服务器端生成的Token是否一致，如果不一致，那就是重复提交了，此时服务器端就可以不处理重复提交的表单。如果相同则处理表单提交，处理完后清除当前用户的Session域中存储的标识号。

jvm:
jvm内存模型，
jvm问题工具,jps,jinfo,jmap...

13. 数据库：
最重要的索引及底层实现
索引主要是B-树和B+树。
一个m阶的B树具有如下几个特征：
1.根结点至少有两个子女。
2.每个中间节点都包含k-1个元素和k个孩子，其中 m/2 <= k <= m
3.每一个叶子节点都包含k-1个元素，其中 m/2 <= k <= m
4.所有的叶子结点都位于同一层。
5.每个节点中的元素从小到大排列，节点当中k-1个元素正好是k个孩子包含的元素的值域分划。
比如，一个3阶树，那么k=2，3，这时候每个阶段包含1,2个元素和2,3个孩子
B+树
一个m阶的B+树具有如下几个特征：
1.有k个子树的中间节点包含有k个元素（B树中是k-1个元素），每个元素不保存数据，只用来索引，所有数据都保存在叶子节点。
2.所有的叶子结点中包含了全部元素的信息，及指向含这些元素记录的指针，且叶子结点本身依关键字的大小自小而大顺序链接。
3.所有的中间节点元素都同时存在于子节点，在子节点元素中是最大（或最小）元素。
B+树的优势：
1.单一节点存储更多的元素，使得查询的IO次数更少。
2.所有查询都要查找到叶子节点，查询性能稳定。
3.所有叶子节点形成有序链表，便于范围查询。


索引失效的场景
1.oracle使用函数
2.有null值
3.使用like
4.索引参与运算

最左原则
查看执行计划
及carndiation
然后是锁的类型，行级表级
悲观乐观锁
解释数据库事物及特性
隔离级别
及实现，redo log .undo log
bin log主从复制
mvcc,Next-Key Lock

堆
是通过数组实现的完全二叉树。
满足子节点一定小于等于，或者大于等于父节点。
数组第一个数字是根节点。
如果当前节点下标是n，父节点下标就是(n-1)/2,左孩子节点是2*n+1,右孩子节点是2*n+2

分布式：
问了CAP，跟base
zookeeper满足了CAP的哪些特性，paxos
缓存穿透怎么解决
redis的io模型
如果保证redis高可用
redis是单线程还是多线程
线上cpu占比过高怎么排查

一致性hash
一致性hash算法是一种分布式算法，它可以取代传统的取模操作，解决了取模操作无法应对增删Memcached Server的问题
简单来说，一致性哈希将整个哈希值空间组织成一个虚拟的圆环，如假设某哈希函数H的值空间为0 - (2^32)-1（即哈希值是一个32位无符号整形）
其中有三台服务器，可以那他们的ip或者名称进行hash计算，作为节点。当数据过来的时候用相同的hash算法，算出值，然后沿着环顺时针行走
遇到的第一台服务器就是应该去的节点。

分库分表
垂直切分：对于海量数据的数据库，如果是因为表多而数据多，这时候适合使用垂直切分,即把关系紧密（比如同一模块）的表切分出来放在一个server上
水平切分：表并不多，但每张表的数据非常多.即把表的数据按某种规则（比如按ID散列）切分到多个数据库(server)上
现实中更多是这两种情况混杂在一起，这时候需要根据实际情况做出选择，也可能会综合使用垂直与水平切分，从而将原有数据库切分成类似矩阵一样可以无限扩充的数据库(server)阵列
分库分表之后，就需要分布式事物等等一系列特殊处理

spring:
ioc,aop原理
IOC就是典型的工厂模式，通过sessionfactory去注入实例
依赖注入（Dependecy Injection）和控制反转（Inversion of Control）是同一个概念。
aop
面向切面编程（AOP）完善spring的依赖注入（DI），面向切面编程在spring中主要表现为两个方面 
1.面向切面编程提供声明式事务管理 
2.spring支持用户自定义的切面 
自定义切面流程
1.写一个切面类在类上用@Aspect @Component（如果是注解方式，这个Componet必须要，如果是xml配置这个bean则不需要）
2.定一个一个切点@Pointcut(value="execution(* com.lvmama.scenic.songjian.SongjianTestDao.*(..))")，意思是这个dao下面所有参数类型，所有返回类型的方法。第一个*是返回类型，*（..）是所有参数类型。
3.@Around(value="pointCut()") 或者@before @after。这两种都无法终止程序执行，而第一种可以。 pointCut是切点注解的方法名。
ioc初始化流程
1.)Resource定位；指对BeanDefinition的资源定位过程。通俗地讲，就是找到定义Javabean信息的XML文件，并将其封装成Resource对象。
2.)BeanDefinition的载入；把用户定义好的Javabean表示为IoC容器内部的数据结构，这个容器内部的数据结构就是BeanDefinition。
3.)向IoC容器注册这些BeanDefinition,就是注册到一个hashmap中。
这里完成的仅仅是BeanDefinition的载入和注册，Javabean之间的依赖关系并不会在初始化的时候完成
springmvc的流程
springboot,spring cloud相关组件
项目


什么是happens-before原则。
happens-before原则，是java内存模型重排序时候的依据。保证如果A bappens before B,那么A所有操作在B可见。
下面是这个原则对应的规则。
程序次序规则：一段代码在单线程中执行的结果是有序的。注意是执行结果，因为虚拟机、处理器会对指令进行重排序（重排序后面会详细介绍）。虽然重排序了，但是并不会影响程序的执行结果，所以程序最终执行的结果与顺序执行的结果是一致的。故而这个规则只对单线程有效，在多线程环境下无法保证正确性。
锁定规则：这个规则比较好理解，无论是在单线程环境还是多线程环境，一个锁处于被锁定状态，那么必须先执行unlock操作后面才能进行lock操作。
volatile变量规则：这是一条比较重要的规则，它标志着volatile保证了线程可见性。通俗点讲就是如果一个线程先去写一个volatile变量，然后一个线程去读这个变量，那么这个写操作一定是happens-before读操作的。
传递规则：提现了happens-before原则具有传递性，即A happens-before B , B happens-before C，那么A happens-before C
线程启动规则：假定线程A在执行过程中，通过执行ThreadB.start()来启动线程B，那么线程A对共享变量的修改在接下来线程B开始执行后确保对线程B可见。
线程终结规则：假定线程A在执行的过程中，通过制定ThreadB.join()等待线程B终止，那么线程B在终止之前对共享变量的修改在线程A等待返回后可见。

synchronize
可以分为锁住类对象和锁住实例对象两种。
锁住类对象的时候，包括同步块锁类，或者静态同步方法。这个时候就算new多个实例，也是共享一个锁。
锁住实例对象的时候，包括同步块锁this，或者同步非静态方法。或者同步块锁类成员变量。


偏向锁/轻量级锁/重量级锁
这三种锁是指锁的状态，并且是针对Synchronized。在Java 5通过引入锁升级的机制来实现高效Synchronized。这三种锁的状态是通过对象监视器在对象头中的字段来表明的。
偏向锁是指一段同步代码一直被一个线程所访问，那么该线程会自动获取锁。降低获取锁的代价。
轻量级锁是指当锁是偏向锁的时候，被另一个线程所访问，偏向锁就会升级为轻量级锁，其他线程会通过自旋的形式尝试获取锁，不会阻塞，提高性能。
重量级锁是指当锁为轻量级锁的时候，另一个线程虽然是自旋，但自旋不会一直持续下去，当自旋一定次数的时候，还没有获取到锁，就会进入阻塞，该锁膨胀为重量级锁。重量级锁会让其他申请的线程进入阻塞，性能降低。

可重入锁
指的是某个线程获取锁之后，可以再次获取锁。synchronized关键字和ReentrantLock都可以重入。
ReentrantLock能实现synchronized的所有功能并且更灵活
lockInterruptibly。当获取锁的时候，锁被另外一个线程占有直接中断当前线程
tryLock(可选时间)，尝试立即，或者在规定时间内获取锁。
公平锁非公平锁
new ReentrantLock的时候参数false非公平，参数true或者无参是公平

volatile
由于happens-before原则，A线程对volatile的写happens-before B线程对volatile的读。
具体流程是，当A对volatile写的时候，会更新主存，同时其他线程工作内存会里面的变量会失效。
volatile并不能保证原子性。比如volatile++由于这个操作不是原子性的，所以就算是volatile也会出现并发问题。


final
一旦赋值，就不能被改变。
类变量：必须要在静态初始化块中指定初始值或者声明该类变量时指定初始值，而且只能在这两个地方之一进行指定；
实例变量：必要要在非静态初始化块，声明该实例变量或者在构造器中指定初始值，而且只能在这三个地方进行指定。
当final修饰基本数据类型变量时，不能对基本数据类型变量重新赋值，因此基本数据类型变量不能被改变。而对于引用类型变量而言，它仅仅保存的是一个引用，final只保证这个引用类型变量所引用的地址不会发生改变，即一直引用这个对象，但这个对象属性是可以改变的

as-if-serial语义
As-if-serial语义的意思是，所有的动作(Action)都可以为了优化而被重排序，但是必须保证它们重排序后的结果和程序代码本身的应有结果是一致的。Java编译器、运行时和处理器都会保证单线程下的as-if-serial语义。
比如，为了保证这一语义，重排序不会发生在有数据依赖的操作之中

内存屏障
内存屏障（Memory Barrier，或有时叫做内存栅栏，Memory Fence）是一种CPU指令，用于控制特定条件下的重排序和内存可见性问题。Java编译器也会根据内存屏障的规则禁止重排序。




原子性，有序性和可见性
原子性是指一个操作是不可中断的，要么全部执行成功要么全部执行失败，有着“同生共死”的感觉
然的有序性可以总结为：如果在本线程内观察，所有的操作都是有序的；如果在一个线程观察另一个线程，所有的操作都是无序的
volatile包含禁止指令重排序的语义，其具有有序性
synchronized语义就要求线程在访问读写共享变量时只能“串行”执行，因此synchronized具有有序性

可见性
synchronized具有可见性。
volatile具有可见性

ASQ
AbstractQueuedSynchronizer（AQS）抽象队列同步器（简称同步器）   
AQS本身是一个抽象类，采用的是模板方法设计模式，将一些方法开放给子类重写。所以Lock会调用同步器，而同步器又去掉用子类的具体实现。
就比如，Reentrantlock里面，会有一个内部类NonfairSync继承AQS，然后会重写tryAqire方法。当调用lock()方法时，就会先去调用AQS.aquire()

Reentrantlock获取锁的过程。
线程获取锁失败，线程被封装成Node进行入队操作，核心方法在于addWaiter()和enq()，同时enq()完成对同步队列的头结点初始化工作以及CAS操作失败的重试;
线程获取锁是一个自旋的过程，当且仅当 当前节点的前驱节点是头结点并且成功获得同步状态时，节点出队即该节点引用的线程获得锁，否则，当不满足条件时就会调用LookSupport.park()方法使得线程阻塞；
释放锁的时候会唤醒后继节点；
lockInterruptibly() 可中断锁
和lock的区别是，当线程被强制中断后，可以直接返回异常。而lock必须等到线程获取锁，然后才能返回异常
tryLock()
尝试获取锁，如果能获取则返回成功否则失败。
tryLock（time，timeUnit）带时间获取锁。
尝试在time内获取所，如果超时未获取所则返回false，在规定时间内可以获取锁，则返回true

ReentrantReadWriteLock 读写锁
锁降级，指的是获取写锁之后再次获取读锁，释放的时候要释放写锁。
有一个int类型的state，高16位表示读锁状态，低16位表示写锁状态。

独占锁和共享锁。
其实区别就在于，独占锁的status=1表示空闲，0表示占用。而共享锁status>1所以可以获取多次。
C.U.T包下的ReeReentrantLock和ReentrantReadWriteLock一个是独享一个是共享锁。
独享锁：该锁每一次只能被一个线程所持有。
共享锁：该锁可被多个线程共有，典型的就是ReentrantReadWriteLock里的读锁，它的读锁是可以被共享的，但是它的写锁确每次只能被独占。

公平锁和非公平锁
公平锁就遵循先进先出，所有线程排队获取锁。
非公平锁可以允许强占，强占成功则获取锁，失败则排队




ReentrantReadWriteLock(读写锁)。
读写锁允许同一时刻被多个读线程访问，但是在写线程访问时，所有的读线程和其他的写线程都会被阻塞。

wait()和notify()和notify()是指的对象上的行为。
在某个对象上wait()的话，要在相同的对象上notify才能唤醒。

Condition和lock类似都可以中断程序，并且Condition必须关联到lock上。
而且一个Lock可以new多个condition，还有Condition可以设置超时时间。
具体实现也是内部维护一个状态，并且维护一个带头结点的单项链表来放置等待队列。
当当前线程调用condition.await()方法后，会使得当前线程释放lock然后加入到等待队列中，直至被signal/signalAll后会使得当前线程从等待队列中移至到同步队列中去，直到获得了lock后才会从await方法返回，或者在等待时被中断会做中断处理
调用condition的signal或者signalAll方法可以将等待队列中等待时间最长的节点移动到同步队列中
signal方法首先会检测当前线程是否已经获取lock，如果没有获取lock会直接抛出异常，如果获取的话再得到等待队列的头指针引用的节点，之后的操作的doSignal方法也是基于该节点
signal是把等待队列里面第一个节点，放入同步队列。signalAll是把等待队列所有的节点放入同步队列。
同步队列是没有头结点的链表

condition是通过调用lockSupport的park和unpark方法进行控制的。
park会使线程进入阻塞
unpark会唤醒线程。
synchronzed致使线程阻塞，线程会进入到BLOCKED状态，而调用LockSupprt方法阻塞线程会致使线程进入到WAITING状态

LockSupport 有park和unpark(thread)来阻塞和唤醒指定线程。

线程的几种状态
NEW 初始化，还没调用start
RUNABLE 运行状态，就绪和正在运行是这个
BLOCKED 阻塞于锁
WAITING 等待状态，需要其他线程唤醒或者中断
TIME_WAITING 超时等待状态，指定时间自行返回，切换为RUNABLE状态
TERMINATED 终止状态，线程执行完毕

线程基本操作。
interrupt 中断
interrupt相当与一个标志位
一个线程A在执行的时候，另外一个线程B可以通过interrupt来改变A线程的isInterrupt状态。这时候A线程可以根据这个标记位进行一些操作。
如果抛出出interruptException的时候，会把标志位清除。isInterrupt变为false
Join 协作
A线程会等待B线程终止后才会继续执行A
Sleep 休眠
是Thread的静态方法，调用会导致线程休眠，不会释放锁。
yield 退位
当yield被调用时，会立即释放时间片，但是自己也会加入抢夺队列。有可能会再次执行。
并且，只有同样优先级的线程才会有资格想到yield的释放的资源。




Lock
lock是一个接口，提供了和synchronize一样的锁功能，但是更加强大。
Lock lock = new ReentrantLock();
lock.lock();
try{
	.......
}finally{
	lock.unlock();
}
synchronized同步块执行完成或者遇到异常是锁会自动释放，而lock必须调用unlock()方法释放锁，因此在finally块中释放锁。

并发容器
ConcurrentHashMap
利用分段锁提高了并发度，ConcurrentHashMap的主干是个Segment数组，Segment继承了ReentrantLock，所以它就是一种可重入锁（ReentrantLock)
Segment里面是HashEntry组成的数组
hashMap是链表加数组。
ConcurrentHashMap是数组链表加数组

CopyOnWriteArrayList
通过读写分离的思想，达到线程安全。
get的话，就和一般的list的get是一样的。
而调用add方法会先获取锁，创建新数组，并复制数据，最终新数组插入数据，然后旧引用指向新数组。
所以add能保证线程安全。

ConcurrentLinkedQueue
ArrayList不是线程安全的，Vector是线程安全。而保障Vector线程安全的方式，是非常粗暴的在方法上用synchronized独占锁，将多线程执行变成串行化
ConcurrentLinkedQueue
通过持有头尾指针进行管理队列
offer和add都可以往队列里面放，队列满了，offer会异常。add返回false
poll和remove 移除第一个元素，poll队列空返回null，remove返回true和false   
peek和element 查询头元素，队列为空，element异常，peek返回null


ThreadLocal
Thread类里面持有一个ThreadLocal.ThreadLocalMap的引用。
其中key是当前的ThreadLocal实例对象，value是要存入的对象。
比如，有10个线程同时给一个ThreadLocal变量赋值，这时候实际上是分别存放到，10个线程自己的threadLocalMap里面。
如果同一个线程定义多个ThreadLocal变量，这时候是会在map里面有多个key的。

threadlocal会有内存泄露的风险。
根据上面的描述我们知道，ThreadLocalMap的key就是ThreadLocal实例。举个例子：
ThreadLocal<String> la= new ThreadLocal();
la.set("hello world")。
这时候ThreadLocalMap里面就会put一个（la,"hello world"）.
而这个la这时候已经被声明为弱引用了。之所以声明为弱引用，是为了不影响ThreadLocal的回收。如果声明为强引用，则即使la=null，
我们new出来的ThreadLocal对象也不会被回收的。
由于key是弱引用，所以当la=null的时候，new ThreadLocal()这个对象会被回收。这时候会产生一个为key为null的键值对。value永远不会被访问，又永远不会被释放。造成内存泄露。
当然如果线程整个执行完成了，那么资源会被回收的。但是如果线程执行时间很长，或者是被线程池托管循环利用。那么就有问题了。
而事实上get，set和remove都会清楚这种entry。所以为了保证


强引用、软引用、弱引用、虚引用的概念
强引用（StrongReference）
强引用就是指在程序代码之中普遍存在的，最基础的赋值操作就是强引用，强引用宁愿出现OOM也不会回收得。
软引用 （SoftReference）
Java中用java.lang.ref.SoftReference类来表示，当内存不足的时候会回收，一般不会。具体用法。
SoftReference<String> sr = new SoftReference<String>(new String("hello"));
System.out.println(sr.get());
弱引用（weakReference）
弱引用是用WeakReference 来表示。无论是否内存充足，只要gc都会被回收。
虚引用（PhantomReference）
软引用和弱引用可以单独使用，虚引用不能单独使用，需要和引用队列一起使用，当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会把这个虚引用加入到与之 关联的引用队列中。
虚引用的作用是就跟踪对象被垃圾回收的状态，程序可以通过检测与虚引用关联的虚引用队列是否已经包含了指定的虚引用，从而了解虚引用的对象是否即将被回收。
ReferenceQueue<String> queue = new ReferenceQueue<String>();
PhantomReference<String> pr = new PhantomReference<String>(new String("hello"), queue);
System.out.println(pr.get());


ScheduledThreadPoolExecutor，大小是无界的
可以延迟和周期性的执行线程。
核心是DelayedWorkQueue和ScheduledFutureTask
核心方法是
提交一个runnable，延迟执行
public ScheduledFuture<?> schedule(Runnable command,long delay, TimeUnit unit);
提交一个callable，延迟执行
public <V> ScheduledFuture<V> schedule(Callable<V> callable,long delay, TimeUnit unit);
提交runnable 按固定频率执行
public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,long initialDelay,long period,TimeUnit unit);
提交runnable，按固定时间间隔执行
public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,long initialDelay,long delay,TimeUnit unit);
DelayedWorkQueue 是基于堆的数据结构，是用数组实现的。初始大小是16.
ScheduledFutureTask 是可周期执行的任务

FutureTask
有三个状态，  未启动，                 已启动，                        已完成 分别调用get和cancel
get           阻塞	                   阻塞                            返回结果或者异常
cancel        无效果                   cancel(true)，中断，false不中断 

Atomic 原子类型工具主要用到的有
AtomicBoolean 
AtomicInteger
AtomicLong
addAndGet(int delta) ：以原子方式将输入的数值与实例中原本的值相加，并返回最后的结果；
incrementAndGet() ：以原子的方式将实例中的原值进行加1操作，并返回最终相加后的结果；
getAndSet(int newValue)：将实例中的值更新为新值，并返回旧值；
getAndIncrement()：以原子的方式将实例中的原值加1，返回的是自增前的旧值；
atomic包下提供能原子更新数组中元素的类有：
AtomicIntegerArray：原子更新整型数组中的元素；
AtomicLongArray：原子更新长整型数组中的元素；
AtomicReferenceArray：原子更新引用类型数组中的元素

CountDownLatch 并发计数器
通过await()只有计数器等于0的时候才会继续往下执行。
countDown()方法用来是计数器减一
还以带上时间参数await(long timeout, TimeUnit unit)。如果到了时间就算计数器不为零也继续执行。

CyclicBarrier 循环栅栏，比CountDownLach更加强大
具体用用法是
初始化的时候，可以初始化拦截次数。
CyclicBarrier(int parties) await的次数等于parties 就继续执行线程。
CyclicBarrier(int parties, Runnable barrierAction) await的次数等于parties，限制性barrierAction再执行线程。

Semaphore 信号量
通过aquire方法获取许可，使用完通过release释放许可。拿到许可才可以继续执行，否则阻塞。

Exchanger 交换器
//当一个线程执行该方法的时候，会等待另一个线程也执行该方法，因此两个线程就都达到了同步点
//将数据交换给另一个线程，同时返回获取的数据
V exchange(V x) throws InterruptedException
//同上一个方法功能基本一样，只不过这个方法同步等待的时候，增加了超时时间
V exchange(V x, long timeout, TimeUnit unit)throws InterruptedException, TimeoutException 

消息系统的好处
系统解耦
异步调用
流量削峰
如何保证数据传输的安全性。
1.针对消费者接受消息后，在调用第三方服务出错的情况。有两种方案
	1.接受消息后，消息落地，然后通过job和主线程分别处理消息，对处理完的消息进行标记。
	2.改些ack机制，手动传确认信息。
2.Mq自身down机。
	可以采用消息持持久化。activemq本身也支持。
3.生产者发送消息丢失
	生产者保证收到mq的确认信息就可以了。mq如果采用的是落地消息的话，就不会丢失。
	
	
我们系统的架构，订单系统直接调用我们的对接系统，然后在对接系统中发送消息，自己接收消息。
接受到消息后进行落地，然后处理，处理完成之后该状态，对于状态没有改掉的，会有补偿措施。
服务启动起来，就有会根据供应商类型启动不同数量的线程，对于没有特殊配置的，会放在common线程里面的linkedBlockQueue。然后通过take阻塞住线程执行。
如果有消息进来就会执行。同时common会同时启动多个线程比如10个，然后根据线程数量对订单号取模，把消息发送不同的线程池里面。
系统里面会有监控线程启动着，如果监控到某个申码的执行时间超时，就会挂掉当前线程然后重新启动线程，这样当前导致线程挂掉的那条消息就相当于暂时不作处理。等待补偿。
后来系统进行了多线程改造，在线程里面维护一个flag，可以通过页面改变这个flag并设置数量。然后线程下次执行的时候，根据这个flag和线程数量会启动对应线程数量的线程池来解决单个线程里面队列数据过多的问题



JVM
jvm参数分为3类。
1.-开头的，标准参数，所有的jvm都要实现，而且向后兼容
2.-X开头的，非标准参数，默认jvm实现，但是不保证所有jvm都实现。
3.-XX 非Statble参数，各个版本的jvm实现会有所不同，换个版本的jvm可能就不支持了。
GC 算法包括
标记-清除算法：先标记所有要回收的对象，然后统一回收
标记-压缩算法：先标记所有要回收的对象，然后存活对象移动到一端
复制算法：可用内存按容量划分为两个区域。每次只使用其中一块，当这一块用完了，所有存活对象复制到另一块上。
分代收集算法：java堆分为新生代和老年代，根据各个年太的特点采用适当的收集算法。
在标记过程中会产生swt。
收集器类型
串行收集器，程序，GC，程序。GC的时候程序暂停
并行收集器，和上面不同是GC阶段多线程。
CMS收集器，程序--》初始标记--》并发标记--》重新标记--》并发清理--》并发重置。只有初始标记，和并发标记的时候是swt的。
cms是老年代算法，不会手机年轻代
使用G1垃圾回收方式有三种young、mixed、Full，young和mixed是G1回收器，Full是G1失败的情况下采用serial回收； 
young会回收全部的eden和survivor区域； 
mixed会回收全部的eden和survivor区域，同时会回收部分old 或Humongous区域； 
Full gc则是G1回收失败的情况下，使用serial回收全部的区域，这时候我们会发现FullGc的次数会+1；

类加载
1.加载
类加载的第一个阶段
根据类的全名来获取类的二进制字节流          
将字节流代表的静态存储结构转化为方法区的运行时数据结构
在java堆中生成一个java.lang.Class的对应对象
总的来说就是找到并加载二进制数据
2.链接
验证：验证类的二进制流格式是否正确
准备：为类的静态变量分配内存，并将其初始化为默认值。
			比如static int i=1；实际上是在准备阶段实际上是i会被置为0，而在初始化<clinit>中才会设置为1
			但是static final int i =1；在准备阶段就会被赋值1
解析：符号引用替换为直接引用
3.初始化
执行类构造器<clinit>
Static变量 赋值语句
Static{}语句

init是instance实例构造器，对非静态变量解析初始化
clinit是class类构造器对静态变量，静态代码块进行初始化

NoClassDefFoundError是一个错误(Error)
连接时从内存找不到需要的class就出现NoClassDefFoundError
在运行时，某个引用类找不着
而ClassNotFoundException是一个异常
加载时从外存储器找不到需要的class就出现ClassNotFoundException
当前类找不着

类加载器从上到下是
Bootstrap ClassLoader 启动加载类
Extension ClassLoader 扩展加载类
Application ClassLoader 系统加载类
自下而上检查是否加载，自上而下尝试加载

自定义类加载器的步骤
（1）继承ClassLoader    
（2）重写findClass（）方法   
（3）调用defineClass（）方法

hotspot的OPP——KLASS模型
对象头包括
）mark world 主要存储对象运行信息，包括所状态，GC年龄分代，线程ID，时间戳，hashcode等等
）元数据指针，指向方法区，instanceKlass实例。就是类的实例
实例数据。

Klass是c++中class的对等体。一般在加载阶段生成。


设计模式：
adatper适配器模式
	分为两种
	类适配器是采用继承要适配接口，实现原本接口
	对象适配器是使用引用，转发调用。
桥接模式
  将实现与抽象放在两个不同的层次类中，使两个层次可以独立改变。
  比较经典的飞机和的厂商的问题，把飞机类型抽象成一个层次，具体的实现层就是不同的厂商。这样既可以自由拼接。
建造者模式
	主要适用于创建复杂对象。
命令模式：
   命令进行抽象，对客户端和执行者之间进行解耦。客户端不需要了解执行细节。比如电脑开机
组合模式
	 模糊部分与整体的差异，让主程序忽略整体和部分的差异。比如树，文件系统
装饰者模式	    
	 引用一个对象，对方法进行装饰
外观模式
	 一个功能调用很多接口，封装成统一的接口
工厂模式

简单工厂：
  定义一个创建对象的类，由这个类来封装创建对象的行为。
工厂方法：
  不再由工厂来决定产生对象，固定工厂产生固定对象。
抽象工厂：
  抽象工厂和工厂不同的地方就在于，当对象更多，层次更多的时候，对工厂再进行一层次的抽象。
  比如除了product还需要生成Goods。而goods正好也和product一样可以分为GoodsOne和GoodsTwo。 	 
迭代子模式
  由统一接口对外提供访问自己元素方式。Iterator
模板模式：
	父类直接调用本身的抽象接口，然后子类具体实现这些个接口.
	或者父类本身提供默认实现，但是子类可以覆盖。
察者模式：
	当某一个对象变更需要通知其他对象的时候适用	  
代理模式
	代理模式一般应用于，代理类不能被直接访问的情况，比如权限控制等等。		
单例模式：
	确保一个类最多有一个实例，并提供一个全局的访问接口。
	有构造函数私有化，这样别的地方就没办法new对象。只能自己new自己的对象。
	new对象的方法getInstance，和对象本身都是静态的，这样可以保证全局唯一性。
	new对象之前先判断。
	懒汉：最初对象为null，第一次getInstance的时候才会去new.
	饿汉：最初就初始化一个对象	
策略模式：                                                               	
  抽象行为为接口，同一个接口有不同行为（也就是实现类）。而这个接口被另外一个Context类所引用。 主程序只需要调用Context即可。
状态模式
	状态模式和策略模式看起来基本是一样的。
	策略模式，更偏重于算法的可替换性，所有算法是等价的。由主程序选择使用哪个算法，但是一般不会同时应用多重算法。
	而状态模式，更偏重于状态的转换对对象造成的影响，而具体状态的变化一般会封装在context里面。对主程序无影响。
	
	
什么是微服务
就是将单个应用程序座位一套小型服务开发的方法，每个小型服务都可以独立运行，而互相之间通过轻量机制进行通信。
有点：独立部署，扩展灵活，资源有效隔离。
缺点：需要考虑分布式事物和异步补偿机制

分布式事物
事物四个特性，原子性，一致性，隔离性，持久性
单数据源的一致性，可以通过单机事物解决，多数据源的话，就只能靠分布式事物了
分布式事物，用于在分布式系统中保证不同节点之间数据一致性
分布式事物的几种解决方案
1.两阶段提交或者3阶段提交（2pc ）
Oracle Tuxedo 提出XA协议包括，两阶段2pc 和3阶段两种实现
两阶段2pc，指的就是做为事物协调者的节点，先发送prepare请求，执行节点收到之后各自执行数据库操作但是不commit，写入undolog和redolog。第二阶段协调节点发送commit请求。如果第一阶段失败，第二阶段就发送abort请求
两阶段问题，1.性能问题。2.协调单点故障问题 3.丢失消息导致的不一致问题
XA三阶段，引入超时机制，一单协调者挂掉，自动commit，解决单点故障，但是性能和不一致问题没有解决
2.事物补偿机制TTC
针对每一个操作，都要注册一个与之相对应的确认和补偿（撤销）操作
先try，如果成功就调用commit如果失败就调用cancel。这三种操作都是自己定义的。
3.本地消息表
消息的生产方需要额外建一张消息表，记录消息的发送状态。消息表和业务数据要在一个事物里面提交。然后消息经过MQ发给消费方。如果发送失败，重试发送
消费方处理这个消息，如果成功，就更新消息的状态，如果失败则发送失败消息给生产方，进行回滚等操作。
最后还要有一个补偿机制，定期扫描本地消息表，对未处理的消息进行处理。
这种方案遵循BASE理论，采用最终一致性。算是比较经典的解决方案

4.MQ事物，例如阿里的rocketMQ
第一阶段prepare消息，会拿到消息的地址
第二阶段执行本地事物
第三阶段通过第一阶段拿到的地址去访问消息并修改状态。
也就是说，生产者要发送两次消息，一次发送确认消息。这样就是MQ本身自动实现本地消息表的功能。但是这个
只能保证发送端的事物，和消息的一致性。如果消费端出了问题就需要人工介入了。

sagas事物模型
就是有一个工作流引擎，负责协调多个事物，如果整个流程正常结束就算完成，如果有一步异常则依次回滚。

Memcached 分布式锁
Memcached 可以使用 add 命令，该命令只有KEY不存在时，才进行添加，或者不会处理。Memcached 所有命令都是原子性的，并发下add 同一个KEY ，只会一个会成功。
利用这个原理，可以先定义一个 锁 LockKEY ，add 成功的认为是得到锁。并且设置[过期超时] 时间，保证宕机后，也不会死锁。
在具体操作完后，判断是否此次操作已超时。如果超时则不删除锁，如果不超时则删除锁


Redist实现分布式锁  
通过setnx，expire，del来实现锁锁，这时候有问题:
设置setnx是set if not exist，当key不存在成功，当key存在则失败。但是由于它没有过期时间。所以需要通过expire来更新过期时间。当执行成功之后通过del来删除key
由于setnx和expire是两个操作，就可能出现线程1，setnx成功，然后其他线程setnx失败，但是所有线程的expire都会成功，那么过期时间会一直被刷新。
所以用2.6.12以后，可以用set来增加过期参数。
jedisCluster.set(key, value, "NX", "EX", expireSeconds); 就是把setnx和expire给做成原子性的操作。解决了上面的问题。
但是，当执行成功del的时候也会有问题。比如线程1的锁，过期了自动释放，线程2获取了。然后线程1执行完成去删除了线程2对应的锁。
所以这种情况下，可以通过添加线程独立的标识，来识别自己的锁。以保证自己只操作自己的锁。

通过ZK实现分布式锁
通过zk的顺序临时节点。可以实现获取锁已经排队的功能
先创建临时节点，同事获取所有兄弟节点，如果自己是第一位的，则认为获取锁，如果不是第一位的则认为获取锁失败进入等待队列，通过watcher监听第一个节点。    


IO分为
IO 同步阻塞IO
NIO 同步非阻塞IO
AIO 真正的异步IO
Mina，Netty NIO的框架
阻塞和非阻塞，针对的是主程序，如果主程序执行的时候，等待IO事件为阻塞，主程序轮询查询IO为非阻塞。
同步和异步 同步和异步指的是IO事件执行完会主动通知为异步，不会主动通知为同步。


NIO
NIO核心有三大部分，Channel，Buffer，Selector
传统IO基于字节流和字符流进行操作，而NIO基于Channel和Buffer(缓冲区)进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中
Selector(选择区)用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个线程可以监听多个数据通道。

NIO文件流使用步骤
读取数据
1.先从流中获得channel
2.准备buffer
3.从channel读取数据到buffer。
写数据。
1.先从流中获取channel
2.准备buffer
3.数据写入buffer
4.buffer写入channel
比较重要的方法buffer.flip() 这个方法的作用是把先把position赋值给limit，然后position重置到0，比如先往buffer里面写数据，指针会随着数据写入
而往后移动，想要读取的话，就需要先flip。而buffer.clear()方法，会把position为0，limit重置为capacit这样就认为没有数据了。
另外，还有一个mark方法，是标记一个特定的position，可以通过reset回复到这个position。

NIO之SocketChannel使用。
客户端
1.打开socket channel
2.设置为非阻塞
3.3.打开selector，把channel注册到选择器上（选择器只能监听非阻塞channel所以文件流是不可以的）。
4.channel上链接网络地址端口号。
5.准备buffer
6.调取select方法，获取就绪channel数量，然后通过selector.selectedKeys()获取就绪的通道。
7.从selectionKey面获取channel，如果是读事件，读取数据到buffer。如果是写事件，就写入数据到buffer然后再写入buffer到channel
服务端
1.打开ServerSocketChannel
2.设置为非阻塞
3.获取server socket
4.在ServerSocket上绑定网络连接端口号。
5.打开选择器
6.注册channel到选择器上。
7.准备buffer
8.调取selector的select方法，select()阻塞到至少有一个通道在你注册的事件上就绪了，获取就绪channel数量，然后通过selector.selectedKeys()获取就绪的通道。
9.判断通道类型，如果是acceptable，就接受serverSocketChannel，然后通过accept监听连进来SocketChannel等待新的连接连进来。如果是其他的，就接受socketchannel然后跟客户端一样读写数据。

缓冲区复用的时候，要特别注意的是，clear并不可以清空数据，如果调用clear，再写入一些数据，然后再调用Buffer.array()，仍然可以取的buffer除了你写入的还有上次遗留的数据。
如果clear之后又重写数据了，正确的做法是先flip，然后把buffer的数据，放入一个长度为limit的byte数组里面


另外注意区别，对于服务端来说，selectedKeys是ServerSocketChannel，需要先getSocketChannel。而client，selectedkeys就是socket channel。


netty
netty是nio的框架
使用方式。
服务端
新建客户端clientBootstrap
新建线程池boss和worker
设置socket工厂NioClientSocketChannelFactory，传入boss和worker。
设置管道工厂，实现channelPipeFactory，重写getPieLine方法。pipeline方法设置编码解码方式和处理类。
处理类可以自己选择实现的方法，比如messageRecived，channelopen，。。close等等。
最后bootstrap通过connect和bin实现连接。然后拿到一个channelFuture
由于netty是异步的，所以刚开始拿到future的时候，有可能channel处各种状态。可以awart(),也可以addlistner（推荐），也可以轮询。
客户端




AIO
AsynchronousSocketChannel，和AsynchronousServerSocketChannel。
真正和NIO不同的是，channel的read和write，返回的是futuretask，需要调用futuretask的get方法获取执行结果。


缓存穿透
访问不存在的数据，由于不存在不会放入缓存，导致直接可以访问数据库。可被利用进行恶意攻击
解决方案，不存在也放入缓存，只是时间设置的短一点。
缓存雪崩
缓存同时大批量失效，例如根据产品ID进行缓存，可能有大批量id同时失效。导致数据库冲高。
尽量把缓存的失效时间分散，可以根据商品Id本身特性进行区分，比如不同品类的缓存时间不一样，同一个品类可以通过取模区分等等。
缓存击穿
一个key非常热点，等它一失效，会有大量访问同时进来。
解决方案，可以考虑缓存永不过期，定时去更新缓存数据


数据源切换。
1.自定义Dynamicdatasource类，继承自abstractRoutingDataSource。
1.1 创建工具类，DataSourceKeyHolder，保存一个ThreadLocal的string变量。存储datasource的key。
2.xml文件中定义多个datasource
3.把多个datasource注入到dynamicDataSource类的targetDataSource，map里面。
	当spring初始化的时候，会调用abs...的对应after properties类，这时候会把targetDataSource给put到resolveDatasource这个map里面。
4.在abstractRoutingDataSource里面实现determineCurrentLookUpkey，自定义选择key的逻辑。一般是从ThreadLocal里面取出key。
6.自定义注解类@ReadOnlyDataSource。
5.在xml里面配置一个Aop，拦截所有serviceImpl类。在处理逻辑里面对拦截到的方法进行判断，是否有
  自定义注解，并对ThreadLocal进行赋值。
6.这样在impl代码执行的时候，当调用到Dao的时候，会去建立connection，建立connection的过程就会调用dynamicDataSource的determineCurrentLookupKey。

数据库连接池
我们系统用的是dbcp
dbcp是apache的一个开源项目。getConnection的时候会个PoolingDataBase
如果有两个数据源的话，实际上是会创建两个数据库连接池。 

负载均衡算法。
轮询
比率
优先权  
最小连接数
最快响应时间
Hash算法（Redis）就用的hash算法。


springcloud
使用方式：
--负载均衡--ribbon
pom文件添加ribbon依赖。
启动一个ribbon项目（基本和普通的client项目一样），注册一个Bean，RestTemplate到本地IOC然后添加@loadBalance注解。
后面就可以通过RestTemplate调用Rest请求，就会实现负载均衡。
--负载均衡--feign
Feign 采用的是基于接口的注解，Feign 整合了ribbon，具有负载均衡的能力，整合了Hystrix，具有熔断的能力
使用方式：
pom添加feign依赖。
application文件里面添加@EnableFeignClients开启feign
新建feign接口，添加@FeignClient(value = "服务名")表明链接那个哪个服务器。然后书写方法定义。
这个feign接口就可以作为service被使用了。
--断路器：
断路器是指，当服务断开的情况下，调用方不会无限制等待，而是会字段断开然后返回指定报错信息。
--ribbon中使用hystrix
pom文件添加依赖
application文件里面@EnableHystrix开启hystrix。
在service上添加@HystrixCommand(fallbackMethod = "方法名")指定熔断方法    
--Feign自带断路器使用方式：
yml文件添加feign.hystrix.enabled=true
直接在@FeignClient注解里面加上fallback=service实现类的类名。
service实现类要实现@feiClient注解的接口
--路由---zuul，有点像nginx。还可以添加过滤器。
zuul使用方法
pom文件天际zuul
application开启@EnableZuulProxy
yml文件配置zuul：routes：。。。。。。
过滤器直接继承ZuulFilter就可以了。
--spring cloud config 分布式配置中心--
bootstrap.properties 配置 spring cloud config对应git项目地址。




java8 新特性
1.lamdba表达式
lamdba表达式允许把函数作为参数传进方法里面。
lamdba表达语法
(parameters) -> expression   这里 expression是一行代码，多行就要大括号了。parameters可为空。
(parameters) ->{ statements; }
2.函数式接口
一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口。
需要添加@FunctionInterface注解。
如果一个接口是函数式接口，那么就可以用lamdba表达式来表示该接口的一个实现
3.方法引用
方法引用主要是对lamdba的进一步简化，当你的lamdba表达式只调用一个已存在的方法的时候，就可以用方法引用简写。
比如，当你用lamdba表达式 Arrays.sort(stringsArray,(s1,s2)->s1.compareToIgnoreCase(s2));
Sort的第二个参数，Comparable是一个函数式接口，这时候可以使用如上的表达式。
又因为实际只是调用String的一个方法。那么就可以使用方法引用。
引用静态方法	ContainingClass::staticMethodName
引用某个对象的实例方法	containingObject::instanceMethodName
引用某个类型的任意对象的实例方法	ContainingType::methodName
引用构造方法	ClassName::new
4.默认方法
就是接口可以有具体的实现方法了。只需要加default就成为默认方法。
5.stream
就是可以把集合，数组，IO channel等等转化为stream方便操作。
6.Optional 类
Optional是一个容器，可以保存类型T的值。



@Resource的作用相当于@Autowired，只不过@Autowired按byType自动注入，而@Resource默认按 byName自动注入


java 动态代理
动态代理，简单的说就是jdk动态的帮我们生成代理类。而我们可以直接调用代理对象的代理方法。
InvocationHandler接口是给动态代理类实现的，负责处理被代理对象的操作的
Proxy是用来创建动态代理类实例对象的，因为只有得到了这个对象我们才能调用那些需要代理的方法。
java动态代理和静态代理的区别是
动态代理不需要编写实现Pet接口具体的PetProxy类，而是通过实现InvocationHandler的invode来代用被代理对象。
然后通过Proxy.newProxyInstance来创建被代理对象。
Proxy.newProxyInstance三个参数1.对象的类加载器，2.动态代理类需要实现的接口，3.实现InvocationHandler的对象，提供Invoke方法。
invoke三个参数：1,.proxy就是代理对象，newProxyInstance方法的返回对象 2.method：调用的方法 3.args: 方法中的参数

Daemon线程/守护线程
线程分为用户线程和守护线程，守护线程指序运行的时候在后台提供一种通用服务的线程，比如垃圾回收线程就是一个很称职的守护者。当所有的非守护线程结束时，程序也就终止了，同时会杀死进程中的所有守护线程。将线程转换为守护线程可以通过调用Thread对象的setDaemon(true)方法来实现。thread.setDaemon(true)必须在thread.start()之前设置，否则会跑出一个IllegalThreadStateException异常。在Daemon线程中产生的新线程也是Daemon的。守护线程应该永远不去访问固有资源，如文件、数据库，因为它会在任何时候甚至在一个操作的中间发生中断
                                                                                                                                                                   
