---
title: AxonFramework工作单元
date: 2018-08-20 13:17:12
categories: AxonFramework
tags: [AxonFramework]
author: 勇赴
---

工作单元是Axon Framework的一个重要的概念,虽然在大多数情况下你可能不太直接与它交互。消息的处理被视为一个独立单元。工作单元的目的是在处理一条消息(命令或事件)期间去协调行为的执行。组件可以在工作单元每一个阶段执行注册行为,如onPrepareCommit或onCleanup。

<!-- more -->

你可能不太需要直接访问工作单元，主要使用Axon提供的构件来访问。不管出于什么原因，如果你需要访问它，有几种方式可以获得它。处理器可以在处理函数中通过一个参数来接收工作单元。如果你使用注解，你可以添加一个UnitOfWork类型参数到你被注解的函数。在其他地方,你可以通过调用CurrentUnitOfWork.get()把工作单元绑定到当前线程。注意，如果没有把工作单元绑定到当前线程，这个方法将抛出一个异常。使用CurrentUnitOfWork.isStarted()可检测它是否可用。

需要访问当前工作单元的一个原因是，在消息处理过程中附加需要多次重复使用的资源，或者，如果创建资源时需要在工作单元结束时进行清理工作。在这种情况下,unitOfWork.getOrComputeResource()和生命周期回调方法,如onRollback(),afterCommit()和onCleanup()，允许你在工作单元处理期间，注册资源和声明行为。

><b>注意</b>
注意工作单元只是变化的缓存，不是事务的替代品。尽管当工作单位被提交时，所有的阶段更改都被提交，但它的提交不是原子性的。这意味着，当提交失败时一些变化可能会被持久化，而另一些则没有。最佳实践要求命令不应该包含多个行为。如果你坚持这种做法，一个工作单元只包含一个单一的行为，从而将使其被安全使用。如果你的工作单元中有更多的行为，那么你可以考虑将一个事务附加到工作单元的提交。当工作单元提交时，使用unitOfWork.onCommit(. .)注册一个行为。*

你的处理器可以抛出一个异常作为处理消息的结果。默认情况下,未经检测的异常将导致UnitOfWork回滚所有的更改。结果是 ，预定的副作用被撤消。

Axon 提供了一些开箱即用的回滚策略:

* RollbackConfigurationType.NEVER ,总是提交工作单元。
* RollbackConfigurationType.ANY_THROWABLE ,当异常发生时总是回滚。
* RollbackConfigurationType.UNCHECKED_EXCEPTIONS，错误和运行时异常回滚。
* RollbackConfigurationType.RUNTIME_EXCEPTION ,运行时异常回滚（但不是错误）

当使用Axon 组件处理消息时，将自动为你管理工作单元的生命周期。如果你选择不使用这些组件，但实现自己的处理，则需要以编程方式启动和提交(或回滚)工作单元。

在大多数情况下，DefaultUnitOfWork将为你提供你所需要的功能。它在单线程中处理进程。在一个工作单元上下文中执行一个任务，可以new一个DefaultUnitOfWork并简单地调用UnitOfWork.execute(Runnable)或UnitOfWork.executeWithResult(Callable)函数。典型的用法如下:

<pre>
// then, either use the autocommit approach: 
uow.executeWithResult(() -> ... logic here);
// or manually commit or rollback:
 try {
    // business logic comes here
    uow.commit();
} catch (Exception e) {
    uow.rollback(e);
    // maybe rethrow...
}`
``
</pre> 

一个工作单元了解各个阶段。每当它进展到不同的阶段时，就会通知UnitOfWork监听器。

**活动阶段:**这是开始工作单元的地方。工作单元通常在当前线程中的这个阶段被注册(通过CurrentUnitOfWork.set(UnitOfWork))。随后，消息通常在此阶段由消息处理器处理。

**提交阶段:**在处理完消息之后，但是在提交工作单元之前，将调用onPrepareCommit监听器。如果一个工作单元被绑定到一个事务中，那么将调用onCommit监听器来提交任何的支持事务。当提交成功时，将调用afterCommit监听器。如果在提交或任何步骤之前失败，将调用onRollback监听器。如果可用，则消息处理器的结果包含在工作单元的ExecutionResult中。

**清理阶段:**在此阶段，该工作单元(如锁)所持有的任何资源都将被释放。如果多个工作单元被嵌套，那么清理阶段将被推迟到外部的单元工作都准备好清理为止。

消息处理过程可以被认为是一个原子过程;它要么完全被处理，要么完全不被处理。Axon Framework使用工作单元来跟踪消息处理器执行的操作。在处理器完成后，Axon 将尝试提交在工作单元中注册的操作。

将事务绑定到工作单元是可能的。许多组件，例如CommandBus的实现和所有异步处理事件处理器，都允许你配置事务管理器（Transaction Manager）。然后，该事务管理器将被用于创建事务，以绑定到用于管理消息处理的工作单元。

当应用程序组件在消息处理的不同阶段需要资源时，比如，数据库连接或实体管理器(EntityManager)，这些资源能

作者：勇赴
链接：https://www.jianshu.com/p/3814c293f4fc
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。
