---
layout: post
title:  "JDK7中hashMap源码解析"
date:   2020-01-04 16:40:39 +0800
categories: java
tag: hashMap
---

* content
{:toc}

# JDK7中hashMap源码解析 #

# 1、hashMap数据结构

hashMap底层数据结构是数组 + 单链表，对 key 计算 hashCode 散列到数组中， 相同的 hashCode 的 key 添加到同一个链表中。

# 2、HashMap 对象的属性

```java

/**
 * The default initial capacity - MUST be a power of two.
 */
// 默认初始容量-必须为2的幂。
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

/**
 * The maximum capacity, used if a higher value is implicitly specified
 * by either of the constructors with arguments.
 * MUST be a power of two <= 1<<30.
 */
// 最大容量，如果两个构造函数都使用参数隐式指定了更高的值，则使用该容量。必须是两个<= 1 << 30的幂。
static final int MAXIMUM_CAPACITY = 1 << 30;

/**
 * The load factor used when none specified in constructor.
 */
// 在构造函数中未指定时使用的负载系数。
static final float DEFAULT_LOAD_FACTOR = 0.75f;

/**
 * An empty table instance to share when the table is not inflated.
 */
// 当表未膨胀时要共享的空表实例。
static final Entry<?,?>[] EMPTY_TABLE = {};

/**
 * The table, resized as necessary. Length MUST Always be a power of two.
 */
// 该表，根据需要调整大小。长度必须始终为2的幂。
transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;

/**
 * The number of key-value mappings contained in this map.
 */
// 
// 此映射中包含的键-值映射数。
transient int size;

/**
 * The next size value at which to resize (capacity * load factor).
 * @serial
 */
// 下一个要调整大小的大小值（容量负载因子）。
// If table == EMPTY_TABLE then this is the initial capacity at which the
// table will be created when inflated.
int threshold;

/**
 * The load factor for the hash table.
 *
 * @serial
 */
// 哈希表的负载因子。
final float loadFactor;

/**
 * The number of times this HashMap has been structurally modified
 * Structural modifications are those that change the number of mappings in
 * the HashMap or otherwise modify its internal structure (e.g.,
 * rehash).  This field is used to make iterators on Collection-views of
 * the HashMap fail-fast.  (See ConcurrentModificationException).
 */
// 对该HashMap进行结构修改的次数结构修改是指更改HashMap中的映射次数或以其他方式修改其内部结构（例如重新哈希）的修改。此字段用于使HashMap的Collection-view上的迭代器快速失败。 （请参见ConcurrentModificationException）。
transient int modCount;

/**
 * The default threshold of map capacity above which alternative hashing is
 * used for String keys. Alternative hashing reduces the incidence of
 * collisions due to weak hash code calculation for String keys.
 * <p/>
 * This value may be overridden by defining the system property
 * {@code jdk.map.althashing.threshold}. A property value of {@code 1}
 * forces alternative hashing to be used at all times whereas
 * {@code -1} value ensures that alternative hashing is never used.
 */
// 映射容量的默认阈值，高于该阈值时，字符串键将使用替代哈希。备用哈希可减少由于字符串键的哈希码计算能力较弱而导致的冲突发生率。 <p>可以通过定义系统属性{@code jdk.map.althashing.threshold}来覆盖此值。属性值{@code 1}强制始终使用替代哈希，而{@code -1}值确保从不使用替代哈希。
static final int ALTERNATIVE_HASHING_THRESHOLD_DEFAULT = Integer.MAX_VALUE;
```

# 3、构造方法

```java

    /**
     * Constructs an empty <tt>HashMap</tt> with the default initial capacity
     * (16) and the default load factor (0.75).
     */
	// 构造一个空的<tt> HashMap <tt>，它具有默认的初始容量（16）和默认的负载系数（0.75）。
    public HashMap() {
        this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
    }

    /**
     * Constructs an empty <tt>HashMap</tt> with the specified initial
     * capacity and the default load factor (0.75).
     *
     * @param  initialCapacity the initial capacity.
     * @throws IllegalArgumentException if the initial capacity is negative.
     */
	//构造一个新的<tt> HashMap <tt>，其映射与指定的<tt> Map <tt>相同。 <tt> HashMap <tt>是使用默认负载因子（0.75）和足以将映射保存在指定的<tt> Map <tt>中的初始容量创建的。 @param m要在其 Map 中放置其映射的 Map @throws NullPointerException如果指定的 Map 为null
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

    /**
     * Constructs an empty <tt>HashMap</tt> with the specified initial
     * capacity and load factor.
     *
     * @param  initialCapacity the initial capacity
     * @param  loadFactor      the load factor
     * @throws IllegalArgumentException if the initial capacity is negative
     *         or the load factor is nonpositive
     */
	//使用指定的初始容量和负载因子构造一个空的<tt> HashMap <tt>。 @param initialCapacity初始容量@param loadFactor负载系数@throws IllegalArgumentException如果初始容量为负或负载系数为非正
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
        threshold = initialCapacity;
        init();
    }


    /**
     * Constructs a new <tt>HashMap</tt> with the same mappings as the
     * specified <tt>Map</tt>.  The <tt>HashMap</tt> is created with
     * default load factor (0.75) and an initial capacity sufficient to
     * hold the mappings in the specified <tt>Map</tt>.
     *
     * @param   m the map whose mappings are to be placed in this map
     * @throws  NullPointerException if the specified map is null
     */
	// 构造一个新的<tt> HashMap <tt>，其映射与指定的<tt> Map <tt>相同。 <tt> HashMap <tt>是使用默认负载因子（0.75）和足以将映射保存在指定的<tt> Map <tt>中的初始容量创建的。 @param m要在其 Map 中放置其映射的 Map @throws NullPointerException如果指定的 Map 为null
    public HashMap(Map<? extends K, ? extends V> m) {
        this(Math.max((int) (m.size() / DEFAULT_LOAD_FACTOR) + 1,
                      DEFAULT_INITIAL_CAPACITY), DEFAULT_LOAD_FACTOR);
        inflateTable(threshold);

        putAllForCreate(m);
    }

```

# 4、put方法

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
	// 将指定值与该映射中的指定键相关联。如果该映射先前包含该键的映射，则将替换旧值。 
	// @param 与指定值关联的键key 
	// @param 与指定键关联的值
	// @return 与<tt> key <tt>或<tt> null <tt>关联的先前值<tt> key <tt>没有映射。 （返回<tt> null <tt>也可以表明该映射先前将<tt> null <tt>与<tt> key <tt>关联。）
    public V put(K key, V value) {
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
        if (key == null)
            return putForNullKey(value);
        int hash = hash(key);
        int i = indexFor(hash, table.length);
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        addEntry(hash, key, value, i);
        return null;
    }


    /**
     * Inflates the table.
     */
    private void inflateTable(int toSize) {
        // 第一次初始化时调用
        // Find a power of 2 >= toSize
        // 找到 >= 2 size的2的幂方数（ 例如：15 则是 16 , 17 则是 32）
        int capacity = roundUpToPowerOf2(toSize);

        // 计算阈值，16 * 0.75 = 12 （>=size的2的幂方数 * 加载因子）
        threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
        table = new Entry[capacity];
        initHashSeedAsNeeded(capacity);
    }


```

