---
title: Java监控系统——使用oshi获取主机信息
tags: [Java, oshi, apm]
date: 2019-01-03 13:16:20
categories: 开源之路
---

>最近在筹划做一个监控系统。其中就要获取主机信息，其中遇到一些问题。在此做个记录，以便以后查阅。

<!-- more -->

在该监控系统中，想要做到主机的CPU、内存、磁盘、网络、线程、JVM内存、JVM GC 等维度的监控，JVM方面的监控还好说，直接用JMX做就行了，关键是主机信息不好做监控，JDK没有直接的方案获取系统主机信息。在刚开始想到了以下几种方式获取主机维度的信息。

## 一、命令行

刚开始想的是用最常用的命令行获取主机维度的信息，具体做法是针对不同的系统，写不同的命令。比如说Linux系统，使用TOP命令就能获取到CPU、内存等方面的信息。

```shell
top -b -n 1
```
执行该TOP命令就能获取到CPU、内存等维度的信息

```shell
Tasks: 515 total,   1 running, 511 sleeping,   3 stopped,   0 zombie 
Cpu(s): 14.5%us,  5.3%sy,  0.0%ni, 77.0%id,  0.8%wa,  0.0%hi,  2.4%si,  0.0%st \n");
Mem:  65779104k total, 62037048k used,  3742056k free,   425804k buffers \n");
Swap:        0k total,        0k used,        0k free, 21948492k cached \n");
```

这种方式虽然好，但是我懒啊，不像做系统兼容，于是就开始寻找现有的开源组件。然后找到了一个sigar的东西，貌似这个在业界内应用还挺广的。

## 二、sigar

sigar的api用起来也挺方便的

```java
// CPU数量（单位：个） 
int cpuLength = sigar.getCpuInfoList().length; 
print(cpuLength); 
   
// CPU的总量（单位：HZ）及CPU的相关信息 
CpuInfo infos[] = sigar.getCpuInfoList(); 
for (int i = 0; i < infos.length; i++) {// 不管是单块CPU还是多CPU都适用 
    CpuInfo info = infos[i]; 
    print("mhz=" + info.getMhz());// CPU的总量MHz 
    print("vendor=" + info.getVendor());// 获得CPU的卖主，如：Intel 
    print("model=" + info.getModel());// 获得CPU的类别，如：Celeron 
    print("cache size=" + info.getCacheSize());// 缓冲存储器数量 
} 
```
但是，sigar需要根据不同的系统下载不同的库文件。


>sigar 库文件
windows平台:sigar-x86-winnt.dll
linux平台:libsigar-x86-linux.so或
solaris平台: libsigar-x86-solaris.so或libsigar-sparc-solaris.so或libsigar-sparc64-solaris.so
64位平台:分为至强的libsigar-ia64-linux.so和AMD的libsigar-amd64-linux.so,sigar-amd64-winnt.dll

这样就让我很为难了，于是继续在网上寻找。终于找到了oshi。

## 三、oshi

看看oshi是怎么获取系统信息的吧

* 首先，引入jar包
```xml
<dependency>
    <groupId>com.github.oshi</groupId>
    <artifactId>oshi-core</artifactId>
    <version>${oshi.version}</version>
</dependency>
```

* 然后，开撸

```java
// 获取主机信息
SystemInfo systemInfo = new SystemInfo();
// 获取操作系统信息
OperatingSystem operatingSystem = systemInfo.getOperatingSystem();
operatingSystem.getNetworkParams().getHostName();
operatingSystem.getFamily();
operatingSystem.getVersion().getVersion();
operatingSystem.getVersion().getBuildNumber();
operatingSystem.getBitness();
operatingSystem.getProcessCount();
operatingSystem.getThreadCount();
```

就是这么简单，不需要不同系统引入不同的库文件，也不用自己做系统兼容。oshi自己做了兼容，目前大概兼容些这些系统： Linux, Windows, Solaris, AIX, HP-UX, FreeBSD and Mac OSX.

## 四、对比

以上几种方式，大概可以做个这样的总结：

* 命令行：灵活度高，兼容性强，但是如果要兼容多个平台的话，自己做还是有些麻烦。

* sigar：社区活跃度高，参考文档全面，麻烦在于需要不同平台引入不同的库文件。

* oshi：拿来主义，使用方便，缺点在于文档少。不过个人认为看看github上的文档就够了，使用过程也没有什么坑。

综上，因为我个人在开发的时候选择的是oshi。
