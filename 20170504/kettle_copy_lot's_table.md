[TOC]

# kettle copy lot's table



xj现场2000多代码表，不可能一个一个的拷贝，发现表码是两个标准字段，那么先将表结构创建出来，然后使用kettle将其全部抽取过来

下面是我测试的脚本，可以直接使用，只需要修改两个脚本，分别是oracle_copy.ktr修改数据库链接，表名；oracle_ins.ktr修改数据库链接

[get_result_set_variable.ktr](../img_src/DA48D6A39470413F92B310F693395456/get_result_set_variable.ktr)

[get_result_set_variable_job.kjb](../img_src/30D22FB0580E4A7397115480BA4596F7get_result_set_variable_job.kjb)

[oracle_copy.ktr](../img_src/FF68020A94064D08A382C882D4048DBE/oracle_copy.ktr)

[oracle_copy_job.kjb](../img_src/7681EB66857648BEA36321A14314A1D4/oracle_copy_job.kjb)

[oracle_ins.ktr](../img_src/6F7068B136EB4EA7BE76AD6CA1EA6DFA/oracle_ins.ktr)

[oracle_ins_job.kjb](../img_src/671C393FEBCF4340A9DAFA15363DB2D9/oracle_ins_job.kjb)

此次脚本命令和华为mppdb对接是，经常会出现运行一段时间就不会再跑了，这个问题需要排查一下到底是什么问题，不过脚本就是这个样子了，后期再修改