---
layout: post
title:  "mysql 查询日志"
date:   2019-03-04
categories: mysql
tag: select
---

* content
{:toc}

# MySQL查询日志简介 #


# MySQL 中 general_log 的使用 #

## general_log 参数介绍 ##

general_log的查询参数有三个：general_log、 general_log_file、 log_output ，这三个参数都是动态参数，可以随时动态修改。

- general_log：全局动态变量，默认关闭 

- general_log_file：全局动态变量，日志文件名，不指定的话默认为hostname。log，位于数据目录下。

- log_output ：全局动态变量，可取FILE、TABLE、NONE。

> 其中TABLE存储方式比较方便按条件检索。
> 
> - 若指定为NONE，则即使general_log开启了也不会记录log。
> - 若log_output指定为TABLE，则会在mysql数据库下边创建一个general_log表。需要注意的是该参数不仅仅影响general的存储方式还影响slow的存储方式，这一点需要特别注意。
	

## general_log 日志配置与参数查询 ##

* general_log 参数的配置：

	mysql > set global general_log = on;

	mysql > set global log_output = FILE;

	mysql > set global general_log_file = 'hostname.log';

* general_log 参数的查询：

	mysql > show variables like '%general_log%';

	mysql > show variables like 'log_output';

> 在查询sql语句之后，在对应的  C:/ProgramData/MySQL/MySQL Server 5.6/data/hostname.log 查看对应的log记录
> 
> set [global \| session] ...
> 
>  - global 全局变量，对所有连接都有效,
>  - session 会话变量，每一个连接单独拥有的变量，不影响其他连接。

![/images/2019-03/2019-03-04-mysql-select-log/TL20190304132321.png](/images/2019-03/2019-03-04-mysql-select-log/TL20190304132321.png)

# MySQL 中 slow_log 的使用 #

## slwo_log 参数介绍 ##

- slow_query_log:是否开启慢查询日志，[1 \| 0] 或者 [ON \| OFF]

- slow_query_log_file: MySQL 数据库(5.6及以上版本)慢查询日志存储路径。可以不设置参数，系统会默认给一个缺省的文件 HOST_NAME-slow.log

- long_query_time:慢查询的阈值，当查询时间查过设定的阈值时，记录该 SQL 语句到慢查询日志。

- log_queries_not_using_indexes:设置为 ON ,可以捕获到所有未使用索引的SQL语句（不建议开启）

- log_output:日志存储方式。可取FILE、TABLE。

> MySQL数据库支持同时两种日志存储方式，配置的时候以逗号隔开即可，如：log_output='FILE,TABLE'。
> 
> - log_output='FILE'，表示将日志存入文件，默认值是'FILE'。      
> - log_output='TABLE'，表示将日志存入数据库，这样日志信息就会被写入到 mysql.slow_log 表中。
> 
> 日志记录到系统的专用日志表中，要比记录到文件耗费更多的系统资源，因此对于需要启用慢查询日志，又需要能够获得更高的系统性能，那么建议优先记录到文件。

## slwo_log 日志配置与参数查询 ##

	mysql> set global slow_query_log=ON;

	mysql> set global slow_query_log_file='slow_query_log.log';

	mysql> set global long_query_time=2;

> 修改完慢连接查询时间后，发现 show variables like 'long_query_time'; 的值没有变化;其实 long_query_time 在 global 里面的只已经发生了改变，重新连接 mysql 会发现 long_query_time 的值已经改变

# 参考资料 #

[https://blog.csdn.net/zyz511919766/article/details/49335949](https://blog.csdn.net/zyz511919766/article/details/49335949)

[http://www.cnblogs.com/kerrycode/p/7130403.html](http://www.cnblogs.com/kerrycode/p/7130403.html)

[http://www.cnblogs.com/hjqjk/p/Mysqlslowlog.html](http://www.cnblogs.com/hjqjk/p/Mysqlslowlog.html)
