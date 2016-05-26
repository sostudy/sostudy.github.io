---
title: Percona Tookit学习手册 - pt-query-digest
date: 2016-05-10 12:06:37
tags: 
 - Tookit
categories: MySQL
#toc: true
---
## pt-query-digest介绍
<!--more-->
pt-query-digest是用于分析mysql慢查询的一个工具，它可以分析binlog、General log、slowlog，也可以通过SHOWPROCESSLIST或者通过tcpdump抓取的MySQL协议数据来进行分析。可以把分析结果输出到文件中，分析过程是先对查询语句的条件进行参数化，然后对参数化以后的查询进行分组统计，统计出各查询的执行时间、次数、占比等，可以借助分析结果找出问题进行优化。

### 慢查询日志开启方法
在mysql的my.ini文件中的mysqld中添加下面参数，然后重启mysql。
``` sql
slow_query_log_file='/home/mysql/sql_log/mysql-slow.log'  //设置慢查询日志位置
log_queries_not_using_indexes=on  //设置是否把没有使用索引的SQL放到慢查询日志中,off是没有开启,on是开始
slow_query_log=on  //设置开启慢查询日志,off是没有开启,on是开始
long_query_time=1  //查过多少秒的查询设置到慢查询日志中,单位是秒
```
> long_query_time=1设置后，实际上mysql也会把一些小于1S的操作记录下来，主要是因为索引的影响，可以直接把额外的记录忽略。

### pt-query-digest语法
> pt-query-digest [OPTIONS] [FILES] [DSN]

--create-review-table  当使用--review参数把分析结果输出到表中时，如果没有表就自动创建。
--create-history-table  当使用--history参数把分析结果输出到表中时，如果没有表就自动创建。
--filter  对输入的慢查询按指定的字符串进行匹配过滤后再进行分析
--limit限制输出结果百分比或数量，默认值是20,即将最慢的20条语句输出，如果是50%则按总响应时间占比从大到小排序，输出到总和达到50%位置截止。
--host  MySQL服务器地址
--user  mysql用户名
--password  mysql用户密码
--history 将分析结果保存到表中，分析结果比较详细，下次再使用--history时，如果存在相同的语句，且查询所在的时间区间和历史表中的不同，则会记录到数据表中，可以通过查询同一CHECKSUM来比较某类型查询的历史变化。
--review 将分析结果保存到表中，这个分析只是对查询条件进行参数化，一个类型的查询一条记录，比较简单。当下次使用--review时，如果存在相同的语句分析，就不会记录到数据表中。
--output 分析结果输出类型，值可以是report(标准分析报告)、slowlog(Mysql slow log)、json、json-anon，一般使用report，以便于阅读。
--since 从什么时间开始分析，值为字符串，可以是指定的某个”yyyy-mm-dd [hh:mm:ss]”格式的时间点，也可以是简单的一个时间值：s(秒)、h(小时)、m(分钟)、d(天)，如12h就表示从12小时前开始统计。
--until 截止时间，配合—since可以分析一段时间内的慢查询，格式与since相同。

### pt-query-digest使用方法

#### 直接使用
``` sql
pt-query-digest /usr/local/mysql/data/slow.log | more
```

#### 输出到数据库
``` sql
pt-query-digest slow.log -review \
h=127.0.0.1,D=test,p=root,P=3306,u=root,t=query_review \
--create-reviewtable \
--review-history t= hostname_slow
```

#### 分析最近12小时内的查询
``` sql
pt-query-digest  --since=12h  slow.log > slow_report2.log
```

#### 分析指定时间范围内的查询
``` sql
pt-query-digest slow.log --since '2014-04-17 09:30:00' --until '2014-04-17 10:00:00'> > slow_report3.log
```

#### 分析指含有select语句的慢查询
``` sql
pt-query-digest--filter '$event->{fingerprint} =~ m/^select/i' slow.log> slow_report4.log
```

#### 针对某个用户的慢查询
``` sql
pt-query-digest--filter '($event->{user} || "") =~ m/^root/i' slow.log> slow_report5.log
```

#### 查询所有所有的全表扫描或full join的慢查询
``` sql
pt-query-digest--filter '(($event->{Full_scan} || "") eq "yes") ||(($event->{Full_join} || "") eq "yes")' slow.log> slow_report6.log
```

#### 把查询保存到query_review表
``` sql
pt-query-digest  --user=root –password=abc123 --review  h=localhost,D=test,t=query_review--create-review-table  slow.log
```

#### 把查询保存到query_history表
``` sql
pt-query-digest  --user=root –password=abc123 --review  h=localhost,D=test,t=query_history--create-review-table  slow.log_20140401
pt-query-digest  --user=root –password=abc123 --review  h=localhost,D=test,t=query_history--create-review-table  slow.log_20140402
```

#### 通过tcpdump抓取mysql的tcp协议数据，然后再分析
pt-query-digest对于抓包有一定的格式。(-x -nn -q -tttt)
``` sql
tcpdump -s 65535 -x -nn -q -tttt -i any -c 1000 port 3306 > mysql.tcp.txt
pt-query-digest --type tcpdump mysql.tcp.txt> slow_report9.log

# -s:源端口
# -c:抓包的数量
```

#### 分析binlog
``` sql
mysqlbinlog mysql-bin.000093 > mysql-bin000093.sql
pt-query-digest  --type=binlog  mysql-bin000093.sql > slow_report10.log
```

#### 分析general log
``` sql
pt-query-digest  --type=genlog  localhost.log > slow_report11.log
```

### 分析结果详解
#### 第一部分
``` sql
# 18.4s user time, 90ms system time, 27.27M rss, 223.26M vsz
# Current date: Tue May 10 11:51:19 2016
# Hostname: mjjw
# Files: /usr/local/mysql/data/mjjw-slow.log
# Overall: 29.20k total, 25 unique, 0.41 QPS, 0.51x concurrency __________
# Time range: 2016-05-09 16:12:24 to 2016-05-10 11:51:19
# Attribute          total     min     max     avg     95%  stddev  median
# ============     ======= ======= ======= ======= ======= ======= =======
# Exec time         36179s   120us     27s      1s     10s      3s   541us
# Lock time            15s       0   454ms   501us   839us     4ms   194us
# Rows sent         98.30k       0    1000    3.45   19.46    8.51    0.99
# Rows examine       5.22G       0   1.48M 187.54k   1.46M 367.29k   28.75
# Query size        46.97M      36   4.46k   1.65k   3.52k   1.46k  363.48
```
Overall: 总共有多少条查询
unique: 唯一查询数量，即对查询条件进行参数化以后，总共有多少个不同的查询，该例为25。
Time range: 查询执行的时间范围。
total: 总计   min:最小   max: 最大  avg:平均  95%: 把所有值从小到大排列，位置位于95%的那个数，这个数一般最具有参考价值。
median: 中位数，把所有值从小到大排列，位置位于中间那个数。

#### 第二部分
``` sql
# Profile
# Rank Query ID           Response time    Calls R/Call  V/M   Item
# ==== ================== ================ ===== ======= ===== ===========
#    1 0x3C82B414C14E5C06 14439.8524 39.9%  3220  4.4844  9.20 SELECT STUDENT STUDENT_SCHOOL SCHOOL_TABLE STUDENT_ORGAN ORGAN ORDER_FORM T_?_USE
R_INFO STUDENT_SELL T_?_USER_INFO STUDENT_CONTACT_PERSON LEA_COU_ONE STUDENT_ASSISTANT T_?_USER_INFO LCO_ALLOT_TEACHER T_?_USER_INFO USER_DEPART
MENT DEPARTMENT COURSE
#    2 0xB86C64C70BAA831C 10486.1545 29.0%   951 11.0265  0.55 SELECT STUDENT STUDENT_SCHOOL SCHOOL_TABLE STUDENT_ORGAN ORGAN ORDER_FORM T_?_USE
R_INFO STUDENT_SELL T_?_USER_INFO STUDENT_CONTACT_PERSON LEA_COU_ONE STUDENT_ASSISTANT T_?_USER_INFO LCO_ALLOT_TEACHER T_?_USER_INFO USER_DEPART
MENT DEPARTMENT COURSE
#    3 0x3A8823AD8857EA9D  4635.8130 12.8%  3200  1.4487  1.33 SELECT STUDENT STUDENT_SCHOOL SCHOOL_TABLE STUDENT_ORGAN ORGAN ORDER_FORM T_?_USE
R_INFO STUDENT_SELL T_?_USER_INFO STUDENT_CONTACT_PERSON LEA_COU_ONE STUDENT_ASSISTANT T_?_USER_INFO LCO_ALLOT_TEACHER T_?_USER_INFO USER_DEPART
MENT DEPARTMENT COURSE
#    4 0x86AC36DBACB13484  4626.3197 12.8%  3200  1.4457  2.07 SELECT STUDENT STUDENT_SCHOOL SCHOOL_TABLE STUDENT_ORGAN ORGAN ORDER_FORM T_?_USE
R_INFO STUDENT_SELL T_?_USER_INFO STUDENT_CONTACT_PERSON LEA_COU_ONE STUDENT_ASSISTANT T_?_USER_INFO LCO_ALLOT_TEACHER T_?_USER_INFO USER_DEPART
MENT DEPARTMENT COURSE
#    5 0x0363FA1F2388C4D2  1527.4343  4.2%  2268  0.6735  0.29 SELECT STUDENT STUDENT_SCHOOL SCHOOL_TABLE STUDENT_ORGAN ORGAN ORDER_FORM T_?_USE
R_INFO STUDENT_SELL T_?_USER_INFO STUDENT_CONTACT_PERSON LEA_COU_ONE STUDENT_ASSISTANT T_?_USER_INFO LCO_ALLOT_TEACHER T_?_USER_INFO USER_DEPART
MENT DEPARTMENT COURSE
# MISC 0xMISC               463.5191  1.3% 16357  0.0283   0.0 <20 ITEMS>
```
这部分对查询进行参数化并分组，然后对各类查询的执行情况进行分析，结果按总执行时长，从大到小排序。
Response: 总的响应时间。
time: 该查询在本次分析中总的时间占比。
calls: 执行次数，即本次分析总共有多少条这种类型的查询语句。
R/Call: 平均每次执行的响应时间。
Item : 查询对象


#### 第三部分
``` sql
# Query 1: 1.04 QPS, 4.65x concurrency, ID 0x3C82B414C14E5C06 at byte 45899007
# Scores: V/M = 9.20
# Time range: 2016-05-10 10:59:36 to 11:51:19
# Attribute    pct   total     min     max     avg     95%  stddev  median
# ============ === ======= ======= ======= ======= ======= ======= =======
# Count         11    3220
# Exec time     39  14440s    89ms     27s      4s     17s      6s   816ms
# Lock time     15      2s   370us   165ms   688us   881us     3ms   467us
# Rows sent      3   3.14k       1       1       1       1       0       1
# Rows examine  36   1.91G 244.46k   1.48M 620.56k   1.46M 573.76k 233.54k
# Query size    17   8.38M   2.67k   2.67k   2.67k   2.67k       0   2.67k
# String:
# Hosts
# Users        admin
# Query_time distribution
#   1us
#  10us
# 100us
#   1ms
#  10ms  ##
#    1s  #############
#  10s+  #############################
# Tables
#    SHOW TABLE STATUS FROM `mysqlnew` LIKE 'STUDENT'\G
#    SHOW CREATE TABLE `mysqlnew`.`STUDENT`\G
#    SHOW TABLE STATUS FROM `mysqlnew` LIKE 'STUDENT_SCHOOL'\G
#    SHOW CREATE TABLE `mysqlnew`.`STUDENT_SCHOOL`\G
#    SHOW TABLE STATUS FROM `mysqlnew` LIKE 'SCHOOL_TABLE'\G
#    SHOW CREATE TABLE `mysqlnew`.`SCHOOL_TABLE`\G
#    SHOW TABLE STATUS FROM `mysqlnew` LIKE 'STUDENT_ORGAN'\G
#    SHOW CREATE TABLE `mysqlnew`.`STUDENT_ORGAN`\G
#    SHOW TABLE STATUS FROM `mysqlnew` LIKE 'ORGAN'\G
#    SHOW CREATE TABLE `mysqlnew`.`ORGAN`\G
#    SHOW TABLE STATUS FROM `mysqlnew` LIKE 'ORDER_FORM'\G
#    SHOW CREATE TABLE `mysqlnew`.`ORDER_FORM`\G
#    SHOW TABLE STATUS FROM `mysqlnew` LIKE 'T_0_USER_INFO'\G
#    SHOW CREATE TABLE `mysqlnew`.`T_0_USER_INFO`\G
#    SHOW TABLE STATUS FROM `mysqlnew` LIKE 'STUDENT_SELL'\G
#    SHOW CREATE TABLE `mysqlnew`.`STUDENT_SELL`\G
#    SHOW TABLE STATUS FROM `mysqlnew` LIKE 'STUDENT_CONTACT_PERSON'\G
#    SHOW CREATE TABLE `mysqlnew`.`STUDENT_CONTACT_PERSON`\G
#    SHOW TABLE STATUS FROM `mysqlnew` LIKE 'LEA_COU_ONE'\G
#    SHOW CREATE TABLE `mysqlnew`.`LEA_COU_ONE`\G
#    SHOW TABLE STATUS FROM `mysqlnew` LIKE 'STUDENT_ASSISTANT'\G
#    SHOW CREATE TABLE `mysqlnew`.`STUDENT_ASSISTANT`\G
#    SHOW TABLE STATUS FROM `mysqlnew` LIKE 'LCO_ALLOT_TEACHER'\G
#    SHOW CREATE TABLE `mysqlnew`.`LCO_ALLOT_TEACHER`\G
#    SHOW TABLE STATUS FROM `mysqlnew` LIKE 'USER_DEPARTMENT'\G
#    SHOW CREATE TABLE `mysqlnew`.`USER_DEPARTMENT`\G
#    SHOW TABLE STATUS FROM `mysqlnew` LIKE 'DEPARTMENT'\G
#    SHOW CREATE TABLE `mysqlnew`.`DEPARTMENT`\G
#    SHOW TABLE STATUS FROM `mysqlnew` LIKE 'COURSE'\G
#    SHOW CREATE TABLE `mysqlnew`.`COURSE`\G
# EXPLAIN /*!50100 PARTITIONS*/
select count(1)from(SELECT ATM_0.STUDENT_ID T_448_0,ATM_0.STU_NAME AFM_1,ATM_0.STU_SEX AFM_2,ATM_0.STU_SEX DIC_AFM_2,ATM_0.STU_PHONE AFM_3,ATM_0
.STU_TYPE AFM_4,ATM_0.STU_TYPE DIC_AFM_4,ATM_0.EDUCATION AFM_5,ATM_0.EDUCATION DIC_AFM_5,ATM_0.ISVIP AFM_10,ATM_0.ISVIP DIC_AFM_10,ATM_0.STU_ABR
OADTIME AFM_12,ATM_0.STU_ABROADTIME DIC_AFM_12,ATM_0.F_33345 AFM_13,ATM_0.SK_STYLE AFM_14,ATM_0.SK_STYLE DIC_AFM_14,ATM_0.CREATE_DATE AFM_16,ATM
_0.IF_XY AFM_25,ATM_0.IF_XY DIC_AFM_25,ATM_0.STUDENT_ID LMF_ID,ATM_24_0.STU_ID RP_STU_ID_0,ATM_23_0.STU_ID RP_STU_ID_1,ATM_17_0.STU_ID RP_STU_ID
_2,ATM_11_0.STU_ID RP_STU_ID_4,ATM_8_0.STU_ID RP_STU_ID_5,ATM_6_0.STU_ID RP_STU_ID_6 FROM STUDENT ATM_0 INNER JOIN(SELECT ATM_24_0.STU_ID FROM S
TUDENT_SCHOOL ATM_24_0 LEFT JOIN SCHOOL_TABLE ATM_24_1 ON ATM_24_1.SCHOOL_TABLE_ID=ATM_24_0.SCH_ID WHERE 1=1 AND(ATM_24_1.SCHOOL_TABLE_ID IN(1))
```
这部分是每种查询的详细情况
最上面的表格列出了执行次数、最大、最小、平均、95%等各项目的统计。
Databases: 库名
Users: 各个用户执行的次数（占比）
Query_time distribution : 查询时间分布, 长短体现区间占比，本例中1s-10s之间查询数量是10s以上的两倍。
Tables: 查询中涉及到的表
Explain: 示例

> 官方文档：http://www.percona.com/doc/percona-toolkit/2.2/pt-query-digest.html

> 参考：http://blog.csdn.net/seteor/article/details/24017913