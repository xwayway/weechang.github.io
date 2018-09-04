---
title: AxonFramework测试
date: 2018-08-29 10:30:35
categories: AxonFramework
tags: [AxonFramework]
author: 勇赴
---

CQRS最大的好处之一，尤其是事件溯源就事件和命令而言，单纯地表达测试是可能的。这两个功能组件，事件和命令对领域专家或业务所有者都有明确的含义。这不仅意味着测试表达就事件和命令而言有明确的功能含义，这也意味着他们不依靠任何实现选择。

<!-- more -->

本章描述的特性需要axon-test模块,可通过配置maven依赖(使用<artifactId>axon-test</artifactId> 和<scope>test</scope)或通过完整包下载。

本章中描述的固件可用于任何测试框架，如JUnit和TestNG。

## 命令组件测试

在任何CQRS基础架构中命令处理组件通常是最复杂的。比其他组件更复杂，这也意味着该组件有额外的与测试相关的需求。

虽然更复杂，但是命令的API处理组件相当容易。它有一个命令进来，然后事件出去。在某些情况下，可能有一个查询作为命令执行的一部分。除此之外，命令和事件是API的唯一部分。这意味着可以在事件和命令的基础上完整地定义测试场景。典型地，以：

* given过去的某些事件，
* when 执行这个命令，
* expect 这些事件将被发布和/或存储

Axon Framework提供了一个测试固件，使你能够做到这一点。AggregateTestFixture允许你配置某些基础设施，包括必要的命令处理器和存储库，并以given-when-then形式的事件和命令来表达你的场景。

下面的示例展示了用JUnit 4对given-when-then测试固件的使用：

<pre>
public class MyCommandComponentTest {
 private FixtureConfiguration fixture;
 @Before
 public void setUp() {
     fixture = new AggregateTestFixture(MyAggregate.class);
 }

 @Test
 public void testFirstFixture() {
     fixture.given(new MyEvent(1))
            .when(new TestCommand())
            .expectSuccessfulHandlerExecution()
            .expectEvents(new MyEvent(2));
     /*
     These four lines define the actual scenario and its expected
     result. The first line defines the events that happened in the
     past. These events define the state of the aggregate under test.
     In practical terms, these are the events that the event store
     returns when an aggregate is loaded. The second line defines the
     command that we wish to execute against our system. Finally, we
     have two more methods that define expected behavior. In the
     example, we use the recommended void return type. The last method
     defines that we expect a single event as result of the command
     execution.
     /*
 }
}
</pre>

given-when-then测试固件定义了三个阶段：配置、执行和验证。每个阶段由不同的接口表示：分别是，FixtureConfiguration, TestExecutor 和 ResultValidator。固件类的静态newGivenWhenThenFixture()方法提供了对第一个的引用，进而可能提供验证，等等。

><b>注意</b>
为了最好地利用这些阶段之间的迁移，最好使用这些方法提供的流式接口，如上面的示例所示。

在配置阶段（即在提供第一个“given”之前），你提供了执行测试所需的构件。作为固件的一部分提供事件总线、命令总线和事件存储的专用版本。有accessor方法来获得对它们的引用。任何命令处理器不直接在聚合上注册，需要显式地使用registerAnnotatedCommandHandler 方法配置。除了带注解的命令处理器外，你还可以配置各种组件和设置，定义应该如何设置测试周围的基础设施。

一旦固件配置好，你就可以定义“given”事件。测试固件将用DomainEventMessage包装这些事件。如果“given”事件实现消息，消息的有效负载和元数据将被纳入DomainEventMessage，否则given事件作为有效负载。DomainEventMessage 的序列号顺序，从0开始。

或者，你也可以为“given”场景提供命令。在这种情况下，在执行实际的测试命令时，这些命令生成的事件将被用于事件源聚合。使用“givenCommands(…)”方法提供命令对象。

执行阶段允许你提供一个针对命令处理组件执行的命令。对调用处理程序的行为(无论是在聚合或外部处理程序)进行监控，并与在验证阶段注册的预期进行比较。

><b>注意</b>
在执行测试过程中，Axon试图检测测试中的所有在聚合上的非法状态的更改。它通过将聚合的状态与命令执行后的聚合状态进行比较，如果它从所有“given”和存储的事件溯源。如果状态不相同，这意味着状态变化发生在聚合事件处理器方法之外。比较时将忽略静态和瞬态字段，因为它们通常包含对资源的引用。
可以使用setReportIllegalStateChange方法在固件的配置中切换检测。

最后一个阶段是验证阶段，允许你检查命令处理组件的活动。这完全是根据返回值和事件来完成的。

测试固件允许你验证命令处理程序的返回值。你可以显式地定义预期的返回值，或者简单地要求成功返回该方法。你也可以表达任何你期望的CommandHandler抛出的异常。

另一个组件是对已发布事件的验证。有两种匹配预期事件的方法。

第一是通过事件实例，它需要与实际的事件是行逐字的比较。将预期事件的所有属性与实际事件中的对应对象进行比较（使用equals()）。如果其中一个属性不相等，则测试失败，并生成一个广泛的错误报告。

表达期望的另一种方式是使用的匹配器(Hamcrest库提供的)。匹配器接口规定了两个方法matches(Object)和describeTo(Description)。第一个返回一个布尔值，指示是否匹配或不匹配。第二个让你表达你的期望。例如，一个“GreaterThanTwoMatcher”可以添加“任何值大于2的事件“的描述。描述允许创建关于测试用例失败的错误消息。

创建事件列表的匹配器可能是繁琐和容易出错的工作。为了简化问题，Axon提供了一组匹配器允许你提供一组特定于事件的匹配器，并告诉Axon应该如何匹配列表。

下面是可用的事件列表匹配器和他们的目的的概述：

* List with all of: Matchers.listWithAllOf(event matchers...)
如果所有的事件匹配器与真实事件列表中至少一个事件匹配，该匹配器将成功。不管是否有多个匹配器匹配相同的事件，或如果列表中一个事件不匹配任何匹配器。

* List with any of: Matchers.listWithAnyOf(event matchers...)
如果一个或多个事件匹配器与实际的事件列表中一个或多个事件匹配，该匹配器将成功。一些匹配器甚至一个也不匹配，而另一个匹配多个。

* Sequence of Events: Matchers.sequenceOf(event matchers...)
使用此匹配器来验证实际事件匹配器和提供的事件匹配器有相同的顺序。如果匹配器与后一个事件相匹配，与前一个匹配器匹配的事件相匹配，该匹配器将成功。这意味着可能出现不匹配事件的“gaps”。
如果评估事件之后，更多的匹配器是可用的，他们都是匹配“null”。它是由事件的匹配器来决定是否接受。

* Exact sequence of Events: Matchers.exactSequenceOf(event matchers...)
“事件的序列”匹配器的变化不允许不匹配事件的空隙。这意味着每个匹配器必须与事件后面的事件相匹配，与前一个匹配器匹配的事件相匹配。每个匹配器都应该与它前一个匹配器相对应的事件的后续一个事件相匹配

为了方便起见,提供了一些普遍需要的事件匹配器。他们与单个事件实例相匹配:

* Equal Event: Matchers.equalTo(instance...)
验证given对象在语义上等于given事件，这个匹配器将比较实际和预期的对象的所有字段的值使用一个null-safe相等方法。这意味着可以比较事件，即使它们不实现equals方法。存储在given参数字段上的对象用equals进行比较，要求他们正确实现。

* No More Events: Matchers.andNoMore() or Matchers.nothing()
仅与空值匹配，这个匹配器可以作为最后一个匹配器添加到事件的准确顺序匹配器,以确保没有不匹配的事件依然存在。

由于匹配器传递一个事件消息列表，有时你只是想验证消息的有效负载。有匹配器来帮助你:

* Payload Matching: Matchers.messageWithPayload(payload matcher)
验证消息的有效负载匹配给定的有效载荷匹配器。

* Payloads Matching: Matchers.payloadsMatching(list matcher)
验证消息的有效负载匹配给定的有效载荷匹配器。给定的匹配器必须匹配列表包含的每个消息的有效负载。有效负载匹配匹配器通常用作外匹配器,以防止重复有效负载匹配器。

下面是一个简单的代码示例，以显示这些匹配器的使用。在这个例子中,我们预期共有两个事件发布。第一个事件必须是一个“ThirdEvent”，第二个是“aFourthEventWithSomeSpecialThings”。可能没有第三个事件，因为那样"andNoMore"匹配器会失败。

<pre>
fixture.given(new FirstEvent(), new SecondEvent())
       .when(new DoSomethingCommand("aggregateId"))
       .expectEventsMatching(exactSequenceOf(
           // we can match against the payload only:
           messageWithPayload(equalTo(new ThirdEvent())),
           // this will match against a Message
           aFourthEventWithSomeSpecialThings(),
           // this will ensure that there are no more events
           andNoMore()
       ));

// or if we prefer to match on payloads only:
       .expectEventsMatching(payloadsMatching(
               exactSequenceOf(
                   // we only have payloads, so we can equalTo directly
                   equalTo(new ThirdEvent()),
                   // now, this matcher matches against the payload too
                   aFourthEventWithSomeSpecialThings(),
                   // this still requires that there is no more events
                   andNoMore()
               )
       ));
</pre>

作者：勇赴
链接：https://www.jianshu.com/p/2e893c9d48bf
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。
