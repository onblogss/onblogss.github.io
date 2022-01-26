---
layout: post
title: SpringCloud（6）分布式配置中心Spring Cloud Config
date: 2019-01-14 00:53:00
author: 薛师兄
tags: SpringCloud
---
## 1.Spring Cloud Config 简介

在分布式系统中，由于服务数量巨多，为了方便服务配置文件统一管理，实时更新，所以需要分布式配置中心组件。在Spring Cloud中，有分布式配置中心组件spring cloud config ，它支持配置服务放在配置服务的内存中（即本地），也支持放在远程Git仓库中。在spring cloud config 组件中，分两个角色，一是Config-Server，二是Config-Client。

## 2.Config Server从本地读取配置文件

创建一个spring-boot项目，取名为 config-server，其pom.xml完整代码.

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
    <artifactId>config-server</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>config-server</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Dalston.RELEASE</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
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

注解 @EnableConfigServer 开启配置服务器

```java
@SpringBootApplication
@EnableConfigServer //开启配置服务器
public class ConfigServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }

}
```

需要在程序的配置文件application.properties文件配置以下。通过 spring.profile.active=native 来配置 ConfigServer 从本地读取配置，读取的路径为 classpath 下的 shared 目录。

```yaml
server:
  port: 8769
spring:
  application:
    name: config-server
  profiles:
    active: native
  cloud:
    config:
      server:
        native:
          search-locations: classpath:/shared
```

在 resources 目录下新建 shared 文件夹，在 shared 文件夹下新建 config-client-dev.yml 文件。

```yaml
server:
  port: 8762
foo: foo version 1
```

启动 config-server 工程！

## 3.构建Config-Client

新建Spring Boot工程，取名为 config-client，其 pom.xml 文件为.

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
    <artifactId>config-client</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>config-client</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Dalston.RELEASE</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
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

在 resources 目录下新建 bootstrap.yml 文件，因为 bootstrap 相对于 application 具有优先的执行顺序。

变量{spring.application.name}和{spring.profiles.active}，两者以“-”相连，构成了向 Config Server 读取的配置文件名。

```yaml
spring:
  application:
    name: config-client
  cloud:
    config:
      uri: http://localhost:8769
      fail-fast: true #读取没有成功，执行快速失败
  profiles:
    active: dev
```

编写一个接口，测试读取配置文件的 foo 变量，并通过 API 接口返回.

```java
@SpringBootApplication
@RestController
public class ConfigClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigClientApplication.class, args);
    }

    @Value("${foo}")
    String foo;

    @RequestMapping(value = "/foo")
    public String hi(){
        return foo;
    }

}
```

启动 config-client 工程，访问 http://localhost:8762/foo，显示

> foo version 1 

可见 config-client 成功向 config-server 工程读取了配置文件中 foo 变量的值。

## 4.Config Server从远程Git仓库读取配置文件

修改 config-server 的配置文件 application.yml，代码如下.

```yaml
server:
  port: 8769
spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: https://github.com/forezp/SpringcloudConfig
          search-paths: respo
          username: miles02@163.com
          password:
      label: master
```
如果Git仓库为公开仓库，可以不填写用户名和密码，如果是私有仓库需要填写，本例子是公开仓库，放心使用。 

| 配置                                       | 解释                  |
| ------------------------------------------ | --------------------- |
| spring.cloud.config.server.git.uri         | 配置git仓库地址       |
| spring.cloud.config.server.git.searchPaths | 配置仓库路径          |
| spring.cloud.config.label                  | 配置仓库的分支        |
| spring.cloud.config.server.git.username    | 访问git仓库的用户名   |
| spring.cloud.config.server.git.password    | 访问git仓库的用户密码 |

远程仓库 https://github.com/forezp/SpringcloudConfig/ 中有个文件config-client-dev.properties文件中有一个属性：

> foo = foo version 2

但是没有规定 server.port 属性，所以会以默认 的 8080 启动，启动程序：访问http://localhost:8080/foo

> foo version 2

可见，config-server 从远程 Git 仓库读取了配置文件，config-client 从config-server 读取了配置文件.

## 5.构建高可用的 Config Server

将配置中心 config-server 做成一个微服务，并且将其集群化，从而达到高可用。

1.启动一个Eureka-Server工程，端口为8761，步骤参考前面的文章。

### 2.改造 config-server 

引入 spring-cloud-starter-eureka-server 起步依赖.

```xml
<dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-eureka-server</artifactId>
</dependency>
```

在工程启动类上加上注解 @EnableEurekaClient，开启 EurekaClient的功能。

在配置文件 application.yml 加入服务注册地址.

```yaml
eureka:
  client:
    service-url:
      defaultZone: http://locahost:8761/eureka/
```

### 3.改造 config-client

引入 spring-cloud-starter-eureka-server 起步依赖.

```xml
<dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-eureka-server</artifactId>
</dependency>
```

在工程启动类上加上注解 @EnableEurekaClient，开启 EurekaClient的功能。

在配置文件 application.yml 加入相关配置，向 service-id 为 config-server 的配置服务读取配置文件.

```yaml
spring:
  application:
    name: config-client
  cloud:
    config:
      fail-fast: true
      discovery:
        enabled: true
        service-id: config-server
  profiles:
    active: dev
server:
  port: 8762
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

启动 config-server、config-client 工程，访问 http://localhost:8762/foo，浏览器显:

> foo version 2 

只需要启动多个 config-server 实例即可搭建高可用的 config-server。

## 6.使用Spring Cloud Bus刷新配置

Spring Cloud Bus 将分布式的节点用轻量的消息代理连接起来。它可以用于广播配置文件的更改或者服务之间的通讯，也可以用于监控。本文要讲述的是用Spring Cloud Bus实现通知微服务架构的配置文件的更改。 

### 1.改造config-client

在pom文件加上起步依赖spring-cloud-starter-bus-amqp.

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

在工程 application 文件添加 RabbitMQ 的相关配置，包括RabbitMq的地址、端口，用户名、密码。为了方便验证，将 management.security.enabled 改为 false。

```properties
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
management.security.enabled=false
```

最后，在需要更新的配置类上加 @RefreshScope 注解。

依次启动工程，将 config-client 开启两个实例，端口分别为 8762 和 8763。启动完成后，在浏览器上访问 http://localhost:8762/foo 或者 http://localhost:8763/foo，浏览器显示:

> foo version 2

更改远程 Git 仓库，将 foo 的值改为“foo version 2”。访问 http://localhost:8762/bus/refresh 请求刷新配置，使用“destination”参数，例如 “/bus/refresh?destination=eureka-client:\*\*”，即刷新服务名为 eureka-client 的所有服务实例。