[TOC]

# PG 培训 DBA 2



​	第二小节主要讲述的是体系结构，主要需要了解PG的逻辑结构，进程结构，物理结构，以及系统表

之间的关系，系统视图，管理函数等。



## 问题

1. 检查点是否是事务提交点

   





## 逻辑结构

​	pg的逻辑结构

![_](../img_src/0-20180719-1-postgres.png)

​	oracled的逻辑结构

![_](../img_src/0-20180719-2-oracle.png)

​	其实两者的区别首先oracle的实例上并没有一层cluster管理，对于postgres而言，cluster就是当前环境$PGDATA，可以通过查询pg_database,或者在命令行上执行\l，就可以查看当前环境下有多少个数据库，除了部分表或者视图是全局共有的，其他的表对于postgres的另外实例必须通过数据库连接或者其他方式才能访问。

​	第二个不同点：在postgresql中用户独立在逻辑体系之外，而oracle数据将user等同于schema来处理。







## 物理结构



​	在postgres数据库中，表的存储是以文件方式存在的，表空间就是一层目录，表就是一个文件，而在oracle当中就不是这个样子，表空间是有n个物理文件组成，而表存储在表空间，具体表到底在那个文件下，一般而言是无法知道

​	postgresql 表文件默认大小是在创建$PGDATA时决定的，而且当表的大小超过时，会生成同名的文件后添加后缀名。

​	查看编译过程

```
\#./configure --help

with-segsize=SEGSIZE  set table segment size in GB [1]
```

​	除了数据文件外，还有参数文件，wal日志，归档文件，系统日志

​	查看表的路径

```
osdba=# select * from pg_relation_filepath('t_etl_task');
 pg_relation_filepath 
----------------------
 base/16384/24886
(1 row)



tutorial=# select * from pg_relation_filepath('t_3');
             pg_relation_filepath             
----------------------------------------------
 pg_tblspc/16511/PG_9.3_201306121/16490/16512
```

​	默认的数据存放在pg_global,或者pg_default下，如果创建了表空间，不在当前目录下，一般会在pg_tablspc下生成一个软链接，指向这个文件

```
下面来解释一下这个链接地址信息：
pg_tblspc/16511/PG_9.3_201306121/16490/16512
pt_tblspc:软连接
16511：表空间oid
PG_9.3_201306121:postgres版本号
16490:数据库oid
16512:表node
```



## 进程结构



​	在进程机构中最需要了解三点，第一点是启动时的postmaster进程，第二点是wal_write,第三点是bgwrite

![_](../img_src/0-20180719-3-postgres.png)



​	当有应用连接postgres数据库时，postmaster会folk出一个子进程backend process来负责和它进行交互，(想没想过，为什么可以通过杀掉进程就可以结束会话，释放锁或者资源呢，而不会影响整个数据库，就是杀掉了子进程不是吗？)

​	bgwriter:将数据刷新到文件中

​	wal writer:将wal buffer中的日志刷新到xlogs中

​	archiver:将xlogs归档到arch files中

​	pgstat -- 收集统计信息

​	pgarch -- 如果开启了归档, 那么postmaster会fork一个归档进程.

​	checkpointer -- 负责检查点的进程

​	autovacuum lanucher -- 负责回收垃圾数据的进程, 如果开启了autovacuum的话,那么postmaster会fork这个进程.

​	autovacuum worker -- 负责回收垃圾数据的worker进程, 是lanucher进程fork出来的.

 

## 系统表

​	

​	