---
layout: post
title: 2小时学会SpringBoot（IDE：eclipse）
date: 2018-02-28 22:21:00
author: 薛师兄
tags: SpringBoot
---
# 一：安装STS插件

官网下载：[点此下载STS](https://spring.io/tools/sts/all)

注意：STS版本必须与eclipse版本对应

安装教程：[http://blog.csdn.net/cryhelyxx/article/details/53894405](http://blog.csdn.net/cryhelyxx/article/details/53894405)

# 二：新建Spring boot项目

1. 文件 &ndash;> 新建 &ndash;> Spring Starter Project
2. 填写类似Mvane项目的一些配置，下一步
3. 选择依赖：我们只勾选**web**即可

# 三：项目开发

## 1.application.properties与application.yml

1.使用application.properties（默认）

```properties
server.port=8081
server.context-path=/demo
```

2.使用application.yml（手动创建 | 推荐）

```yaml
server:
  port: 8080
  context-path: /demo
```

2.1 新建application-dev.yml

```yaml
server:
  port: 8080
  context-path: /demo
#以下为自定义变量
girl:
  cupSize: A
  age: 18
content: "cupSize: ${cupSize}, age: ${age}"
```

2.2 新建application.yml

```yaml
#指定使用哪个子配置
spring:
  profiles:
    active: 
    - prod
#配置数据库信息
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: 123456
  jpa:
    show-sql: true
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
        format_sql: true
        hibernate.dialect: org.hibernate.dialect.MySQL5Dialect
```

## 2.常用pom.xml依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>spring-boot-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>spring-boot-demo</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.10.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <!-- web项目 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- 模板引擎 ：用于Controller返回html页面
            可以在配置文件修改默认的
            spring.thymeleaf.prefix: /templates/  
            spring.thymeleaf.suffix: .html  
            -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>

        <!-- 数据库方面 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>

        <!-- AOP面向切面 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>

        <!-- 使用 (项目名Properties.class) 类加载application.yml中的自定义变量 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- 默认 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

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

## 3.如何使用yml中自定义变量

1.第一种方法

```java
@Value("${cupSize}")
private String cupSize;
```

2.第二种方法

```java
package com.example.properties;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix="girl")//对应yml配置
public class SpringBootDemoProperties {

    private String cupSize;

    private int age;

    public String getCupSize() {
        return cupSize;
    }

    public void setCupSize(String cupSize) {
        this.cupSize = cupSize;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

}
```

使用时

```java
@Autowired
private SpringBootDemoProperties sbdp;
....
sbdp.getCupSize()
```

## 4.如何使用Controller

```java
package com.example.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import com.example.properties.SpringBootDemoProperties;

@RestController     //返回json 替代@ResponseBody加@Controller
//@Controller   返回页面
public class HelloController {

    //获取url参数
    //@RequestMapping(value="/hello/{id}", method=RequestMethod.GET)
    @GetMapping(value="/hello/{id}")
    public String say(@PathVariable(name="id") Integer id){
        return id+"";
    }

    //获取请求参数
    @RequestMapping(value={"/hello","/hi"}, method=RequestMethod.GET)   //required= false参数不是必有
    public String sayy(@RequestParam(name="id", required= false, defaultValue="0") Integer id){
        return id+"";
    }

}
```

## 5.注释映射数据库表

需要在application.yml配置

```java
package com.example.entity;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.Table;
import javax.validation.constraints.Min;

import org.hibernate.validator.constraints.Length;

@Entity
@Table(name="girl")
public class Girl {
    @Id
    @GeneratedValue
    private int id;
    @Min(value=18, message="未成年少女禁止入内")
    private Integer age;
    @Length(min=1,max=1, message="长度必须等于1")
    private String cupSize;

    public Girl() {
        super();
    }

    public Girl(int id, Integer age, String cupSize) {
        super();
        this.id = id;
        this.age = age;
        this.cupSize = cupSize;
    }


    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }

    public Integer getAge() {
        return age;
    }
    public void setAge(Integer age) {
        this.age = age;
    }
    public String getCupSize() {
        return cupSize;
    }
    public void setCupSize(String cupSize) {
        this.cupSize = cupSize;
    }
    @Override
    public String toString() {
        return "Girl [id=" + id + ", age=" + age + ", cupSize=" + cupSize + "]";
    }

}
```

## 5.如何配置dao层访问数据库

创建一个接口类即可

```java
import java.util.List;

import org.springframework.data.jpa.repository.JpaRepository;

import com.example.entity.Girl;

public interface GirlRepository extends JpaRepository<Girl, Integer> {
    /**
     * 通过年龄来查询
     * @return
     */
    public List<Girl> findByAge(Integer age);
}
```

使用方法

```javascript
@Autowired
private GirlRepository girlRepository;
....
girlRepository.save(girl);  
girlRepository.findOne(id);
```

## 6.事务管理

```java
/**
 * 事务管理测试
 */
@Transactional
public void insertTwo(){
    Girl girl = new Girl();
    girl.setAge(19);
    girl.setCupSize("F");   
    girlRepository.save(girl);  

    girl = new Girl();
    girl.setAge(20);
    girl.setCupSize("BB");  
    girlRepository.save(girl);  
}
```

## 7.使用AOP面向切面处理请求

```java
package com.example.aspect;

import javax.servlet.http.HttpServletRequest;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

/**
* Created by 巅峰小学生
* 2018年2月27日 下午5:26:41
*/
@Aspect
@Component
public class HttpAspect {
    //开启日志
    private final static Logger logger = LoggerFactory.getLogger(HttpAspect.class);

    //声明切点
    @Pointcut("execution(public * com.example.controller.GirlController.girlFindAll(..))")
    public void pointcut(){}

    //前置通知
    @Before("pointcut()")
    public void before(JoinPoint joinPoint){
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();
        logger.info("url={}", request.getRequestURI());
        logger.info("method={}", request.getMethod());
        logger.info("ip={}", request.getRemoteAddr());
        //类方法
        logger.info("class_method={}", joinPoint.getSignature().getDeclaringTypeName() + "." + joinPoint.getSignature().getName() );
        //参数
        logger.info("args={}", joinPoint.getArgs());
    }

    //后置通知
    @After("pointcut()")
    public void after(){
        logger.info("后置通知");
    }

    //返回通知
    @AfterReturning(pointcut="pointcut()" ,returning="retrunValue")
    public void afterReturning(Object retrunValue){
        logger.info("response={}", retrunValue.toString());
    }

}
```

## 8.统一异常处理

1）自定义异常类

```java
package com.example.exception;

import com.example.enums.ResultEnum;

/**
* Created by 巅峰小学生
* 2018年2月28日 下午4:17:42
* --自定义异常类
*/
public class GirlException extends RuntimeException {
    private Integer code;

    public GirlException(Integer code, String message) {
        super(message);
        this.code=code;
    }

    public GirlException(ResultEnum resultEnum) {
        super(resultEnum.getMsg());
        this.code=resultEnum.getCode();
    }

    public Integer getCode() {
        return code;
    }

    public void setCode(Integer code) {
        this.code = code;
    }

}
```

2）自定义异常信息

```java
package com.example.enums;
/**
* Created by 巅峰小学生
* 2018年2月28日 下午4:54:43
*/
public enum ResultEnum {
    UNKNOW_ERROR(-1, "未知错误"),
    SUCCESS(0, "成功"),
    PRIMARY_SCHOOL(100, "你可能还在上小学" ),
    MIDDLE_SCHOOL(101, "你可能还在上初中"),
    NO_SCHOOL(102, "你可能不上学了"),
    ;

    private Integer code;
    private String msg;

    private ResultEnum(Integer code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    public Integer getCode() {
        return code;
    }

    public String getMsg() {
        return msg;
    }

}
```

3）在需要的地方抛出异常

```java
/**
 * @param id
 * @throws Exception
 */
public void getAge(int id) throws Exception{
    Girl girl = girlRepository.findOne(id);
    Integer age = girl.getAge();
    if(age < 10){
        throw new GirlException(ResultEnum.PRIMARY_SCHOOL);
    }else if(age>10 && age<16){
        throw new GirlException(ResultEnum.MIDDLE_SCHOOL);
    }else{
        throw new GirlException(ResultEnum.NO_SCHOOL);
    }
}
```

4）定义异常捕获类（核心类：上面3步可忽略，直接定义该类即可使用）

```java
package com.example.handle;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

import com.example.controller.GirlController;
import com.example.entity.Result;
import com.example.exception.GirlException;
import com.example.util.ResultUtil;

/**
* Created by 巅峰小学生
* 2018年2月28日 下午3:51:40
*/
@ControllerAdvice
public class ExceptionHandle {

    //记录日志
    private final static Logger logger = LoggerFactory.getLogger(GirlController.class);

    /**
     * 捕获异常 封装返回数据
     * @return
     */
    @ExceptionHandler(value = Exception.class)
    @ResponseBody
    public Result<?> handle(Exception e){
        if(e instanceof GirlException){
            return new Result(((GirlException) e).getCode(), e.getMessage());
        }else{
            logger.info("[系统异常] {}",e);
            return new Result(ResultEnum.UNKNOW_ERROR.getCode(), e.getMessage());
        }
    }
}
```

## 9.部署在Tomcat服务器

1.）使启动类继承SpringBootServletInitializer 覆写configure()方法。

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.support.SpringBootServletInitializer;

@SpringBootApplication
public class SpringBootDemoApplication extends SpringBootServletInitializer{

    public static void main(String[] args) {
        SpringApplication.run(SpringBootDemoApplication.class, args);
    }

     @Override
     protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
            return builder.sources(SpringBootDemoApplication.class);
      }
}
```

2.）修改pom.xml打包方式为war

```xml
<packaging>war</packaging>
```

3.）确保嵌入servlet容器不干扰外部servlet容器部署war文件

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-tomcat</artifactId>
   <scope>provided</scope>
</dependency>
```

> 若war在部署到容器中时遇到Project facet Cloud Foundry Standalone Application version 1.0 is not supported.错误；   解决办法: 项目右键Build Path -> Configure Build Path -> Project facet -> 勾掉Cloud Foundry Standalone Application