---
title: 【Java学习之从头开始】Java多线程-3——synchronized是如何工作的
date: 2018-10-03 22:15:42
categories: Java多线程
tags: [Java多线程, Java]
---

> 这是我Java多线程的第四篇博文了。本来计划一天一更的，但是国庆假期，都浪去了，没坚持更新。今天主要讲讲在Java多线程中，synchronized是如何工作的。

<!-- more -->

# synchronized 原理

在Java中，每一个对象有且仅有一个同步锁。即同步锁是依赖于对象而存在。

当我们调用某对象的synchronized方法时，就获取了该对象的同步锁。且不同线程对同步锁的访问是互斥的。

# synchronized 基本规则

synchronized 遵循以下三条规则：

* **一**、当一个线程访问“某对象”的“synchronized方法”或者“synchronized代码块”时，其他线程对“该对象”的该“synchronized方法”或者“synchronized代码块”的访问将被阻塞。
* **二**、当一个线程访问“某对象”的“synchronized方法”或者“synchronized代码块”时，其他线程仍然可以访问“该对象”的非同步代码块。
* **三**、当一个线程访问“某对象”的“synchronized方法”或者“synchronized代码块”时，其他线程对“该对象”的其他的“synchronized方法”或者“synchronized代码块”的访问将被阻塞。

# synchronized 的几种使用方法

在Java中，synchronized 关键字是用来控制线程同步的，就是在多线程的环境下，控制 synchronized 代码段不被多个线程同时执行。synchronized 既可以加在一段代码上，也可以加在方法上。下面将仔细了解 synchronized 关键字的几种使用方法。

## synchronized 作用于实例方法

synchronized 修饰实例方法，作用于当前实例加锁，进入同步代码前要获得当前实例的锁。

正确示列：

```
public class InstanceSync implements Runnable {

    static int i = 0;

    public synchronized void syncCount() {
        i++;
    }

    public void run() {
        for (int j = 0; j < 100000; j++) {
            syncCount();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        InstanceSync instance = new InstanceSync();
        Thread t1 = new Thread(instance);
        Thread t2 = new Thread(instance);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(i);
    }
}

```

运行结果:

```
200000
```

错误示例：

```
public class InstanceSyncBad implements Runnable{

    static int i = 0;

    public synchronized void syncCount() {
        i++;
    }

    public void run() {
        for (int j = 0; j < 100000; j++) {
            syncCount();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(new InstanceSyncBad());
        Thread t2 = new Thread(new InstanceSyncBad());
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(i);
    }
}
```

运行结果：

```
175000
```

**代码分析** ：上面两个例子中，第一个获得了我们预期结果，第二个未获得预期结果。说明在第二个例子中 synchronized 方法出现了问题，因为第二个例子中， t1 和 t2 锁住的不是同一个对象。在t1、t2中变量是不共享的。解决这个问题的办法是，让 synchronized 锁住 静态方法。

**实例锁**：锁在某一个实例对象上。如果该类是单例，那么该锁也具有全局锁的概念。实例锁对应的就是synchronized关键字。（即修饰实例方法的锁）

## synchronized 作用于静态方法

synchronized 修饰静态方法，作用于当前类对象加锁，进入同步代码前要获得当前类对象的锁。

```
public class StaticSync implements Runnable {

    static int i=0;

    public static synchronized void increase(){
        i++;
    }

    public void run() {
        for(int j=0;j<100000;j++){
            increase();
        }
    }
    public static void main(String[] args) throws InterruptedException {
        // 新实例
        Thread t1=new Thread(new StaticSync());
        Thread t2=new Thread(new StaticSync());

        // 启动线程
        t1.start();
        t2.start();

        t1.join();
        t2.join();
        System.out.println(i);
    }

}
```

运行结果：

```
200000
```

**代码分析**：该实例中，synchronized 锁住的是静态方法，与锁住实例方法不一样。锁住静态方法，就相当于锁住整个类。

**全局锁**：该锁针对的是类，无论实例多少个对象，那么线程都共享该锁。全局锁对应的就是static synchronized（或者是锁在该类的class或者classloader对象上）（即修饰静态方法的锁）。

## synchronized 作用于代码块

synchronized 修饰代码块，指定加锁对象，对给定对象加锁，进入同步代码库前要获得给定对象的锁。

```
public class BlockSync implements Runnable {

    static BlockSync instance = new BlockSync();
    static int i = 0;

    public void run() {
        //使用同步代码块对变量i进行同步操作,锁对象为instance
        synchronized (instance) {
            for (int j = 0; j < 100000; j++) {
                i++;
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(instance);
        Thread t2 = new Thread(instance);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(i);
    }
}
```

运行结果：

```
200000
```