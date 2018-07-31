[TOC]



# linux  vnc install



​	之前在新疆出差时，同事使用vnc远程连接linux桌面操作系统，非常好用，但是当时没有时间好好研究，就暂时放弃了，这两天学习linux基础时，想到vnc的安装使用还没有了解，就开启vnc的学习之旅

 

 

## 总结

 

【2018-01-18】今天分别在centos6.5和centos7.2部署vnc服务，发现前面都是大同小异，第一点都使用了yum来安装自带的tigervnc-server,第二点都是关闭了防火墙，第三点都是在非root用户下创建了密码，第四点都是在root开启了vnc服务

 

下面分别介绍在不同系统下安装vnc的操作流程

 

说明

符号“#”代表root用户，符合“$”代表其他用户，在本次中代表postgres用户

 

## centos6.5_install

 

### 配置yum源

 

略过

 

### 查询并安装tigervnc-server

 

\# yum search vnc

\# yum -y install tigervnc-server

 

### 修改配置参数并且新建用户

 

\# vim /etc/sysconfig/vncservers

VNCSERVERS="1:postgres"

 

\# groupadd postgres

\# useradd -g postgres postgres

\# passwd postgres

 

解释一下”1:postgres”,1可以认为是访问端口，postgres这是一个用户，可以使用其他用户替换（没有测试root用户是否可以使用,不过不建议使用root用户，原因是root权限太高了)

 

 

### 设置密码

 

\# su - postgres

$ vncpasswd

 

### 关闭防火墙启动vnc服务

 

\# chkconfig iptables off

\# chkconfig --list iptables

\# service  vncserver start 

 

 

 

 

## centos7.2_install

 

 

### 配置yum源

 

略过

 

### 查询并安装tigervnc-server

 

参考centos6.5_install>2

 

### 修改配置参数并且新建用户

 

\# cp /lib/systemd/system/vncserver@.service /lib/systemd/system/vncserver@:1.service

\# vim /lib/systemd/system/vncserver@

 

将橙色的改为自己的用户名

 

User=postgres

 

PIDFile=/home/postgres/.vnc/%H%i.pid

 

 

\# groupadd postgres

\# useradd -g postgres postgres

\# passwd postgres

 

### 设置密码

 

参考centos6.5_install>4

 

### 关闭防火墙启动vnc服务   

 

\# systemctl stop firewalld.service

\# systemctl disable firewalld.service

\# firewall-cmd --state

\# systemctl start vncserver@:1.service

 

 

 

## vnc-viewer使用

 

 

在windows上安装vnc-viewer软件，点击后输入IP:port,其中port就是前面写的访问端口

 

![img](../img_src/9CCBEAB1CD5C40D48DD5FC494FDD5749/wps4eda.tmp.jpeg)

 

 ## 链接地址



http://blog.51cto.com/svenman/1359021