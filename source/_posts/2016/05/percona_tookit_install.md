---
title: CentOS7下Percona Tookit rmp安装
date: 2016-05-01 12:06:37
tags: Tool
categories: MySQL
#toc: true
---
## Percona Tookit下载
<!--more-->
CentOS7下默认是没有安装wget命令的，所以先安装wget命令。
``` ps
yum -y install wget
```
在Percona官网找到最新版本下载地址（注：要是rpm哦），并复制到下面命令中，也可以直接下载到本地。
> https://www.percona.com/downloads/percona-toolkit/

``` ps
wget https://www.percona.com/downloads/percona-toolkit/2.2.17/RPM/percona-toolkit-2.2.17-1.noarch.rpm
```

## 工具安装
``` ps
yum -y install perl-DBI perl-DBD-MySQL perl-Time-HiRes perl-IO-Socket-SSL perl-Digest-MD5 perl-TermReadKey
```

## Tookil安装
``` ps
rpm -Uvh percona-toolkit-2.2.17-1.noarch.rpm
```