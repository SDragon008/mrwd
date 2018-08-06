[TOC]

# postgresql savepoint

​	在生产环境下，不论是在oracle还是在postgresql,普通都是用的begin commit两个事务标记，一个代表事务开始，一个代表事务提交，或者还有一个是rollback,事务回滚

​	那么这篇文章来介绍一下savepoint在postgresql的作用

## savepoint

​	在官网文档的定义:define a new savepoint within  the current transaction (在当前会话中定义一个新的保存点)。

​	savepoint 在当前事务里建立一个新的保存点。

​	保存点是事务中的一个特殊记号，它允许将那些在它建立后执行的命令全部回滚， 把事务的状态恢复到保存点所在的时刻。

### 语法

```
savepoint savepoint_name
```

### 参数

`savepoint_name`赋予新保存点的名字



### 注意

​	使用[ROLLBACK TO SAVEPOINT](http://www.postgres.cn/docs/9.4/sql-rollback-to.html)回滚到一个保存点。使用[RELEASE SAVEPOINT](http://www.postgres.cn/docs/9.4/sql-release-savepoint.html) 删除一个保存点，但是保留该保存点建立后执行的命令的效果 



### 例子



































## 链接地址

https://www.postgresql.org/docs/9.1/static/sql-savepoint.html

