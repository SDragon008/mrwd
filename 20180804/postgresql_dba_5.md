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

