---
layout: post
title: SpringCloud（3）服务消费者（Feign）
date: 2019-01-13 00:35:00
author: 薛师兄
tags: SpringCloud
---
上一篇文章，讲述了如何通过 RestTemplate+Ribbon 去消费服务，这篇文章主要讲述如何通过Feign去消费服务。

## 1.Feign简介

Feign是一个声明式的伪Http客户端，它使得写Http客户端变得更简单。使用Feign，只需要创建一个接口并注解。它具有可插拔的注解特性，可使用Feign 注解和JAX-RS注解。Feign支持可插拔的编码器和解码器。Feign默认集成了Ribbon，并和Eureka结合，默认实现了负载均衡的效果。

简而言之：

- Feign 采用的是基于接口的注解
- Feign 整合了ribbon

## 2.准备工作

继续用上一节的工程， 启动eureka-server，端口为8761。启动 service-hi 两次，端口分别为 8762 、8763。

## 3.建一个feign的服务

新建一个spring-boot工程，取名为 serice-feign，在它的pom文件引入Feign的起步依赖spring-cloud-starter-feign、Eureka的起步依赖spring-cloud-starter-eureka、Web的起步依赖spring-boot-starter-web，代码如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>service-feign</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>service-feign</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Dalston.SR1</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-feign</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

在工程的配置文件 application.yml 文件，指定程序名为 service-feign，端口号为8765，服务注册地址为<http://localhost:8761/eureka/> ，代码如下：

```yml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
server:
  port: 8765
spring:
  application:
    name: service-feign
```

在程序的启动类 ServiceFeignApplication，加上 @EnableFeignClients 注解开启Feign的功能.

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class ServiceFeignApplication {

    public static void main(String[] args) {
        SpringApplication.run(ServiceFeignApplication.class, args);
    }
}
```

定义一个 feign 接口，通过 @FeignClient（“服务名”），来指定调用哪个服务。比如在代码中调用了 service-hi 服务的“/hi”接口，代码如下：

```java
@FeignClient(value = "service-hi")
public interface SchedualServiceHi {
    @RequestMapping(value = "/hi",method = RequestMethod.GET)
    String sayHiFromClientOne(@RequestParam(value = "name") String name);//@RequestParam注解必须写
}
```

在Web层的controller层，对外暴露一个”/hi”的API接口，通过上面定义的Feign客户端SchedualServiceHi 来消费服务。代码如下：

```java
@RestController
public class HiController {

    @Autowired
    SchedualServiceHi schedualServiceHi;
    
    @RequestMapping(value = "/hi",method = RequestMethod.GET)
    public String sayHi(@RequestParam String name){
        return schedualServiceHi.sayHiFromClientOne(name);
    }
}
```

启动程序，多次访问<http://localhost:8765/hi?name=forezp>，浏览器交替显示：

> hi forezp,i am from port:8762
>
> hi forezp,i am from port:8763


## 4.Feign的源码实现过程

总的来说，Feign 的源码实现过程如下。

1. 首先通过 @EnableFeignClients 注解开启 FeignClient 的功能。只有这个注解存在，才会在程序启动时开启 @FeignClient 注解的包扫描。
2. 根据Feign的规则实现接口，并在接口上面加上 @FeignClient 注解。
3. 程序启动后，会进行包扫描，扫描所有的@FeignClient 的注解的类，并将这些信息注入 IOC容器中。
4. 当接口的方法被调用时，通过JDK的代理来生成具体的 RequestTemplate 模板对象。
5. 根据 RequestTemplate 再生成 Http 请求的 Request 对象。
6. Request 对象交给 Client 去处理，其中 Client 的网络请求框架可以是 HTTPURLConnection、HttpClient 和 OkHttp。
7. 最后Client被封装到LoadBalanceClient类，这个类结合类 Ribbon 做到了负载均衡。


关于@FeignClient注解参考 [FeignClient注解及性能优化注意事项](https://www.cnblogs.com/moonandstar08/p/7565442.html)

> *参考方志朋《深入理解Spring Cloud与微服务构建》*