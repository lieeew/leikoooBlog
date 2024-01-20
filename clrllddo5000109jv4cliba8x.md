---
title: "SpringStateMachine 状态机"
datePublished: Sat Jan 20 2024 04:50:54 GMT+0000 (Coordinated Universal Time)
cuid: clrllddo5000109jv4cliba8x
slug: springstatemachine
tags: spring-statemachine

---

环境：

> SpringBoot 3.2.1
> 
> Java 17
> 
> stateMachine 4.0.0

```xml
        <!-- https://mvnrepository.com/artifact/org.springframework.statemachine/spring-statemachine-data-redis -->
        <dependency>
            <groupId>org.springframework.statemachine</groupId>
            <artifactId>spring-statemachine-data-redis</artifactId>
            <version>4.0.0</version>
        </dependency>
```

官方文档：[https://spring.io/projects/spring-statemachine/](https://spring.io/projects/spring-statemachine/)

config 文件

```java
/**
 * @author <a href="https://github.com/lieeew">leikooo</a>
 * @description 订单状态机配置
 */
@Configuration
@EnableStateMachine(name = "orderStateMachine")
public class OrderStateMachineConfig extends StateMachineConfigurerAdapter<OrderState, OrderStateChangeAction> {

    @Override
    public void configure(StateMachineStateConfigurer<OrderState, OrderStateChangeAction> states) throws Exception {
        states.withStates()
                .initial(OrderState.ORDER_WAIT_PAY)
                .states(EnumSet.allOf(OrderState.class));
    }

    @Override
    public void configure(StateMachineTransitionConfigurer<OrderState, OrderStateChangeAction> transitions) throws Exception {
        transitions
                .withExternal()
                // 从 OrderState.ORDER_WAIT_PAY 状态转化成 OrderState.ORDER_WAIT_SEND ，需要 OrderStateChangeAction.PAY_ORDER
                .source(OrderState.ORDER_WAIT_PAY)
                .target(OrderState.ORDER_WAIT_SEND)
                .event(OrderStateChangeAction.PAY_ORDER)
                .and()
                // 从 OrderState.ORDER_WAIT_SEND 状态转化成 OrderState.ORDER_WAIT_RECEIVE ，需要 SEND_ORDER
                .withExternal().source(OrderState.ORDER_WAIT_SEND)
                .target(OrderState.ORDER_WAIT_RECEIVE)
                .event(OrderStateChangeAction.SEND_ORDER)
                .and()
                .withExternal().source(OrderState.ORDER_WAIT_RECEIVE)
                .target(OrderState.ORDER_FINISH)
                .event(OrderStateChangeAction.RECEIVE_ORDER);
    }

    @Bean
    public StateMachinePersist<OrderState, OrderStateChangeAction, String> stateMachinePersist(RedisConnectionFactory connectionFactory) {
        RedisStateMachineContextRepository<OrderState, OrderStateChangeAction> repository =
                new RedisStateMachineContextRepository<>(connectionFactory);
        return new RepositoryStateMachinePersist<>(repository);
    }

    @Bean
    public RedisStateMachinePersister<OrderState, OrderStateChangeAction> redisStateMachinePersister(
            StateMachinePersist<OrderState, OrderStateChangeAction, String> stateMachinePersist) {
        return new RedisStateMachinePersister<>(stateMachinePersist);
    }
}
```

service 代码

```java
@Slf4j
@Service
public class OrderService {
    @Resource
    private RedisCommonProcessor redisCommonProcessor;

    @Resource
    private StateMachine<OrderState, OrderStateChangeAction> stateMachine;

    @Resource
    private StateMachinePersister<OrderState, OrderStateChangeAction, String> stateMachinePersister;

    @Resource
    private OrderCommand orderCommand;

    @Resource
    private PayFace payFace;

    public Order createOrder(final String productId) {
        String orderId = "OID" + productId;
        Order order = Order.builder().productId(productId).orderId(orderId).orderState(OrderState.ORDER_WAIT_PAY).build();
        redisCommonProcessor.set(orderId, order, 600);
        OrderCommandInvoker orderCommandInvoker = new OrderCommandInvoker();
        orderCommandInvoker.invoke(orderCommand, order);
        return order;
    }

    public Order payOrder(final String orderId) {
        Order order = (Order) redisCommonProcessor.get(orderId);
        Message<OrderStateChangeAction> message = MessageBuilder.withPayload(OrderStateChangeAction.PAY_ORDER).setHeader(StateMachineConstant.MESSAGE_HEADER_KEY, order).build();
        if (!changeStateAction(message, order)) {
            return null;
        }
        return order;
    }

    public Order sendOrder(final String orderId) {
        Order order = (Order) redisCommonProcessor.get(orderId);
        Message<OrderStateChangeAction> message = MessageBuilder.withPayload(OrderStateChangeAction.SEND_ORDER).setHeader(StateMachineConstant.MESSAGE_HEADER_KEY, order).build();
        if (!changeStateAction(message, order)) {
            return null;
        }
        return order;
    }

    public Order receiveOrder(final String orderId) {
        Order order = (Order) redisCommonProcessor.get(orderId);
        Message<OrderStateChangeAction> message = MessageBuilder.withPayload(OrderStateChangeAction.RECEIVE_ORDER).setHeader(StateMachineConstant.MESSAGE_HEADER_KEY, order).build();
        if (!changeStateAction(message, order)) {
            return null;
        }
        return order;
    }

    public String payFace(String orderId, Integer payType, Float price) {
        Order order = (Order) redisCommonProcessor.get(orderId);
        Order completeOrder = Order.builder().price(price).productId(order.getProductId()).orderState(order.getOrderState()).orderId(orderId).build();
        return payFace.pay(completeOrder, payType);
    }
    /**
     * 修改 stateMachine 的订单状态
     *
     * @param message
     * @param order
     * @return
     */
    private boolean changeStateAction(Message<OrderStateChangeAction> message, Order order) {
        try {
            stateMachine.start();
            stateMachinePersister.restore(stateMachine, order.getOrderId() + StateMachineConstant.STATEMACHINE_KEY_SUFFIX);
            boolean result = stateMachine.sendEvent(message);
            // 存储状态机到 redis
            stateMachinePersister.persist(stateMachine, order.getOrderId() + StateMachineConstant.STATEMACHINE_KEY_SUFFIX);
            return result;
        } catch (Exception e) {
            log.error(e.getMessage());
        } finally {
            stateMachine.stop();
        }
        return false;
    }
}
```

StateMachine Listener 具体的代码，当状态发生改变的时候我们就会调用对应的代码，同时一定要注意：这里的 order 对应的 state 不会自动改变的，需要我们**手动**改变

```java
@Component
@WithStateMachine(name = "orderStateMachine")
public class OrderStateListener {
    @Resource
    private RedisCommonProcessor redisCommonProcessor;

    @Resource
    private OrderCommand orderCommand;

    @OnStateChanged(source = "ORDER_WAIT_PAY", target = "ORDER_WAIT_SEND")
    public boolean payToSend(Message<OrderStateChangeAction> message) {
        final Order order = (Order) message.getHeaders().get(StateMachineConstant.MESSAGE_HEADER_KEY);
        assert order != null : "order 不存在";
        if (order.getOrderState() != OrderState.ORDER_WAIT_PAY) {
            throw new UnsupportedOperationException("order state error !");
        }
        final Order updateOrder = Order.builder().orderId(order.getOrderId()).orderState(OrderState.ORDER_WAIT_SEND).price(order.getPrice()).productId(order.getProductId()).build();
        redisCommonProcessor.set(order.getOrderId(), updateOrder, 900);
        OrderCommandInvoker orderCommandInvoker = new OrderCommandInvoker();
        orderCommandInvoker.invoke(orderCommand, updateOrder);
        return true;
    }

    @OnStateChanged(source = "ORDER_WAIT_SEND", target = "ORDER_WAIT_RECEIVE")
    public boolean sendToReceive(Message<OrderStateChangeAction> message) {
        final Order order = (Order) message.getHeaders().get(StateMachineConstant.MESSAGE_HEADER_KEY);
        assert order != null : "order 不存在";
        if (order.getOrderState() != OrderState.ORDER_WAIT_SEND) {
            throw new UnsupportedOperationException("order state error !");
        }
        final Order updateOrder = Order.builder().orderId(order.getOrderId()).orderState(OrderState.ORDER_WAIT_RECEIVE).price(order.getPrice()).productId(order.getProductId()).build();
        redisCommonProcessor.set(order.getOrderId(), updateOrder, 900);
        OrderCommandInvoker orderCommandInvoker = new OrderCommandInvoker();
        orderCommandInvoker.invoke(orderCommand, updateOrder);
        return true;
    }

    @OnStateChanged(source = "ORDER_WAIT_RECEIVE", target = "ORDER_FINISH")
    public boolean receiveToFinish(Message<OrderStateChangeAction> message) {
        final Order order = (Order) message.getHeaders().get(StateMachineConstant.MESSAGE_HEADER_KEY);
        assert order != null : "order 不存在";
        if (order.getOrderState() != OrderState.ORDER_WAIT_RECEIVE) {
            throw new UnsupportedOperationException("order state error !");
        }
        final Order updateOrder = Order.builder().orderId(order.getOrderId()).orderState(OrderState.ORDER_FINISH).price(order.getPrice()).productId(order.getProductId()).build();
        redisCommonProcessor.set(order.getOrderId(), updateOrder, 600);
        redisCommonProcessor.remove(order.getOrderId() + StateMachineConstant.STATEMACHINE_KEY_SUFFIX);
        OrderCommandInvoker orderCommandInvoker = new OrderCommandInvoker();
        orderCommandInvoker.invoke(orderCommand, updateOrder);
        return true;
    }
}
```

```java
public enum OrderState {
    ORDER_WAIT_PAY, // 待支付
    ORDER_WAIT_SEND, // 待发货
    ORDER_WAIT_RECEIVE, // 待收货
    ORDER_FINISH; // 完成订单
}

public enum OrderStateChangeAction {
    PAY_ORDER, // 支付操作
    SEND_ORDER, // 发货操作
    RECEIVE_ORDER; // 收获操作
}
```