---
layout: post
title: Hibernate各种基本注解及一对一(多)关系映射采坑笔记
date: 2018-06-03 00:36:00
author: 薛师兄
tags: Hibernate
---
>*  hibernate提供两种方式配置关系映射，一种XMl配置，一种注解。SpringBoot已经自带了hibernate注解方式，我也是特别喜欢使用注解，特此记下常用的知识点。

---

# 1.基本注解

```java
@Table(name = " ",catalog=" ", schema=" ")
//name表名，虽然可选，建议写上。catalog在MySql不支持，不必写。schema在MySql中指数据库名。

@Table(uniqueConstraints = {@UniqueConstraint(columnNames="name")})
或
@Column(name = "name",unique = true)
//指定建表时需要建唯一约束的列（使除主键外的列保持唯一约束）

 @Table(indexes = {@Index(columnList = "ip")}) 
//指定建表时需要加索引的列

@Embeddable  
//表示一个非Entity类嵌入到一个Entity类作为属性而存在。
//使用方法，新建一个类注上该注解即可。在此类同样可使用注解对字段进行约束。

@embedded
//该注解是用来注释属性的，表示该类为嵌入类，同时，该类也得注释@Embeddable注解

@GeneratedValue(strategy=GenerationType)
GenerationType.AUTO //默认，根据底层数据库自动选择
GenerationType.INDENTITY //根据数据库的Identity字段生成
GenerationType.SEQUENCE //根据sequenqe来决定主键的取值
GenerationType.TABLE //使用指定表来决定主键取值，结合TableGenerator使用

@Id //自定义主键生成策略
@GeneratedValue(generator="sid")    //名字
@GenericGenerator(name="sid",strategy="assigned")   //策略

@EmbeddedId
//使用嵌入式主键类实现复合主键
//主键类必须实现Serializable接口，必须有默认的public无参构造方法，
//必须覆盖equals和hashCode()方法，必须注解@Embeddable 
//使用时，把主键类对象当做参数传入Table类，对Table类进行保存即可。


@Transient
//表示该属性并非是到数据库表的字段的映射，否则默认注解@Basic

@Column(columnDefinition="TEXT", nullable=true)
//表示该字段为数据库中的TEXT类型，存储长文本<br /><br />@ElementCollection(fetch = FetchType.LAZY)//定义基本类型或可嵌入类的实例集合 <br />@OrderColumn(name="position")//如果使用的是List，你需要多定义一个字段维护集合顺序 <br />private List<String> part;<br />

```

##### 关系映射注解@OneToOne(cascade = {CascadeType.ALL})

>*  CascadeType.PERSIST：级联新增（又称级联保存）：对order对象保存时也对items里的对象也会保存。对应EntityManager的presist方法。
>*  CascadeType.MERGE：级联合并（级联更新）：若items属性修改了那么order对象保存时同时修改items里的对象。对应EntityManager的merge方法 。
>*  CascadeType.REMOVE：级联删除：对order对象删除也对items里的对象也会删除。对应EntityManager的remove方法。
>*  CascadeType.REFRESH：级联刷新：获取order对象里也同时也重新获取最新的items时的对象。对应EntityManager的refresh(object)方法有效。即会重新查询数据库里的最新数据。
>*  CascadeType.ALL：以上四种都是。

## 2.级联保存异常

`org.springframework.dao.InvalidDataAccessApiUsageException: detached entity passed to persist` 
**解决方法：** 
在进行多对多(一)保存操作时，数据表定义主键为自增，但在执行插入前需设置ID为-1。有多个级联关系，每个对象都要进行设置。即可避免该异常。如下：

```java
public void test() {
     User user=new User();
     user.setId(-1);  //加上这一句即可！！！
     user.setUsername("李彤");
     user.setPassword("1144");
     user.setRoles(roles);
 }
```

## 级联查询异常

`org.hibernate.LazyInitializationException: failed to lazily initialize a collection of .... could not initialize proxy -`

*  翻译过来就是Hibernate无法延迟加载，该异常多出现在一对多查询时，解决方法：
*  `@OneToMany(cascade={CascadeType.ALL}, fetch=FetchType.LAZY)`
*  `Lazy`：延迟加载该关联对象，改为`EAGER`即可。

---

**下面讲关系映射，务必牢记：** 
**级联保存\删除时取决于Entity类中cascade = {xxx}注解。** 
若是cascade = {CascadeType.ALL}，直接delete主表对象即可级联删除属性对象

---

# 3.一对一

#### （1）单向

```java
@Entity
@Table(name="User")
public class User {
    @Id
    @GeneratedValue
    @Column(name="sid")
    private int sid;

    @Column(name="name")
    private String name;

    @OneToOne(cascade = {CascadeType.ALL})
    @JoinColumn(name = "pid",unique=true)
    //name=定义外键在本表的字段名，若只配置本类，则为单向关联
    //unique=true是指这个字段的值在这张表里不能重复，所有记录值都要唯一，就像主键那样
    private Room room;

    public User() {
        super();
        // 多对一
    }
    ...
}
```

### （2）双向

```java
@Entity
@Table(name="Room")
public class Room {
    @Id
    @Column(name="cid")
    private int cid;

    @Column(name="addr")
    private String addr;

    @OneToOne(mappedBy="room")//被控方
    //mappedBy同样指定由对方来进行维护关联关系
    private User user;
    public Room() {
        super();
        // 多对一
    }
    ...

}
```

# 4.一对多单向

```java
@Entity
@Table(name="Room")
public class Room {
    @Id
    @GeneratedValue
    @Column(name="cid")
    private int cid;

    @Column(name="addr")
    private String addr;

    @OneToMany(cascade={CascadeType.ALL}, fetch=FetchType.LAZY) //一对多为Lazy，多对一为Eager
    @JoinColumn(name="cid") //name=定义外键在本表的字段名 rCN=关联外键对象的哪个字段
    private Set<User> users;

    public Room() {
        super();
        // 一对多:一方持有多方的引用
    }
    ...

}
```

```java
//正常建表
@Entity
@Table(name="User")
public class User {
    @Id
    @GeneratedValue
    @Column(name="sid")
    private int sid;
    @Column(name="name")
    private String name;

    public User() {
        super();
        //一对多
    }
    ...

}
```

# 5.多对一单向

```java
@Entity
@Table(name="User")
public class User {
    @Id
    @GeneratedValue
    @Column(name="sid")
    private int sid;

    @Column(name="name")
    private String name;

    @ManyToOne(cascade={CascadeType.ALL}, fetch=FetchType.EAGER)    //一对多为Lazy，多对一为Eager
    @JoinColumn(name="cid", referencedColumnName="cid") //name=定义外键在本表的字段名 rCN=关联外键对象的哪个字段
    private Room room;

    public User() {
        super();
        // 多对一
    }
    ...
}
```

```java
//正常建表
@Entity
@Table(name="Room")
public class Room {
    @Id
    @Column(name="cid")
    private int cid;
    @Column(name="addr")
    private String addr;

    public Room() {
        super();
        // 多对一
    }
    ...
}
```

# 6.一对多（多对一）双向

```java
@Entity
@Table(name="User")
public class User {
    @Id
    @GeneratedValue
    @Column(name="sid")
    private int sid;

    @Column(name="name")
    private String name;

    @ManyToOne(cascade={CascadeType.ALL}, fetch=FetchType.EAGER)    //一对多为Lazy，多对一为Eager
    @JoinColumn(name="cid") //name=定义外键在本表的字段名
    private Room room;

    public User() {
        super();
        // 多对一
    }
    ...
}
```

```java
@Entity
@Table(name="Room")
public class Room {
    @Id
    @GeneratedValue
    @Column(name="cid")
    private int cid;

    @Column(name="addr")
    private String addr;

    @OneToMany(cascade={CascadeType.ALL}, fetch=FetchType.LAZY) //一对多为Lazy，多对一为Eager
    @JoinColumn(name="cid") //name=定义外键在本表的字段名
    private Set<User> users;

    public Room() {
        super();
        // 一对多:一方持有多方的引用
    }
    ...

}
```

# 7.多对多

### (1)单向

```java
@Entity
@Table(name="t_course")
public class Course
{
    @Id
    @GeneratedValue
    private int id;

    private String name;

    @ManyToMany　　　--->　ManyToMany指定多对多的关联关系
    @JoinTable(name="t_teacher_course", 
    joinColumns={@JoinColumn(name="cid")}, 
    inverseJoinColumns={ @JoinColumn(name = "tid") })　
    /*因为多对多之间会通过一张中间表来维护两表直接的关系，所以通过 JoinTable        
    这个注解来声明，我方是Course，所以在对方外键的名称就是 rid，
    inverseJoinColumns也是一个 @JoinColumn类型的数组，
    表示的是对方在我这放中的外键名称，对方是Teacher，所以在我方外键的名称就是 tid*/
    private Set<Teacher> teachers;

     ...

}
```

### (2)双向

```java
@Entity
@Table(name="t_teacher")
public class Teacher
{   
    @Id
    @GeneratedValue

    private String name;

    @ManyToMany(mappedBy="teachers")//表示由Course那一方来进行维护
    private Set<Course> courses;
    ...

}
```