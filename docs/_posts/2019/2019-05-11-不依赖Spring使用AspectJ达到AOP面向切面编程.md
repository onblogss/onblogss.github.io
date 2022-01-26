---
layout: post
title: 不依赖Spring使用AspectJ达到AOP面向切面编程
date: 2019-05-11 13:32:00
author: 薛师兄
tags: AOP
---
网上大多数介绍AspectJ的文章都是和Spring容器混用的，但有时我们想自己写框架就需要抛开Spring造轮子，类似使用原生AspectJ达到面向切面编程。步骤很简单，只需要两步。

## 1.导入依赖

```xml
<dependency>
     <groupId>org.aspectj</groupId>
     <artifactId>aspectjweaver</artifactId>
     <version>1.9.3</version>
</dependency>
```

## 2.Maven插件

```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>aspectj-maven-plugin</artifactId>
    <version>1.10</version>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
        <complianceLevel>1.8</complianceLevel>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>compile</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

## 3.使用注解

```java
@Aspect
public class AspectDemo {

    @Pointcut("execution(* com.example.App.say())")
    private void pointcut() {}  // signature

    @Before("pointcut()")
    public void before(){
        System.out.println("Hello");
    }
}
```
App.java
```java
public class App {

    public static void main( String[] args ) {
        System.out.println( new App().say() );
    }

    public String say() {
        return "World";
    }
}
```

这一步就和平常使用Spring AOP注解没有什么区别了。

## 4.织入/代理

我们都知道，Spring AOP是通过动态代理生成一个代理类，这种方式的最大缺点就是对于对象内部的方法嵌套调用不会走代理类，比如下面这段代码：

```java
@Component
public class TestComponent {

    @TestAspect
    public void work(){
        //do sth
    }

    public void call(){
        work();
    }
}
```

原因很简单，对象内部的方法调用该对象的其他方法是通过自身this进行引用，并不是通过代理类引用。而AspectJ则不同，AspectJ是通过织入的方式将切面代码织入进原对象内部，并不会生成额外的代理类。关于这一点，我们反编译看一下切点代码：

```java
//原方法
public void say() {
    System.out.println(this.getClass().getName());
    hi();
}
//反编译
public void say() {
    ResourceAspect.aspectOf().before();
    System.out.println(this.getClass().getName());
    this.hi();
}
```

深究下去，在Spring AOP中，我们只有调用代理类的切点方法才能触发Before方法，因为代理类本质上是对原类的一层封装，原类是没有变化的，原类的方法内部的this指向的依旧是原类，这就导致了原类方法内部的嵌套调用无法被代理类感知到，而AspectJ的织入就不同了，它会动态改变你的原类代码，将Before等方法全部写入进你的原方法中，这就保证了面向切面编程的万无一失。两种方式，各有利弊，如何使用还需要视情况而行。

关于更多的AspectJ的介绍，可以参考下面这一篇，写的相当不错。

[原生AspectJ用法分析以及spring-aop原理分析](https://blog.mythsman.com/2017/12/21/1/)