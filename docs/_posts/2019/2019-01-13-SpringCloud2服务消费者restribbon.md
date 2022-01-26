---
layout: post
title: SpringCloud（2）服务消费者（rest+ribbon）
date: 2019-01-13 00:00:00
author: 薛师兄
tags: SpringCloud
---
## 1.准备工作

这一篇文章基于上一篇文章的工程。启动eureka-server 工程，端口为 8761。分别以端口 8762 和 8763 启动 service-hi 工程。访问 localhost:8761 你会发现，service-hi 在eureka-server 注册了2个实例，这就相当于一个小的集群。

## 2.建1个服务消费者

重新新建一个 spring-boot 工程，取名为 service-ribbon。

在它的 pom.xml 文件分别引入起步依赖 spring-cloud-starter-eureka、spring-cloud-starter-ribbon、spring-boot-starter-web，代码如下：

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
    <artifactId>service-ribbon</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>service-ribbon</name>
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
            <artifactId>spring-cloud-starter-ribbon</artifactId>
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

在工程的配置文件指定服务的注册中心地址，程序名称为 service-ribbon，程序端口为 8764。完整配置文件application.yml如下：

```yml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
server:
  port: 8764
spring:
  application:
    name: service-ribbon
```

在工程的启动类中，通过 @EnableDiscoveryClient 向服务中心注册，并且向Spring的IOC容器注入一个Bean：RestTemplate，并通过 @LoadBalanced 注解表明这个 RestRemplate 开启负载均衡的功能。

```java
@SpringBootApplication
@EnableDiscoveryClient
public class ServiceRibbonApplication {

    public static void main(String[] args) {
        SpringApplication.run(ServiceRibbonApplication.class, args);
    }

    @Bean
    @LoadBalanced
    RestTemplate restTemplate() {
        return new RestTemplate();
    }

}
```

写一个测试接口 HelloController，通过之前注入 Ioc 容器的 RestTemplate 来消费 service-hi 服务的“/hi”接口，在这里我们直接用的程序名替代了具体的url地址，在 ribbon 中它会根据服务名来选择具体的服务实例，根据服务实例在请求的时候会用具体的 url 替换掉服务名，代码如下：

```java
@RestController
public class HelloController {
    @Autowired
    RestTemplate restTemplate;

    @RequestMapping(value = "/hi")
    public String hi(@RequestParam String name){
        return restTemplate.getForObject("http://SERVICE-HI/hi?name="+name,String.class);
    }

}
```

在浏览器上多次访问<http://localhost:8764/hi?name=forezp>，浏览器交替显示：

> hi forezp,i am from port:8762
>
> hi forezp,i am from port:8763

这说明当我们通过调用`restTemplate.getForObject(“[http://SERVICE-HI/hi?name=](http://service-hi/hi?name=)“+name,String.class)`方法时，已经做了负载均衡，访问了不同的端口的服务实例。


## 3.此时的架构

![](./20190113SpringCloud2服务消费者restribbon/1136672-20190112235934296-1157757573.png)


> *参考方志朋《深入理解Spring Cloud与微服务构建》*