

[TOC]

# LINUX 常用命令



## 查看Linux操作系统

linux查看版本当前操作系统内核信息

[root@centos65 ~]# uname -a

Linux centos65 2.6.32-431.el6.x86_64 #1 SMP Fri Nov 22 03:15:09 UTC 2013 x86_6

4 x86_64 x86_64 GNU/Linux

linux查看当前操作系统版本信息

[root@centos65 ~]# cat /proc/version

Linux version 2.6.32-431.el6.x86_64 (mockbuild@c6b8.bsys.dev.centos.org) (gcc 

version 4.4.7 20120313 (Red Hat 4.4.7-4) (GCC) ) #1 SMP Fri Nov 22 03:15:09 UTC 2013

linux查看版本当前操作系统发行版信息

[root@centos65 ~]# cat /etc/issue

CentOS release 6.5 (Final)

[root@centos65 ~]# cat /etc/redhat-release

CentOS release 6.5 (Final)



## 文本处理

### 复制

在相同目录下复制，需要重新为目标文件起名

```
 cp gh.txt gh2.txt
```

在不同目录下，只需要跟着需要放置的路径

```
 cp gh2.txt ./filedir/
```

将文件夹下的内容拷贝到其他文件夹下

```
cp -r filedir/* ./filedir2/
```







文件浏览

cat

more

less

head

tail 

基于关键字进行搜索

grep 

