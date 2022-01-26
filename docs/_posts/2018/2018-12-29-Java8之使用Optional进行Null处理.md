---
layout: post
title: Java8之使用Optional进行Null处理
date: 2018-12-29 15:12:00
author: 薛师兄
tags: Java
---
![](./20181229Java8之使用Optional进行Null处理/1136672-20181229151420068-1277416757.png)


Optional类这是Java 8新增的一个类，用以解决程序中常见的`NullPointerException`异常问题，本篇文章将详细介绍`Optional`类，以及如何用它消除代码中的`null`检查。

## 1.创建optional对象

*empty()* 方法用于创建一个没有值的Optional对象：

```java
Optional<String> emptyOpt = Optional.empty();
```

*of()* 方法使用一个非空的值创建Optional对象：

```java
String str = "Hello World";
Optional<String> notNullOpt = Optional.of(str);
```

*ofNullable()* 方法接收一个可以为null的值：

```java
Optional<String> nullableOpt = Optional.ofNullable(str);
```

## 2.判断Null

*isPresent()*

如果创建的对象没有值，调用`isPresent()`方法会返回`false`，调用`get()`方法抛出`NullPointerException`异常。

## 3.获取对象

*get()*

## 4.使用map提取对象的值

如果我们要获取`User`对象中的`roleId`属性值，常见的方式是直接获取：

```java
String roleId = null;
if (user != null) {
    roleId = user.getRoleId();
}
```

使用`Optional`中提供的`map()`方法可以以更简单的方式实现：

```java
Optional<User> userOpt = Optional.ofNullable(user);
Optional<String> roleIdOpt = userOpt.map(User::getRoleId);
```

## 5.使用orElse方法设置默认值

`Optional`类还包含其他方法用于获取值，这些方法分别为：

- `orElse()`：如果有值就返回，否则返回一个给定的值作为默认值；
- `orElseGet()`：与`orElse()`方法作用类似，区别在于生成默认值的方式不同。该方法接受一个`Supplier<? extends T>`函数式接口参数，用于生成默认值；
- `orElseThrow()`：与前面介绍的`get()`方法类似，当值为`null`时调用这两个方法都会抛出`NullPointerException`异常，区别在于该方法可以指定抛出的异常类型。

```java
String str = "Hello World";
Optional<String> strOpt = Optional.of(str);
String orElseResult = strOpt.orElse("Hello Shanghai");
String orElseGet = strOpt.orElseGet(() -> "Hello Shanghai");
String orElseThrow = strOpt.orElseThrow(
        () -> new IllegalArgumentException("Argument 'str' cannot be null or blank."));
```

## 6.使用filter()方法过滤

`filter()`方法可用于判断`Optional`对象是否满足给定条件，一般用于条件过滤：

```java
Optional<String> optional = Optional.of("wangyi@163.com");
optional = optional.filter(str -> str.contains("164"));
```

在上面的代码中，如果`filter()`方法中的Lambda表达式成立，`filter()`方法会返回当前`Optional`对象值，否则，返回一个值为空的`Optional`对象。

## 7.使用建议

1. 尽量避免在程序中直接调用`Optional`对象的`get()`和`isPresent()`方法；
2. 避免使用`Optional`类型声明实体类的属性；

## 8.正确示例

**orElse()方法的使用**

```java
return str != null ? str : "Hello World"
```

上面的代码表示判断字符串`str`是否为空，不为空就返回，否则，返回一个常量。使用`Optional`类可以表示为：

```java
return strOpt.orElse("Hello World")
```

**简化if-else**

```java
User user = ...
if (user != null) {
    String userName = user.getUserName();
    if (userName != null) {
        return userName.toUpperCase();
    } else {
        return null;
    }
} else {
    return null;
}
```

上面的代码可以简化成：

```java
User user = ...
Optional<User> userOpt = Optional.ofNullable(user);

return user.map(User::getUserName)
            .map(String::toUpperCase)
            .orElse(null);
```

> 本文参考自：[https://lw900925.github.io/java/java8-optional.html](https://lw900925.github.io/java/java8-optional.html)