[TOC]

# pg_truncate_原理分析



## 问题说明

​	在测试在测试shared_buffers中脏数据是有checkpoint刷新进去的，还是由后台程序bgwrite刷新进去的过程中，对一张表反复进行测试，查看文件的大小情况，发现truncate table ,原来的文件路径过段时间就不见了，或者说是不存在了。 

## 测试

测试脚本

```
osdba=# create table t_1(id int);
CREATE TABLE
osdba=# select pg_relation_filepath('t_1');
 pg_relation_filepath 
----------------------
 base/16384/16392
(1 row)

osdba=# \dt+ t_1;
                   List of relations
 Schema | Name | Type  | Owner |  Size   | Description 
--------+------+-------+-------+---------+-------------
 public | t_1  | table | osdba | 0 bytes | 
(1 row)

```

这是查看路径信息

```
osdba@pg41 16384]$ ls -ls 16392* && ls -ls 1639*
0 -rw-------. 1 osdba osdba 0 Feb  4 02:24 16392
0 -rw-------. 1 osdba osdba 0 Feb  4 02:24 16392

```

```
下面开启一个事务A:存入100数据并回滚

osdba=# begin;
BEGIN
osdba=# insert into t_1 (id) select generate_series(1,100);
INSERT 0 100
osdba=# rollback;
ROLLBACK
osdba=# \dt+ t_1;
                  List of relations
 Schema | Name | Type  | Owner | Size  | Description 
--------+------+-------+-------+-------+-------------
 public | t_1  | table | osdba | 16 kB | 
(1 row)


在另外一个窗口发现
[osdba@pg41 16384]$ ls -ls 16392* && ls -ls 1639*
8 -rw-------. 1 osdba osdba 8192 Feb  4 02:27 16392
8 -rw-------. 1 osdba osdba 8192 Feb  4 02:27 16392
```

```
下面开启一个事务B:存入100数据并提交

osdba=# begin;
BEGIN
osdba=# insert into t_1 (id) select generate_series(1,100);
INSERT 0 100
osdba=# commit;
COMMIT
osdba=# \dt t_1;
       List of relations
 Schema | Name | Type  | Owner 
--------+------+-------+-------
 public | t_1  | table | osdba
(1 row)

在另外一个窗口发现
[osdba@pg41 16384]$ ls -ls 16392* && ls -ls 1639*
 8 -rw-------. 1 osdba osdba  8192 Feb  4 02:32 16392
16 -rw-------. 1 osdba osdba 16384 Feb  4 02:31 16392_fsm
 0 -rw-------. 1 osdba osdba     0 Feb  4 02:31 16392_vm
 8 -rw-------. 1 osdba osdba  8192 Feb  4 02:32 16392
16 -rw-------. 1 osdba osdba 16384 Feb  4 02:31 16392_fsm
 0 -rw-------. 1 osdba osdba     0 Feb  4 02:31 16392_vm

```

```
下面开启一次事务C:truncate table 后插入100数据，先不要提交或者回滚
osdba=# begin;
BEGIN
osdba=# truncate table t_1;
TRUNCATE TABLE
osdba=# insert into t_1 select generate_series(1,100);
INSERT 0 100
osdba=# select pg_relation_filepath('t_1');
 pg_relation_filepath 
----------------------
 base/16384/16395
(1 row)

在另外一个窗口发现
[osdba@pg41 16384]$ ls -ls 16392* && ls -ls 1639*
 8 -rw-------. 1 osdba osdba  8192 Feb  4 02:32 16392
16 -rw-------. 1 osdba osdba 16384 Feb  4 02:31 16392_fsm
 0 -rw-------. 1 osdba osdba     0 Feb  4 02:31 16392_vm
 8 -rw-------. 1 osdba osdba  8192 Feb  4 02:32 16392
16 -rw-------. 1 osdba osdba 16384 Feb  4 02:31 16392_fsm
 0 -rw-------. 1 osdba osdba     0 Feb  4 02:31 16392_vm
 8 -rw-------. 1 osdba osdba  8192 Feb  4 02:33 16395


```

通过测试发现truncate table 后，其实并没有真正的删除该文件，而是重新创建一个文件，当新的数据入库时，存入的是新的文件，但是对于上层访问的表来说，表的结构等并没有发生变化。也就是可以理解为什么在postgres事务内，truncate table 这种ddl语句可以支持回滚的原因。

```
事务C回滚后，就会发现事务中的操作都会撤销。


osdba=# rollback;
ROLLBACK

$ ls -ls 16392* && ls -ls 1639*
 8 -rw-------. 1 osdba osdba  8192 Feb  4 02:35 16392
16 -rw-------. 1 osdba osdba 16384 Feb  4 02:31 16392_fsm
 0 -rw-------. 1 osdba osdba     0 Feb  4 02:31 16392_vm
 8 -rw-------. 1 osdba osdba  8192 Feb  4 02:35 16392
16 -rw-------. 1 osdba osdba 16384 Feb  4 02:31 16392_fsm
 0 -rw-------. 1 osdba osdba     0 Feb  4 02:31 16392_vm

而新出现的文件名16395就被彻底删除
```

```
事务C提交后，文件16392存在一段时间后，之后被回收。

osdba=# begin;
BEGIN
osdba=# truncate table t_1;
TRUNCATE TABLE
osdba=# select pg_relation_filepath('t_1');
 pg_relation_filepath 
----------------------
 base/16384/16396
(1 row)

osdba=# insert into t_1 select generate_series(1,100);
INSERT 0 100
osdba=# commit;
COMMIT



[osdba@pg41 16384]$ ls -ls 16392* && ls -ls 1639*
0 -rw-------. 1 osdba osdba 0 Feb  4 02:48 16392
0 -rw-------. 1 osdba osdba    0 Feb  4 02:48 16392
8 -rw-------. 1 osdba osdba 8192 Feb  4 02:48 16396
```

事务C如果提交，就会发现16392其他后缀文件被删除,_fsm的文件记录每个数据块的空闲空间，_vm的文件用来标记数据块中是否存在需要清理的行。 

```
在过一段时间再次执行查看文件名

[osdba@pg41 16384]$ ls -ls 1639*
8 -rw-------. 1 osdba osdba 8192 Feb  4 02:50 16396

只有一个t_1的最新文件存在。
```

