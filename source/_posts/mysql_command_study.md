---
title: MySQL命令学习记录
date: 2016-05-02 21:37:54
tags: 
 - Command
categories:
 - MySQL
#: true
---
工作中用到的一些MySQL命令，记录下方便日后查询
<!--more-->
## 用户登录命令
### 本地登录
```sql
mysql -u root -p
```

### 远程登陆
```sql
Mysql  -P 端口号  -h  mysql主机名\ip -u（用户）  -p（密码）
```

## 创建用户命令
### 创建本地用户命令
<!--more-->
```sql
grant all privileges on *.* to admin@localhost identified by 'password' with grant option
```

### 创建远程用户
```sql
grant all privileges on *.* to admin@"%" identified by 'password' with grant option
```

## 查询命令
### 查看当前编码
```sql
show variables like 'character%';
```

### 查看当前数据库类型
```sql
show variables like 'character%';
```

### 查看死锁状态
```sql
show status like 'table%';
```