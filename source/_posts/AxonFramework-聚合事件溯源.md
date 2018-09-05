---
title: AxonFramework聚合事件溯源
date: 2018-08-27 13:56:57
categories: AxonFramework
tags: [AxonFramework]
author: 勇赴
---

除了存储一个聚合的当前状态，还可以根据过去发布的事件恢复一个聚合的状态。为此，所有状态的更改必须由一个事件来表示。

<!-- more -->

主要部分，事件溯源聚合类似于“有规律”聚合：他们必须声明一个标识符并可以使用apply函数发布事件。然而，事件溯源聚合中状态的变化(也就字段的任何变化)必须在@EventSourcingHandler注解的方法中被排他地执行。这包括设置聚合标识符。

注意,聚合标识必须被设置在聚合发布的第一个事件的@EventSourcingHandler上，这通常是创建事件。

事件溯源聚合的聚合根还必须包含一个无参的构造函数，Axon Framework使用这个构造函数创建一个空的聚合实例，在使用过去的事件之前初始化它。没有提供这种构造函数加载聚合时将导致异常。

<pre>
public class MyAggregateRoot {
    @AggregateIdentifier
    private String aggregateIdentifier;
    // fields containing state...
    @CommandHandler
    public MyAggregateRoot(CreateMyAggregate cmd) {
        apply(new MyAggregateCreatedEvent(cmd.getId()));
    }
    // constructor needed for reconstruction
    protected MyAggregateRoot() {
    }
    @EventSourcingHandler
    private void handleMyAggregateCreatedEvent(MyAggregateCreatedEvent event) {
        // make sure identifier is always initialized properly
        this.aggregateIdentifier = event.getMyAggregateIdentifier();
        // ... update state
    }
}
</pre>

带@EventSourcingHandler注解的方法使用特定的规则来解析。这些规则对于带@EventHandler注解的方法也同样适合，并在Defining Event Handlers这一章节中彻底解释。

><b>注意</b>
事件处理器(Event handler)的方法可以是私有的，只要JVM的安全设置允许Axon Framework改变方法的可访问性。这使你能够清楚地分离聚合的公共API，该方法从处理事件的内部逻辑中公开生成事件的方法。大多数的IDE有一个选项来忽略“未使用的私有方法”的警告为特定注解的方法。或者,你可以添加一个@SuppressWarnings(UnusedDeclaration)注解到方法,以确保你不意外地删除事件处理器方法。

有时候，特别是当聚合结构的增长远远超过两个实体时，对同一聚合的其他实体中事件发布的影响更明显。然而，由于重构聚合状态时事件处理器方法也会被调用，所以必须采取特殊的预防措施。

在事件溯源处理器方法内可以apply()新事件。这使得实体B可以apply一个事件来响应实体A做的一些事情成为可能。Axond重演历史事件时将忽略apply()调用。注意,在这种情况下，内部apply()调用事件只发布给实体，在所有实体收到第一个事件后。如果有更多的事件需要公布，在apply一个内部事件后的实体状态的基础上，可以用apply(...).andThenApply(...)

你还可以使用静态AggregateLifecycle.isLive()方法来检查聚合是否“存活”。基本上,一个聚合如果已经完成重演历史事件会被认为是存活的。当重演这些事件时，isLive()将返回false。在处理新生成的事件时，使用islive()方法是唯一可以执行的活动。

复杂的聚合结构

复杂的业务逻辑通常需要多个带聚合根的聚合来提供支持。在这种情况下，重要的是把复杂性分布在聚合内的多个实体中。当使用事件溯源时，不仅聚合根需要使用事件来触发状态转换，而且聚合内的实体也如此。

><b>注意</b>
一个常见的误解，聚合的实体不应该包含任何属性访问方法来暴露状态。这并非如此。事实上，在同一聚合内的实体向其他的实体暴露状态，可能会使一个聚合受益很多。然而，建议不要向外部暴露聚合的状态。

Axon在复杂的聚合结构中提供了对事件溯源的支持。实体，就像聚合根，简单的对象。子实体的字段声明必须使用@AggregateMember来注解。这个注释告诉Axon被注解的字段，包含一个应该对命令和事件处理程序进行检查的类。

当一个实体(包括聚合根)应用一个事件时，它首先是由聚合根处理，然后向下通过所有带@AggregateMember注解的字段到其子实体。

可能包含子实体的字段必须用@AggregateMember注解，此注释可用于多种字段类型：

字段直接引用的实体类型;
内部包含一个迭代器字段(包括所有集合,例如,列表,等等);
内部包含java.util.Map字段的值。

作者：勇赴
链接：https://www.jianshu.com/p/7b88c5077006
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。
