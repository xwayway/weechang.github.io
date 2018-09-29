---
title: 【Java学习之从头开始】Java集合-0——HashMap类
date: 2018-07-30 17:05:03
categories: Java集合
tags: [Java集合, HashMap]
---

# 一、原理实现
hashMap的本质是一个数组，数组中每一个元素称为一个Node，Node中存放的是hash值与链表（或红黑树）的键值对。
hashMap的存储过程如下:
根据 key 计算出它的哈希值 h。
假设Node的个数为 n，那么这个键值对应该放在第 (h % n) 个Node中。
<!-- more -->
如果该Node中已经有了键值对，就使用<a href="http://www.cnblogs.com/lizhanwu/p/4303410.html" target="_blank" rel="noopener noreferrer">拉链法</a>解决冲突。
hashMap还有一个重要的属性: 负载因子(load factor)，它用来衡量hashMap的 空/满 程度，一定程度上也可以体现查询的效率，计算公式为:

负载因子 = 总键值对数 / Node个数

负载因子越大，意味着哈希表越满，越容易导致冲突，性能也就越低。因此，一般来说，当负载因子大于某个常数(可能是 1，或者 0.75 等)时，hashMap将自动扩容。
hashMap在自动扩容时，一般会创建两倍于原来个数的Node，因此即使 key 的hashMap不变，对箱子个数取余的结果也会发生改变，因此所有键值对的存放位置都有可能发生改变，这个过程也称为重哈希(rehash)。
hashMap的扩容并不总是能够有效解决负载因子过大的问题。假设所有 key 的哈希值都一样，那么即使扩容以后他们的位置也不会变化。虽然负载因子会降低，但实际存储在每个Node中的链表长度并不发生改变，因此也就不能提高hashMap的查询性能。

基于以上总结，细心的读者可能会发现hashMap的两个问题:

如果hashMap中本来Node就比较多，扩容时需要重新哈希并移动数据，性能影响较大。
如果哈希函数设计不合理，hashMap在极端情况下会变成线性表，性能极低。
于是针对这种问题，java8中采用了红黑树与链表相互转换的方式来解决。当Node中的键值对大于8的时候，链表结构将变为树形结构。该方式能够在一定程度上解决链表的查询效率问题，但是最好的解决方案还是在hash函数的设计上，尽量减少hash冲突。

根据概率论分析，理想状态下，hashMap中的Node满足泊松分布
当负载因子为 0.75 时，上述公式中 λ 约等于 0.5，因此箱子中元素个数和概率的关系如下:

| 数量 | 概率 |
| ------ | ------ |
| 0 | 0.60653066 |
| 1	| 0.30326533 |
| 2	| 0.07581633 |
| 3	| 0.01263606 |
| 4	| 0.00157952 |
| 5	| 0.00015795 |
| 6	| 0.00001316 |
| 7	| 0.00000094 |
| 8	| 0.00000006 |

# 二、简介
## 2.1 成员变量

```
    /**
     * 默认初始容量 - 必须是2的幂。
     * 个数不能太多或太少。如果太少，很容易触发扩容，如果太多，遍历哈希表会比较慢。
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
    
    /**
     *
     * 最大容量。
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;
    
    /**
     * 默认的负载因子。因此初始情况下，当键值对的数量大于 16 * 0.75 = 12 时，就会触发扩容。
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    
    /**
     * 使用树而不是链表的计数阈值。
     *如果哈希函数不合理，即使扩容也无法减少箱子中链表的长度，因此 Java 的处理方案是当链表太长时，转换成红黑树。
     * 这个值表示当某个链表长度大于 8 时，有可能会转化成树。
     * 备注：从JDK1.8开始，HashMap容量在大于一定值后，底层将会由链表结构转为树形结构
     */
    static final int TREEIFY_THRESHOLD = 8;
    
    /**
     * 当容量小于该值时，将由树状结构转为链表结构
     */
    static final int UNTREEIFY_THRESHOLD = 6;
    
    /**
     * 在转变成树之前，还会有一次判断，只有键值对数量大于 64 才会发生转换。
     * 这是为了避免在哈希表建立初期，多个键值对恰好被放入了同一个链表中而导致不必要的转化。
     */
    static final int MIN_TREEIFY_CAPACITY = 64;
    
    // 数组，该数组用于存储不同的hash值与链表的对应关系
    transient Node<K,V>[] table;
    
    // 单向链表，该链表用于处理哈希冲突
    Set<Map.Entry<K,V>> entrySet;
    
    // hashMap长度
    int size;
    
    // 用来实现fail-fast机制的。 
    int modCount; 
    
    // HashMap的阈值，用于判断是否需要调整HashMap的容量。threshold的值="容量*加载因子"，当HashMap中存储数据的数量达到threshold时，就需要将HashMap的容量加倍。 
    int threshold; 
    
    // 加载因子 
    final float loadFactor;
```

# 三、常用方法
## 3.1 构造方法

```
    /**
    * @param initialCapacity 初始大小
    * @param loadFactor 扩容因子
    */
    public HashMap(int initialCapacity, float loadFactor)
    
    /**
    *@param initialCapacity 初始大小
    */
    public HashMap(int initialCapacity)
    
    public HashMap()
    
    /**
    *@param m Map子类
    */
    public HashMap(Map < ? extends K, ? extends V> m)
```

## 3.2 对外方法

```
    /**
    * 清空hashMap
    * 通过将所有Node置空实现
    */
    public void clear()
    
    /**
    * 是否存在键为key的键值对
    */
    public boolean containsKey(Object key)
    
    /**
    * 是否存在值为value的键值对
    */
    public boolean containsValue(Object value)
    
    /**
    * 获取所有的键值对
    */
    public Set<Map.Entry<K,V>> entrySet()
    
    /**
    * 获取所有键值对 值的集合
    */
    public Collection<V> values()
    
    /**
    * 获取所有键值对 键的集合
    */
    public Set<K> keySet()
    
    /**
    * 根据键值对的键获取值
    */
    public V get(Object key)
    
    /**
    * put 一个键值对
    */
    public V put(K key, V value)
    
    /**
    *将"m"的全部元素都添加到HashMap中
    */
    public void putAll(Map<? extends K, ? extends V> m)

    /**
    *移除键为key的元素
    */
    public V remove(Object key)
```

# 四、HashMap的遍历
## 4.1 遍历HashMap的键值对

第一步：根据entrySet()获取HashMap的“键值对”的Set集合。
第二步：通过Iterator迭代器遍历“第一步”得到的集合。

```
    // 假设map是HashMap对象
    // map中的key是String类型，value是Integer类型
    Integer integ = null;
    Iterator iter = map.entrySet().iterator();
    while(iter.hasNext()) {
        Map.Entry entry = (Map.Entry)iter.next();
        // 获取key
        key = (String)entry.getKey();
        // 获取value
        integ = (Integer)entry.getValue();
    }
```

## 4.2 遍历HashMap的键

第一步：根据keySet()获取HashMap的“键”的Set集合。
第二步：通过Iterator迭代器遍历“第一步”得到的集合。

```
    // 假设map是HashMap对象
    // map中的key是String类型，value是Integer类型
    String key = null;
    Integer integ = null;
    Iterator iter = map.keySet().iterator();
    while (iter.hasNext()) {
            // 获取key
        key = (String)iter.next();
            // 根据key，获取value
        integ = (Integer)map.get(key);
    }
```

## 4.3 遍历HashMap的值

第一步：根据value()获取HashMap的“值”的集合。
第二步：通过Iterator迭代器遍历“第一步”得到的集合。

```
    // 假设map是HashMap对象
    // map中的key是String类型，value是Integer类型
    Integer value = null;
    Collection c = map.values();
    Iterator iter= c.iterator();
    while (iter.hasNext()) {
        value = (Integer)iter.next();
    }
```

<b>PS:本文章相关代码均已上传至 GitHub <a href="https://github.com/weechang/java-zero" target="_blank">查看详情</a></b>