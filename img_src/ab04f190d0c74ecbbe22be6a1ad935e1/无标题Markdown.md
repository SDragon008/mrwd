
## 关于kettle将oracle数据库迁移到另外oracle数据库方案


### 背景


由于目标库为汇集库，需要将各个业务单位的数据库都要事先汇总过来，现在就各个业务单位的数据库大都是oracle，而目标库也是oracle数据库，任务比较繁重，如果每张表都要事先调研，分析表的主键，时间戳，是否分区，对于现场人手不够的情况下，用户有要求数据灌入，建议使用其他方式创建表和首批全量数据灌入。

### 要求

- 原始数据库表名，目标数据库表名，表的大概数据量，表中文注释，表是否分区

- 字段名，字段类型，字段长度，字段默认初始值，字段注释，是否非空约束

- 索引，唯一索引，主键信息

这些信息是设计要求，后期要尽量满足。

### 实现

- 原始数据库表名，目标数据库表名，表的大概数据量，表中文注释，表是否分区,但是无法取得分区表是list,hash,range分区。

- 字段名，字段类型，字段长度，字段默认初始值，字段注释,但是是否非空约束无法更好的定义，暂时没有想到更好的sql

- 索引可以获取，主键信息可以获取，唯一索引也是可以获取，但是不建议在初次加载时使用，建议历史数据入库完成后在重新创建


### 系统表

- user_tables
- user_tab_columns
- user_constraints
- user_cons_columns
- others

- 原始表名，数据量，中文注释，是否分区

```

select table_name,
       num_rows,
       comments,
       (case
         when cn = '0' then
          'f'
         else
          't'
       end) as sffq
  from (select t.table_name,
               t.num_rows,
               trim(c.comments) as comments,
               (select count(1)
                  from user_tab_partitions p
                 where p.table_name = t.TABLE_NAME) as cn　from user_tables t,
               user_tab_comments c where t.table_name = c.table_name) f
               
```

- 字段名，字段类型，字段长度，字段默认初始值，字段注释

```
select t.table_name,
               t.column_name,
               t.data_type,
               t.data_length,
               REPLACE(replace(t.data_type || '(' || t.data_length || ')',
                               'DATE(7)',
                               'DATE'),
                       'CLOB(4000)',
                       'CLOB') as zdlx,
                       t.data_default,
               s.comments
          from user_tab_cols t, user_col_comments s
         where t.TABLE_NAME = s.table_name
           and t.COLUMN_NAME = s.column_name
         order by t.table_name, t.column_id


```
 
注：字段data_default中的字段类型是long类型，在oracle中很难直接转换为varchar类型，需要后期与kettle控件组合使用。
创建语句如果报错，可能与字符类型有关，可以自行修改脚本方案


-索引，唯一索引，主键索引
```




```

暂时不考虑索引和唯一索引，只是考虑主键索引











