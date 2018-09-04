---
title: AxonFramework跟踪最后期限
date: 2018-08-29 10:11:46
categories: AxonFramework
tags: [AxonFramework]
author: 勇赴
---

当有事发生时，很容易使一个saga采取行动。毕竟，有一个事件要通知saga。但是如果你想让你的saga，在什么事情都没发生的情况下做些什么呢？这就是最后期限。在发票中，通常是几周，而信用卡支付的确认可能在几秒钟内。

<!-- more -->

在Axon中，你可以使用一个EventScheduler计划发布一个事件。在发票的例子中，你希望发票在30内付清。saga将在发送CreateInvoiceCommand后，安排一个InvoicePaymentDeadlineExpiredEvent在30天后发布。计划一个事件后EventScheduler返回ScheduleToken。这个令牌可以用来取消计划，例如已收到发票的付款。

Axon提供了两种EventScheduler实现:一个纯Java和一个使用Quartz 2作为调度机制支持。

这个纯java实现的EventScheduler使用ScheduledExecutorService去计划事件发布。虽然这个计划器的定时非常可靠，但它是一个纯内存实现。一旦JVM关闭，所有的计划都将丢失。这使得这种实现不适合长期限的计划。

SimpleEventScheduler 需要配置一个EventBus和一个SchedulingExecutorService（看java.util.concurrent.Executors类的静态方法的helper方法）。

QuartzEventScheduler是一个更可靠，企业信得过的实现。使用Quartz 作为底层调度机制，它提供了更强大的功能，如持久性、集群和失败的管理。这意味着事件发布将被保证。可能会晚一点，但会发布。

需要配置一个Quartz调度器和EventBus。另外，你可以设置以组的名称进行计划，默认为"AxonFramework-Events"

一个或多个组件将监听计划的事件。这些组件可能依赖于绑定到调用它们的线程的事务。计划的事件由EventScheduler管理的线程发布。管理事务在这些线程上，你可以配置一个TransactionManager或UnitOfWorkFactory创建一个事务绑定工作单元。

><b>注意</b>
Spring用户可以使用quartzeventschedulerfactorybean或simpleeventschedulerfactorybean进行更简单的配置。它允许你直接设置PlatformTransactionManager。

## 注入资源
saga一般做的不仅仅是维护基于事件的状态。它们与外部的组件进行交互。要做到这一点，他们需要访问处理组件所需的资源。通常，这些资源实际上并不是saga状态的一部分，也不应该持久化。但是，一旦重新构建一个saga，在事件被路由到该实例之前，必须注入这些资源。

为了这个目的有了ResourceInjector。它使用SagaRepository将资源注入到一个saga。Axon提供了SpringResourceInjector用应用程序上下文中的资源注入带注解的字段和方法，和一个SimpleResourceInjector，注册已经注册的资源到用@Inject注解的方法和字段。

><b>提示</b>
由于资源与saga不应该被持久化，所以务必向这些字段添加临时关键字。这将防止序列化机制尝试将这些字段的内容写入存储库。存储库将在saga被反序列化之后自动重新注入所需的资源。

Simpleresourceinjector允许一个预先指定的资源集合被注射。它扫描saga的(setter)方法和字段，以找到被@Inject注解的那个。

当使用配置API，Axon将默认为the ConfigurationResourceInjector。它将注入配置中可用的任何资源。组件像EventBus、EventStore CommandBus和CommandGateway默认情况下是可用的,但你也可以使用configurer.registerComponent()注册自己的组件。

Springresourceinjector使用Spring的依赖注入机制注入资源聚合。这意味着，如果需要，可以使用setter注入或直接字段注入。要注入的方法或字段需要注解，以便Spring识别它为依赖项。例如@Autowired。

作者：勇赴
链接：https://www.jianshu.com/p/02e3636fc735
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。