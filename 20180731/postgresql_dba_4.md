[TOC]

# postgresql dba 4

​	

​	本章节主要讲述的是索引的作用，索引的种类，模糊匹配，近似度匹配



​	

## 索引

### 使用索引的好处



1. 利用索引进行排序减少cpu开销

2. 加速带条件的查询，删除，更新

3. 加速JOIN操作

4. 加速外键约束更新和删除操作

5. 加速唯一值约束，排他约束



####  利用索引进行排序减少cpu开销

```
tutorial=# create table test1(id int,info text,crt_time timestamp(0));
CREATE TABLE
tutorial=# insert into test1 select generate_series(1,10000),md5(random()::text),now();
INSERT 0 10000

```

利用索引进行排序减小cpu开销

1、查询条件是索引列

```
tutorial=# create index idx_test1_id on test1(id);
CREATE INDEX
tutorial=# explain analyze select * from test1 where id < 100 order by id;
                                                       QUERY PLAN             
                                           
------------------------------------------------------------------------------
-------------------------------------------
 Index Scan using idx_test1_id on test1  (cost=0.29..10.04 rows=100 width=45) 
(actual time=0.005..0.018 rows=99 loops=1)
   Index Cond: (id < 100)
 Planning time: 0.315 ms
 Execution time: 0.040 ms
(4 rows)
```

查询条件没有索引列排序

```
tutorial=# drop index idx_test1_id;
DROP INDEX
tutorial=# explain analyze select * from test1 where id < 100 order by id;
                                                QUERY PLAN                                                 
-----------------------------------------------------------------------------------------------------------
 Sort  (cost=222.32..222.57 rows=100 width=45) (actual time=0.717..0.722 rows=99 loops=1)
   Sort Key: id
   Sort Method: quicksort  Memory: 32kB
   ->  Seq Scan on test1  (cost=0.00..219.00 rows=100 width=45) (actual time=0.008..0.704 rows=99 loops=1)
         Filter: (id < 100)
         Rows Removed by Filter: 9901
 Planning time: 0.043 ms
 Execution time: 0.741 ms
(8 rows)
```

2、查询条件不是索引列

​	如果查询条件没有索引，默认会扫描全表

```
tutorial=# explain analyze select * from test1 where info = 'ad18651ba470f129723cdd1260a243ed' order by id;
                                               QUERY PLAN                                               
--------------------------------------------------------------------------------------------------------
 Sort  (cost=219.01..219.01 rows=1 width=45) (actual time=1.601..1.601 rows=1 loops=1)
   Sort Key: id
   Sort Method: quicksort  Memory: 25kB
   ->  Seq Scan on test1  (cost=0.00..219.00 rows=1 width=45) (actual time=0.009..1.594 rows=1 loops=1)
         Filter: (info = 'ad18651ba470f129723cdd1260a243ed'::text)
         Rows Removed by Filter: 9999
 Planning time: 0.062 ms
 Execution time: 1.621 ms
(8 rows)
```

​	将全表扫描关闭,利用索引来查询

```
tutorial=# set enable_seqscan = off;
SET

tutorial=# explain analyze select * from test1 where info = 'ad18651ba470f129723cdd1260a243ed' order by id;
                                                      QUERY PLAN                                                       
-----------------------------------------------------------------------------------------------------------------------
 Index Scan using idx_test1_id on test1  (cost=0.29..392.29 rows=1 width=45) (actual time=0.024..2.068 rows=1 loops=1)
   Filter: (info = 'ad18651ba470f129723cdd1260a243ed'::text)
   Rows Removed by Filter: 9999
 Planning time: 0.091 ms
 Execution time: 2.088 ms
(5 rows)

```

这个只是例子, 不一定适合实际应用场景. 



#### 加速带条件的查询，删除，更新

```
tutorial=# explain analyze select * from test1 where id = 1;
                                                     QUERY PLAN                                                      
---------------------------------------------------------------------------------------------------------------------
 Index Scan using idx_test1_id on test1  (cost=0.29..8.30 rows=1 width=45) (actual time=0.011..0.012 rows=1 loops=1)
   Index Cond: (id = 1)
 Planning time: 0.067 ms
 Execution time: 0.028 ms
(4 rows)

```

将索引扫描关闭

```
tutorial=# set enable_indexscan = off;
SET
tutorial=# set enable_bitmapscan = off;
SET
tutorial=# explain analyze select * from test1 where id = 1;
                                            QUERY PLAN                                            
--------------------------------------------------------------------------------------------------
 Seq Scan on test1  (cost=0.00..219.00 rows=1 width=45) (actual time=0.011..0.799 rows=1 loops=1)
   Filter: (id = 1)
   Rows Removed by Filter: 9999
 Planning time: 0.091 ms
 Execution time: 0.827 ms
(5 rows)

```



#### 加速JOIN操作



```
tutorial=# create table test2(id int,info text,crt_time timestamp(0));
CREATE TABLE
tutorial=# insert into test2 select generate_series(1,10000),md5(random()::text),now();
INSERT 0 10000

```

test2 没有索引

```
tutorial=#  explain analyze select t1.*,t2.* from test1 t1 join test2 t2 on (t1.id=t2.id and t2.id=1); 
                                                          QUERY PLAN                                                          
------------------------------------------------------------------------------------------------------------------------------
 Nested Loop  (cost=0.29..227.31 rows=1 width=90) (actual time=0.019..0.984 rows=1 loops=1)
   ->  Index Scan using idx_test1_id on test1 t1  (cost=0.29..8.30 rows=1 width=45) (actual time=0.007..0.010 rows=1 loops=1)
         Index Cond: (id = 1)
   ->  Seq Scan on test2 t2  (cost=0.00..219.00 rows=1 width=45) (actual time=0.008..0.968 rows=1 loops=1)
         Filter: (id = 1)
         Rows Removed by Filter: 9999
 Planning time: 0.428 ms
 Execution time: 1.042 ms
(8 rows)

```

为test2创建索引

```
tutorial=# create index idx_test2_id on test2(id);
CREATE INDEX
tutorial=#  explain analyze select t1.*,t2.* from test1 t1 join test2 t2 on (t1.id=t2.id and t2.id=1); 
                                                          QUERY PLAN                                                          
------------------------------------------------------------------------------------------------------------------------------
 Nested Loop  (cost=0.57..16.62 rows=1 width=90) (actual time=0.030..0.032 rows=1 loops=1)
   ->  Index Scan using idx_test1_id on test1 t1  (cost=0.29..8.30 rows=1 width=45) (actual time=0.007..0.007 rows=1 loops=1)
         Index Cond: (id = 1)
   ->  Index Scan using idx_test2_id on test2 t2  (cost=0.29..8.30 rows=1 width=45) (actual time=0.021..0.022 rows=1 loops=1)
         Index Cond: (id = 1)
 Planning time: 0.285 ms
 Execution time: 0.058 ms
(7 rows)

```

merge join 也能用到索引

```
tutorial=#  explain analyze select t1.*,t2.* from test1 t1 join test2 t2 on (t1.id=t2.id ); 
                                                               QUERY PLAN                                                               
----------------------------------------------------------------------------------------------------------------------------------------
 Merge Join  (cost=0.57..884.57 rows=10000 width=90) (actual time=0.011..13.670 rows=10000 loops=1)
   Merge Cond: (t1.id = t2.id)
   ->  Index Scan using idx_test1_id on test1 t1  (cost=0.29..367.29 rows=10000 width=45) (actual time=0.004..3.351 rows=10000 loops=1)
   ->  Index Scan using idx_test2_id on test2 t2  (cost=0.29..367.29 rows=10000 width=45) (actual time=0.003..3.804 rows=10000 loops=1)
 Planning time: 0.127 ms
 Execution time: 14.492 ms
(6 rows)


```

![_](../img_src/000/2018-08-06_194928.png)



没有索引（与案例不符）

```
tutorial=#  explain analyze select t1.*,t2.* from test1 t1 join test2 t2 on (t1.id=t2.id ); 
                                                                QUERY PLAN                                                                 
-------------------------------------------------------------------------------------------------------------------------------------------
 Hash Join  (cost=20000000319.00..20000000763.00 rows=10000 width=90) (actual time=2.728..12.057 rows=10000 loops=1)
   Hash Cond: (t1.id = t2.id)
   ->  Seq Scan on test1 t1  (cost=10000000000.00..10000000194.00 rows=10000 width=45) (actual time=0.004..0.805 rows=10000 loops=1)
   ->  Hash  (cost=10000000194.00..10000000194.00 rows=10000 width=45) (actual time=2.712..2.712 rows=10000 loops=1)
         Buckets: 1024  Batches: 1  Memory Usage: 782kB
         ->  Seq Scan on test2 t2  (cost=10000000000.00..10000000194.00 rows=10000 width=45) (actual time=0.001..1.013 rows=10000 loops=1)
 Planning time: 0.244 ms
 Execution time: 12.746 ms
(8 rows)


```

![_](../img_src/000/2018-08-06_195353.png)

为什么是这个样子？



#### 加速外键约束更新和删除操作

```
tutorial=#  create table p(id int primary key, info text, crt_time timestamp); 
CREATE TABLE
tutorial=#  create table f(id int primary key, p_id int references p(id) on delete cascade on update cascade, info text, crt_time timestamp); 
CREATE TABLE
tutorial=#  insert into p select generate_series(1,10000), md5(random()::text), clock_timestamp(); 
INSERT 0 10000
tutorial=#  insert into f select generate_series(1,10000), generate_series(1,10000), md5(random()::text), clock_timestamp(); 
INSERT 0 10000

```

```
tutorial=#  explain (analyze,verbose,costs,buffers,timing) update p set id=0 where id=1; 
                                                       QUERY PLAN             
                                          
------------------------------------------------------------------------------
------------------------------------------
 Update on public.p  (cost=0.29..8.30 rows=1 width=46) (actual time=0.035..0.0
35 rows=0 loops=1)
   Buffers: shared hit=8
   ->  Index Scan using p_pkey on public.p  (cost=0.29..8.30 rows=1 width=46) 
(actual time=0.011..0.012 rows=1 loops=1)
         Output: 0, info, crt_time, ctid
         Index Cond: (p.id = 1)
         Buffers: shared hit=3
 Planning time: 0.063 ms
 Trigger RI_ConstraintTrigger_a_24670 for constraint f_p_id_fkey on p: time=1.
715 calls=1
 Trigger RI_ConstraintTrigger_c_24672 for constraint f_p_id_fkey on f: time=0.
052 calls=1
 Execution time: 1.829 ms
(10 rows)

```



```
tutorial=#  create index idx_f_1 on f(p_id); 
CREATE INDEX
tutorial=#  explain (analyze,verbose,costs,buffers,timing) update p set id=1 where id=0;
                                                       QUERY PLAN                                                       
------------------------------------------------------------------------------------------------------------------------
 Update on public.p  (cost=0.29..8.30 rows=1 width=47) (actual time=0.026..0.026 rows=0 loops=1)
   Buffers: shared hit=8
   ->  Index Scan using p_pkey on public.p  (cost=0.29..8.30 rows=1 width=47) (actual time=0.005..0.005 rows=1 loops=1)
         Output: 1, info, crt_time, ctid
         Index Cond: (p.id = 0)
         Buffers: shared hit=3
 Planning time: 0.162 ms
 Trigger RI_ConstraintTrigger_a_24670 for constraint f_p_id_fkey on p: time=0.430 calls=1
 Trigger RI_ConstraintTrigger_c_24672 for constraint f_p_id_fkey on f: time=0.111 calls=1
 Execution time: 0.589 ms
(10 rows)

tutorial=# 

```

#### 索引在排他约束中作用

​	对排他操作符的要求，左右操作数互换对结果没有影响，例如X=Y,Y=X

```
tutorial=#  CREATE TABLE test(id int,geo point,EXCLUDE USING btree (id WITH pg_catalog.=)); 
CREATE TABLE
tutorial=#  insert into test (id) values (1); 
INSERT 0 1
tutorial=#  insert into test (id) values (1);
ERROR:  conflicting key value violates exclusion constraint "test_id_excl"
DETAIL:  Key (id)=(1) conflicts with existing key (id)=(1).
tutorial=# 

```

​	加速唯一值约束

​	primary key

​	unique key

​	

### 是否使用索引和什么有关



​	首先是前面提到的Access Method,然后是使用的operetor class,以及opc中定义的operator或者function

​	这些满足后，还要遵循CBO的选择

```
 #seq_page_cost = 1.0  
 #random_page_cost = 4.0 
 #cpu_tuple_cost = 0.01 
 #cpu_index_tuple_cost = 0.005  
 #cpu_operator_cost = 0.0025             
 #effective_cache_size = 128MB
```

​	遵循完CBO的选择，还需要符合当前配置的Planner配置

```
 #enable_bitmapscan = on 
 #enable_hashagg = on 
 #enable_hashjoin = on 
 #enable_indexscan = on 
 #enable_material = on 
 #enable_mergejoin = on 
 #enable_nestloop = on 
 #enable_seqscan = on 
 #enable_sort = on  
 #enable_tidscan = on
```



### 多列索引使用

​	多列索引，使用任何列作为条件，只要条件中的操作符或者函数满足opclass的匹配都可以使用索引，索引被扫描的部分还是全部基本取决于条件是否有索引的第一列作为条件之一

```
tutorial=# create table test1(c1 int,c2 int);
CREATE TABLE
tutorial=# insert into test1 select 1 ,generate_series(1,100000);
INSERT 0 100000
tutorial=# create index idx_test1_1 on test1(c1,c2);
CREATE INDEX
tutorial=# analyze test1;
ANALYZE

```



```
tutorial=# explain analyze select * from test1 where c2 = 100;
                                            QUERY PLAN                        
                     
------------------------------------------------------------------------------
---------------------
 Seq Scan on test1  (cost=0.00..1693.00 rows=1 width=8) (actual time=0.027..10
.477 rows=1 loops=1)
   Filter: (c2 = 100)
   Rows Removed by Filter: 99999
 Planning time: 0.313 ms
 Execution time: 10.673 ms
(5 rows)

```

将全局扫描置为off

```
tutorial=# set enable_seqscan = off;
SET
tutorial=# explain analyze select * from test1 where c2 = 100;
                                                        QUERY PLAN            
                                             
------------------------------------------------------------------------------
---------------------------------------------
 Index Only Scan using idx_test1_1 on test1  (cost=0.29..1858.30 rows=1 width=
8) (actual time=0.014..1.403 rows=1 loops=1)
   Index Cond: (c2 = 100)
   Heap Fetches: 1
 Planning time: 0.063 ms
 Execution time: 1.422 ms
(5 rows)


```

**创建多列索引时，选择第一个字段尤为重要**





### 索引带来的弊端



​	索引随着表的记录块的变迁需要更新，因此会对这类操作带来一定的性能影响（块不变更的情况下触发HOT特性，可以不需要更新索引)

​	

###  使用索引的注意事项



​	正常创建索引时，会阻断除了查询以外的其他操作

​	使用并行concurrently选项后，可以允许同时对表的DML操作，但是对于频繁的DML的表，这种创建索引的时间比较长

​	某些索引不记录WAL，所以如果有用WAL进行数据恢复的情况（流复制），这种索引需要在使用前重建(hash索引)



### 索引类型

​	索引算法不同，B-tree,HASH,Gist,SP-GiST 

​	`select amname from pg_am`



### 索引应用场景



### Btree

<

\>

=

\>=

<=

###  HASH

### GIN

### Gist 



除此之外，gist索引还支持紧邻排序，例如

​	 