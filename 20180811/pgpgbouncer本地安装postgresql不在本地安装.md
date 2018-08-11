[TOC]

# pg pgbouncer 本地安装 postgresql 不在本地安装



​	一直以来，认为pgbouncer和postgresql数据库要一起安装，pgbouncer才能够使用。之前自己学习和认知不够，一直对此有误解，现在经过测试，pgbouncer本地安装，不需要安装postgresql也可以使用。



## 环境介绍



​	软件：pgbouncer-1.5.5.tar.gz，libevent-2.0.22-stable.tar.gz



## 适用场景



​	1、访问用户非常多，且都是多连接

​	2、不适合在安装数据库的服务器上再次安装其他软件(容易对数据库软件误操作)





## 说明



#:如果没有特别声明，代表root用户

$:如果没有特别声明，代表普通用户

{user}:代表安装postgres的用户



本次postgres安装的用户为ysys



## 安装

### yum 安装依赖包

```
# yum -y install readline-devel perl-ExtUtils-Embed bison flex zlib zlib-devel python python-devel gcc
```



### 上传文件并解压

```
 # tar xvf libevent-2.0.22-stable.tar.gz  
 # tar xvf pgbouncer-1.5.5.tar.gz  
```

### libevent安装

```
 # ./configure --prefix=/home/{user}/libevent
 # make
 # make install
```

### pgbouncer 安装

```
# ./configure --prefix=/home/{user}/pgbouncer/ --with-libevent=/home/{user}/libevent/
# make
# make install
```

### 授权用户{user}对安装目录的权限

```
 # chown -R ysys:ysys /home/ysys/pgbouncer/
 # chown -R ysys:ysys /home/ysys/libevent/
```

### 配置{user}环境变量并生效

```
$ vim .bash_profile
export LD_LIBRARY_PATH=/home/ysys/libevent/lib:$LD_LIBRARY_PATH
```

```
$ source .bash_profile
```

### 在pgbouncer安装目录下创建目录并拷贝配置文件修改

```
$ cd /home/ysys/pgbouncer/
$ mkdir etc
$ mkdir log
$ cp pgbouncer-1.5.5/etc/pgbouncer.ini /home/ysys/pgbouncer/etc/
$ cd /home/ysys/pgbouncer/etc/
$ vim pgbouncer.ini

[databases]
db45 = host=192.168.1.45 port=5432 dbname=tutorial client_encoding=sql_ascii datestyle=ISO pool_size=20
[pgbouncer]
pool_mode= transaction
listen_port= 6543
listen_addr= 0.0.0.0
auth_type= md5
auth_file= /home/ysys/pgbouncer/etc/users.txt
logfile= /home/ysys/pgbouncer/log/pgbouncer.log
pidfile= /home/ysys/pgbouncer/etc/pgbouncer.pid
unix_socket_dir= /home/ysys/pgbouncer/etc
admin_users= pgadmin
stats_users= pgmon
server_reset_query= DISCARD ALL
server_check_query= select 1
server_check_delay= 30
max_client_conn= 50000
default_pool_size= 20
reserve_pool_size= 5
dns_max_ttl= 15 --可以不添加
```

本次并没有安装dns相关的内容

### 编辑密码文件

```
$ cat users.txt 
"ysys" "md5d9832a8c4b37f5aee2d27146cb212008"
"pgmon" "123mon"
"pgadmin" "123admin"
```

ysys:指的是postgres数据库用户,它的密码来自于(pg_shadow的passwd)

pgmon:admin_users

pgadmin:stats_users

pgmon和pgadmin查看pgbouncer信息用户，密码是自己设置的



## 启动 关闭



### 启动

```
$ /home/ysys/pgbouncer/bin/pgbouncer -d /home/ysys/pgbouncer/etc/pgbouncer.ini 
```

### 关闭

```
$ cat /home/ysys/pgbouncer/etc/pgbouncer.pid
$ kill {number}
```

### 重启

```
$ /home/ysys/pgbouncer/bin/pgbouncer -R /home/ysys/pgbouncer/etc/pgbouncer.ini  
```



## 访问

### 访问pgbouncer

需要在其他postgres服务器才能执行

```
$ psql -h {pgbouncer_ip} -p 6543 -U pgadmin pgbouncer
$ psql -h {pgbouncer_ip} -p 6543 -U pgmon pgbouncer
```



### 访问数据库

需要在其他postgres服务器才能执行

```
$ psql -h {pgbouncer_ip} -p 6543 -U ysys -d guohui
```



## 备注

​	默认pgbouncer不能直接通过root用户启动，需要一个普通用户启动关闭或者重启才可以，所以本次安装部署时依然采用ysys用户