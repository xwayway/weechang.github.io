---
title: 【Java学习之从头开始】Java多线程-1——Java中如何实现多线程
date: 2018-09-30 17:39:39
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

我们可以通过实现Runnable接口来实现一个多线程。
利用Thread方法，可以直接实现多线程。

Thread 和 Runnable比较：

# 通过 Thread 实现多线程

# 通过 Runnable 实现多线程

