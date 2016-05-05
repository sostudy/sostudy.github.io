---
title: CentOS7下MySQL源码安装
date: 2016-04-30 00:55:54
tags: 
 - Install
categories:
 - MySQL
#: true
---
## CentOS下MySQL源码安装
<!--more-->
### 卸载系统自带的MySQL
```ps
# 卸载系统默认安装包
yum remove mysql mysql-server mysql-libs compat-mysql51
# 删除原始文件夹
rm -rf /var/lib/mysql
rm /etc/my.cnf
# 检查一下是否还有残留，有则删除
rpm -qa|grep mysql
```
### MySQL源码安装
#### MySQL下载
<!--more-->
##### 下载地址，选择最下面“Generic Linux”版本
> http://dev.mysql.com/downloads/mysql/5.6.html#downloads

![](http://7xt9fi.com2.z0.glb.clouddn.com/mysql/installmysql-do.png)
#### 解压编译
```ps
tar xvf mysql-5.6.30.tar.gz
cd mysql-5.6.30
```
> 编译，注意cmake \后面操作都带有“-”符号

```sql
cmake \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DMYSQL_DATADIR=/usr/local/mysql/data \
-DSYSCONFDIR=/etc \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_MEMORY_STORAGE_ENGINE=1 \
-DWITH_READLINE=1 \
-DMYSQL_UNIX_ADDR=/var/lib/mysql/mysql.sock \
-DMYSQL_TCP_PORT=3306 \
-DENABLED_LOCAL_INFILE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DEXTRA_CHARSETS=all \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci
```
编译成功，执行安装
```ps
    make && make install
```
#### 创建用户
```ps
# 创建用户组和用户
groupadd mysql
useradd -g mysql mysql
# 赋予权限
chown -R mysql:mysql /usr/local/mysql
# 链接配置文件
ln -s /usr/local/mysql/bin/mysql /usr/bin
```

> 有时候执行service mysql start命令报错env: /etc/init.d/mysql:权限不够问题
> 解决办法：执行下面的语句
> chmod a+wrx /etc/init.d/mysql

### 实例安装

#### 进入安装包的路径
```ps
cd /usr/local/mysql
```
#### 执行脚本
```ps
scripts/mysql_install_db --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --user=mysql
```
> /usr/local/mysql/my.cnf 为my.cnf默认路径

#### 添加端口到防火墙
```ps
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload
```

#### 添加服务，拷贝服务脚本到init.d目录，并设置开机启动
```ps
cp support-files/mysql.server /etc/init.d/mysql
chkconfig mysql on
```

# 启动MySQL
```ps
service mysql start
```
