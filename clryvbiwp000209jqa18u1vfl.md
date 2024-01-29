---
title: "Amqp 发送消息、消费消息、设置消息发送成功回调"
datePublished: Mon Jan 29 2024 11:50:24 GMT+0000 (Coordinated Universal Time)
cuid: clryvbiwp000209jqa18u1vfl
slug: amqp
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/oqStl2L5oxI/upload/3a7bf87d4e5cbf4aef59ec6ebf5aabb2.jpeg
tags: java, rabbitmq

---

SpringBoot 版本：3.2.1  
Java 版本：17  
rabbitmq 版本:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1706528464445/ca28325a-670b-4d09-a633-c74179251e73.png align="left")

pom 依赖

```java
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

yml 配置

这里要注意是 Spring.username 不是 Spring.stream.username！

```java
Spring:
  rabbitmq:
  host: IP
  port: 5672
  username: 账户
  password: 密码
```

## 配置文件

```java
package com.leikooo.config;

import com.leikooo.constant.AmqpConstants;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.*;
import org.springframework.amqp.rabbit.config.SimpleRabbitListenerContainerFactory;
import org.springframework.amqp.rabbit.connection.CachingConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.boot.autoconfigure.amqp.SimpleRabbitListenerContainerFactoryConfigurer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;

/**
 * @author <a href="https://github.com/lieeew">leikooo</a
 */
@Slf4j
@Configuration
public class RabbitMQConfig {
    private final CachingConnectionFactory cachingConnectionFactory;

    public RabbitMQConfig(CachingConnectionFactory cachingConnectionFactory) {
        this.cachingConnectionFactory = cachingConnectionFactory;
        this.cachingConnectionFactory.setPublisherReturns(true);
        this.cachingConnectionFactory.setPublisherConfirmType(CachingConnectionFactory.ConfirmType.CORRELATED);
    }

    @Bean
    public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory(SimpleRabbitListenerContainerFactoryConfigurer configurer) {
        SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
        configurer.configure(factory, cachingConnectionFactory);
        factory.setAcknowledgeMode(AcknowledgeMode.MANUAL);
        factory.setDefaultRequeueRejected(false);
        return factory;
}

    @Bean
    public Queue createNormalQueue() {

        return QueueBuilder.durable(AmqpConstants.NORMAL_QUEUE)
                .withArgument("x-dead-letter-exchange", AmqpConstants.DEAD_EXCHANGE)
                .withArgument("x-dead-letter-routing-key", AmqpConstants.DEAD_ROUTING_KEY)
                .build();
    }

    @Bean
    public Declarables createDeadQueue() {
        return new Declarables(
                new Queue(AmqpConstants.DEAD_QUEUE),
                new DirectExchange(AmqpConstants.NORMAL_EXCHANGE),
                new DirectExchange(AmqpConstants.DEAD_EXCHANGE),
                new Binding(AmqpConstants.NORMAL_QUEUE, Binding.DestinationType.QUEUE, AmqpConstants.NORMAL_EXCHANGE, AmqpConstants.NORMAL_ROUTING_KEY, null),
                new Binding(AmqpConstants.DEAD_QUEUE, Binding.DestinationType.QUEUE, AmqpConstants.DEAD_EXCHANGE, AmqpConstants.DEAD_ROUTING_KEY, null)
        );
    }

    @Bean
    public Jackson2JsonMessageConverter converter() {
        return new Jackson2JsonMessageConverter();
    }

    @Bean
    public RabbitTemplate rabbitTemplate(Jackson2JsonMessageConverter converter) {
        RabbitTemplate template = new RabbitTemplate(cachingConnectionFactory);
        template.setMessageConverter(converter);
        template.setConfirmCallback((correlationData, ack, cause) -> log.info("消息发送成功:correlationData({}),ack({}),cause({})", correlationData, ack, cause));
        return template;
    }

}
```

## 发送消息

```java
    MessageProperties messageProperties = new MessageProperties();
    messageProperties.setExpiration(delayTime);
    Message message = new Message(productId.getBytes(), messageProperties);
	// 这个可以实现消息回调，哪里看的 官方文档！！！！
    CorrelationData correlationData = new CorrelationData(uniqueId);
    rabbitTemplate.convertAndSend(AmqpConstants.NORMAL_EXCHANGE, AmqpConstants.NORMAL_ROUTING_KEY, message, correlationData);
```

**实现消息回调逻辑**

  
Confirmed (with correlation) and returned messages are supported by setting the CachingConnectionFactory property publisherConfirmType to ConfirmType.CORRELATED and the publisherReturns property to 'true'. 出自官方文档！

示例代码

```java
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class MessageSender {

    private final RabbitTemplate rabbitTemplate;

    @Autowired
    public MessageSender(RabbitTemplate rabbitTemplate) {
        this.rabbitTemplate = rabbitTemplate;
    }

    public void sendMessage(String exchange, String routingKey, Object message) {
        //  // 设置唯一的相关数据，可以用来跟踪消息
        CorrelationData correlationData = new CorrelationData("unique-id");
        rabbitTemplate.convertAndSend(exchange, routingKey, message, correlationData);
    }
}
```

## **消费消息**

查了很多地方，如果这里不接受 Chanle 的话会报错的，但是我看 execute（（chanle -&gt; {}））可与这样但是官方文档没有查阅到  
Shutdown Signal: channel error; protocol method: #method&lt;channel.close&gt;(reply-code=406, reply-text=PRECONDITION\_FAILED - unknown delivery tag 1, class-id=60, method-id=80)  
我属实解决不了，而且只要接受 Chanle 的话是我现阶段最好的选择  

正确案例

```java
 @RabbitListener(queues = {AmqpConstants.DEAD_QUEUE}) // 没有写 ackMode 是因为在配置文件里面指定了
    public void consumerMes(String message, Channel channel, @Header(AmqpHeaders.DELIVERY_TAG) long deliveryTag) {
        log.info("Received message: {}", message);
        try {
            Products referenceById = productRepository.findByProductId(message);
            if (referenceById != null && referenceById.getSendRedBag() == 1) {
                //  nack 消息怎么办？
                channel.basicAck(deliveryTag, false);
            } else {
                channel.basicNack(deliveryTag, false, true);
            }
        } catch (Exception e) {
            try {
                channel.basicNack(deliveryTag, false, true);
            } catch (IOException ex) {
                log.error(ex.getMessage());
            }
        };
    }
```

**错误案例！**  
我这样写没有实现确认逻辑

```java
  @RabbitListener(queues = {AmqpConstants.DEAD_QUEUE}, ackMode = "MANUAL")
    public void consumerMes(String message, @Header(AmqpHeaders.DELIVERY_TAG) long deliveryTag) {
        log.info("Received message: {}", message);
        rabbitTemplate.receiveAndReply((ReceiveAndReplyCallback<String, String>) payload -> {
            log.info("Received message: {}", payload);
            return payload;
        });
        try {
            Products referenceById = productRepository.findByProductId(message);
            if (referenceById != null && referenceById.getSendRedBag() == 1) {
                //  nack 消息怎么办？
                rabbitTemplate.execute(channel -> {
                    channel.basicAck(deliveryTag, false);
                    return "";
                });
            } else {
                rabbitTemplate.execute(channel -> {
                    channel.basicNack(deliveryTag, false, false);
                    return "";
                });
            }
        } catch (Exception e) {
            rabbitTemplate.execute(channel -> {
                channel.basicNack(deliveryTag, false, false);
                return "";
            });
        }
  }
```

### 各种消息回调

1. **Confirm Callback (**`ConfirmCallback`):
    
    * 用于处理消息是否成功投递到 Broker 的确认信息。
        
    * `correlationData`：发送消息时传递的相关数据。
        
    * `ack`：消息是否成功投递到 Broker。
        
    * `cause`：如果 `ack` 为 false，则包含失败的原因。
        
    
    示例：
    
    ```java
    rabbitTemplate.setConfirmCallback((correlationData, ack, cause) -> {
        if (ack) {
            log.info("Message successfully delivered to the broker");
        } else {
            log.error("Failed to deliver message to the broker. Cause: {}", cause);
        }
    });
    ```
    
2. **Return Callback (**`ReturnsCallback`):
    
    * 用于处理未被路由到合适队列的消息。
        
    * 在消息无法路由时触发。
        
    
    示例：
    
    ```java
    rabbitTemplate.setReturnsCallback(returned -> {
        log.error("Message returned: {}", returned);
    });
    ```
    
3. **Recovery Callback (**`RecoveryCallback`):
    
    * 用于处理 RabbitMQ 连接恢复时的回调。
        
    * `context`：包含一些有关恢复的信息。
        
    
    示例：
    
    ```java
    rabbitTemplate.setRecoveryCallback((RecoveryCallback<String>) context -> {
        log.info("Recovering connection. Context: {}", context);
        return null;
    });
    ```
    

这些回调的设置允许你在与 RabbitMQ 交互的不同阶段执行自定义逻辑。根据你的具体需求，你可以在相应的回调方法中添加逻辑来处理不同的情况。