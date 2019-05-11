---
layout: post
title:  "Centos 抓包"
date:   2019-05-10
categories: centos
tag: centos7
---

* content
{:toc}



# 我的使用 #

1、开启抓包命令

	tcpdump -i eth0 -w netinfo.pcap

2、下载 netinfo.pcap 

3、利用 Wireshark.exe 工具打开 netinfo.pcap 文件，并分析行管信息


