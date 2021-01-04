---
layout: post
title:  "JDK8中hashMap源码解析"
date:   2020-01-04 16:40:39 +0800
categories: java
tag: hashMap
---

* content
{:toc}

# JDK8中hashMap源码解析 #

# 1、hashMap数据结构

因为主要说的是1.8版本中的实现。而1.8中HashMap是数组+链表+红黑树实现的，大概如下图所示。后面还是主要介绍Hash Map中主要的一些成员以及方法原理。



那么上述图示中的结点Node具体类型是什么，源码如下。Node是HashMap的内部类，实现了Map.Entery接口，主要就是存放我们put方法所添加的元素。其中的next就表示这可以构成一个单向链表，这主要是通过链地址法解决发生hash冲突问题。而当桶中的元素个数超过阈值的时候就换转为红黑树。

## Node<K,V>

```java

    /**
     * Basic hash bin node, used for most entries.  (See below for
     * TreeNode subclass, and in LinkedHashMap for its Entry subclass.)
     */
	/**
     * 基本哈希箱节点，用于大多数条目。 （请参阅下面的
     * TreeNode子类，以及LinkedHashMap中的Entry子类。）
     */
	// //hash桶中的结点Node,实现了Map.Entry
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        // 链表的next指针
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        // 重写Object的hashCode
        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }
        
		// equals方法
        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }

```

## **TreeNode**<**k**,**v**>

```java
    /* ------------------------------------------------------------ */
    // Tree bins

    /**
     * Entry for Tree bins. Extends LinkedHashMap.Entry (which in turn
     * extends Node) so can be used as extension of either regular or
     * linked node.
     */
    static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;
        TreeNode(int hash, K key, V val, Node<K,V> next) {
            super(hash, key, val, next);
        }

        /**
         * Returns root of tree containing this node.
         */
        final TreeNode<K,V> root() {
            for (TreeNode<K,V> r = this, p;;) {
                if ((p = r.parent) == null)
                    return r;
                r = p;
            }
        }
    }

```

# 2、HashMap中的成员变量以及含义

```java
//默认初始化容量初始化=16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

//最大容量 = 1 << 30
static final int MAXIMUM_CAPACITY = 1 << 30;

//默认加载因子.一般HashMap的扩容的临界点是当前HashMap的大小 > DEFAULT_LOAD_FACTOR * 
//DEFAULT_INITIAL_CAPACITY = 0.75F * 16
static final float DEFAULT_LOAD_FACTOR = 0.75f;

//当hash桶中的某个bucket上的结点数大于该值的时候，会由链表转换为红黑树
static final int TREEIFY_THRESHOLD = 8;

//当hash桶中的某个bucket上的结点数小于该值的时候，红黑树转变为链表
static final int UNTREEIFY_THRESHOLD = 6;

//桶中结构转化为红黑树对应的table的最小大小
static final int MIN_TREEIFY_CAPACITY = 64;

//hash算法,计算传入的key的hash值，下面会有例子说明这个计算的过程
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
} 

//tableSizeFor(initialCapacity)返回大于initialCapacity的最小的二次幂数值。下面会有例子说明
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}

//hash桶
transient Node<K,V>[] table;

//保存缓存的entrySet
transient Set<Map.Entry<K,V>> entrySet;

//桶的实际元素个数 != table.length
transient int size;

//扩容或者更改了map的计数器。含义：表示这个HashMap结构被修改的次数，结构修改是那些改变HashMap中的映射数量或者
//修改其内部结构（例如，重新散列rehash）的修改。 该字段用于在HashMap失败快速（fast-fail）的Collection-views
//上创建迭代器。
transient int modCount;

//临界值，当实际大小（cap*loadFactor）大于该值的时候，会进行扩充
int threshold;

//加载因子
final float loadFactor;
```

# 3、HashMap构造方法

## HashMap()

```java
    /**
     * Constructs an empty <tt>HashMap</tt> with the default initial capacity
     * (16) and the default load factor (0.75).
     */
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }
```

## HashMap(int initialCapacity)

```java

    /**
     * Constructs an empty <tt>HashMap</tt> with the specified initial
     * capacity and the default load factor (0.75).
     *
     * @param  initialCapacity the initial capacity.
     * @throws IllegalArgumentException if the initial capacity is negative.
     */
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

```

## HashMap(int initialCapacity, float loadFactor)

```java

    /**
     * Constructs an empty <tt>HashMap</tt> with the specified initial
     * capacity and load factor.
     *
     * @param  initialCapacity the initial capacity
     * @param  loadFactor      the load factor
     * @throws IllegalArgumentException if the initial capacity is negative
     *         or the load factor is nonpositive
     */
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }

```



## HashMap(Map<? extends K, ? extends V> m)

```java

    /**
     * Constructs a new <tt>HashMap</tt> with the same mappings as the
     * specified <tt>Map</tt>.  The <tt>HashMap</tt> is created with
     * default load factor (0.75) and an initial capacity sufficient to
     * hold the mappings in the specified <tt>Map</tt>.
     *
     * @param   m the map whose mappings are to be placed in this map
     * @throws  NullPointerException if the specified map is null
     */
    public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }
```

### putMapEntries(Map<? extends K, ? extends V> m, boolean evict)

```java

    /**
     * Implements Map.putAll and Map constructor.
     *
     * @param m the map
     * @param evict false when initially constructing this map, else
     * true (relayed to method afterNodeInsertion).
     */
    final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
        int s = m.size();
        if (s > 0) {
            if (table == null) { // pre-size
                float ft = ((float)s / loadFactor) + 1.0F;
                int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                         (int)ft : MAXIMUM_CAPACITY);
                if (t > threshold)
                    threshold = tableSizeFor(t);
            }
            else if (s > threshold)
                resize();
            for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
                K key = e.getKey();
                V value = e.getValue();
                putVal(hash(key), key, value, false, evict);
            }
        }
    }

```





