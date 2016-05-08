---
title: Windows下安装MongoDB
date: 2016-05-03 21:56:54
tags: 
 - Install
categories:
 - MongDB
 - Windows
#: true
---
## 安装MongoDB
<!--more-->
#### 在mongo官网下载最新版本的安装包
> https://www.mongodb.org/downloads#production

下载完直接安装就可以了
#### 创建库目录和Log目录
进入mongo的安装目录，在“bin”目录的同级创建“data”目录，然后在“data”目录下创建“db”和“logs”目录
在logs目录下新建一个txt文件，将文件名改为MongoDB.log
> txt文件改为log格式文件

#### 安装mongo
管理员运行cmd，cd打开进入mongo的bin目录
``` c
cd C:\Program Files\MongoDB\Server\3.2\bin
```

安装mongo
```cpp
mongod --dbpath "C:\Program Files\MongoDB\Server\3.2\data\db" --logpath "C:\Program Files\MongoDB\Server\3.2\data\logs\MongoDB.log" --install --serviceName "MongoDB"
```
出现下面内容，生成一个以日期命名的log文件时就是安装成功了
![](http://7xt9fi.com1.z0.glb.clouddn.com/windows_mongo_install.png)
以后执行mongo都可以用windows的net启动命令就可以了
> net start/stop/restart MongoDB

## 小问题
#### 重启后无法启动mongo问题
当电脑突然断电或其他不正常关闭时，又可能会发生mongo无法启动的情况，网上的参考方法很多，但是我用的时候发现启动的时候不要指定log比较好，当然这样就找不到log文件了。这里先记录下启动的命令，更好的处理方法日后再做更改。
> mongod --dbpath "C:\Program Files\MongoDB\Server\3.2\data\db"

#### 使用客户端工具创建的库不能使用的问题
不知道是不是我下载的版本有问题还是什么，总之用客户端创建的数据库刷新后mongo就给自动删除了。这时候创建数据库的方法是手工建库。
进入mongo的bin目录，执行mongo.exe文件，输入下面命令
```cpp
use mydb
db
db.movie.insert({"name":"asdf"})
show dbs
```