---
layout: post
title:  "对象的内存布局"
date:   2020-12-19
categories: java
tag: jvm
---

* content
{:toc}
# 对象的内存布局

对象的内存布局分为3块区域：对象头、实例数据和对齐填充

## 对象头

- 包含两部分信息：第一部分专用于存储对象自身的运行时数据（hash码、GC分代年龄、锁状态标志等等），另一部分是类型指针，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。

## 实例数据

- 实例数据部分是对象真正存储的有效信息，也就是程序中定义的各种类型的字段内容。

## 对齐填充

- 不是必然存在，仅仅起占位作用。
- 因为Hotspot虚拟机的自动内存管理系统要求对象起始地址必须是8字节的整倍数，换句话说就是对象的大小必须是8字节的整倍数。而对象头部分正好是8字节的倍数（1倍或者2倍），因此，当对象实例数据部分没有对齐时，就需要通过对齐填充来补全。

























































