[TOC]

# pg pgbouncer install



## 前言

​	每次应用程序和postgres连接，都会克隆出一个服务进程来为应用程序服务，在频繁的创建和销毁进程，会耗费比较多的资源。而pgbouncer会把与后端postgresql数据库的连接缓存住，当有前端请求时，只为分配一个空闲的连接给前端程序使用，这样就降低了资源的消耗。

​	

## 环境

软件：pgbouncer-1.5.5.tar.gz，libevent-2.0.22-stable.tar.gz

 安装必须在已经安装好的postgresql

 **安装**

1. 

2.  	**上传软件到****opt****并解压**

 \# tar xvf libevent-2.0.22-stable.tar.gz  

 \# tar xvf pgbouncer-1.5.5.tar.gz  

1. 

2.  	**安装****libevent**

 \# ./configure --prefix=/home/postgres/libevent

 \# make

 \# make install

1. 

2.  	**安装****pgbouncer**

 \# ./configure --prefix=/home/postgres/pgbouncer/ --with-libevent=/home/postgres/libevent/

 \# make

 \# make install

 **4****、配置****.bash_profile,****修改文件权限**

 \# cp /opt/pgbouncer-1.5.5/etc/pgbouncer.ini /home/postgres/pgbouncer/

 \# chown -R postgres:postgres pgbouncer/

 \# chown -R postgres:postgres libevent/

 \# su - postgres

 $ vim .bash_profile

 export LD_LIBRARY_PATH=/home/postgres/libevent/lib:$LD_LIBRARY_PATH

 **5****、修改配置文件****pgbouncer.ini****文件并参考该文件要求创建****userlist.txt**

 $ vim pgbouncer.ini

 [databases]

 postgres = host=localhost port=5432 dbname=postgres user=postgres password=postgres connect_query='SELECT 1'

 [pgbouncer]

 logfile = /home/postgres/pgbouncer/pgbouncer.log

 pidfile = /home/postgres/pgbouncer/pgbouncer.pid

 listen_addr = *

 listen_port = 6432

 auth_type = trust

 auth_file = /home/postgres/pgbouncer/userlist.txt

 pool_mode = transaction

 server_reset_query = DISCARD ALL

 max_client_conn = 100

 default_pool_size = 20

 $ vim userlist.txt

 "postgres" "postgres"

 **6****、启动****pgbouncer**

 ./pgbouncer  -d /home/postgres/pgbouncer/pgbouncer.ini

 **7****、登录**

 $ psql -p 6432 postgres

 **8****、停止****pgbouncer**

 $ cat /home/postgres/pgbouncer/pgbouncer.pid

 $ kill number

 **9****、查看连接池信息**

 $ psql -p 6432 pgbouncer

 =# show help;

 NOTICE:  Console usage

 DETAIL:   

 	SHOW HELP|CONFIG|DATABASES|POOLS|CLIENTS|SERVERS|VERSION

 	SHOW STATS|FDS|SOCKETS|ACTIVE_SOCKETS|LISTS|MEM

 	SHOW DNS_HOSTS|DNS_ZONES

 	SET key = arg

 	RELOAD

 	PAUSE [<db>]

 	RESUME [<db>]

 	KILL <db>

 	SUSPEND

 	SHUTDOWN

 SHOW

 =# show clients;

 =# show pools;

 **报错信息**

 error 1:

 [postgres@postgres93 bin]$ ./pgbouncer -d /home/postgres/pgbouncer/pgbouncer.ini  

 ./pgbouncer: error while loading shared libraries: libevent-2.0.so.5: cannot open shared object file: No such file or directory

 [solution]

 \# su - postgres

 $ vim .bash_profile

 export LD_LIBRARY_PATH=/home/postgres/libevent/lib:$LD_LIBRARY_PATH

 error 2:

 [postgres@postgres93 ~]$ psql -p 6432 pgbouncer

 psql: ERROR:  not allowed

 [solution]

 $ vim pgbouncer.init

 admin_users = postgres

 需要重启pgbouncer

 $ psql -p 6432 pgbouncer

 参考文档：

 http://pgbouncer.github.io/downloads/

 <https://yq.aliyun.com/articles/43328>

 http://www.bubuko.com/infodetail-1203143.html

 http://blog.csdn.net/lk_db/article/details/77939005?locationNum=8&fps=1