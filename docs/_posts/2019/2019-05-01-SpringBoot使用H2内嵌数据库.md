---
layout: post
title: SpringBoot使用H2内嵌数据库
date: 2019-05-01 19:00:00
author: 薛师兄
tags: SpringBoot
---
## 1.驱动

我们知道，JDBC是JDK自带的接口规范，不同的数据库有不同的实现，只需要引入相应的驱动包即可。

在使用MySQL数据库时，引入的是MySQL驱动，相应的，使用H2数据库时，也需要引入H2驱动包：

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.199</version>
    <scope>runtime</scope>
</dependency>

<!--
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.35</version>
    <scope>runtime</scope>
</dependency>
-->
```

## 2.配置

在SpringBoot的application.properties文件配置相应属性：

```properties
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.url=jdbc:h2:~/folder
spring.datasource.username=root
spring.datasource.password=123456
```

JDBC URL的作用可以决定H2是用内存还是磁盘文件存储数据等，详细介绍如下：



### 本地文件

连接语法（[] 可选，<>可变）：

```java
jdbc:h2:[file:][<path>]<databaseName>
```

例如：

```java
jdbc:h2:~/test       //连接位于用户目录下的test数据库

jdbc:h2:file:/data/sample

jdbc:h2:file:E:/H2/gacl  //只在Windows下使用
```

在Window操作系统下，"~"这个符号代表的就是当前登录到操作系统的用户对应的用户目录，比如我当前是使用Administrator用户登录操作系统的，所以在"C:\Documents and Settings\Administrator\.h2"目录中就可以找到test数据库对应的数据库文件了。



### 内存数据库

连接语法：

```java
jdbc:h2:mem:<databasename>
```

示例：

```java
jdbc:h2:mem:test_mem
```



### 远程连接

这种连接方式就和其他数据库类似了，是基于Service的形式进行连接的，因此允许多个客户端同时连接到H2数据库。

连接语法：

```java
jdbc:h2:tcp://<server>[:<port>]/[<path>]<databaseName>
```

范例：

```java
jdbc:h2:tcp://localhost/~/test  //用户目录下

jdbc:h2:tcp://localhost/E:/H2/gacl  //指定目录

jdbc:h2:tcp://localhost/mem:gacl  //内存数据库
```

然后，就可以像使用MySQL一样的使用H2了。

### 扩展

[https://www.cnblogs.com/xuyatao/p/7080095.html](https://www.cnblogs.com/xuyatao/p/7080095.html)