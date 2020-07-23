---
attachments: [Clipboard_2020-07-14-15-20-36.png, Clipboard_2020-07-14-15-29-07.png, Clipboard_2020-07-14-15-42-21.png]
tags: [java/杂记]
title: spring oracle事物上挂在mysql的事物
created: '2020-07-10T06:41:39.898Z'
modified: '2020-07-14T07:42:23.187Z'
---

# spring oracle事物上挂在mysql的事物

现在我们要做数据库迁移，要做数据双写。本来往oracle里面写的逻辑，要双写，所以需要理清服务的逻辑。

下面分步骤来实现demo

1. 先实现一个service调用两个dao，同时往oracle里面写数据。<font color=#A52A2A>完成</font>

2. 第二次写的时候抛出异常。spring托管事物，所以会回滚。<font color=#A52A2A>完成</font>
    > 需要注意的是，只有runtime exception 和 Error会触发回滚。

3. 双写，往mysql和oracle里面分别写。当代码执行完第一个插入的时候，oracle表里面是没有数据的，但是mysql里面已经有数据了。第二个插入执行之后也一样。再然后抛出异常。oracle数据回滚，而mysql已经插入的数据不会回滚。这里就可以看出，transaction是oracle的，无法控制mysql的回滚。    <font color=#A52A2A>完成</font>

4. 验证下，再配置一套mysql的事物，是否会和oracle冲突。经过验证发现，并不会冲突。两个切面正常运行<font color=#A52A2A>完成</font>

5. 那么现在唯一要做的就是，如何识别特定异常。<font color=#A52A2A>spring只会针对runntime exception进行回滚。所以两个事物切面可以完成我们的需求。</font>


# spring事物是如何从service层传播到dao层的。

1. 首先我们在spring有配置切面，所以我们service的方法会被org.springframework.transaction.interceptor.TransactionAspectSupport的invoke方法拦截。第一次调用没有事物，所以需要开启事物，而在开启事物的过程中，我们会把datasource对饮的connection放到org.springframework.transaction.support.TransactionSynchronizationManager的一个叫做resources的ThreadLocal变量中。这个变量是一个map。当有多个切面的时候会有多个keyvalue。
![](@attachment/Clipboard_2020-07-14-15-20-36.png)
2. 当我们调用 DAO的某个方法时，会去调用sqlSessionTemplate。而这个sqlSessionTemplate封装了一个sqlSessoionProxy的动态代理。这个动态代理之中，会去获取sqlSession。
![](@attachment/Clipboard_2020-07-14-15-29-07.png)
3. 
```
    private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
        Transaction tx = null;

        DefaultSqlSession var8;
        try {
            Environment environment = this.configuration.getEnvironment();
            TransactionFactory transactionFactory = this.getTransactionFactoryFromEnvironment(environment);
            //这里是新建了一个springManagedTransaction，而在springManagedTransaction，当调用openConnecton的时候
            //其实是去TransactionSynchronizationManager的resource之中取到，开起事物的时候存入的connection。由于是以对应的datasource做为key所以不会有取错的情况。
            tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
            Executor executor = this.configuration.newExecutor(tx, execType, autoCommit);
            var8 = new DefaultSqlSession(this.configuration, executor);
        } catch (Exception var12) {
            this.closeTransaction(tx);
            throw ExceptionFactory.wrapException("Error opening session.  Cause: " + var12, var12);
        } finally {
            ErrorContext.instance().reset();
        }

        return var8;
    }
```
下面是springManagedTransaction 的代码
```
    private void openConnection() throws SQLException {
        this.connection = DataSourceUtils.getConnection(this.dataSource);
        this.autoCommit = this.connection.getAutoCommit();
        this.isConnectionTransactional = DataSourceUtils.isConnectionTransactional(this.connection, this.dataSource);
        if (this.logger.isDebugEnabled()) {
            this.logger.debug("JDBC Connection [" + this.connection + "] will" + (this.isConnectionTransactional ? " " : " not ") + "be managed by Spring");
        }

    }
```
下面是DataSourceUtils的代码
```
    public static boolean isConnectionTransactional(Connection con, DataSource dataSource) {
        if (dataSource == null) {
            return false;
        } else {
            ConnectionHolder conHolder = (ConnectionHolder)TransactionSynchronizationManager.getResource(dataSource);
            return conHolder != null && connectionEquals(conHolder, con);
        }
    }    
``` 
下面是何时调用openConnection方法的。
![](@attachment/Clipboard_2020-07-14-15-42-21.png)
