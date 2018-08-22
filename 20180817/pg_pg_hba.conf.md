[TOC]

# pg pg_hba.conf

​	

​	今天在TJ遇到一个场景，就是kettle的资源库是postgresql数据库，而kettle服务器是在某一台服务器，当时没有记录相关信息，而且在那台服务器上配置了调度计划(windows计划任务)，想到了在pg_hba.conf限制ip的访问。



​	pg_hba.conf是访问控制配置文件

​	在postgresql中，运行那些IP的机器访问数据库服务器是由pg_hba.conf文件控制的，HBA的意思是host-based authentication,也就是基于主机的认证。

## pg_hba.conf

​	initdb初始化数据目录时，会生成一个默认的pg_hba.conf文件。

​	pg_hba.conf文件的格式由很多记录组成，每条记录占一行

​	一条记录是由若干个空格或制表符分割的字段组成，如果字段用引号包围，那么它可以包含空白

​	每条记录生命一种连接类型，一个客户端ip地址范围(如果和连接类型相关)，一个数据库，一个用户名字，以及队匹配这些参数的连接所使用的认证方法。第一条匹配连接类型，客户端地址，连接请求的数据库名和用户名的记录将用于执行认证。

​	

​	

​		


