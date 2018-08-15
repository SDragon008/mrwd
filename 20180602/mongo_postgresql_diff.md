[TOC]

# mongo postgresql 替代 是否可行



​	最早接触到mongo，是在TJ出差时，研发告知DX数据库选型是mongo数据库，当时就想知道为什么研发会想选用MONGO作为数据库选型。经过一段时间的学习，觉得研发选型可能有以下三种考虑



## 选择mongo原因

1、由于DX要求数据库对于字段需要可扩展，普通的数据库创建表时，都要事先将字段进行确认才能开始创建，如果需要新添加某个字段，只有通过命令行来添加该字段，这样普通关系型数据库就被抛弃。

2、更新、插入操作

1） 数据主键如果存在，其他字段不存在，需要在这条数据上插入其他字段数据。

2） 数据主键如果存在，其他字段存在，需要将这些字段数据更新

3） 数据主键不存在，那么要插入一条最新数据

从 1），2），3）循环判断；这样普通数据格式更是无法完成该操作。

3、数据库是否可以集群。

数据库是否支持分布式数据库，如果支持，效率如何？



## postgresql jsonb

那么其他关系型数据库是否有数据库可能支持上述操作

postgresql可以支持 第一种和第二种情况，第三种情况，postgresql本身是单机数据库，无法支持分布式，但是postgresql-xc和postgresql-xl都是可以支持分布式，并且在数据库性能上得到优化。

那么为什么postgresql 可以支持第一种和第二种情况？

首先由于MongoDB的文档结构为BJSON格式（BJSON全称：Binary JSON），而BJSON格式本身就支持保存二进制格式的数据，因此可以把文件的二进制格式的数据直接保存到MongoDB的文档结构中，而postgresql数据库在9.3是支持json类型，在9.4上支持jsonb类型，jsonb类型其实就是二进制存储的，这样来看，两者没有什么区别。

但是在更新时，mongodb可以直接采用upsert方式，可以postgresql应该是9.5后才支持upsert方式，但是不支持下面语句

```insert into tbalea select * from tbaleb on conflict (key) do update jsonb=jsonb||excluded.jsonb；```

报jsonb冲突。

函数暂时能解决该方案

```
CREATE OR REPLACE FUNCTION public.f_upsert_jsonb(

    text,

    jsonb)

  RETURNS void AS

$BODY$  

declare  

  res int;  begin  

  update t_gh_jsonb set info= info|| $2 where id=$1;  

  if not found then  

    insert into t_gh_jsonb (id,info) values ($1,$2);  

  end if;  

  exception when others then  

    return;  end;  

$BODY$

  LANGUAGE plpgsql VOLATILE STRICT

  COST 100;


```




​	经过上述判断，觉得研发可以将postgresql可以作为mongodb的替代品。
	后期发现mongodb客户端软件，上面有集成sql语句，可以满足普通的sql查询，那么对现场实施要求的能力就比较小了。






