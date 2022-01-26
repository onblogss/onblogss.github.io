---
layout: post
title: 两行代码玩转Spring Data排序和分页
date: 2018-07-31 17:18:00
author: 薛师兄
tags: SpringData
---
## 一：唠嗑

在实际项目中对Spring Data的各种使用相当多，简单的增删改查Spring Data提供了现成的方法，一些复杂的，我们可以在接口方法写And,Not等关键字来搞定，想写原生SQL，CQL（Neo4j），Query DSL （Elasticsearch）的，直接使用@Query(“......”)注解搞定，真的是方便到不行！

本篇博客不打算讲Spring Data如何使用，不同的模块（JPA，Neo4j....）使用也略不相同，但Spring Data的排序`Sort`和分页`Pageable`接口都是差不多的，所以带大家搞明白搞明白Spring Data的排序和分页是如何使用的。

## 二：介绍

Spring Data的任务是为数据访问提供一个熟悉的、一致的、基于Spring的编程模型，同时仍然保留底层数据存储的特殊特性。

Spring Data 项目的目的是为了简化构建基于 Spring 框架应用的数据访问计数，包括非关系数据库、Map-Reduce 框架、云数据服务等等；另外也包含对关系数据库的访问支持。 

SpringData让数据访问变得更加方便。 

## 三：模块

[Spring Data](https://spring.io/projects/spring-data)

- [Spring Data for Apache Cassandra](https://projects.spring.io/spring-data-cassandra)
- [Spring Data Commons](https://spring.io/projects/spring-data-commons)
- [Spring Data Couchbase](https://projects.spring.io/spring-data-couchbase)
- [Spring Data Elasticsearch](https://projects.spring.io/spring-data-elasticsearch)
- [Spring Data Envers](https://projects.spring.io/spring-data-envers/)
- [Spring Data for Pivotal GemFire](https://projects.spring.io/spring-data-gemfire)
- [Spring Data Graph](https://spring.io/projects/spring-hadoop)
- [Spring Data JDBC](https://projects.spring.io/spring-data-jdbc)
- [Spring Data JDBC Extensions](https://projects.spring.io/spring-data-jdbc-ext)
- [Spring Data JPA](https://spring.io/projects/spring-data-jpa)
- [Spring Data LDAP](https://projects.spring.io/spring-data-ldap)
- [Spring Data MongoDB](https://projects.spring.io/spring-data-mongodb)
- [Spring Data Neo4J](https://projects.spring.io/spring-data-neo4j)
- [Spring Data Redis](https://projects.spring.io/spring-data-redis)
- [Spring Data REST](https://projects.spring.io/spring-data-rest)
- [Spring Data for Apache Solr](https://projects.spring.io/spring-data-solr)
- [Spring for Apache Hadoop](https://spring.io/projects/spring-hadoop)

## 四：排序

通过一行代码就可以快速使用：

```java
Sort sort = new Sort(Sort.Direction.DESC, "id");
```

在Sort类中定义了一个枚举类型`Direction`，该枚举类型声明了两个常量`ASC`，`DESC`定义方向。该构造方法的第一个参数指明方向降序（`DESC`）或升序（ASC），第二个参数指明以`id`列的值为准进行排序。



你也可以创建一个多属性的Sort实例。

```java
Sort(Sort.Direction direction, List<String> properties)
```

你也可以只传入属性而不声明方向：

```java
Sort(String... properties)
```

不过官方已经弃用该方法，推荐使用

```java
public static Sort by(String... properties)
```

当你不声明方向时，默认方向为升序。

```java
public static final Direction DEFAULT_DIRECTION = Direction.ASC;
```



Sort的一些方法

| 修饰符和类型           | 方法和描述                                                   |
| ---------------------- | ------------------------------------------------------------ |
| `Sort`                 | `and(Sort sort)`  <br />返回由当前排序的排序顺序与给定的排序顺序组成的新排序。 |
| `Sort`                 | `ascending()` <br />返回具有当前设置但升序方向的新排序。     |
| `static Sort`          | `by(List<Sort.Order> orders)` <br />为给定的`Sort.Order`创建一个新的排序。 |
| `static Sort`          | `by(Sort.Direction direction, String... properties)`<br />创建一个新的排序。 |
| `static Sort`          | `by(Sort.Order... orders)`<br />Creates a new [`Sort`](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/domain/Sort.html) for the given [`Sort.Order`](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/domain/Sort.Order.html)s. |
| `static Sort`          | `by(String... properties)`<br />Creates a new [`Sort`](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/domain/Sort.html) for the given properties. |
| `Sort`                 | `descending()` <br />返回具有当前设置但顺序相反的新排序。    |
| `boolean`              | `equals(Object obj)`                                         |
| `Sort.Order`           | `getOrderFor(String property)`<br />根据property获取Order    |
| `boolean`              | `isSorted()`                                                 |
| `boolean`              | `isUnsorted()`                                               |
| `Iterator<Sort.Order>` | `iterator()`                                                 |
| `static Sort`          | `unsorted()` <br />返回一个根本没有排序设置的排序实例。      |



### Sort.Order

`Sort.Order`是`Sort的`一个静态内部类，官方说明是：PropertyPath实现了排序的配对。方向和属性。它用于提供排序的输入 。

简单来讲，你可以定义一个Order，在需要时传入order构建Sord实例。

```java
Order(Sort.Direction direction, String property)
```

更独特的使用是加入自己的空处理提示的枚举：

```java
Order(Sort.Direction direction, String property, Sort.NullHandling nullHandlingHint)
```

 `Sort.NullHandling`是可用于排序表达式的空处理提示的枚举。对使用的数据存储的一种提示，用于在非空条目之后对具有空值的条目进行排序。 

在需要Sort时，可通过Order创建：

```java
Sort(Sort.Order... orders)
```



Sort.Order的一些方法

| 修饰符和类型        | 方法和描述                                                   |
| ------------------- | ------------------------------------------------------------ |
| `static Sort.Order` | `asc(String property)`<br />创建升序的Order实例              |
| `static Sort.Order` | `by(String property)`<br />创建默认方向的Order实例           |
| `static Sort.Order` | `desc(String property)`<br />创建降序的Order实例             |
| `Sort.Direction`    | `getDirection()`                                             |
| `Sort.NullHandling` | `getNullHandling()`                                          |
| `String`            | `getProperty()`                                              |
| `Sort.Order`        | `ignoreCase()`<br />开启不分大小写排序                       |
| `boolean`           | `isAscending()`<br /> 返回此属性的排序是否要升序。           |
| `boolean`           | `isDescending()`<br /> 返回此属性的排序是否应该降序。        |
| `boolean`           | `isIgnoreCase()`<br /> 返回该排序是否区分大小写。            |
| `Sort.Order`        | `nullsFirst()`<br />返回 [`Sort.Order`](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/domain/Sort.Order.html) 使用 [`Sort.NullHandling.NULLS_FIRST`](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/domain/Sort.NullHandling.html#NULLS_FIRST) 作为空处理提示。First：第一个 |
| `Sort.Order`        | `nullsLast()`<br />返回 [`Sort.Order`](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/domain/Sort.Order.html) 使用[`Sort.NullHandling.NULLS_LAST`](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/domain/Sort.NullHandling.html#NULLS_LAST) 作为空处理提示。Last：最后一个 |
| `Sort.Order`        | `nullsNative()`<br />返回 [`Sort.Order`](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/domain/Sort.Order.html) 使用[`Sort.NullHandling.NATIVE`](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/domain/Sort.NullHandling.html#NATIVE) 作为空处理提示。NATIVE：原生的 |
| `Sort.Order`        | `with(Sort.Direction direction)`<br />创建Order实例.         |
| `Sort.Order`        | `with(Sort.NullHandling nullHandling)`<br />创建Order实例.   |
| `Sort`              | `withProperties(String... properties)`<br />创建Order实例.   |
| `Sort.Order`        | `withProperty(String property)`<br />创建Order实例.          |



如果你还想全面了解它的使用，推荐阅读官方英文文档：<https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/domain/Sort.html>



## 五：分页

Pageable只是 Spring Data 提供的分页信息的抽象接口。 

实现类：

AbstractPageRequest：抽象类。供`PageRequest`和`QPageRequest`继承。

PageRequest： 基本的可页面化Java Bean实现。 

QPageRequest：基本的Java Bean实现，可以支持QueryDSL。 



分页功能也只需要一行代码：

```java
Pageable pageable = new PageRequest(int page, int size, Sort sort);
```

`page` - 从零开始的索引页 

`size`- 要返回的页面的大小

`sort` - 排序

你也可以创建没有排序的分页

```java
PageRequest(int page, int size)
```

更方便的是可以一行代码创建有排序方向和属性的分页

```java
PageRequest(int page, int size, Sort.Direction direction, String... properties)
```



如果你使用的是2.0以后的版本，官方已经弃用以上构造方法的形式，推荐使用静态方法：

| 修饰符和类型         | 方法和描述                                                   |
| -------------------- | ------------------------------------------------------------ |
| `static PageRequest` | `of(int page, int size)`                                     |
| `static PageRequest` | `of(int page, int size, Sort.Direction direction, String... properties)` |
| `static PageRequest` | `of(int page, int size, Sort sort)`                          |


比如我们想遍历整个数据表，就可以使用分页遍历，这样不至于一次把数据全部加载到内存。

| 修饰符和类型  | 方法和描述     |
| ------------- | -------------------------- |
| `Pageable`    | `first()` 请求第一页。     |
| `Sort`        | `getSort()` 返回排序参数。 |
| `Pageable`    | `next()` 请求下一个页面。  |
| `PageRequest` | `previous()` 请求前一页。  |

**注意：**在使用`next()`方法时，不要把`pageable.next()`直接作为参数传入方法，如`repository.findAll(page.next())`这样的写法会导致死循环。查看next()方法的源码发现这个方法只是帮我们new了一个新的`Pageable`对象，原来的`pageable`还是没啥变化。一直next()下去也只是在原地踏步。

```java
	public Pageable next() {
		return new PageRequest(getPageNumber() + 1, getPageSize(), getSort());
	}
```

正确的写法：

```java
repository.findAll(pageable = pageable.next());
```

## 六：使用

Spring Data Jpa除了会通过命名规范帮助我们扩展Sql语句外，还会帮助我们处理类型为`Pageable`的参数，将`pageable`参数转换成为sql语句中的条件，同时，还会帮助我们处理类型为`Page`的返回值，当发现返回值类型为`Page`，Spring Data Jpa将会把数据的整体信息、当前数据的信息，分页的信息都放入到返回值中。这样，我们就能够方便的进行个性化的分页查询。 

```java
public interface UserRepository extends JpaRepository<User,Long> {

    @Override
    Page<Medical> findAll(Pageable pageable);
}
```



如果你想用@Query写原生查询语句并实现分页：

Spring Data JPA目前不支持原生查询的动态排序，因为它必须操作声明的实际查询，这对于原生SQL是无法可靠地做到的。但是，您可以通过自己指定count查询来使用原生查询进行分页，如下面的示例所示: 

```java
public interface UserRepository extends JpaRepository<User, Long> {

  @Query(value = "SELECT * FROM USERS WHERE LASTNAME = ?1",
    countQuery = "SELECT count(*) FROM USERS WHERE LASTNAME = ?1",
    nativeQuery = true)
  Page<User> findByLastname(String lastname, Pageable pageable);
}
```



## 七：读取

分页查询：

```java
Page<Book> sampleEntities = userRepository.findAll(pageable);
```

接口`Page`继承了接口`Slice`，话不多说，直接上干货！



总页数

```java
int getTotalPages()
```
元素的总数

```java
long getTotalElements()
```

返回当前页的索引（是第几页） 

```java
int	getNumber()
```

返回作为`List`的页面内容

```java
List<T>	getContent()
```

返回当前在这个页上的元素的数量

```java
int	getNumberOfElements()
```

返回用于请求当前页的`Pageable` 

```java
default Pageable	getPageable()
```

返回页的大小。 

```java
int	getSize()
```

返回页的排序参数。 

```java
Sort getSort()
```

页面是否有内容。

```java
boolean	hasContent()
```

是否有下一页。 

```java
boolean	hasNext()
```

是否有上一页

```java
boolean	hasPrevious()
```

当前页是否是第一个 

```java
boolean	isFirst()
```

当前页是否是最后一个 

```java
boolean	isLast()
```
下一页的Pageable 
```java
Pageable nextPageable()
```
上一页的Pageable 
```java
Pageable previousPageable()
```



------

关于Spring Data的排序与分页就到这里，记得点赞哦~~~