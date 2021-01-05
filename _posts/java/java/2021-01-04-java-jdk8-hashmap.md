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
	// hash桶中的结点Node,实现了Map.Entry
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;	// 链表的next指针

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
        TreeNode<K,V> parent;  // 父节点
        TreeNode<K,V> left;	//左子树
        TreeNode<K,V> right;	//右子树
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;	// 颜色属性
        TreeNode(int hash, K key, V val, Node<K,V> next) {
            super(hash, key, val, next);
        }

        /**
         * Returns root of tree containing this node.
         */
        // 返回当前节点的根节点
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
	/**
     * 使用默认的初始容量
     *（16）和默认的加载因子（0.75）构造一个空的<tt> HashMap </ tt>。 
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
	/**
	 * 构造一个空的<tt> HashMap </ tt>，它具有指定的初始*容量和默认负载因子（0.75）。 
	 * 
	 * @param initialCapacity初始容量。 
	 * @如果初始容量为负，则抛出IllegalArgumentException。 
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
	/**
     * 使用指定的初始*容量和负载因子构造一个空的<tt> HashMap </ tt>。 
     * 
     * @param initialCapacity初始容量
     * @param loadFactor负载系数
     * @如果初始容量为负*或负载系数为非正数，则抛出IllegalArgumentException 
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
	/**
     * 使用与指定的<tt> Map </ tt>相同的映射构造一个新的<tt> HashMap </ tt>。 <tt> HashMap </ tt>是使用默认负载因子（0.75）和足以将*映射保存在指定的<tt> Map </ tt>中的初始容量创建的。 
     *
     * @param要在其地图中放置其映射的地图*如果指定的地图为null，则抛出NullPointerException
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
	/**
     * 实现Map.putAll和Map构造函数。映射时的
     * @param最初构造此映射时，返回@false，否则为true（中继到afterNodeInsertion方法）。 
     */
	//该函数将传递的map集合中的所有元素加入本map实例中
    final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
        int s = m.size();
        if (s > 0) {
       		// 如果本map实例的table为null，没有初始化，那么需要初始化
            if (table == null) { // pre-size
                // 实际大小：ft = m.size() / 0.75 + 1;
                float ft = ((float)s / loadFactor) + 1.0F;
                // 判断刚刚计算的大小是否小于最大值1<<<30
                int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                         (int)ft : MAXIMUM_CAPACITY);
                //计算的实际大小ft大于当前的阈值threshhold，那么将threshhold重新计算，tableSizeFor传递的
            	//参数是计算的大小，即重新计算大于ft的最小二次幂
                if (t > threshold)
                    threshold = tableSizeFor(t);
            }
            // 如果table!=null，并且m.size() > threshhold，直接进行扩容处理
            else if (s > threshold)
                resize();
            // 将map中的所有元素加入本map实例中
            for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
                K key = e.getKey();
                V value = e.getValue();
                putVal(hash(key), key, value, false, evict);
            }
        }
    }

```

# 4、put 方法

```java
   /**
     * Associates the specified value with the specified key in this map.
     * If the map previously contained a mapping for the key, the old
     * value is replaced.
     *
     * @param key key with which the specified value is to be associated
     * @param value value to be associated with the specified key
     * @return the previous value associated with <tt>key</tt>, or
     *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
     *         (A <tt>null</tt> return can also indicate that the map
     *         previously associated <tt>null</tt> with <tt>key</tt>.)
     */
	/**
     * 将指定值与该映射中的指定键相关联。
     * 如果该映射先前包含键的映射，则将替换旧
     * 值。
     *
     * @param键与指定值关联的键
     * @param值与指定键关联的值
     * @返回与<tt> key </ tt>关联的先前值，或者
     * <tt> null </ tt>（如果没有<tt> key </ tt>的映射）。
     * （返回<tt> null </ tt>还可以表明该地图
     * 先前将<tt> null </ tt>与<tt> key </ tt>关联。）
     */
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

```

## putVal(int hash, K key, V value, boolean onlyIfAbsent,               boolean evict)


```java
   /**
     * Implements Map.put and related methods.
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
     /**
      * 实现Map.put和相关方法。
      *
      * @param 哈希键的哈希值
      * @param 键的键
      * @param 值的值
      * @param only如果为true，则不更改现有值
      * @param 退出，如果为false，则表处于创建模式。 
      * @返回上一个值；如果没有，则返回null
      */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        //table == null 或者table的长度为0，调用resize方法进行扩容
   		//这里也说明：table 被延迟到插入新数据时再进行初始化
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        
        // 这里就是调用了Hash算法的地方，具体的计算可参考后面写到的例子
    	// 这里定位坐标的做法在上面也已经说到过
        if ((p = tab[i = (n - 1) & hash]) == null)
            // 如果计算得到的桶下标值中的Node为null，就新建一个Node加入该位置(这个新的结点是在
        	// table数组中)。而该位置的hash值就是调用hash()方法计算得到的key的hash值
            tab[i] = newNode(hash, key, value, null);
        //这里表示put的元素用自己key的hash值计算得到的下表和桶中的第一个位置元素产生了冲突，具体就是
    	//(1)key相同，value不同
   	 	//(2)只是通过hash值计算得到的下标相同，但是key和value都不同。这里处理的方法就是链表和红黑树
        else {
            Node<K,V> e; K k;
            //上面已经计算得到了该hash对应的下标i，这里p=tab[i]。这里比较的有：
        	//(1)tab[i].hash是否等于传入的hash。这里的tab[i]就是桶中的第一个元素
        	//(2)比较传入的key和该位置的key是否相同
        	//(3)如果都相同，说明是同一个key，那么直接替换对应的value值(在后面会进行替换)
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                // 将桶中的第一个元素赋给e，用来记录第一个位置的值
                e = p;
            else if (p instanceof TreeNode)
                // 这里判断为红黑树。hash值不相等，key不相等；为红黑树结点
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                //判断为链表结点
                for (int binCount = 0; ; ++binCount) {
                    //如果达到链表的尾部
                    if ((e = p.next) == null) {
                        // 在尾部插入新的结点
                        p.next = newNode(hash, key, value, null);
                        // 前面的binCount是记录链表长度的，如果该值大于8，就会转变为红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    //如果在遍历链表的时候，判断得出要插入的结点的key和链表中间的某个结点的key相
                	//同，就跳出循环,后面也会更新旧的value值
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    //e = p.next。遍历链表所用
                    p = e;
                }
            }
            //判断插入的是否存在HashMap中，上面e被赋值，不为空，则说明存在，更新旧的键值对
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value; // 用传入的参数value更新旧的value值
                afterNodeAccess(e);
                return oldValue;// 返回旧的value值
            }
        }
        // modCount修改
        ++modCount;
        // 容量超出就扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

>​	可以看到主要逻辑在put方法中调用了putVal方法，传递的参数是调用了hash()方法计算key的hash值，主要逻辑在putVal中。可以结合注释熟悉这个方法的执行，我在这里大概总结一下这个方法的执行：
>
>1. 首先 **(tab = table) == null || (n = tab.length) == 0**这一块判断hash桶是否为null，如果为null那么会调用resize方法扩容。后面我们会说到这个方法
>
>2. 定位元素在桶中的位置，具体就是通过**key的hash值和hash桶的长度**计算得到下标i，如果计算到的位置处没有元素(null)，那么就新建结点然后添加到该位置。
>
>3. 如果table[i]处不为null，已经有元素了，那么就表明产生hash冲突,这里可能是三种情况
>
>   ①判断key是不是一样，如果key一样，那么就将新的值替换旧的值；
>
>   ②如果不是因为key一样，那么需要判断当前该桶是不是已经转为了红黑树，是的话就构造一个TreeNode结点插入红黑树；
>
>   ③不是红黑树，就使用链地址法处理冲突问题。这里主要就是遍历链表，如果在遍历过程中也找到了key一样的元素，那么久还是使用新值替换旧值。否则会遍历到链表结尾处，到这里就直接新添加一个Node结点插入链表，插入之后还需要判断是不是已将超过了转换为红黑树的阈值8，如果超过就会转为红黑树。
>
>4. 最后需要修改modCount的值。
>
>5. 判断插入后的size大小是不是超过了threshhold，如果超过需要进行扩容。

## resize()

```java
   /**
     * Initializes or doubles table size.  If null, allocates in
     * accord with initial capacity target held in field threshold.
     * Otherwise, because we are using power-of-two expansion, the
     * elements from each bin must either stay at same index, or move
     * with a power of two offset in the new table.
     *
     * @return the table
     */
	/**
     * 初始化或加倍表大小。如果为null，则根据字段阈值中保持的初始容量目标分配。 
     * 否则，因为我们使用的是2的幂，所以每个bin中的
     * 元素必须保持相同的索引，或者在新表中以2的偏移量移动。 
     * 
     * @返回 table
     */
    final Node<K,V>[] resize() {
        // oldTab 指向旧的 table 数组
        Node<K,V>[] oldTab = table;
        // oldTab 不为 null 的话，oldCap 为原 table 的长度，oldTab为null的话，oldCap为0
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        // 阈值
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            // 如果大于最大容量了，就赋值为整数最大的阀值
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            // 如果数组的长度在扩容后小于最大容量 并且oldCap大于默认值16(这里的newCap也是在原来的
        	// 长度上扩展两倍)
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            // 这里的oldThr=tabSizeFor(initialCapacity),从上面的构造方法看出，如果不是调用的
        	// 无参构造，那么threshhold肯定都会是经过tabSizeFor运算得到的2的整数次幂的,所以可以将
        	// 其作为Node数组的长度(个人理解)
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            // 这里说的是我们调用无参构造函数的时候(table == null,threshhold = 0)，新的容量等于默
        	// 认的容量，并且threshhold也等于默认加载因子*默认初始化容量
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
         // 计算新的resize上限
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;//容量 * 加载因子
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        //以新的容量作为长度，创建一个新的Node数组存放结点元素
    	//当然，桶数组的初始化也是在这里完成的
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        //原来的table不为null
        if (oldTab != null) {
            // 把每个bucket都移动到新的buckets中
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                // 原table中下标j位置不为null
                if ((e = oldTab[j]) != null) {
                    //将原来的table[j]赋为null
                    oldTab[j] = null;
                    //如果该位置没有链表，即只有数组中的那个元素
                    if (e.next == null)
                        // 通过新的容量计算在新的table数组中的下标：(n-1)&hash
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        // 如果是红黑树结点，重新映射时，需要对红黑树进行拆分
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        // 链表优化重hash的代码块
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            //遍历链表，进行重新映射
                            next = e.next;
                            // 原位置
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    // loTail处为null，那么直接加到该位置
                                    loHead = e;
                                else
                                     // loTail为链表尾结点，添加到尾部
                                    loTail.next = e;
                                loTail = e;
                            }
                            // 原位置+旧容量
                            else {
                                if (hiTail == null)
                                    // hiTail处为null，就直接点添加到该位置
                                    hiHead = e;
                                else
                                    // hiTail为链表尾结点，尾插法添加
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        // 将分组后的链表映射到新桶中
                    	// 原索引放到bucket里
                        if (loTail != null) {
                            // 旧链表迁移新链表,链表元素相对位置没有变化; 
                        	// 实际是对对象的内存地址进行操作 
                            loTail.next = null; // 链表尾元素设置为null
                            newTab[j] = loHead; // 数组中位置为j的地方存放链表的head结点
                        }
                        // 原索引+oldCap放到bucket里
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```

> 判断当前oldTab长度是否为空，如果为空，则进行初始化桶数组，也就回答了**无参构造函数初始化为什么没有对容量和阈值进行赋值**，如果不为空，则进行位运算，左移一位，2倍运算扩容。
>
> 扩容，创建一个新容量的数组，遍历旧的数组： 
>
> - 如果节点为空，直接赋值插入
> - 如果节点为红黑树，则需要进行进行拆分操作（个人对红黑树还没有理解，所以先不说明）
> - 如果为链表，根据hash算法进行重新计算下标，将链表进行拆分分组（相信看到这里基本上也知道链表拆分的大致过程了）

# 5、get方法

```java
    /**
     * Returns the value to which the specified key is mapped,
     * or {@code null} if this map contains no mapping for the key.
     *
     * <p>More formally, if this map contains a mapping from a key
     * {@code k} to a value {@code v} such that {@code (key==null ? k==null :
     * key.equals(k))}, then this method returns {@code v}; otherwise
     * it returns {@code null}.  (There can be at most one such mapping.)
     *
     * <p>A return value of {@code null} does not <i>necessarily</i>
     * indicate that the map contains no mapping for the key; it's also
     * possible that the map explicitly maps the key to {@code null}.
     * The {@link #containsKey containsKey} operation may be used to
     * distinguish these two cases.
     *
     * @see #put(Object, Object)
     */
    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }

```

### **getNode**(**int** hash, Object key)

```java
    /**
     * Implements Map.get and related methods.
     *
     * @param hash hash for key
     * @param key the key
     * @return the node, or null if none
     */
	/**
     * 实现Map.get和相关方法。
     *
     * @param hash key的哈希值
     * @param key key
     * @返回节点，如果没有则返回null
     */
    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        //计算存放在数组table中的位置.具体计算方法上面也已经介绍了
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            //先查找是不是就是数组中的元素
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            //该位置为红黑树根节点或者链表头结点
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    //如果first为红黑树结点，就在红黑树中遍历查找
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    //不是树结点，就在链表中遍历查找
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```

