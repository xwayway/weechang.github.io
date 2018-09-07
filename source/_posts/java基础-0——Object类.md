---
title: 【Java学习之从头开始】Java基础-0——Object类
date: 2018-06-07 18:39:22
categories: Java基础
tags: [Java基础, Object]
---

从这篇文章起就正式开始Java学习之旅了。java中，万物皆对象，所以第一篇先学习Java的Object类，循序渐进。
<!-- more -->
# 一、getClass()

<pre>
public final native Class<?> getClass();
</pre>

返回此Object的运行时类类型。如： class io.github.weechang.java.base.JavaObject。
不可重写，要调用的话，一般和getName()联合使用，如getClass().getName(); // io.github.weechang.java.base.JavaObject


# 二、hashCode()

<pre>
public native int hashCode();
</pre>

返回该对象的哈希值。如：123961122
该方法用于哈希查找，可以减少在查找中使用equals的次数，重写了equals方法一般都要重写hashCode方法。这个方法在一些具有哈希功能的Collection中用到。如HashSet、HashMap。
如果不重写hashcode(),在HashSet中添加两个equals的对象，会将两个对象都加入进去。

# 三、equals()

<pre>
public boolean equals(Object obj) {
    return (this == obj);
}
</pre>

Object中的equals方法是直接判断this和obj是否是同一对象，所谓同一对象就是指内存中同一块存储单元，如果this和obj指向的是同一块内存对象，则返回true,如果this和obj指向的不是同一块内存，则返回false。

<pre>
注意：即便是内容完全相等的两块不同的内存对象，也返回false。
     如果是同一块内存，则object中的equals方法返回true,如果是不同的内存，则返回false
     如果希望不同内存但相同内容的两个对象equals时返回true,则我们需要重写父类的equal方法
     Java中一部分类已经重写了object中的equals方法（这样就是比较内容是否相等了），如基本的包装类型 Integer、Long、Boolean等，还有String。
</pre>


# 四、clone()

<pre>
protected native Object clone() throws CloneNotSupportedException;
</pre>

保护方法，实现对象的浅复制，只有实现了Cloneable接口才可以调用该方法，否则抛出CloneNotSupportedException异常。
Java里除了8种基本类型传参数是值传递，其他的类对象传参数都是引用传递，我们有时候不希望在方法里讲参数改变，这是就需要在类中复写clone方法（实现深复制）。

## 4.1 clone与copy的区别

在通常操作中，我们常采用的赋值方式为 User u1 = new User(); User u2 = u1;此时，User u2 = u1这种赋值方式就称为copy。此时的赋值只是简单地将u1所指向的内存地址赋值给了u2,所以此时u1、u2引用的是同一块内存空间。所以对u1或者u2任意对象的操作如u1.setId(2);都会带来另一对象的值的改变，此时可以发现u2.getId() 返回值也为2。若我们不希望带来这样的改变，就需要用到clone.User u2 = u1.clone();等价于User u2 = new User(); u2.setId(u1.getId());

<b>copy：只是简单地将内存地址进行引用，实则两者是同一对象,任一对象值的改变，会带来另一对象值的改变。</b>
<b>clone: 两者是不同内存地址的对象，两者值只有在初始clone时一致，一个对象值的改变，不会带来另一对象值的改变。</b>

## 4.2 Shallow Clone与Deep Clone
Object在对某个对象实施仅仅是简单地执行域对域的copy，这就是Shallow Clone。若User中存在一属性为 Date birthDay，此时u1与U2的引诱关系图如下：
{% asset_img clone.png Shallow Clone %}
此时，除了基本数据类型是值的clone外，非基本数据类型都只是引用的clone。所以此时clone对象与原对象存在一部分相同的引用。此时任一对象的非基本类型的值改变，会导致另一对象的值的改变。若要解决此问题就需要Deep Clone。
针对User 对象可以对clone方法进行改造，如下：
<pre>
@Override
protected Object clone() throws CloneNotSupportedException {
    User cloned = (User) super.clone();
    cloned.birthDay = (Date) birthDay.clone();
    return cloned;
}
</pre>

# 五、toString()

<pre>
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
</pre>

返回值为 字符串，该字符串由类名（对象是该类的一个实例）、at 标记符“@”和此对象哈希码的无符号十六进制表示组成。如 ：io.github.weechang.java.base.obj.User@762efe5d

# 六、notify()

<pre>
public final native void notify();
</pre>

该方法唤醒在该对象上等待的某个线程

# 七、notifyAll()

<pre>
public final native void notifyAll();
</pre>

该方法唤醒在该对象上等待的所有线程。

# 八、wait()

<pre>
public final void wait() throws InterruptedException {
    wait(0);
}
</pre>

<pre>
public final native void wait(long timeout) throws InterruptedException;
</pre>

<pre>
/**
*@param timeout  等待该线程终止的时间最长为timeout毫秒。 
*@param nanos 等待时间为 timeout毫秒+nanos纳秒
*/
public final void wait(long timeout, int nanos) throws InterruptedException {
      if (timeout < 0) {
          throw new IllegalArgumentException("timeout value is negative");
      }
  
      if (nanos < 0 || nanos > 999999) {
          throw new IllegalArgumentException("nanosecond timeout value out of range");
      }
 
     if (nanos > 0) {
         timeout++;
     }
 
     wait(timeout);
 }
</pre>

wait方法就是使当前线程等待该对象的锁，当前线程必须是该对象的拥有者，也就是具有该对象的锁。wait()方法一直等待，直到获得锁或者被中断。wait(long timeout)设定一个超时间隔，

如果在规定时间内没有获得锁就返回。

# 九、finalize()

<pre>
 protected void finalize() throws Throwable { }
</pre>
该方法用于释放资源。因为无法确定该方法什么时候被调用，很少使用。
Java允许在类中定义一个名为finalize()的方法。它的工作原理是：一旦垃圾回收器准备好释放对象占用的存储空间，将首先调用其finalize()方法。并且在下一次垃圾回收动作发生时，才会真正回收对象占用的内存。

<pre>
关于垃圾回收，有三点需要记住：
　　1、对象可能不被垃圾回收。只要程序没有濒临存储空间用完的那一刻，对象占用的空间就总也得不到释放。
　　2、垃圾回收并不等于“析构”。
　　3、垃圾回收只与内存有关。使用垃圾回收的唯一原因是为了回收程序不再使用的内存。
</pre>

<pre>
finalize()的用途：
　　无论对象是如何创建的，垃圾回收器都会负责释放对象占据的所有内存。这就将对finalize()的需求限制到一种特殊情况，即通过某种创建对象方式以外的方式为对象分配了存储空间。
　　不过这种情况一般发生在使用“本地方法”的情况下，本地方法是一种在Java中调用非Java代码的方式。
</pre>

<b>PS:本文章相关代码均已上传至 GitHub <a href="https://github.com/weechang/java-zero" target="_blank">查看详情</a></b>