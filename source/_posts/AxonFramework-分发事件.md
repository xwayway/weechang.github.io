---
title: AxonFramework-分发事件
date: 2018-08-31 10:47:38
categories: AxonFramework
tags: [AxonFramework]
author: 勇赴
---

在某些情况下，有必要发布事件到外部系统,比如消息broker。

<!-- more -->

## Spring AMQP

Axon提供了开箱即用的支持从一个AMQP message broker中转递事件和传递事件到broker中，比如Rabbit MQ。

## 将事件转发到AMQP Echange

SpringAMQPPublisher将事件转发给一个AMQP Exchange。它使用SubscribableMessageSource初始化，一般是EventBus或EventStore。从理论上讲，这可能是发布者可以订阅的任何事件源。

要配置SpringAMQPPublisher，只需一个实例定义为Spring Bean。有一些setter方法允许你指定你预期的行为，如事务支持，publisher 确认(如果由broker支持)，和交换名称。

默认的exchange名称为'Axon.EventBus'

><b>注意</b>
注意，exchanges不会被自动创建。你还必须声明队列、Exchanges 和你希望使用的Bindings 。检查Spring文档了解更多信息。

## 从AMQP 队列中读取事件

Spring已经广泛的支持从一个AMQP队列读取消息。然而,这需要与Axon“架桥”，以便这些消息可以从Axon处理，就像它们是常规事件消息一样。

SpringAMQPMessageSource允许事件处理器从队列中读取消息，而不是事件存储或事件总线。它作为一个Spring AMQP和SubscribableMessageSource之间的适配器需要这些处理器。

配置SpringAMQPMessageSource最简单的方法，是通过定义一个bean，重写默认的onMessage方法并使用@RabbitListener注解，如下:

```
@Bean
public SpringAMQPMessageSource myMessageSource(Serializer serializer) {
    return new SpringAMQPMessageSource(serializer) {
        @RabbitListener(queues = "myQueue")
        @Override
        public void onMessage(Message message, Channel channel) throws Exception {
            super.onMessage(message, channel);
        }
    };
}
```

Spring的@RabbitListener注解告诉Spring，这个方法需要被给定队列上的每个消息调用(myQueue的例子)。此方法简单地调用super.onMessage()方法，执行实际事件的发布到已经订阅它的所有处理器。

订阅这个MessageSource的处理器，将正确的SpringAMQPMessageSource实例传递给订阅处理器的构造函数:

```
// in an @Configuration file:
@Autowired
public void configure(EventHandlingConfiguration ehConfig, SpringAmqpMessageSource myMessageSource) {
    ehConfig.registerSubscribingEventProcessor("myProcessor", c -> myMessageSource);
}
```

请注意跟踪处理器与SpringAMQPMessageSource不兼容。

## 异步事件处理

异步处理事件推荐的方法是使用跟踪事件处理器。这个实现可以保证所有事件的处理，甚至在发生系统故障的情况下(假定事件已经被持久化)。

然而,也有可能在SubscribingProcessor中异步处理事件。要做到这一点，SubscribingProcessor必须用EventProcessingStrategy配置。这种策略可以用来改变事件监听器的调用应如何管理。

默认策略(DirectEventProcessingStrategy)在传递事件的线程中调用这些事件处理程序。这允许处理器使用现有的事务。

其他Axon-provided strategy是AsynchronousEventProcessingStrategy。它使用一个Executor异步调用事件侦听器。

尽管AsynchronousEventProcessingStrategy异步执行，某些事件按顺序处理仍然是可取的。SequencingPolicy定义事件是否必须按顺序处理，并行或两者的结合。策略返回给定事件的序列标识符。如果两个事件的策略返回一个相等的标识符,这意味着他们必须由事件处理程序按顺序处理。一个空序列标识符意味着事件可能与任何其他事件并行处理。

Axon提供了一些可以使用的通用策略：

* FullConcurrencyPolicy会告诉Axon，事件处理程序可以处理所有并发事件。这意味着需要按特定顺序处理的事件之间没有关系。
* SequentialPolicy告诉Axon，所有活动必须按照顺序进行处理。处理事件的操作将在处理前一个事件完成时开始。
* SequentialPerAggregatePolicy将迫使领域事件从同一聚合顺序处理。然而，事件来自不同聚合可能并发处理。这通常是一个合适的政策，用于事件侦听器更新数据库中聚合的细节。

除了这些提供的策略之外，你可以定义自己的策略。所有策略都必须实现SequencingPolicy接口。这个接口定义了一个方法getSequenceIdentifierFor，返回值为一个给定的事件序列标识符。返回相等序列标识符的事件必须按顺序处理。产生不同序列标识符的事件可以同时处理。出于性能原因，如果事件可以并行处理任何其他事件，则策略实现应该返回null。这样更快，因为Axon不必检查对事件处理的任何限制条件。

当使用AsynchronousEventProcessingStrategy时建议明确定义一个ErrorHandler。默认的ErrorHandler传播异常，但在异步执行中没有什么可以传播的，除了Executor。这可能导致事件没有被处理。相反，建议使用一个ErrorHandler报告错误，并允许继续处理。ErrorHandler被配置在SubscribingEventProcessor的构造函数上，还提供了EventProcessingStrategy。

作者：勇赴
链接：https://www.jianshu.com/p/c333f2c05239
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。