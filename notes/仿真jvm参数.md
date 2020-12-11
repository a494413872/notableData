---
tags: [lvmm]
title: 仿真jvm参数
created: '2020-11-20T04:38:01.773Z'
modified: '2020-11-20T04:45:39.519Z'
---

# 仿真jvm参数

-Dfile.encoding=UTF-8
-Djava.util.logging.config.file=/opt/apache-tomcat_vst_passport/conf/logging.properties
-Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager
-Xmx3172M
-Xms1024M
-Xss256k
>线程堆栈大小
-XX:MaxPermSize=256M
-XX:+HeapDumpOnOutOfMemoryError
-Djava.awt.headless=true
>为了再linux下处理图片
-Dcom.sun.management.jmxremote
-Dcom.sun.management.jmxremote.port=12345
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.ssl=false
-Djava.rmi.server.hostname=10.201.3.67
-Danchor.domainName=vst_passport
-javaagent:/opt/anchor-packages/vst_passport/anchor/anchor-agent-0.0.1-SNAPSHOT.jar
-Danchor.test=test
-Djava.endorsed.dirs=/opt/apache-tomcat_vst_passport/endorsed
>用来覆盖java原生api的
-Dcatalina.base=/opt/apache-tomcat_vst_passport
-Dcatalina.home=/opt/apache-tomcat_vst_passport
-Djava.io.tmpdir=/opt/apache-tomcat_vst_passport/temp 
