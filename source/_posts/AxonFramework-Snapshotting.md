---
title: AxonFramework-Snapshotting
date: 2018-08-31 11:07:08
categories: AxonFramework
tags: [AxonFramework]
author: 勇赴
---

当聚合存活很长一段时间，它们的状态不断变化，它们会生成大量的事件。不得不加载所有这些事件去复原一个聚合的状态，可能会有很大的性能影响。快照事件是一个有着特殊用途的领域事件：它将任意数量的事件归纳为单个事件。通过定期创建和存储快照事件，事件存储不必返回长的事件列表。只返回最后一个快照事件和在快照之后发生的所有事件

<!-- more -->

例如，库存物品往往会经常变化。每销售一件物品，事件就减少一件库存。每次一批新物品进来，库存就增加一些。如果你每天销售一百件，你每天会产生至少100个事件。几天之后，你的系统将会花太多的时间读取所有这些事件，只是为了弄清楚它是否应该raise一个“ItemOutOfStockEvent”。单个快照事件仅仅通过存储当前的库存数量就可以取代很多这些事件。

## Creating a snapshot创建一个快照
快照的创建可由多种因素触发，例如，从上次快照创建以来的事件的数量，初始化一个聚合的时候超过了某个阈值，基于时间的，等等。目前，Axon提供了一种机制，允许你基于事件计数阈值触发快照。

当要创建快照时的定义，由SnapshotTriggerDefinition接口提供。

当加载聚合所需的事件数量超过一定的阈值时，EventCountSnapshotTriggerDefinition提供触发快照创建的机制。如果加载一个聚合需要的事件的数量超过某个可配置的阈值，触发器告诉Snapshotter为聚合创建一个快照。

快照触发器在一个事件溯源存储库上配置，并有很多属性允许你调整触发:

* 快照设置实际的快照实例，负责创建和存储实际的快照事件;
* 触发器设置触发快照创建的阈值;

Snapshotter负责快照的实际创建。通常，快照是一个应该尽可能少的扰乱操作进程的进程。因此,建议在不同的线程运行Snapshotter。Snapshotter接口声明了单独的方法：scheduleSnapshot()，以聚合的类型和标识符作为参数。

Axon提供了AggregateSnapshotter，它创建并存储AggregateSnapshot实例。这是一种特殊类型的快照，因为它包含了在它内部的实际的聚合实例。Axon提供的存储库知道这种类型的快照，并从它提取聚合，而不是实例化一个新的。快照事件之后加载的所有事件传输到取出的聚合实例。

><b>注意</b>
确保你使用的序列化器实例(默认为XStreamSerializer)是能够序列化你的聚合的。XStreamSerializer要求使用Hotspot JVM，或者你的聚合要有一个可访问的默认的构造函数或实现Serializable接口。

AbstractSnapshotter提供了一组基本的属性，允许你调整创建快照的方式：

* EventStore设置事件存储，用于加载过去的事件和存储快照。这个事件存储必须实现SnapshotEventStore接口。
* Executor设计executor，比如ThreadPoolExecutor提供了线程来处理实际快照的创建。默认情况下，快照的创建是在线程中调用scheduleSnapshot()方法，一般不建议用于生产。

AggregateSnapshotter提供另一个属性:

* AggregateFactories是允许你设置创建聚合实例工厂的属性。配置多个聚合工厂允许你使用一个单独的Snapshotter为各种聚合类型创建快照。EventSourcingRepository实现提供了访问他们使用的AggregateFactory。这可以用于配置相同的聚合工厂像在存储库中使用的Snapshotter一样。

><b>注意</b>
如果你使用一个executor在另一个线程中执行快照创建，如果必要的话，确保你为潜在的事件存储配置正确的事务管理。
Spring用户可以使用SpringAggregateSnapshotter，当需要创建一个快照时，它将从应用程序上下文自动查找合适的AggregateFactory。

## 存储快照事件
当快照存储在事件存储中时，它会自动使用快照归纳所有之前的事件并将其返回到它们的位置。所有事件存储实现允许并发创建快照。这意味着它们允许快照被存储的同时，另一个进程为同一个聚合添加事件。这允许快照进程作为一个完全独立进程。

><b>注意</b>
通常情况下，一旦它们是快照事件的一部分，你就可以归档所有的事件。快照事件将永远不会在常规操作场景中再次读取事件存储。然而,如果你希望能够重建快照创建前一刻的聚合状态，你必须保持事件为最新。

Axon提供了一种特殊类型的快照事件：AggregateSnapshot,它将整个聚合存储为一个快照。动机很简单:你的聚合应该只包含与业务决策相关的的状态。这正是你想要在一个快照中捕获的信息。所有事件溯源存储库由Axon承认的AggregateSnapshot提供，并将从它提取的聚合。注意，使用这个快照事件要求事件序列化机制需要能够对聚合进行序列化。

## 根据快照事件初始化聚合
快照事件是一个和其他事件一样的事件。这意味着一个快照事件就像任何其他领域事件一样被处理。当使用注解来划分事件处理程序(@EventHandler)时，你可以注解一个方法，基于快照事件初始化全部的聚合状态。下面的代码示例演示了，如何像对待任何其他聚合中的领域事件一样对待快照事件。

```
public class MyAggregate extends AbstractAnnotatedAggregateRoot {

    // ... code omitted for brevity

    @EventHandler
    protected void handleSomeStateChangeEvent(MyDomainEvent event) {
        // ...
    }

    @EventHandler
    protected void applySnapshot(MySnapshotEvent event) {
        // the snapshot event should contain all relevant state
        this.someState = event.someState;
        this.otherState = event.otherState;
    }
}
```

有一种类型的快照事件处理方式不同:AggregateSnapshot。这种类型的快照事件包含实际的聚合。聚合工厂识别这种类型的事件并从快照中提取聚合。然后，将所有其他事件重新应用到提取的快照。这意味着聚合从不需要能够处理AggregateSnapshot实例自身。

## 先进的冲突检测和解决方案
明确改变的含义作为一个主要的优势，就是你可以更精确地检测冲突的变化。通常,这些冲突的变化，发生在两个用户同时处理相同的数据(几乎)时。想象一下两个用户都查看一个特定版本的数据。他们都决定对这些数据进行修改。他们都将发送一个命令就像“在这个聚合的X版本上，那样做”，其中X是聚合的预期版本。其中一个会将修改实际应用于预期的版本。另一个用户不会。

当聚合已经被另一个进程修改时，你可以检查用户的意图与任何看不见的修改是否冲突，而不是简单地拒绝所有传入命令。
检测冲突，传递一个ConflictResolver类型的参数到你的聚合的 @CommandHandler方法。这个接口提供了detectConflicts方法，允许你在执行特定类型的命令时，定义被认为是冲突的事件类型。

><b>注意</b>
注意ConflictResolver只会包含任何潜在的冲突事件，如果聚合用一个预期的版本加载。使用@TargetAggregateVersion在一个命令的字段上标示聚合的预期的版本。

如果找到事件匹配的断言（predicate），抛出异常(detectConflicts可选的第二个参数允许你定义抛出的异常)。如果没有找到，处理将继续正常进行。

如果没有调用detectConflicts，并有潜在冲突的事件,@CommandHandler将失败。这可能是提供一个预期的版本的情况下，在@CommandHandler方法的参数中没有可用的ConflictResolver 。

作者：勇赴
链接：https://www.jianshu.com/p/0cf9c4c0e037
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。