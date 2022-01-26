---
layout: post
title: 使用try-with-resources优雅的关闭IO流
date: 2019-04-17 18:49:00
author: 薛师兄
tags: Java
---
Java类库中包括许多必须通过调用close方法来手工关闭的资源。例如InputStream、OutputStream和java.sql.Connection。客户端经常会忽略资源的关闭，造成严重的性能后果也就可想而知了。根据经验，try-finally 语句是确保资源会被适当关闭的最佳方法，就算是发生异常或者返回也一样：

```java
public String tryfinally(String path) throws IOException {
    BufferedReader reader = new BufferedReader(new FileReader(new File(path)));
    try{
        return reader.readLine();
    }finally {
        reader.close();
    }
}
```

即便用 try-finally 语句正确地关闭了资源，它也存在着些许不足。因为在try块和finally块中的代码，都会抛出异常。例如，底层的物理设备出现异常，那么调用readLine就会抛出异常，基于同样的原因，调用close也会出现异常。在这种情况下，第二个异常完全抹除了第一个异常。在异常堆栈轨迹中，完全没有关于第一个异常的记录，这在现实的系统中会导致调试变得非常复杂，因为通常需要看到第一个异常的记录，这在现实的系统中会导致调试变得非常复杂，因为通常需要看到第一个异常才能诊断出问题何在。虽然可以通过编写代码来禁止第二个异常，保留第一个异常，但事实上没有人会这么做，因为实现起来太繁琐了。

**如何完美解决这种问题呢？当然是Java7中引入的 try-with-resources 语句。**

还是上面的这段代码，使用 try-with-resources 语句之后，新的代码如下：

```java
public String tryresources(String path) throws IOException {
    try(BufferedReader reader = new BufferedReader(new FileReader(new File(path)))){
        return reader.readLine();
    }
}
```

使用这种 try-with-resources 不仅使代码变得简洁易懂，也更容易进行诊断。以上段代码为例，如果调用 readLine 和 不可见的 close 方法都抛出异常，后一个异常就会被禁止，以保留第一个异常。事实上，为了保留你想看到的那个异常，即便多个异常都可以被禁止。这些被禁止的异常并不是简单地被抛弃了，而是会被打印在堆栈轨迹中，并注明它们是被禁止的异常。通过编程调用 getSuppressed 方法还可以访问到它们，getSuppressed 方法也已经添加在Java7的Throwable中了。

**使用 try-with-resources 语句有什么要求吗？**

是的，要使用这个构造的资源，必须先实现 AutoCloseable 接口，其中包含了单个返回 void 的 close 方法。Java类库与第三方类库中的许多类和接口，现在都实现或扩展了 AutoCloseable 接口。

```java
public interface AutoCloseable {
    void close() throws Exception;
}
```

以下是使用 try-with-resources 的第二个范例：

```java
void tryresources(String src,String dst) throws IOException {
     try(InputStream in = new FileInputStream(src);
        OutputStream out = new FileOutputStream(dst)){
        byte[] bytes = new byte[1024];
        int n;
        while ((n=in.read(bytes))>=0){
            out.write(bytes,0,n);
        }
     }
}
```

在 try-with-resources 语句中还可以使用 catch 子句，就像在平时的 try-finally 语句中一样。

```java
public String tryresources(String path){
   try(BufferedReader reader = new BufferedReader(new FileReader(new File(path)))){
        return reader.readLine();
   } catch (IOException e) {
         return null;
   }
}
```

有了 try-with-resources 语句，在使用必须关闭的资源时，就能更轻松地正确编写代码了。