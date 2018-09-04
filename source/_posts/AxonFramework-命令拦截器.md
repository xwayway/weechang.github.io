---
title: AxonFramework命令拦截器
date: 2018-08-31 10:29:32
categories: AxonFramework
tags: [AxonFramework]
author: 勇赴
---

使用命令总线的优点之一，是能够基于所有传入的命令采取相应的行动。例子是，不论哪种命令类型，你都希望去做日志记录或身份验证。这是使用拦截器来完成的。

<!-- more -->

有不同类型的拦截器：Dispatch拦截器和Handler拦截器。前者在命令被分发到命令处理程序之前被调用。在那时，它甚至不能确定该命令有任何处理器的存在。后者在命令处理程序被调用之前调用。

## Message Dispatch Interceptors（消息分发拦截器）

当命令在命令总线上被分发时调用消息分发拦截器。例如，它们可以通过添加元数据来更改命令消息，或通过抛出异常来阻塞命令。这些拦截器总是在分发命令的线程上被调用。

##Structural validation（结构性验证）

如果它没有包含正确格式的所有必需的信息，那么处理命令是没有意义的,。事实上，一个命令缺乏信息应该尽可能早地被阻塞，甚至最好是任何事务开始之前。因此，拦截器应该检查所有传入命令的信息的可用性。这就是所谓的结构性验证。

Axon Framework支持基于JSR 303 Bean Validation的验证。这允许你用像@NotEmpty和@Pattern这样的注解，去注解命令上的字段。你需要在你的类路径中include一个JSR 303实现(比如Hibernate-Validator)。然后，在命令总线上配置一个BeanValidationInterceptor，它会自动找到并配置你的验证器实现。虽然它使用合理的默认值，但你可以根据具体需要调整它。

><b>提示</b>
你想在一个无效的命令上使用尽可能少的资源。因此，该拦截器通常是位于拦截器链的最前端。在某些情况下，一个日志或审计拦截器可能需要放在前面，紧跟着它的是验证拦截器。

BeanValidationInterceptor还实现了MessageHandlerInterceptor，也允许你把它配置为一个处理程序（Handler）拦截器。

## Message Handler Interceptors（消息处理程序拦截器）

消息处理程序拦截器可以在命令处理之前和之后执行操作。拦截器甚至可以完全阻止命令处理，例如出于安全原因。

拦截器必须实现MessageHandlerInterceptor接口。该接口声明了一个方法handle，它需要三个参数：命令消息，当前的UnitOfWork和InterceptorChain。InterceptorChain用于继续分发处理。

与分发拦截器不同，处理程序拦截器在命令处理程序上下文中被调用。这意味着它们可以根据正在处理消息的工作单元附上相关数据。然后这个相关数据将被附加到在工作单元的上下文中被创建的消息。

处理程序拦截器也通常用于管理围绕命令处理的事务。这么做，注册一个TransactionManagingInterceptor，使用TransactionManager依次配置启动和提交(或回滚)实际事务。

## Distributing the Command Bus(分布式命令总线)

CommandBus的实现在早期声称只允许命令消息在单个JVM上分发。有时候，你想把不同JVM中的命令总线的多个实例作为一个。当返回任何结果时，在一个JVM命令总线上发出的命令应该无缝地传到到另一个JVM中的命令处理程序。

这就是DistributedCommandBus（分布式命令总线）的由来。不像其他CommandBus的实现，DistributedCommandBus不调用任何处理器。它的作用是在不同的JVM的命令总线实现之间形成一座“桥”。每个JVM上的DistributedCommandBus实例称为“Segment”。

{% asset_img distributed.png distributed %}

><b>注意</b>
虽然分布式命令总线本身是Axon Framework核心模块的一部分，但它需要的组件，你可以在其中一个以axon-distributed-commandbus -开头的模块中找到。如果你使用Maven，确保你有适当的依赖集。groupId和version与核心模块相同。

DistributedCommandBus依赖于两个组件：一个是CommandBusConnector，实现JVM的之间的通信协议；一个是CommandRouter，为每个传入的命令选择目的地。这个路由器定义分布式总线命令的segment应该given一个命令，根据路由键计算得到路由策略。两个具有相同路由键的命令将始终路由到相同的segment，只要segment数量和配置没有改变。一般来说，用目标聚合的标识符作为路由键。

提供两个RoutingStrategy的实现：MetaDataRoutingStrategy，它使用元数据属性在命令消息中查找路由键，而AnnotationRoutingStrategy，它使用注解在命令消息有效负载上的@TargetAggregateIdentifier来提取路由键。显然，你也可以提供自己的实现。

默认情况下，当命令消息没有键能被解析时，RoutingStrategy实现将抛出一个异常时。这种行为可以改变，通过在MetaDataRoutingStrategy或AnnotationRoutingStrategy的构造函数中提供一个UnresolvedRoutingKeyPolicy。有三个可能的策略:

* ERROR:这是默认值，当路由键不可用时，会抛出一个异常。
* RANDOM_KEY：将返回一个随机值，当一个路由键不能从命令消息解析。这实际上意味着这些命令将被路由到命令总线的随机segment。
* STATIC_KEY:将返回一个静态键(现有的“未被解析的”)为未被解析的路由键。这实际上意味着所有这些命令将被路由到相同的segment，只要segment的配置不改变。

作者：勇赴
链接：https://www.jianshu.com/p/d603a00bc5cc
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。
