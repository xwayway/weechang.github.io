---
title: 【Java学习之从头开始】Java集合-1——HashTable类
date: 2018-07-31 14:00:29
categories: Java集合
tags: [Java集合, HashTable]
---

# 一、简介
和HashMap一样，HashTable 也是一个散列表，它存储的内容是键值对(key-value)映射。
HashTable 继承于Dictionary，实现了Map、Cloneable、java.io.Serializable接口。
HashTable 的函数都是同步的，这意味着它是线程安全的。它的key、value都不可以为null。此外，HashTable中的映射不是有序的。
<!-- more -->

## 1.1 成员变量
<pre>
    // 是一个Entry[]数组类型，而Entry实际上就是一个单向链表。哈希表的"key-value键值对"都是存储在Entry数组中的。 
    private transient Entry<?,?>[] table;

    // 是Hashtable的大小，它是Hashtable保存的键值对的数量。 
    private transient int count;

    // 是Hashtable的阈值，用于判断是否需要调整Hashtable的容量。threshold的值="容量*加载因子"。
    private int threshold;
    
    // 就是加载因子。 
    private float loadFactor;

    // 是用来实现fail-fast机制的
    private transient int modCount = 0;
</pre>

## 1.2 构造函数
<pre>
    // 默认构造函数。
    public Hashtable() 
    
    // 指定“容量大小”的构造函数
    public Hashtable(int initialCapacity) 
    
    // 指定“容量大小”和“加载因子”的构造函数
    public Hashtable(int initialCapacity, float loadFactor) 
    
    // 包含“子Map”的构造函数
    public Hashtable(Map<? extends K, ? extends V> t)
</pre>

## 1.3 常用API
<pre>
    // 将此哈希表清空，使其不包含任何键。 
    synchronized void                clear()
    
    synchronized Object              clone()
    
    // 测试此映射表中是否存在与指定值关联的键。 
                 boolean             contains(Object value)
                 
    // 测试指定对象是否为此哈希表中的键。 
    synchronized boolean             containsKey(Object key)
    
    // 如果此 Hashtable 将一个或多个键映射到此值，则返回 true。 
    synchronized boolean             containsValue(Object value)
    
    synchronized Enumeration<V>      elements()
    
    synchronized Set<Entry<K, V>>    entrySet()
    
    synchronized boolean             equals(Object object)
    
    synchronized V                   get(Object key)
    
    synchronized int                 hashCode()
    
    synchronized boolean             isEmpty()
    
    synchronized Set<K>              keySet()
    
    synchronized Enumeration<K>      keys()
    
    synchronized V                   put(K key, V value)
    
    synchronized void                putAll(Map<? extends K, ? extends V> map)
    
    synchronized V                   remove(Object key)
    
    synchronized int                 size()
    
    synchronized String              toString()
    
    synchronized Collection<V>       values()
    
    // 增加此哈希表的容量并在内部对其进行重组，以便更有效地容纳和访问其元素。 
    protected  void rehash() 
</pre>

# 二、HashTable遍历方式

## 2.1 遍历HashTable的键值对
<b>第一步：根据entrySet()获取HashTable的“键值对”的Set集合。</b>
<b>第二步：通过Iterator迭代器遍历“第一步”得到的集合。</b>

<pre>
    // 假设table是Hashtable对象
    // table中的key是String类型，value是Integer类型
    Integer integ = null;
    Iterator iter = table.entrySet().iterator();
    while(iter.hasNext()) {
        Map.Entry entry = (Map.Entry)iter.next();
        // 获取key
        key = (String)entry.getKey();
        // 获取value
        integ = (Integer)entry.getValue();
    }
</pre>
 

## 2.2 通过Iterator遍历HashTable的键
<b>第一步：根据keySet()获取HashTable的“键”的Set集合。</b>
<b>第二步：通过Iterator迭代器遍历“第一步”得到的集合。</b>
<pre>
    // 假设table是Hashtable对象
    // table中的key是String类型，value是Integer类型
    String key = null;
    Integer integ = null;
    Iterator iter = table.keySet().iterator();
    while (iter.hasNext()) {
            // 获取key
        key = (String)iter.next();
            // 根据key，获取value
        integ = (Integer)table.get(key);
    }
</pre>
 
## 2.3 通过Iterator遍历HashTable的值
<b>第一步：根据value()获取HashTable的“值”的集合。</b>
<b>第二步：通过Iterator迭代器遍历“第一步”得到的集合。</b>
<pre>
    // 假设table是HashTable对象
    // table中的key是String类型，value是Integer类型
    Integer value = null;
    Collection c = table.values();
    Iterator iter= c.iterator();
    while (iter.hasNext()) {
        value = (Integer)iter.next();
    }
</pre>
 
## 2.4 通过Enumeration遍历HashTable的键
<b>第一步：根据keys()获取HashTable的集合。</b>
<b>第二步：通过Enumeration遍历“第一步”得到的集合。</b>
<pre>
    Enumeration enu = table.keys();
    while(enu.hasMoreElements()) {
        System.out.println(enu.nextElement());
    } 
</pre>  
 
## 2.5 通过Enumeration遍历HashTable的值
<b>第一步：根据elements()获取HashTable的集合。</b>
<b>第二步：通过Enumeration遍历“第一步”得到的集合。</b>
<pre>
    Enumeration enu = table.elements();
    while(enu.hasMoreElements()) {
        System.out.println(enu.nextElement());
    }
</pre>

<b>PS:本文章相关代码均已上传至 GitHub <a href="https://github.com/weechang/java-zero" target="_blank">查看详情</a></b>