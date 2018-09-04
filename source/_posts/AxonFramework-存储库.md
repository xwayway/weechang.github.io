---
title: AxonFramework-存储库
date: 2018-08-31 10:50:48
categories: AxonFramework
tags: [AxonFramework]
author: 勇赴
---

存储库是提供对聚合访问的机制。存储库充当了用于保存数据的实际存储机制的网关。在CQRS中，存储库只需要能够根据他们的惟一标识符找到聚合。任何其他类型的查询，应该在查询数据库中执行。

<!-- more -->

在Axon Framework中,所有存储库必须实现Repository接口。这个接口规定了三种方法:load(identifier, version)， load(identifier)和newInstance(factoryMethod)。load方法允许你从存储库加载聚合。version可选参数是用来检测并发修改(见Advanced conflict detection and resolution)。newInstance用于注册新创建的聚合到存储库中。

基于你潜在的持久性存储和审计需求，有一些基础实现提供大了部分存储库所需的基本功能。Axon Framework对保存聚合当前状态的存储库(见Standard Repositories)和那些存储聚合事件的存储库(见 Event Sourcing Repositories)进行了区分。

注意，存储库接口没有声明delete(identifier)方法。删除聚合是通过在一个聚合内部调用AggregateLifecycle.markDeleted()方法完成的。删除聚合是一个与其他迁移一样的状态迁移，唯一的区别是它在许多情况下是不可逆的。你应该在聚合上创建自己的有意义的方法，来将聚合的状态设置为“已删除”。这也允许你注册你想要发布的任何事件。

## Standard repositories（标准存储库）

标准存储库存储聚合的实际状态。在每次修改后，新的状态将覆盖旧的。这使得命令组件也可以使用应用程序的查询组件使用的相同信息。标准存储库存储聚合的实际状态。这可能取决于你正在创建的应用程序的类型，这是最简单的解决方案。如果是这样的话，Axon提供了一些帮助你实现这样一个存储库的构件。

Axon为标准存储库提供了一个开箱即用的实现：GenericJpaRepository。它认为聚合是一个有效的JPA实体。它使用EntityManagerProvider配置，EntityManagerProvider提供EntityManager来管理实际的持久化，并且一个类指定聚合的实际类型存储在存储库中。当聚合调用静态AggregateLifecycle.apply()方法时，你也可以通过EventBus去发布事件。

你也可以轻松实现自己的存储库。在这种情况下，最好从抽象类LockingRepository扩展。作为聚合的包装器类型，建议使用AnnotatedAggregate。看GenericJpaRepository的源码示例。

## Event Sourcing repositories（事件溯源存储库）

聚合根能够根据事件重建它们的状态，也可以配置为通过事件溯源存储库加载。这些存储库不存储聚合本身，但存储聚合生成的一系列事件。基于这些事件，可以随时恢复聚合的状态。

在AxonFramework中，EventSourcingRepository实现提供了任何事件溯源存储库所需的基本功能。这依赖于EventStore(见Implementing your own Event Store)，它抽象了实际的存储机制。

根据情况，你可以提供一个聚合工厂。AggregateFactory指定了如何创建一个聚合实例。一旦创建了一个聚合，EventSourcingRepository可以使用从事件存储中加载的事件来初始化。Axon Framework自带了一些你可以使用的AggregateFactory实现。如果他们还不够，可以很容易创建你自己的实现。

## GenericAggregateFactory

GenericAggregateFactory是一种特别的AggregateFactory实现，可用于任何类型的事件溯源聚合根。GenericAggregateFactory创建存储库管理的聚合类型的一个实例。聚合类必须是非抽象的，声明一个默认的不进行初始化的无参构造函数化。
GenericAggregateFactory适用于大多数场景，聚合不需要专门注入non-serializable资源。

## SpringPrototypeAggregateFactory

根据你的架构选择，使用Spring将依赖项注入到聚合中可能是有用的。例如，你可以将查询库注入到你的聚合，以确保某些值的存在(或不存在)。

注入依赖项到你的聚合，在定义了SpringPrototypeAggregateFactory的Spring上下文中，你需要配置一个聚合根的属性bean。不是使用构造函数创建的常规的实例，而是使用Spring应用程序上下文实例化你聚合。这也将在你的聚合中注入的任何依赖项。

## 实现自己的AggregateFactory

在某些情况下，GenericAggregateFactory不能提供你所需要的东西。例如，你可以有一个抽象的聚合类型与不同的场景的多个实现(例如，PublicUserAccount和BackOfficeAccount都扩展一个帐户)。而不是为每个聚合创建不同的存储库，你可以使用一个单独的存储库，并配置一个AggregateFactory意识到不同的实现。

聚合工厂大部分的工作是创建未初始化的聚合实例。它必须使用一个给定的聚合标识符和流中的第一个事件。通常，这个事件是一个创建事件，其中包含关于预期聚合类型的提示。你可以使用这些信息来选择一个实现并调用其构造函数。确保没有事件被应用于构造函数；聚合必须未初始化。

相对于简单的存储库直接加载聚合的实现，基于事件初始化聚合可能是一项耗时的工作,。CachingEventSourcingRepository提供一个可以从中加载聚合的缓存

作者：勇赴
链接：https://www.jianshu.com/p/585e95e31704
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。
