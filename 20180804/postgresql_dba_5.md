[TOC]

# pg dba 5

​	

​	本章节主要讲述了执行计划，成本公式解说，代价因此校准，自动追踪sql执行计划



## explain

### explain 语法



explain:显示sql的执行计划

analyze:通过实际执行的sql来获取相应的执行计划

verbose:显示计划的附加信息，如计划树中每个节点输出的各个列。

costs：每个节点的启动成本和总成本，以及估计行数和每行的宽度。

buffers：显示缓存区使用的信息，该参数只能和analyze参数一起使用

format:指定的输出格式，text,xml,json,yaml

timing:输出时间开销



