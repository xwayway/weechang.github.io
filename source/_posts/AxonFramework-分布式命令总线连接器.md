---
title: AxonFramework-分布式命令总线连接器
date: 2018-08-31 10:40:27
categories: AxonFramework
tags: [AxonFramework]
author: 勇赴
---

## JGroupsConnector
JGroupsConnector使用(正如它的名字已经说明)JGroups作为底层发现和调度机制。描述JGroups的特性有太多的参考指南,请参阅JGroups用户指南以了解更多的细节。

<!-- more -->

因为JGroups既处理节点的发现又处理它们之间的通信，所以JGroupsConnector既充当CommandBusConnector也充当CommandRouter。

><b>注意</b>
你可以在axon-distributed-commandbus-jgroups模块中，为DistributedCommandBus找到JGroups特定组件。

JGroupsConnector有四个强制性配置元素：

* 第一种是JChannel，它定义了JGroups协议栈。一般来说，用JGroups配置文件的引用构造JChannel。JGroups附带的默认配置,可以用作自己配置的依据。请记住，IP多路广播一般不工作在云服务中，像亚马逊。中这种类型的环境中， TCP Gossip通常是一个好的开端。
* 集群名称定义了每个segment应登记到的集群的名称。具有相同的集群名称的Segment最终会探测到彼此，并在彼此间分发命令。
* “本地segment”是命令总线实现，分发命令去往本地的JVM。这些命令可能已通过其他JVM或从本地的一个实例分发。
* 最后，序列化器是用来序列化之前通过线路发送的命令消息。

><b>注意</b>
当使用缓存时，当ConsistentHash更改以避免潜在的数据损坏时，它应该被清空（例如，当命令没有指定一个@TargetAggregateVersion和新成员快速加入和离开JGroup，修改聚合然而它还要缓存到其他地方）。

最终，JGroupsConnector需要实际连接，按顺序分发消息到其他segment。这样做，调用connect()方法。

<pre>
JChannel channel = new JChannel("path/to/channel/config.xml");
CommandBus localSegment = new SimpleCommandBus();
Serializer serializer = new XStreamSerializer();

JGroupsConnector connector = new JGroupsConnector(channel, "myCommandBus", localSegment, serializer);
DistributedCommandBus commandBus = new DistributedCommandBus(connector, connector);

// on one node:
commandBus.subscribe(CommandType.class.getName(), handler);
connector.connect();

// on another node, with more CPU:
commandBus.subscribe(CommandType.class.getName(), handler);
commandBus.subscribe(AnotherCommandType.class.getName(), handler2);
commandBus.updateLoadFactor(150); // defaults to 100
connector.connect();

// from now on, just deal with commandBus as if it is local...
</pre>

><b>注意</b>
注意，并非所有的segment都必需具有相同类型的命令的命令处理器。你完全可以为不同的命令类型使用不同的segment。分布式命令总线总是选择一个节点分发命令到那个支持特定类型的命令。

如果你使用Spring，你可能需要考虑使用JGroupsConnectorFactoryBean。它自动连接连接器当ApplicationContext启动后，并且在ApplicationContext关闭时完全的断开。此外,它为测试环境使用合理的默认值(但不应被视为生产准备)和自动装配配置。

## Spring Cloud Connector

Spring Cloud连接器装置，用Spring Cloud来描述使用服务注册和发现机制来分配命令总线。因此，你可以自由选择使用哪一个Spring Cloud实现用来分发你的命令。实现的一个例子是 Eureka Discovery/Eureka 服务器组合。

><b>注意</b>
当前版本(Axon 3.0.4)SpringCloudCommandRouter使用ServiceInstance。Metadata 字段来通知所有系统中的节点，通过CommandNameFilter它可以处理命令。这是很重要的,Spring Cloud实现选择支持ServiceInstance.Metadata字段的使用。例如Spring Cloud Consul目前不支持该字段，因此SpringCloudCommandRouter并不是一个可行的解决方案。我们正在研究一个额外的解决方案，从中检索CommandNameFilter 。

提供每个SpringCloud实现的描述将推动本参考指南。因此，我们参考他们各自的文件以获得进一步的信息。

Spring Cloud连接器装置是一个SpringCloudCommandRouter和SpringHttpCommandBusConnector的组合，分别填充CommandRouter的地点和 DistributedCommandBus的CommandBusConnector。

><b>注意</b>
Spring Cloud连接器特定的组件DistributedCommandBus可以在axon-distributed-commandbus-springcloud模块中找到 。

SpringCloudCommandRouter必须由以下提供的来创建：

* 一个DiscoveryClient类型“discovery client”。这可以通过用@EnableDiscoveryClient注解你的Spring Boot应用程序来提供,将在你的类路径中寻找Spring Cloud 的实现。
* 一个RoutingStrategy类型的"routing strategy"。目前axon-core模块提供了一些实现，但是一函数调用也可以满足要求。例如，如果你想路由命令基于“聚合标识符”，你可以使用AnnotationRoutingStrategy和注解有效载荷的字段，用@TargetAggregateIdentifier识别聚合。

SpringHttpCommandBusConnector需要创建三个参数:

* 一个CommandBus类型的“local command bus”。这是命令总线实现，它将分发命令到本地的JVM。这些命令可能是由其他JVM上的或本地的实例分发。
* RestOperations对象来执行一个命令消息的发布到另一个实例。
* 最后一个Serializer类型的“序列serializer”。序列化器用于在命令发送到网络之前序列化命令消息。

SpringCloudCommandRouter和SpringHttpCommandBusConnector应该都被用于创建DistributedCommandsBus。在Spring Java 配置中，看起来如下:

<pre>
// Simple Spring Boot App providing the `DiscoveryClient` bean
@EnableDiscoveryClient
@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

    // Example function providing a Spring Cloud Connector
    @Bean
    public CommandRouter springCloudCommandRouter(DiscoveryClient discoveryClient) {
        return new SpringCloudCommandRouter(discoveryClient, new AnnotationRoutingStrategy());
    }

    @Bean
    public CommandBusConnector springHttpCommandBusConnector(@Qualifier("localSegment") CommandBus localSegment,
                                                             RestOperations restOperations,
                                                             Serializer serializer) {
        return new SpringHttpCommandBusConnector(localSegment, restOperations, serializer);
    }

    @Primary // to make sure this CommandBus implementation is used for autowiring
    @Bean
    public DistributedCommandBus springCloudDistributedCommandBus(CommandRouter commandRouter, 
                                                                  CommandBusConnector commandBusConnector) {
        return new DistributedCommandBus(commandRouter, commandBusConnector);
    }

}

// if you don't use Spring Boot Autoconfiguration, you will need to explicitly define the local segment:
@Bean
@Qualifier("localSegment")
public CommandBus localSegment() {
    return new SimpleCommandBus();
}
</pre>

><b>注意</b>
注意，并非所有的segment都必需具有相同类型的命令的命令处理器。你完全可以为不同的命令类型使用不同的segment。分布式命令总线总是选择一个节点分发命令到那个支持特定类型的命令。

作者：勇赴
链接：https://www.jianshu.com/p/ccef9924f8d7
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。