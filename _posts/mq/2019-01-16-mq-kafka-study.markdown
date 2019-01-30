---
layout: post
title:  "Windows平台下搭建Kakfa环境"
date:   2019-01-16 10:21:49 +0800
categories: mq
tag: kafka
---

* content
{:toc}


## 准备工作 ##

1. 安装jdk环境：本次演示安装的是JDK1.8。下载地址：[http://www.oracle.com/technetwork/java/javase/downloads/index.html ](http://www.oracle.com/technetwork/java/javase/downloads/index.html )
2. 安装kafka的程序包：本次演示安装的是kafka_2.11-2.1.0，下载后解压到文件夹`E:\Programfiles\kafka_2.11-2.1.0`下。下载地址：[http://kafka.apache.org/downloads](http://kafka.apache.org/downloads )


> 1. 安装kafka之前需要安装zookeeper服务，我这里使用的kafka_2.11-2.1.0中自带zookeeper，所以安装过程中不需要安装zookeeper了。
> 
> 2. kafka_2.11-2.1.0解压后的目录介绍：
>     .\bin目录下存放的是程序运行的脚本，.sh文件是linux下的运行脚本，.bat是windows下运行的脚本。
>     
>     .\config目录下存放的是服务运行的配置文件
>     
>     .libs目录下存放的是打包好的jar包，这个版本自带了zookeeper的jar包
>     
> 3. kafka的解压目录不要有空格，例如:`E:\Program Files\`

## 启动服务 ##
启动服务之前先配置服务的数据输出目录，方便管理，

1.  先在`E:\Programfiles\kafka_2.11-2.1.0`文件夹底下创建`zookeeper_data`目录与`kafka_server_data`目录。
2.  修改配置文件：`E:\Programfiles\kafka_2.11-2.1.0\config\zookeeper.properties`
    
    	# the directory where the snapshot is stored.
    	dataDir=E:\Programfiles\kafka_2.11-2.1.0\zookeeper_data
    	# the port at which the clients will connect
    	clientPort=2181
    	# disable the per-ip limit on the number of connections since this is a non-production config
		maxClientCnxns=0
3.  修改配置文件：`E:\Programfiles\kafka_2.11-2.1.0\config\server.properties`
		
		############################# Log Basics #############################
		
		# A comma separated list of directories under which to store log files
		log.dirs=E:\Programfiles\kafka_2.11-2.1.0\kafka_server_data
		
		# The default number of log partitions per topic. More partitions allow greater
4.  在`E:\Programfiles\kafka_2.11-2.1.0`目录下启动zookeeper服务：

		.\bin\windows\zookeeper-server-start.bat .\config\zookeeper.properties

    ![kafka_png](/images/2019-01/2019-01-16-mq-kafka-study/mq_kafka01.png)

5.  在`E:\Programfiles\kafka_2.11-2.1.0`目录下启动kafka服务：

		.\bin\windows\kafka-server-start.bat .\config\server.properties

    ![kafka_png](/images/2019-01/2019-01-16-mq-kafka-study/mq_kafka02.png) 

## 创建测试数据 ##

1. 创建一个主题

		.\bin\windows\kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic ylTest

    ![kafka_png](/images/2019-01/2019-01-16-mq-kafka-study/mq_kafka03.png)

2. 查看主题列表

		.\bin\windows\kafka-topics.bat --list --zookeeper localhost:2181

    ![kafka_png](/images/2019-01/2019-01-16-mq-kafka-study/mq_kafka04.png)

3. 启动生产者与发送消息

		.\bin\windows\kafka-console-producer.bat --broker-list localhost:9092 --topic ylTest

    ![kafka_png](/images/2019-01/2019-01-16-mq-kafka-study/mq_kafka05.png)

4. 启动消费者与接收消息

		.\bin\windows\kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic ylTest --from-beginning

    ![kafka_png](/images/2019-01/2019-01-16-mq-kafka-study/mq_kafka06.png)

# 参考资料 #

[https://www.cnblogs.com/UniqueColor/p/8657319.html](https://www.cnblogs.com/UniqueColor/p/8657319.html)

