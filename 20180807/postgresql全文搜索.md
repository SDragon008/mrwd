[TOC]

# postgresql 全文搜索

​	

## 文本搜索类型

​	postgresql提供了两种数据类型用于支持全文搜索，即通过自然语言documents的集合来找到那些匹配一个query的检索。tsvector类型产生一个文档，tsquery类型表示一个文本查询。

	### tsvector

​	tsvector的值是一个无重复值的lexemes排序列表，即一些同一个词的不同变种的标准化

```
tutorial=# select $$a fat cat sat on a mat and ate a fat rat$$::tsvector;
                      tsvector                      
----------------------------------------------------
 'a' 'and' 'ate' 'cat' 'fat' 'mat' 'on' 'rat' 'sat'
```

​	'fat'字段出现了2，'a'值出现了3次

​	

​	在这个例子中，使用'$$'字符串文本来区分子弹

```
tutorial=# select $$the lexeme ' ' contains spacess$$::tsvector;
                tsvector                 
-----------------------------------------
 ' ' 'contains' 'lexeme' 'spacess' 'the'
(1 row)
tutorial=# select $$the lexeme 'Joe''s' contains a quote$$::tsvector;
                    tsvector                    
------------------------------------------------
 'Joe''s' 'a' 'contains' 'lexeme' 'quote' 'the'
(1 row)
```



​	整型positions也可以放到词汇中

```
tutorial=# SELECT 'a:1 fat:2 cat:3 sat:4 on:5 a:6 mat:7 and:8 ate:9 a:10 fat:11 rat:12'::tsvector;
                                   tsvector                                    
-------------------------------------------------------------------------------
 'a':1,6,10 'and':8 'ate':9 'cat':3 'fat':2,11 'mat':7 'on':5 'rat':12 'sat':4
(1 row)


```

​	位置通常表示文档中的源头的位置，位置信息可用在proximity ranking ;位置值得范围从1到16383；相同词的重复位会被忽略掉

```
tutorial=# SELECT 'a:1 fat:2 cat:3 sat:4 on:5 a:6 mat:7 and:8 ate:200000 a:10 fat:16900 rat:12'::tsvector;
                                       tsvector                                       
--------------------------------------------------------------------------------------
 'a':1,6,10 'and':8 'ate':16383 'cat':3 'fat':2,16383 'mat':7 'on':5 'rat':12 'sat':4
(1 row)

```

​	

​	拥有位置的词汇甚至可以用一个权来标记，这个权可以是A,B,C或D,默认是D，因此输出中不会出现

```
tutorial=# SELECT 'a:1A fat:2B,4C cat:5D'::tsvector;
          tsvector          
----------------------------
 'a':1A 'cat':5 'fat':2B,4C
(1 row)


```

​	权可以用来反映文档结构，如：标记标题以与主体相区别。 全文检索排序函数可以为不同的权标记来分配不同的优先级。  

​	

​	tsvector类型不能自己标准化自己这一点是很重要的，它假设传递给它的单词对应程序来说都是恰到标准化的。

```
tutorial=# select 'The Fat Rats'::tsvector;
      tsvector      
--------------------
 'Fat' 'Rats' 'The'
(1 row)

tutorial=# select to_tsvector('The Fat Rats');
   to_tsvector   
-----------------
 'fat':2 'rat':3
(1 row)

tutorial=# select to_tsvector('english','The Fat Rats');
   to_tsvector   
-----------------
 'fat':2 'rat':3
(1 row)

```



### tsquery

​	`tsquery`存储用于检索的词汇，并且使用布尔操作符 `&`(AND)，`|`(OR)和`!`(NOT) 来组合它们 

​	括号用来强调操作符的分组 

```
SELECT 'fat & rat'::tsquery;
    tsquery    
---------------
 'fat' & 'rat'

SELECT 'fat & (rat | cat)'::tsquery;
          tsquery          
---------------------------
 'fat' & ( 'rat' | 'cat' )

SELECT 'fat & rat & ! cat'::tsquery;
        tsquery         
------------------------
 'fat' & 'rat' & !'cat'
```

​	在没有括号的情况下，!(NOT)结合的最紧密，而&(AND)结合的比|(OR)紧密

​	

​	tsquery中的词汇可以被一个或者多个权字母来标记，这些权字母用来限制它们只能带有匹配权的tsvector词汇进行匹配

```
tutorial=# select 'fat:ab & cat '::tsquery;
     tsquery      
------------------
 'fat':AB & 'cat'
(1 row)
```

​	

​	tsquery中的词汇可以用*进行标记来指定前缀匹配

```
tutorial=# select 'super:*'::tsquery;
  tsquery  
-----------
 'super':*

```

​	这个查询可以匹配tsvector中以"super"开始的任意单词。请注意，前缀首先被文本搜索配置处理，这也就意味着下面的结果为真： 

```

tutorial=# SELECT to_tsvector( 'postgraduate' );
  to_tsvector  
---------------
 'postgradu':1
(1 row)
tutorial=# select to_tsquery('postgres:*');
 to_tsquery 
------------
 'postgr':*
tutorial=# SELECT to_tsvector( 'postgraduate' ) @@ to_tsquery( 'postgres:*' );
 ?column? 
----------
 t
(1 row)

```



```

tutorial=# SELECT to_tsquery('Fat:ab & Cats');
    to_tsquery    
------------------
 'fat':AB & 'cat'
(1 row)

tutorial=# select to_tsvector('Fat:ab & Cats');
      to_tsvector       
------------------------
 'ab':2 'cat':3 'fat':1
(1 row)

tutorial=# select 'Fat:ab & Cats'::tsquery;
      tsquery      
-------------------
 'Fat':AB & 'Cats'
(1 row)

```





### 文本检索函数和操作符





**文本检索操作符**

| 操作符 | 描述                        | 示例                                                         | 结果                        |
| ------ | --------------------------- | ------------------------------------------------------------ | --------------------------- |
| `@@`   | `tsvector` 匹配 `tsquery` ? | `to_tsvector('fat cats ate rats') @@ to_tsquery('cat & rat')` | `t`                         |
| `@@@`  | 弃用的`@@`的同义词          | `to_tsvector('fat cats ate rats') @@@ to_tsquery('cat & rat')` | `t`                         |
| `||`   | 连接`tsvector`s             | `'a:1 b:2'::tsvector || 'c:1 d:2 b:3'::tsvector`             | `'a':1 'b':2,5 'c':3 'd':4` |
| `&&`   | `tsquery`与                 | `'fat | rat'::tsquery && 'cat'::tsquery`                     | `( 'fat' | 'rat' ) & 'cat'` |
| `||`   | `tsquery`或                 | `'fat | rat'::tsquery || 'cat'::tsquery`                     | `( 'fat' | 'rat' ) | 'cat'` |
| `!!`   | `tsquery`非                 | `!! 'cat'::tsquery`                                          | `!'cat'`                    |
| `@>`   | `tsquery` 包含另一个?       | `'cat'::tsquery @> 'cat & rat'::tsquery`                     | `f`                         |
| `<@`   | `tsquery` 包含于 ?          | `'cat'::tsquery <@ 'cat & rat'::tsquery`                     | `t`                         |





**文本检索函数**

| 函数                                                         | 返回类型    | 描述                               | 示例                                                         | 结果                            |
| ------------------------------------------------------------ | ----------- | ---------------------------------- | ------------------------------------------------------------ | ------------------------------- |
| `get_current_ts_config()`                                    | `regconfig` | 获取文本检索的默认配置             | `get_current_ts_config()`                                    | `english`                       |
| `length(tsvector)`                                           | `integer`   | `tsvector`的单词数                 | `length('fat:2,4 cat:3 rat:5A'::tsvector)`                   | `3`                             |
| `numnode(tsquery)`                                           | `integer`   | `tsquery`中的单词加上操作符的数量  | ` numnode('(fat & rat) | cat'::tsquery)`                     | `5`                             |
| `plainto_tsquery([ *config* regconfig , ] *query* text)`     | `tsquery`   | 产生`tsquery`忽略标点              | `plainto_tsquery('english', 'The Fat Rats')`                 | `'fat' & 'rat'`                 |
| `querytree(*query* tsquery)`                                 | `text`      | 获取`tsquery`可索引的部分          | `querytree('foo & ! bar'::tsquery)`                          | `'foo'`                         |
| `setweight(tsvector, "char")`                                | `tsvector`  | 给`tsvector`的每个元素赋予权值     | `setweight('fat:2,4 cat:3 rat:5B'::tsvector, 'A')`           | `'cat':3A 'fat':2A,4A 'rat':5A` |
| `strip(tsvector)`                                            | `tsvector`  | 删除`tsvector`中的位置和权值       | `strip('fat:2,4 cat:3 rat:5A'::tsvector)`                    | `'cat' 'fat' 'rat'`             |
| `to_tsquery([ *config* regconfig , ] *query* text)`          | `tsquery`   | 标准化单词并转换为`tsquery`        | `to_tsquery('english', 'The & Fat & Rats')`                  | `'fat' & 'rat'`                 |
| `to_tsvector([ *config* regconfig , ] *document* text)`      | `tsvector`  | 减少文档中的文本到 `tsvector`      | `to_tsvector('english', 'The Fat Rats')`                     | `'fat':2 'rat':3`               |
| `ts_headline([ *config* regconfig, ] *document* text, *query* tsquery [, *options* text ])` | `text`      | 显示一个查询的匹配项               | `ts_headline('x y z', 'z'::tsquery)`                         | `x y <b>z</b>`                  |
| `ts_rank([ *weights* float4[], ] *vector* tsvector, *query* tsquery [, *normalization* integer ])` | `float4`    | 文档查询排名                       | `ts_rank(textsearch, query)`                                 | `0.818`                         |
| `ts_rank_cd([ *weights* float4[], ] *vector* tsvector, *query* tsquery [, *normalization* integer ])` | `float4`    | 排序文件查询使用覆盖密度           | `ts_rank_cd('{0.1, 0.2, 0.4, 1.0}', textsearch, query)`      | `2.01317`                       |
| `ts_rewrite(*query* tsquery, *target* tsquery, *substitute* tsquery)` | `tsquery`   | 替换带有查询的替代目标             | `ts_rewrite('a & b'::tsquery, 'a'::tsquery, 'foo|bar'::tsquery)` | `'b' & ( 'foo' | 'bar' )`       |
| `ts_rewrite(*query* tsquery, *select* text)`                 | `tsquery`   | 从一条`SELECT`命令的替代目标做替换 | `SELECT ts_rewrite('a & b'::tsquery, 'SELECT t,s FROM aliases')` | `'b' & ( 'foo' | 'bar' )`       |
| `tsvector_update_trigger()`                                  | `trigger`   | 自动更新`tsvector`列的触发器函数   | `CREATE TRIGGER ... tsvector_update_trigger(tsvcol, 'pg_catalog.swedish', title, body)` | ``                              |
| `tsvector_update_trigger_column()`                           | `trigger`   | 自动更新`tsvector`列的触发器函数   | `CREATE TRIGGER ... tsvector_update_trigger_column(tsvcol, configcol, title, body)` | ``                              |





**文本检索调试函数**

| 函数                                                         | 返回类型       | 描述                       | 示例                                               | 结果                                                         |
| ------------------------------------------------------------ | -------------- | -------------------------- | -------------------------------------------------- | ------------------------------------------------------------ |
| `ts_debug([ *config* regconfig, ] *document* text, OUT *alias* text, OUT *description* text, OUT *token* text, OUT *dictionaries* regdictionary[], OUT *dictionary* regdictionary, OUT *lexemes* text[])` | `setof record` | 测试一个配置               | `ts_debug('english', 'The Brightest supernovaes')` | `(asciiword,"Word, all ASCII",The,{english_stem},english_stem,{}) ...` |
| `ts_lexize(*dict* regdictionary, *token* text)`              | `text[]`       | 测试一个数据字典           | `ts_lexize('english_stem', 'stars')`               | `{star}`                                                     |
| `ts_parse(*parser_name* text, *document* text, OUT *tokid* integer, OUT *token* text)` | `setof record` | 测试一个解析               | `ts_parse('default', 'foo - bar')`                 | `(1,foo) ...`                                                |
| `ts_parse(*parser_oid* oid, *document* text, OUT *tokid* integer, OUT *token* text)` | `setof record` | 测试一个解析               | `ts_parse(3722, 'foo - bar')`                      | `(1,foo) ...`                                                |
| `ts_token_type(*parser_name* text, OUT *tokid* integer, OUT *alias* text, OUT *description* text)` | `setof record` | 由解析器获取分词类型的定义 | `ts_token_type('default')`                         | `(1,asciiword,"Word, all ASCII") ...`                        |
| `ts_token_type(*parser_oid* oid, OUT *tokid* integer, OUT *alias* text, OUT *description* text)` | `setof record` | 由解析器获取分词类型的定义 | `ts_token_type(3722)`                              | `(1,asciiword,"Word, all ASCII") ...`                        |
| `ts_stat(*sqlquery* text, [ *weights* text, ] OUT *word* text, OUT *ndoc* integer, OUT *nentry* integer)` | `setof record` | 获取`tsvector`列的统计数据 | `ts_stat('SELECT vector from apod')`               | `(foo,10,15) ...`                                            |



## 全文搜索

​	

​	

## 链接地址



http://www.postgres.cn/docs/9.4/textsearch-intro.html