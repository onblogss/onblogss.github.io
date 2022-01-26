---
layout: post
title: 十分钟学会Java8：lambda表达式和Stream API
date: 2018-10-01 17:47:00
author: 薛师兄
tags:
  - Java
  - lambda
---
Java8 的新特性：Lambda表达式、强大的 Stream API、全新时间日期 API、ConcurrentHashMap、MetaSpace。总得来说，Java8 的新特性使 Java 的运行速度更快、代码更少、便于并行、最大化减少空指针异常。

本篇博客将以笔者的一些心得帮助大家快速理解lambda表达式和Stream API.

# 一：lambda

## 1.引言

在IDE中，你是否遇到在写以下列代码时，被友情提示的情况：

```java
new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("thread");
    }
});
```

这时候，我们按一下快捷键，IDE自动帮我们把代码优化为酱个样子：

```java
	new Thread(() -> System.out.println("thread"));
```

这就是Java8的新特性：lambda表达式

## 2.lambda表达式

借用引言中的示例，在调用new Thread的含参构造方法时，我们通过匿名内部类的方式实现了Runnable对象，但其实有用的代码只有`System.out.println("thread");`这一句，而我们却要为了这一句去写这么多行代码。正是这个问题，才有了Java8中的lambda表达式。那lambd表达式究竟是如何简化代码的呢？

先来看lambda表达式的语法：

```java
() -> {}
```

() : 括号就是接口方法的括号，接口方法如果有参数，也需要写参数。只有一个参数时，括号可以省略。

-> : 分割左右部分的，没啥好讲的。

{} : 要实现的方法体。只有一行代码时，可以不加括号，可以不写return。

看了上面的解释，也就不难理解IDE优化后的代码了。不过看到这里你也许意识到，如果接口中有多个方法时，按照上面的逻辑lambda表达式恐怕不行了。没错，lambda表达式只适用于函数型接口。说白了，函数型接口就是只有一个抽象方法的接口。这种类型的接口还有一个对应的注解：`@FunctionalInterface`

为了让我们在需要这种接口时不再自己去创建，Java8中内置了四大核心函数型接口：

消费型接口（有参无返回值）

```java
Consumer<T>

void accept(T t);
```

供给型接口（无参有返回值）

```java
Supplier<T>

T get();
```

函数型接口（有参有返回值）

```java
Function<T, R>

R apply(T t);
```

断言型接口（有参有布尔返回值）

```java
Predicate<T>

boolean test(T t);
```

看到这里如果遇到一般的lambda表达式，你应该可以从容面对了，但高级点的恐怕看到还是懵，不要急，其实也不难。

### 方法引用

lambda表达式还有两种简化代码的手段，它们是**方法引用**、**构造引用**。

方法引用是什么呢？如果我们要实现接口的方法与另一个方法A类似，（这里的类似是指参数类型与返回值部分相同），我们直接声明A方法即可。也就是，不再使用lambda表达式的标准形式，改用高级形式。无论是标准形式还是高级形式，都是lambda表达式的一种表现形式。

举例：

```java
Function function1 = (x) -> x;
Function function2 = String::valueOf;
```

对比Function接口的抽象方法与String的value方法，可以看到它们是类似的。

```java
R apply(T t);

public static String valueOf(Object obj) {
    return (obj == null) ? "null" : obj.toString();
}
```

方法引用的语法：

```java
对象::实例方法
类::静态方法
类::实例方法
```

前两个很容易理解，相当于对象调用实例方法，类调用静态方法一样。只是第三个需要特殊说明。

当出现如下这种情况时：

```java
Compare<Boolean> c = (a, b) -> a.equals(b);
```

用lambda表达式实现Compare接口的抽象方法，并且方法体只有一行，且该行代码为参数1调用方法传入参数2。此时，就可以简化为下面这种形式：

```java
Compare<Boolean> c = String::equals;
```

也就是“类::实例方法”的形式。

值得一提的是，当参数b不存在时，该方式依旧适用。例如：

```java
Function function1 = (x) -> x.toString();

Function function1 = Object::toString;
```

### 构造引用

先来创建一个供给型接口对象：

```java
Supplier<String> supplier = () -> new String();
```

在这个lammbda表达式中只做了一件事，就是返回一个新的Test对象，而这种形式可以更简化：

```java
Supplier<String> supplier = String::new;
```

提炼一下构造引用的**语法**：

```java
类名::new
```

当通过含参构造方法创建对象，并且参数列表与抽象方法的参数列表一致，也就是下面的这种形式：

```java
Function1 function = (x) -> new String(x);
```

也可以简化为：

```java
Function1 function = String::new;
```

特殊点的数组类型：

```java
Function<Integer,String[]> function = (x) -> new String[x];
```

可以简化为：

```java
Function<Integer,String[]> function = String[]::new;
```

## 3.lambda总结

上面并没有给出太多的lambda实例，只是侧重讲了如何去理解lambda表达式。到这里，不要懵。要记住lambda的本质：**为函数型接口的匿名实现进行简化与更简化**。

所谓的简化就是lambda的标准形式，所谓的更简化是在标准形式的基础上进行方法引用和构造引用。

方法引用是拿已有的方法去实现此刻的接口。

构造引用是对方法体只有一句new Object()的进一步简化。

# 二：Stream

在我看来，学习lambda与学习Stream的联系就是因为在许多博客、文档中对Stream API的讲解大量使用lambda表达式，导致不学lambda表达式看不懂Stream API。

## 1.如何理解Stream

Stream 不是集合元素，它不是数据结构并不保存数据，它是有关算法和计算的，它更像一个高级版本的 Iterator。简单来说，它的作用就是通过一系列操作将数据源（集合、数组）转化为想要的结果。

## 2.Stream特点

1. Stream 是不会存储元素的。
2. Stream 不会改变原对象，相反，他们会返回一个持有结果的新Stream。
3. Stream 操作是延迟执行的。意味着它们会等到需要结果的时候才执行。

## 3.生成Stream的方式

```java
//Collection系的 stream() 和 parallelStream();
List<String> list = new ArrayList<>();
Stream<String> stream = list.stream();
Stream<String> stringStream = list.parallelStream();

//通过Arrays
Stream<String> stream1 = Arrays.stream(new String[10]);

//通过Stream
Stream<Integer> stream2 = Stream.of(1, 2, 3);

//无限流
//迭代
Stream<Integer> iterate = Stream.iterate(0, (x) -> x + 2);
iterate.limit(10).forEach(System.out::println);

//生成
Stream<Double> generate = Stream.generate(() -> Math.random());
generate.forEach(System.out::println);
```

## 4.Stream的中间操作

多个中间操作连接而成为流水线，流水线不遇到终止操作是不触发任何处理的，所为又称为“惰性求值”。

```java
list.stream()
                .map(s -> s + 1)  //映射
                .flatMap(s -> Stream.of(s)) //和map差不多，但返回类型为Stream，类似list.add()和list.addAll()的区别
                .filter(s -> s < 1000)    //过滤
                .limit(5)   //限制
                .skip(1)    //跳过
                .distinct() //去重
                .sorted()   //自然排序
                .sorted(Integer::compareTo) //自定义排序
```

关于map方法，参数为一个Function函数型接口的对象，也就是传入一个参数返回一个对象。这个参数就是集合中的每一项。类似Iterator遍历。其它的几个操作思想都差不多。

执行上面的方法没什么用，因为缺少终止操作。

## 5.Stream的终止操作

```java
list.stream().allMatch((x) -> x == 555); // 检查是否匹配所有元素
list.stream().anyMatch(((x) -> x>600)); // 检查是否至少匹配一个元素
list.stream().noneMatch((x) -> x>500); //检查是否没有匹配所有元素
list.stream().findFirst(); // 返回第一个元素
list.stream().findAny(); // 返回当前流中的任意一个元素
list.stream().count(); // 返回流中元素的总个数
list.stream().forEach(System.out::println); //内部迭代
list.stream().max(Integer::compareTo); // 返回流中最大值
Optional<Integer> min = list.stream().min(Integer::compareTo);//返回流中最小值
System.out.println("min "+min.get());
```
reduce （归约）：将流中元素反复结合起来得到一个值
```java
Integer reduce = list.stream()
        .map(s -> (s + 1))
        .reduce(0, (x, y) -> x + y);    //归约：0为第一个参数x的默认值，x是计算后的返回值，y为每一项的值。
System.out.println(reduce);

Optional<Integer> reduce1 = list.stream()
        .map(s -> (s + 1))
        .reduce((x, y) -> x + y);  // x是计算后的返回值，默认为第一项的值，y为其后每一项的值。
System.out.println(reduce);
```

collect（收集）：将流转换为其他形式。需要Collectors类的一些方法。

```java
//转集合
Set<Integer> collect = list.stream()
        .collect(Collectors.toSet());

List<Integer> collect2 = list.stream()
        .collect(Collectors.toList());

HashSet<Integer> collect1 = list.stream()
        .collect(Collectors.toCollection(HashSet::new));

//分组 {group=[444, 555, 666, 777, 555]}
Map<String, List<Integer>> collect3 = list.stream()
        .collect(Collectors.groupingBy((x) -> "group"));//将返回值相同的进行分组
System.out.println(collect3);

//多级分组 {group={777=[777], 666=[666], 555=[555, 555], 444=[444]}}
Map<String, Map<Integer, List<Integer>>> collect4 = list.stream()
        .collect(Collectors.groupingBy((x) -> "group", Collectors.groupingBy((x) -> x)));
System.out.println(collect4);

//分区 {false=[444], true=[555, 666, 777, 555]}
Map<Boolean, List<Integer>> collect5 = list.stream()
        .collect(Collectors.partitioningBy((x) -> x > 500));
System.out.println(collect5);

//汇总
DoubleSummaryStatistics collect6 = list.stream()
        .collect(Collectors.summarizingDouble((x) -> x));
System.out.println(collect6.getMax());
System.out.println(collect6.getCount());

//拼接 444,555,666,777,555
String collect7 = list.stream()
        .map(s -> s.toString())
        .collect(Collectors.joining(","));
System.out.println(collect7);

//最大值
Optional<Integer> integer = list.stream()
        .collect(Collectors.maxBy(Integer::compare));
System.out.println(integer.get());
```

关于Stream的其它用法推荐参考下源码与API文档。
