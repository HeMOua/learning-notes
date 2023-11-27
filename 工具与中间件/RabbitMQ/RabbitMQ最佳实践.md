# RabbitMQ最佳实践

> 在使用消息机制时，我们通常需要考虑以下几个问题:
>
> - 消息不能丢失
> - 保证消息一定能投递到目的地
> - 保证业务处理和消息发送/消费的一致性

## 如何确保信息不丢失？

如果希望RabbitMQ重启之后消息不丢失，那么需要对以下3种实体均配置持久化：

- exchange
- queue
- message

声明exchange时设置持久化（`durable = true`）并且不自动删除(autoDelete = false)：

```java
boolean durable = true;
boolean autoDelete = false;
channel.exchangeDeclare("dlx", TOPIC, durable, autoDelete, null)
```

声明queue时设置持久化（`durable = true`）并且不自动删除(autoDelete = false)：

```java
boolean durable = true;
boolean autoDelete = false;
channel.queueDeclare("order-summary-queue", durable, false, autoDelete, queueArguments);
```

发送消息时通过设置`deliveryMode=2`持久化消息：

```java
AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
                    .contentType("application/json")
                    .deliveryMode(2)
                    .priority(0)
                    .build();
channel.basicPublish("order", "order.created", false, properties, "sample-data".getBytes())
```

## 如何确保信息发送成功？

有时，业务处理成功，消息也发了，但是我们并不知道消息是否成功到达了rabbitmq，如果由于网络等原因导致业务成功而消息发送失败，那么发送方将出现不一致的问题，此时可以使用rabbitmq的发送确认功能，即要求rabbitmq显式告知我们消息是否已成功发送。

首先需要在channel上设置`ConfirmListener`:

```java
channel.addConfirmListener(new ConfirmListener() {
    public void handleAck(long seqNo, boolean multiple) {
        if (multiple) {
            logger.info(seqNo + "号及其以前的所有消息发送成功，当消息发送成功后执行相应逻辑，比如标记事件为已发送或者删除原来事件");
        } else {
            logger.info(seqNo + "号发送成功，当消息发送成功后执行相应逻辑，比如标记事件为已发送或者删除原来事件");
        }
    }

    public void handleNack(long seqNo, boolean multiple) {
        if (multiple) {
            logger.info(seqNo + "号及其以前的所有消息发送失败，当消息发送失败后执行相应逻辑，比如重试或者标记事件发送失败");
        } else {
            logger.info(seqNo + "号发送失败，当消息发送失败后执行相应逻辑，比如重试或者标记事件发送失败");

        }
    }
});
```

然后在发送消息直线需要开启发送确认模式：

```java
//开启发送者确认
channel.confirmSelect();
```

然后发送消息：

```java
channel.basicPublish("order", "order.created", false, properties, "sample-data".getBytes());
```

当消息正常投递时，rabbitmq 客户端将异步调用`handleAck()`表示消息已经成功投递，此时程序可以自行处理投递成功之后的逻辑，比如在数据库中将消息设置为`已发送`。当消息投递出现异常时，`handleNack()`将被调用。

## 如何解决重复消费？

采用幂等性解决：幂等性指一次和多次请求某一个资源，对于资源本身应该具有同样的结果。也就是说，其任意多次执行对资源本身所产生的影响均与一次执行的影响相同。

在MQ中指，消费多条相同的消息，得到与消费该消息一次相同的结果。

## RabbitMQ 中的死信队列

死信队列，英文缩写：DLX。Dead Letter Exchange（死信交换机），当消息成为 Dead Message 后，可以被重新发送到另一个交换机，这个交换机就是DLX。

消息成为死信的三种情况：

1. 队列消息长度到达限制：

2. 消费者拒接消费消息，basicNack/basicReject，并且不把消息重新放入原目
   标队列，requeue=false;
3. 原队列存在消息过期设置，消息到达超时时间未被消费；

## RabbitMQ 如何实现消息可靠性

