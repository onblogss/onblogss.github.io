---
layout: post
title: SpringBoot整合Shiro使用Ehcache等缓存无效问题
date: 2018-05-18 13:14:00
author: 薛师兄
tags: SpringBoot
---
# 前言

1. 整合有缓存、事务的spring boot项目一切正常。
2. 在该项目上整合shiro安全框架，发现部分类的缓存Cache不能正常使用。
3. 然后发现该类的注解基本失效，包括事务Transaction注解。事务不能正常运行。

# 分析

1. 注解失效的类，都是在shiro框架中(UserRealm)使用过@Autowire注入的类。
2. 基本确定是shiro框架与spring框架的BeanFactory有所冲突，导致注入shiro框架的类不能被spring正确初始化。

# 以上参考

[https://blog.csdn.net/elonpage/article/details/78965176](https://blog.csdn.net/elonpage/article/details/78965176)

---

# 我的解决方法

1.比如使用Shiro时我们自定义了Realm，并在其中使用了

```java
@Autowired
private UserService userService;
```

2.现在修改为

```java
@Autowired
@Lazy
private UserService userService;
```

即可，还有方法，比如ApplicationContextRegister.getBean()手动注入Bean。

*延伸阅读：<a id="cb_post_title_url" href="/">史上最全的Spring Boot Cache使用与整合</a>*