---
title: AxonFramework-事件向上转换
date: 2018-08-31 11:03:17
categories: AxonFramework
tags: [AxonFramework]
author: 勇赴
---

由于软件应用程序的不断变化的性质，很可能事件定义也随着时间而变化。由于事件存储被认为是只读和只追加（没有修改和删除）的数据源，所以应用程序必须能够读取所有事件，而不管它们何时添加。这时upcasting 出现了。

<!-- more -->

最初是面向对象编程的一个概念，“一个子类在需要的时候自动转换成超类”，upcasting 概念也可以应用于事件溯源。upcast一个事件意味着将它从原来的结构转换成新的结构。不像OOP的upcasting，事件的upcasting无法全部自动化完成，因为旧的事件对新事件的结构是不了解的。手工编写的Upcasters，必须提供指定如何将旧的结构upcast成新结构。

Upcasters类，获取一个x版本的输入事件，并且输出为零或更多版本x+1的新事件。此外，upcasters在一个链中被处理，这意味着一个upcaster的输出发送到下一个upcaster的输入。这允许你以增量的方式更新事件，为每一个新事件版次编写一个Upcaster ，使其小、隔离、并且容易理解。

><b>注意</b>
也许upcasting最大的好处是，它允许你做非破坏性重构，即完整的事件历史仍然保持不变。

在本节中，我们将解释如何编写upcaster，描述随着Axon不同的的Upcaster实现，并解释事件的序列化形式如何影响写upcasters。

允许upcaster看到什么版本的序列化对象被接收，Event Store存储版本号以及事件的完全限定名称。这个版本号是由RevisionResolver生成,在序列化器中配置。Axon提供了几个RevisionResolver的实现，比如AnnotationRevisionResolver，它检查在事件有效负载上的@Revision注解，SerialVersionUIDRevisionResolver 使用Java Serialization API和FixedValueRevisionResolver所定义的serialVersionUID，它总是返回一个预定义的值。后者在注入当前应用程序版本时是有用的。这将允许你看哪个版本的应用程序生成一个特定的事件。

Maven用户可以使用MavenArtifactRevisionResolver自动使用项目的版本。它使用项目获取的groupId，artifactId版本初始化。因为这只适用由Maven创建的JAR文件,版本不总能通过IDE来解析。如果无法解析版本，则返回null。

## 编写一个upcaster

事件的老版本:

<pre>
@Revision("1.0")
public class ComplaintEvent {
    private String id;
    private String companyName;

    // Constructor, getter, setter...
}
</pre>

新版本的事件:

<pre>
@Revision("2.0")
public class ComplaintEvent {
    private String id;
    private String companyName;
    private String complain; // New field

    // Constructor, getter, setter...
}
</pre>

Upcaster:

<pre>
// Upcaster from 1.0 revision to 2.0 revision
public class ComplaintEventUpcaster extends SingleEventUpcaster {
    private static SimpleSerializedType targetType = new SimpleSerializedType(ComplainEvent.class.getTypeName(), "1.0");

    @Override
    protected boolean canUpcast(IntermediateEventRepresentation intermediateRepresentation) {
        return intermediateRepresentation.getType().equals(targetType);
    }

    @Override
    protected IntermediateEventRepresentation doUpcast(IntermediateEventRepresentation intermediateRepresentation) {
        return intermediateRepresentation.upcastPayload(
                new SimpleSerializedType(targetType.getName(), "2.0"),
                org.dom4j.Document.class,
                document -> {
                    document.getRootElement().addElement("complaint");
                    document.getRootElement().element("complaint").setText("no complaint description"); // Default value
                    return document;
                }
        );
    }
}
</pre>

Spring boot configuration:

<pre>
@Configuration
public class AxonConfiguration {

    @Bean
    public SingleEventUpcaster myUpcaster() {
        return new ComplaintEventUpcaster();
    }

    @Bean
    public JpaEventStorageEngine eventStorageEngine(Serializer serializer,
                                                    DataSource dataSource,
                                                    SingleEventUpcaster myUpcaster,
                                                    EntityManagerProvider entityManagerProvider,
                                                    TransactionManager transactionManager) throws SQLException {
        return new JpaEventStorageEngine(serializer,
                myUpcaster::upcast,
                dataSource,
                entityManagerProvider,
                transactionManager);
    }
}
</pre>

TODO - Describe

* Upcasters工作在中间的表现层
* 它们更新流到流
* 抽象一对一的upcasting实现
* 代码示例

## Content type conversion（内容类型转换）
一个upcaster工作在给定内容类型上(如dom4j文档)。upcasters之间提供额外的灵活性，内容类型在链接的upcasters之间可能会有所不同。Axon将尝试使用ContentTypeConverters在内容类型之间自动地转换。它将寻找从类型x到类型y最短的路径，执行转换并交值转换成请求的upcaster。考虑到性能因素 ，如果receiving upcaster上的canUpcast方法产生true，转换才会被执行

ContentTypeConverters可能依赖于使用的序列化器类型。试图把一个byte[]转换成dom4j文档，这没有任何意义，除非使用序列化器把事件作为XML来写。确保UpcasterChain有权访问serializer-specific ContentTypeConverters,你可以通过UpcasterChain的构造函数引用序列化器。

><b>提示</b>
为了达到最佳性能,确保所有upcasters在同一链上(其中一个的输出是另一个的输入)处理相同的内容类型。

如果你需要不是由Axon提供的内容类型转换，你可以使用ContentTypeConverter接口编写一个自己的。
XStreamSerializer支持Dom4J像支持 XOM一样作为XML文档表示。JacksonSerializer 支持Jackson的JsonNode。

作者：勇赴
链接：https://www.jianshu.com/p/e4e943937993
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。