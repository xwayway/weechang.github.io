---
title: 【Java学习之从头开始】Java多线程-4——如何实现线程间通信
date: 2018-10-04 14:41:56
categories: Java多线程
tags: [Java多线程, Java]
---

> Java多线程中，当一个线程执行结束后，如何通知主线程当前线程已执行结束？在一个线程中，如何等待一个线程执行结束？这都涉及到线程间通信,本章我们就来谈一谈Java线程间通信的问题。

<!-- more -->

# 线程间通信方法介绍

在Object类中，我们有简单介绍过 wait() 、 notify() 、 notifyAll() 等方法，下面将具体介绍线程间通信所要用到的方法。

* notify() ：唤醒在此对象监视器上等待的单个线程。
* notifyAll() ：唤醒在此对象监视器上等待的所有线程。
* wait() ：让当前线程处于“等待(阻塞)状态”，“直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法”，当前线程被唤醒(进入“就绪状态”)。
* wait(long timeout) ：让当前线程处于“等待(阻塞)状态”，“直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者超过指定的时间量”，当前线程被唤醒(进入“就绪状态”)。
* wait(long timeout, int nanos) ：让当前线程处于“等待(阻塞)状态”，“直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者其他某个线程中断当前线程，或者已超过某个实际时间量”，当前线程被唤醒(进入“就绪状态”)。

# 实现线程等待与唤醒

下面将通过 wait() 与 notify() 实现线程的等待与唤醒。

```
public class ThreadMessage {

    public static void main(String[] args) {
        WaitTest wT = new WaitTest();
        wT.test();
    }
}

class WaitTest {

    public void test() {
        ThreadA tA = new ThreadA("t1");

        synchronized (tA) {
            try {
                // 启动“线程tA”
                System.out.println(Thread.currentThread().getName() + " start tA");
                tA.start();

                // 主线程等待tA通过notify()唤醒。
                System.out.println(Thread.currentThread().getName() + " wait()");
                tA.wait();

                System.out.println(Thread.currentThread().getName() + " continue");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

/**
 * 线程A
 */
class ThreadA extends Thread {

    public ThreadA(String name) {
        super(name);
    }

    public void run() {
        synchronized (this) {
            System.out.println(Thread.currentThread().getName() + " run notify()");
            // 唤醒当前的wait线程
            notify();
        }
    }
}
```

运行结果：

```
main start tA
main wait()
t1 run notify()
main continue
```

结果分析：


