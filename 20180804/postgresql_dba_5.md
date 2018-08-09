[TOC]

# pg dba 5

​	

​	本章节主要讲述了执行计划，成本公式解说，代价因此校准，自动追踪sql执行计划



## explain

### explain 语法



explain:显示sql的执行计划

analyze:通过实际执行的sql来获取相应的执行计划

verbose:显示计划的附加信息，如计划树中每个节点输出的各个列。

costs：每个节点的启动成本和总成本，以及估计行数和每行的宽度。

buffers：显示缓存区使用的信息，该参数只能和analyze参数一起使用

format:指定的输出格式，text,xml,json,yaml

timing:输出时间开销



**需要注意analyze的使用，analyze会真的执行相关sql语句**

可以通过 begin; explain analyze sql; rollback;方式解决



### explain 输出

```
tutorial=# explain(analyze,verbose,costs,buffers,timing) select count(1) from test1;
                                                        QUERY PLAN                                                   
     
---------------------------------------------------------------------------------------------------------------------
-----
 Aggregate  (cost=1887.00..1887.01 rows=1 width=0) (actual time=24.904..24.904 rows=1 loops=1)
   Output: count(1)
   Buffers: shared hit=637
   ->  Seq Scan on public.test1  (cost=0.00..1637.00 rows=100000 width=0) (actual time=0.009..12.563 rows=100000 loop
s=1)
--seq scan on table:这个节点的路径
--0.00 表示第一行输出的成本，后面一个是总的成本，actual表示真实时间
         Output: id, info, crt_time
         Buffers: shared hit=637 --从shared buffer命中637page,如果在shared buffer中没有命中，就需要到os缓存或者硬盘中查找了read
 Planning time: 0.030 ms--计划执行时间
 Execution time: 24.936 ms--总的执行时间
(8 rows)

```



### 嵌套连接

nested loop join: The right relation is scanned once for every row found in the left relation. This strategy is easy to implement but can be very time consuming. (However, if the right relation can be scanned with an index scan, this can be a good strategy. It is possible to use values from the current row of the left relation as keys for the index scan of the right.)

嵌套循环

```
for  tuple in 左表查询 loop
右表查询(根据左表查询得到的行作为右表查询的条件依次输出最终结果)
end loop;
```



适合右表的关联列发生在唯一键值列或者主键上的情况



```
tutorial=# explain(analyze ,verbose,costs,buffers,timing) select t1.* from test1 t1,test2 t2 where t1.id = t2.id 
and t1.id < 100;
                                                                QUERY PLAN                                           
                     
---------------------------------------------------------------------------------------------------------------------
---------------------
 Nested Loop  (cost=0.43..39.86 rows=1 width=19) (actual time=0.048..0.270 rows=99 loops=1)
   Output: t1.id, t1.info, t1.crt_time
   Buffers: shared hit=200 read=1
   ->  Index Scan using idx_test1_id on public.test1 t1  (cost=0.29..10.10 rows=103 width=19) (actual time=0.009..0.0
29 rows=99 loops=1)
         Output: t1.id, t1.info, t1.crt_time
         Index Cond: (t1.id < 100)
         Buffers: shared hit=3
   ->  Index Only Scan using idx_test2_id on public.test2 t2  (cost=0.14..0.28 rows=1 width=4) (actual time=0.001..0.
002 rows=1 loops=99)
         Output: t2.id
         Index Cond: (t2.id = t1.id)
         Heap Fetches: 99
         Buffers: shared hit=197 read=1
 Planning time: 0.346 ms
 Execution time: 0.321 ms
(14 rows)


```



### HASH连接

hash join: the right relation is first scanned and loaded into a hash table, using its join attributes as hash keys. Next the left relation is scanned and the appropriate values of every row found are used as hash keys to locate the matching rows in the table

首先右表扫描加载到内存HASH表, hash key为JOIN列

然后左表扫描, 并与内存中的HASH表进行关联, 输出最终结果

```
tutorial=# explain(analyze ,verbose,costs,buffers,timing) select t1.* from test1 t1,test2 t2 where t1.id = t2.id 
and t1.id < 100;
                                                               QUERY PLAN                                                                
-----------------------------------------------------------------------------------------------------------------------------------------
 Hash Join  (cost=3.54..13.74 rows=1 width=19) (actual time=0.034..0.106 rows=99 loops=1)
   Output: t1.id, t1.info, t1.crt_time
   Hash Cond: (t1.id = t2.id)
   Buffers: shared hit=4
   ->  Index Scan using idx_test1_id on public.test1 t1  (cost=0.29..10.10 rows=103 width=19) (actual time=0.003..0.028 rows=99 loops=1)
         Output: t1.id, t1.info, t1.crt_time
         Index Cond: (t1.id < 100)
         Buffers: shared hit=3
   ->  Hash  (cost=2.00..2.00 rows=100 width=4) (actual time=0.027..0.027 rows=100 loops=1)
         Output: t2.id
         Buckets: 1024  Batches: 1  Memory Usage: 4kB
         Buffers: shared hit=1
         ->  Seq Scan on public.test2 t2  (cost=0.00..2.00 rows=100 width=4) (actual time=0.000..0.011 rows=100 loops=1)
               Output: t2.id
               Buffers: shared hit=1
 Planning time: 0.148 ms
 Execution time: 0.135 ms
(17 rows)

```

### MERGE连接

merge join: Each relation is sorted on the join attributes before the join starts. Then the two relations are scanned in parallel, and matching rows are combined to form join rows. This kind of join is more attractive because each relation has to be scanned only once. The required sorting might be achieved either by an explicit sort step, or by scanning the relation in the proper order using an index on the join key.



首先两个JOIN的表根据join key进行排序,然后根据join key的排序顺序并行扫描两个表进行匹配输出最终结果,适合大表并且索引列进行关联的情况 



```
tutorial=# explain(analyze ,verbose,costs,buffers,timing) select t1.* from test1 t1,test2 t2 where t1.id = t2.id 
and t1.id < 100;
                                                               QUERY PLAN                                                                
-----------------------------------------------------------------------------------------------------------------------------------------
 Merge Join  (cost=5.63..6.22 rows=1 width=19) (actual time=0.025..0.079 rows=99 loops=1)
   Output: t1.id, t1.info, t1.crt_time
   Merge Cond: (t1.id = t2.id)
   Buffers: shared hit=4
   ->  Index Scan using idx_test1_id on public.test1 t1  (cost=0.29..10.10 rows=103 width=19) (actual time=0.004..0.022 rows=99 loops=1)
         Output: t1.id, t1.info, t1.crt_time
         Index Cond: (t1.id < 100)
         Buffers: shared hit=3
   ->  Sort  (cost=5.32..5.57 rows=100 width=4) (actual time=0.020..0.023 rows=100 loops=1)
         Output: t2.id
         Sort Key: t2.id
         Sort Method: quicksort  Memory: 29kB
         Buffers: shared hit=1
         ->  Seq Scan on public.test2 t2  (cost=0.00..2.00 rows=100 width=4) (actual time=0.002..0.009 rows=100 loops=1)
               Output: t2.id
               Buffers: shared hit=1
 Planning time: 0.136 ms
 Execution time: 0.098 ms
(18 rows)


```



## explain 成本计算

​	成本计算相关的参数和系统表或视图

​	表或视图

- pg_stats

- pg_class --用到relpages和reltuples

  参数

- seq_page_cost --全表扫描的单个数据块的代价因子

- random_page_cost --索引扫描的单个数据块的代价因子

- cpu_tuple_cost --处理每条记录的CPU开销代价因子

- cpu_index_tuple_cost --索引扫描时每个索引条目的CPU开销代价因子

- cpu_operator_cost --操作符或函数的开销代价因子



### pg_stats

| 名字                     | 类型       | 引用                   | 描述                                                         |
| ------------------------ | ---------- | ---------------------- | ------------------------------------------------------------ |
| `schemaname`             | `name`     | `pg_namespace.nspname` | 包含此表的模式名字                                           |
| `tablename`              | `name`     | `pg_class.relname`     | 表的名字                                                     |
| `attname`                | `name`     | `pg_attribute.attname` | 这一行描述的字段的名字                                       |
| `inherited`              | `bool`     |                        | 如果为真，那么这行包含继承的子字段，不只是指定表的值。       |
| `null_frac`              | `real`     |                        | 记录中字段为空的百分比                                       |
| `avg_width`              | `integer`  |                        | 字段记录以字节记的平均宽度                                   |
| `n_distinct`             | `real`     |                        | 如果大于零，就是在字段中独立数值的估计数目。如果小于零，    就是独立数值的数目被行数除的负数。用负数形式是因为`ANALYZE`    认为独立数值的数目是随着表增长而增长；    正数的形式用于在字段看上去好像有固定的可能值数目的情况下。比如，    -1 表示一个唯一字段，独立数值的个数和行数相同。 |
| `most_common_vals`       | `anyarray` |                        | 一个字段里最常用数值的列表。如果看上去没有啥数值比其它更常见，则为 null |
| `most_common_freqs`      | `real[]`   |                        | 一个最常用数值的频率的列表，也就是说，每个出现的次数除以行数。    如果`most_common_vals`是 null ，则为 null。 |
| `histogram_bounds`       | `anyarray` |                        | 一个数值的列表，它把字段的数值分成几组大致相同热门的组。    如果在`most_common_vals`里有数值，则在这个饼图的计算中省略。    如果字段数据类型没有`<`操作符或者`most_common_vals`    列表代表了整个分布性，则这个字段为 null。 |
| `correlation`            | `real`     |                        | 统计与字段值的物理行序和逻辑行序有关。它的范围从 -1 到 +1 。    在数值接近 -1 或者 +1 的时候，在字段上的索引扫描将被认为比它接近零的时候开销更少，    因为减少了对磁盘的随机访问。如果字段数据类型没有`<`操作符，那么这个字段为null。 |
| `most_common_elems`      | `anyarray` |                        | 经常在字段值中出现的非空元素值的列表。（标量类型为空。）     |
| `most_common_elem_freqs` | `real[]`   |                        | 最常见元素值的频率列表，也就是，至少包含一个给定值的实例的行的分数。    每个元素频率跟着两到三个附加的值；它们是在每个元素频率之前的最小和最大值，    还有可选择的null元素的频率。（当`most_common_elems`    为null时，为null） |
| `elem_count_histogram`   | `real[]`   |                        | 该字段中值的不同非空元素值的统计直方图，跟着不同非空元素的平均值。（标量类型为空。） |

### 全表扫描

​	全表扫描的成本计算

```
tutorial=# analyze test1;
ANALYZE
tutorial=# explain select * from test1; 
                          QUERY PLAN                          
--------------------------------------------------------------
 Seq Scan on test1  (cost=0.00..1637.00 rows=100000 width=17)
(1 row)
```

cost等于1637是怎么得来的，全表计算的成本只需要计算pg_class

```
tutorial=# select relpages,reltuples from pg_class where oid='test1'::regclass;
 relpages | reltuples 
----------+-----------
      637 |    100000
(1 row)
tutorial=# show seq_page_cost;
 seq_page_cost 
---------------
 1
(1 row)

tutorial=# show cpu_tuple_cost;
 cpu_tuple_cost 
----------------
 0.01
(1 row)

tutorial=# select 637*1+100000*0.01;
 ?column? 
----------
  1637.00
(1 row)

tutorial=# 

```

### 行数评估

#### 柱状图

```

tutorial=# explain select * from test1 where id < 2000;
                         QUERY PLAN                         
------------------------------------------------------------
 Seq Scan on test1  (cost=0.00..1887.00 rows=1903 width=17)
   Filter: (id < 2000)
(2 rows)

```

rows等于1903是怎么得来的呢？

```
tutorial=# select * from pg_stats where tablename ='test1' and attname ='id';
-[ RECORD 1 ]----------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
schemaname             | public
tablename              | test1
attname                | id
inherited              | f
null_frac              | 0
avg_width              | 4
n_distinct             | -1
most_common_vals       | 
most_common_freqs      | 
histogram_bounds       | {1,1017,2105,3251,4331,5285,6345,7319,8366,9317,10292,11254,12263,13307,14302,15333,16369,17328,18273,19272,20191,21184,22150,23182,24218,25229,26259,27228,28217,29110,30072,31065,32113,33017,34024,34994,35983,36977,37946,38875,39749,40665,41686,42651,43702,44636,45636,46590,47611,48608,49635,50655,51633,52660,53602,54593,55642,56551,57489,58473,59551,60514,61561,62556,63583,64657,65730,66650,67615,68716,69680,70654,71660,72723,73775,74757,75666,76693,77621,78610,79620,80655,81745,82675,83703,84689,85797,86802,87798,88772,89812,90768,91767,92809,93792,94862,95902,96883,97945,98990,100000}
correlation            | 1
most_common_elems      | 
most_common_elem_freqs | 
elem_count_histogram   | 

```



一共有100个柱面，其中2000位于1017~2105之间占比 之后占据总的占比

select (1+(2000-1017)/(2105-1017))/100.0;

```
tutorial=# select 0.01903492647058823529*100000;
-[ RECORD 1 ]-----------------------
?column? | 1903.49264705882352900000

```

