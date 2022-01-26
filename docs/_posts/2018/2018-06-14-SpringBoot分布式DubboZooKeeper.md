---
layout: post
title: SpringBoot分布式 - Dubbo+ZooKeeper
date: 2018-06-14 21:21:00
author: 薛师兄
tags:
  - SpringBoot
  - Dubbo
  - ZooKeeper
---
# 一：介绍

*  ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务。它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务等。
*  Dubbo是Alibaba开源的分布式服务框架，它最大的特点是按照分层的方式来架构，使用这种方式可以使各个层之间解耦合（或者最大限度地松耦合）。从服务模型的角度来看，Dubbo采用的是一种非常简单的模型，要么是提供方提供服务，要么是消费方消费服务，所以基于这一点可以抽象出服务提供方（Provider）和服务消费方（Consumer）两个角色。

---

# 二：入门

## 1.准备：ZooKeeper安装

**步骤1：下载ZooKeeper**

*  要在你的计算机上安装ZooKeeper框架，请访问以下链接并下载最新版本的ZooKeeper。[http://zookeeper.apache.org/releases.html](http://zookeeper.apache.org/releases.html)

*  到目前为止，最新版本的ZooKeeper是3.4.6(ZooKeeper-3.4.6.tar.gz)。

**步骤2：提取tar文件**

*  使用以下命令提取tar文件

```java
$ cd opt/
$ tar -zxf zookeeper-3.4.6.tar.gz
$ cd zookeeper-3.4.6
$ mkdir data
```

**步骤3：创建配置文件**

*  使用命令 vi conf/zoo.cfg 和所有以下参数设置为起点，打开名为 conf/zoo.cfg 的配置文件。

```java
<span class="hljs-variable">$ </span>vi conf/zoo.cfg
```

```java
tickTime = 2000
dataDir = /path/to/zookeeper/data
clientPort = 2181
initLimit = 5
syncLimit = 2
```

*  一旦成功保存配置文件，再次返回终端。你现在可以启动zookeeper服务器。

**步骤4：启动ZooKeeper服务器**

*  执行以下命令

```java
<span class="hljs-variable">$ </span>bin/zkServer.sh start
```

*  执行此命令后，你将收到以下响应

```java
$ JMX enabled by default
$ Using config: /Users/../zookeeper-3.4.6/bin/../conf/zoo.cfg
$ Starting zookeeper ... STARTED
```

---

## 2.编程 - 需要的依赖

```java
<!--Dubbo与SpringBoot的集成 -->
<dependency>
    <groupId>com.alibaba.boot</groupId>
    <artifactId>dubbo-spring-boot-starter</artifactId>
    <version>0.1.0</version>
</dependency>
<!-- zookeeper客户端 -->
<dependency>
    <groupId>com.github.sgroschupf</groupId>
    <artifactId>zkclient</artifactId>
    <version>0.1</version>
</dependency>
```

---

## 3. 在IDEA新建空白工程

*  spring-boot版本为1.5
*  需要建立两个模块，服务提供者与服务消费者

### 3.1 服务提供者

*  新建模块provider-ticket并导入步骤2的包

*  1）在目录下新建service包，在该包下新建一个接口类

```java
package com.example.ticket.service;

public interface TicketService {
    String getTicket();
}
```

*  2）实现该接口方法

```java
package com.example.ticket.service;

import com.alibaba.dubbo.config.annotation.Service;
import org.springframework.stereotype.Component;

/**
 * Create by Martin 2018/6/4/004 15:57
 */
@Component  //Spring注解
@Service    //dubbo的注解
public class TicketServiceImpl implements TicketService {
    @Override
    public String getTicket() {
        return "<<厉害了，我的国>>";
    }
}
```

*  3）application.yml

```java
server:
  port: 8082
dubbo:
  application:
    name: provider-ticket
  registry:
    address: zookeeper://127.0.0.1:2181
  scan:
    base-packages: com.example.ticket.service
```

**总结**

```java
将服务提供者注册到注册中心-->
1.引入Dubbo和Zookeeper的相关依赖
2.配置Dubbo的扫描包和注册中心地址
3.使用@Service发布服务
```

---

### 3.2 服务消费者

*  新建模块consumer-user并导入步骤2的包

*  1）配置application.yml

```java
server:
  port: 8081
dubbo:
  application:
    name: consumer-user
  registry:
    address: zookeeper://127.0.0.1:2181
```

*  2）拷贝服务端的接口类

```java
package com.example.ticket.service;

/*必须保证客户端与服务端的类路径一致，只保留该接口类即可。*/

public interface TicketService {
    String getTicket();
}
```

*  3）使用服务端接口

```java
package com.example.user.service;

import com.example.ticket.service.TicketService;
import com.alibaba.dubbo.config.annotation.Reference;
import org.springframework.stereotype.Service;

@Service
public class UserSerivce{
    @Reference   /*引用接口需注解dubbo的@Reference*/
    private TicketService ticketService;

    public void getHello(){
        String ticket = ticketService.getTicket();
        System.out.println("买到票了：" + ticket);
    }
}
```

*  4）测试：服务消费者消费服务

```java
package com.example;

import com.example.user.service.UserSerivce;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class ConsumerUserApplicationTests {
    @Autowired
    private UserSerivce userSerivce;

    @Test
    public void contextLoads() {
        userSerivce.getHello();
    }

}
```

---