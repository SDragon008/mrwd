[TOC]

# linux rpm

linux 源代码

1-1绝大多数开源软件都是以源代码形式发布

1-2一般打包成tar.gz的归档压缩文件

1-3源代码需要编译称为二进制形式才能使用

1-3-1>./configure 检查编译环境并生成makefile

1-3-2>make 对源代码进行编译，生成可执行文件

1-3-3>make install 将生成的可执行文件安装到当前计算机中



好处：兼用性及可控制性较好

坏处：使用起来比较麻烦，编译时间较长，极容易出现错误



rpm

常用命令

1>安装软件:rpm -i software.rpm

2>卸载软件：rpm -e software

只跟程序的名字

3>升级形式安装:rpm -U software-new.rpm

注：rpm支持http,ftp协议安装软件

rpm -ivh <http://www.baidu.com/software.com>

4>rpm参数

-v 显示详细信息

-h 显示进度条

-qa 安装所有安装的rpm软件

-qi 软件名：查询软件的名字，创建者，版本，所属组

-ql 软件名：列出所有该软件的文件