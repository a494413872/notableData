---
attachments: [Clipboard_2020-06-23-15-31-56.png, Clipboard_2020-06-23-15-32-08.png, Clipboard_2020-06-23-16-55-23.png]
tags: [java/杂记]
title: mybatis拦截器
created: '2020-06-23T07:29:09.242Z'
modified: '2020-06-23T08:55:27.797Z'
---

# mybatis拦截器

配置一个mybatis拦截器需要3步
1. 自定义一个拦截类，实现org.apache.ibatis.plugin.Interceptor接口。
2. 添加拦截器注解org.apache.ibatis.plugin.Intercepts。
![](@attachment/Clipboard_2020-06-23-15-31-56.png)
3. 配置文件中添加拦截器。
![](@attachment/Clipboard_2020-06-23-15-32-08.png)

在mybatis中可以被拦截的对象有下面四种
1. Excutor 拦截执行器的方法
2. ParameterHandler 拦截参数的处理
3. ResultHandler 拦截结果集的处理
4. StatementHandler 拦截sql语法构建的处理

|拦截的类|拦截方法|
|:---|:---|
|Executor|update, query, flushStatements, commit, rollback,getTransaction, close, isClosed|
|ParameterHandler|getParameterObject, setParameters|
|StatementHandler|prepare, parameterize, batch, update, query|
|ResultSetHandler|handleResultSets, handleOutputParameters|

官方开发插件方式
```
@Intercepts({@Signature(type = Executor.class, method = "query",
        args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class})})
public class TestInterceptor implements Interceptor {
   public Object intercept(Invocation invocation) throws Throwable {
     Object target = invocation.getTarget(); //被代理对象
     Method method = invocation.getMethod(); //代理方法
     Object[] args = invocation.getArgs(); //方法参数
     // do something ...... 方法拦截前执行代码块
     Object result = invocation.proceed();
     // do something .......方法拦截后执行代码块
     return result;
   }
   public Object plugin(Object target) {
     return Plugin.wrap(target, this);
   }
}
```
拦截器里面的方法
```
//总共有三个方法
public interface Interceptor {   
   //处理拦截的对象
   Object intercept(Invocation invocation) throws Throwable;       
   //这个方法是让mybatis判断是否进行拦截，如果需要拦截返回代理，不需要拦截的话返回原对象。每个拦截端详都会调用plugin方法。
   Object plugin(Object target);   
   //支持从配置文件传入属性 
   void setProperties(Properties properties);
}

//由于所有类型的对象都会被拦截所以下面是根据类型决定返回原对象还是代理对象
@Override
public Object plugin(Object target) {
    if (target instanceof StatementHandler) {
        return Plugin.wrap(target, this);
    }
    return target;
}
//拦截器可能通过多层代理，这里是如何获得原始对象。
@SuppressWarnings("unchecked")
public static <T> T realTarget(Object target) {
  if (Proxy.isProxyClass(target.getClass())) {
      MetaObject metaObject = SystemMetaObject.forObject(target);
      return realTarget(metaObject.getValue("h.target"));
  }
  return (T) target;
}
```
![](@attachment/Clipboard_2020-06-23-16-55-23.png)



