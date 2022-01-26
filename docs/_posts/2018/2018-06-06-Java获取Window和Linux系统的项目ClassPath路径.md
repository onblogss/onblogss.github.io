---
layout: post
title: Java获取Window和Linux系统的项目ClassPath路径
date: 2018-06-06 15:54:00
author: 薛师兄
tags:
  - Java
  - Windows
  - Linux
---
## 不啰嗦，直接复制工具类

```java
/**
 * 在windows和linux系统下均可正常使用
 * Create by Martin 2018/6/6/006 14:51
 */
public class PathUtil {
    //获取项目的根路径
    public final static String classPath;

    static {
        //获取的是classpath路径，适用于读取resources下资源
        classPath = Thread.currentThread().getContextClassLoader().getResource("").getPath();
    }

    /**
     * 项目根目录
     */
    public static String getRootPath() {
        return RootPath("");
    }

    /**
     * 自定义追加路径
     */
    public static String getRootPath(String u_path) {
        return RootPath("/" + u_path);
    }

    /**
     * 私有处理方法
     */
    private static String RootPath(String u_path) {
        String rootPath = "";
        //windows下
        if ("\\".equals(File.separator)) {
            //System.out.println(classPath);
            rootPath = classPath + u_path;
            rootPath = rootPath.replaceAll("/", "\\\\");
            if (rootPath.substring(0, 1).equals("\\")) {
                rootPath = rootPath.substring(1);
            }
        }
        //linux下
        if ("/".equals(File.separator)) {
            //System.out.println(classPath);
            rootPath = classPath + u_path;
            rootPath = rootPath.replaceAll("\\\\", "/");
        }
        return rootPath;
    }

    //更多扩展方法任你发挥

}
```

## 使用方法

```java
//自定义追加路径并格式化
System.out.println(ProjectPath.getRootPath("userImg/test.txt"));
//获取根目录
System.out.println(ProjectPath.getRootPath());
```

## 注意

在使用`System.getProperty("user.dir")`时：

如果是在IDE中启动，则获得的路径为D:\xxxx\projectName,包括项目名；

如果是以Jar包方式启动，得到该jar包所在的路径。如project.jar在D:\xxxx下，获得的路径就是D:\xxxx

但是如果是以war包方式启动获得的是：D:\apache-tomcat-9.0.7\bin

所以此方法适合不依赖Tomcat容器（或者内嵌Tomcat如SpringBoot）的项目。