---
tags: [java/杂记]
title: shiro学习笔记
created: '2020-07-24T08:24:07.197Z'
modified: '2020-07-27T05:52:48.430Z'
---

# shiro学习笔记

HTML <form> 标签的 enctype 属性

enctype 属性规定在发送到服务器之前应该如何对表单数据进行编码。

值|描述
-|-
application/x-www-form-urlencoded	|在发送前编码所有字符（默认）
multipart/form-data	|不对字符编码。在使用包含文件上传控件的表单时，必须使用该值。
text/plain|	空格转换为 "+" 加号，但不对特殊字符编码。


form-data和x-www-form-urlencoded的区别：
1、form-data:

就是http请求中的multipart/form-data,它会将表单的数据处理为一条消息，以标签为单元，用分隔符分开。既可以上传键值对，也可以上传文件。当上传的字段是文件时，会有Content-Type来表名文件类型；content-disposition，用来说明字段的一些信息； 
由于有boundary隔离，所以multipart/form-data既可以上传文件，也可以上传键值对，它采用了键值对的方式，所以可以上传多个文件。

2、x-www-form-urlencoded： 
就是application/x-www-from-urlencoded,会将表单内的数据转换为键值对，比如,name=java&age = 23

设置enctype=”multipart/form-data”时会导致参数绑定失败。 
解决方法： 
需要在mvc配置文件中进行如下配置
```
<!-- 文件上传 -->
<bean id="multipartResolver"
    class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <!-- 设置上传文件的最大尺寸为5MB -->
    <property name="maxUploadSize">
        <value>5242880</value>
    </property>
</bean>
```

