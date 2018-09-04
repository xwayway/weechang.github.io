---
title: AxonFramework配置
date: 2018-08-20 18:34:05
categories: AxonFramework
tags: [AxonFramework, DDD, CQRS]
author: 勇赴
---

获得一个默认的配置是非常容易的：
Configuration config = DefaultConfigurer.defaultConfiguration().buildConfiguration();

<!-- more -->

这个配置为分发消息提供了在线程上处理消息、分发它们的实现。显然，这种配置不会很有用。你必须将你的命令模型对象和事件处理器注册到该配置中。

为此,通过.defaultConfiguration()方法返回配置实例。
Configurer configurer = DefaultConfigurer.defaultConfiguration();

配置提供了多种允许你注册这些组件的方法。在每个组件的相应章节有如何配置这些组件的详细描述。

组件一般形式的注册,如下:
Configurer configurer = DefaultConfigurer.defaultConfiguration();
configurer.registerCommandHandler(c -> doCreateComponent());
注意registerCommandBus中lambda表达式的调用。这个表达式的参数c是描述完整配置的配置对象。如果组件需要其他组件正常运行，则可以使用此配置来检索它们。

例如,注册一个需要序列化器的命令处理程序
configurer.registerCommandHandler(c -> new MyCommandHandler(c.serializer());

不是所有的组件都有其明确的accesor方法。检索一个配置中的组件,使用：
configurer.registerCommandHandler(c -> new MyCommandHandler(c.getComponent(MyOtherComp
onent.class));

该组件必须和配置一起注册,使用
configurer.registerComponent(componentType, builderFunction) .builder函数将接受配置对象作为输入参数。

使用Spring设置配置
使用Spring时,不需要显式地使用配置。相反,你可以简单地把@EnableAxon设置到一个Spring @configuration类上。

Axon将使用Spring应用程序上下文来定位构件的特定实现，并为那些不存在的提供默认值。因此,取代在配置中注册构件,你只需要在Spring应用程序上下文中让@Bean可用即可。

作者：勇赴
链接：https://www.jianshu.com/p/7fcbc0dcc39c
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。