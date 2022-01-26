---
layout: post
title: Java性能优化之String字符串优化
date: 2018-08-22 21:59:00
author: 薛师兄
tags: Java
---
> 字符串是软件开发中最重要的对象之一。通常，字符串对象在内存中是占据了最大的空间块，因此如何高效地处理字符串，必将是提高整体性能的关键所在。

## 1.字符串对象及其特点

Java中八大基本数据类型没有String类型，因为String类型是Java对char数组的进一步封装。

String类的实现主要由三部分组成：char数组，offset偏移量，String的长度。

String类型有三个基本特点：

1. 不变性

   不变性是指String对象一旦生成，则不能再对它进行改变。

   不变性的作用在于当一个对象需要被多线程共享，并且频繁访问时，可以省略同步和锁等待的时间，从而大幅提高系统性能。

2. 针对常量池的优化

   当两个String对象拥有相同的值时，它们只引用常量池中的同一个拷贝。

3. 类的final定义

   作为final类的String对象在系统中不能有任何子类，这是对系统安全性的保护！

## 1.1 subString()方法的内存泄漏

关于这一点，在JDK的1.7及以后就已经解决了！

在1.7之前，subString()方法截取字符串只是移动了偏移量，截取之后的字符串实际上还是原来的大小。

现在，当使用subString()方法截取字符串时会把截取后的字符串拷贝到新对象。

## 1.2 字符串分割与查找

#### 1、原始的String.split()

String.split()方法使用简单，功能强大，支持正则表达式，但是，在性能敏感的系统中频繁的使用这个方法是不可取的。

> 注意 * ^ : | . \ 这些符号记得\\\转义

#### 2、使用效率更高的StringTokenizer类分割字符串

StringTokenizer类是JDK中提供的专门用来处理字符串分割的工具类。构造方法：

```java
public StringTokenizer(String str, String delim, boolean returnDelims) 
```
其中str是要分割的字符串，delim是分割符，returnDelims是否返回分隔符，默认false。
```java
String s = "a;b;c";
StringTokenizer stringTokenizer = new StringTokenizer(s, ";", false);
System.out.println(stringTokenizer.countTokens());
while (stringTokenizer.hasMoreTokens()) {
    System.out.println(stringTokenizer.nextToken());
}
```

#### 3、最优化的字符串分割方式

indexOf()方法是一个执行速度非常快的方法，subString()是采用了时间换空间技术，因此速度相对快。

```java
public static List<String> mySplit(String str, String delim){
    List<String> stringList = new ArrayList<>();
    while(true) {
        int k = str.indexOf(delim);
        if (k < 0){
            stringList.add(str);
            break;
        }
        String s = str.substring(0, k);
        stringList.add(s);
        str = str.substring(k+1);
    }
    return stringList;
}
```

#### 4、三种分割方法的对比与选择

split()方法功能强大，但是效率最差；

StringTokenizer性能优于split方法，能用StringTokenizer就没必要用split()；

自己实现的分割算法性能最好，但代码的可读性和系统的可维护性最差，只有当系统性能成为主要矛盾时，才推荐使用该方法。

#### 5、高效率的charAt方法

charAt(int index) 返回指定索引处的 char 值。功能和indexOf()相反，效率却一样高。

#### 6、字符串前后辍判断

`public boolean startsWith(String prefix)`         测试此字符串是否以指定的前缀开始

`public boolean endsWith(String suffix)`             测试此字符串是否以指定的后缀结束

这两个Java内置函数效率远远低于charAt()方法。单元测试：

```java
@Test
public void test(){
    String str = "hello";
    if (str.charAt(0)=='h'&&str.charAt(1)=='e'){
        System.out.println(true);
    }
    if (str.startsWith("he")){
        System.out.println(true);
    }
}
```

## 1.3 StringBuffer和StringBuilder

#### 1、String常量的累加操作

```java
String s = "123"+"456"+"789";
```

虽然从理论上说字符串的累加的效率并不高，但该语句执行耗时为0；反编译代码后，我们发现代码是

```java
String s = "123456789";
```

显然，是Java在编译时做了充分的优化。因此，并没有想象中那样生成大量的String实例。

> 对于静态字符串的连接操作，Java在编译时会进行彻底的优化，将多个连接操作的字符串在编译时合成一个单独的长字符串。

#### 2、String变量累加的操作

      String str = "hello";
      str+="word";
      str+="!!!";
我们利用“+=”改变字符串内容的值，实际上字符串根本没有改变。

当 `str+="word"` 时，堆内存开辟了`word`的内存空间和`helloword`的两个内存空间（相当于实例化了两个String对象），并把str的引用指向了`helloword`，原来的`hello`和`word`成为了垃圾被JVM回收。

#### 3、concat() 连接字符串

String的concat()是专门用于字符串连接操作的方法，效率远远高于“+”或者“+=”。

#### 4、StringBuffer和StringBuilder

不用多说了，就是为字符串连接而生的，效率最高。不同的是，StringBuffer几乎对所有的方法都做了同步，StringBuilder并没有做任何同步，效率更高一些。只不过在多线程系统中，StringBuilder无法保证线程安全，不能使用。

#### 5、容量参数

StringBuffer和StringBuilder的是对String的封装，String是对char数组的封装。是数组就有大小，就有不够用的时候，不够用只能扩容，也就是把原来的再复制到新的数组中。合适的容量参数自然能够减少扩容的次数，达到提高效率的目的。

在初始化时，容量参数默认是16个字节。在构造方法中指定容量参数：

```java
public StringBuilder(int capacity) 
```

## 1.4 附一些实用的方法

判断字符串相等（忽略大小写）

```java
equalsIgnoreCase(String anotherString)
```

判断是否存在子字符串（返回布尔类型）

```java
contains(CharSequence s)
```

 将指定字符串连接到此字符串的结尾

```java
concat(String str)
```

使用指定的格式字符串和参数返回一个格式化字符串

```java
format(String format, Object... args)
```
使用默认语言环境的规则将此 String 中的所有字符都转换为小写。
```java
toLowerCase() 
```
使用默认语言环境的规则将此 String 中的所有字符都转换为大写。
```java
toUpperCase() 
```
返回字符串的副本，忽略前导空白和尾部空白。
```java
trim() 
```
使用给定的 replacement 替换此字符串所有匹配给定的正则表达式的子字符串。

```java
String replaceAll(String regex, String replacement)
```
按字典顺序比较两个字符串，不考虑大小写。
```java
int compareToIgnoreCase(String str)
```

- 如果参数字符串等于此字符串，则返回值 0；
- 如果此字符串小于字符串参数，则返回一个小于 0 的值；
- 如果此字符串大于字符串参数，则返回一个大于 0 的值。






> 本文参考《Java程序性能优化》葛一鸣著