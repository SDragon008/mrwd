# pg_rule

​	规则系统:讲真话真不懂什么意思，不过就是知道你查询这个语句，不是你真实查询的语句。

## SELECT 规则

- 视图语句创建

  ```
  create view vw_name as select column_name1,column_name2 ,... from table_name ...
  ```

  上面创建的语句等价于下面语句

- SELECT RULE 创建

  ```
  create table table_name_a as select *　from table_name where 1=2;
  CREATE RULE "_RETURN" AS ON SELECT TO table_name_a DO INSTEAD SELECT * FROM table_name;
  ```

- 备注

  1. SELECT 规则的后续操作只能是"INSTEAD SELECT"
  2. SELECT 规则名称只能是"_RETURN"

   	﻿