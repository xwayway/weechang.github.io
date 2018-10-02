---
title: 【Java学习之从头开始】Java多线程-2——Thread中run()和start()的区别
date: 2018-10-02 18:12:03
categories: Java多线程
tags: [Java多线程, Java]
---

>前一篇博文中，我们讨论了Java中多线程的常用实现方式及Thread和Runnable的区别。在本篇，我们将讨论Thread中run()和start()的区别。

<!-- more -->

# 实现

下面我们将通过代码来实现，两者的调用。并展示两者区别

```
public class RunAndStart {

    public static void main(String[] args) {
        MyThread myThread = new MyThread("test");

        System.out.println("*** myThread run method");
        myThread.run();

        System.out.println("*** myThread start method");
        myThread.start();
    }

}

class MyThread extends Thread{

    public MyThread(String name){
        super(name);
    }

    @Override
    public void run(){
        System.out.println("my name is " + Thread.currentThread().getName());
    }
}
```

其运行结果如下：

```
*** myThread run method
my name is main
*** myThread start method
my name is test
```

对以上运行结果呢，我做一下解释：
* Thread.currentThread().getName()是用于获取“当前线程”的名称。
* myThread.run()是在“主线程main()”中调用的，该run()方法直接运行在“主线程main()”上。
* myThread.start()会启动“线程myThread”，“线程myThread”启动之后，会调用run()方法。

# 区别

从以上代码的运行结果我们可以看到run() 和 start() 的区别：

* start() : 它的作用是启动一个新线程，新线程会执行相应的run()方法。start()不能被重复调用（该处我没给出具体代码，感兴趣的同学可以自行验证）。
* run()   : run()就和普通的成员方法一样，直接调用的话会在当前线程中执行run()，而并不会启动新线程。且能多次重复调用。

# 源码实现

* start()

```
public synchronized void start() {
    // 检查线程启动状态
    // 如果不是就绪状态，则抛出异常
    if (threadStatus != 0)
        throw new IllegalThreadStateException();

    // 将当前线程，加入group中
    group.add(this);

    boolean started = false;
    try {
        // 通过本地方法 start0 启动线程
        start0();
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {

        }
    }
}

private native void start0();
```

源码解释： 在调用start方法的时候，其实是通过本地方法start0() 来启动的。通过start0() 创建一个新的线程，且调用run()执行。

* run()

```
public void run() {
    if (target != null) {
        target.run()
    }
}
```

源码解析: 直接调用目标对象的 run() 方法，并不会创建新线程。

**PS：此章节内容相关源码已上传至[GitHub](https://github.com/weechang/java-zero/tree/master/p2-java-thread/src/main/java/io/github/weechang/java/thread/runAndStart)**