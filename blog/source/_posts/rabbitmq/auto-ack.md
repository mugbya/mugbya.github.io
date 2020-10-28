---
title: 探究rabbitmq接收自动应答
toc: true
date: 2020-09-27 14:33:02
tags:
 - 技术
categories:
 - rabbitmq
---

## 1.缘起

最近一直在补rabbitmq知识，必须往深入拓展嘛

然后我在浏览别人的博客时，发现了关于自动应答的两种描述

- [自动应答：如果报错了，消息不会丢失，会无限循环消费，很容易就吧磁盘空间耗完](https://segmentfault.com/a/1190000015369917)
- [MQ broker只需要确认消息发送成功，无需等待应答就会丢弃消息](https://blog.csdn.net/zhetmdoubeizhanyong/article/details/103223777)


第一个说自动应答，消费异常会无限循环；第二个说只要消息发送成功，无需等待应答就会丢弃消息。那么肯定就不会有无限循环

又是两种不同的结论，我疑惑了，也来了兴趣，写了demo，也有了这篇博文



## 2.实践

### 2.1 错误尝试

实践其实很简答，配置设置自动模式

```yaml
spring:
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: guest
    password: guest
    virtual-host: test
    ttl: 5000
    publisher-confirm-type: correlated
    publisher-returns: true
    listener:
      simple:
        acknowledge-mode: auto
```

然后消费逻辑如此而已

```java
@Slf4j
@Component
public class ReceiveHandler {
    @RabbitListener(queues = {RabbitMQConfig.ORDER_QUEUE})
    public void receiveOrderMessage(String msg, Message message, Channel channel) throws IOException, InterruptedException {
        log.info("接受订单的消息: {}", msg);
        int a = 1 / 0;
    }
}
```

结果发现消费端确实无限循环消费，如第一贴博主所说。而且这个情况完全跟这边文章中的消息无限投递一样 [用了 springboot + rabbitmq 消息确认机制，我感觉掉坑里了](https://juejin.im/post/6844904205438681095)

消息被无限循环投递，消费端无限循环消费

> 可以在消费日志看到 Delivery Tag 一直在增加


其中rabbitmq UI看到如下


![](/images/rabbitmq/auto_ack-unack.png)


当停止消费程序, 消息重新回到ready状态, 该图略


这个情况让我有点诧异。自动ack居然跟手工ack 下 nack 重回队列完全一致

> **更多尝试**
> 通过开启重试机制，设置`最大重试次数`可以避免消息无限重复投递，此种情况则符合`fire-and-forget`, 即使没有catch住异常。
> - 达到最大重试次数，即使消费还失败，此时队列会删除该消息
> - rabbitmq只发送一次，重试是在消费端内部进行，重试时Delivery Tag不变

------

然后稍微修改了下代码，catch住异常

```java
@Slf4j
@Component
public class ReceiveHandler {
    @RabbitListener(queues = {RabbitMQConfig.ORDER_QUEUE})
    public void receiveOrderMessage(String msg, Message message, Channel channel) throws IOException, InterruptedException {
        log.info("接受订单的消息: {}", msg);
        try {
            int a = 1 / 0;
        }catch (Exception e) {
            log.info(message.getMessageProperties().getDeliveryTag() + "");
        }
    }
}

```

结果就发现没有无限循环投递跟消费情况


demo地址: [点击查看](https://github.com/mugbya/Java-Collection/tree/master/springboot-rabbitmq)

> 注意：demo很可能进行其他试验调整，还请相应更改



### 2.2 拨乱反正

上面做了demo，还导致我做出了一个(**错误**)的总结。但是我心里确纳闷，为啥java没有严格按照规范实现。心里有刺，去了技术群询问，也去了别的博文留言，结果别人心细一下就找到问题。先感谢下博主的回答[RabbitMQ - 消息确认](https://segmentfault.com/a/1190000022763182?_ea=67081401)


`spring.rabbitmq.listener.simple.acknowledge-mode` 也有是三个值

- none: 这才是自动确认
- auto: 根据情况确认(根据是正常返回还是引发异常来发出确认/否定)
- manual: 手工确认

源码 `org.springframework.amqp.core.AcknowledgeMode` 定义如下

```jave
	/**
	 * No acks - {@code autoAck=true} in {@code Channel.basicConsume()}.
	 */
	NONE,

	/**
	 * Manual acks - user must ack/nack via a channel aware listener.
	 */
	MANUAL,

	/**
	 * Auto - the container will issue the ack/nack based on whether
	 * the listener returns normally, or throws an exception.
	 * <p><em>Do not confuse with RabbitMQ {@code autoAck} which is
	 * represented by {@link #NONE} here</em>.
	 */
	AUTO;
```

> 所以一定要注意 none 才是自动确认啊！！！ 


当acknowledge-mode设置为none时，这回真的做到了`fire-and-forget`


## 3.总结

### 3.1 错误总结

至此，我有如下看法：springboot-rabbitmq 的自动ack，~~并没有如 `fire-and-forget` 实现，而是消费逻辑处理完成就会自动返回ack, catch住异常保证了程序正常运行，如果没有，则一旦异常，消费端就不会返回 ack。而是返回的类似nack requeue=true 这种~~.

~~所以并不存在发送就丢弃的情况，不知道是否在实现自动ack进行的改良~~

<br>

### 3.2 再次总结

自动应答模式下(none)：发送后即丢弃。即使消息消费异常
动态模式下(auto): 消费正常，则自动应答。 如果异常且没有处理，则返回nack, requeue=true


## 4.结尾


在写博客一直避免 以其昏昏，使人昭昭。 因为我曾经被错误的博客误导，所以想自己的博客不会误导别人。

这次明显发现冲突的地方，但是却没有真正点击进去查看到底有哪些参数, 这个错误犯的真是低级！

记录下完整的经过，时时回来看该篇文章，以便提醒自己做事细心谨慎，慢慢改变自己不谨慎的毛病







 
