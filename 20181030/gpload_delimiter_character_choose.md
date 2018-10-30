[TOC]

# gpload delimiter character choose

**文档整理**

ysys

**日期**

2018-10-30

**标签**

greenplum,gpload,delimiter character



## 背景

​	最近一段时间需要将数据从Oracle同步到Gp数据库中，最好的同步手段是通过gpload加载csv文件到Gp数据库中，但是发现单个字符没有办法总是会将某些数据拦腰斩断，导致数据无法入库，后面发现gpload只能使用一个分割符分割，那么如何选择分隔符，选择什么分隔符比较好，不会造成太大数据遗留，选择什么分割符，可能让数据完美入库。



## 单个字符



​	之前单个字符经常导致出现‘extra data ...’,就选用了两个字符，后面执行时报错

```
A gpload control file processing warning occurred. A delimiter must be single ASCII charactor, you can also use unprintable characters(for example: '\x1c' / E'\x1c' or '\u001c' / E'\u001c'

A gpload control file processing error occurred. Invalid delimiter, gpload quit immediately
```

​	那么选用什么字符能够避免这种情况的发生。

```

```



















## 链接地址

http://ascii.911cha.com/

https://www.cnblogs.com/zzsdream/p/5802254.html

https://kimnote.com/tools/ascii/

