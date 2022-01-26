---
layout: post
title: Maven项目pom.xml文件简单解析
date: 2018-02-24 21:48:00
author: 薛师兄
tags: Maven
---
#### Maven项目pom.xml简单解析

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion><!-- 当前pom版本 -->

    <groupId>com.example.hello</groupId><!-- 反写的公司网址+项目名 -->
    <artifactId>hello_maven</artifactId><!-- 项目名+模块名 -->
    <version>0.0.1-SNAPSHOT</version><!-- 版本号 第一个0表示大版本号，第二个0表示大版本号，第三个0表示大版本号 
        0.0.1snapshot快照 (snapshot 快照/alpha 内部测试/beta 公测/Release稳定/GA正式发布) -->
    <packaging>jar</packaging><!-- 打包方式 默认是jar (war zip pom) -->

    <name>hello-maven</name><!-- 项目描述名 -->
    <url>http://maven.apache.org</url><!-- 项目地址 -->

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <!-- maven继承 -->
    <parent>
        <groupId>com.example</groupId>
        <artifactId>hello_maven_parent</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>

    <dependencies>
        <!-- 依赖传递 依赖冲突 -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <!-- 排除依赖传递：排除引用的junit的某些依赖 -->
              <exclusions>
                <exclusion>
                  <groupId>***</groupId>
                  <artifactId>***</artifactId>
                </exclusion>
              </exclusions>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <!-- 定义插件:打包带项目源码(对webApp无效) -->
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-source-plugin</artifactId>
                <version>2.4</version>
                <!-- 定义插件的执行 -->
                <executions>
                    <execution>
                        <!-- 执行时期 -->
                        <phase>package</phase>
                        <!-- 执行目标 -->
                        <goals>
                            <goal>jar-no-fork</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.2</version>
                <!-- <groupId>org.eclipse.jetty</groupId> <artifactId>jetty-maven-plugin</artifactId> 
                    <version>9.4.8.v20171121</version> -->
                <!--在打包成功后运行jetty:run -->
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>run</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```