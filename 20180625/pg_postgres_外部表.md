
## pg_postgres_外部表

pg和pg跨库查询，既可以通过dblink,也可以通过postgres_外部表，本章节先讲一下postgres_外部表

### 配置步骤

本身就是postgresql数据库，所以对于自己的支持还是比较大，建议使用源码安装，这样就可以想file_fdw执行一样简单

- 编译postgres_fdw

```

$ cd postgres_fdw

$ make

$ make install

```

- 创建extension

```
osdba=# create extension postgres_fdw;
CREATE EXTENSION
```

- 创建server

```


```

- 创建user

- 创建foreign table 