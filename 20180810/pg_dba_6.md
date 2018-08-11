[TOC]

# pg dba 6

​	

​	本章节主要讲述的是连接池及数据库高速缓存



## 连接池



### 短连接长连接tps

#### 短连接tps

```
[osdba@mysql45 ~]$ cat test.sql
select 1;
[osdba@mysql45 ~]$ pgbench -M extended -n -r -f ./test.sql -c 16 -j 4 -C -T 30
transaction type: Custom query
scaling factor: 1
query mode: extended
number of clients: 16
number of threads: 4
duration: 30 s
number of transactions actually processed: 11662
latency average: 41.159 ms
tps = 388.651729 (including connections establishing)
tps = 135108.207053 (excluding connections establishing)
statement latencies in milliseconds:
	30.871288	select 1;
[osdba@mysql45 ~]$ 



# sar -w 1 100000
06:22:25 AM      1.01    104.04
06:22:26 AM      0.00     71.00
06:22:27 AM      0.00     94.95
06:22:28 AM    389.90   7618.18
06:22:29 AM    460.61   9070.71
06:22:30 AM    394.95   7926.26
06:22:31 AM    407.07   8241.41
06:22:32 AM    311.46   6239.58
06:22:33 AM    386.46   7741.67
06:22:34 AM    384.85   7783.84
06:22:35 AM    394.95   7871.72
06:22:36 AM    383.33   7734.44
06:22:37 AM    412.00   8223.00
06:22:38 AM    378.12   7631.25

06:22:38 AM    proc/s   cswch/s
06:22:39 AM    373.47   7502.04
06:22:40 AM    414.14   8301.01
06:22:41 AM    389.00   7790.00
06:22:42 AM    356.12   7200.00
06:22:43 AM    433.66   8646.53
06:22:44 AM    380.00   7656.00
06:22:45 AM    426.26   8601.01
06:22:46 AM    398.99   8042.42
06:22:47 AM    328.28   6621.21
06:22:48 AM    414.14   8260.61
06:22:49 AM    423.23   8408.08
06:22:50 AM    423.76   8435.64
06:22:51 AM    370.71   7438.38
06:22:52 AM    382.65   7711.22
06:22:53 AM    425.25   8517.17
06:22:54 AM    321.21   6538.38
06:22:55 AM    408.00   8155.00
06:22:56 AM    407.07   7994.95
06:22:57 AM    400.00   8091.30
06:22:58 AM     81.00   1808.00


```



#### 长连接tps

```
[osdba@mysql45 ~]$ pgbench -M extended -n -r -f ./test.sql -c 16 -j 4  -T 30
transaction type: Custom query
scaling factor: 1
query mode: extended
number of clients: 16
number of threads: 4
duration: 30 s
number of transactions actually processed: 551789
latency average: 0.870 ms
tps = 18386.280802 (including connections establishing)
tps = 18419.581888 (excluding connections establishing)
statement latencies in milliseconds:
	0.866824	select 1;



# sar -w 1 1000000
06:24:40 AM      0.00     80.00
06:24:41 AM     21.00  31223.00
06:24:42 AM      0.00  36553.61
06:24:43 AM      0.00  40828.00
06:24:44 AM      0.00  37095.92
06:24:45 AM      0.00  42553.00

06:24:45 AM    proc/s   cswch/s
06:24:46 AM      0.00  36169.00
06:24:47 AM      0.00  36923.76
06:24:48 AM      0.00  36021.88
06:24:49 AM      0.00  45886.46
06:24:50 AM      0.00  37419.59
06:24:51 AM      0.00  47719.19
06:24:52 AM      0.00  42354.08
06:24:53 AM      0.00  38317.17
06:24:54 AM      0.00  37618.18

```

#### prepared

```
[osdba@mysql45 ~]$ pgbench -M prepared -n -r -f ./test.sql -c 16 -j 4  -T 30
transaction type: Custom query
scaling factor: 1
query mode: prepared
number of clients: 16
number of threads: 4
duration: 30 s
number of transactions actually processed: 835698
latency average: 0.574 ms
tps = 27842.177752 (including connections establishing)
tps = 27889.353489 (excluding connections establishing)
statement latencies in milliseconds:
	0.572068	select 1;
[osdba@mysql45 ~]$ 


# sar -w 1 100000
06:30:18 AM      0.00     81.00
06:30:19 AM      0.00     85.86
06:30:20 AM     20.79  40064.36
06:30:21 AM      0.00 108161.62
06:30:22 AM      0.00  83697.94
06:30:23 AM      0.00  80395.96
06:30:24 AM      0.00  83589.00
06:30:25 AM      0.00  95197.98

06:30:25 AM    proc/s   cswch/s
06:30:26 AM      0.00 116096.94
06:30:27 AM      0.00 111639.80
06:30:28 AM      0.00 119719.00
06:30:29 AM      0.00 118795.96
06:30:30 AM      0.00 111051.52
06:30:31 AM      1.03 105829.90
06:30:32 AM      0.00 114250.51
06:30:33 AM      0.00 109773.74
06:30:34 AM      0.00 114905.10
06:30:35 AM      0.00 118304.95
06:30:36 AM      0.00 123064.29
06:30:37 AM      0.00 120571.00
06:30:38 AM      0.00 122222.22
06:30:39 AM      0.00 121755.00

```

​	比较三者，其中短连接的tps值为388.651729，长连接的tps值为18386.280802,prepared的tps值为27842.177752，明显短连接的tps太低，消耗性能



​	

### pgbouncer 安装



[readme](pg_pgbouncer_install.md)













## 数据库高速缓存