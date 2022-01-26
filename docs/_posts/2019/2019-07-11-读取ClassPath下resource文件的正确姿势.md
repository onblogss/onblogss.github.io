---
layout: post
title: 读取ClassPath下resource文件的正确姿势
date: 2019-07-11 23:01:00
author: 薛师兄
tags: Java
---
## 1.前言

为什么要写这篇文章？

身为Java程序员你有没有过每次需要读取 ClassPath 下的资源文件的时候，都要去百度一下，然后看到下面的这种答案：

```java
Thread.currentThread().getContextClassLoader().getResource("ss.properties").getPath();
```

亦或是：

```java
Object.class.getResourceAsStream("ss.properties")；
```

你复制粘贴一下然后放到自己的项目里运行，还真跑起来了。但是当打成 jar 包作为其它项目的依赖时，或者打成 war 包被 Tomcat 加载时，你还能保证你的resources 资源文件被读取到吗？

答案是不能的。

其中的原因如何而又如何解决，怎样才能写出万无一失根本不用担心任何环境的代码？下面我会一一道来。

## 2.再看类加载机制

看到这个标题你也许会有些意外，不是说的读取ClassPath下的文件吗？为什么要讲类加载机制。

那你有没有想过，ClassPath下的资源文件标准存放的是什么？

顾名思义，是 .class 类文件。为什么我们的类可以被正确加载到Java虚拟机（JVM），而自己添加的资源文件却加载失败呢？归根结底是我们没有理解类加载机制，也就无法做到举一反三。

### 类加载机制与类加载器

程序员将源代码写入.Java文件中，经过（javac）编译，生成.class二进制文件。虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的Java类型，这就是虚拟机的类加载机制。

从宏观上理解了类加载机制后，接下来就要从细节上说一说类加载器，以及类加载器的工作原理。

类加载器，顾名思义，是加载类的器件。JVM只存在两种不同的类加载器：启动类加载器（Bootstrap ClassLoader），使用C++实现，是虚拟机自身的一部分。另一种是所有其他的类加载器，使用JAVA实现，独立于JVM，并且全部继承自抽象类java.lang.ClassLoader。包括扩展类加载器、应用程序类加载器。

它我们在写代码时，总是会new很多对象，我们之所以可以new出对象，是因为该对象对应的类已经被JVM加载为Class类的对象实例。这句话有点绕，我用代码展示一下：

```java
Obj obj = new Obj(); //Obj对象实例
Class o = obj.getClass(); //Obj类是Class类的对象实例
```

在JVM中，一般情况下，我们的类的类实例是唯一的，这得益于类加载机制的双亲委派模型。

如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都是应该传送到顶层的启动类加载器中，只有当父类加载器反馈自己无法完成这个加载请求（它的搜索范围中没有找到所需的类）时，子加载器才会尝试自己去加载。

## 3.类也是一种Resource

言归正传，通过上述对类加载机制的学习，我们可以得出这样的一个结论：一个类文件是由某个类加载器负责加载到JVM中的，且只会有一个类加载器去加载。反过来说，由一个类实例就可以获取到加载它到JVM中的那个类加载器。

用代码阐述我的上段话如下所示：

```java
Obj obj = new Obj();
ClassLoader classLoader = obj.getClass().getClassLoader();
```

跟着思路继续走，该类加载器之所以可以加载这个类，是因为这个类在该类加载器的搜索范围内。类加载器既然可以加载这个类文件，那么也可以加载该类文件同级目录下的所有资源文件。

所以，我们要想确保可以读取到某个资源文件，**只需调用和该资源文件在同一目录下的类的Class对象的getClassLoader()方法获取该类加载器即可**。

举个例子，我们有一个properties文件和Obj.class在同一个目录下， 那我们读取该properties文件的最正确的方式就是通过`Obj.class.getClassLoader().getResourceAsStream()`方法。

## 4.一个错误的例子

为了印证上面的结论，先看下 `Object.class.getResourceAsStream()` 的源码：

```java
// Class.java
public InputStream getResourceAsStream(String name) {
    name = resolveName(name);
    ClassLoader cl = getClassLoader0();
    if (cl==null) {
        // A system class.
        return ClassLoader.getSystemResourceAsStream(name);
    }
    return cl.getResourceAsStream(name);
}
```

从 Javadoc 文档和源码中可以看出：

Class.getResourceAsStream() 代理给了加载该 class 的 ClassLoader 去实现，调用 classLoader.getResourceAsStream()，如果该类的 ClassLoader 为 null，说明该 class 一个系统 class，所以委托给 ClassLoader.getSystemResourceAsStream。

这一点也印证了之前讲解的：**资源文件都是由ClassLoader负责加载的，类也是一种resources文件**。

但通过`Object.class.getResourceAsStream()`不一定可以搜索到指定的资源文件，原因就在于前面说过的类加载器的搜索范围，所以这种方式并不推荐使用。

## 5.FileHelper

最后推荐一个操作Resources资源的框架FileHelper：

[https://github.com/onblog/FileHelper](https://github.com/onblog/FileHelper)

读取Resources下的资源

```java
ClassPathResource resource = new ClassPathResource();
String html = resource.readString("commons.html",StandardCharsets.UTF_8);
String htm = resource.readString("commons.htm");
byte[] bytes = resource.readByte("commons.html");
InputStream inputStream = resource.read("commons.html");
String resourcePath = resource.getPath(); //获取resources根目录
```

关于如何正确读取ClassPath下的资源文件相信你已经掌握了正确姿势。