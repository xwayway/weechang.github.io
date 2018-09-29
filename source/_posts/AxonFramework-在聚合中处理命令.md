---
title: AxonFramework在聚合中处理命令
date: 2018-08-27 14:04:52
categories: AxonFramework
tags: [AxonFramework]
author: 勇赴
---

建议在包含处理状态命令的聚合中直接定义命令处理器，因为命令处理器有可能需要该集合的状态来执行其任务。

<!-- more -->

要在一个聚合上定义一个命令处理器,只需用@CommandHandler注解命令处理方法即可。带@CommandHandler注解方法的规则和其他处理方法都是一样的。然而,命令不仅通过他们的有效载荷(payload)进行路由。命令消息携带一个名字,该名称默认为命令对象的完全限定类名。

默认情况下,带@CommandHandler注解的方法允许以下参数类型:

* 第一个参数是命令消息的有效载荷。它的类型也可能是Message或CommandMessage，如果@CommandHandler 注解明确定义命令处理器的名称。默认情况下，命令名是命令的有效载荷的完全限定类名。

* 用@MetaDataValue注解的参数，将用注解上的键对元数据值进行解析。如果需要为false（默认值），则在元数据值不存在时传递NULL。如果需要为True，在元数据值不存在时，该解析器将不匹配并阻止该方法被调用。

* 参数的类型元数据将注入整个CommandMessage的元数据。

* UnitOfWork类型的参数获取当前工作单元注入。这允许命令处理器注册的行为在工作单元的特定阶段执行，或获得与它注册的资源的访问。

* Message或CommandMessage类型的参数，将得到完整的消息，包括有效载荷和元数据。如果方法需要多个元数据字段或包装消息的其他属性，则此方法非常有用。

为了使Axon知道哪一个聚合类型的实例应该处理命令消息，命令对象的属性传送聚合标识符，必须用@TargetAggregateIdentifier注解。注解可以放置在任何字段或访问器方法上（例如getter）。

创建聚合实例的命令不需要标识目标聚合标识符，虽然建议标注聚合标识符。

如果你喜欢使用另一个机制路由命令，这种行为可以通过提供一个自定义CommandTargetResolver来重写。这个类应该返回聚合标识符和预期的版本(如果有的话)基于给定的命令。

><b>注意</b>
当@CommandHandler注解放在一个聚合的构造函数上时，相应的命令将创建一个新的聚合实例，并将它添加到存储库。这些命令不需要针对特的定聚合实例。因此，这些命令不需要任何@TargetAggregateIdentifier或@TargetAggregateVersion注解，也不会调用自定义CommandTargetResolver。
当一个命令创建一个聚合实例时，该命令的回调函数在命令执行成功执行后，将得到聚合标识符。

```
public class MyAggregate {
    @AggregateIdentifier
    private String id;
    @CommandHandler
    public MyAggregate(CreateMyAggregateCommand command) {
        apply(new MyAggregateCreatedEvent(IdentifierFactory.getInstance().generateIden
tifier()));
    }
    // no-arg constructor for Axon
    MyAggregate() {
    }
    @CommandHandler
    public void doSomething(DoSomethingCommand command) {
    // do something...
    }
    // code omitted for brevity. The event handler for MyAggregateCreatedEvent must set the id field
}
public class DoSomethingCommand {
    @TargetAggregateIdentifier
    private String aggregateId;
    // code omitted for brevity
}
```

Axon的配置API可用于配置聚合。例如:

```
Configurer configurer = ...
// to use defaults:
configurer.configureAggreate(MyAggregate.class);
// allowing customizations:
configurer.configureAggregate(
AggregateConfigurer.defaultConfiguration(MyAggregate.class)
.configureCommandTargetResolver(c -> new CustomCommandTargetResolver()));
```

@CommandHandler注释并不局限于聚合根。把所有命令处理器放在根里，有时会导致聚合根中存在大量的方法,而它们中的许多只简单地调用转发给底层实体之一。如果是这样,你可以把@CommandHandler注解在一个底层的实体的方法上。Axon找到这些带注释的方法,聚合根中声明的实体字段必须用@AggregateMember标明。注意,命令处理器只检查带注解的字段的声明类型。如果一个字段值为空时传入命令到实体,就会抛出一个异常。

```
public class MyAggregate {
    @AggregateIdentifier
    private String id;
    @AggregateMember
    private MyEntity entity;
    @CommandHandler
    public MyAggregate(CreateMyAggregateCommand command) {
        apply(new MyAggregateCreatedEvent(...);
    }
    // no-arg constructor for Axon
    MyAggregate() {
    }
    @CommandHandler
    public void doSomething(DoSomethingCommand command) {
        // do something...
    }
    // code omitted for brevity. The event handler for MyAggregateCreatedEvent must set the id field
    // and somewhere in the lifecycle, a value for "entity" must be assigned to be able to accept
    // DoSomethingInEntityCommand commands.
}
public class MyEntity {
    @CommandHandler
    public void handleSomeCommand(DoSomethingInEntityCommand command) {
        // do something
    }
}
```

请注意，在聚合中每个命令必须只对应一个处理器。这意味着你不能用@CommandHandler标注多个实体(either root nor not，包含是根和不是根的所有实体)来处理相同的命令类型。如果你需要有条件地路由命令到一个实体,这些实体的父类应该处理命令,并根据apply的条件转发该命令。

字段的运行时类型不需要精确地声明类型。然而，@CommandHandle方法只检查被@AggregateMember标记的字段的声明类型。

也可以用@AggregateMember去注释包含实体的集合和Map。在后一种情况下，map的值有望包含实体，而键包含一个用作它们引用的值。

作为一个命令需要被路由到正确的实例，这些实例必须被正确地标识。它们的“ID”字段必须用@ EntityId标记。命令的属性将用于查找该消息应被路由到的实体，默认为被标识的字段的名称。例如，当标记一个名为“myentityid”字段，命令必须具有相同名称的属性。这意味着必须提供个getmyentityid或myentityid()方法。如果字段的名称和路由属性不同，你可以提供一个值显式使用 @EntityId(routingKey = "customRoutingProperty")。

如果在带注解的集合和Map中没有实体能被找到，Axon会抛出一个IllegalStateException异常。显然,聚合不能够在那个时间点上处理命令。

><b>注意</b>
字段声明的集合或Map应该包含适当的泛型,允许Axon识别实体的类型包含在集合或Map中。如果不可能添加泛型在声明中(例如因为你已经使用了一个自定义泛型类型的实现),你必须指定实体的类型，用于entityType属性@AggregateMember注解。

## 外部命令处理器
在某些情况下,想要直接向一个聚合实例路由命令是不可能。在这种情况下,可以注册一个命令处理器对象。命令处理器对象是一个简单的(常规的)对象，是带@CommandHandle注解的方法。与集合的情况不同，命令处理器对象只有单个实例，该对象处理其方法中声明的所有命令类型。

```
public class MyAnnotatedHandler {
@CommandHandler
public void handleSomeCommand(SomeCommand command, @MetaDataValue("userId") String
userId) {
// whatever logic here
}
@CommandHandler(commandName = "myCustomCommand")
public void handleCustomCommand(SomeCommand command) {
// handling logic here
}
}
// To register the annotated handlers to the command bus:
Configurer configurer = ...
configurer.registerCommandHandler(c -> new MyAnnotatedHandler());
```

作者：勇赴
链接：https://www.jianshu.com/p/64ddd5a8f517
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。