---
title: "Api 网关"
datePublished: Thu Sep 28 2023 02:25:18 GMT+0000 (Coordinated Universal Time)
cuid: cln2jz0nq000308jkabjy824f
slug: api
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/npxXWgQ33ZQ/upload/35ff4ddc688a442798c4e9368815d7a0.jpeg
tags: gateway

---

**场景**：当我们在做接口平台的时候，我们需要在每一个接口结束调用相关的方法进行记录次数，但是这时候就有问题了，如果我们需要在每一个接口调用的时候到要执行的话难道不能「提取出来」吗？

答案时肯定的，但是我们要用什么技术？AOP ？网关？

AOP 这个 Spring 非常重要的特性，那么该如何实现`这个项目`的相关接口执行之后调用统计方法呢？

```java
@Aspect
@Component
public class MethodExecutionCounterAspect {

    // 定义切点，可以根据自己的需求进行配置
    @After("execution(* com.example.yourpackage.YourService.*(..))")
    public void afterMethodExecution(JoinPoint joinPoint) {
        // 记录方法执行次数
        // 查询数据库进行加一
        methodExecutionCount++;
        System.out.println("Method executed: " + joinPoint.getSignature());
        System.out.println("Method execution count: " + methodExecutionCount);
        // 执行目标方法
        Object result = joinPoint.proceed();

        // 可以在这里添加额外的逻辑，根据需要修改 result
        return result;    
    }
}
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695809092352/f12e8aa0-a305-4757-9d5d-11539b61297d.png align="center")

但是 AOP 也有缺点，只能在单个项目中使用，虽然我们可以封装成 SDK 但是还是需要在每一个项目中引入和调用，有没有更简单的呢？当然那就是 `网关`使用网关之后就可以做 \*\*「统一」\*\*管理

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695809368884/af0747ff-73fb-4c94-b0fe-4f6e4ffb18bf.png align="center")

# 网关

什么是网关？理解成火车站的检票口，**统一** 去检票。

## 作用

统一去进行一些操作、处理一些问题。  
比如：

1. 路由
    
2. 负载均衡
    
3. 统一鉴权
    
4. 跨域
    
5. 统一业务处理（缓存）
    
6. 访问控制
    
7. 发布控制
    
8. 流量染色
    
9. 接口保护
    
    1. 限制请求
        
    2. 信息脱敏
        
    3. 降级（熔断）
        
    4. 限流：学习令牌桶算法、学习漏桶算法，学习一下 RedisLimitHandler
        
    5. 超时时间
        
10. 统一日志
    
11. 统一文档
    

### 路由

起到`转发`的作用，比如有接口 A 和接口 B，网关会记录这些信息，根据用户访问的地址和参数，转发请求到对应的接口（服务器 / 集群）  
/a =&gt; 接口A  
/b =&gt; 接口

[官方文档](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gateway-request-predicates-factories) 官方文档永远都是最重要的，记住这一点！

### 负载均衡

在路由的基础上  
/c =&gt; 服务 A / 集群 A（随机转发到其中的某一个机器）  
uri 从固定地址改成 lb:xxxx

### **统一处理跨域**

网关统一处理跨域，不用在每个项目里单独处理，这个的原因是网关一般是挡在最前面的原因。[文档](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#cors-configuration)

### **发布控制**

灰度发布，比如上线新接口，先给新接口分配 20% 的流量，老接口 80%，再慢慢调整比重。[文档](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-weight-route-predicate-factory)

### **流量染色**

给请求（流量）添加一些标识，一般是设置请求头中，添加新的请求头。这样就可以区分是不是经过了网关，没有经过网关的就不会有对应的请求头，可以防止直接请求后端接口

[文档](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#cors-configuration) [全局染色](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#default-filters)

### **统一接口保护**

1. 限制请求：[https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#requestheadersize-gatewayfilter-factory](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#requestheadersize-gatewayfilter-factory%EF%BF%BC2%E4%BF%A1%E6%81%AF%E8%84%B1%E6%95%8F%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-removerequestheader-gatewayfilter-factory%EF%BF%BC3%E9%99%8D%E7%BA%A7%EF%BC%88%E7%86%94%E6%96%AD%EF%BC%89%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#fallback-headers%EF%BF%BC4%E9%99%90%E6%B5%81%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-requestratelimiter-gatewayfilter-factory%EF%BF%BC5%E8%B6%85%E6%97%B6%E6%97%B6%E9%97%B4%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#http-timeouts-configuration%EF%BF%BC6%E9%87%8D%E8%AF%95%EF%BC%88%E4%B8%9A%E5%8A%A1%E4%BF%9D%E6%8A%A4%EF%BC%89%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-retry-gatewayfilter-factory)
    
2. 信息脱敏：[https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-removerequestheader-gatewayfilter-factory](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#requestheadersize-gatewayfilter-factory%EF%BF%BC2%E4%BF%A1%E6%81%AF%E8%84%B1%E6%95%8F%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-removerequestheader-gatewayfilter-factory%EF%BF%BC3%E9%99%8D%E7%BA%A7%EF%BC%88%E7%86%94%E6%96%AD%EF%BC%89%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#fallback-headers%EF%BF%BC4%E9%99%90%E6%B5%81%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-requestratelimiter-gatewayfilter-factory%EF%BF%BC5%E8%B6%85%E6%97%B6%E6%97%B6%E9%97%B4%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#http-timeouts-configuration%EF%BF%BC6%E9%87%8D%E8%AF%95%EF%BC%88%E4%B8%9A%E5%8A%A1%E4%BF%9D%E6%8A%A4%EF%BC%89%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-retry-gatewayfilter-factory)
    
3. 降级（熔断）：[https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#fallback-headers](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#requestheadersize-gatewayfilter-factory%EF%BF%BC2%E4%BF%A1%E6%81%AF%E8%84%B1%E6%95%8F%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-removerequestheader-gatewayfilter-factory%EF%BF%BC3%E9%99%8D%E7%BA%A7%EF%BC%88%E7%86%94%E6%96%AD%EF%BC%89%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#fallback-headers%EF%BF%BC4%E9%99%90%E6%B5%81%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-requestratelimiter-gatewayfilter-factory%EF%BF%BC5%E8%B6%85%E6%97%B6%E6%97%B6%E9%97%B4%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#http-timeouts-configuration%EF%BF%BC6%E9%87%8D%E8%AF%95%EF%BC%88%E4%B8%9A%E5%8A%A1%E4%BF%9D%E6%8A%A4%EF%BC%89%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-retry-gatewayfilter-factory)
    
4. 限流：[https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-requestratelimiter-gatewayfilter-factory](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#requestheadersize-gatewayfilter-factory%EF%BF%BC2%E4%BF%A1%E6%81%AF%E8%84%B1%E6%95%8F%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-removerequestheader-gatewayfilter-factory%EF%BF%BC3%E9%99%8D%E7%BA%A7%EF%BC%88%E7%86%94%E6%96%AD%EF%BC%89%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#fallback-headers%EF%BF%BC4%E9%99%90%E6%B5%81%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-requestratelimiter-gatewayfilter-factory%EF%BF%BC5%E8%B6%85%E6%97%B6%E6%97%B6%E9%97%B4%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#http-timeouts-configuration%EF%BF%BC6%E9%87%8D%E8%AF%95%EF%BC%88%E4%B8%9A%E5%8A%A1%E4%BF%9D%E6%8A%A4%EF%BC%89%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-retry-gatewayfilter-factory)
    
5. 超时时间：[https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#http-timeouts-configuration](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#requestheadersize-gatewayfilter-factory%EF%BF%BC2%E4%BF%A1%E6%81%AF%E8%84%B1%E6%95%8F%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-removerequestheader-gatewayfilter-factory%EF%BF%BC3%E9%99%8D%E7%BA%A7%EF%BC%88%E7%86%94%E6%96%AD%EF%BC%89%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#fallback-headers%EF%BF%BC4%E9%99%90%E6%B5%81%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-requestratelimiter-gatewayfilter-factory%EF%BF%BC5%E8%B6%85%E6%97%B6%E6%97%B6%E9%97%B4%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#http-timeouts-configuration%EF%BF%BC6%E9%87%8D%E8%AF%95%EF%BC%88%E4%B8%9A%E5%8A%A1%E4%BF%9D%E6%8A%A4%EF%BC%89%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-retry-gatewayfilter-factory)
    
6. 重试（业务保护）： [https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-retry-gatewayfilter-factory](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#requestheadersize-gatewayfilter-factory%EF%BF%BC2%E4%BF%A1%E6%81%AF%E8%84%B1%E6%95%8F%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-removerequestheader-gatewayfilter-factory%EF%BF%BC3%E9%99%8D%E7%BA%A7%EF%BC%88%E7%86%94%E6%96%AD%EF%BC%89%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#fallback-headers%EF%BF%BC4%E9%99%90%E6%B5%81%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-requestratelimiter-gatewayfilter-factory%EF%BF%BC5%E8%B6%85%E6%97%B6%E6%97%B6%E9%97%B4%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#http-timeouts-configuration%EF%BF%BC6%E9%87%8D%E8%AF%95%EF%BC%88%E4%B8%9A%E5%8A%A1%E4%BF%9D%E6%8A%A4%EF%BC%89%EF%BC%9Ahttps://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-retry-gatewayfilter-factory)
    

### **统一业务处理**

把一些每个项目中都要做的通用逻辑放到上层（网关），统一处理，比如本项目的次数统计

### 统一的**访问控制**

黑白名单，比如限制 DDOS IP

### **统一日志**

统一的请求、响应信息记录

### **统一文档**

将下游项目的文档进行聚合，在一个页面统一查看  
建议用：[https://doc.xiaominfo.com/docs/middleware-sources/aggregation-introduction  
](https://doc.xiaominfo.com/docs/middleware-sources/aggregation-introduction%EF%BF%BC)

## **网关的分类**

1. 全局网关（接入层网关）：作用是负载均衡、请求日志等，不和业务逻辑绑定
    
2. 业务网关（微服务网关）：会有一些业务逻辑，作用是将请求转发到不同的业务 / 项目 / 接口 / 服务
    

## **网关实现**

1. Nginx（全局网关）、Kong 网关（API 网关，Kong)，编程成本相对高一点，还需要学新的语言
    
2. [Spring](https://github.com/Kong/kong%EF%BC%89%EF%BC%8C%E7%BC%96%E7%A8%8B%E6%88%90%E6%9C%AC%E7%9B%B8%E5%AF%B9%E9%AB%98%E4%B8%80%E7%82%B9%EF%BF%BC2Spring) Cloud Gateway（取代了 Zuul）性能高、可以用 Java 代码来写逻辑，适于学习
    

## **Spring Cloud Gateway 用法**

路由（根据什么条件，转发请求到哪里）  
断言：一组规则、条件，用来确定如何转发路由  
过滤器：对请求进行一系列的处理，比如添加请求头、添加请求参数

创建项目需要安装的依赖

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695862683921/3c1bfb22-aae1-4cae-bec5-0eba3449e11f.png align="center")

**请求流程：**

1. 客户端发起请求
    
2. Handler Mapping：根据断言，去将请求转发到对应的路由
    
3. Web Handler：处理请求（一层层经过过滤器）
    
4. 实际调用服务
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695860928569/f485ee39-0fb2-4f2d-9ff4-99ddb9a15385.png align="center")

### **两种配置方式**

1. 配置式（方便、规范）  
    1） 简化版
    
    ```xml
    spring:
      cloud:
        gateway:
          routes:
          - id: after_route
            uri: https://example.org
            predicates:
            - Cookie=mycookie,mycookievalue
    ```
    
    1. 全称版
        
    
    ```xml
    spring:
      cloud:
        gateway:
          routes:
          - id: after_route
            uri: https://example.org
            predicates:
            - name: Cookie
              args:
                name: mycookie
                regexp: mycookievalue
    ```
    
2. 编程式（灵活、相对麻烦）
    

```java
@SpringBootApplication
public class DemogatewayApplication {
	@Bean
	public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
		return builder.routes()
			.route("path_route", r -> r.path("/get")
				.uri("http://httpbin.org"))
			.route("host_route", r -> r.host("*.myhost.org")
				.uri("http://httpbin.org"))
			.route("rewrite_route", r -> r.host("*.rewrite.org")
				.filters(f -> f.rewritePath("/foo/(?<segment>.*)", "/${segment}"))
				.uri("http://httpbin.org"))
			.route("hystrix_route", r -> r.host("*.hystrix.org")
				.filters(f -> f.hystrix(c -> c.setName("slowcmd")))
				.uri("http://httpbin.org"))
			.route("hystrix_fallback_route", r -> r.host("*.hystrixfallback.org")
				.filters(f -> f.hystrix(c -> c.setName("slowcmd").setFallbackUri("forward:/hystrixfallback")))
				.uri("http://httpbin.org"))
			.route("limit_route", r -> r
				.host("*.limited.org").and().path("/anything/**")
				.filters(f -> f.requestRateLimiter(c -> c.setRateLimiter(redisRateLimiter())))
				.uri("http://httpbin.org"))
			.build();
	}
}
```

### 开启日志

```xml
logging:
  level:
    org:
      springframework:
        cloud:
          gateway: trace
```

### **断言**

1. After 在 xx 时间之后
    
    场景：一般某个活动的结束、开始就需要这种`断言`
    
2. Before 在 xx 时间之前
    
3. Between 在 xx 时间之间
    
4. 请求类别
    
5. 请求头（包含 Cookie）
    
6. 查询参数
    
7. 客户端地址
    
8. **权重**
    

### **过滤器**

基本功能：对请求头、请求参数、响应头的增删改查

1. 添加请求头
    
2. 添加请求参数
    
3. 添加响应头
    
4. 降级
    
5. 限流
    
6. 重试
    

依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
</dependency>
```