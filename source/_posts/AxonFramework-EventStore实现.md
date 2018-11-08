---
title: AxonFramework-EventStore实现
date: 2018-08-31 10:52:45
categories: AxonFramework
tags: [AxonFramework]
author: 勇赴
---

事件可跟踪性存储库需要事件存储（事件存储）来存储和加载来自聚合的事件。事件存储提供事件总线的功能，它增加了持久发布事件和基于聚合标识符检索事件的功能。

<!-- more -->

Axon提供了一个开箱即用的事件存储，EmbeddedEventStore。它委托事件实际的存储和检索到EventStorageEngine。
有多个EventStorageEngine的实现：

## JpaEventStorageEngine

JpaEventStorageEngine将事件存储在与JPA兼容的数据源中。 JPA事件在所谓的条目中存储和存储事件。这些条目包含序列化的事件形式，并存储一些域元数据以快速查找这些条目。使用JpaEventStorageEngine，您必须在类路径中使用JPA注释（javax.persistence）。

默认情况下，事件存储需要你配置你的持久化上下文(如在META-INF/persistence.xml中定义)包含DomainEventEntry和SnapshotEventEntry（两者都在org.axonframework.eventsourcing.eventstore.jpa包中)。
下面是一个持久化上下文配置的示例配置：

```
<persistence xmlns="http://java.sun.com/xml/ns/persistence" version="1.0">
    <persistence-unit name="eventStore" transaction-type="RESOURCE_LOCAL"> (1)
        <class>org...eventstore.jpa.DomainEventEntry</class> (2)
        <class>org...eventstore.jpa.SnapshotEventEntry</class>
    </persistence-unit>
</persistence>
```

在这个示例中，事件存储有一个特定的持久化单元。然而，你可能会选择将第三行添加到任何其他持久化单元的配置中。
本行注册DomainEventEntry(由JpaEventStore使用的类)到持久化上下文。

><b>注意</b>
Axon使用锁来防止两个线程访问相同的聚合。但是，如果同一数据库上有多个JVM，这对您没有帮助。在这种情况下，您必须依靠数据库来检测冲突。对事件存储的并发访问将导致违反键约束违例，因为表允许聚合只有一个具有任何序列号的事件，因此为具有现有序列号的现有聚合插入第二个事件将导致错误。
JpaEventStorageEngine可以检测到此错误并将其转换为ConcurrencyException。但是，每个数据库系统都以不同的方式报告此违规。如果您使用JpaEventStore注册数据源，它将尝试检测数据库的类型，并发现错误代码是一个关键的违反条件违规。或者，您可以提供一个Persistence Exception Translator实例，如果给定的异常表示违反了Key Constraint Violation，则可以对该实例进行区分。
如果没有提供数据源或PersistenceExceptionTranslator，从数据库驱动程序按原样抛出异常。

默认情况下，JPA事件存储引擎需要EntityManagerProvider实现，该实现返回EventStorageEngine使用的EntityManager实例。这还允许应用程序管理使用的持久性上下文。 EntityManagerProvider的职责是提供EntityManager的正确示例。

有几个EntityManagerProvider实现可用，每个实现都有不同的需求。 SimpleEntityManagerProvider仅在构建时才向其返回EntityManager实例。这使得它成为容器管理上下文的简单选择。 Container Managed Entity Manager Provider作为选项返回默认持久性上下文，默认情况下通过JPA事件存储其使用。
如果你有一个名为"myPersistenceUnit"的持久化单元，你想在JpaEventStore中使用它，那么这就是EntityManagerProvider实现，它可能看起来像：

```
public class MyEntityManagerProvider implements EntityManagerProvider {

    private EntityManager entityManager;

    @Override
    public EntityManager getEntityManager() {
        return entityManager;
    }

    @PersistenceContext(unitName = "myPersistenceUnit")
    public void setEntityManager(EntityManager entityManager) {
        this.entityManager = entityManager;
    }
}
```
 
默认情况下，JPA事件存储把条目存储在DomainEventEntry和SnapshotEventEntry实体中。虽然在许多情况下这就足够了，你可能会遇到这些实体提供的元数据不够的情况。或者你可能想将不同的聚合类型的事件存储在不同的表。
如果是这样，您可以展开JpaEventStorageEngine。它包含一些受保护的方法，您可以重写它们以调整其行为。

><b>警告</b>
注意持久化提供者，如Hibernate，在它们的EntityManager实现上使用一级缓存。通常，这意味着在查询中使用或返回的所有隶属于EntityManager的实体。它们只有在周围事务被提交或在事务中执行显式“清除”时才被清除。当查询在事务上下文中执行时尤其如此。
要解决这个问题，请确保仅仅查询非实体对象。你可以使用JPA的“SELECT new SomeClass(parameters) FROM ...”风格的查询来解决这个问题。或者，获取一批事件后调用EntityManager.flush()和EntityManager.clear()。未能这样做当加截大事件流时可能导致OutOfMemoryExceptions。

## JDBC Event Storage Engine

JDBC事件存储引擎使用JDBC连接将事件存储在JDBC兼容的数据存储。通常，这些都是关系数据库。从理论上讲，任何一个JDBC驱动程序都可以用来支持JDBC事件存储引擎。

类似于JPA，JDBC事件存储引擎将事件存储在条目中。默认情况下，每个事件存储在一个单独的条目中，对应表中的一行。一个表用于事件，另一个用于快照。

JdbcEventStorageEngine使用ConnectionProvider来获取连接。通常，这些连接可以直接从数据源中获得。然而，Axon会将这些连接绑定到一个工作单元，以便在一个工作单元使用一个连接。这将确保一个单独的事务用于存储所有事件，即使在同一线程中嵌套多个工作单元。

><b>注意</b>
Spring用户建议使用SpringDataSourceConnectionProvider从数据源附加连接到现有的事务。

## MongoDB Event Storage Engine

MongoDB是一个基于文档的NoSQL存储。其可伸缩特性使它适合用于事件存储。Axon提供了MongoEventStorageEngine,使用MongoDB作为数据库支持。它包含在Axon Mongo模块(Maven artifactId axon-mongo)。

事件存储在两个独立的集合中：一个用于实际的事件流，一个用于快照。

默认情况下，MongoEventStorageEngine将存储各自的事件在各自的文档。然而，它是可能改变StorageStrategy使用。

Axon提供的选择是DocumentPerCommitStorageStrategy，为在一个单独的提交中存储所有事件，创建一个单独的文档(即在同一DomainEventStream)。

在一个单独的文档中存储整个提交的好处在于提交是原子存储的。此外，它只需要对任意数量的事件进行一次往返。缺点是,它变得更加难以直接在数据库中查询事件。例如，当重构领域模型时，如果他们被包含在“commit document”中，很难从一个聚合“transfer”事件到另一个聚合。

MongoDB不需要很多配置。它所需要的只是对一个存储事件集合的引用，然后你就可以开始了。在生产环境中，你可能想要对集合中的索引进行双重检查。

## Event Store Utilities

Axon提供了一些事件存储引擎，可能在某些情况下是有用的。

SequenceEventStorageEngine是围绕其它两个事件存储引擎的包装器。当读取时,它从这两个事件存储引擎中返回事件。附加事件只是附加到第二个事件存储引擎。出于性能原因使用事件存储两种不同的实现的情况下，这是有用的，例如。第一个是一个更大的，但较慢事件存储，而第二个是优化的快速读取和写入。

还有一个常驻内存的存储事件EventStorageEngine实现：InMemoryEventStorageEngine。虽然它可能优于任何其他的事件存储，这并不意味着长期生产使用。然而，它在需要事件存储的short-lived工具或测试中非常有用,。

## Influencing the serialization process（影响序列化过程）

事件存储需要为存储准备一种序列化事件的方法。默认情况下，Axon使用XStreamSerializer，其使用XStream序列化成XML事件。XStream是相当快的，比Java序列化更灵活。此外，XStream序列化的结果是人类可读的。对日志和调试而言非常有用。

XStreamSerializer可以配置。你可以定义它应该用于某些包、类甚至字段的别名。除了可以缩短潜在的长名称之外，还可以在事件的类定义更改时使用别名。有关别名的更多信息，访问XStream网站。

另外，Axon还提供了JacksonSerializer，使用Jackson将事件序列化为JSON。当它生成一个更紧凑的序列化形式，它要求类遵守Jackson所要求的约定(或配置)。

><b>注意</b>
使用Java代码(或其他JVM语言)配置序列化器是很容易的。然而，由于它调用方法的局限性，在Spring XML程序上下文看配置它并不那么简单。其中一个选项是创建一个FactoryBean，创建一个XStreamSerializer实例，并配置它的代码。检查 Spring 参考更多的信息。

作者：勇赴
链接：https://www.jianshu.com/p/7ffff660d9a1
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。
