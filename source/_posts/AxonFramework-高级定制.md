---
title: AxonFramework-高级定制
date: 2018-08-31 11:14:03
categories: AxonFramework
tags: [AxonFramework]
author: 勇赴
---

## 参数解析器
你可以配置额外的ParameterResolvers，通过扩展ParameterResolverFactory类和创建一个名为/META-INF/service/org.axonframework.common.annotation.ParameterResolverFactory的文件，包含实现类的完全限定名称。

<!-- more -->

><b>警告</b>
此时，OSGi支持仅限于在清单文件中被提到的所需的头这一事实。ParameterResolverFactory实例的自动检测在OSGi上工作，但由于类加载器的局限性，它可能需要复制/META-INF/service/org.axonframework.common.annotation.ParameterResolverFactory 文件的内容到OSGi包，包含用于解析参数的类（即事件处理程序）。

## Meta Annotations
TODO

## Customizing Message Handler behavior
TODO

## 性能调优
待办事项:更新Axon3
本章包含一个清单和在为生产级性能做准备时需要考虑的一些指导方针。现在，你可能已经使用了测试固件测试你的命令处理逻辑和sagas。然而，生产环境不像测试环境那么宽容。聚合往往存活得更久，更频繁地和并发的使用。对于额外的性能和稳定性，你最好调整配置满足你的具体需求。

## 数据库索引和列类型
## SQL DatabasesSQL数据库
如果你使用你的JPA实现自动生成表(例如Hibernate)，你可能没有把所有正确的索引设置在你的表上。为获得最佳性能，事件存储的不同用法需要不同的索引设置。该列表显示，为默认EventStorageEngine实现使用的不同类型的查询添加不同类型的索引：

* 标准操作使用(存储和加载事件):
Table 'DomainEventEntry', columns aggregateIdentifier and sequenceNumber (unique index)
Table 'DomainEventEntry', eventIdentifier (unique index)

* 快照：
Table 'SnapshotEventEntry', aggregateIdentifier column.
Table 'SnapshotEventEntry', eventIdentifier (unique index)

* Sagas
Table 'AssociationValueEntry', columns associationKey and sagaId,

默认生成的列长度可以工作，例如Hibernate，但不会是最优的。例如，一个UUID总是有相同的长度。而不是可变长度列255个字符，你可以为聚合标识符使用一个固定长度36个字符的列。

“时间戳”列在DomainEventEntry表只储存ISO 8601时间戳。如果所有时间存储在UTC时区，他们需要一个长度为24个字符的列。如果你使用另一个时区，这可能高达28位。使用可变长度列通常是没有必要的，因为时间戳总是具有相同的长度。

><b>警告</b>
强烈建议所有时间戳以UTC格式存储。在夏令时的国家，用当地时间存储时间戳，在时区转换时可能会导致事件生成的顺序错误。使用UTC时不会发生这种情况。有些服务器配置为始终使用UTC。另外你应该在存储它们之前配置事件存储将时间戳转换成UTC。

在DomainEventEntry中的“type”列存储聚合的标识符类型。一般来说，这些都是聚合的简单的名称。在Spring中事件臭名昭著的“AbstractDependencyInjectionSpringContextTests”只算45个字符。在这里，再一次，一个长度较短(但可变)的字段应该足够了。

## MongoDB
默认情况下，MongoEventStore只会为正确的操作生成它需要的索引。这意味着当事件存储被创建时，所需的惟一索引在“聚合标识符”、“聚合类型”和“事件序列号”上也被创建。然而，当为某些操作使用MongoEventStore时，可能是值得添加一些额外索引的。

注意，在查询优化和更新速度之间的总有一个平衡点。负载测试最终是发现哪些索引提供最佳性能的最好方法。

* 正常操作使用：
在“aggregateIdentifier”上自动创建一个索引，“type”和“sequenceNumber”在领域事件(缺省名称:“domainevents”)集合中。

* Snapshotting：
在事件快照(缺省名称:“snapshotevents”)集合中把(unique)索引放“aggregateIdentifier”,“type”和“sequenceNumber“上。

* Replaying events:
在领域事件(缺省名称:“domainevents”)集合中，把非唯一索引放在“timestamp”和“sequenceNumber”上。
* Sagas：
在saga（默认名称：“sagas”）集合中把(唯一)索引放到“sagaIdentifier”上。
在saga（默认名称：“sagas”）集合中把索引放到”sagaType”、“associations.key”和“associations.value“属性上。

## Caching
一个设计良好的命令处理模块当实现缓存时应该不会构成任何问题。尤其是当使用事件溯源时，从事件存储中加载一个聚合是一项昂贵的操作。用正确配置的缓存，加载一个聚合可以转化为一个纯粹的in-memory过程。

下面是一些帮助您最大限度地利用缓存解决方案的指导原则：

* 确保工作单元永远不需要为功能原因执行回滚。
回滚意味着一个聚合已经达到了一个无效的状态。Axon会自动将相关的缓存项失效。下一个请求将迫使聚合从事件中重建。如果你使用异常作为一个潜在的(功能的)返回值，你可以在命令总线上配置一个RollbackConfiguration。默认情况下，当运行时异常时这个工作单元将回滚。

* 单个聚合的所有命令必须到达在缓存中具有该聚合的机器上。
这意味着命令应该始终被路由到同一台机器，只要这台机器是“健康”的。路由命令总是阻止缓存过期。命中一个过期缓存将导致一个命令被执行，并且事件存储在事件存储中会失败。

* 配置一个合理的生存时间/闲置时间
默认情况下，缓存倾向于有一个相对较短的生存时间，即几分钟。对于具有一致的路由的命令处理组件，一个较长的闲置时间和生存时间通常是更好的。这可以防止需要初始化一个基于它的事件的聚合，仅仅因为它的缓存条目过期了。缓存的生存时间应该与你的聚合的预期寿命相匹配。

## Snapshotting
快照删除需要重载和重放大量的事件。单个快照代表在在某一特定时刻整个聚合状态。然而，快照的处理本身也需要处理时间。因此，在构建快照时所花费的时间和阻止许多事件被读取节省的时间应该保持平衡。

对所有类型的应用程序都没有默认行为。一些将指定一些事件之后将创建一个快照，而其他应用程序需要一个基于时间的快照间隔。无论你选择何种方式为您的应用程序，如果你有long-living聚合，确保快照就位。

作者：勇赴
链接：https://www.jianshu.com/p/1dc158d81511
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。