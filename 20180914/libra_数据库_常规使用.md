[TOC]

# libra 数据库 常规使用



## 对于libra疑问

1、对于高并发事务的支持力度

2、对于update的优化

3、gds是否集成了update

4、表空间pg_default，pg_global是否有优化，倾斜

5、分布键倾斜的程度影响查询

6、多表关联（分布键，普通字段) 如何聚合

7、查询select * from table limit 100，是否是查询出来所有数据再limit 100

8、负载均衡(ip) √

9、数据库



## 访问

### data Studio

​	目前实施团队主要使用data Studio客户端来访问华为Libra数据库，当前将主要的操作流程记录下来。



### 其他访问

​	暂无



## 数据库操作

### 创建数据库

​	使用管理员用户登陆到libra环境中

```
# create database tutorial 
```

```
# - drop database tutorial	
```

​	数据库默认字符集可能需要手动设置

### 创建用户

```
# create role
```

```
# - drop role
```



### 创建模式

```
# create schema 
```

```
# drop schema
```

### 创建表空间(但是不建议)

```
# create tablespace 
```

```
# - drop tablespace 
```

​	当删除tablespace时，会发现原始目录还会存在，华为libra(2017) 



### 创建表

```
# create table
```

### 创建函数

```
# create function
```



### 创建存储过程

```
# create procedure 
```



### 授权



```
# grant 
```

```
# - revoke
```



### GDS搭建并使用





### 数据插入模版



### 数据更新模版



### 注意事项



#### vacuum  analyze table

​	数据库运行一段时间后，经常会遇到查询较慢，统计信息不实的情况，这个时候可能是因为数据所在的表经常被更新，目前libra基于postgresql的底层数据库所采用的mvcc方式(更新时重新插入一条新数据)，这样很容易引发垃圾数据过多冗余，需要对表进行一些瘦身操作

​	目前对于表的“瘦身”有两种操作，分别是 vacuum full table和vacuum table，不建议使用vacuum full table操作，建议使用vacuum table

```
# vacuum tablename
```

```
# ananlyze tablename
```

​	vacuum table并不会使用排它锁，不会影响数据插入更新查询操作



​	

#### pg_stat_activity

#### 撤销删除进程

#### 并发数

#### ip限制









