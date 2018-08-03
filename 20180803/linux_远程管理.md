[TOC]

# linux 远程管理

​	

​	与个人计算机不同，服务器一般都是运行在IDC机房中，通常不会直接接触服务器硬件，而是通过远程管理方式对服务器进行控制



## 常见远程管理工具方式

​	RDP(remote desktop protocol)协议，windows远程桌面管理

​	telnet:CLI界面下的远程管理，几乎所有操作系统都有(内容明文传输),现在大多只用于网络传输

​	SSH(secure shell):CLI界面下的远程管理，几乎所有操作系统都有(内容加密传输)，类UNIX系统下的主要的远程管理方式（LINUX,BSD,MacOS X)

​	RFB（remote framebuffer),图形化远程管理协议，VNC(virtual network computing)使用的协议，主要作为类UNIX系统下的主要的图形化远程管理方式(LINUX,BSD,MacOS X)

​	

## SSH

​	secure shell是linux,unix,max最常用的远程文本管理协议

​	ssh使用秘钥对数据进行加密传输

​	ssh2:是现在广泛的ssh版本

​	openssh是ssh的一个开源程序

​	ssh分为服务端和客户端，ssh服务端默认启动，作为常驻服务进行启动

```
# service sshd status
openssh-daemon (pid  1481) is running...

```



```
ssh user@ip 
ssh user@ip command
```



第一次在两台主机之前建立连接SSH连接时，需要交换公钥用以进行加密

ssh信息保存在用户家目录的.sssh隐藏文件夹下



### SCP

SCP通过ssh在两台计算机进行快速的，加密的数据传输



```
scp 源文件 root@ip:/root/
```

-r 递归将文件夹拷贝到其他服务器



### rsync

rsync 将两台计算机之间通过SSH协议同步文件



```
rsync 源文件 root@ip:/root/
```



## VNC

vnc 分为服务端和客户端

需要在服务器上安装vnc服务，在客户端安装vnc客户端软件

[vnc 和 vnc_clien安装部署t](../20170601/linux_vnc_install.md)







## 链接地址

[xmanager vnc 区别](LINUX_XMANAGER_VNC.md)

[linux 免密码登陆](linux_免密码登陆.md)

[LINUX关闭一批服务器](LINUX_关闭_一批服务器.md)