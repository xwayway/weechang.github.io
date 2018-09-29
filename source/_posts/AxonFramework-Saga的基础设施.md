---
title: AxonFramework-Saga的基础设施
date: 2018-08-29 10:17:20
categories: AxonFramework
tags: [AxonFramework]
author: 勇赴
---

事件需要被重定向到适当的saga实例。为此,一些基础设施类是必需的。最重要的组件是SagaManager和SagaRepository。

<!-- more -->

## Saga Manager

与处理事件的任何组件一样，processing也是由事件处理器完成的。然而，因为saga不是单例实例处理事件，但是有独特的生命周期，它们需要管理。

Axon通过AnnotatedSagaManager来支持生命周期管理，这是提供给一个事件Processor来执行处理器的实际的调用。它的初始化使用saga的类型来管理，也使用可以存储和恢复的SagaRepository这种saga类型。一个AnnotatedSagaManager只能管理一个saga类型。

当使用配置API时，Axon将对大多数组件使用合理的默认设置。不管怎样，强烈建议定义一个SagaStore的实现使用。SagaStore是在某处物理存储saga实例的机制，AnnotatedSagaRepository(默认)要求它们使用SagaStore去存储和检索saga实例。

```
Configurer configurer = DefaultConfigurer.defaultConfiguration();
configurer.registerModule(
        SagaConfiguration.subscribingSagaManager(MySagaType.class)
                         // Axon defaults to an in-memory SagaStore, defining another is recommended
                         .configureSagaStore(c -> new JpaSagaStore(...)));

// alternatively, it is possible to register a single SagaStore for all Saga types:
configurer.registerComponent(SagaStore.class, c -> new JpaSagaStore(...));
```

## saga存储库和saga存储

SagaRepository负责存储和检索saga，使用SagaManager。它能够通过标识符和关联值检索特定的saga实例。然而，有一些特殊的要求。因为在saga中的并发处理是一个非常微妙的过程，存储库必须确保每个概念的saga实例（具有相同的标识符）只有一个实例存在于JVM。

Axon提供了 AnnotatedSagaRepository 实现，允许saga实例的查找，同时保证在同一时间只有一个saga实例被访问。它使用一个SagaStore执行saga实例实际的持久化。

实现的选择主要取决于应用程序使用的存储引擎。Axon提供了JdbcSagaStore、InMemorySagaStore JpaSagaStore 和MongoSagaStore。

在某些情况下,应用程序受益于缓存Saga实例。在这种情况下，有一个CachingSagaStore包装另一个实现添加缓存行为。
请注意，CachingSagaStore 是一 个write-through 缓存，这意味着保存操作总是立即转发到后备存储器，以确保数据安全。

## JpaSagaStore

JpaSagaStore使用JPA来存储状态和关联saga的值。saga本身并不需要任何JPA注解；Axon将使用序列化器序列化saga(类似于事件序列化，你可以使用JavaSerializer或XStreamSerializer)。

JpaSagaStore通过EntityManagerProvider来配置,它提供对EntityManager实例的访问。这种抽象允许使用应用程序管理和容器管理EntityManagers。根据情况,你可以定义序列化器去序列化saga实例。Axon默认为XStreamSerializer。

## JdbcSagaStore

JdbcSagaStore使用原来的JDBC存储saga实例及其关联值。类似于JpaSagaStore，saga实例不需要知道他们是如何存储的。序列化使用序列化器。

用DataSource 或ConnectionProvider初始化JdbcSagaStore 。虽然不是必需的，用ConnectionProvider初始化时，建议在UnitOfWorkAwareConnectionProviderWrapper包装实现。它将检查已经打开的数据库连接的当前工作单元，以确保工作单元中的所有活动都是在单个连接上完成的。

不像JPA，JdbcSagaRepository使用普通的SQL语句，存储和检索信息。这可能意味着某些操作依赖于数据库特有的SQL方言。情况也可能是这样，某些数据库供应商提供了你希望使用的非标准特性。允许这个，你可以提供自己的SagaSqlSchema。SagaSqlSchema接口定义的存储库的所有操作要在底层数据库上执行。它允许你定制为它们执行的SQL语句。默认是GenericSagaSqlSchema。其他可用的实现是PostgresSagaSqlSchema,Oracle11SagaSqlSchema 和HsqlSagaSchema。

## MongoSagaStore

MongoSagaStore存储saga实例和它们的关联到MongoDB数据库中。MongoSagaStore存储在单个集合中的所有saga到MongoDB数据库中。每个saga实例创建一个文档。

MongoSagaStore还确保在任何时候，在单个JVM中任何唯一的saga只有一个saga实例存在。这样可以确保不因并发问题而丢失任何状态更改。

使用MongoTemplate和一个可选的序列化器初始化MongoSagaStore。MongoTemplate提供了一个对集合的引用存储Sagas。Axon 提供了DefaultMongoTemplate，DefaultMongoTemplate获取MongoClient实例，还获取数据库名称和集合的名称存储Sagas。数据库名称和集合的名称可以省略。在这种情况下，他们分别默认为“axonframework”和“sagas”。

## 缓存
如果使用一个支持的saga存储的数据库，保存和加载Saga 实例可能是相对昂贵的操作。特别是在很短的时间段内多次调用同一个saga实例的情况下，缓存对应用程序的性能是有利的。

Axon 提供了CachingSagaStore实现。这个SagaStore包装实际的存储。加载saga或关联值时，CachingSagaStore将首先查阅其缓存，优于委托给包装的存储库。当存储信息时，所有调用总是被委托，以确保后备存储器总是有一个与saga的状态一致的视图。

配置缓存,简单包装任何SagaStore到CachingSagaStore中。CachingSagaStore的构造函数接受三个参数:分别是，包装的存储库、用于关联值的缓存和saga的实例。后两个参数可以引用相同或不同的缓存。这取决于你的具体的程序的逐出需求。

作者：勇赴
链接：https://www.jianshu.com/p/57bd121412fa
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。
