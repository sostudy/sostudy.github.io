---
title: MySQL参数学习
date: 2016-05-28 16:17:37
tags: 
 - Optimize
categories: 
 - MySQL
#toc: true
---
# MySQL参数学习
<!--more-->
## [mysqld_safe]
pid-file=/usr/mysql/run/localhost.pid
> 单独设置一个pid的存放地址，以保护不被误删

##[mysql]
port=3306
> 端口号

default-character-set=utf8
> 编码格式

no-auto-rehash
> 自动补全，如同linux下的table键

## [mysqld]
### 目录
basedir=/usr/mysql
> MySQL主目录

datadir=/usr/data
> mysql 全局数据文件及结构的存放位置；表数据（包括innodb引擎），索引，日志（除非单独设置）等文件都会存放在这里。

tmpdir=/tmp
> 创建临时表文件目录

lc_messages=en_US
> 生成文件的语言环境，建议为en_US

lc_messages_dir=/u02/mysql/share
> 语言环境文件路径

socket=/u02/mysql/run/mysql.sock
> 套字文件，保护不被误删，每次重启mysql都会重新创建，在[client]也要设置相同位置

### 日志
#### 错误日志
log_error=/u02/mysql/log/localhost.log
> 指定Log错误文件路径

#### 慢查询日志
slow_query_log_file=/u02/mysql/log/slow.log
> 指定慢查询日志文件路径

slow_query_log=on
> 开启慢查询日志“【OFF】是关闭日志”，用来分析执行时间较长的SQL

long_query_time=2
> 设置慢查询日志多长时间后才会记录，单位为s

log_queries_not_using_indexes=on
> 设置是否把没有使用索引的SQL放到慢查询日志中,off是没有开启,on是开

#### 普通日志
general_log=off
> 是否开启普通日志，此参数开启后会记录MySQL所有执行sql记录，包括错误的sql，除特殊情况不建议开启

general_log_file=/u02/mysql/log/general.log
> 普通日志文件位置

#### Innodb设置
innodb_data_home_dir=/u02/data
> innodb引擎的共享表空间数据文件根目录

innodb_data_file_path=ibdata1:512M;ibdata2:16M:autoextend
> 指定了所有InnoDB数据文件初始大小分配，最大分配以及超出起始分配界线时是否应当增加文件的大小，位置由innodb_data_home_dir指定。

innodb_log_group_home_dir=/u02/data
> 此参数确定日志文件组中的文件的位置，日志组中文件的个数由innodb_log_files_in_group确定，此位置设置默认为MySQL的datadir 

innodb_log_files_in_group=2
> 为提高性能，MySQL可以以循环方式将日志文件写到多个文件。推荐设置为2M

innodb_buffer_pool_size=200M
> 如果只用innodb的话可使用70%的可用内存，建议不要超过80%，否则会影响swap

innodb_buffer_pool_instances=4
> 可以开启多少个内存缓冲池，与size搭配使用，大的size建议instances设置为1.

innodb_log_file_size=200M
> 日志文件的大小，此参数越大性能越好，但会增加故障恢复所用时间。依据服务器使用情况设置，推荐设置为0.25 * innodb_buffer_pool_size

innodb_log_buffer_size=5M
> 确定些日志文件所用的内存大小，以M为单位。缓冲区更大能提高性能，但意外的故障将会丢失数据。如果它的值设置太高了，可能会浪费内存 ，它每秒都会刷新一次，因此无需设置超过1秒所需的内存空间。

innodb_flush_log_at_trx_commit=1
> 参数可供选择0-1-2，0时性能最好，2时安全性最高，默认选中1

innodb_additional_mem_pool_size=20M
> 设置 InnoDB 存储的数据目录信息和其它内部数据结构的内存池大小，根据项目的InnoDB表的数目相应地增加。参数对系统整体性能并无太大的影响，所以只要能存放需要的数据即可。推荐1/200*buffer_pool
