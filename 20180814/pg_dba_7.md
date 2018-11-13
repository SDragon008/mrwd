[TOC]

# pg dba 7



​	postgresql 备份 还原

​	

## 数据库热备份与还原



### 物理备份与还原

备份$PGDATA,归档文件，以及所有表空间目录,适用于跨小版本的恢复，但是不能跨平台

需要开启归档

**目前PG还不支持基于数据文件数据块变更的增量备份，仅仅支持数据文件+归档的备份的备份方式**



### 数据库物理备份开启



#### 开启归档

首先要开启归档，日志模式>=`archive`,步骤如下

```
$ vim postgresql.conf
wal_level = hot_standby                 # minimal, archive, or hot_standby
                                        # (change requires restart)
```

**创建归档目录**

```
# mkdir -p /home/backup/arch
# chown -R ysys:ysys /home/backup/arch
```

**配置归档命令**

* %p 表示xlog文件名 $PGDATA的相对路径，如`pg_xlog/000000010000000000000005`
* %f 表示xlog文件名，如`000000010000000000000005`

```
# vim postgresql.conf

archive_mode = on               # allows archiving to be done
                                # (change requires restart)
archive_command = 'DATE=`date+%Y%m%d`;DIR="/home/backup/arch/$DATE";(test -d $DIR||mkdir -p $DIR)&& cp %p $DIR/%f'   
```

```
$ psql
psql (9.2.4)
Type "help" for help.

ysys=# checkpoint;
CHECKPOINT
ysys=# select pg_switch_xlog();
 pg_switch_xlog 
----------------
 0/30D84A8
(1 row)

ysys=# \q
```

**测试归档是否正常**

```
$ ls -ls /home/backup/arch/
total 16384
16384 -rw------- 1 ysys ysys 16777216 Nov  6 15:07 000000010000000000000003
```



​	归档完了之后可以在数据库异常时(wal日志损害),是可以恢复数据库。但是想要创建一个真正意义的备份，就需要创建一个基础备份，基础备份+归档日志就可以做一个简单数据库备份；基础备份相当于很

#### 创建备份

​	远程和本地都是可以创建备份，在本地可以使用`dba`用户来创建备份

​	可以使用流复制协议来备份，或者执行命令拷贝当前文件。











## 逻辑备份与还原

 备份数据



