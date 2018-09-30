---
title: 【Java学习之从头开始】Java多线程-1——Java中如何实现多线程
date: 2018-10-01 00:08:39
categories: Java多线程
tags: [Java多线程, Java]
---

> 上一节，学习了Java多线程的基本概念，接下来就要正式进入多线程的学了。首先学习的是，在Java中如何实现一个多线程。

<!-- more -->

# 简介

Java中主要通过 **Thread** 和 **Runnable** 两种方式来实现多线程。另外通过JUC中的线程池也能实现线程，该方法暂时不在我的学习范围之内。

# 关于 Thread 和 Runnable

* Runnable 是一个接口。其中只定义了一个 run() 方法。

```
@FunctionalInterface
public interface Runnable {

    public abstract void run();
}
```



* Thread 是一个Java类。是通过实现Runnable而成。

```
public class Thread implements Runnable {}
```

我们可以通过实现Runnable接口来实现一个多线程。利用Thread方法，可以直接实现多线程。下面我们将通过具体的例子说明如何实现多线程。


# 通过 Thread 实现多线程

```
public class ImplementThread {

    public static void main(String[] args) {
        ImplementThread implementThread = new ImplementThread();
        implementThread.byThread();
    }

    /**
     * 继承 Thread 方式
     */
    public void byThread(){
        MyThread t1 = new MyThread();
        MyThread t2 = new MyThread();
        t1.start();
        t2.start();
    }
}


class MyThread extends Thread {
    private int count = 10;

    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            if (this.count > 0) {
                System.out.println(this.getName() + " 计数器：count" + this.count--);
            }
        }
    }

}
```

运行结果

```
Thread-1 计数器：count10
Thread-0 计数器：count10
Thread-1 计数器：count9
Thread-0 计数器：count9
Thread-1 计数器：count8
Thread-0 计数器：count8
Thread-1 计数器：count7
Thread-0 计数器：count7
Thread-1 计数器：count6
Thread-0 计数器：count6
Thread-1 计数器：count5
Thread-0 计数器：count5
Thread-1 计数器：count4
Thread-0 计数器：count4
Thread-1 计数器：count3
Thread-0 计数器：count3
Thread-0 计数器：count2
Thread-0 计数器：count1
Thread-1 计数器：count2
Thread-1 计数器：count1
```

通Thread继承方式，我们发现。MyThread 继承自Thread 是一个自定义线程。在主线程main()中，创建并启动了2个子线程，这两个子线程分别计数10次。

# 通过 Runnable 实现多线程

```
public class ImplementThread {

    public static void main(String[] args) {
        ImplementThread implementThread = new ImplementThread();
        implementThread.byRunnable();
    }

    /**
     * 实现 Runnable方式
     */
    public void byRunnable(){
        MyRunnable myRunnable = new MyRunnable();

        Thread t1 = new Thread(myRunnable);
        Thread t2 = new Thread(myRunnable);

        t1.start();
        t2.start();
    }
}

class MyRunnable implements Runnable{

    private int count = 10;

    public void run() {
        for (int i = 0; i < 20; i++) {
            if (this.count > 0) {
                System.out.println(Thread.currentThread().getName() + " 计数器：count" + this.count--);
            }
        }
    }

}
```

运行结果

```
Thread-0 计数器：count10
Thread-1 计数器：count9
Thread-0 计数器：count8
Thread-0 计数器：count6
Thread-0 计数器：count5
Thread-1 计数器：count7
Thread-0 计数器：count4
Thread-0 计数器：count2
Thread-1 计数器：count3
Thread-0 计数器：count1
```

通过Runnable方式我们可以发现，主线程main()中虽然启动了2个线程。但是这2个线程一共计数10次。说明这两个线程是基于MyRunnable共享的。

# Thread 与 Runnable的异同

**相同点**：都是一种多线程的实现方式。
**不同点**：
1) Thread 是 类， Runnable 是接口；
2) “一个类只能有一个父类，但是却能实现多个接口”，因此Runnable具有更好的扩展性；
3) Runnable还可以用于“资源的共享”。即，多个线程都是基于某一个Runnable对象建立的，它们会共享Runnable对象上的资源。

**建议使用Runnable去实现多线程**

# 关于Thread的补充

在很多博客上看到说，Runnable 相较于 Tread 能够实现资源共享，没有一个好的解释。就我上面的例子而言。可以看出Thread 与 Runnable 的写法不同。有些人肯定会心存疑虑。
所以在此，我对 Thread 的 写法做另一种补充以证清白。

```
public class ImplementThread {

    public static void main(String[] args) {
        ImplementThread implementThread = new ImplementThread();
        implementThread.byThread2();
    }

    /**
     * Thread 资源不共享的补充说明
     */
    public void byThread2(){
        MyThread myThread = new MyThread();

        Thread t1 = new Thread(myThread);
        Thread t2 = new Thread(myThread);

        t1.start();
        t2.start();
    }

}


class MyThread extends Thread {
    private int count = 10;

    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            if (this.count > 0) {
                System.out.println(this.getName() + " 计数器：count" + this.count--);
            }
        }
    }

}
```

通过和 Runnable一样的启动方式启动Thread 多线程。

运行结果

```
Thread-0 计数器：count10
Thread-0 计数器：count9
Thread-0 计数器：count8
Thread-0 计数器：count6
Thread-0 计数器：count5
Thread-0 计数器：count4
Thread-0 计数器：count7
Thread-0 计数器：count3
Thread-0 计数器：count2
Thread-0 计数器：count1
```

从上述结果可以看到，在main()主线程中，只有一个线程的运行结果。这是为什么呢，因为Thread 一旦被start() 就没法再次start()。若想同时启动多个 Thread 只能创建多个不同的Thread实例。但是多个不同Thread 实例中，资源是不共享的。

**本博客代码已托管到[github](https://github.com/weechang/java-zero/tree/master/p2-java-thread/src/main/java/io/github/weechang/java/thread/implementThread)**
