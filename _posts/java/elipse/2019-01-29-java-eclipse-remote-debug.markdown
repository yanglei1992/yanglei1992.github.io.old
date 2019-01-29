---
layout: post
title:  "eclipse远程调试centos7下的springboot程序"
date:   2019-01-29 14:30:39 +0800
categories: java
tag: eclipse
---

* content
{:toc}

# eclipse的设置项 #
本地设置`debug configuration`

![install_step](/images/2019-01/2019-01-29-java-eclipse-remote-debug/TL20190129151436.png)

从里面选择`debug configuration`

![install_step](/images/2019-01/2019-01-29-java-eclipse-remote-debug/TL20190129150454.png)

> 注意：端口号不要与项目端口号写的一样，这个端口是debug调试连接使用的端口号

# 远程服务的启动配置 #

用如下参数启动项目：

    java -jar -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=4004 springboot-test-1.0-SNAPSHOT.jar

> -Xdebug：启用调试特性。、
> 
> -Xrunjdwp:<sub-options>
> 在目标 VM 中加载 JDWP 实现。它通过传输和 JDWP 协议与独立的调试器应用程序通信。
> 
> **下面介绍一些特定的子选项。**
>  
- transport：这里通常使用套接字传输。但是在 Windows 平台上也可以使用共享内存传输。
> 
- server：如果值为 y，目标应用程序监听将要连接的调试器应用程序。否则，它将连接到特定地址上的调试器应用程序。
> 
- address：这是连接的传输地址。如果服务器为 n，将尝试连接到该地址上的调试器应用程序。否则，将在这个端口监听连接。
> 
- suspend：如果值为 y，目标 VM 将暂停，直到调试器应用程序进行连接。

# 防火墙配置 #

想本地调试远程服务，需要开启远程服务的防火墙端口。

开启调试端口的防火墙

    firewall-cmd --zone=public --add-port=4004/tcp

重新载入

    firewall-cmd --reload

# 调试 #
在本地代码中，将需要调试的部分打上断点，就可以在本地调试远程服务了。
