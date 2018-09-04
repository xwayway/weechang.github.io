---
title: AxonFramework事件处理
date: 2018-08-27 14:47:54
categories: AxonFramework
tags: [AxonFramework]
author: 勇赴
---

saga中的事件处理非常接近一个普通的事件监听器。上述的对于方法和参数解析的规则在这里是有效的。不过,有一个主要区别。虽然存在事件监听器只有单个实例处理所有传入事件，但也存在一个saga有多个实例，每个实例都对不同的事件感兴趣。例如,关于Order的id为1的管理业务Saga对Order“2”的事件不感兴趣，反之亦然。

<!-- more -->

Axon不会将所有事件都发布给所有saga实例（这将是对资源的完全浪费），而是只发布与saga相关联的属性的事件。这个通过使用AssociationValues完成。一个AssociationValue由key和value组成。key代表标识符使用的类型，例如“orderId”或“order”。value表示前面例子中相应“1”或“2”值。

带@SagaEventHandler注解的方法被评估的顺序与带@EventHandler的相同。如果处理器方法的参数与传入的事件匹配，那么方法就匹配，如果saga有一个定义在处理器方法上的association属性。

@SagaEventHandler注解有两个attribute,其中associationProperty是最重要的。这是传入事件上property的名称，应该用来寻找相关的saga。association值的key是property的名称。这个值是由property的getter方法返回的值。

例如,考虑一个带”String getOrderId()”方法传入事件,返回“123”。如果一个带@SagaEventHandler(associationProperty = orderId)注解的方法接受这个事件,这个事件被路由到所有已经与带一个键为orderId和值为“123”的AssociationValues关联的saga。这可能是一个,多个,甚至没有。

有时,想要关联的属性的名称不是想要使用的关联的名称。例如,你有一个销售订单相匹配购买订单的saga。你可以有一个包含“buyOrderId”和“sellOrderId”的事务对象。如果你想要的saga将“orderId”作为关联的值，你可以定义一个不同的keyName 在@SagaEventHandler注解中。它将变成@SagaEventHandler(associationProperty="sellOrderId", keyName="orderId")。

## 管理关联
当一个saga事务管理跨多个域的概念，如订单、发货,、发票，等等，saga需要与这些概念的实例关联。一个关联需要两个参数：key，识别关联的类型(订单、发货等)和一个value，该值表示该概念的标识符。
在以下几个方面完成用概念关联到saga。第一，当新创建一个saga时将调用一个用@StartSaga注解的事件处理器时，它将自动与@SagaEventHandler方法中标识的关联。所有其他的关联用SagaLifecycle.associateWith(String key, String/Number value)方法创建。用SagaLifecycle.removeAssociationWith(String key, String/Number value)方法去移除一个特定的关联。

想象一下为一个围绕着订单的事务而已经被创建的一个saga。saga自动关联订单，方法被@StartSaga注解。saga是负责创建该订单的发票，并告诉航运创建一个载货量。一旦货物到达和发票支付,交易完成后,saga被关闭。
这是一个saga的代码：

<pre>
public class OrderManagementSaga {
    private boolean paid = false;
    private boolean delivered = false;
    @Inject
    private transient CommandGateway commandGateway;
    @StartSaga
    @SagaEventHandler(associationProperty = "orderId")
    public void handle(OrderCreatedEvent event) {
        // client generated identifiers
        ShippingId shipmentId = createShipmentId();
        InvoiceId invoiceId = createInvoiceId();
        // associate the Saga with these values, before sending the commands
        associateWith("shipmentId", shipmentId);
        associateWith("invoiceId", invoiceId);
        // send the commands
        commandGateway.send(new PrepareShippingCommand(...));
        commandGateway.send(new CreateInvoiceCommand(...));
    }
    @SagaEventHandler(associationProperty = "shipmentId")
    public void handle(ShippingArrivedEvent event) {
        delivered = true;
        if (paid) { end(); }
    }
    @SagaEventHandler(associationProperty = "invoiceId")
    public void handle(InvoicePaidEvent event) {
        paid = true;
        if (delivered) { end(); }
    }
    // ...
}
</pre>
通过允许客户端生成标识符，可以很容易地与一个概念相关联，而不需要请求响应类型命令。在发布命令之前，我们将事件与这些概念关联起来。通过这种方式，我们也保证捕捉到作为该命令的一部分生成的事件。一旦发票付清，货物到达，saga也将结束。

作者：勇赴
链接：https://www.jianshu.com/p/297622466ca9
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。
