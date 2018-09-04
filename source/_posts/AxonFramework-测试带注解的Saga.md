---
title: AxonFramework-测试带注解的Saga
date: 2018-08-29 11:30:56
categories: AxonFramework
tags: [AxonFramework]
author: 勇赴
---

与命令处理组件类似，saga有一个明确定义的接口：它们只对事件作出响应。另一方面，saga通常具有时间概念，并且可以作为事件处理过程的一部分与其他组件交互。Axon Framework的测试支持模块包含帮助你编写saga测试的固件。

<!-- more -->

每个测试固件包含三个阶段，类似于前面部分描述的命令处理组件固件。

* given 某些事件(从某些聚合),
* when事件到达或时间流逝，
* expect某些行为或状态。

“given ”和“when”阶段都接受事件作为它们交互的一部分。在“given”阶段，可能会忽略所有副作用，如生成的命令。在“when”阶段，另一方面，从被记录和被验证的saga生成事件和命令。

<pre>
FixtureConfiguration fixture = new SagaTestFixture(InvoicingSaga.class);
fixture.givenAggregate(invoiceId).published(new InvoiceCreatedEvent()) 
       .whenTimeElapses(Duration.ofDays(31)) 
       .expectDispatchedCommandsMatching(Matchers.listWithAllOf(aMarkAsOverdueCommand())); 
       // or, to match against the payload of a Command Message only 
       .expectDispatchedCommandsMatching(Matchers.payloadsMatching(Matchers.listWithAllOf(aMarkAsOverdueCommand())));
</pre>

saga可以分发命令使用一个回调函数通知命令处理的结果。由于在测试中没有实际的命令处理完成，该行为使用CallbackBehavior对象定义。这个对象使用setCallbackBehavior()在固件上注册，并定义当一个命令被分发后，是否和如何必须调用回调函数。

不是直接使用CommandBus，而是你还可以使用命令网关。见下文如何规定他们的行为。

通常，saga将与资源交互。这些资源不是saga状态的一部分，但在saga加载和创建后被注入。测试固件允许你注册需要被注入到saga中的资源。要注册一个资源，只需用资源作为参数调用fixture.registerResource(Object)方法。固件将在saga上检测合适的setter方法或字段(带有@Inject注解)并用一个可用的资源调用它。

><b>提示</b>
注入模拟对象(例如Mockito或Easymock)到你的saga，可能是非常有益的。它允许你验证saga与外部资源的正确交互。

命令网关为saga提供了一个更简单的方式来调度命令。使用自定义命令网关还可以更容易地创建模拟或桩来定义其在测试中的行为。然而，当提供一个模拟或桩时，实际的命令可能不不会被分发，因此无法验证测试固件中发送的命令。

因此，固件提供了两个方法，让你注册命令网关和一个可选的模拟定义其行为:registerCommandGateway(Class)和registerCommandGateway(Class, Object)。这两种方法都返回一个given类的实例，表示要使用的网关。这个实例也注册为资源，使它具有资源注入的资格。

当registerCommandGateway(Class)用来注册一个网关时，它分发命令到由固件管理的CommandBus。网关的行为主要是通过CallbackBehavior定义于固件。如果没有提供明确的CallbackBehavior，回调不调用，从而无法为网关提供任何返回值。

当registerCommandGateway(Class, Object)是用来注册一个网关，第二个参数是用于定义网关的行为。

测试固件试图尽可能消除系统时间迁延。这意味着它将会在测试执行时显得没有时间迁延。除非你显式地声明使用whenTimeElapses()。所有事件都将拥有测试固件创建那一刻的时间戳。

在测试期间暂停时间，可以更容易地预测在什么时间计划发布事件。如果你的测试用例验证一个事件在30秒后计划发布，它将保持30秒，而不管实际调度和测试执行之间的时间。

><b>注意</b>
固件使用StubScheduler作为基于时间的活动，如调度事件和推进时间。固件将发送到saga实例的任何事件的时间戳设置为这个scheduler的时间。这意味着一旦固件开始，时间就“stopped”，并可能提前确定使用whenTimeAdvanceTo和whenTimeElapses方法。

你还可以使用与测试固件无关的StubEventScheduler，如果你需要测试事件的调度。这个EventScheduler实现允许你验证哪个事件被安排在哪个时间，并且给你选择操纵时间的进展。你可以将时间提前到一个特定的时间，将时钟移动到一个特定的日期和时间，或者将时间提前到下一个计划的事件。所有这些操作将返回进度间隔内计划的事件。

作者：勇赴
链接：https://www.jianshu.com/p/49429788e7da
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。