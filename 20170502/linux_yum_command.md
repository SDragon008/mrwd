[TOC]

# LINUX YUM CONFIG

linux yum

yum解决源代码和rpm的不足，解决rpm的依赖关系

优势：自动解决依赖关系

​      可以对rpm进行分组，并基于组进行安装操作

​      引入仓库概念

​      配置简单

​    

仓库：用来存放当前现有的rpm软件包

yum的配置文件存放在/etc/yum.repos.d/

[dvd]  --------------->必须写的，中括号的内容可以随便写，但一定要有中括号

name = yum server  ----------->可写可不写，内容随便，主要是个提示作用

baseurl=file:///mnt/cdrom  --------------->一定要写的，定义yum源的仓库所在

enabled=1 --------------------->数字1为启用当前yum源，0为禁用，默认为1。

gpgcheck=0  ----------------------->是否检查rpm包的数字签名，数字1为检查，0为不检查，可以不写。

yum配置文件必须以.repo结尾

yum常用命令

1>yum install software

2>yum remove software

3>yum update software

4>其他命令

4-1>yum list all:显示配置所有配置的rpm软件

​        yum info tigervnc：类似rpm -qi rpm软件

​        yum search software:查询rpm软件

4-2>yum clean all:清空缓存（配置yum后，清空缓存）

​    