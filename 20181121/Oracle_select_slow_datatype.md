[TOC]

# Oracle select slow datatype

**文档整理**

ysys

**日期**

2018-11-21

**标签**

oracle,select slow,datatype



## 背景

​	最近一段时间在排查数据库性能时，发现一个sql语句很神奇，有索引却消耗足够多的elapsed time



## 测试

​	sql语句是通过awr获取的

`select * from tal where ybzj = ?`

​	发现表tal中ybzj为主键，有索引，那么为什么不走索引，检查了一下字段类型为varchar,而昨天在修改对应的来源表视图时，源库的YBZJ字段类型是number

​	使用number字段值查询发现sql查询特别慢，执行查询计划，发现不走索引。





​	创建一张表，插入数据和创建索引

```
create table test_del_181121(id varchar2(100),dqsj varchar2(14),sjs varchar2(100),sjz varchar2(100));


insert into test_del_181121
  (ID, dqsj,sjs,sjz)
  select rownum as id,
         to_char(sysdate + rownum / 24 / 3600, 'yyyymmddhh24miss') as dqsj,
         trunc(dbms_random.value(0, 100)) as sjs,
         dbms_random.string('x', 20) sjz
    from dual
  connect by level <= 1000000;
commit;

create index idx_test_del_181121 on test_del_181121(id)
```



```
explain plan for select * from TEST_DEL_181121 t where id =1452;
 SELECT plan_table_output FROM TABLE(DBMS_XPLAN.DISPLAY('PLAN_TABLE'));
 
 Plan hash value: 3365479814
 
-------------------------------------------------------------------------------------
| Id  | Operation         | Name            | Rows  | Bytes | Cost (%CPU)| Time     |
-------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT  |                 |   100 | 16500 |  2478   (1)| 00:00:30 |
|*  1 |  TABLE ACCESS FULL| TEST_DEL_181121 |   100 | 16500 |  2478   (1)| 00:00:30 |
-------------------------------------------------------------------------------------
 
Predicate Information (identified by operation id):
---------------------------------------------------
 
   1 - filter(TO_NUMBER("ID")=1452)
 
Note
-----
   - dynamic sampling used for this statement (level=2)

```

​	仔细查看一下查询条件被解析成`filter(TO_NUMBER("ID")=1452)`,如果创建为to_number索引那么是否可以呢？

```
create index idx_test_del_181121_id_num on test_del_181121(to_number(id));
```

```
explain plan for select * from TEST_DEL_181121 t where id =1452;
 SELECT plan_table_output FROM TABLE(DBMS_XPLAN.DISPLAY('PLAN_TABLE'));

Plan hash value: 3552133597
 
----------------------------------------------------------------------------------------------------------
| Id  | Operation                   | Name                       | Rows  | Bytes | Cost (%CPU)| Time     |
----------------------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT            |                            | 13508 |  2348K|  2736   (1)| 00:00:33 |
|   1 |  TABLE ACCESS BY INDEX ROWID| TEST_DEL_181121            | 13508 |  2348K|  2736   (1)| 00:00:33 |
|*  2 |   INDEX RANGE SCAN          | IDX_TEST_DEL_181121_ID_NUM |  5403 |       |     3   (0)| 00:00:01 |
----------------------------------------------------------------------------------------------------------
 
Predicate Information (identified by operation id):
---------------------------------------------------
 
   2 - access(TO_NUMBER("ID")=1452)
 
Note
-----
   - dynamic sampling used for this statement (level=2)

```

​	请仔细查看这个被解析的条件`access(TO_NUMBER("ID")=1452)`,我们创建了`to_number(id)`索引，所以才走了索引`IDX_TEST_DEL_181121_ID_NUM`

​	

​	通过测试发现尽量各个等值条件尽量字段类型一致，否则可能创建的索引就会失效。

​	sql解析等值是是由字段来适应值的需要，在本次值为number,查询条件将查询的字段强制转为number类型与之对应，中间如果查询的字段内不是数值型，就会报错。

```
drop index idx_test_del_181121_id_num
--update id 变为非数字


SQL>  select * from TEST_DEL_181121 t where id =1452;
select * from TEST_DEL_181121 t where id =1452
ORA-01722: 无效数字

SQL> 
```



​	切记：查询条件要和索引类型一致后才能走索引。





## 链接地址

http://blog.chinaunix.net/uid-21187846-id-3022916.html

https://blog.csdn.net/lizhangyong1989/article/details/45013509