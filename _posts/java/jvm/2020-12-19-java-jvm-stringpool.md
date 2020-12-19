---
layout: post
title:  "String和常量池引"
date:   2020-12-19
categories: java
tag: jvm
---

* content
{:toc}
# String类和常量池

- 字面量会到java堆中的常量池去找
- new的方式会在对中创建

![1608361537946](C:\Users\yanglei\AppData\Roaming\Typora\typora-user-images\1608361537946.png)

# String常量池的实现方式

- 字面量，双引号括起来的
- String.intern():他的作用是：如果运行时常量池中已经包含一个等于此Spring对象内容的字符串，则返回常量池中该字符串的引用

# String s1 = new String("abc")；这句话创建了几个字符串对象？

- 将创建一个或者两个字符串，如果常量池中已存在字符串常量"abc"，则只会在堆空间创建一个字符串常量"abc"。如果池中没有字符串常量"abc"，那么它将首先在池中创建，然后在堆空间中创建，因此将创建总共两个字符串对象。

# 8种基本类型的包装类和常量池

- Java基于类型的包装类的大部分都实现了常量池技术，及Byte、Short、Long、Character、Boolean；前面四种包装类默认创建了数字[-128,127]的相应类型的缓存数据，Character创建了数值在[0,127]范围的缓存数据，Boolean直接返回True or False。如果查出对应范围仍然会去创建新的对象。

  > 为啥把缓存设置为[-128,127]区间？ (参见issue、461)性能和资源之间的权衡。

- 算术运算符或比较运算符，对包装类型会有一个拆箱操作。

