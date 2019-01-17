---
layout: post
title:  "jvm(HotSpot)的垃圾回收器"
date:   2018-12-27 21:30:39 +0800
categories: java
tag: jvm
---

* content
{:toc}

# jvm的7种垃圾回收器 #

## 1.Serial收集器 ##

Serial收集器时最基本、发展历史最悠久的收集器。是单线程的收集器。它在进行垃圾收集时，必须暂停其他所有的工作线程，直到它收集完成。
Serial收集器依然是虚拟机运行在Cleent模式下默认新生代收集器，对于运行在Client模式下的虚拟机来说是一个很好的选择。
## 2.ParNew收集器 ##

ParNew收集器其实就是Serial收集器的多线程版本，除了使用多线程进行垃圾收集之外，其余行为包括Serial收集器可用的所有控制参数、收集算法、Stop The Worl、对象可分配规则、回收策略等都与Serial收集器完全一样。
ParNew收集器是许多运行在Server模式下的虚拟机中首选新生代收集器，其中有一个与性能无关但很重要的原因是除了Serial收集器之外，目前只有arNew它能与CMS收集器配合工作。
## 3.Parallel Scavenge(并行回收)收集器 ##

Parallel Scavenge收集器是一个新生代收集器，它也是使用复制算法的收集器，又是并行的多线程收集器

该收集器的目标是达到一个可控制的吞吐量（Throughput）。所谓吞吐量就是CPU用于运行用户代码的时间与CPU总消耗时间的比值，即吞吐量=运行用户代码时间/（运行用户代码时间+垃圾收集时间）
停顿时间越短就越适合需要与用户交互的程序良好的响应速度能提升用户体验，二高吞吐量则可用高效率的利用CPU时间，尽快完成程序的运算任务，主要适合在后台运算二不需要太多的交互的任务。

Parallel Scavenga收集器提供两个参数用于精确控制吞吐量，分别是控制最大垃圾收起停顿时间的-XX:MaxGCPauseMillis参数以及直接设置吞吐量大小的-XX:GCTimeRatio参数

Parallel Scavenge收集器还有一个参数：-XX:+UseAdaptiveSizePPolicy/这是一个开关参数，当这个参数打开后，就不需要手工指定新生代的大小（-Xmn）、Eden与Survivor去的比例（-XX:SurvivorRatio）、晋升老年代对象年龄（-XX:PretenureSizeThreshold）等细节参数，只需要把基本的内存数据设置好（如-Xmx设置最大堆），然后使用MaxGvPauseMillis参数或者SGCTimeRation参数给虚拟机设立一个优化目标。

自适应调节策略也是paralel Scavenge收集器与ParNew收集器的一个重要区别

## 4.Serial Old收集器 ##
## 5.aralled Old收集器 ##
## 6.CMS收集器 ##
## 7.G1收集器 ##

## jvm垃圾的回收作用范围 ##
    

## 参考资料 ##

[https://www.cnblogs.com/chengxuyuanzhilu/p/7088316.html](https://www.cnblogs.com/chengxuyuanzhilu/p/7088316.html)


