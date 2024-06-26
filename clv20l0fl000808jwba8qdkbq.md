---
title: "Spring 事务的原理"
datePublished: Tue Apr 16 2024 06:40:11 GMT+0000 (Coordinated Universal Time)
cuid: clv20l0fl000808jwba8qdkbq
slug: spring
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/hxCIfi47FgQ/upload/3aac96d7faed03d9aa8879cc8d28ed92.jpeg
tags: transational

---

官方文档：[https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html#:~:text=The%20%40Transactional%20annotation%20is%20metadata,suspending%20any%20existing%20transaction%22](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html#:~:text=The%20%40Transactional%20annotation%20is%20metadata,suspending%20any%20existing%20transaction%22)).

推荐阅读：[https://www.marcobehler.com/guides/spring-transaction-management-transactional-in-depth](https://www.marcobehler.com/guides/spring-transaction-management-transactional-in-depth)

# **Spring 事务的原理**

@Transactional 注解的底层实现使用的是 `JDBC` 事务实现的，所以如果需要使用 @Transactional 的话那么就需要使用支持的数据库，比如 MySQL 的 innodb 就至此事务，下面是简化代码：

```java
import java.sql.Connection;
​
Connection connection = dataSource.getConnection(); // (1)
​
try (connection) {
    connection.setAutoCommit(false); // (2)
    // execute some SQL statements...
    connection.commit(); // (3)
​
} catch (SQLException e) {
    connection.rollback(); // (4)
}
```

（1）你需要连接到数据库才能启动事务。DriverManager.getConnection(url, user, password) 也可以，但在大多数企业应用程序中，你会配置数据源并从中获取连接。这是在 Java 中“启动”数据库事务的唯一方式，尽管名称听起来有点奇怪。

（2）setAutoCommit(true) 确保每个单独的 SQL 语句都自动包装在自己的事务中，而 setAutoCommit(false) 则相反：你是事务的管理者，需要开始调用 commit 和其他相关方法。请注意，autoCommit 标志在连接打开的整个时间段内有效，这意味着你只需调用该方法一次，而不是重复调用。

（3）让我们提交我们的事务...

（4）或者，如果出现异常，回滚我们的更改。

> 这段代码虽然简单，但是确实 Spring 的 @Transactional 帮我们实现事务的核心代码

同时对应的 @Transactional 对应的注解是相关的参数都有 JDBC 的阐述

比如

```java
@Transactional(propagation=TransactionDefinition.NESTED,
               isolation=TransactionDefinition.ISOLATION_READ_UNCOMMITTED)
```

对应的就是 JDBC 的 savePoint 和 isolate

```java
import java.sql.Connection;
​
// isolation=TransactionDefinition.ISOLATION_READ_UNCOMMITTED
​
connection.setTransactionIsolation(Connection.TRANSACTION_READ_UNCOMMITTED); // (1)
​
// propagation=TransactionDefinition.NESTED
​
Savepoint savePoint = connection.setSavepoint(); // (2)
...
connection.rollback(savePoint);
```

1. Spring @Transactional 对应的 isolation 也是设置的 JDBC 实现的
    
2. 嵌入式事务的实现是通过设置 savePoint
    

## **Spring 的事务管理如何工作**

我们需要明白一点就是 Spring 的 @Transactional 不是洪水猛兽，我们不需要太过于害怕。反正就是 JDBC 事务的 start 、commit 、rollback 就是这些操作。

但是问题出在，使用普通 JDBC，只有一种方法 (setAutocommit(false)) 来管理事务，而 Spring 为您提供了许多不同的、更方便的方法来实现相同的目的。

如何实现的？我们后面会慢慢来说！

## **如何使用 Spring 编程式事务 ？**

```java
@Service
public class UserService {
​
    @Autowired
    private TransactionTemplate template;
​
    public Long registerUser(User user) {
        Long id = template.execute(status ->  {
            // execute some SQL that e.g.
            // inserts the user into the db and returns the autogenerated id
            return id;
        });
    }
}
```

和普通 JDBC 事务进行对比：

1. 不需要手动创建、关闭资源
    
2. 不需要捕获 SQLExceptions ，Spring 会把 SQLExceptions 转换成 RuntimExceptions
    
3. 更符合 Spring 生态，使用 @Autowired
    

使用编程式事务有一个好处就是，我们可以控制事务的最小粒度增加执行的效率！

## **使用 xml 配置 Spring 事务**

之前这种配置很常见，但是现在注解式编程更加常见‘

```java
<!-- the transactional advice (what 'happens'; see the <aop:advisor/> bean below) -->
    <tx:advice id="txAdvice" transaction-manager="txManager">
        <!-- the transactional semantics... -->
        <tx:attributes>
            <!-- all methods starting with 'get' are read-only -->
            <tx:method name="get*" read-only="true"/>
            <!-- other methods use the default transaction settings (see below) -->
            <tx:method name="*"/>
        </tx:attributes>
    </tx:advice>
```

还需要指定对应的 AOP 切面

```java
<aop:config>
    <aop:pointcut id="userServiceOperation" expression="execution(* x.y.service.UserService.*(..))"/>
    <aop:advisor advice-ref="txAdvice" pointcut-ref="userServiceOperation"/>
</aop:config>
​
<bean id="userService" class="x.y.service.UserService"/>
```

service 代码

```java
public class UserService {
​
    public Long registerUser(User user) {
        // execute some SQL that e.g.
        // inserts the user into the db and retrieves the autogenerated id
        return id;
    }
}
```

## **使用注解配置 Spring 事务**

```java
@Service
public class UserService {
​
    @Transactional
    public Long registerUser(User user) {
       // execute some SQL that e.g.
        // inserts the user into the db and retrieves the autogenerated id
        // userDao.save(user);
        return id;
    }
}
```

没有一点 xml 的配置，但是我们需要注意两点：

1. 需要确保 Spring 配置类需要 @EnableTransactionManagement 注解（SpringBoot 会自动帮我们配置）
    
    ```java
    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Inherited
    @SpringBootConfiguration
    @EnableAutoConfiguration
    @ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
            @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
    public @interface SpringBootApplication {
        .....
    }
    ```
    
2. 确保您在 Spring 配置中指定了事务管理器（无论如何都需要这样做）
    
3. 然后 Spring 非常方便，用 @Transactional 注释标识的任何 bean 的公共方法都将在数据库事务内执行
    

所以如果需要 @Transactional 生效的代码：

```java
@Configuration
@EnableTransactionManagement
public class MySpringConfig {
​
    @Bean
    public PlatformTransactionManager txManager() {
        return yourTxManager; // more on that later
    }
​
}
```

我们根据上文提到的 JDBC 方面的知识，我们可以猜到底层的实现:

```java
public class UserService {
​
    public Long registerUser(User user) {
        Connection connection = dataSource.getConnection(); // (1)
        try (connection) {
            connection.setAutoCommit(false); // (1)
​
            // execute some SQL that e.g.
            // inserts the user into the db and retrieves the autogenerated id
            // userDao.save(user); <(2)
​
            connection.commit(); // (1)
        } catch (SQLException e) {
            connection.rollback(); // (1)
        }
    }
}
​
```

## **CGlib & JDK Proxies**

还是动态代理实现的 @Transactional 的底层，Spring 帮我们在底层实现了

当一个 bean 上使用 @Transactional 时，Spring 都会使用到动态代理。它不仅实例化 UserService，而且还实例化该 UserService 的事务代理。它通过使用 Cglib 库中的 proxy-through-subclassing 方法来实现

![image-20240318093220184](https://leeikoooo-1313589692.cos.ap-beijing.myqcloud.com/imgs/image-20240318093220184.png align="left")

从上面的图片可以了解到 动态代理 帮我们做的工作

* 打开和关闭数据库连接/事务。
    
* 然后调用真正的 UserService
    
* 其他的 bean 比如 UserRestController “永远” 不会知道 TA 调用的 UseService 不是真的而是动态代理的对象
    

## **为什么需要事务管理器（如 PlatformTransactionManager）？**

UserService 会动态代理，而代理会为您管理事务。但实际上并非代理本身处理所有的事务状态（打开、提交、关闭），而是代理将这些工作委托给了事务管理器

Spring 提供 PlatformTransactionManager / TransactionManager 两个接口并且提供了实现

它完全做的就是你迄今为止管理事务所做的一切，但首先，让我们看一下所需的 Spring 配置：

```java
@Bean
public DataSource dataSource() {
    return new MysqlDataSource(); // (1)
}

@Bean
public PlatformTransactionManager txManager() {
    return new DataSourceTransactionManager(dataSource()); // (2)
}
```

1. 创建一个特定于数据库或连接池的数据源。本示例中使用MySQL
    
2. 创建你的事务管理器，它需要一个数据源来管理事务
    

下面就是一个简化版本的 transaction class doBegin 方法作用是开启事务， doCommit 方法就是提交事务

```java
public class DataSourceTransactionManager implements PlatformTransactionManager {

    @Override
    protected void doBegin(Object transaction, TransactionDefinition definition) {
        Connection newCon = obtainDataSource().getConnection();
        // ...
        con.setAutoCommit(false);
        // 是的就这些，没其他的了！
    }

    @Override
    protected void doCommit(DefaultTransactionStatus status) {
        // ...
        Connection connection = status.getTransaction().getConnectionHolder().getConnection();
        try {
            con.commit();
        } catch (SQLException ex) {
            throw new TransactionSystemException("Could not commit JDBC transaction", ex);
        }
    }
}
```

![image-20240318094417687](https://leeikoooo-1313589692.cos.ap-beijing.myqcloud.com/imgs/image-20240318094417687.png align="left")

1. 如果 Spring 检测到 bean 上的 @Transactional 注释，它会创建该 bean 的动态代理。
    
2. 代理可以访问事务管理器，并要求它打开和关闭事务/连接。
    
3. 事务管理器本身将做在普通 Java 部分中所做的操作，creat、rollback、commit ...
    

# **物理事务和逻辑事务有什么区别？**

```java
@Service
public class UserService {

    @Autowired
    private InvoiceService invoiceService;

    @Transactional
    public void invoice() {
        invoiceService.createPdf();
        // send invoice as email, etc.
    }
}

@Service
public class InvoiceService {

    @Transactional
    public void createPdf() {
        // ...
    }
}
```

UserService 有一个事务性 Invoice() 方法。它调用 InvoiceService 上的另一个事务方法 createPdf()。

现在数据库只有**一个**数据库事务. （就是 *getConnection(). setAutocommit(false). commit()*）Spring 称这个叫做物理事务

但是在对于 Spring 来说有发生了两个逻辑事务：第一个在 UserService 中，另一个在 InvoiceService 中。 Spring 非常智能，知道这两个 @Transactional 方法应该使用相同的物理事务之中

但是当我做出如下的修改：

```java
@Service
public class InvoiceService {

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void createPdf() {
        // ...
    }
}
```

那么物理事务就是两个了（ *getConnection() x2. setAutocommit(false) x2. commit() x2*），同时逻辑事务也是两个

# **@Transactional 传播级别**

首先介绍一下 @Transactional 的传播级别

```java
  @Transactional(propagation = Propagation.REQUIRED)

  // or

  @Transactional(propagation = Propagation.REQUIRES_NEW)
  // etc
```

一共有一下几个传播级别：

* REQUIRED
    
* SUPPORTS
    
* MANDATORY
    
* REQUIRES\_NEW
    
* NOT\_SUPPORTED
    
* NEVER
    
* NESTED
    

我们知道 JDBC 的事务能够做到的事情，但是 Spring 到底是如何通过 JDBC 的事务实现的这么多的级别？

* REQUIRED（默认）方法需要事务，但是如果存在一个事务就使用存在的事务 → *getConnection(). setAutoCommit(false). commit*
    
* SUPPORTS 无论有没有事务都会执行 → *JDBC* 什么也不会做
    
* MANDATORY 如果没有事务就会抛出异常，有事务正常执行 → *JDBC* 什么也不会做
    
* REQUIRES\_NEW 我想要一个完全的属于我自己的事务 → *getConnection(). setAutocommit(false). commit()*.
    
* NOT\_SUPPORTED 以非事务方式执行，如果存在当前事务，则暂停当前事务 → *JDBC* 什么也不做
    
* NEVER 以非事务执行，如果有事务就报错 → *JDBC* 什么也不做
    
* NESTED 如果有事务那么就用嵌入式的事务执行，好像有些复杂，但是其实这个是关于 savePoint → *connection.setSavepoint()*
    

# **@Transactional 隔离级别**

当我们进行下面配置

```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
```

底层其实是这样的：

```java
connection.setTransactionIsolation(Connection.TRANSACTION_REPEATABLE_READ);
```

还要注意，在事务过程中切换隔离级别时，必须确保 JDBC 驱动程序/数据库，了解支持的情况和不支持的情况

# **最常见的 @Transactional 坑**

```java
@Service
public class UserService {

    @Transactional
    public void invoice() {
        createPdf();
        // send invoice as email, etc.
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void createPdf() {
        // ...
    }
}
```

这个 UserService 里面有两个 @Transactional 标识的方法，而且其中的 creatPdf 方法的传播级别是 **REQUIRES\_NEW**

那么现在有一个问题，这个类之中有几个**物理事务** ？

答案是 1 个事务而不是 2 个物理事务！！！

为什么？

Spring 为创建事务性 UserService 代理，但是一旦您进入 UserService 类并调用其他内部方法，就不再涉及代理了！这意味着不会再有新的事务

![image-20240318115454760](https://leeikoooo-1313589692.cos.ap-beijing.myqcloud.com/imgs/image-20240318115454760.png align="left")

但是这个问题也是有对应的解法

1、可以自己注入自己然后调用 @Lazy @Resource private UserService userService; 2、可以使用 ((UserService) AopContext.currentProxy()).creatPdf() 3、使用 SpringUtil 获取 bean 调用 creatPdf() 方法