[TOC]

# kettle 获取远程文件并下载



kettle remote file ftp download  step

前提：在本次TL项目建设中，需要的数据资源有一部分通过ftp的方式交换的。

1>获取远程更新或者新增的文件名

这个方案第一个难点就是如何获取新增的或者更改的文件名

1-1>执行脚本获取ftp中的文件名，新增或者修改

时间

1-2>与本地文件比对查看那些文件需要下载

2>从ftp中下载新增文件

这个方案第二个难点就是新增的文件可能多个需要将其全部下载完成

2-1>获取文件名并传参

2-2>设置循环下载

3>文件解析入库

这个方案第三个难点就是文本文件的分隔符

附件为远程ftp下载

![ftp_set_variable.ktr](../img_src/0dfefb224ec248689afea81f727bfcf3/attachment.png)

[ftp_set_variable.ktr](../img_src/FD8295EB28C84A528080D01B7BBD49E4/ftp_set_variable.ktr)

![hhht_ftp_copy_rows_to_result.ktr](../img_src/C04B2AEEDF9C450D9107586A47A5844B/attachment.png)

[hhht_ftp_copy_rows_to_result.ktr](../img_src/E61846925C6348859FECA9589B1EBDF7/hhht_ftp_copy_rows_to_result.ktf)

![hhht_ftp_result_variable_system_download.kjb](../img_src/4471F7A808FA405C829CA6B14F217B1A/attachment.png)
[hhht_ftp_result_variable_system_download.kjb](../img_src/5991B955EFF64D14B20DF5832E4DE6BF/hhht_ftp_result_variable_system_download.kjb)

![hhht_ftp_system_download.kjb](../img_src/C0AF2099C1024A239A2DFDAE1973434F/attachment.png)
[hhht_ftp_system_download.kjb](../img_src/15B220077B1B406A9091A17D3EB31CB7/hhht_ftp_system_download.kjb)

![hhht_ftp_variable_system_download.kjb](../img_src/B9C77523B339486AB4E515818D99BFEC/attachment.png)
[hhht_ftp_variable_system_download.kjb](../img_src/B8E5CDF4C1044258B3CB768FF85F1880/hhht_ftp_variable_system_download.kjb)

![SYSTEM_INFO.ktr](../img_src/3B5C8E6ECE3C4DD2AC0B4D7140897AB3/attachment.png)
[SYSTEM_INFO.ktr](../img_src/FC1BF674349F424E87F60286E8D6FFC7/SYSTEM_INFO.ktr)