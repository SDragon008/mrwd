# postgresql 外部表

​	postgresql通过外部表可以访问非常多的数据库，如关系型数据库oracle,mysql,postgresql本身，也可以访问nosql数据库 redis和mongo,其他支持的数据库
	需要到官网文档查找，在本次文档整理中，目前整理postgresql，oracle,mysql;mongo,redis;file这几类，后期可能还会继续整理这一章节



​	不兼容

​	有些版本的fdw未必能和postgresql当前版本兼容，这个要注意，后期部署时会将版本信息说明处理

​        [pg_fdw_不兼容](pg_fdw_不兼容.md)



链接地址:

file:

[pg_file_外部表](pg_file_外部表.md)

rdbms:

[pg_mysql_外部表](pg_mysql_外部表.md)

[pg_oracle_外部表](pg_oracle_外部表.md)

[pg_postgres_外部表](pg_postgres_外部表.md)

nosql:

[pg_mongo_外部表](pg_mongo_外部表.md)

[pg_redis_外部表](pg_redis_外部表.md)

