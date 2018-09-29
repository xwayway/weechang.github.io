---
title: 【Java学习之从头开始】Java基础-1——String类
date: 2018-06-08 13:59:02
categories: Java基础
tags: [Java基础, String]
---

# 一、简介

## 1.1 成员变量

```
    public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
        
        private final char value[];
    
        private int hash; // Default to 0
    
        private static final long serialVersionUID = -6849794470754667710L;
    
        private static final ObjectStreamField[] serialPersistentFields = new ObjectStreamField[0];
    }
```


从源码可以看出，<b>String底层是通过一个不可变的字符串常量来进行维护的。所以只要一个字符改变就会生成一个新的String类型对象。</b>
<!-- more -->
## 1.2 构造方法

```
    String()
    String(String original)
    String(char value[])
    String(char value[], int offset, int count)
    String(int[] codePoints, int offset, int count)
    String(byte bytes[], int offset, int length, String charsetName)
    String(byte bytes[], int offset, int length, Charset charset)
    String(byte bytes[], String charsetName)
    String(byte bytes[], Charset charset)
    String(byte bytes[], int offset, int length)
    String(byte bytes[])
    String(StringBuffer buffer)
    String(StringBuilder builder)
    String(char[] value, boolean share)
```


# 二、创建方式的区别
## 2.1 直接赋值方式
String str = "str"; //<b>直接赋值方式创建对象是在方法区的常量池</b>

## 2.2 构造方法创建
String str = "str";//<b>通过构造方法创建字符串对象是在堆内存</b>

```
    public class JavaString {
        /**
         * 创建方式的比较
         */
        public void createMethod() {
            String str1 = "str";
            String str2 = new String("str");
            String str3 = str1;
            String str4 = str2;
            System.out.println(str1 == str2); // false
            System.out.println(str1 == str3); // true
            System.out.println(str2 == str4); // true
            System.out.println(str1 == str4); // false
        }
    
        public static void main(String[] args) {
            JavaString javaString = new JavaString();
            javaString.createMethod();
        }
    }
```

## 2.3 内存分析

{% asset_img memory.png 内存示意 %}
<b>在字符串中，如果采用直接赋值的方式（String str="str"）进行对象的实例化，则会将匿名对象“str”放入字符串常量池，每当下一次对不同的对象进行直接赋值的时候会直接利用池中原有的匿名对象。</b>
这样，所有直接赋值的String对象，如果利用相同的“str”，则String对象==返回true；
PS：字符串还可以采用手工入池的方式
```
    /**
     * 字符串手工入池
     */
    public void intern(){
        String str1 = new String("str").intern();
        String str2 = "str";
        System.out.println(str1 == str2); // true
    }
```

## 2.4 区别
1）直接赋值（String str = "str"）：只开辟一块堆内存空间，并且会自动入池，不会产生垃圾。
2）构造方法（String str=  new String("str");）:会开辟两块堆内存空间（具体原因阐述见此处），其中一块堆内存会变成垃圾被系统回收，而且不能够自动入池，需要通过public  String intern();方法进行手工入池。
<b>PS:String类对象一旦声明则不可以改变；而改变的只是地址，原来的字符串还是存在的，并且产生垃圾。</b>

# 三、字符串常量池
## 3.1 字符串常量池概述

1）常量池表（Constant_Pool table）

Class文件中存储所有常量（包括字符串）的table。
这是Class文件中的内容，还不是运行时的内容，不要理解它是个池子，其实就是Class文件中的字节码指令。

2）运行时常量池（Runtime Constant Pool）　

JVM内存中方法区的一部分，这是运行时的内容
这部分内容（绝大部分）是随着JVM运行时候，从常量池转化而来，每个Class对应一个运行时常量池
上一句中说绝大部分是因为：除了 Class中常量池内容，还可能包括动态生成并加入这里的内容

3）字符串常量池（String Pool）

这部分也在方法区中，但与Runtime Constant Pool不是一个概念，String Pool是JVM实例全局共享的，全局只有一个
JVM规范要求进入这里的String实例叫“被驻留的interned string”，各个JVM可以有不同的实现，HotSpot是设置了一个哈希表StringTable来引用堆中的字符串实例，被引用就是被驻留。

## 3.2 亨元模式
其实字符串常量池这个问题涉及到一个设计模式，叫“享元模式”，顾名思义 - - - > 共享元素模式
也就是说：一个系统中如果有多处用到了相同的一个元素，那么我们应该只存储一份此元素，而让所有地方都引用这一个元素
Java中String部分就是根据享元模式设计的，而那个存储元素的地方就叫做“字符串常量池 - String Pool”

## 3.3 详细分析

举例：
```
    int x  = 10;
    String y = "hello";
```

1)首先，10和"hello"会在经过javac（或者其他编译器）编译过后变为Class文件中constant_pool table的内容
2)当我们的程序运行时，也就是说JVM运行时，每个Classconstant_pool table中的内容会被加载到JVM内存中的方法区中各自Class的Runtime Constant Pool。
3)一个没有被String Pool包含的Runtime Constant Pool中的字符串（这里是"hello"）会被加入到String Pool中（HosSpot使用hashtable引用方式），步骤如下：　　　
    一是：在Java Heap中根据"hello"字面量create一个字符串对象
    二是：将字面量"hello"与字符串对象的引用在hashtable中关联起来，键 - 值 形式是："hello" = 对象的引用地址。
另外来说，当一个新的字符串出现在Runtime Constant Pool中时怎么判断需不需要在Java Heap中创建新对象呢？
策略是这样：会先去根据equals来比较Runtime Constant Pool中的这个字符串是否和String Pool中某一个是相等的（也就是找是否已经存在），如果有那么就不创建，直接使用其引用；反之，如上3。如此，就实现了享元模式，提高的内存利用效率。
举例：
使用String s = new String("hello"); 会创建2个对象。
首先，出现了字面量"hello"，那么去String Pool中查找是否有相同字符串存在，因为程序就这一行代码所以肯定没有，那么就在Java Heap中用字面量"hello"首先创建1个String对象。
接着，new String("hello")，关键字new又在Java Heap中创建了1个对象，然后调用接收String参数的构造器进行了初始化。最终s的引用是这个String对象。

# 四、常用方法

## 4.1 基本操作
```
    // 获取字符串的长度
    int length()
    
    // 返回指定字符在该字符串中第一次出现的位置，若无则返回-1
    int indexOf(int ch)
    
    // 返回指定字符在该字符串中最后一次出现的位置，若无则返回-1
    int lastIndexOf(int ch)
    
    // 返回指定位置的字符，其中index取值范围为（0~字符长度-1）
    char charAt(int index)
```

## 4.2 转换操作

```
    // 转换为字符数组
    char[] toCharArray()
    
    // 将传入值转换为String （常见入参为Integer、Long、Double等）
    String valueOf(xxx)
    
    // 转换为小写
    String toLowerCase()
    
    // 转换为大写
    String toUpperCase()
```

## 4.3 替换操作
```
    // 使用 replacement替换原字符串中所有的target
    String replace(CharSequence target, CharSequence replacement)
    
    // 替换字符串中的所有空格
    String trim()
```

## 4.4 截取操作
```
    // 根据 regex 将原字符串拆分为 数组
    String[] split(String regex)
    
    // 截取 beginIndex 到 endIndex之间的字符串
    String substring(int beginIndex, int endIndex)
```

## 4.5判断操作
```
    // 判断值是否相等
    boolean equals(Object anObject)
    
    // 忽略大小写的判断
    boolean equalsIgnoreCase(String anotherString)
    
    // 是否以prefix 开头
    startsWith(String prefix)
    
    // 是否以prefix 结尾
    endsWith(String prefix)
    
    // 是否包含 s
    boolean contains(CharSequence s)
    
    // 是否是空字符串
    boolean isEmpty()
```

#五、String、StringBuffer、StringBuilder
## 5.1 异同点
同：三者底层都是通过字符数组来进行维护的。

异：

| | String | StringBuffer | StringBuilder|
| ------ | ------ | ------ | ------ |
| 是否可变 | ×| √ | √ | 
| 线程安全| √| √ | × | 

## 5.2 效率比较
<b>通常情况下：StringBuilder > StringBuffer > String</b>

<b>PS:本文章相关代码均已上传至 GitHub <a href="https://github.com/weechang/java-zero" target="_blank">查看详情</a></b>