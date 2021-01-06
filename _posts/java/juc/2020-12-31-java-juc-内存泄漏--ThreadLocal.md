---
layout: post
title:  "java内存泄漏--ThreadLocal"
date:   2020-12-31
categories: java
tag: juc
---

* content
{:toc}
# 内存泄漏

- 内存泄漏是堆内存中不再使用的对象，但是垃圾收集器没有将它们删除的情况，因此它们将会被不必要的一直存在。这样会消耗内存资源，降低系统性能，最终可能导致OOM发生。

# Java中内存泄漏的类型

## static字段引起内存泄漏

- 静态的资源一般与整个应用拥有相匹配的生命周期
- 尽量减少使用。

## 未关闭的资源导致内存泄漏

- 创建连接或打开流时，JVM会为这些资源分配内存。如果连接没有关闭，会导致占有内存。
- 可以在finally块中或try-with-resources块中关闭

## 不正确的hashCode和equals

- 重写不合理会导致内存泄漏
- 用最佳的方式重写hashCode和equals

## 引用了外部类的内部类

- 使用了一个非静态的内部类对象，这个时候内部类依赖于外部类对象。即使用完了内部类，这个对象也不会被收集
- 如果内部类不访问外部类的成员，可以转换为静态类

## 重写了finalize方法

## ThreadLoacl造成内存泄漏

# ThreadLocal

- 每个线程拥有自己本地专属的变量，可以用ThreadLoal类。
- ThreadLoal类让每个线程绑定自己的值，存储线程私有数据。避免线程出现竞争。
- ThreadLocal类是使用ThreadLoalMap类来实现定制化的HashMap的。调用ThreadLocal类的set或get方法，实际调用的是ThreadLocalMap类的get、set方法。
- 变量最终是存在ThreadLocalMap中。
- ThreadLocalMap可以存以ThreadLocal为key，Object对象为value的值对。
- ThreadLocalMap中的key是ThreadLocal的弱引用，而value是强引用。所以，如果 ThreadLocal 没有被外部强引用的情况下，在垃圾回收的时候，key 会被清理掉，而 value 不会被清理掉。这样一来，ThreadLocalMap 中就会出现key为null的Entry。假如我们不做任何措施的话，value 永远无法被GC 回收，这个时候就可能会产生内存泄露。
- ThreadLocalMap实现中已经考虑了这种情况，在调用 set()、get()、remove() 方法的时候，会清理掉 key 为 null 的记录**。使用完 ThreadLocal方法后 最好手动调用remove()方法**

## 内部结构

> - 每个线程中都有一个`ThreadLocalMap`数据结构，当执行set方法时，其值是保存在当前线程的`threadLocals`变量中，当执行set方法中，是从当前线程的`threadLocals`变量获取。
>
> - 所以在线程1中set的值，对线程2来说是摸不到的，而且在线程2中重新set的话，也不会影响到线程1中的值，保证了线程之间不会相互干扰。

## hash冲突

> 没有链表结构，那发生hash冲突了怎么办？
>
> 1. 每个ThreadLocal对象都有一个hash值`threadLocalHashCode`，每初始化一个ThreadLocal对象，hash值就增加一个固定的大小`0x61c88647`。
> 2. 在插入过程中，根据ThreadLocal对象的hash值，定位到table中的位置i，过程如下： 1、如果当前位置是空的，那么正好，就初始化一个Entry对象放在位置i上； 2、不巧，位置i已经有Entry对象了，如果这个Entry对象的key正好是即将设置的key，那么重新设置Entry中的value； 3、很不巧，位置i的Entry对象，和即将设置的key没关系，那么只能找下一个空位置；
> 3. 这样的话，在get的时候，也会根据ThreadLocal对象的hash值，定位到table中的位置，然后判断该位置Entry对象中的key是否和get的key一致，如果不一致，就判断下一个位置
> 4. 可以发现，set和get如果冲突严重的话，效率很低，因为ThreadLoalMap是Thread的一个属性，所以即使在自己的代码中控制了设置的元素个数，但还是不能控制其它代码的行为。