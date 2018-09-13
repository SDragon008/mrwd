[TOC]

# mongodb cluster install for linux



## 环境准备

操作系统：centos6.5

服务器：五台

软件：mongodb-linux-x86_64-3.4.6.tgz

服务器规划

| 服务器1        | 服务器2        | 服务器3        | 服务器4        | 服务器5        |
| -------------- | -------------- | -------------- | -------------- | -------------- |
| mongos server  | mongos server  | config server  | config server  | config server  |
| shared1 server | shared2 server | shared3 server | shared4 server | shared5 server |
| shared5 server | shared1 server | shared2 server | shared3 server | shared4 server |
| shared4 server | shared5 server | shared1 server | shared2 server | shared3 server |



## 使用虚拟机操作

​	使用一台服务器将部分可以直接拷贝的文件先安装

​	上传文件解压文件

```
# mkdir /software --文件上传到这个路径下
# cd /software
# tar -xzvf mongodb-linux-x86_64-3.4.6.tgz -C /usr/local/
# cd /usr/local/
# mv mongodb-linux-x86_64-3.4.6 mongodb
```

```
mkdir -p /usr/local/mongodb/conf
mkdir -p /data/mongos/log
mkdir -p /data/config/data
mkdir -p /data/config/log
mkdir -p /data/shard1/data
mkdir -p /data/shard1/log
mkdir -p /data/shard2/data
mkdir -p /data/shard2/log
mkdir -p /data/shard3/data
mkdir -p /data/shard3/log
mkdir -p /data/shard4/data
mkdir -p /data/shard4/log
mkdir -p /data/shard5/data
mkdir -p /data/shard5/log
```

```
# cd ~
# vim .bahs_profile
export MONGODB_HOME=/usr/local/mongodb
export PATH=$MONGODB_HOME/bin:$PATH
# source .bash_profile
```

```
# groupadd ysys
# useradd -g ysys ysys
# chown -R ysys:ysys /usr/local
# chown -R ysys:ysys /data
```



校验mongo环境是否配置正常

```
# mongod -v
...
2018-09-11T07:23:53.759+0800 I CONTROL  [initandlisten] db version v3.4.6
...
```



关机将修改其他四台的ip和主机名

```
mongo 192.168.1.33:27000
```



```
config = {_id : "config", members : [ {_id : 0, host : "192.168.1.33:21000" }, {_id : 1, host : "192.168.1.34:21000" },{_id : 2, host : "192.168.1.35:21000" } ]}
```

```
rs.initiate(config)
```

```
rs.status();
```

```
> config = {_id : "config", members : [ {_id : 0, host : "192.168.1.33:21000" }, {_id : 1, host : "192.168.1.34:21000" },{_id : 2, host : "192.168.1.35:21000" } ]}
{
	"_id" : "config",
	"members" : [
		{
			"_id" : 0,
			"host" : "192.168.1.33:21000"
		},
		{
			"_id" : 1,
			"host" : "192.168.1.34:21000"
		},
		{
			"_id" : 2,
			"host" : "192.168.1.35:21000"
		}
	]
}
> rs.initiate(config)
{
	"ok" : 0,
	"errmsg" : "Attempting to initiate a replica set with name config, but command line reports configs; rejecting",
	"code" : 93,
	"codeName" : "InvalidReplicaSetConfig"
}
> rs.status();
{
	"info" : "run rs.initiate(...) if not yet done for the set",
	"ok" : 0,
	"errmsg" : "no replset config has been received",
	"code" : 94,
	"codeName" : "NotYetInitialized"
}
> 
```



```
mongo 192.168.1.31:27001
```

```
use admin
```

```
config = {_id : "shard1",members : [{_id : 0, host : "192.168.1.31:27001" },{_id : 1, host : "192.168.1.32:27001" },{_id : 2, host : "192.168.1.33:27001" }] }
```

















## 链接地址

https://www.cnblogs.com/ityouknow/p/7566682.html