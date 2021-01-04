---
layout: post
title:  "JDK7中hashMap源码解析"
date:   2020-01-04 16:40:39 +0800
categories: java
tag: hashMap
---

* content
{:toc}
# JDK7中hashMap源码解析

# 1、hashMap数据结构

hashMap底层数据结构是数组 + 单链表，对 key 计算 hashCode 散列到数组中， 相同的 hashCode 的 key 添加到同一个链表中。

```java
    //hash标中的结点Node,实现了Map.Entry
    static class Entry<K,V> implements Map.Entry<K,V> {
        final K key;
        V value;
        Entry<K,V> next;
        int hash;

        /**
         * Creates new entry.
         */
        //Entry构造器，需要key的hash，key，value和next指向的结点
        Entry(int h, K k, V v, Entry<K,V> n) {
            value = v;
            next = n;
            key = k;
            hash = h;
        }

        public final K getKey() {
            return key;
        }

        public final V getValue() {
            return value;
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry e = (Map.Entry)o;
            Object k1 = getKey();
            Object k2 = e.getKey();
            if (k1 == k2 || (k1 != null && k1.equals(k2))) {
                Object v1 = getValue();
                Object v2 = e.getValue();
                if (v1 == v2 || (v1 != null && v1.equals(v2)))
                    return true;
            }
            return false;
        }

        //重写Object的hashCode
        public final int hashCode() {
            return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
        }

        public final String toString() {
            return getKey() + "=" + getValue();
        }

        /**
         * This method is invoked whenever the value in an entry is
         * overwritten by an invocation of put(k,v) for a key k that's already
         * in the HashMap.
         */
        void recordAccess(HashMap<K,V> m) {
        }

        /**
         * This method is invoked whenever the entry is
         * removed from the table.
         */
        void recordRemoval(HashMap<K,V> m) {
        }
    }
```



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

## HashMap()

```java
    /**
     * Constructs an empty <tt>HashMap</tt> with the default initial capacity
     * (16) and the default load factor (0.75).
     */
	// 构造一个空的<tt> HashMap <tt>，它具有默认的初始容量（16）和默认的负载系数（0.75）。
    public HashMap() {
        this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
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
	//构造一个新的<tt> HashMap <tt>，其映射与指定的<tt> Map <tt>相同。 <tt> HashMap <tt>是使用默认负载因子（0.75）和足以将映射保存在指定的<tt> Map <tt>中的初始容量创建的。 @param m要在其 Map 中放置其映射的 Map @throws NullPointerException如果指定的 Map 为null
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
        // 第一次调用时，table是空的，进行初始化
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
        if (key == null)
            // 如果 key 值为空，则调用 putForNullKey 的方法
            return putForNullKey(value);
        // 计算 hash 值
        int hash = hash(key);
        // 计算 key 在 Entry 数组数组中的位置，相当于对该数组取余。
        int i = indexFor(hash, table.length);
        // 遍历该位置上的链表
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            // 如果存在相同的 key 值， 则直接覆盖并返回旧值
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        // 没有找到，则添加该 entity
        addEntry(hash, key, value, i);
        return null;
    }



```

## inflateTable(threshold)

```java

    /**
     * Inflates the table.
     */
	// 填充表
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

## putForNullKey(V value)

```java

    /**
     * Offloaded version of put for null keys
     */
	// 空键的put方法
    private V putForNullKey(V value) {
        // 遍历下标是 0 的数组
        for (Entry<K,V> e = table[0]; e != null; e = e.next) {
            // 如果存在键值为 null ， 则覆盖原有的值，并且返回原始值 
            if (e.key == null) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }
        modCount++;
        // 如果不存在，则添加元素节点
        addEntry(0, null, value, 0);
        return null;
    }

```

## hash(Object k)

```java

    /**
     * Retrieve object hash code and applies a supplemental hash function to the
     * result hash, which defends against poor quality hash functions.  This is
     * critical because HashMap uses power-of-two length hash tables, that
     * otherwise encounter collisions for hashCodes that do not differ
     * in lower bits. Note: Null keys always map to hash 0, thus index 0.
     */
	/**
     * 检索对象哈希码，并将补充哈希函数应用于
     * 结果哈希，以防止质量差的哈希函数。 
     * 这很关键，因为HashMap使用2的幂的哈希表，否则
     * 哈希码的冲突在低位没有区别。注意：空键始终映射到哈希0，因此索引为0。
     */
    final int hash(Object k) {
        int h = hashSeed;
        if (0 != h && k instanceof String) {
            return sun.misc.Hashing.stringHash32((String) k);
        }

        h ^= k.hashCode();

        // This function ensures that hashCodes that differ only by
        // constant multiples at each bit position have a bounded
        // number of collisions (approximately 8 at default load factor).
        // 此函数可确保在每个位位置仅相差
        // 恒定倍数的hashCode具有有限的冲突次数（默认负载因子为约8）。
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }

```



## indexFor(int h, int length)

```java
    /**
     * Returns index for hash code h.
     */
	// 返回哈希码h的索引。
    static int indexFor(int h, int length) {
        // hashCode 逻辑与(length-1)   所以要长度必须为2的非零幂
        // assert Integer.bitCount(length) == 1 : "length must be a non-zero power of 2";
        return h & (length-1);
    }
```

## addEntry(int hash, K key, V value, int bucketIndex)

```java

    /**
     * Adds a new entry with the specified key, value and hash code to
     * the specified bucket.  It is the responsibility of this
     * method to resize the table if appropriate.
     *
     * Subclass overrides this to alter the behavior of put method.
     */
	/**
      将具有指定键，值和哈希码的新条目添加到指定存储桶。如果有必要，此方法负责调整表的大小。 
	  子类重写此方法以更改put方法的行为。
	*/
	// 添加元素， bucketIndex 表示数组下标
    void addEntry(int hash, K key, V value, int bucketIndex) {
        // 元素个数大于阈值，并且数组元素不为空
        if ((size >= threshold) && (null != table[bucketIndex])) {
            // 2倍扩容
            resize(2 * table.length);
            hash = (null != key) ? hash(key) : 0;
            // 扩容后重新计算数组的下标值
            bucketIndex = indexFor(hash, table.length);
        }
		
        // 创建元素节点
        createEntry(hash, key, value, bucketIndex);
    }

```

### resize(int newCapacity)

```java

    /**
     * Rehashes the contents of this map into a new array with a
     * larger capacity.  This method is called automatically when the
     * number of keys in this map reaches its threshold.
     *
     * If current capacity is MAXIMUM_CAPACITY, this method does not
     * resize the map, but sets threshold to Integer.MAX_VALUE.
     * This has the effect of preventing future calls.
     *
     * @param newCapacity the new capacity, MUST be a power of two;
     *        must be greater than current capacity unless current
     *        capacity is MAXIMUM_CAPACITY (in which case value
     *        is irrelevant).
     */
	/ ** 
	* 将此映射的内容重新映射为
	* 具有更大容量的新数组。当此映射中的
	* 键数达到其阈值时，将自动调用此方法。 
	* 
    * 如果当前容量为MAXIMUM_CAPACITY，则此方法不会
	* 调整地图大小，而是将阈值设置为Integer.MAX_VALUE。 
	* 这样可以防止将来的通话。 
	* 
    * @param newCapacity新容量，必须是2的幂； 
    * 必须大于当前容量，除非当前
    * 容量为MAXIMUM_CAPACITY（在这种情况下，无关紧要）。
    * /
    // 扩容
    void resize(int newCapacity) {
        Entry[] oldTable = table;
        int oldCapacity = oldTable.length;
        // 数组最大扩容到2的30次方
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }

        Entry[] newTable = new Entry[newCapacity];
        // 把旧数组的所有元素拷贝到新数组里
        transfer(newTable, initHashSeedAsNeeded(newCapacity));
        table = newTable;
        // 扩容后，重新计算阈值
        threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
    }
```

##### transfer(Entry[] newTable, boolean rehash)

```java

    /**
     * Transfers all entries from current table to newTable.
     */
    /**
     * 将所有条目从当前表传输到newTable.
     */
	// 转换
    void transfer(Entry[] newTable, boolean rehash) {
        int newCapacity = newTable.length;
        for (Entry<K,V> e : table) {
            while(null != e) {
                Entry<K,V> next = e.next;
                if (rehash) {
                    e.hash = null == e.key ? 0 : hash(e.key);
                }
                // 重新计算新数组的下标位置
                int i = indexFor(e.hash, newCapacity);
                // 头插法
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            }
        }
    }
```

### createEntry(int hash, K key, V value, int bucketIndex)

```java

    /**
     * Like addEntry except that this version is used when creating entries
     * as part of Map construction or "pseudo-construction" (cloning,
     * deserialization).  This version needn't worry about resizing the table.
     *
     * Subclass overrides this to alter the behavior of HashMap(Map),
     * clone, and readObject.
     */
	/** 
	* 与addEntry相似，只是在创建条目时将使用此版本
	* 作为Map构造或“伪构造”（克隆，
	* 反序列化）的一部分。此版本无需担心调整表的大小。 
	*
    * 子类重写此方法，以更改HashMap（Map），
    * clone和readObject的行为。
    */
	// 创建元素，放在头节点
    void createEntry(int hash, K key, V value, int bucketIndex) {
        Entry<K,V> e = table[bucketIndex];
        table[bucketIndex] = new Entry<>(hash, key, value, e);
        size++;
    }

```

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
        if (key == null)
            return getForNullKey();
        Entry<K,V> entry = getEntry(key);

        return null == entry ? null : entry.getValue();
    }

```

## getForNullKey()

```java

    /**
     * Offloaded version of get() to look up null keys.  Null keys map
     * to index 0.  This null case is split out into separate methods
     * for the sake of performance in the two most commonly used
     * operations (get and put), but incorporated with conditionals in
     * others.
     */
    private V getForNullKey() {
        if (size == 0) {
            return null;
        }
        for (Entry<K,V> e = table[0]; e != null; e = e.next) {
            if (e.key == null)
                return e.value;
        }
        return null;
    }

```

## Entry<K,V> getEntry(Object key)

```java
   /**
     * Returns the entry associated with the specified key in the
     * HashMap.  Returns null if the HashMap contains no mapping
     * for the key.
     */
    final Entry<K,V> getEntry(Object key) {
        if (size == 0) {
            return null;
        }

        int hash = (key == null) ? 0 : hash(key);
        for (Entry<K,V> e = table[indexFor(hash, table.length)];
             e != null;
             e = e.next) {
            Object k;
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k))))
                return e;
        }
        return null;
    }
```

# 6. 常见面试题

## 6.1 HashMap的put方法逻辑

1. 判断entry是否是空数组，如果是初始化一个长度是16，阈值是12的数组
2. 判断key是否等于null，如果是null，就放在下标是0的数组位置上，并插入头结点
3. 对key的hashCode二次hash，并对(length-1)逻辑与，算出数组下标位置
4. 遍历该下标位置上的链表，如果找到该key，就覆盖旧值并返回
5. 判断当前元素个数是否大于阈值，如果大于就执行2倍扩容，把原数组的元素重新hash到新数组中
6. 用当前key创建一个节点，插到下标数组链表的头结点

## 6.2 为什么HashMap的容量必须是2的倍数

```
static int indexFor(int h, int length) {
    return h & (length-1);
}
```

**计算机中，与运算比求余运算更快，采用了 hashCode & (length-1)。** 

>  假如length是16，(length-1)的二进制就是 1111，比15小的数逻辑与之后就是自身，比15大的数，只有低4位的数才能运算出值。是为了更方便与运算。

## 6.3 HashMap线程不安全体现在哪些方面？

由于hash冲突的时候插入链表，采用的是头插法，导致扩容后链表的顺序和原来顺序相反，多个线程同时扩容会出现环形链表，get的时候陷入死循环。

# 7. HashMap还有哪些缺点

1. 使用单链表解决hash冲突，导致最坏的情况get的效率降至O(N)
2. 链表采用头插法，导致多线程扩容出现环形链表
3. 扩容需要把每个元素重新hash放到新数组里，性能太差