---
title: "锁的注意事项和使用 Redisson 实现分布式锁"
datePublished: Thu Sep 21 2023 08:01:32 GMT+0000 (Coordinated Universal Time)
cuid: clmsvwgkq000h08me5zkkc35w
slug: redisson
tags: redis, java, redisson

---

> 场景：我们每天 0 点需要查询海量数据查询，并且存到 **Redis** 之中降低首屏开启时间，**提升用户体验**

如果部署到多台服务器上，但是我们只需要一台服务器上面存储到 Redis 之中，不需要所有的服务器上都存储 Redis 数据，因此我们就需要一些 **限制的手段** ，如果我们不限制的话，会导致

1. 浪费资源，如果部署到 100000 太服务器上面....
    
2. ***脏数据***，比如重复写入
    

我们的目的：**要控制定时任务在同一时间只有一个服务器能执行**

## 一些方案

1. 分离定时任务程序和主程序，让定时任务程序只在 1 个服务器运行定时任务。成本太大
    
2. 写死配置，每个服务器都执行定时任务，但是只有 ip 符合配置的服务器才真实执行业务逻辑，其他的直接返回。**成本最低**；缺点：但是我们的 IP 可能是不固定的，把 IP 写的太死了 ，修改起来非常麻烦
    
3. 动态配置，配置是可以轻松的、很方便地更新的（**代码无需重启**），但是只有 ip 符合配置的服务器才真实执行业务逻辑。
    
    * 数据库
        
    * Redis
        
    * 配置中心（Nacos、Apollo、Spring Cloud Config）
        
    
    问题：服务器多了、IP 不可控还是很麻烦，还是要人工修改
    
4. 分布式锁，只有抢到锁的服务器才能执行业务逻辑。
    
    * 坏处：增加成本
        
    * 好处：不用手动配置，多少个服务器都一样。
        

> 注意，只要是单机，就会存在单点故障。

### **锁**

有限资源的情况下，控制同一时间（段）只有某些线程（用户 / 服务器）能访问到资源。  
Java 实现锁：synchronized 关键字、并发包的类。但存在问题：只对**单个** JVM 有效，所以多台服务器就不能发挥作用

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695276473025/beffa70c-5bea-4030-bc24-e55cd96ba4ba.png align="center")

#### **分布式锁实现的关键**

**抢锁机制**  
怎么保证同一时间只有 1 个服务器能抢到锁？

**核心思想**

先来的人先把数据改成自己的标识（服务器 ip），后来的人发现标识已存在，就抢锁失败，继续等待。等先来的人执行方法结束，把标识清空，其他的人继续抢锁。

* MySQL 数据库：select for update 行级锁（最简单），或者用乐观锁。
    
* Redis 实现：内存数据库，读写速度快。支持 setnx、lua 脚本，比较方便我们实现分布式锁。
    
    setnx：set if not exists 如果不存在，则设置；只有设置成功才会返回 true，否则返回 false。这个可以保证 redis 只有一个服务器可以抢到锁
    

当我们**只是 setnx** 有什么问题 ？

**注意事项**

1）用完锁要释放（腾地方）  
2）锁一定要加过期时间

会导致问题：

1. 连锁效应：释放掉别人的锁。解决方法：解锁之前看看是不是自己加锁
    
2. 如果方法执行时间过长，锁提前过期了。 解决方法：「续期」
    
    ```java
    boolean end = false;
    	new Thread(() -> {
    		if (!end)}{
    			// 续期
    	})
    end = true;
    ```
    
3. 释放的时候还是有可能释放别人的锁
    
    > 场景：当我们 A 执行结束之后，已经到达了查询对应的锁是不是自己加的时候，此时 A 方法已经到了下面代码块标识的位置，我们的 key 突然**过期了,** 但是 A 不知道就把 B 的锁给删除了，此时 C 见缝插针直接又加上了 C 的锁，那么又会出现问题
    
    ```java
    if(get lock == A) {
        // 此时 A 程序位置
    	// set lock B
    	del lock
    }
    ```
    

解决方案： Redis + lua 脚本保证操作**原子性**

### Redisson 实现锁

Redisson：[https://github.com/redisson/redisson](https://github.com/redisson/redisson)

可以就像操作本地集合的方式操作 redission

#### 快速入门

不建议使用 SpringBoot 自动注入 Redisson（版本迭代太快），建议自己写配置文件

依赖：

```xml
        <!-- https://mvnrepository.com/artifact/org.redisson/redisson -->
        <dependency>
            <groupId>org.redisson</groupId>
            <artifactId>redisson</artifactId>
            <version>3.21.0</version>
        </dependency>
```

配置类：

```java
@Configuration
@ConfigurationProperties(prefix = "spring.redis")
@Data
public class RedissonConfig {

    /**
     * 端口
     */
    private String port;

    /**
     * 主机
     */
    private String host;

    /**
     * redisson 需要的 redis 库
     */
    private String redissonDatabase;
    @Bean
    public RedissonClient redissonClient() {
        // 1. Create config object
        Config config = new Config();
        // 这里不要写死
        String redissonUrl = String.format("redis://%s:%s", host, port);

        config.useSingleServer().setAddress(redissonUrl).setDatabase(Integer.parseInt(redissonDatabase));

        // 2. Create Redisson instance
        // Sync and Async API
        RedissonClient redisson = Redisson.create(config);
        return redisson;
    }

}
```

lock.tryLock 的参数：

1. 第一个参数（**等待时间**）：表示尝试获取锁的最长等待时间，以毫秒为单位。在这里，传递了 0，表示不等待，如果锁当前不可用，它会立即返回，意识是就会抢一次。如果传递了正整数值，它将等待指定的时间来获取锁。
    
2. 第二个参数（锁的持有时间）：表示成功获取锁后，锁将被持有的时间，以毫秒为单位。在这里，传递了 -1，表示锁将无限期地被持有，直到显式释放。
    
3. 第三个参数（时间单位）：表示等待时间和锁的持有时间的时间单位。在这里，传递了 TimeUnit.MILLISECONDS，表示时间单位是毫秒。你也可以选择其他时间单位，如秒、分钟等，具体取决于你的需求。
    

```java

    RLock lock = redissonClient.getLock("yupao:precachejob:docache:lock");
    try {
        // 只有一个线程能获取到锁
        if (lock.tryLock(0, -1, TimeUnit.MILLISECONDS)) {
            // todo 实际要执行的方法
            doSomeThings();
            System.out.println("getLock: " + Thread.currentThread().getId());
        }
    } catch (InterruptedException e) {
        System.out.println(e.getMessage());
    } finally {
        // 只能释放自己的锁
        // 必须在 finally 释放锁，因为在执行 try 里面的语句报错
        if (lock.isHeldByCurrentThread()) {
            System.out.println("unLock: " + Thread.currentThread().getId());
            lock.unlock();
        }
    }
```

#### Radisson 中提供的续期机制

##### 开一个监听线程，如果方法还没执行完，就帮你重置 redis 锁的过期时间。

##### **原理：**

1. ##### 监听当前线程，默认过期时间是 30 秒，每 10 秒续期一次（补到 30 秒）
    
2. ##### 如果线程挂掉（注意 debug 模式也会被它当成服务器宕机），则不会续期