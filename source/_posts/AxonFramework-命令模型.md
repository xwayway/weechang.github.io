---
title: AxonFramework命令模型
date: 2018-08-21 14:00:06
categories: AxonFramework
tags: [AxonFramework]
author: 勇赴
---

在一个基于CQRS的应用程序中，领域模型(由Eric Evans和Martin Fowler定义)可以是一个非常强大的机制，来驾驭在验证和执行状态的变化时所涉及的复杂性，虽然典型的领域模型提供了大量的构建块，但当在CQRS中应用命令处理时有一个构件起着主导性的作用：聚合。

<!-- more -->

应用程序中的一个状态的改变，始于一个命令。命令不但是表达意图(描述你想要做什么)的组合，而且是基于这一意图采取行动所需的信息。命令模型用于处理传入的命令，以验证并定义它的结果。在这个模型中，一个命令处理器负责处理某种类型的命令，并根据它包含的信息采取行动。

## 聚合
聚合是一个总是保持一致性状态的实体或一组实体。聚合根是聚合树顶部负责维护这个一致性状态的对象。这使得聚合主要构件在任何基于CQRS应用中实现命令模型。

><b>注意</b>
“聚合”指的是埃文斯在领域驱动设计中所定义的聚合:
”作为数据变更的一个单元来处理的一组相关联的对象。在外部只能引用聚合的聚合根对象。在聚合边界内使用一组一致性规则。”

例如，“联系人”聚合可以包含两个实体：联系人和地址。若要保持整个聚合处于一致性状态，向联系人添加地址时应通过联系人实体完成。在这种情况下，联系人实体是约定的聚合根。

在Axon中，聚合由聚合标识符标识。这可能是任何对象，但也有一些标识符良好实现的指导原则。
标识符必须:

实现equals和hashCode来保证与其他实例进行相等比较，
实现一个提供一致结果的toString()方法(相等的标识符，toString()方法的结果也应该相等),
并且最好是可序列化的。
当聚合使用不兼容标识符时，测试固件（见测试）将验证这些条件并使测试失败。 字符串类型的标识符，UUID和数值类型都适用。 不要使用原始类型作为标识符，因为它们不允许进行延迟初始化。在某些情况下，Axon可能会错误地假设原始类型的默认值是标识符的值。

注意
一个经过深思熟虑的好实践是使用随机生成的标识符，而不是使用序列。使用序列会大大降低应用程序的可伸缩性，因为机器需要保持彼此最后一次使用的最新的序列号。UUID冲突的机率非常地小(假如你生成8.2乘以10的11次方个UUID，冲突的机率也只有10的负15次方)。

此外，聚合应小心地使用函数式标识符。因为它们有变化的倾向，使得它很难适应相应的应用程序。

## 聚合的实现
聚合总是通过一个称为聚合根的实体访问。通常，这个实体的名称和聚合完全一样。例如,Order聚合可能由Order实体引用几个Orderline实体组合而成。Order 和Orderline一起形成聚合。

一个聚合是一个合乎规范的对象,其中包含状态和和改变这个状态的方法。虽然根据CQRS原则并不完全正确，也可能通过访问器方法暴露出聚合的状态。

聚合根必须声明一个包含聚合标识符的字段。这个标识符字段必须最迟在第一个事件发布时初始化。这个标识符字段必须由@AggregateIdentifier来注解。如果你在聚合上使用JPA注解，Axon也可以使用由JPA提供的@Id注解。

聚合可以使用AggregateLifecycle.apply()方法来注册发布的事件。与EventBus不同，这些信息需要被包装在一个EventMessage中，apply()允许你直接传递负载对象。

```
@Entity // Mark this aggregate as a JPA Entity
public class MyAggregate {

    @Id // When annotating with JPA @Id, the @AggregateIdentifier annotation is not necessary
    private String id;
    
    // fields containing state...
    
    @CommandHandler
    public MyAggregate(CreateMyAggregateCommand command) {
        // ... update state
        apply(new MyAggregateCreatedEvent(...));
    }
    
    // constructor needed by JPA
    protected MyAggregate() {
    }
}
```

通过定义一个带@EventHandler注解的方法，聚合内的实体能监听聚合发布的事件。当一个EventMessage发布时这些方法将被调用（在任何外部处理器被发布之前）。

作者：勇赴
链接：https://www.jianshu.com/p/6cfc9ab399b2
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。
