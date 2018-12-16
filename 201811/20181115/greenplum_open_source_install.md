[TOC]

# greenplum open source install

**文档整理**

ysys

**日期**

2018-11-15(2018-11-17)

**标签**

greenplum,greenplum open source,greenplum open source install



## 背景

​	最近一段时间听说greenplum从版本5开始推出了open source，商用版可能不考虑，但是open source还是需要了解怎样部署的。

## 版本

|          | 版本 | 备注 |
| -------- | ---- | ---- |
| 操作系统 |      |      |
|          |      |      |
|          |      |      |



## 环境





## 步骤





```
# python
Python 2.6.6 (r266:84292, Nov 22 2013, 12:16:22) 
[GCC 4.4.7 20120313 (Red Hat 4.4.7-4)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> exit();
```





















## 报错

1、./configure --configure: error: Conan not found.  Please run 'pip install conan'

```
[root@centos65 depends]# ./configure
checking for a thread-safe mkdir -p... /bin/mkdir -p
checking for a BSD-compatible install... /usr/bin/install -c
checking for conan... no
configure: error: Conan not found.  Please run 'pip install conan'
```

直接执行`pip install conan`报错

```
# pip install conan
-bash: pip: command not found
```





https://www.80sy.com/625.html





