---
title: Windows下MySql安装
date: 2016-06-02 16:06:37
tags: 
 - Install
categories: MySQL
#toc: true
---
## MySql下载
<!--more-->
去MySql官网下载MySql Zip文件
解压安装包，记住解压路径，后面的文件路径以实际路径为准
进入mysql目录，更改my-default.ini文件名为my.ini，在[mysqld]下面添加一行编码：
``` sql
basedir = D:/mysql-5.6.25-winx64/
datadir = D:/mysql-5.6.25-winx64/data/
tmpdir = D:/mysql-5.6.25-winx64/temp/
character-set-server = utf8
user=mysql
```
> 注意路径中的反斜杠，不要用Windows的正斜杠

### 配置环境变量
在环境变量里配置系统变量，新建MYSQL_HOME变量，添加Path记录
``` sql
MYSQL_HOME=C:\Program Files\mysql\mysql-5.6.25-winx64
Path=%MYSQL_HOME%/bin
```
> 注意环境变量的配置规则

### 安装MySQL

管理员执行CMD，进入MySQL的bin目录
``` sql
mysqld --install MySQL --defaults-file="my.ini"
```

“Win+R”组合键，查找“regedit”，打开注册表，找到路径
> HKEY_LOCAL_MACHINE - SYSTEM - CurrentControlSet - services - mysql

找到其中的ImagePath，将值改为
> "D:\mysql-5.6.25-winx64\bin\mysqld" --defaults-file="D:\mysql-5.6.25-winx64\my.ini" MySQL

此时MySql已经安装好了，可以直接启动了

### MySQL服务的启动、停止与卸载
启动: net start MySQL
停止: net stop MySQL
卸载: sc delete MySQL