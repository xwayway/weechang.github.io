---
title: AxonFramework-事件序列化器调优
date: 2018-08-31 11:20:34
categories: AxonFramework
tags: [AxonFramework]
author: 勇赴
---

## XStream序列化器
XStream是完全可配置和可扩展的。如果你只使用一个无格式XStreamSerializer，有一些速效方案随时取用。
<!-- more -->

XStream允许你为包名称和事件类名称配置别名。别名通常更短(特别是如果你有长包名)，使事件的序列化形式更小。由于我们讨论的是XML，所以每个字符从XML中移除是两次(一个开始标记，一个结束标记)。

在XStream中一个更高级的话题创建自定义转换器。默认的基于反射的转换器是简单的，但不会生成最紧凑的XML。总是仔细观察生成的XML，看看是否真正需要重建原始实例的所有信息。

如果有可能避免upcasters的使用。XStream允许别名用于字段，当他们已经改变了名字。想象版本为0事件,使用一个名为“clientId”字段。业务更喜欢“客户”一词，所以版本1使用被称为“customerId”的字段创建。在XStream中这个使用字段别名完全可以配置。你需要配置两个别名，按照以下顺序：别名“customerId”到“clientId”然后别名“customerId”到“customerId”。这将告诉XStream，如果遇到一个叫做“customerId”字段，它将调用相应的XML元素“customerId”(第二个别名覆盖第一个)。但如果XStream遇到一个XML称为“clientId”的元素，这是一个已知的别名，将解析为字段名称“customerId”。查看XStream文档了解更多信息。

对于终极性能，没有基于反射机制的alltogether你可能更好。在这种情况下，创建一个自定义的序列化机制，这可能是最明智的。DataInputStream和DataOutputStream允许你容易将事件的内容写入输出流。ByteArrayOutputStream和ByteArrayInputStream允许写入和读取字节数组。

## 防止重复序列化
特别是在分布式系统中，事件消息需要在多个场合进行序列化。Axon的组件检测这个并支持SerializationAware消息。如果检测到SerializationAware消息，其方法用来序列化一个对象,而不是简单地传递载荷到序列化器。这允许对性能进行优化。

当你序列化你自己消息时，希望受益于SerializationAware优化，使用MessageSerializer类序列化消息的有效负载和元数据。所有优化逻辑是在这个类中实现。更多细节请参考MessageSerializer的JavaDoc。

## 自定义标识符生成器
AxonFramework使用IdentifierFactory生成所有的标识符，不论它们是事件或命令。默认情况下，IdentifierFactory随机生成基于java.util.UUID的标识符。尽管它们使用起来非常安全，但生成它们的过程的性能并出色。

IdentifierFactory是一个抽象工厂，使用Java的ServiceLoader(从Java 6)机制找到实现来使用。这意味着你可以创建自己的工厂的实现，将实现的名称放在一个叫做“/META-INF/services/org.axonframework.common.IdentifierFactory”文件中。Java的ServiceLoader机制将检测到文件并尝试创建名为inside类的实例。

IdentifierFactory有几个的需求。必须实现：

* 在类路径中让它的完全限定类名作为/META-INF/services/org.axonframework.common.IdentifierFactory文件的内容,
* 有一个可访问的无参数构造函数,
* 扩展IdentifierFactory,
* 通过应用程序的类加载器上下文或类加载器加载IdentifierFactory类来访问，并且必须的
* 是线程安全的。

作者：勇赴
链接：https://www.jianshu.com/p/801dade37318
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。