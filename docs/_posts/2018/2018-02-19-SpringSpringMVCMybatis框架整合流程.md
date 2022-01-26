---
layout: post
title: Spring+SpringMVC+Mybatis框架整合流程
date: 2018-02-19 15:13:00
author: 薛师兄
tags:
  - Spring
  - SpringMVC
  - Mybatis
---
### 一：基本步骤

1. 新建Maven项目，导入相关**依赖**。（推荐）   *&mdash;&mdash;&mdash;&mdash;&ndash;Mybatis配置 &mdash;&mdash;&mdash;&mdash;&mdash;-*
2. 新建entity包，并根据数据库(表)新建相关**实体类**。
3. 新建dao包，并根据业务创建必要的mapper **接口类**。
4. 在resources下新建**mybatis-config.xml**配置文件。
5. 在resources源文件夹下新建mapper文件夹，根据3创建的接口类配置相应的**mapper.xml**   *&mdash;&mdash;&mdash;&mdash;Spring整合Mybatis&mdash;&mdash;&mdash;&mdash;&ndash;*
6. 在resources文件夹下新建spring文件夹，新建**spring-dao.xml**，然后添加二者整合的配置。   *&mdash;&mdash;&mdash;&mdash;Spring Service层配置&mdash;&mdash;&mdash;&ndash;*
7. 在spring文件夹下新建**spring-service.xml**，配置service层的相关bean。   *&mdash;&mdash;&mdash;&mdash;&ndash;SpringMVC配置&mdash;&mdash;&mdash;&mdash;&mdash;-*
8. 在WEB-INF的**web.xml**中进行我们前端控制器DispatcherServlet的配置。
9. 在spring文件夹下创建**spring-web.xml**，进行web层相关bean(即Controller)的配置。

### 二：详细配置

1.新建Maven项目，导入相关依赖。（推荐） 
若不使用maven：请前往[Maven官网](http://mvnrepository.com/)依次下载jar包导入）

```xml
<properties>
    <!-- 统一源码的编码方式 -->
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <!-- 统一各个框架版本 -->
    <spring.version>4.1.7.RELEASE</spring.version>
    <mybatis.version>3.3.0</mybatis.version>
    <mybatis-spring>1.2.3</mybatis-spring>
</properties>

<dependencies>
    <dependency>
  <!--3.0的junit是使用编程的方式来进行测试，而junit4是使用注解的方式来运行junit-->
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.11</version>
  <scope>test</scope>
</dependency>


<!--补全项目依赖-->
<!--1.日志 java日志有:slf4j,log4j,logback,common-logging
    slf4j:是规范/接口
    日志实现:log4j,logback,common-logging
    使用:slf4j+logback
-->
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-api</artifactId>
  <version>1.7.12</version>
</dependency>
<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-core</artifactId>
  <version>1.1.1</version>
</dependency>
<!--实现slf4j接口并整合-->
<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-classic</artifactId>
  <version>1.1.1</version>
</dependency>


<!--1.数据库相关依赖-->
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>5.1.35</version>
  <scope>runtime</scope>
</dependency>
<dependency>
  <groupId>c3p0</groupId>
  <artifactId>c3p0</artifactId>
  <version>0.9.1.1</version>
</dependency>

<!--2.dao框架:MyBatis依赖-->
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>${mybatis.version}</version>
</dependency>
<!--mybatis自身实现的spring整合依赖-->
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis-spring</artifactId>
  <version>${mybatis-spring}</version>
</dependency>

<!--3.Servlet web相关依赖-->
<dependency>
  <groupId>taglibs</groupId>
  <artifactId>standard</artifactId>
  <version>1.1.2</version>
</dependency>
<dependency>
  <groupId>jstl</groupId>
  <artifactId>jstl</artifactId>
  <version>1.2</version>
</dependency>
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-databind</artifactId>
  <version>2.5.4</version>
</dependency>
<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>javax.servlet-api</artifactId>
  <version>3.1.0</version>
  <scope>provided</scope>
</dependency>

<!--4:spring依赖-->
<!--1)spring核心依赖-->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-core</artifactId>
  <version>${spring.version}</version>
</dependency>
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-beans</artifactId>
  <version>${spring.version}</version>
</dependency>
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-context</artifactId>
  <version>${spring.version}</version>
</dependency>
<!--2)spring dao层依赖-->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-jdbc</artifactId>
  <version>${spring.version}</version>
</dependency>
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-tx</artifactId>
  <version>${spring.version}</version>
</dependency>
<!--3)springweb相关依赖-->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-web</artifactId>
  <version>${spring.version}</version>
</dependency>
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-webmvc</artifactId>
  <version>${spring.version}</version>
</dependency>
<!--4)spring test相关依赖-->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-test</artifactId>
  <version>${spring.version}</version>
</dependency>

</dependencies>
```

2.新建entity包，并根据数据库(表)新建相关实体类。

```java
//举个栗子 getter/setter
public class Seckill {
    private int id;
    private String userName;
    private int userAge;
    private String userAddress;
    ....
```

3.新建dao包，并根据业务创建必要的mapper接口类。

```java
//再举个栗子
public interface SeckillDao {

    /**
     * 根据查询对象
     * @param seckillId
     * @return
     */
    Seckill queryById(long seckillId);

}
```

4.在resources下新建mybatis-config.xml配置文件。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration> 
    <!-- 配置全局属性 -->
    <settings>
        <!-- 使用jdbc的getGeneratedKeys 获取数据库自增主键值 -->
        <setting name="useGeneratedKeys" value="true"/>
        <!-- 使用列别名替换列名 默认:true
         select name as title from table
         -->
        <setting name="useColumnLabel" value="true"/>
        <!-- 开启驼峰命名转换:Table(create_time) -> Entity(createTime) -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
</configuration>
```

5.在resources源文件夹下新建mapper文件夹，根据第3步创建的接口类配置相应的mapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="换成你的包.dao.SeckillDao">
    <!-- 目的:为DAO接口方法提供sql语句配置-->

    <select id="queryById" resultType="Seckill" parameterType="long">
        <!-- 具体sql -->
        select seckill_id,name,number,start_time,end_time,create_time
        from seckill
        where seckill_id = #{seckillId}
    </select>

</mapper>
```

6.在resources文件夹下新建spring文件夹，新建spring-dao.xml，然后添加二者整合的配置。

```properties
/*推荐：在resources包下创建jdbc.properties用于配置数据库的连接信息*/
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/seckill?useUnicode=true&characterEncoding=utf-8
username=root
password=
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
    <!-- Spring配置整合mybatis过程 -->
    <!-- 1:配置数据库相关参数properties的属性：${url} -->
    <context:property-placeholder location="classpath:jdbc.properties"/>

    <!-- 2:数据库连接池 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <!-- 配置连接池属性 -->
        <property name="driverClass" value="${jdbc.driver}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>

        <!-- c3p0连接池的私有属性 -->
        <property name="maxPoolSize" value="30"/>
        <property name="minPoolSize" value="10"/>
        <!-- 关闭连接后不自动commit -->
        <property name="autoCommitOnClose" value="false"/>
        <!-- 获取连接超时时间 -->
        <property name="checkoutTimeout" value="3000"/>
        <!-- 当获取连接失败重试次数 -->
        <property name="acquireRetryAttempts" value="2"/>
    </bean>

    <!-- 约定大于配置 -->
    <!-- 3:配置SqlSessionFactory对象 -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- 注入数据库连接池 -->
        <property name="dataSource" ref="dataSource"/>
        <!-- 配置MyBatis全局配置文件:mybatis-config.xml -->
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
        <!-- 扫描entity包 使用别名 -->
        <property name="typeAliasesPackage" value="换成你的包.entity"/>
        <!-- 扫描sql配置文件:mapper需要的xml文件 -->
        <property name="mapperLocations" value="classpath:mapper/*.xml"/>
    </bean>

    <!-- 4:配置扫描Dao接口包,动态实现Dao接口，注入到spring容器中-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!-- 注入sqlSessionFactory -->
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
        <!-- 给出需要扫描Dao接口包 -->
        <property name="basePackage" value="换成你的包.dao"/>
    </bean>

</beans>
```

7.在spring文件夹下新建spring-service.xml，配置service层的相关bean。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!--扫描service包下所有使用注解的类型-->
    <context:component-scan base-package="换成你的包.service"/>

    <!-- 说明：下面的Spring事务管理并不是必需 -->

    <!--配置事务管理器 -->
    <bean id="transactionManager"
        class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--注入数据库连接池 -->
        <property name="dataSource" ref="dataSource" />
    </bean>

    <!--配置基于注解的声明式事务 默认使用注解来管理事务行为 -->
    <tx:annotation-driven transaction-manager="transactionManager" />

</beans>
```

Service层的相关bean示例：

```java
//推荐：先定义service接口包再定义service.impl实现包
//@Component 通用 @Service @Dao @Controller控制器
@Service
public class SeckillServiceImpl implements SeckillService{

    //注入Service依赖
    @Autowired //@Resource
    private SeckillDao seckillDao;

    @Override
    public List<Seckill> getSeckillList() {

        return seckillDao.queryAll(0,4);
    }
    //....
```

8.在WEB-INF的web.xml中进行我们前端控制器DispatcherServlet的配置。

```xml
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee 
    http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
    version="3.1" metadata-complete="true">
<!-- metadata-complete="true" 不自动扫描注释：因为action由Spring MVC管理-->

<!--用maven创建的web-app需要修改servlet的版本为3.1-->
<!--配置DispatcherServlet-->
    <servlet>
        <servlet-name>seckill-dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!--
            配置SpringMVC 需要配置的文件
            spring-dao.xml，spring-service.xml,spring-web.xml
            Mybites -> spring -> springMvc
        -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring/spring-*.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>seckill-dispatcher</servlet-name>
        <!--默认匹配所有请求-->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

9.在spring文件夹下创建spring-web.xml，进行web层相关bean(即Controller)的配置。

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context 
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/mvc 
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!--配置spring mvc-->
    <!--1,开启springmvc注解模式
    a.自动注册DefaultAnnotationHandlerMapping,AnnotationMethodHandlerAdapter
    b.默认提供一系列的功能:数据绑定，数字和日期的format@NumberFormat,@DateTimeFormat
    c:xml,json的默认读写支持-->
    <mvc:annotation-driven/>

    <!--2.静态资源默认servlet配置-->
    <!--
        1).加入对静态资源处理：js,gif,png
        2).允许使用 "/" 做整体映射
    -->
    <mvc:default-servlet-handler/>

    <!--3：配置JSP 显示ViewResolver-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

    <!--4:扫描web相关的bean-->
    <context:component-scan base-package="换成你的包.web"/>
</beans>
```

web层的相关bean示例：

```java
@Controller //Controller层调用Service层，Service层调用Dao层
@RequestMapping("/seckill") // url:/模块/资源/{id}/细分 /seckill/list
public class SeckillController {

    @Autowired
    private SeckillService seckillService;

    @RequestMapping(value = "/list", method = RequestMethod.GET)
    public String list(Model model) {
        //获取列表页
        List<Seckill> list = seckillService.getSeckillList();
        model.addAttribute("list", list);
        //list.jsp + model = ModelAndView
        return "list";      // /WEB-INF/jsp/"list".jsp
    }
}
```