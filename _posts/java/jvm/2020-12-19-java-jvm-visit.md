---
layout: post
title:  "对象的访问定位"
date:   2020-12-19-12-21
categories: java
tag: jvm
---

* content
{:toc}
# 对象的访问定位

java程序通过栈上的 reference 数据来操作堆上的具体对象

对象的访问方式2种：**使用句柄，直接指针**

## 句柄

- 如果使用句柄的话，那么java堆中将会划分出一块内存来作为句柄池， reference 中存储的就是对象的句柄地址，而句柄中包含了对象实例数据 与 类型数据各自的具体地址信息；

![1608360700073](C:\Users\yanglei\AppData\Roaming\Typora\typora-user-images\1608360700073.png)

## 直接指针

- 如果使用直接指针访问，那么java堆对象的布局中就必须考虑如何放置访问类型数据的相关信息，而 refrence 中存储的直接就是对象的地址

![1608360894649](C:\Users\yanglei\AppData\Roaming\Typora\typora-user-images\1608360894649.png)

- 使用句柄来访问的最好好处是 reference 中存储的是稳定的句柄地址，在对象被移动时只会改变句柄中的实例数据指针，而 reference 本身不需要修改
- 使用直接指针访问方式最大的好处就是速度快，他节约了一次指针定位的时间开销。

































