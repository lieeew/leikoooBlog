---
title: "生成自定义 spring-boot-starter"
datePublished: Mon Sep 25 2023 09:24:58 GMT+0000 (Coordinated Universal Time)
cuid: clmyon57g000i09mfh3s71f2h
slug: spring-boot-starter
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/npxXWgQ33ZQ/upload/35ff4ddc688a442798c4e9368815d7a0.jpeg
tags: maven

---

创建对应项目必须要选着的**依赖 「Spring Configuration Processor」**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695605340368/8fcbc961-1262-40bc-96ac-aaa26e9f922b.png align="center")

一定把对应 pom 文件 build 删除

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695605651453/fd16247c-86f9-49ac-b8c0-61bf61d48579.png align="center")

需要把 主类替换成 YuApiClientConfig.java

```java
@Data
@Configuration
@ConfigurationProperties("yuapi.client")
@Component
public class YuApiClientConfig {
    private String accessKey;
    private String secretKey;

    @Bean
    public YupiApiClient yuApiClient() {
        return new YupiApiClient(secretKey, accessKey);
    }
}
```

然后在 **resource** 里面新建一个 **META-INF** 里面创建一个文件 **spring.factories**

```yaml
# spring-boot-starter
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.yupi.yuapiclientsdk.YuApiClientConfig
```

org.springframework.boot.autoconfigure.EnableAutoConfiguration 这个的需要的是 YuApiClientConfig.java 的路径

最后需要运行 maven 的 **install** 指令进行安装