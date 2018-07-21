[TOC]

# PG pageinspect

​	在学习数据页和数据行是发现pg提供一些函数能够更具体查看相关信息，就百度了一些文档，参考文档和视频来学习这方面知识



## 安装

​	1、检查函数是否存在

```
osdba=# \df pg_header*
                       List of functions
 Schema | Name | Result data type | Argument data types | Type 
--------+------+------------------+---------------------+------
(0 rows)

osdba=# \df *page*
                                                                                                         
                                                    List of functions
 Schema |       Name        | Result data type |                                                         
                                                                      Argument data types                
                                                                                                         
       |  Type  
--------+-------------------+------------------+---------------------------------------------------------
---------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------
-------+--------
 public | bt_page_items     | SETOF record     | relname text, blkno integer, OUT itemoffset smallint, OU
T ctid tid, OUT itemlen smallint, OUT nulls boolean, OUT vars boolean, OUT data text                     
                                                                                                         
       | normal
 public | bt_page_stats     | record           | relname text, blkno integer, OUT blkno integer, OUT type
 "char", OUT live_items integer, OUT dead_items integer, OUT avg_item_size integer, OUT page_size integer
, OUT free_size integer, OUT btpo_prev integer, OUT btpo_next integer, OUT btpo integer, OUT btpo_flags i
nteger | normal
 public | fsm_page_contents | text             | page bytea                                              
                                                                                                         
                                                                                                         
       | normal
 public | get_raw_page      | bytea            | text, integer                                           
                                                                                                         
                                                                                                         
       | normal
 public | get_raw_page      | bytea            | text, text, integer                                     
                                                                                                         
                                                                                                         
       | normal
 public | heap_page_items   | SETOF record     | page bytea, OUT lp smallint, OUT lp_off smallint, OUT lp
_flags smallint, OUT lp_len smallint, OUT t_xmin xid, OUT t_xmax xid, OUT t_field3 integer, OUT t_ctid ti
d, OUT t_infomask2 integer, OUT t_infomask integer, OUT t_hoff smallint, OUT t_bits text, OUT t_oid oid  
       | normal
 public | page_header       | record           | page bytea, OUT lsn pg_lsn, OUT checksum smallint, OUT f
lags smallint, OUT lower smallint, OUT upper smallint, OUT special smallint, OUT pagesize smallint, OUT v
ersion smallint, OUT prune_xid xid                                                                       
       | normal
(7 rows)



```

​	出现page_header和get_raw_page函数说明之前就编译过，跳过2,3，直接查看使用命令 ；如果没有就需要参考2，3，



​	

​	2、首先编译数据库时尽量使用源码安装并且安装地址保持不变

```
cd pageinspcet
make
make install
```



​	3、创建extension

```
create extension pageinspect
```



## 使用





































## 链接地址

https://yq.aliyun.com/articles/2291

https://www.postgresql.org/docs/current/static/pageinspect.html?spm=a2c4e.11153940.blogcont2291.5.36292b2bYnZkIy



