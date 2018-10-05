[TOC]



# kettle 总结

kettle是一款开源的etl软件，非常适合小型公司数据抽取，转换，整合，加工，清洗，报表的软件。



## 未完成

- [kettle 支持 greenplum gpload加载数据](../20170504/kettle_greenplum_gpload.md)
- [kettle 支持 greenplum bulk loader 加载数据](../20170504/kettle_greenplum_bulk_loader.md)
- [kettle oracle udpate then insert 效率](../20170504/kettle_oracle_update_then_insert.md)
- [kettle window 迁移 linux](../20170504/kettle_迁移.md)
- [kettle cdc](../20170504/kettle_cdc.md)
- [kettle 多线程和单线程](../20170504/kettle_multiprocess_or_Single_Process.md)
- [kettle配置数据库链接两种方式](../20180615/kettle配置数据库链接两种方式.md)
- [kettle配置oracle集群](../20180615/kettle配置oracle集群.md)
- [kettle配置数据库资源库](../20180615/kettle配置数据库资源库.md)
- [kettle file 文件解析](../20170504/kettle_file_analysis.md)
- [kettle 一般报错](../20170504/kettle_errors.md)
- [kettle job 空跑](../20170504/kettle_job_避免空跑.md)
- [kettle 慢](../20170504/kettle_确定_效率慢_方法.md)
- [kettle复制同样结构的表](../20170504/kettle_copy_lot's_table.md)
- [kettle下载ftp文件](../20170504/kettle下载ftp文件.md)
- [kettle获取远程文件并下载](../20170504/kettle获取远程文件并下载.md)
- [kettle_mysql配置数据库资源库](../20170504/kettle_mysql_配置数据库资源库.md)
- [kettle 配置oracle集群](../img_src/kettle配置oracle集群方式.md)
- [kettle json 17M 文件解析](../20170504/kettle_17Mjson文件解析.md)
- [kettle gpload](../20170504/KETTLE_GPLOAD_LOAD.md)
- [wps_excel_超多行数据导入](../20180820/wps_excel_超多行数据导入.md)



## 未整理

- [kettle 支持 greenplum gpload加载数据](../20170504/kettle_greenplum_gpload.md)
- [kettle 支持 greenplum bulk loader 加载数据](../20170504/kettle_greenplum_bulk_loader.md)
- [kettle oracle udpate then insert 效率](../20170504/kettle_oracle_update_then_insert.md)
- [kettle window 迁移 linux](../20170504/kettle_迁移.md)
- [kettle cdc](../20170504/kettle_cdc.md)
- [kettle 多线程和单线程](../20170504/kettle_multiprocess_or_Single_Process.md)

- [kettle配置数据库链接两种方式](../20180615/kettle配置数据库链接两种方式.md)
- [kettle配置oracle集群](../20180615/kettle配置oracle集群.md)
- [kettle配置数据库资源库](../20180615/kettle配置数据库资源库.md)
- [kettle file 文件解析](../20170504/kettle_file_analysis.md)
- [kettle 一般报错](../20170504/kettle_errors.md)

- [kettle job 空跑](../20170504/kettle_job_避免空跑.md)
- [kettle 慢](../20170504/kettle_确定_效率慢_方法.md)
- [kettle复制同样结构的表](../20170504/kettle_copy_lot's_table.md)
- [kettle下载ftp文件](../20170504/kettle下载ftp文件.md)
- [kettle获取远程文件并下载](../20170504/kettle获取远程文件并下载.md)
- [kettle_mysql配置数据库资源库](../20170504/kettle_mysql_配置数据库资源库.md)
- [kettle 配置oracle集群](../img_src/kettle配置oracle集群方式.md)
- [kettle json 17M 文件解析](../20170504/kettle_17Mjson文件解析.md)
- [kettle gpload](../20170504/KETTLE_GPLOAD_LOAD.md)




## kettle 集群

- 安装流程

[kettle_集群](../20180626/kettle_集群.md)

## kettle 数据库迁移

- oracle对oracle整库迁移

[关于kettle将oracle数据库整体迁移到另外oracle数据库方案](../20180622/关于kettle将oracle数据库整体迁移到另外oracle数据库方案.md)


## kettle for 循环

- job

- js



### kettle 配置同一个库下n张表抽取

- 删除后插入

[kettle配置一次性抽取n张表](../20180616/kettle配置一次性抽取n张表.md)

- 更新后插入

oracle merge update insert

[kettle配置一次性抽取n张表(二)](../20180619/kettle配置一次性抽取n张表(二).md)


其实配置n张表都是依赖于**kettle for 循环**的job设计


