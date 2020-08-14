---
attachments: [Clipboard_2020-08-03-17-00-14.png, Clipboard_2020-08-03-17-16-02.png, Clipboard_2020-08-03-17-21-12.png, Clipboard_2020-08-03-17-21-25.png, Clipboard_2020-08-03-17-26-39.png, Clipboard_2020-08-03-17-34-52.png, Clipboard_2020-08-03-17-41-03.png]
tags: [lvmama]
title: 拆分dao
created: '2020-07-30T09:26:24.670Z'
modified: '2020-08-07T09:15:09.913Z'
---

# 拆分dao

1. 对PassCode.java 进行拆分。在vst_passport项目里面遗留PassCode.java 继承PassCodeBase.java。而PassCodeBase放在vst_dao_proxy项目之中。而在vst_dao_proxy项目之中把对应的PassCode.java的引用全部替换成PassCodeBase.java。在vst_passport项目之中，把所有的PassCodeBase进行强制类型转换。

2. 对比Dao文件夹下面所有的文件，差异文件如下。经过二次确认，所有的差异都不会对逻辑产生影响。这里面的不同包括三个方面。
    1. 第一条所说的替换
    2. 无用导包的删除
    3. Json格式化的时候，直接调用fastJson替代vst_passport分装的方法（方法也是调用fastJson）。

![](@attachment/Clipboard_2020-08-03-17-00-14.png)
比较完成后，删除vst_passport的Dao文件夹下面所有的文件。

3. 比较enums，然后删掉vst_passport重复文件
![](@attachment/Clipboard_2020-08-03-17-16-02.png)

4. map文件没有任何改动，直接删除vst_passport里面的。

5. po文件夹里面有两个改动
    1. PassCode.java拆分
![](@attachment/Clipboard_2020-08-03-17-21-12.png)
    2. 无用导包删除
![](@attachment/Clipboard_2020-08-03-17-21-25.png)
解决方式是，vst_passcode只留下PassCode.java其他的全部删除

6. processor里面只迁移了一个PO,这个可以考虑移动到po文件夹。现在先不动，仅仅是删除vst_passport里面的文件
![](@attachment/Clipboard_2020-08-03-17-26-39.png)

7. vo包里面唯一的不同是删除无用导包不会影响逻辑，所以vst_passport重复文件删掉就可以了。
![](@attachment/Clipboard_2020-08-03-17-41-03.png)

8. 解决1所导致的报错，解决方案强制类型转换。



#### Dao 批量替换
首先替换以Dao;$ 正则
替换Dao.
替换Dao空格
Dao"
Dao=
Dao)


