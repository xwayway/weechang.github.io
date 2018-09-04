---
title: AxonFramework-事件发布和处理
date: 2018-08-31 10:44:40
categories: AxonFramework
tags: [AxonFramework]
author: 勇赴
---

由应用程序生成的事件需要被分发到更新查询数据库的组件，搜索引擎或其他需要它们的资源：事件处理程序（Event Handlers）。事件总线（Event Bus）的职责是分发事件消息到所有感兴趣的组件。接收端，事件处理器（Event Processors）负责处理这些事件，其中包括相应的事件处理程序（Event Handlers）的调用。

<!-- more -->

## 发布事件（Publishing Events）

在绝大多数情况下，聚合将通过申请（applying）它们发布事件。然而，有时候也有必要发布一个事件(可能来自另一个组件)，直接到事件总线。发布一个事件，在EventMessage中简单封装描述事件的有效负载（payload）。

GenericEventMessage.asEventMessage(Object)方法允许你将任何对象包装成EventMessage。如果传递的对象已经是EventMessage，它只是返回。

## 事件总线（Event Bus）

EventBus是分发事件到已订阅的事件处理器的机制。Axon 提供了两个事件总线的实现：SimpleEventBus和EmbeddedEventStore。两个实现都支持订阅和跟踪处理器(processor)，EmbeddedEventStore持久化事件，它允许你在以后的阶段重放它们。SimpleEventBus有一个易失性存储器，然后一旦事件已经发布到订阅组件上，就会“忘记”它们。

当使用配置API时，默认情况下使用SimpleEventBus。配置EmbeddedEventStore则相反，需要提供一个StorageEngine的实现，它对事件进行实际存储。

<pre>
Configurer configurer = DefaultConfigurer.defaultConfiguration();
configurer.configureEmbeddedEventStore(c -> new InMemoryEventStorageEngine());
</pre>

## Event Processors

事件处理程序（Event Handlers）定义了接收事件时执行的业务逻辑。事件处理器（Event Processors）负责解决该过程的技术方面的组件。它们启动一个工作单元，也可能一个事务，并且确保相关的数据在事件处理期间，可以正确附加到所有创建的消息上。

事件处理器大致有两种形式：订阅和跟踪。订阅事件处理器订阅它们自己给事件源，并由发布机制管理的线程调用。另一方面，跟踪事件处理器使用自己管理的线程从源中取出它们的消息。

## 将处理程序分配给处理器

所有处理器都有一个名称，它跨JVM实例标识一个处理器实例。两个名称相同的处理器，可以被认为是同一处理器的两个实例。
所有事件处理程序都被附加到一个处理器，它的名字是事件处理程序类的包名。
例如, 下面的类：

* org.axonframework.example.eventhandling.MyHandler,
* org.axonframework.example.eventhandling.MyOtherHandler
* org.axonframework.example.eventhandling.module.MyHandler

将触发两个处理器的创建：

* org.axonframework.example.eventhandling with 2 handlers, and
* org.axonframework.example.eventhandling.module with a single handler

配置API允许你为分配的类配置其他策略给处理器，甚至将特定实例分配给特定的处理器。

## 配置处理器

默认情况下，Axon将使用订阅事件处理器。这是有可能改变处理程序是如何分配和如何使用配置API的EventHandlingConfiguration类配置处理器的。

EventHandlingConfiguration类定义了一些的方法，可用于定义处理器需要如何配置。

* registerEventProcessorFactory允许你定义一个默认的工厂方法，创建没有明确定义工厂的事件处理器。
* registerEventProcessor(String name, EventProcessorBuilder builder)定义了用于创建一个带有给定名称的处理器的工厂方法。注意，此种处理器只有当名称被选择作为任何可用的事件处理程序bean的处理器时才会创建。
* registerTrackingProcessor(String name)定义了一个带有给定名称的处理器，使用默认设置，应该被配置成一个跟踪事件处理器。它使用TransactionManager和TokenStore配置。
* usingTrackingProcessors()设置默认跟踪处理器，而不是订阅处理器。

跟踪处理器不像订阅处理器，需要一个令牌存储来存储它们的进程。跟踪处理器通过其事件流接收的每个消息都伴随着一个令牌。这个令牌允许处理器在任何以后的点重新打开流，并在最后一个事件中提取它。

配置API负责令牌存储，以及大多数其他组件处理器需要从全局配置实例。如果没有明确定义TokenStore ，在生产中不推荐使用InMemoryTokenStore。

作者：勇赴
链接：https://www.jianshu.com/p/217ac0d8822d
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。
