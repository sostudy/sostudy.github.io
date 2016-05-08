---
title: Linux下安装MongoDB
date: 2016-05-08 19:20:43
tags: 
 - Install
categories:
 - MongDB
 - Linux
#: true
---
### 安装MongoDB
<!--more-->
#### 在mongo官网下载最新版本的安装包
> https://www.mongodb.org/downloads#production

#### 解压mongodb程序包
解压文件
> 我这里为了方便，直接离线下载的程序包放到/usr/local/目录下，安装位置随意，注意下后面文件配置的路径就好了。

``` ps
tar zxvf mongodb-linux-x86_64-rhel70-3.2.6.tgz
```

为了后面操作方便，改下文件名
``` ps
mv mongodb-linux-x86_64-rhel70-3.2.6 mongodb
```

进入mongodb的目录
``` ps
cd mongodb
```

新建mongdb的data和log目录
``` ps
mkdir db
cd db
mkdir data
mkdir logs
```

创建MongoDB.log文件
``` ps
cd logs
vi MongoDB.log
#直接使用wq命令保存文件
```

#### 新建mongo的conf文件
``` ps
cd ../bin
vi mongodb.conf
```
> 填入下面的内容，使用wq命令保存退出

``` ps
dbpath=/usr/local/mongodb/db/data
logpath=/usr/local/mongodb/db/logs/MongoDB.log
port=27017
fork=true
nohttpinterface=true
```

### 配置MongoDB
#### mongodb的配置文件地址
输入下面的命令
``` ps
./mongod -f /usr/local/mongodb/bin/mongodb.conf
```
> 弹出下面的内容则证明配置成功

``` ps
[root@mjjw bin]# ./mongod -f /usr/local/mongodb/bin/mongodb.conf
about to fork child process, waiting until server is ready for connections.
forked process: 2292
child process started successfully, parent exiting
```

> 这时候已经可以正常启动关闭了，启动是直接运行bin目录下的mongo文件

#### 设置mongdb为自动启动
``` ps
vi /etc/rc.d/rc.local
```
> 在最后面填入下面的内容

``` ps
/usr/local/mongodb/bin/mongod --config /usr/local/mongodb/bin/mongodb.conf
```