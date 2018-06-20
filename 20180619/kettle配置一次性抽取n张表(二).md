## kettle配置一次性抽取n张表(二)

- 适用场景

适合目标数据库和源头数据库都是不变的，并且数据为轨迹类可变类的(如LG数据，存在更新TFSJ字段),这种方式就需要考虑到目标表的更新插入，建议利用临时表(物理表)，通过过程来为其更新插入。


- 方案描述

上一个方案和本方案不同主要实在更新这个问题上，这次以oracle为基础来设计此方案(postgresql9.5后才有了upsert语句)

第一步配置一张记录来源表，目标表字段名，以及来源表的时间戳字段，以及时间戳值，以及临时表名称

第二步使用kettle类似for循环将其一一取出并且变为变量传递

第三步如果临时表存在则将临时表将其删除后重建，如果没有临时表也要重建; 删除后重建意味着可能此次方案没有执行成功

第四步就是将数据抽取到临时表，通过存储过程(merge into)动态表的方式来更新数据。

第五步记录当前最大时间戳


- kettle版本

本次测试版本是kettle5.4,建议后期都尽量使用kettle5.4及其以上，本次测试都是使用oracle数据库。

### 方案配置


- 时间戳记录脚本

oracle

```
CREATE TABLE t_etl_time_stamp (id int primary key, source_obj varchar2(100),dest_obj varchar2(100),sjc_column varchar2(100),sjc_time varchar2(14),status varchar2(1),lsb_obj varchar2(100),zj_column varchar2(100),rksj date default sysdate,gxsj date);
```

#### 配置流程

- 配置删除语句

```
delete from ${dest_obj} where ${sjc_column} > to_date('${sjc_time}','yyymmddhh24miss')
```

![_](../img_src/kettle_loop_2.png)  


- 配置获取时间戳，时间戳字段，来源表，目标表


```
select source_obj,dest_obj,sjc_column,sjc_time from t_etl_time_stamp where status ='1'

```

演示数据

```
CREATE TABLE t_etl_time_stamp (id int primary key, source_obj varchar2(100),dest_obj varchar2(100),sjc_column varchar2(100),sjc_time varchar2(14),status varchar2(1),lsb_obj varchar2(100),zj_column varchar2(100),rksj date default sysdate,gxsj date);

insert into t_etl_time_stamp values(1,'t_gh_cs1','t_gh_cs2','sjc','20180501000000','1','T_GH_CS2_TMP','ID',SYSDATE,SYSDATE);


insert into t_etl_time_stamp values(1,'t_gh_cs3','t_gh_cs4','sjc','20180501000000','1','T_GH_CS4_TMP','ID',SYSDATE,SYSDATE);


create table t_gh_cs1(id int primary key,info varchar2(100),sjc varchar2(14));

create table t_gh_cs2(id int primary key,info varchar2(100),sjc varchar2(14));

create table t_gh_cs3(id int primary key,info varchar2(100),sjc varchar2(14));

create table t_gh_cs4(id int primary key,info varchar2(100),sjc varchar2(14));

insert into t_gh_cs1 values(1,'guohui','20180601000000');

insert into t_gh_cs3 values(1,'guohui','20180601120000');

insert into t_gh_cs1 values(2,'guose','20180601000000');

insert into t_gh_cs1 values(3,'guoqy','20180601120000');


```



