[TOC]

# pg_技术内幕

​	摘自《POSTGRESQL 从小工到专家》

## 说明

​	本章节第一小节主要讲述了oid,tableoid,xmin,xmax,cmin,cmax,ctid在数据行的隐藏或者没有开启的字段，第二小节讲述了多版本并发控制(这个需要好好了解)，第三部分讲述了物理存储结构，第四部分讲述了部分技术并解密相关原理。

## 表中的系统字段

​	表中系统字段主要讲述如下几个，分别是oid,xmin,xmax,cmin,cmax,ctid

​	这里先说一下tableoid，tableoid指的时所属表的oid,那么什么场景可能会用到，在9.x系列中经常出现继承表，当子表继承父表，如果在查询语句中查询父表，默认情况下会出现子表数据，那么我们如何区分到底是子表数据还是父表独有数据时，不需要非得使用select * from only father_table来查看父表独有数据，可以添加tableoid字段来区分是否是子表，还是父表，还是其他一系列继承表。



### OID

​	接触oid最早是在pg_class表中，这个pg_class表存储表，索引，视图等一系列对象。在pg_class的表结构(使用\d展现)都没有找到oid字段，当时就觉得好神奇。

​	<u>小技巧:查看这张表是否有oid字段,可以使用\d+ tabname，查看最后一行是否拥有"HASH OIDS:YES"</u>

- [ ] ```
  osdba=# \d+ pg_class;
                            Table "pg_catalog.pg_class"
       Column     |   Type    | Modifiers | Storage  | Stats target | Description 
  ----------------+-----------+-----------+----------+--------------+-------------
   relname        | name      | not null  | plain    |              | 
   relnamespace   | oid       | not null  | plain    |              | 
   reltype        | oid       | not null  | plain    |              | 
   reloftype      | oid       | not null  | plain    |              | 
   relowner       | oid       | not null  | plain    |              | 
   relam          | oid       | not null  | plain    |              | 
   relfilenode    | oid       | not null  | plain    |              | 
   reltablespace  | oid       | not null  | plain    |              | 
   relpages       | integer   | not null  | plain    |              | 
   reltuples      | real      | not null  | plain    |              | 
   relallvisible  | integer   | not null  | plain    |              | 
   reltoastrelid  | oid       | not null  | plain    |              | 
   relhasindex    | boolean   | not null  | plain    |              | 
   relisshared    | boolean   | not null  | plain    |              | 
   relpersistence | "char"    | not null  | plain    |              | 
   relkind        | "char"    | not null  | plain    |              | 
   relnatts       | smallint  | not null  | plain    |              | 
   relchecks      | smallint  | not null  | plain    |              | 
   relhasoids     | boolean   | not null  | plain    |              | 
   relhaspkey     | boolean   | not null  | plain    |              | 
   relhasrules    | boolean   | not null  | plain    |              | 
   relhastriggers | boolean   | not null  | plain    |              | 
   relhassubclass | boolean   | not null  | plain    |              | 
   relispopulated | boolean   | not null  | plain    |              | 
   relreplident   | "char"    | not null  | plain    |              | 
   relfrozenxid   | xid       | not null  | plain    |              | 
   relminmxid     | xid       | not null  | plain    |              | 
   relacl         | aclitem[] |           | extended |              | 
   reloptions     | text[]    |           | extended |              | 
  Indexes:
      "pg_class_oid_index" UNIQUE, btree (oid)
      "pg_class_relname_nsp_index" UNIQUE, btree (relname, relnamespace)
      "pg_class_tblspc_relfilenode_index" btree (reltablespace, relfilenode)
  Has OIDs: yes
  
  ```

  postgresql在内部使用对象标识符(oid)作为各种系统的主键。

  系统不会给用户创建的表增加一个oid系统字段，当然用户可以在创建表时添加“with oids"就可以创建出来，并且可以查询添加oid就可以了。

- [ ] ```
  tutorial=# create table t_1(id int4,name text) with oids;
  CREATE TABLE
  tutorial=# \d+ t_1;
                            Table "public.t_1"
   Column |  Type   | Modifiers | Storage  | Stats target | Description 
  --------+---------+-----------+----------+--------------+-------------
   id     | integer |           | plain    |              | 
   name   | text    |           | extended |              | 
  Has OIDs: yes
  
  tutorial=# select * from t_1;
   id | name 
  ----+------
  (0 rows)
  
  tutorial=# select oid,id,name from t_1;
   oid | id | name 
  -----+----+------
  (0 rows)
  
  tutorial=# insert into t_1 values(1,'gh');
  INSERT 70142 1
  tutorial=# select * from t_1;
   id | name 
  ----+------
    1 | gh
  (1 row)
  
  tutorial=# select oid,id,name from t_1;
    oid  | id | name 
  -------+----+------
   70142 |  1 | gh
  (1 row)
  
  tutorial=# 
  
  ```

- [ ] 再看如下一条语句

- [ ] ```
  tutorial=# create table t_2(id int4,name text);
  CREATE TABLE
  tutorial=# insert into t_2 values(1,'gh');
  INSERT 0 1
  
  ```

- [ ] 为什么同时让表内插入数据，第一个返回"INSERT 70142 1",而第二个返回"INSERT 0 1"这就是因为前者表中拥有oid,后者表中没有该字段，第二列数据返回oid值，如果没有oid字段就变为0，第三列代表数据量。

- [ ] ```
  tutorial=# insert into t_1 select generate_series(2,1000),'gh';
  INSERT 0 999
  tutorial=# select oid,* from t_1 limit 10;
    oid  | id | name 
  -------+----+------
   72150 |  2 | gh
   72151 |  3 | gh
   72152 |  4 | gh
   72153 |  5 | gh
   72154 |  6 | gh
   72155 |  7 | gh
   72156 |  8 | gh
   72157 |  9 | gh
   72158 | 10 | gh
   72159 | 11 | gh
  (10 rows)
  
  
  ```

- [ ] 那么当前情况下，为何在第二列返回0,而不是返回相对性oid呢，我的猜测是因为返回的数据量太多，将所有的oid展现出来，不是很美观，所以当多条数据一次性插入是，就将第二列置为0

- [ ] 

​	



