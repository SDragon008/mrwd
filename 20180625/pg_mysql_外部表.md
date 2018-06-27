
## pg_mysql_外部表

- 环境

本次测试环境是在同一台服务器，它的信息分别是（centos6.5_x64,1.5g内存，1核cpu,30g磁盘空间）,软件（mysql5.7.17，postgres 9.4.1)

### mysql_fdw 安装步骤

mysql_fdw文件依赖mysql的lib包，为了更好的安装部署mysql_fdw,建议在本地安装一套mysql数据库，或者拷贝测试通过压缩包(就是将mysql数据库安装完成后重新打的包)

本次将以压缩包为例讲解

- 下载压缩包

百度链接：

