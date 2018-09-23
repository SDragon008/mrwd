[TOC]

# redis cluster install

**jl**

ysys

**rq**

2018-09-17

**bq**

redis,redis cluster,install



## 背景介绍

​	在TJ形成一套体系后，现场这边需要部署一套redis集群(tj的redis集群是由华三部署)，从支持部CQL这边要到信息后，准备进行部署安装，顺带要了安装的链接地址(https://www.cnblogs.com/yingchen/p/6763524.html)，后期参考官网文档(https://redis.io/topics/cluster-tutorial)



## REDIS 介绍

​	后期整理

​	

## REDIS 集群介绍

​	后期整理



## REDIS 集群部署

​	

```
# make
# make install PREFIX=/usr/andy/redis-cluster
```













## 错误

1、	make 编译报错

```
# make
cd src && make all
make[1]: Entering directory `/software/redis-3.2.12/src'
    CC adlist.o
In file included from adlist.c:34:
zmalloc.h:50:31: error: jemalloc/jemalloc.h: No such file or directory
zmalloc.h:55:2: error: #error "Newer version of jemalloc required"
make[1]: *** [adlist.o] Error 1
make[1]: Leaving directory `/software/redis-3.2.12/src'
make: *** [all] Error 2
```

​	执行命令

```
# make MALLOC=libc
```







## 链接地址

https://www.cnblogs.com/yingchen/p/6763524.html

https://blog.csdn.net/fengshizty/article/details/51368004

https://blog.csdn.net/yejingtao703/article/details/78484151

https://redis.io/topics/cluster-tutorial

https://blog.csdn.net/wzygis/article/details/51705559

https://blog.csdn.net/dream_an/article/details/77199462