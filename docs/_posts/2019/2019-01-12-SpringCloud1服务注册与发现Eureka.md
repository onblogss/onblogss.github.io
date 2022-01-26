---
layout: post
title: SpringCloud（1）服务注册与发现Eureka
date: 2019-01-12 22:53:00
author: 薛师兄
tags: SpringCloud
---
## 1.创建1个空白的工程

## 2.创建2个model工程

一个module（即SpringBoot）工程作为服务注册中心，即Eureka Server，另一个作为Eureka Client。

Eureka Server创建完后的工程 pom.xml 文件如下：

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
    <artifactId>eureka-server</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>eureka-server</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Dalston.SR1</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka-server</artifactId>
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

## 3.启动服务注册中心

eureka是一个高可用的组件，它没有后端缓存，每一个实例注册之后需要向注册中心发送心跳（因此可以在内存中完成），在默认情况下erureka server也是一个eureka client，必须要指定一个 server。在启动之前，首先对eureka server配置application.yml文件.

```yml
server:
  port: 8761

eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

通过 eureka.client.registerWithEureka=false 和 fetchRegistry=false 来表明自己是一个eureka server。

然后再把注解 @EnableEurekaServer 加在springboot工程的启动application类上

```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaserverApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaserverApplication.class, args);
    }
}
```
Eureka Server 是有界面的，启动工程，打开浏览器访问 [http://localhost:8761](http://localhost:8761/)即可查看。

## 5.创建1个服务提供者

当client向server注册时，它会提供一些元数据，例如主机和端口，URL，主页等。Eureka server 从每个client实例接收心跳消息。 如果心跳超时，则通常将该实例从注册server中删除。

创建工程名为service-hi，创建过程同server类似，创建完pom.xml如下：

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
    <artifactId>service-hi</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>service-hi</name>
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

通过注解@EnableEurekaClient 表明自己是一个eurekaclient.

```java
@SpringBootApplication
@EnableEurekaClient
@RestController
public class ServiceHiApplication {

    public static void main(String[] args) {
        SpringApplication.run(ServiceHiApplication.class, args);
    }

    @Value("${server.port}")
    String port;
    @RequestMapping("/hi")
    public String home(@RequestParam String name) {
        return "hi "+name+",i am from port:" +port;
    }

}
```

仅仅@EnableEurekaClient是不够的，还需要在配置文件中注明自己的服务注册中心的地址，application.yml配置文件如下：

```yml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
server:
  port: 8763
spring:
  application:
    name: service-hi
```

需要指明spring.application.name，这个很重要，这在以后的服务与服务之间相互调用一般都是根据这个name 。 
启动工程，打开[http://localhost:8761](http://localhost:8761/) ，即eureka server 的网址，你会发现一个服务已经注册在服务中了，服务名为SERVICE-HI，端口为7862。

这时打开 <http://localhost:8763/hi?name=forezp> ，你会在浏览器上看到 :

> hi forezp,i am from port:8763

## 6.构建高可用的Eureka Server集群

更改 eureka-server 的配置文件，再分别以 spring.profiles.active=peer1 和 peer2 启动两次eureka-serve工程。

```yml
spring:
  profiles:
    active: peer1 #peer2

---

spring:
  profiles: peer1
server:
  port: 8761
eureka:
  instance:
    hostname: localhost
  client:
    serviceUrl:
      defaultZone: http://127.0.0.1:8762/eureka/

---

spring:
  profiles: peer2
server:
  port: 8762
eureka:
  instance:
    hostname: 127.0.0.1
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

再启动 eureka-client 工程。

访问 http://localhost:8761/ 可以发现，eureka-client 已经向8761端口的 eureka-server 注册了，并且在DS Replicas选项中显示了节点127.0.0.1，界面这里不展示。

这时 eureka-client 工程的配置文件并没有向端口为8762的 eureka-server 注册。访问 http://127.0.0.1:8762/ 可以发现，eureka-client 也已经向 127.0.0.1:8762 注册了，可见 localhost:8761 的注册信息已经同步到了 127.0.0.1:8762 节点。

也就是说，端口为8761和8762的两台 Eureka-server 相互感应，当有服务注册时，两个Eureka-eserver是对等的，它们都存有相同的信息，这就是通过服务器的冗余来增加可靠性，当有一台服务器宕机了，服务并不会终止，因为另一台服务存有相同的数据。

> *参考方志朋《深入理解Spring Cloud与微服务构建》*