---
layout: post
title: SpringBoot中注入ApplicationContext对象的三种方式
date: 2019-04-21 20:57:00
author: 薛师兄
tags: SpringBoot
---
在项目中，我们可能需要手动获取spring中的bean对象，这时就需要通过 ApplicationContext 去操作一波了！

### 1、直接注入（Autowired）

```java
@Component
public class User {

    @Autowired
    private ApplicationContext applicationContext;
}
```

### 2、构造器方法注入

```java
@Component
public class User{
    private ApplicationContext applicationContext;

    public User(ApplicationContext applicationContext) {
        this.applicationContext = applicationContext;
    }
}
```

### 3、手动构建类实现接口

```java
/**
 * Spring的ApplicationContext的持有者,可以用静态方法的方式获取spring容器中的bean
 *
 * @date 2018年5月27日 下午6:32:11
 */
@Component
public class SpringContextHolder implements ApplicationContextAware {

    private static ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        SpringContextHolder.applicationContext = applicationContext;
    }

    public static ApplicationContext getApplicationContext() {
        assertApplicationContext();
        return applicationContext;
    }

    @SuppressWarnings("unchecked")
    public static <T> T getBean(String beanName) {
        assertApplicationContext();
        return (T) applicationContext.getBean(beanName);
    }

    public static <T> T getBean(Class<T> requiredType) {
        assertApplicationContext();
        return applicationContext.getBean(requiredType);
    }

    private static void assertApplicationContext() {
        if (SpringContextHolder.applicationContext == null) {
            throw new RuntimeException("applicaitonContext属性为null,请检查是否注入了SpringContextHolder!");
        }
    }

}
```

可以在使用类上添加 @DependsOn(“springContextHolder”)，确保在此之前 SpringContextHolder 类已加载！

> 本文转载自：<https://blog.csdn.net/Abysscarry/article/details/80490624>