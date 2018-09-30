[TOC]

# redis cluster install multiple

**文档整理**

ysys

**日期**

2018-09-28

**标签**

redis,redis cluster,multiple install	



## 背景介绍

​	之前在GZ规划时，规划了三台服务器作为redis集群的部署，前一段时间在一台虚拟机上虚拟了集群环境，现在在三台虚拟机上虚拟一下环境。



## 环境

| 服务器ip     | 操作系统  | 主机名 |
| ------------ | --------- | ------ |
| 192.168.1.15 | centos6.5 | gh15   |
| 192.168.1.16 | centos6.5 | gh16   |
| 192.168.1.17 | centos6.5 | gh17   |





## 部署安装



### 每台服务器都执行一遍

```
# vim /etc/hosts
192.168.1.15 gh15
192.168.1.16 gh16
192.168.1.17 gh17
# mkdir /software
# cd /software
# tar -zxvf redis-3.2.12.tar.gz 
# cd /etc/yum.repos.d/
# mv * /software/
# cd /etc/yum.repos.d/
# vim ysys.repo
# yum -y install gcc*
# make 
# make install PREFIX=/usr/local/redis-cluster
```

### 其他操作

分别在服务器gh15,gh16,gh17上创建目录并且将redis.conf创建并修改

```
[gh15]
# cd /usr/local/redis-cluster/
# mkdir 7000 7001
[gh16]
# cd /usr/local/redis-cluster/
# mkdir 7002 7003
[gh17]
# cd /usr/local/redis-cluster/
# mkdir 7004 7005
```

分别在7000,70001,7002,7003,7004,70005目录下创建redis.conf，并且按照要求修改ip地址和port端口

```
# cd /usr/local/redis-cluster
# vim redis.conf

bind 192.168.1.15
daemonize    yes                          
pidfile  /var/run/redis_7000.pid          
port  7000                               
cluster-enabled  yes                      
cluster-config-file  nodes_7000.conf      
cluster-node-timeout  5000                
appendonly  yes

```

**三台节点启动redis服务**

```
[gh15]
# cd /usr/local/redis-cluster/bin
# ./redis-server ../7000/redis.conf
# ./redis-server ../7001/redis.conf

[gh16]
# cd /usr/local/redis-cluster/bin
# ./redis-server ../7002/redis.conf
# ./redis-server ../7003/redis.conf

[gh17]
# cd /usr/local/redis-cluster/bin
# ./redis-server ../7004/redis.conf
# ./redis-server ../7005/redis.conf
```

**节点检查redis服务**

```
# ps -ef|grep redis
```

### 创建集群



在每台服务器上执行命令

```
# yum -y install  ruby ruby-devel rubygems rpm-build 
```

如果不成功，可能缺少一些依赖包，下面的是我用其他方式测试redis安装，可以使用

rpm依赖包(centos6.5)

链接：https://pan.baidu.com/s/1eY8SBKl-Vmeu1XNQPH1qbg 
提取码：br0n

```
# cd /software/
# gem install redis-3.2.2.gem
```



#### 复制解压文件src的redis-trib.rb文件到redis-cluster下 

```
# cd /usr/local/redis-cluster/bin
# cp /software/redis-3.2.12/src/redis-trib.rb ./
```



#### 测试redis-trib.rb

```
# redis-trib.rb 
Usage: redis-trib <command> <options> <arguments ...>

  set-timeout     host:port milliseconds
  add-node        new_host:new_port existing_host:existing_port
                  --slave
                  --master-id <arg>
  fix             host:port
                  --timeout <arg>
  import          host:port
                  --from <arg>
                  --copy
                  --replace
  help            (show this help)
  call            host:port command arg arg .. arg
  reshard         host:port
                  --to <arg>
                  --from <arg>
                  --timeout <arg>
                  --slots <arg>
                  --pipeline <arg>
                  --yes
  create          host1:port1 ... hostN:portN
                  --replicas <arg>
  check           host:port
  del-node        host:port node_id
  rebalance       host:port
                  --weight <arg>
                  --timeout <arg>
                  --use-empty-masters
                  --threshold <arg>
                  --pipeline <arg>
                  --simulate
                  --auto-weights
  info            host:port

For check, fix, reshard, del-node, set-timeout you can specify the host and port of any working node in the cluster.
```

### 启动集群

```
# ./redis-trib.rb create --replicas 1 192.168.1.15:7000 192.168.1.15:7001 192.168.1.16:7002 192.168.1.16:7003 192.168.1.17:7004 192.168.1.17:7005 
>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
192.168.1.17:7004
192.168.1.16:7002
192.168.1.15:7000
Adding replica 192.168.1.16:7003 to 192.168.1.17:7004
Adding replica 192.168.1.17:7005 to 192.168.1.16:7002
Adding replica 192.168.1.15:7001 to 192.168.1.15:7000
M: aed9f883906406839aad3162d46c8840ab1db33d 192.168.1.15:7000
   slots:10923-16383 (5461 slots) master
S: 9eae0f6dd599fc708ce318f749d8be441e6f2262 192.168.1.15:7001
   replicates aed9f883906406839aad3162d46c8840ab1db33d
M: 73ed7c5e65c524dd8a98f56e61113ae2c207e856 192.168.1.16:7002
   slots:5461-10922 (5462 slots) master
S: e7d536e855da0098fef58985dc4e497a4bb55b78 192.168.1.16:7003
   replicates 62148456ec17469a36ae19bfe684fc343b4ae2fd
M: 62148456ec17469a36ae19bfe684fc343b4ae2fd 192.168.1.17:7004
   slots:0-5460 (5461 slots) master
S: 763d115cab452a5d86902229e4356efab6b4676e 192.168.1.17:7005
   replicates 73ed7c5e65c524dd8a98f56e61113ae2c207e856
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join..
>>> Performing Cluster Check (using node 192.168.1.15:7000)
M: aed9f883906406839aad3162d46c8840ab1db33d 192.168.1.15:7000
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
M: 62148456ec17469a36ae19bfe684fc343b4ae2fd 192.168.1.17:7004
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
S: 9eae0f6dd599fc708ce318f749d8be441e6f2262 192.168.1.15:7001
   slots: (0 slots) slave
   replicates aed9f883906406839aad3162d46c8840ab1db33d
M: 73ed7c5e65c524dd8a98f56e61113ae2c207e856 192.168.1.16:7002
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: 763d115cab452a5d86902229e4356efab6b4676e 192.168.1.17:7005
   slots: (0 slots) slave
   replicates 73ed7c5e65c524dd8a98f56e61113ae2c207e856
S: e7d536e855da0098fef58985dc4e497a4bb55b78 192.168.1.16:7003
   slots: (0 slots) slave
   replicates 62148456ec17469a36ae19bfe684fc343b4ae2fd
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.

```

### 检查集群

```
# ./redis-trib.rb check 192.168.1.15:7000
>>> Performing Cluster Check (using node 192.168.1.15:7000)
M: aed9f883906406839aad3162d46c8840ab1db33d 192.168.1.15:7000
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
M: 62148456ec17469a36ae19bfe684fc343b4ae2fd 192.168.1.17:7004
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
S: 9eae0f6dd599fc708ce318f749d8be441e6f2262 192.168.1.15:7001
   slots: (0 slots) slave
   replicates aed9f883906406839aad3162d46c8840ab1db33d
M: 73ed7c5e65c524dd8a98f56e61113ae2c207e856 192.168.1.16:7002
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: 763d115cab452a5d86902229e4356efab6b4676e 192.168.1.17:7005
   slots: (0 slots) slave
   replicates 73ed7c5e65c524dd8a98f56e61113ae2c207e856
S: e7d536e855da0098fef58985dc4e497a4bb55b78 192.168.1.16:7003
   slots: (0 slots) slave
   replicates 62148456ec17469a36ae19bfe684fc343b4ae2fd
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```







## 错误

```
 # ./redis-trib.rb create --replicas 1 192.168.1.15:7000 192.168.1.15:7001 192.168.1.16:7002 192.168.1.16:7003 192.168.1.17:7004 192.168.1.17:7005
 >>> Creating cluster
[ERR] Sorry, can't connect to node 192.168.1.15:7000
```



```
# vim redis.conf
bind 192.168.1.15
```





## 链接地址

https://www.cnblogs.com/yuanermen/p/5717885.html

https://blog.csdn.net/jy0902/article/details/19248299

https://blog.csdn.net/wumiqing1/article/details/53610858

https://blog.csdn.net/xiaojin21cen/article/details/70445545

https://blog.csdn.net/honer123/article/details/79762572