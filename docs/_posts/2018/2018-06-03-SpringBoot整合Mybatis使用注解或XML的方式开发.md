---
layout: post
title: SpringBoot整合Mybatis使用注解或XML的方式开发
date: 2018-06-03 14:05:00
author: 薛师兄
tags: SpringBoot
---
2018-6-4

# 补充mybatis-spring-boot注解的使用

---

# 1.导包

只需要再导入mysql+mybatis两个包

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.2</version>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

# 2.数据源

application.yml

```java
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://118.89.177.110:3306/test?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: 336699yst
mybatis:
  configuration:
    map-underscore-to-camel-case: true #开启驼峰命名（数据库d_id匹配实体类dId）
logging:
  level:
    com.example.xxxx.mapper: debug #打印SQL日志
```

# 3.实体类

```java
public class User {
    private int id;
    private String name;
    private Integer age;

    public User( String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    public User() {
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

   ...getter 
   ...setter
}
```

# 4.@Mapper

在启动类中添加对mapper包扫描@MapperScan

```java
//@MapperScan("") 相比指定扫描包的路径，我更喜欢在mapper接口上加mapper注解
@SpringBootApplication
public class MybatisZhujieApplication {

    public static void main(String[] args) {
        SpringApplication.run(MybatisZhujieApplication.class, args);
    }
}
```

或者直接在Mapper类上面添加注解@Mapper

```java
@Mapper
//@Repository此注解可不加，加是防止在使用@Autowired注解时IDEA报错
public interface UserMapper {

    @Delete("drop table if exists user")
    void dropTable();

    @Insert("CREATE TABLE IF NOT EXISTS user(id INT UNSIGNED AUTO_INCREMENT,name VARCHAR(100) NOT NULL," +
            "  age INT NOT NULL,PRIMARY KEY (id)" +
            ")ENGINE=InnoDB DEFAULT CHARSET=utf8;")
    void createTable();

    @Insert("insert into user(name,age) values(#{name},#{age})")
    void insert(User user);

    @Select("select id,name,age from user")
    List<User> findAll();
}
```

# 5.测试使用

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class MybatisZhujieApplicationTests {

    @Autowired
    UserMapper userMapper;

    // 每次执行Test之前先删除表，创建表
    @Before
    public void before() throws Exception {
        userMapper.dropTable();
        userMapper.createTable();
    }
    @Test
    public void contextLoads() {
        userMapper.insert(new User("name",18));
        userMapper.insert(new User("name2",19));
        userMapper.insert(new User("name3",20));
        System.out.println(userMapper.findAll());
    }


}
```

执行结果：

`[User{id=1, name='name', age=18}, User{id=2, name='name2', age=19}, User{id=3, name='name3', age=20}]`

---

2018-6-3

# 二：XML配置方式

---

# 1.mybatis-config配置

在resources下新建mybatis-config.xml

```java
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

# 2.Entity实体类

示例：

```java
package com.example.entity;

import java.util.Date;

/**
 * Created by 巅峰小学生 2018年3月11日 下午1:21:17
*/
public class Area {
    //主键ID
    private Integer areaId;
    //名称
    private String areaName;
    //权重，越大越靠前显示
    private Integer priority;
    //创建时间
    private Date createTime;
    //更新时间
    private Date lastEditTime;
    ...


}
```

# 3.Dao层接口

```java
package com.example.dao;

import java.util.List;
import com.example.entity.Area;

/**
 * Created by 巅峰小学生 2018年3月11日 下午7:12:04
*/
public interface AreaDao {
    /**
     * 列出区域列表
     * 
     * @return areaList
     */
    List<Area> queryArea();

    /**
     * 根据Id列出具体区域
     * 
     * @return area
     */
    Area queryAreaById(int areaId);

    /**
     * 插入区域信息
     * 
     * @param area
     * @return
     */
    int insertArea(Area area);

    /**
     * 更新区域信息
     * 
     * @param area
     * @return
     */
    int updateArea(Area area);

    /**
     * 删除区域信息
     * 
     * @param areaId
     * @return
     */
    int deleteArea(int areaId);
}
```

# 4.Mapper映射

在/resources/mapper/下新建AreaDao.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <!-- 声明接口地址 -->
    <mapper namespace="com.example.dao.AreaDao">

    <!-- 接口方法 -->
    <select id="queryArea" resultType="com.example.entity.Area">
        SELECT area_id, area_name,
        priority, create_time, last_edit_time
        FROM tb_area
        ORDER BY priority
        DESC
    </select>
    <select id="queryAreaById" resultType="com.example.entity.Area">
        SELECT area_id, area_name,
        priority, create_time, last_edit_time
        FROM tb_area
        WHERE
        area_id=#{areaId}
    </select>
    <insert id="insertArea" useGeneratedKeys="true" keyProperty="areaId"
        keyColumn="area_id" parameterType="com.example.entity.Area">
        INSERT INTO
        tb_area(area_name,priority,
        create_time,last_edit_time)
        VALUES
        (#{areaName},#{priority},
        #{createTime},#{lastEditTime})
    </insert>
    <update id="updateArea" parameterType="com.example.entity.Area">
        update tb_area
        <set>
            <if test="areaName != null">area_name=#{areaName},</if>
            <if test="priority != null">priority=#{priority},</if>
            <if test="lastEditTime != null">last_edit_time=#{lastEditTime}</if>
        </set>
        where area_id=#{areaId}
    </update>
    <delete id="deleteArea">
        DELETE FROM
        tb_area
        WHERE
        area_id =
        #{areaId}
    </delete>
</mapper>
```

# 5.application.properties自定义配置

```java
#mysql
jdbc.driverClass=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=utf-8&useSSL=false
jdbc.username=root
jdbc.password=123456
#Mybatis
mybatis_config_file=mybatis-config.xml
mapper_path=/mapper/**.xml
type_alias_package=com.example.entity
```

# 6.配置SqlSessionFactoryBean

DataSourceBean

```java
import java.beans.PropertyVetoException;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.mchange.v2.c3p0.ComboPooledDataSource;

@Configuration
//配置MyBatis mapper的扫描路径
@MapperScan("com.example.dao")
public class DataSourceConfiguration {

    @Value("${jdbc.driverClass}")
    private String jdbcDriverClass;
    @Value("${jdbc.url}")
    private String jdbcUrl;
    @Value("${jdbc.username}")
    private String jdbcUser;
    @Value("${jdbc.password}")
    private String jdbcPassword;

    @Bean("dataSource")
    public ComboPooledDataSource createDataSource() throws PropertyVetoException{
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setDriverClass(jdbcDriverClass);
        dataSource.setJdbcUrl(jdbcUrl);
        dataSource.setUser(jdbcUser);
        dataSource.setPassword(jdbcPassword);
        //关闭连接后不自动commit
        dataSource.setAutoCommitOnClose(false);
        return dataSource;
    }

}
```

SqlSessionFactoryBean

```java
import javax.sql.DataSource;

import org.mybatis.spring.SqlSessionFactoryBean;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.core.io.support.ResourcePatternResolver;

/**
 * Created by 巅峰小学生
 *  2018年3月11日 下午2:20:19
*/
@Configuration
public class SessionFactoryConfiguration {
    @Value("${mybatis_config_file}")
    private String mybatisConfigFile;
    @Value("${mapper_path}")
    private String mapperPath;
    // 实体类所在的package
    @Value("${type_alias_package}")
    private String typeAliasPackage;
    @Autowired
    @Qualifier("dataSource")
    private DataSource dataSource;

    @Bean("sqlSessionFactory")
    public SqlSessionFactoryBean creatSqlSessionFactoryBean() throws IOException{
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        // 设置mybatis configuration 扫描路径
        sqlSessionFactoryBean.setConfigLocation(new ClassPathResource(mybatisConfigFile));
        // 添加mapper 扫描路径
        PathMatchingResourcePatternResolver pathMatchingResourcePatternResolver = new PathMatchingResourcePatternResolver();
        String packageSearchPath = ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX + mapperPath;
        sqlSessionFactoryBean.setMapperLocations(pathMatchingResourcePatternResolver.getResources(packageSearchPath));
        // 设置dataSource
        sqlSessionFactoryBean.setDataSource(dataSource);
        // 设置typeAlias 包扫描路径
        sqlSessionFactoryBean.setTypeAliasesPackage(typeAliasPackage);
        return sqlSessionFactoryBean;
    }

}
```

# 7.开始使用

```java
@Service
public class AreaServiceImpl implements AreaService {
    @Autowired
    private AreaDao areaDao;

    @Override
    public List<Area> getAreaList() {
        // 返回所有的区域信息
        return areaDao.queryArea();
    }

    @Override
    public Area getAreaById(int areaId) {
        return areaDao.queryAreaById(areaId);
    }
    ....
```