---
title: AxonFramework-SpringBoot自动配置
date: 2018-08-31 11:10:15
categories: AxonFramework
tags: [AxonFramework]
author: 勇赴
---

Axon支持的SpringBoot自动配置是迄今为止开始配置Axon基础设施组件最简单的选择。只需添加axon-spring-boot-starter依赖性，Axon会自动配置基础设施组件(命令总线，事件总线)，以及运行和储存聚合和Saga所需的任何组件。

<!-- more -->

根据在应用程序上下文中的其他组件，如果他们在应用程序上下文中不是已经明确定义，Axon将定义某些组件。这意味着只需要配置不同于默认值的组件。

## 事件总线和事件存储配置
如果JPA可用，事件存储默认使用JPA事件存储引擎。这允许聚合的存储使用事件溯源而无需任何明确的配置。
如果JPA不可用，Axon默认使用SimpleEventBus，这意味着你需要为每个聚合指定一个非事件溯源存储库，或者在你的Spring配置中配置一个EventStorageEngine 。

配置一个不同的事件存储引擎，即使JPA在class path上，只需定义一个EventStorageEngine类型的bean (使用事件溯源)或EventBus(如果不需要事件溯源)。

## 命令总线配置
如果在应用程序上下文中没有显式定义CommandBus实现，Axon会配置一个SimpleCommandBus。这个CommandBus将使用PlatformTransactionManager管理事务，如果它在上下文中可用。

如果只有CommandBus bean定义是一个DistributedCommandBus实现，Axon仍将配置一个CommandBus实现作为DistributedCommandBus本地segment。这个bean将获取一个“localSegment”限定符。建议定义DistributedCommandBus为@Primary，以便它优先考虑依赖注入。

## 聚合配置
@Aggregate注解(在org.axonframework.spring.stereotype包中)触发自动配置，配置使用带注解的类型的必要组件作为聚合 。注意，只有聚合根需要注解。

Axon会用命令总线自动注册所有带@CommandHandler注解的方法，并且如果不存在则建立一个存储库。

建立一个与默认情况不同的存储库，在应用程序上下文中定义一个。可选地，你可以定义要使用的存储库的名称，在@Aggregate上使用存储库属性。如果没有定义存储库属性，Axon将尝试以聚合的名称使用存储库(第一个字符小写)，后缀为存储库。依此类推，一个MyAggregate类的类型，默认的存储库名字叫myAggregateRepository。如果没有找到那个名称的bean，Axon将定义一个EventSourcingRepository(如果没有EventStore可用其会失败)。

## Saga配置
基础设施组件的配置操作的Saga是@Saga注解触发的(在org.axonframework.spring.stereotype包中)。Axon会配置一个SagaManagert和SagaRepository。SagaRepository将在上下文中使用一个可用的SagaStore(如果找到JPA默认为JPASagaStore)为实际Saga存储。

为Saga使用不同的SagaStores，在每个@Saga注解的sagaStore属性中，提供要使用的SagaStore的bean名称。
Saga将从应用程序上下文中注入资源。注意，这并不意味着Spring-injecting用于注入这些资源。@Autowired和@javax.inject.Inject注解可用于标定依赖关系，但它们由Axon通过寻找这些被注解的字段和方法来注入。构造函数注入(还)不支持。

## 事件处理（Event Handling）配置
默认情况下，所有单例Spring bean组件包含带@EventHandler注解的方法，将订阅一个事件处理器去接收事件消息发布到事件总线。

EventHandlingConfiguration bean，在应用程序上下文可用，有方法来调整事件处理程序的配置。有关详细信息,请参阅配置API配置事件处理程序和事件处理器。

更新事件处理（EventHandling）配置，创建一个autowired方法,设置你想要的配置:

<pre>
@Autowired
public void configure(EventHandlingConfiguration config) {
    config.usingTrackingProcessors(); // default all processors to tracking mode.
}
</pre>

事件处理器(（Event Processors）的某些方面也可以在application.properties中配置。

<pre>
axon.eventhandling.processors["name"].mode=tracking
axon.eventhandling.processors["name"].source=eventBus
</pre>

使用application.yml:

<pre>
axon:
    eventhandling:
        processors:
            name:
                mode: tracking
                source: eventBus
</pre>

源文件属性指的是bean的名称实现了SubscribableMessageSource或StreamableMessageSource，名称应该被用来做上述处理器的事件源。源文件默认事件总线或事件存储在应用程序上下文中定义。

## 启用AMQP
启用AMQP支持，确保axon-amqp模块在类路径上，并且在应用程序上下文中AMQP ConnectionFactory是可用的(例如通过引用spring-boot-starter-amqp)。
在应用程序中转发生成的事件到一个AMQP Channel，一行application.properties配置就够了：

<pre>
axon.amqp.exchange=ExchangeName
</pre>

这将以给定的名称自动发送所有已经发布的事件到AMQP Channel。

默认情况下，发送时没有使用AMQP事务。这可以使用axon.amqp.transaction-mode属性覆盖，并设置它为事务或publisher-ack。

从队列中接收事件，并在Axon应用程序内处理它们，你需要配置一个SpringAMQPMessageSource：

<pre>
@Bean
public SpringAMQPMessageSource myQueueMessageSource(AMQPMessageConverter messageConverter) {
    return new SpringAMQPMessageSource(messageConverter) {

        @RabbitListener(queues = "myQueue")
        @Override
        public void onMessage(Message message, Channel channel) throws Exception {
            super.onMessage(message, channel);
        }
    };
}
</pre>

然后配置一个处理器，使用这个bean作为其消息来源:

<pre>
axon.eventhandling.processors["name"].source=myQueueMessageSource
</pre>

## 使用JGroups分发命令
进行中…如果你不能等，添加一个依赖项到axon-spring-boot-starter-jgroups模块。

作者：勇赴
链接：https://www.jianshu.com/p/d843fe8bd7d1
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。