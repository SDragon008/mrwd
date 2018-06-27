﻿
## pg_file_外部表


### file_外部表

file_外部表可以很方便的把外部文本文件映射成一张外部表，当前此类外部表是只读的。

- 定义extension

```
create extension file_fdw;

```

postgreql安装部署建议使用源码安装，这样就会在安装包中有contrib/file_fdw文件信息，make make install后就可以在数据库执行file_fdw

- 创建server

```

create server file_fdw_server FOREIGN DATA WRAPPER file_fdw 

```

- 创建file_外部表

```

CREATE FOREIGN TABLE passwd( username text,pass text,uid int4,gid int4,gecos text,home text,shell text)SERVER file_fdw_server OPTIONS(format 'text',filename '/etc/passwd',delimiter ':',null '');

```

-----------

filename:制定外部文件名

format:指定文件的格式，与COPY命令中的format选项相同

header:指定文件是否有行头，与COPY命令中HEADER选项相同

delimiter:指定分割字符，与COPY命令中的DELIMITER选项相同

quote:指定字符串的包裹字符，与COPY命令中的QUOTE选项系统

escape:指定的转义字符，与COPY命令中的ESCAPE选项相同

null:指定为“空”字符串，与COPY命令中的NULL选项系统

encoding:指定文件的字符集编码，与COPY命令中的encoding选项相同

实际上file_fdw是通过COPY API来访问外部文本文件，所以file_fdw的选项除了filename外都与COPY命令相同。

-----------


### error

- 无法创建 file_fdw

```
osdba=# create extension file_fdw;
ERROR:  could not open extension control file "/usr/local/pgsql/share/extension/file_fdw.control": No such file or directory

```

需要在contrib/file_fdw下make ,make install执行


```
$ cd file_fdw
$ make
$ make install
$ psql
osdba=# create extension file_fdw;
CREATE EXTENSION


```