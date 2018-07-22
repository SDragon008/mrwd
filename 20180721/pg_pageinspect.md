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



函数：`get_raw_page(relname text, fork text, blkno int) returns bytea` 

`get_raw_page` reads the specified block of the named relation and returns a copy as a `bytea` value. This allows a single time-consistent copy of the block to be obtained. *fork* should be `'main'` for the main data fork, `'fsm'` for the free space map, `'vm'` for the visibility map, or `'init'` for the initialization  

函数：`get_raw_page(relname text, blkno int) returns bytea`

A shorthand version of `get_raw_page`, for reading from the main fork. Equivalent to `get_raw_page(relname, 'main', blkno)`

函数：`page_header(page bytea) returns record` 

`page_header` shows fields that are common to all PostgreSQL heap and index pages.



```
osdba=# truncate table test_1;
TRUNCATE TABLE
osdba=# insert into test_1 select generate_series(1,10);
INSERT 0 10
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/500C4ED0 |        0 |     0 |    64 |  7872 |    8192 |     8192 |       4 |         0
(1 row)


```

The returned columns correspond to the fields in the `PageHeaderData` struct. See `src/include/storage/bufpage.h` for details. 

查看一下源码

```
/*
 * disk page organization
 *
 * space management information generic to any page
 *
 *		pd_lsn		- identifies xlog record for last change to this page.
 *		pd_checksum - page checksum, if set.
 *		pd_flags	- flag bits.
 *		pd_lower	- offset to start of free space.
 *		pd_upper	- offset to end of free space.
 *		pd_special	- offset to start of special space.
 *		pd_pagesize_version - size in bytes and page layout version number.
 *		pd_prune_xid - oldest XID among potentially prunable tuples on page.
 *
 * The LSN is used by the buffer manager to enforce the basic rule of WAL:
 * "thou shalt write xlog before data".  A dirty buffer cannot be dumped
 * to disk until xlog has been flushed at least as far as the page's LSN.
 *
 * pd_checksum stores the page checksum, if it has been set for this page;
 * zero is a valid value for a checksum. If a checksum is not in use then
 * we leave the field unset. This will typically mean the field is zero
 * though non-zero values may also be present if databases have been
 * pg_upgraded from releases prior to 9.3, when the same byte offset was
 * used to store the current timelineid when the page was last updated.
 * Note that there is no indication on a page as to whether the checksum
 * is valid or not, a deliberate design choice which avoids the problem
 * of relying on the page contents to decide whether to verify it. Hence
 * there are no flag bits relating to checksums.
 *
 * pd_prune_xid is a hint field that helps determine whether pruning will be
 * useful.  It is currently unused in index pages.
 *
 * The page version number and page size are packed together into a single
 * uint16 field.  This is for historical reasons: before PostgreSQL 7.3,
 * there was no concept of a page version number, and doing it this way
 * lets us pretend that pre-7.3 databases have page version number zero.
 * We constrain page sizes to be multiples of 256, leaving the low eight
 * bits available for a version number.
 *
 * Minimum possible page size is perhaps 64B to fit page header, opaque space
 * and a minimal tuple; of course, in reality you want it much bigger, so
 * the constraint on pagesize mod 256 is not an important restriction.
 * On the high end, we can only support pages up to 32KB because lp_off/lp_len
 * are 15 bits.
 */




typedef struct PageHeaderData
{
	/* XXX LSN is member of *any* block, not only page-organized ones */
	PageXLogRecPtr pd_lsn;		/* LSN: next byte after last byte of xlog
								 * record for last change to this page */
	uint16		pd_checksum;	/* checksum */
	uint16		pd_flags;		/* flag bits, see below */
	LocationIndex pd_lower;		/* offset to start of free space */
	LocationIndex pd_upper;		/* offset to end of free space */
	LocationIndex pd_special;	/* offset to start of special space */
	uint16		pd_pagesize_version;
	TransactionId pd_prune_xid; /* oldest prunable XID, or zero if none */
	ItemIdData	pd_linp[1];		/* beginning of line pointer array */
} PageHeaderData;


```



pg_lsn:最后一次被修改后xlog值

pg_checksum:不知道含义

pg_flags:不知道含义

pg_lower和pg_upper代表空闲空间的位移

pg_special:特殊空间的位移

pg_pagesize_version:不知道含义

pg_prune_xid:修改(不包括插入)的xid值

pg_linp:不知道含义



```
osdba=# truncate table test_1;
TRUNCATE TABLE
osdba=# insert into test_1 select generate_series(1,10);
INSERT 0 10
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/5016F6C8 |        0 |     0 |    64 |  7872 |    8192 |     8192 |       4 |         0
(1 row)

查看当前的lsn是在那个xlog文件中

osdba=# select pg_xlogfile_name('3/5016F6C8');
     pg_xlogfile_name     
--------------------------
 000000010000000300000050
(1 row)

osdba=# insert into test_1 select generate_series(1,100);
INSERT 0 100
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/50171010 |        0 |     0 |   464 |  4672 |    8192 |     8192 |       4 |         0
(1 row)

osdba=# select pg_xlog_location_diff('3/50171010','3/5016F6C8');
 pg_xlog_location_diff 
-----------------------
                  6472

随着数据插入的增加,lower变大，upper变小，等到他们的差值不足以满足新插入的要求时，会重新开启一个page,来插入数据。

手动检查点，切换新的xlog日志

osdba=# checkpoint;
CHECKPOINT

osdba=# select pg_switch_xlog();
 pg_switch_xlog 
----------------
 3/50173190
(1 row)

osdba=# insert into test_1 select generate_series(1,10);
INSERT 0 10
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/51001270 |        0 |     0 |   504 |  4352 |    8192 |     8192 |       4 |         0
(1 row)

osdba=# select pg_xlogfile_name('3/51001270');
     pg_xlogfile_name     
--------------------------
 000000010000000300000051
(1 row)

osdba=# select ctid,* from test_1 where id = 100;
  ctid   | id  
---------+-----
 (0,110) | 100
(1 row)

osdba=# begin;
BEGIN
osdba=# delete from test_1 where id = 100;
DELETE 1
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/510012E0 |        0 |     0 |   504 |  4352 |    8192 |     8192 |       4 |      7005
(1 row)

osdba=# select txid_current();
 txid_current 
--------------
         7005
(1 row)

osdba=# rollback;

ROLLBACK
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/510012E0 |        0 |     0 |   504 |  4352 |    8192 |     8192 |       4 |      7005
(1 row)

回滚后为什么prune_xid没有变回0

```



heap_page_items(page bytea):items的相关信息

`src/include/storage/itemid.h` ，`src/include/access/htup_details.h` 

```
osdba=# truncate table test_1;
TRUNCATE TABLE
osdba=# insert into test_1 select generate_series(1,10);
INSERT 0 10


osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/51009210 |        0 |     0 |    64 |  7872 |    8192 |     8192 |       4 |         0
(1 row)

osdba=# select * from heap_page_items(get_raw_page('test_1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   7010 |      0 |        0 | (0,1)  |           1 |       2048 |     24 |        |      
  2 |   8128 |        1 |     28 |   7010 |      0 |        0 | (0,2)  |           1 |       2048 |     24 |        |      
  3 |   8096 |        1 |     28 |   7010 |      0 |        0 | (0,3)  |           1 |       2048 |     24 |        |      
  4 |   8064 |        1 |     28 |   7010 |      0 |        0 | (0,4)  |           1 |       2048 |     24 |        |      
  5 |   8032 |        1 |     28 |   7010 |      0 |        0 | (0,5)  |           1 |       2048 |     24 |        |      
  6 |   8000 |        1 |     28 |   7010 |      0 |        0 | (0,6)  |           1 |       2048 |     24 |        |      
  7 |   7968 |        1 |     28 |   7010 |      0 |        0 | (0,7)  |           1 |       2048 |     24 |        |      
  8 |   7936 |        1 |     28 |   7010 |      0 |        0 | (0,8)  |           1 |       2048 |     24 |        |      
  9 |   7904 |        1 |     28 |   7010 |      0 |        0 | (0,9)  |           1 |       2048 |     24 |        |      
 10 |   7872 |        1 |     28 |   7010 |      0 |        0 | (0,10) |           1 |       2048 |     24 |        |      
(10 rows)

删除数据，prune_xid和tmax都发生变化。
osdba=# delete from test_1 where id = 4;
DELETE 1
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/51009480 |        0 |     0 |    64 |  7872 |    8192 |     8192 |       4 |      7011
(1 row)

osdba=# select * from heap_page_items(get_raw_page('test_1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   7010 |      0 |        0 | (0,1)  |           1 |       2304 |     24 |        |      
  2 |   8128 |        1 |     28 |   7010 |      0 |        0 | (0,2)  |           1 |       2304 |     24 |        |      
  3 |   8096 |        1 |     28 |   7010 |      0 |        0 | (0,3)  |           1 |       2304 |     24 |        |      
  4 |   8064 |        1 |     28 |   7010 |   7011 |        0 | (0,4)  |        8193 |        256 |     24 |        |      
  5 |   8032 |        1 |     28 |   7010 |      0 |        0 | (0,5)  |           1 |       2304 |     24 |        |      
  6 |   8000 |        1 |     28 |   7010 |      0 |        0 | (0,6)  |           1 |       2304 |     24 |        |      
  7 |   7968 |        1 |     28 |   7010 |      0 |        0 | (0,7)  |           1 |       2304 |     24 |        |      
  8 |   7936 |        1 |     28 |   7010 |      0 |        0 | (0,8)  |           1 |       2304 |     24 |        |      
  9 |   7904 |        1 |     28 |   7010 |      0 |        0 | (0,9)  |           1 |       2304 |     24 |        |      
 10 |   7872 |        1 |     28 |   7010 |      0 |        0 | (0,10) |           1 |       2304 |     24 |        |      
(10 rows)

vacuum后，lp_off变为0，vacuum full 后重新整理数据
osdba=# vacuum test_1;
VACUUM
osdba=# select * from heap_page_items(get_raw_page('test_1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   7010 |      0 |        0 | (0,1)  |           1 |       2304 |     24 |        |      
  2 |   8128 |        1 |     28 |   7010 |      0 |        0 | (0,2)  |           1 |       2304 |     24 |        |      
  3 |   8096 |        1 |     28 |   7010 |      0 |        0 | (0,3)  |           1 |       2304 |     24 |        |      
  4 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  5 |   8064 |        1 |     28 |   7010 |      0 |        0 | (0,5)  |           1 |       2304 |     24 |        |      
  6 |   8032 |        1 |     28 |   7010 |      0 |        0 | (0,6)  |           1 |       2304 |     24 |        |      
  7 |   8000 |        1 |     28 |   7010 |      0 |        0 | (0,7)  |           1 |       2304 |     24 |        |      
  8 |   7968 |        1 |     28 |   7010 |      0 |        0 | (0,8)  |           1 |       2304 |     24 |        |      
  9 |   7936 |        1 |     28 |   7010 |      0 |        0 | (0,9)  |           1 |       2304 |     24 |        |      
 10 |   7904 |        1 |     28 |   7010 |      0 |        0 | (0,10) |           1 |       2304 |     24 |        |      
(10 rows)

osdba=# vacuum full test_1;
VACUUM
osdba=# select * from heap_page_items(get_raw_page('test_1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   7010 |      0 |        0 | (0,1)  |           1 |       2816 |     24 |        |      
  2 |   8128 |        1 |     28 |   7010 |      0 |        0 | (0,2)  |           1 |       2816 |     24 |        |      
  3 |   8096 |        1 |     28 |   7010 |      0 |        0 | (0,3)  |           1 |       2816 |     24 |        |      
  4 |   8064 |        1 |     28 |   7010 |      0 |        0 | (0,4)  |           1 |       2816 |     24 |        |      
  5 |   8032 |        1 |     28 |   7010 |      0 |        0 | (0,5)  |           1 |       2816 |     24 |        |      
  6 |   8000 |        1 |     28 |   7010 |      0 |        0 | (0,6)  |           1 |       2816 |     24 |        |      
  7 |   7968 |        1 |     28 |   7010 |      0 |        0 | (0,7)  |           1 |       2816 |     24 |        |      
  8 |   7936 |        1 |     28 |   7010 |      0 |        0 | (0,8)  |           1 |       2816 |     24 |        |      
  9 |   7904 |        1 |     28 |   7010 |      0 |        0 | (0,9)  |           1 |       2816 |     24 |        |      
(9 rows)

osdba=# select * from test_1 where ctid='(0,7)';
 id 
----
  8
(1 row)

osdba=# 

delete 后vacuum 后，插入新的数据是否从这个新的值开始

osdba=# insert into test_1 select generate_series(1,10);
INSERT 0 10
osdba=# select * from heap_page_items(get_raw_page('test_1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   7010 |      0 |        0 | (0,1)  |           1 |       2816 |     24 |        |      
  2 |   8128 |        1 |     28 |   7010 |      0 |        0 | (0,2)  |           1 |       2816 |     24 |        |      
  3 |   8096 |        1 |     28 |   7010 |      0 |        0 | (0,3)  |           1 |       2816 |     24 |        |      
  4 |   8064 |        1 |     28 |   7010 |      0 |        0 | (0,4)  |           1 |       2816 |     24 |        |      
  5 |   8032 |        1 |     28 |   7010 |      0 |        0 | (0,5)  |           1 |       2816 |     24 |        |      
  6 |   8000 |        1 |     28 |   7010 |      0 |        0 | (0,6)  |           1 |       2816 |     24 |        |      
  7 |   7968 |        1 |     28 |   7010 |      0 |        0 | (0,7)  |           1 |       2816 |     24 |        |      
  8 |   7936 |        1 |     28 |   7010 |      0 |        0 | (0,8)  |           1 |       2816 |     24 |        |      
  9 |   7904 |        1 |     28 |   7010 |      0 |        0 | (0,9)  |           1 |       2816 |     24 |        |      
 10 |   7872 |        1 |     28 |   7013 |      0 |        0 | (0,10) |           1 |       2048 |     24 |        |      
 11 |   7840 |        1 |     28 |   7013 |      0 |        0 | (0,11) |           1 |       2048 |     24 |        |      
 12 |   7808 |        1 |     28 |   7013 |      0 |        0 | (0,12) |           1 |       2048 |     24 |        |      
 13 |   7776 |        1 |     28 |   7013 |      0 |        0 | (0,13) |           1 |       2048 |     24 |        |      
 14 |   7744 |        1 |     28 |   7013 |      0 |        0 | (0,14) |           1 |       2048 |     24 |        |      
 15 |   7712 |        1 |     28 |   7013 |      0 |        0 | (0,15) |           1 |       2048 |     24 |        |      
 16 |   7680 |        1 |     28 |   7013 |      0 |        0 | (0,16) |           1 |       2048 |     24 |        |      
 17 |   7648 |        1 |     28 |   7013 |      0 |        0 | (0,17) |           1 |       2048 |     24 |        |      
 18 |   7616 |        1 |     28 |   7013 |      0 |        0 | (0,18) |           1 |       2048 |     24 |        |      
 19 |   7584 |        1 |     28 |   7013 |      0 |        0 | (0,19) |           1 |       2048 |     24 |        |      
(19 rows)

osdba=# delete from test_1 where id = 4;
DELETE 1
osdba=# insert into test_1 select generate_series(1,10);
INSERT 0 10
osdba=# select * from heap_page_items(get_raw_page('test_1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   7010 |      0 |        0 | (0,1)  |           1 |       2816 |     24 |        |      
  2 |   8128 |        1 |     28 |   7010 |      0 |        0 | (0,2)  |           1 |       2816 |     24 |        |      
  3 |   8096 |        1 |     28 |   7010 |      0 |        0 | (0,3)  |           1 |       2816 |     24 |        |      
  4 |   8064 |        1 |     28 |   7010 |      0 |        0 | (0,4)  |           1 |       2816 |     24 |        |      
  5 |   8032 |        1 |     28 |   7010 |      0 |        0 | (0,5)  |           1 |       2816 |     24 |        |      
  6 |   8000 |        1 |     28 |   7010 |      0 |        0 | (0,6)  |           1 |       2816 |     24 |        |      
  7 |   7968 |        1 |     28 |   7010 |      0 |        0 | (0,7)  |           1 |       2816 |     24 |        |      
  8 |   7936 |        1 |     28 |   7010 |      0 |        0 | (0,8)  |           1 |       2816 |     24 |        |      
  9 |   7904 |        1 |     28 |   7010 |      0 |        0 | (0,9)  |           1 |       2816 |     24 |        |      
 10 |   7872 |        1 |     28 |   7013 |      0 |        0 | (0,10) |           1 |       2304 |     24 |        |      
 11 |   7840 |        1 |     28 |   7013 |      0 |        0 | (0,11) |           1 |       2304 |     24 |        |      
 12 |   7808 |        1 |     28 |   7013 |      0 |        0 | (0,12) |           1 |       2304 |     24 |        |      
 13 |   7776 |        1 |     28 |   7013 |   7014 |        0 | (0,13) |        8193 |        256 |     24 |        |      
 14 |   7744 |        1 |     28 |   7013 |      0 |        0 | (0,14) |           1 |       2304 |     24 |        |      
 15 |   7712 |        1 |     28 |   7013 |      0 |        0 | (0,15) |           1 |       2304 |     24 |        |      
 16 |   7680 |        1 |     28 |   7013 |      0 |        0 | (0,16) |           1 |       2304 |     24 |        |      
 17 |   7648 |        1 |     28 |   7013 |      0 |        0 | (0,17) |           1 |       2304 |     24 |        |      
 18 |   7616 |        1 |     28 |   7013 |      0 |        0 | (0,18) |           1 |       2304 |     24 |        |      
 19 |   7584 |        1 |     28 |   7013 |      0 |        0 | (0,19) |           1 |       2304 |     24 |        |      
 20 |   7552 |        1 |     28 |   7015 |      0 |        0 | (0,20) |           1 |       2048 |     24 |        |      
 21 |   7520 |        1 |     28 |   7015 |      0 |        0 | (0,21) |           1 |       2048 |     24 |        |      
 22 |   7488 |        1 |     28 |   7015 |      0 |        0 | (0,22) |           1 |       2048 |     24 |        |      
 23 |   7456 |        1 |     28 |   7015 |      0 |        0 | (0,23) |           1 |       2048 |     24 |        |      
 24 |   7424 |        1 |     28 |   7015 |      0 |        0 | (0,24) |           1 |       2048 |     24 |        |      
 25 |   7392 |        1 |     28 |   7015 |      0 |        0 | (0,25) |           1 |       2048 |     24 |        |      
 26 |   7360 |        1 |     28 |   7015 |      0 |        0 | (0,26) |           1 |       2048 |     24 |        |      
 27 |   7328 |        1 |     28 |   7015 |      0 |        0 | (0,27) |           1 |       2048 |     24 |        |      
 28 |   7296 |        1 |     28 |   7015 |      0 |        0 | (0,28) |           1 |       2048 |     24 |        |      
 29 |   7264 |        1 |     28 |   7015 |      0 |        0 | (0,29) |           1 |       2048 |     24 |        |      
(29 rows)


osdba=# vacuum test_1;
VACUUM
osdba=# select * from heap_page_items(get_raw_page('test_1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   7010 |      0 |        0 | (0,1)  |           1 |       2816 |     24 |        |      
  2 |   8128 |        1 |     28 |   7010 |      0 |        0 | (0,2)  |           1 |       2816 |     24 |        |      
  3 |   8096 |        1 |     28 |   7010 |      0 |        0 | (0,3)  |           1 |       2816 |     24 |        |      
  4 |   8064 |        1 |     28 |   7010 |      0 |        0 | (0,4)  |           1 |       2816 |     24 |        |      
  5 |   8032 |        1 |     28 |   7010 |      0 |        0 | (0,5)  |           1 |       2816 |     24 |        |      
  6 |   8000 |        1 |     28 |   7010 |      0 |        0 | (0,6)  |           1 |       2816 |     24 |        |      
  7 |   7968 |        1 |     28 |   7010 |      0 |        0 | (0,7)  |           1 |       2816 |     24 |        |      
  8 |   7936 |        1 |     28 |   7010 |      0 |        0 | (0,8)  |           1 |       2816 |     24 |        |      
  9 |   7904 |        1 |     28 |   7010 |      0 |        0 | (0,9)  |           1 |       2816 |     24 |        |      
 10 |   7872 |        1 |     28 |   7013 |      0 |        0 | (0,10) |           1 |       2304 |     24 |        |      
 11 |   7840 |        1 |     28 |   7013 |      0 |        0 | (0,11) |           1 |       2304 |     24 |        |      
 12 |   7808 |        1 |     28 |   7013 |      0 |        0 | (0,12) |           1 |       2304 |     24 |        |      
 13 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
 14 |   7776 |        1 |     28 |   7013 |      0 |        0 | (0,14) |           1 |       2304 |     24 |        |      
 15 |   7744 |        1 |     28 |   7013 |      0 |        0 | (0,15) |           1 |       2304 |     24 |        |      
 16 |   7712 |        1 |     28 |   7013 |      0 |        0 | (0,16) |           1 |       2304 |     24 |        |      
 17 |   7680 |        1 |     28 |   7013 |      0 |        0 | (0,17) |           1 |       2304 |     24 |        |      
 18 |   7648 |        1 |     28 |   7013 |      0 |        0 | (0,18) |           1 |       2304 |     24 |        |      
 19 |   7616 |        1 |     28 |   7013 |      0 |        0 | (0,19) |           1 |       2304 |     24 |        |      
 20 |   7584 |        1 |     28 |   7015 |      0 |        0 | (0,20) |           1 |       2304 |     24 |        |      
 21 |   7552 |        1 |     28 |   7015 |      0 |        0 | (0,21) |           1 |       2304 |     24 |        |      
 22 |   7520 |        1 |     28 |   7015 |      0 |        0 | (0,22) |           1 |       2304 |     24 |        |      
 23 |   7488 |        1 |     28 |   7015 |      0 |        0 | (0,23) |           1 |       2304 |     24 |        |      
 24 |   7456 |        1 |     28 |   7015 |      0 |        0 | (0,24) |           1 |       2304 |     24 |        |      
 25 |   7424 |        1 |     28 |   7015 |      0 |        0 | (0,25) |           1 |       2304 |     24 |        |      
 26 |   7392 |        1 |     28 |   7015 |      0 |        0 | (0,26) |           1 |       2304 |     24 |        |      
 27 |   7360 |        1 |     28 |   7015 |      0 |        0 | (0,27) |           1 |       2304 |     24 |        |      
 28 |   7328 |        1 |     28 |   7015 |      0 |        0 | (0,28) |           1 |       2304 |     24 |        |      
 29 |   7296 |        1 |     28 |   7015 |      0 |        0 | (0,29) |           1 |       2304 |     24 |        |      
(29 rows)
osdba=# insert into test_1 values(4);
INSERT 0 1
osdba=# select * from heap_page_items(get_raw_page('test_1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   7010 |      0 |        0 | (0,1)  |           1 |       2816 |     24 |        |      
  2 |   8128 |        1 |     28 |   7010 |      0 |        0 | (0,2)  |           1 |       2816 |     24 |        |      
  3 |   8096 |        1 |     28 |   7010 |      0 |        0 | (0,3)  |           1 |       2816 |     24 |        |      
  4 |   8064 |        1 |     28 |   7010 |      0 |        0 | (0,4)  |           1 |       2816 |     24 |        |      
  5 |   8032 |        1 |     28 |   7010 |      0 |        0 | (0,5)  |           1 |       2816 |     24 |        |      
  6 |   8000 |        1 |     28 |   7010 |      0 |        0 | (0,6)  |           1 |       2816 |     24 |        |      
  7 |   7968 |        1 |     28 |   7010 |      0 |        0 | (0,7)  |           1 |       2816 |     24 |        |      
  8 |   7936 |        1 |     28 |   7010 |      0 |        0 | (0,8)  |           1 |       2816 |     24 |        |      
  9 |   7904 |        1 |     28 |   7010 |      0 |        0 | (0,9)  |           1 |       2816 |     24 |        |      
 10 |   7872 |        1 |     28 |   7013 |      0 |        0 | (0,10) |           1 |       2304 |     24 |        |      
 11 |   7840 |        1 |     28 |   7013 |      0 |        0 | (0,11) |           1 |       2304 |     24 |        |      
 12 |   7808 |        1 |     28 |   7013 |      0 |        0 | (0,12) |           1 |       2304 |     24 |        |      
 13 |   7264 |        1 |     28 |   7016 |      0 |        0 | (0,13) |           1 |       2048 |     24 |        |      
 14 |   7776 |        1 |     28 |   7013 |      0 |        0 | (0,14) |           1 |       2304 |     24 |        |      
 15 |   7744 |        1 |     28 |   7013 |      0 |        0 | (0,15) |           1 |       2304 |     24 |        |      
 16 |   7712 |        1 |     28 |   7013 |      0 |        0 | (0,16) |           1 |       2304 |     24 |        |      
 17 |   7680 |        1 |     28 |   7013 |      0 |        0 | (0,17) |           1 |       2304 |     24 |        |      
 18 |   7648 |        1 |     28 |   7013 |      0 |        0 | (0,18) |           1 |       2304 |     24 |        |      
 19 |   7616 |        1 |     28 |   7013 |      0 |        0 | (0,19) |           1 |       2304 |     24 |        |      
 20 |   7584 |        1 |     28 |   7015 |      0 |        0 | (0,20) |           1 |       2304 |     24 |        |      
 21 |   7552 |        1 |     28 |   7015 |      0 |        0 | (0,21) |           1 |       2304 |     24 |        |      
 22 |   7520 |        1 |     28 |   7015 |      0 |        0 | (0,22) |           1 |       2304 |     24 |        |      
 23 |   7488 |        1 |     28 |   7015 |      0 |        0 | (0,23) |           1 |       2304 |     24 |        |      
 24 |   7456 |        1 |     28 |   7015 |      0 |        0 | (0,24) |           1 |       2304 |     24 |        |      
 25 |   7424 |        1 |     28 |   7015 |      0 |        0 | (0,25) |           1 |       2304 |     24 |        |      
 26 |   7392 |        1 |     28 |   7015 |      0 |        0 | (0,26) |           1 |       2304 |     24 |        |      
 27 |   7360 |        1 |     28 |   7015 |      0 |        0 | (0,27) |           1 |       2304 |     24 |        |      
 28 |   7328 |        1 |     28 |   7015 |      0 |        0 | (0,28) |           1 |       2304 |     24 |        |      
 29 |   7296 |        1 |     28 |   7015 |      0 |        0 | (0,29) |           1 |       2304 |     24 |        |      
(29 rows)


osdba=# select ctid,* from test_1 where id = 4;
  ctid  | id 
--------+----
 (0,13) |  4
 (0,23) |  4
(2 rows)

osdba=# vacuum full test_1;
VACUUM
osdba=# select ctid,* from test_1 where id = 4;
  ctid  | id 
--------+----
 (0,13) |  4
 (0,23) |  4
(2 rows)

osdba=# select * from heap_page_items(get_raw_page('test_1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   7010 |      0 |        0 | (0,1)  |           1 |       2816 |     24 |        |      
  2 |   8128 |        1 |     28 |   7010 |      0 |        0 | (0,2)  |           1 |       2816 |     24 |        |      
  3 |   8096 |        1 |     28 |   7010 |      0 |        0 | (0,3)  |           1 |       2816 |     24 |        |      
  4 |   8064 |        1 |     28 |   7010 |      0 |        0 | (0,4)  |           1 |       2816 |     24 |        |      
  5 |   8032 |        1 |     28 |   7010 |      0 |        0 | (0,5)  |           1 |       2816 |     24 |        |      
  6 |   8000 |        1 |     28 |   7010 |      0 |        0 | (0,6)  |           1 |       2816 |     24 |        |      
  7 |   7968 |        1 |     28 |   7010 |      0 |        0 | (0,7)  |           1 |       2816 |     24 |        |      
  8 |   7936 |        1 |     28 |   7010 |      0 |        0 | (0,8)  |           1 |       2816 |     24 |        |      
  9 |   7904 |        1 |     28 |   7010 |      0 |        0 | (0,9)  |           1 |       2816 |     24 |        |      
 10 |   7872 |        1 |     28 |   7013 |      0 |        0 | (0,10) |           1 |       2816 |     24 |        |      
 11 |   7840 |        1 |     28 |   7013 |      0 |        0 | (0,11) |           1 |       2816 |     24 |        |      
 12 |   7808 |        1 |     28 |   7013 |      0 |        0 | (0,12) |           1 |       2816 |     24 |        |      
 13 |   7776 |        1 |     28 |   7016 |      0 |        0 | (0,13) |           1 |       2816 |     24 |        |      
 14 |   7744 |        1 |     28 |   7013 |      0 |        0 | (0,14) |           1 |       2816 |     24 |        |      
 15 |   7712 |        1 |     28 |   7013 |      0 |        0 | (0,15) |           1 |       2816 |     24 |        |      
 16 |   7680 |        1 |     28 |   7013 |      0 |        0 | (0,16) |           1 |       2816 |     24 |        |      
 17 |   7648 |        1 |     28 |   7013 |      0 |        0 | (0,17) |           1 |       2816 |     24 |        |      
 18 |   7616 |        1 |     28 |   7013 |      0 |        0 | (0,18) |           1 |       2816 |     24 |        |      
 19 |   7584 |        1 |     28 |   7013 |      0 |        0 | (0,19) |           1 |       2816 |     24 |        |      
 20 |   7552 |        1 |     28 |   7015 |      0 |        0 | (0,20) |           1 |       2816 |     24 |        |      
 21 |   7520 |        1 |     28 |   7015 |      0 |        0 | (0,21) |           1 |       2816 |     24 |        |      
 22 |   7488 |        1 |     28 |   7015 |      0 |        0 | (0,22) |           1 |       2816 |     24 |        |      
 23 |   7456 |        1 |     28 |   7015 |      0 |        0 | (0,23) |           1 |       2816 |     24 |        |      
 24 |   7424 |        1 |     28 |   7015 |      0 |        0 | (0,24) |           1 |       2816 |     24 |        |      
 25 |   7392 |        1 |     28 |   7015 |      0 |        0 | (0,25) |           1 |       2816 |     24 |        |      
 26 |   7360 |        1 |     28 |   7015 |      0 |        0 | (0,26) |           1 |       2816 |     24 |        |      
 27 |   7328 |        1 |     28 |   7015 |      0 |        0 | (0,27) |           1 |       2816 |     24 |        |      
 28 |   7296 |        1 |     28 |   7015 |      0 |        0 | (0,28) |           1 |       2816 |     24 |        |      
 29 |   7264 |        1 |     28 |   7015 |      0 |        0 | (0,29) |           1 |       2816 |     24 |        |      
(29 rows)

osdba=# 

```

























## 链接地址

https://yq.aliyun.com/articles/2291

https://www.postgresql.org/docs/current/static/pageinspect.html?spm=a2c4e.11153940.blogcont2291.5.36292b2bYnZkIy



