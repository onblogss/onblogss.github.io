---
layout: post
title: SpringBoot集成Freemarker与Thymeleaf
date: 2018-05-10 14:21:00
author: 薛师兄
tags: SpringBoot
---
# 一:概括

>1. pom.xml添加依赖
>2. 配置application.yml
>3. HTML页面使用表达式

---

# 二：Freemarker模板引擎

1.添加依赖

```xml
<!-- ftl模板引擎 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
```

2.配置参数

```yaml
#Freemarker模板引擎
spring:
  freemarker:
    template-loader-path:
    - classpath:/templates
    charset: UTF-8
    check-template-location: true
    content-type: text/html
    expose-request-attributes: true
    expose-session-attributes: true
    request-context-attribute: request
    suffix: .ftl
    cache: false
    #关闭缓存，及时刷新，上线需要改成true
```

3.模板使用 
在templates文件夹下新建freemarker文件夹，然后在该文件夹下新建index.html

```html
<!DOCTYPE>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8"></meta>
<title>Freemarket</title>
</head>
<body>
${message}
</body>
</html>
```

4.Controller返回视图

```java
@Controller()
@RequestMapping("/ftl")
public class FtlController {

    @GetMapping("/hello")
    public String hello(ModelMap map) {
        map.addAttribute("message", "Hello! freemarket!");
        return "/freemarker/index";
    }

}
```

# 三：Thymeleaf模板引擎

1.添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

2.配置参数（一般情况不用配置）

```yaml
#Thymeleaf 静态资源配置
spring:
  thymeleaf:
      prefix: classpath:/templates/
      suffix: .html
      mode: HTML5
      encoding: UTF-8
      content-type: text/html
      cache: false
#关闭缓存，即使刷新，上线需要改成true
#i18n配置
  messages:
    basename: i18n/messages
    cache-seconds: 3600
    encoding: UTF-8
```

在源文件夹下新建i18n/messages文件夹目录，在该目录下新建messages.properties：

```java
roles<span class="hljs-preprocessor">.manager</span>=manager
roles<span class="hljs-preprocessor">.superadmin</span>=superadmin
```

2.模板使用 
thymeleaf涉及的标签很多

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head lang="en">
    <meta charset="UTF-8" />
    <title></title>

<!--    <script th:src="@{/js/test.js}"></script> -->

</head>
<body>
<!--@{}是用来输出内容的，${}是用来输出对象的，#{}是输出i18n的参数-->
<div>
    用户姓名：<input th:id="${user.name}" th:name="${user.name}" th:value="${user.name}"/>
    <br/>
    用户年龄：<input th:value="${user.age}"/>
    <br/>
    用户生日：<input th:value="${user.birthday}"/>
    <br/>
    用户生日：<input th:value="${#dates.format(user.birthday, 'yyyy-MM-dd')}"/>
    <br/>
</div>

<br/>

<div th:object="${user}">
    用户姓名：<input th:id="*{name}" th:name="*{name}" th:value="*{name}"/>
    <br/>
    用户年龄：<input th:value="*{age}"/>
    <br/>
    用户生日：<input th:value="*{#dates.format(birthday, 'yyyy-MM-dd hh:mm:ss')}"/>
    <br/>
</div>

<br/>

text 与 utext ：<br/>
<span th:text="${user.desc}">abc</span>
<br/>
<span th:utext="${user.desc}">abc</span>
<br/>
<br/>

URL:<br/>
<a href="" th:href="@{http://www.imooc.com}">网站地址</a>
<br/>

<br/>
<form th:action="@{/th/postform}" th:object="${user}" method="post" th:method="post">
    <input type="text" th:field="*{name}"/>
    <input type="text" th:field="*{age}"/>
    <input type="submit"/>
</form>
<br/>

<br/>
<div th:if="${user.age} == 18">十八岁的天空</div>
<div th:if="${user.age} gt 18">你老了</div>
<div th:if="${user.age} lt 18">你很年轻</div>
<div th:if="${user.age} ge 18">大于等于</div>
<div th:if="${user.age} le 18">小于等于</div>
<br/>

<br/>
<select>
     <option >选择框</option>
     <option th:selected="${user.name eq 'lee'}">lee</option>
     <option th:selected="${user.name eq 'imooc'}">imooc</option>
     <option th:selected="${user.name eq 'LeeCX'}">LeeCX</option>
</select>
<br/>

<br/>
<table>
    <tr>
        <th>姓名</th>
        <th>年龄</th>
        <th>年龄备注</th>
        <th>生日</th>
    </tr>
    <tr th:each="person:${userList}">
        <td th:text="${person.name}"></td>
        <td th:text="${person.age}"></td>
        <td th:text="${person.age gt 18} ? 你老了 : 你很年轻">18岁</td>
        <td th:text="${#dates.format(user.birthday, 'yyyy-MM-dd hh:mm:ss')}"></td>
    </tr>
</table>
<br/>

<br/>
<!--i18n配置-->
<div th:switch="${user.name}">
  <p th:case="'lee'">lee</p>
  <p th:case="#{roles.manager}">普通管理员</p>
  <p th:case="#{roles.superadmin}">超级管理员</p>
  <p th:case="*">其他用户</p>
</div>
<br/>

</body>
</html>
```

4.Controller返回视图

```java
@Controller
@RequestMapping("th")
public class ThymeleafController {

    @RequestMapping("/index")
    public String index(ModelMap map) {
        map.addAttribute("name", "thymeleaf-imooc");
        return "thymeleaf/index";
    }

    @RequestMapping("center")
    public String center() {
        return "thymeleaf/center/center";
    }

    @RequestMapping("test")
    public String test(ModelMap map) {

        User u = new User();
        u.setName("superadmin");
        u.setAge(10);
        u.setPassword("123465");
        u.setBirthday(new Date());
        u.setDesc("<font color='green'><b>hello imooc</b></font>");

        map.addAttribute("user", u);

        User u1 = new User();
        u1.setAge(19);
        u1.setName("imooc");
        u1.setPassword("123456");
        u1.setBirthday(new Date());

        User u2 = new User();
        u2.setAge(17);
        u2.setName("LeeCX");
        u2.setPassword("123456");
        u2.setBirthday(new Date());

        List<User> userList = new ArrayList<>();
        userList.add(u);
        userList.add(u1);
        userList.add(u2);

        map.addAttribute("userList", userList);

        return "thymeleaf/test";
    }

    @PostMapping("postform")
    public String postform(User u) {

        System.out.println("姓名：" + u.getName());
        System.out.println("年龄：" + u.getAge());

        return "redirect:/th/test";
    }

    @RequestMapping("showerror")
    public String showerror(User u) {

        int a = 1 / 0;

        return "redirect:/th/test";
    }
}
```