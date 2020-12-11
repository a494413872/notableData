---
attachments: [Clipboard_2020-06-16-17-38-40.png, Clipboard_2020-06-16-19-40-58.png, Clipboard_2020-06-16-19-44-52.png, Clipboard_2020-06-16-20-06-12.png, Clipboard_2020-06-16-20-37-01.png, Clipboard_2020-06-16-20-40-48.png, Clipboard_2020-06-16-20-54-33.png, Clipboard_2020-06-16-21-03-17.png, Clipboard_2020-06-17-11-02-16.png, Clipboard_2020-06-17-11-27-09.png, Clipboard_2020-06-17-14-33-05.png, Clipboard_2020-06-17-14-50-14.png, Clipboard_2020-06-17-16-23-15.png, Clipboard_2020-06-17-17-43-08.png]
tags: [java/jvm]
title: jvm调优
created: '2020-06-16T09:37:08.546Z'
modified: '2020-12-10T09:05:20.754Z'
---

# jvm调优
首先可以看出来，java的内存占比很高。
![](@attachment/Clipboard_2020-06-16-17-38-40.png)
由于我并没有指定垃圾处理器，所以看下当前系统默认的垃圾处理器,很奇怪的是并没有指定默认的垃圾处理器，这个应该也属于正常范围吧。
![](@attachment/Clipboard_2020-06-16-19-40-58.png)
现在已知我的虚拟机，RAM有512，那么我把大小设置为128为什么占用内存还如此之多？
![](@attachment/Clipboard_2020-06-16-19-44-52.png)
因为是经过大众验证的开源项目，所以不怀疑有代码上的问题，这次尝试的目的是调研清楚为什么内存占用如此严重。
由于我安装的是openjdk所以没有开发工具需要我自己安装。
首先,yum list --showduplicate | grep java-1.8 | grep devel 查看 有哪些可安装的jdk1.8 开发工具 包,如下:
![](@attachment/Clipboard_2020-06-16-20-06-12.png)
安装之。
好像装错了，系统是242的我装成了252的，希望不影响使用。
好像出问题了，命令没办法使用，没关系我们找到程序所在地直接调用。
先jps一下找到线程id 776。（跟上面图片不一样了，因为我重启了。）
![](@attachment/Clipboard_2020-06-16-20-37-01.png)
然后通过jinfo查看java程序的扩展参数。
![](@attachment/Clipboard_2020-06-16-20-40-48.png)
先来分析一波。首先可以看出初始化内存和堆内存确实是128M。其中新生代是42.625老年代是85.375。这里正好验证了新生代和老年代是1：2的比例。同时开启了普通指针压缩和类指针压缩。但是有一个很奇怪的问题没有垃圾回收器啊!现在还无法确定是真的没有还是不显示总之我已经有了隐隐的怀疑。
下面我们通过jstat查看下gc详情。
jstat命令可以查看堆内存各部分的使用量以及加载类的数量。
我们现在先看下GC情况
![](@attachment/Clipboard_2020-06-16-20-54-33.png)
附上一个字典
```
S0C：第一个幸存区的大小      4352
S1C：第二个幸存区的大小      4352
S0U：第一个幸存区的使用大小   2948
S1U：第二个幸存区的使用大小   0
EC：伊甸园区的大小           34944
EU：伊甸园区的使用大小        13939.7
OC：老年代大小               87424
OU：老年代使用大小           50487
MC：方法区大小               86780
MU：方法区使用大小           81286
CCSC:压缩类空间大小          11028
CCSU:压缩类空间使用大小      10265.6
YGC：年轻代垃圾回收次数      112
YGCT：年轻代垃圾回收消耗时间  1.686
FGC：老年代垃圾回收次数       3
FGCT：老年代垃圾回收消耗时间   0.336
GCT：垃圾回收消耗总时间       2.022
```
我连续执行了几次jstat，发现只是eden区有些微的上升。那就暂时先不管。等明天再看看效果。
![](@attachment/Clipboard_2020-06-16-21-03-17 (2).png)
第二天的效果
![](@attachment/Clipboard_2020-06-17-11-02-16.png)
访问了几次页面后的效果
![](@attachment/Clipboard_2020-06-17-11-27-09.png)
直接使用
jstat -gccause 776 1000 进行监视
![](@attachment/Clipboard_2020-06-17-14-33-05.png)
字典
```
s0、s1:表示两个survior区域的使用百分比
e：eden区域使用百分比
o：老年代使用百分比
m：metaspace（元数据空间）使用百分比
ccs:压缩使用比例
ygc：新生代gc次数
ygct：新生代gc累计总时间
fgc：full gc次数
fgct：full gc累计总时间
gct：gc累计总时间
lgcc: 上次垃圾回收的原因
gcc: 本次GC回收的原因
```
可以看出，young GC进行了一次，但是老年代占比并没有变，数据直接进入了元空间。感觉这个也有问题啊。
然后通过jmap查看内存占比。jmap -heap 776。 命令的意思是打印堆信息。
![](@attachment/Clipboard_2020-06-17-14-50-14.png)
查看线程数
![](@attachment/Clipboard_2020-06-17-16-23-15.png)
一直访问一直到触发full gc
![](@attachment/Clipboard_2020-06-17-17-43-08.png)
最终解决方案就是调整新生代的大小。


拉取线程dump
jstack -l pid > 1.txt

