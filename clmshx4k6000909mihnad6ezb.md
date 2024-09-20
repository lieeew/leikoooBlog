---
title: "swagger 接口文档"
datePublished: Thu Sep 21 2023 01:30:09 GMT+0000 (Coordinated Universal Time)
cuid: clmshx4k6000909mihnad6ezb
slug: swagger
tags: swagger

---

什么是接口文档？写接口信息的文档。

每个接口的信息包括：

* 请求参数
    
* 响应参数
    
    * 错误码
        
* 接口地址
    
* 接口名称
    
* 请求类型
    
* 请求格式
    
* 备注
    

谁用接口文档？  
答：一般是后端或者负责人来提供，后端和前端都要使用。

为什么需要接口文档？

●有个书面内容（背书或者归档），便于大家参考和查阅，便于 沉淀和维护 ，拒绝口口相传  
●接口文档便于前端和后端开发对接，前后端联调的 介质 。后端 =&gt; 接口文档 &lt;= 前端  
●好的接口文档支持在线调试、在线测试，可以作为工具提高我们的开发测试效率

怎么做接口文档？

●手写：比如腾讯文档、Markdown 笔记  
●自动化接口文档生成：自动根据项目代码生成完整的文档或在线调试的网页。Swagger、Postman（侧重接口管理）（国外）；apifox、apipost、eolink（国产）

使用的 SpringBoot 版本是： 2.7.13 使用的 Knife4j 版本是：https://doc.xiaominfo.com/v2/documentation/get\_start.html

对应的 pom 文件

```plaintext

<!-- Swagger-->  
<dependency>  
	<groupId>com.github.xiaoymin</groupId>  
	<artifactId>knife4j-spring-boot-starter</artifactId>  
	<version>2.0.7</version>  
</dependency>
```

如果如果 springboot version &gt;= 2.6，需要添加如下配置：

```yml
spring: 
  mvc:  
   pathmatch:  
    matching-strategy: ANT_PATH_MATCHER
```

访问的地址是：http://localhost:端口/doc.htm

```java
/**
 * knife4j 配置类，集成了 swagger
 *
 * @author leikooo
 */
@Configuration
@EnableSwagger2WebMvc
public class Knife4jConfiguration {

    @Bean
    public Docket defaultApi2() {
        Docket docket=new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(new ApiInfoBuilder()
                        //.title("swagger-bootstrap-ui-demo RESTful APIs")
                        .description("用户匹配系统")
                        .termsOfServiceUrl("https://github.com/lieeew")
                        .contact("xx@qq.com")
                        .version("1.0")
                        .build())
                //分组名称
                .groupName("2.0版本")
                .select()
                //这里指定Controller扫描包路径
                .apis(RequestHandlerSelectors.basePackage("com.yupi.user_center.controller"))
                .paths(PathSelectors.any())
                .build();
        return docket;
    }
}
```