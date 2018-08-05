## linux postgres installing from source

### 源码安装

#### 前期准备

&ensp; &ensp;下载路径:http://www.postgresql.org/ftp/source/

&ensp; &ensp;下载包：postgresql-9.6.5.tar.gz

&ensp; &ensp;操作系统:centos6.5x64

#### 安装步骤

1. 关闭防火墙

&ensp; &ensp;[root@pg96 ~]# chkconfig --list iptables;

2. 创建文件夹并使用xftp上传文件

&ensp; &ensp;[root@pg96 /]# mkdir /software

3. 解压文件

&ensp; &ensp;[root@pg96 software]# tar zxvf postgresql-9.6.5.tar.gz 

4. 进入目录并配置postgres安装目录

&ensp; &ensp;[root@pg96 software]# cd postgresql-9.6.5 

&ensp; &ensp;[root@pg96 postgresql-9.6.5]# ./configure --prefix=/pgsql

5. 源码编译

&ensp; &ensp;[root@pg96 postgresql-9.6.5]# make

6. 安装

&ensp; &ensp;[root@pg96 postgresql-9.6.5]# make install

7. 创建用户和组

&ensp; &ensp;groupadd postgres

&ensp; &ensp;useradd -g postgres postgres

&ensp; &ensp;cd /pgsql/

&ensp; &ensp;mkdir data/

&ensp; &ensp;chown postgres:postgres data/

8. 切换用户并设置环境变量

&ensp; &ensp;su - postgres

&ensp; &ensp;vi .bash_profile

&ensp; &ensp;export PGHOME=/pgsql

&ensp; &ensp;export PATH=$PGHOME/bin/:$PATH

&ensp; &ensp;export PGDATA=/pgsql/data

&ensp; &ensp;source .bash_profile

9. 初始化数据库

&ensp; &ensp;/pgsql/bin/initdb  -E UNICODE -D /pgsql/data


10. 创建日志文件 

&ensp; &ensp;[postgres@pg96 data]$ touch pgsql.log


11. 创建系统自启动服务文件

&ensp; &ensp;[root@pg96 pgsql]# cp /opt/software/postgresql-8.3.10/contrib/start-scripts/linux /etc/rc.d/init.d/postgresql

&ensp; &ensp;[root@pg96 pgsql]# chmod u+x /etc/rc.d/init.d/postgresql


12. 修改自启动服务文件的参数

&ensp; &ensp;[root@pg96 pgsql]# vi /etc/rc.d/init.d/postgresql

&ensp; &ensp;prefix=/pgsql

&ensp; &ensp;PGDATA="/pgsql/data"

&ensp; &ensp;PGUSER=postgres

&ensp; &ensp;PGLOG="$PGDATA/pgsql.log"

13. 添加自启动服务并启动数据库服务

&ensp; &ensp;[root@pg96 pgsql]# chkconfig --add postgresql

&ensp; &ensp;[root@pg96 pgsql]# service postgresql start

14. 数据库连接

&ensp; &ensp; su - postgres

&ensp; &ensp; psql


**错误**

源码安装前configure经常会报错，报错信息会在config.log中，本次报错有两个第一个是gcc包没有找到，第二个是readline没有找到；
分别执行 yum -y install gcc;yum -y install readline-devel



