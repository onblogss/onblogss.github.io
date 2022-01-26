---
layout: post
title: SpringBoot分布式 - SpringCloud
date: 2018-06-14 22:04:00
author: 薛师兄
tags: SpringCloud
---
# 一：介绍

*  Spring Cloud是一个基于Spring Boot实现的云应用开发工具，它为基于JVM的云应用开发中涉及的配置管理、服务发现、断路器、智能路由、微代理、控制总线、全局锁、决策竞选、分布式会话和集群状态管理等操作提供了一种简单的开发方式。
*  微服务(Microservices Architecture)是一种架构风格，一个大型复杂软件应用由一个或多个微服务组成。系统中的各个微服务可被独立部署，各个微服务之间是松耦合的。每个微服务仅关注于完成一件任务并很好地完成该任务。在所有情况下，每个任务代表着一个小的业务能力。
*  Spring Cloud 中文文档 [https://springcloud.cc/spring-cloud-dalston.html](https://springcloud.cc/spring-cloud-dalston.html)
*  Spring Cloud 官方文档 [http://projects.spring.io/spring-cloud/#quick-start](http://projects.spring.io/spring-cloud/#quick-start)

# 二：入门

*  在IDEA新建空白工程

## 1. 注册中心

*  在工程下新建模块eureka-server

1）导包

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka-server</artifactId>
</dependency>
```

2 )application.yml配置Eureka信息

```yaml
server:
  port: 8761
eureka:
  instance:
    hostname: eureka-server #eureka实例的主机名
  client:
    register-with-eureka: false #不把自己注册到eureka
    fetch-registry: false #不从eureka上获取服务的注册信息
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

3）开启@EnableEurekaServer注解

```java
package com.example.eurekaserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

/**
 * 开启@EnableEurekaServer注解
 */
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

4）开启注册中心

*  启动该应用，访问[http://localhost:8761/](http://localhost:8761/) 进入Spring Eureka页面即成功。

## 2.服务提供者

*  新建模块：provider-ticket

1）导入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka-server</artifactId>
</dependency>
```

2）application.yml配置Eureka信息

```yaml
server:
  port: 8001 #8002
spring:
  application:
    name: provider-ticket
eureka:
  instance:
    prefer-ip-address: true #注册服务的时候使用服务的Ip地址
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

3）Service层方法

```java
package com.example.providerticket.service;

import org.springframework.stereotype.Service;

/**
 * Create by Martin 2018/6/4/004 18:37
 */
@Service
public class TicketService {

    public String getTicket(){
        return "《厉害了，我的国》";
    }
}
```

4）暴露HTTP接口

```java
/**
 * Create by Martin 2018/6/4/004 18:39
 */
@RestController
public class TicketControllert {
    //轻量级HTTP

    @Autowired
    private TicketService ticketService;

    @GetMapping("/ticket")
    public String getTicket(){
        System.out.println("8001"); //8002
        return ticketService.getTicket();
    }

}
```

5）然后更改端口，分别打包该模块为jar包。运行。

## 3.服务消费者

*  新建模块consumer-user

1）application.yml配置Eureka信息

```yaml
server:
  port: 8200
spring:
  application:
    name: consumer-user
eureka:
  instance:
    prefer-ip-address: true #注册服务的时候使用服务的Ip地址
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

2）开启发现服务的功能，使用负载均衡机制（默认轮询）

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@EnableDiscoveryClient /*开启发现服务功能*/
@SpringBootApplication
public class ConsumerUserApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerUserApplication.class, args);
    }

    @LoadBalanced /*开启负载均衡机制*/
    @Bean
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }

}
```

3）获取服务

```java
/**
 * Create by Martin 2018/6/4/004 19:13
 */
@RestController
public class UserController {
    @Autowired
    private RestTemplate restTemplate;

    @GetMapping("/buy")
    public String getTicket(){
        String s = restTemplate.getForObject("http://PROVIDER-TICKET/ticket",String.class);
        return "购买了 "+s;
    }
}
```

*  启动服务消费者模块。查看服务提供者的控制台打印。
*  会发现第一次请求8001，第二次8002，8001，8002，，这是因为使用了负载均衡。
