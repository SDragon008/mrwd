[TOC]

# linux nmap port

**document support**

ysys

**date**

2018-12-11

**label**

linux,nmap,port



## background

​	最近一段时间GA网内部很多服务器都在修改远程网络端口，可能修改过程中port设置和同一的不一样，而之后又忘记了这个port,给我打过电话，当时没有好的办法，今天偶然在一篇文章中找到这个一个方法试验一下。



## nmap

​    使用如下命令就可以查到当前ip地址下，有那些端口在使用

```
# nmap 192.168.1.35 -p1-65535

Starting Nmap 6.40 ( http://nmap.org ) at 2018-12-11 16:38 CST
Failed to resolve "1092.168.1.35".
WARNING: No targets were specified, so 0 hosts scanned.
Nmap done: 0 IP addresses (0 hosts up) scanned in 0.03 seconds
[root@gh8 yum.repos.d]# nmap 192.168.1.35 -p1-65535

Starting Nmap 6.40 ( http://nmap.org ) at 2018-12-11 16:38 CST
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for 192.168.1.35
Host is up (0.00019s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE
22/tcp    open  ssh
111/tcp   open  rpcbind
10022/tcp open  unknown
48891/tcp open  unknown
MAC Address: 08:00:27:C4:FF:59 (Cadmus Computer Systems)
```

  















​	redhat 6.x

```
for i in b c;
do
echo "KERNEL==\"sd*\", BUS==\"scsi\", PROGRAM==\"/sbin/scsi_id --whitelisted --replace-whitespace --device=/dev/\$name\", RESULT==\"`/sbin/scsi_id --whitelisted --replace-whitespace --device=/dev/sd$i`\", NAME=\"asm-disk$i\", OWNER=\"grid\", GROUP=\"asmadmin\", MODE=\"0660\""      >> /etc/udev/rules.d/99-oracle-asmdevices.rules
done

```

​    redhat 7.x

```

for i in b c;
do
echo "KERNEL==\"sd*\", SUBSYSTEM==\"block\", PROGRAM==\"/usr/lib/udev/scsi_id --whitelisted --replace-whitespace --device=/dev/\$name\", RESULT==\"`/usr/lib/udev/scsi_id --whitelisted --replace-whitespace --device=/dev/sd$i`\", SYMLINK+=\"asm-disk$i\", OWNER=\"grid\", GROUP=\"asmadmin\", MODE=\"0660\"" >> /etc/udev/rules.d/99-oracle-asmdevices.rules
done

```

###  







## 链接地址

https://www.cnblogs.com/nmap/p/6232207.html