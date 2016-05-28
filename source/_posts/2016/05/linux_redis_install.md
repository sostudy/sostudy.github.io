---
title: CentOS7下Redis安装
date: 2016-05-28 16:17:37
tags: 
 - Install
categories: 
 - Redis
 - Linux
#toc: true
---
# Redis安装
<!--more-->
## 下载Redis
> http://redis.io/ 下载最新版的Redis

也可以使用wget直接下载redis
``` ps
wget http://download.redis.io/releases/redis-3.2.0.tar.gz
```
## 安装Redis
解压Redis
```ps
tar zxvf redis-3.2.0.tar.gz
mv redis-3.2.0 redis
cd redis
```

编译安装
```ps
yum -y install make gcc-c++ cmake bison-devel ncurses-devel libaio bison libaio libaio-devel perl-Data-Dumper net-tools

make
```
出现下面结果表示成功
>Hint: To run ‘make test’ is a good idea 
make[1]: Leaving directory `/opt/redis-3.2.0/src’

```ps
make install 
```
出现下面结果表示成功
>Hint: To run 'make test' is a good idea  
    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
make[1]: Leaving directory `/opt/redis-3.2.0/src'

```ps
cd src && make install
```
出现下面结果表示成功
>Hint: To run 'make test' is a good idea  
    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL instal
    
 此时Redis已经安装成功，直接执行redis-server就可以启动redis
# Redis使用脚本
当Redis多实例启动时，没有脚本需得启动多个redis.conf文件，比较麻烦，使用脚本启动比较方便。
## 脚本步骤

	1.  在/etc/rc.d/init.d/目录下新建redis文件，将脚本内容拷贝进去
	2. chkconfig --add redis   #注册服务
	3. chkconfig --level 345 redis on  #指定服务在3、4、5级别运行
	4. systemctl daemon-reload
	5. 多端口时新建redis.conf文件命名方法建议为“redis-6381.conf”这种格式以便跟脚本中对应，如以其他命名，脚本需做相应更改。
    
## 脚本参数
```ps
      service redis -p [port]  [start|stop|status|restart]
参数说明：
         -p [port] : 指定redis实例的端口，用于多实例的服务器
         start：启动指定端口的Redis服务
         stop：停止指定端口的Redis服务
         status：进程状态
         restart：先关闭Redis服务,再启动Redis服务
注：不指定端口时，脚本默认指定启动6379端口的redis
```
使用示例

    service redis -p 6381 start  #启动6381端口实例的redis
     /etc/init.d/redis  start  #默认启动6379端口实例的redis

## 脚本内容
```ps
#!/bin/bash
#chkconfig: 2345 55 25
#description: Starts,stops and restart the redis-server
#Ver:1.1  
#Write by ND chengh(200808)
#usage: ./script_name -p [port] {start|stop|status|restart}
# Source function library.
. /etc/rc.d/init.d/functions
# Source networking configuration.
. /etc/sysconfig/network
# Check networking is up.
[ "$NETWORKING" = "no" ] && exit 0
RETVAL=0
REDIS_PORT=6379
PID=
if [ "$1" = "-p" ]; then
    REDIS_PORT=$2
    shift 2
fi
REDIS_DIR="/data/redis-3.2.0"
REDIS="${REDIS_DIR}/src/redis-server"
PROG=$(basename $REDIS)
CONF="${REDIS_DIR}/port/redis-${REDIS_PORT}.conf"
if [ ! -f $CONF ]; then
   if [ -f "${REDIS_DIR}/port/redis.conf" ];then
      CONF="${REDIS_DIR}/port/redis.conf"
   else
      echo -n $"$CONF not exist.1";warning;echo
      exit 1
   fi
fi
PID_FILE=`grep "pidfile" ${CONF}|cut -d ' ' -f2`
PID_FILE=${PID_FILE:=/var/run/redis.pid}
LOCKFILE="/var/lock/subsys/redis-${REDIS_PORT}"
if [ ! -x $REDIS ]; then
    echo -n $"$REDIS not exist.2";warning;echo
    exit 0
fi
start() {
    echo -n $"Starting $PROG: "
    $REDIS $CONF
    RETVAL=$?
    if [ $RETVAL -eq 0 ]; then
        success;echo;touch $LOCKFILE
    else
        failure;echo
    fi
    return $RETVAL
}
stop() {
    echo -n $"Stopping $PROG: "
    if [ -f $PID_FILE ] ;then
       read PID <  "$PID_FILE" 
    else 
       failure;echo;
       echo -n $"$PID_FILE not found.";failure;echo
       return 1;
    fi
    if checkpid $PID; then
     kill -TERM $PID >/dev/null 2>&1
        RETVAL=$?
        if [ $RETVAL -eq 0 ] ;then
                success;echo 
                echo -n "Waiting for Redis to shutdown .."
         while checkpid $PID;do
                 echo -n "."
                 sleep 1;
                done
                success;echo;rm -f $LOCKFILE
        else 
                failure;echo
        fi
    else
        echo -n $"Redis is dead and $PID_FILE exists.";failure;echo
        RETVAL=7
    fi    
    return $RETVAL
}
restart() {
    stop
    start
}
rhstatus() {
    status -p ${PID_FILE} $PROG
}
hid_status() {
    rhstatus >/dev/null 2>&1
}
case "$1" in
    start)
        hid_status && exit 0
        start
        ;;
    stop)
        rhstatus || exit 0
        stop
        ;;
    restart)
        restart
        ;;
    status)
        rhstatus
        RETVAL=$?
        ;;
    *)
        echo $"Usage: $0 -p [port] {start|stop|status|restart}"
        RETVAL=1
esac
exit $RETVAL
```
> 注意你安装Redis的目录，要对脚本中目录做出相应更改。