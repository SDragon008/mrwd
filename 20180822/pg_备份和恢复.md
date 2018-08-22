[TOC]

# pg 备份和恢复

​	数据都是有价值的(生产数据)，postgresql数据库应当定期的备份，虽然过程较为简单，但清晰的理解其底层技术和假设是相当重要的。

​	有三种不同的方式来备份postgresql数据。

​	sql dump

​	file system level backup

​	continuous archiving and point-in-time recovery (pitr)

​	每一种都有优势和不足；我们在下面的环节讨论这些问题

## sql dump

​	sql dump 的思想是创建一个由SQL命令组成的文件，当把这个文件回馈给服务器时，服务器将利用其中的SQL命令重建与dump状态一样的数据库。postgresql为此提供了工具pg_dump。这个工具的基本用法是:

```
pg_dump dbname > dbname_bak.sql
```

​	pg_dump把结果输出到标准输出。尽管上述命令会创建一个文本文件,pg_dump可以用其他格式创建文件以支持并行和细粒度的对象恢复控制

​	pg_dump是一个普通的postgresql客户端应用。这意味着你可以在任何可以访问该数据库的远端主机上进行备份工作。

