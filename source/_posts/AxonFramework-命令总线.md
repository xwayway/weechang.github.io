---
title: AxonFramework命令总线
date: 2018-08-31 10:25:51
categories: AxonFramework
tags: [AxonFramework]
author: 勇赴
---

命令总线是将命令分发给各自的命令Handler的机制。每个命令总是被精确发送到一个命令Handler。如果没有可用的命令Handler为已分发的命令，将会抛出NoHandlerForCommandException异常。对同一命令类型订阅多个命令Handler将导致订阅互相取代。在这种情况下，最后一个订阅获胜。

<!-- more -->

## 分发命令

CommandBus提供了两个方法去分发命令到它们各自的Handler：dispatch(commandMessage,callback)和dispatch(commandMessage)。第一个参数是一个包含要发送的实际命令的消息。第二个可选的参数接收一个回调，允许在命令处理完成时通知分发组件。这个回调有两个方法:onSuccess()和onFailure()，分别会在命令处理返回后被调用，或者当它抛出一个异常时调用。

调用组件可能不采取在分发命令的同一线程中调用回调。如果调用线程在继续之前依赖于结果，你可以使用FutureCallback。这是一个Future(在java.concurrent包中定义)和Axon的CommandCallback的组合。或者，考虑使用命令网关。

如果一个应用程序不直接对命令的结果感兴趣，可以使用dispatch(commandMessage) 方法。

## SimpleCommandBus

SimpleCommandBus，顾名思义，最简单的实现。它在分发它们的线程中简单的处理命令。命令处理后，修改后的聚合在同一线程被保存和发布生成的事件。在大多数情况下，如web应用程序，该实现将符合你的需求。配置API中SimpleCommandBus是默认使用的实现。

像大多数CommandBus实现一样，SimpleCommandBus允许拦截器进行配置。在命令总线上分发一个命令后调用CommandDispatchInterceptors。在实际的命令handler 方法之前调用CommandHandlerInterceptors，允许你修改或阻塞命令。有关更多信息，请参考命令处理器拦截器。

因为所有命令处理都在同一线程中完成，这个实现仅限于JVM的边界。这个实现的性能是很好的，但不超凡。跨JVM边界，或使你的CPU cycles发挥最大的功效，看看其他CommandBus实现。

## AsynchronousCommandBus

顾名思义，AsynchronousCommandBus实现从分发它们的线程异步执行命令。它使用一个Executor在不同的线程来执行实际的处理逻辑。

默认情况下，AsynchronousCommandBus使用一个unbounded缓存的线程池。这意味着分发一个命令时会创建线程。完成处理命令的线程将被重新用于新命令。如果60秒线程没有处理命令，则会停止线程。

或者，Executor实例可以提供不同的线程策略配置。

注意，应用程序停止时应该关闭AsynchronousCommandBus，以确保任何等待线程正确关闭。关闭，调用shutdown()方法。这也将关闭任何Executor实例，如果它实现ExecutorService接口的话。

## DisruptorCommandBus

SimpleCommandBus具有合理的性能特性，特别是当你经历了性能调优技巧。事实上，SimpleCommandBus需要锁来防止多个线程并发访问同一聚合，导致处理开销和锁争用。

DisruptorCommandBus采用不同的方法进行多线程处理。不是多个线程每个都执行同样的处理，而是有多个线程，每个负责一件处理。DisruptorCommandBus使用Disruptor，一个小的并发编程框架，通过不同的方法对多线程进行处理来实现更好的性能。任务不是在调用线程中进行处理，而是将任务移交给两组线程进行处理，每组线程负责一部分处理。第一组的线程将执行命令handler，更改一个聚合的状态。第二组将存储并将事件发布到事件存储。

虽然DisruptorCommandBus轻易优于SimpleCommandBus 4倍(!)，但有一些限制:

* DisruptorCommandBus仅支持事件溯源聚合。这个命令总线充当由Disruptor处理聚合的存储库。获取一个存储库的引用，使用createRepository(AggregateFactory)。
* 一个命令只能导致一个聚合实例状态变化。
* 当使用缓存时，它只允许给定的标识符为单个聚合。这意味着它是不可能有两个具有相同的标识符的不同类型的聚合。
* 命令一般不会引发需要回滚工作单元的故障。当发生回滚时，DisruptorCommandBus不能保证命令按照它们被分发的顺序进行处理。此外，它需要重试其他命令，从而造成不必要的计算。
* 在创建一个新的聚合实例时，命令更新所创建实例可能并不完全按照所提供的顺序进行。一旦创建了聚合，所有命令将按照它们被分发顺序执行。为了确保顺序，在创建命令上使用回调去等待正在创建的聚合。它不应该耗时超过几毫秒。
构建一个DisruptorCommandBus实例，你需要一个EventStore。该组件在Repositories and Event Stores中有解释。
或者，你可以提供一个DisruptorConfiguration实例，它允许你调整配置优化你的特定环境下的性能：

* Buffer size:在ringBuffer上注册传入命令的槽数。更高的值可能会增加吞吐量,但也导致更高的延迟。必须是2的次方数，默认为4096。
* ProducerType: 表示条目是由单线程或多线程生成的。默认为多线程。
* WaitStrategy：当处理器线程（三个线程负责的实际处理）需要等待对方时使用的策略。最好的WaitStrategy取决于机器上可用的处理器数量，和正在运行的其他进程的数量。如果低延迟是至关重要的，DisruptorCommandBus可以自己认领内核，你可以使用BusySpinWaitStrategy。为了使命令总线索取更少的CPU并且允许其他线程处理，使用YieldingWaitStrategy。最后，你可以使用SleepingWaitStrategy和BlockingWaitStrategy允许其他进程共享CPU。如果命令总线不需要进行专职处理，则后者是合适的。默认为BlockingWaitStrategy。
* Executor：设置Executor为DisruptorCommandBus提供线程。这个Executor必须能够提供至少4个线程。其中的3个线程，由DisruptorCommandBus的处理组件认领。额外的线程用于调用回调函数，并计划重试以防检测到错误的聚合状态。默认是CachedThreadPool提供线程从一个称为“DisruptorCommandBus”的线程组中。
* TransactionManager：定义了事务管理器，应该确保存储和事件发布以事务的方式执行。
* InvokerInterceptors:定义了在调用处理中使用的CommandHandlerInterceptors。这个处理调用实际的命令处理器方法。
* PublisherInterceptors:定义了在发布处理中使用的CommandHandlerInterceptors。这个发布处理存储和发布生成的事件。
* RollbackConfiguration:定义工作单元应该回滚的异常。默认配置为回滚未经检查的异常。
* RescheduleCommandsOnCorruptState:指示已经执行过命令但损坏的聚合(如:因为一个工作单元是回滚)是否应该重新计划。如果为假，回调的onFailure()方法将被调用。如果为的(默认),命令将被重新计划。
* CoolingDownPeriod:设置等待的秒数，以确保所有命令被处理。在冷却期间，不接受新命令，但是现有的命令仍然处理，并在必要时重新计划。冷却期间确保线程可供重新安排命令和调用回调之用。默认为1000(1秒)。
* 缓存:设置缓存存储从Event Store中恢复的聚合实例。缓存用disruptor存储不活跃的聚合实例。
* InvokerThreadCount:给命令处理器的调用分配线程的数量。一个好的起始点是机器内核数量的一半。
* PublisherThreadCount:用于发布事件的线程数量。一个好的起始点是一半的内核数量，如果IO上花费大量的时间，可以增加。
* SerializerThreadCount:使用pre-serialize事件的线程数量。默认为1，但如果没有配置序列化器将被忽略。
* Serializer:用于执行pre-serialization的序列化器。当配置序列化器时，DisruptorCommandBus将包装所有生成的事件在一个SerializationAware消息上。附加有效负载和元数据的序列化形式，在发布到事件存储之前。

作者：勇赴
链接：https://www.jianshu.com/p/6aee33abd7f0
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。
