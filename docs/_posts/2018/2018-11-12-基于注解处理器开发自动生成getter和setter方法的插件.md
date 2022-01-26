---
layout: post
title: 基于注解处理器开发自动生成getter和setter方法的插件
date: 2018-11-12 13:24:00
author: 薛师兄
tags:
  - lombok
  - Java
---
昨天无意中，逛到了[lombok](https://www.projectlombok.org/)的网站，并看到了首页的5分钟视频，视频中的作者只是在实体类中写了几个字段，就可以自动编译为含setter、getter、toString()等方法的class文件。看着挺新奇的，于是自己研究了一下原理，整理下发出来。

# 1.何处下手

视频中作者的流程为：

(1)编写Java文件，在类上写@Data注解

```java
@Data
public class Demo {
    private String name;
    private double abc;
}
```

(2)javac编译，lombok.jar是lombok的jar包。

```java
javac -cp lombok.jar Demo.java
```

(3)javap查看Demo.class类文件

```java
javap Demo
```
Demo.class：
```java
public class Demo {
  public Demo();
  public java.lang.String getName();
  public void setName(java.lang.String);
  public double getAbc();
  public void setAbc(double);
}
```

可以看到Demo.class内部竟然多了很多未定义的setter、getter方法，而视频作者主要使用的就是注解+编译，那么我们就从这方面入手。

# 2.必备知识

## 2.1 注解

注解，相信大部分人都用过，不少人还会自定义注解，并会利用反射等搞点小东西。但本文所讲的并非是利用注解加反射在运行期自定义行为，而是在编译期。

自定义注解离不开四大元注解。

@Retention：注解保留时期

| 保留类型 | 说明                                 |
| -------- | ------------------------------------ |
| SOURCE   | 只保留到源码中,编译出来的class不存在 |
| CLASS    | 保留到class文件中,但是JVM不会加载    |
| RUNTIME  | 一直存在,JVM会加载,可用反射获取      |

@Target：用于标记可以应用于哪些类型上

| 元素类型       | 适用场合          |
| -------------- | ----------------- |
| ANOTATION_TYPE | 注解类型声明      |
| PACKAGE        | 包                |
| TYPE           | 类,枚举,接口,注解 |
| METHOD         | 方法              |
| CONSTRUCTOR    | 构造方法         |
| FIELD          | 成员域,枚举常量  |
| PARAMETER      | 方法或构造器参数 |
| LOCAL_VARIABLE | 局部变量         |
| TYPE_PARAMETER | 类型参数 |
| TYPE_USE       | 类型用法 |

@Documented：作用是能够将注解中的元素包含到 Javadoc 中

@Inherited：继承。假设注解A使用了此注解，那么类B使用了注解A，类C继承了类B，那么类C也使用了注解A。（这里的**使用**是为了区分易理解，实际为**被注解**）

## 2.1 注解处理器

注解处理器就是 javac 包中专门用来处理注解的工具。所有的注解处理器都必须继承抽象类`AbstractProcessor`然后重写它的几个方法。

注解处理器是运行在它自己的JVM中。javac 启动一个完整Java虚拟机来运行注解处理器，这意味着你可以使用任何你在其他java应用中使用的的东西。其中抽象方法`process`是必须要重写的，再该方法中注解处理器可以遍历所有的源文件，然后通过`RoundEnvironment`类获取我们需要处理的注解所标注的所有的元素，这里的元素可以代表包，类，接口，方法，属性等。再处理的过程中可以利用特定的工具类自动生成特定的.java文件或者.class文件，来帮助我们处理自定义注解。 

一个普通的注解处理器文件如下：

```java
package com.example;

import java.util.LinkedHashSet;
import java.util.Set;
import javax.annotation.processing.AbstractProcessor;
import javax.annotation.processing.ProcessingEnvironment;
import javax.annotation.processing.RoundEnvironment;
import javax.annotation.processing.SupportedAnnotationTypes;
import javax.annotation.processing.SupportedSourceVersion;
import javax.lang.model.SourceVersion;
import javax.lang.model.element.TypeElement;

public class MyProcessor extends AbstractProcessor {

    @Override
    public boolean process(Set<? extends TypeElement> annoations,
            RoundEnvironment env) {
        return false;
    }

    @Override
    public Set<String> getSupportedAnnotationTypes() {
        Set<String> annotataions = new LinkedHashSet<String>();
        annotataions.add("com.example.MyAnnotation");
        return annotataions;
    }

    @Override
    public SourceVersion getSupportedSourceVersion() {
        return SourceVersion.latestSupported();
    }

    @Override
    public synchronized void init(ProcessingEnvironment processingEnv) {
        super.init(processingEnv);
    }

}
```

- `init(ProcessingEnvironment processingEnv)` ：所有的注解处理器类都必须有一个无参构造函数。然而，有一个特殊的方法init()，它会被注解处理工具调用，以ProcessingEnvironment作为参数。ProcessingEnvironment 提供了一些实用的工具类Elements, Types和Filer。
- `process(Set<? extends TypeElement> annoations, RoundEnvironment env)` ：这类似于每个处理器的main()方法。你可以在这个方法里面编码实现扫描，处理注解，生成 java 文件。使用RoundEnvironment参数，你可以查询被特定注解标注的元素。
- `getSupportedAnnotationTypes()`：在这个方法里面你必须指定哪些注解应该被注解处理器注册。注意，它的返回值是一个String集合，包含了你的注解处理器想要处理的注解类型的全称。换句话说，你在这里定义你的注解处理器要处理哪些注解。
- `getSupportedSourceVersion()` ： 用来指定你使用的 java 版本。通常你应该返回`SourceVersion.latestSupported()` 。不过，如果你有足够的理由坚持用 java 6 的话，你也可以返回`SourceVersion.RELEASE_6`。

关于`getSupportedAnnotationTypes()`和`getSupportedSourceVersion()`这两个方法，你也可以使用相应注解进行代替。代码如下：

```java
@SupportedSourceVersion(SourceVersion.RELEASE_8)
@SupportedAnnotationTypes("com.example.MyAnnotation")
public class MyProcessor extends AbstractProcessor {
....
```

不过为了兼容Java6，最好是重载这俩方法。

# 3.开始编码

知识我们已经学会，现在开始实战。

## 3.1 自定义注解

```java
@Retention(RetentionPolicy.CLASS)
@Target(ElementType.TYPE)
public @interface Data {

}
```

## 3.2 自定义注解处理器

```java
public class DataAnnotationProcessor extends AbstractProcessor {
    private Messager messager; //用于打印日志
    private Elements elementUtils; //用于处理元素
    private Types typeUtils;
    private Filer filer;  //用来创建java文件或者class文件

    @Override
    public synchronized void init(ProcessingEnvironment processingEnv) {
        super.init(processingEnv);
        messager = processingEnv.getMessager();
        elementUtils = processingEnv.getElementUtils();
        filer = processingEnv.getFiler();
        typeUtils = processingEnvironment.getTypeUtils();
    }
    
    @Override
    public SourceVersion getSupportedSourceVersion() {
        return SourceVersion.latestSupported();
    }

    @Override
    public Set<String> getSupportedAnnotationTypes(){
        Set<String> set = new HashSet<>();
        set.add(Data.class.getCanonicalName());
        return Collections.unmodifiableSet(set);
    }

    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        messager.printMessage(Diagnostic.Kind.NOTE,"-----开始自动生成源代码");
        try {
            // 标识符
            boolean isClass = false;
            // 类的全限定名
            String classAllName = null;
            // 返回被注释的节点
            Set<? extends Element> elements = roundEnv.getElementsAnnotatedWith(Data.class);
            Element element = null;
            for (Element e : elements) {
                // 如果注释在类上
                if (e.getKind() == ElementKind.CLASS && e instanceof TypeElement) {
                    TypeElement t = (TypeElement) e;
                    isClass = true;
                    classAllName = t.getQualifiedName().toString();
                    element = t;
                    break;
                }
            }
            // 未在类上使用注释则直接返回，返回false停止编译
            if (!isClass) {
                return true;
            }
            // 返回类内的所有节点
            List<? extends Element> enclosedElements = element.getEnclosedElements();
            // 保存字段的集合
            Map<TypeMirror, Name> fieldMap = new HashMap<>();
            for (Element ele : enclosedElements) {
                if (ele.getKind() == ElementKind.FIELD) {
                    //字段的类型
                    TypeMirror typeMirror = ele.asType();
                    //字段的名称
                    Name simpleName = ele.getSimpleName();
                    fieldMap.put(typeMirror, simpleName);
                }
            }
            // 生成一个Java源文件
            JavaFileObject sourceFile = filer.createSourceFile(getClassName(classAllName));
            // 写入代码
            createSourceFile(classAllName, fieldMap, sourceFile.openWriter());
            // 手动编译
            compile(sourceFile.toUri().getPath());
        } catch (IOException e) {
            messager.printMessage(Diagnostic.Kind.ERROR,e.getMessage());
        }
        messager.printMessage(Diagnostic.Kind.NOTE,"-----完成自动生成源代码");
        return true;
    }

    private void createSourceFile(String className, Map<TypeMirror, Name> fieldMap, Writer writer) throws IOException {
        // 生成源代码
        JavaWriter jw = new JavaWriter(writer);
        jw.emitPackage(getPackage(className));
        jw.beginType(getClassName(className), "class", EnumSet.of(Modifier.PUBLIC));
        for (Map.Entry<TypeMirror, Name> map : fieldMap.entrySet()) {
            String type = map.getKey().toString();
            String name = map.getValue().toString();
            //字段
            jw.emitField(type, name, EnumSet.of(Modifier.PRIVATE));
        }
        for (Map.Entry<TypeMirror, Name> map : fieldMap.entrySet()) {
            String type = map.getKey().toString();
            String name = map.getValue().toString();
            //getter
            jw.beginMethod(type, "get" + humpString(name), EnumSet.of(Modifier.PUBLIC))
                    .emitStatement("return " + name)
                    .endMethod();
            //setter
            jw.beginMethod("void", "set" + humpString(name), EnumSet.of(Modifier.PUBLIC), type, "arg")
                    .emitStatement("this." + name + " = arg")
                    .endMethod();
        }
        jw.endType().close();
    }

    /**
     * 编译文件
     * @param path
     * @throws IOException
     */
    private void compile(String path) throws IOException {
        //拿到编译器
        JavaCompiler complier = ToolProvider.getSystemJavaCompiler();
        //文件管理者
        StandardJavaFileManager fileMgr =
                complier.getStandardFileManager(null, null, null);
        //获取文件
        Iterable units = fileMgr.getJavaFileObjects(path);
        //编译任务
        JavaCompiler.CompilationTask t = complier.getTask(null, fileMgr, null, null, null, units);
        //进行编译
        t.call();
        fileMgr.close();
    }

    /**
     * 驼峰命名
     *
     * @param name
     * @return
     */
    private String humpString(String name) {
        String result = name;
        if (name.length() == 1) {
            result = name.toUpperCase();
        }
        if (name.length() > 1) {
            result = name.substring(0, 1).toUpperCase() + name.substring(1);
        }
        return result;
    }

    /**
     * 读取类名
     * @param name
     * @return
     */
    private String getClassName(String name) {
        String result = name;
        if (name.contains(".")) {
            result = name.substring(name.lastIndexOf(".") + 1);
        }
        return result;
    }

    /**
     * 读取包名
     * @param name
     * @return
     */
    private String getPackage(String name) {
        String result = name;
        if (name.contains(".")) {
            result = name.substring(0, name.lastIndexOf("."));
        }else {
            result = "";
        }
        return result;
    }
}
```

在自定义注解处理器中，注释非常详细的说明了每一步的思路，首先是读取被注释的节点，判断是否是类节点，然后生成Java源文件，并使用javawriter框架写入Java代码，最后手动编译该java源文件。

javawriter框架引用如下：

```java
compile 'com.squareup:javawriter:2.5.1'
```

## 3.3 注解处理器的注册

编码结束后，还需要把注解处理器注册到javac编译器，所以需要提供一个 .jar 文件。就像其他 .jar 文件一样，你将你已经编译好的注解处理器打包到此文件中。并且，在你的 .jar 文件中，你必须打包一个特殊的文件javax.annotation.processing.Processor到META-INF/services目录下。因此你的 .jar 文件目录结构看起来就你这样： 

```java
MyProcess.jar
    -com
        -example
            -MyProcess.class
    -META-INF
        -services
            -javax.annotation.processing.Processor
```

javax.annotation.processing.Processor 文件的内容是一个列表，每一行是一个注解处理器的全称。例如：

```java
com.example.MyProcess
```

在IDE中，只需在resources目录下新建META-INF/services/javax.annotation.processing.Processor文件即可。

**其它注册方式**

前面的注册方式很底层，个人推荐使用。当处理的注解处理器过多时，这种方式不免过于繁琐，所以另一种方式就是使用自动注册注解处理器的框架。

添加对谷歌自动注册注解库的引用

```java
implementation ‘com.google.auto.service:auto-service:1.0-rc4’
```

在注解处理器类前面声明

```java
@AutoService(Processor.class)
```

# 4.打包使用

此时我们把项目打包为jar包即可使用，下面演示下使用过程。

(1)写个Demo.java

```java
import com.example.annotation.Data;

@Data
public class Demo {
    private String name;
    private double abc;
}
```

(2)编译java文件，在该Demo.java文件夹下打开控制台窗口，记得把打包的jar包一起放在此目录。

```java
javac -cp annotation-1.0-SNAPSHOT.jar Demo.java
```

(3)使用javap查看编译后的Demo.class

```java
Compiled from "Demo.java"
public class Demo {
  public Demo();
  public double getAbc();
  public void setAbc(double);
  public java.lang.String getName();
  public void setName(java.lang.String);
}
```

再看此时的Demo.java代码

```java
public class Demo {
  private double abc;
  private String name;
  public double getAbc() {
    return abc;
  }
  public void setAbc(double arg) {
    this.abc = arg;
  }
  public String getName() {
    return name;
  }
  public void setName(String arg) {
    this.name = arg;
  }
}
```

到此，我们正式开发出了自动生成getter、setter方法的插件。有的小伙伴可能觉得这个并没有多大作用，用IDE快捷键就能非常轻易的办到。其实，知识已经学会，能做出什么多姿多彩的框架就要靠小伙伴们的智慧了。比如，我们一般都会新建Entity类，然后基于此新建Dao层，Service层代码，用本文所述知识足可以打造一款适合自己的代码生成器，节约时间，提高开发效率。

*附赠*

*[IntelliJ IDEA lombok插件的安装和使用](https://jingyan.baidu.com/article/0a52e3f4e53ca1bf63ed725c.html)*

### 补充

处理器最好作为Jar包依赖，打包失败可以指定版本。

``` xml
<plugins>
<plugin>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-compiler-plugin</artifactId>
<version>2.0.2</version>
<executions>
<execution>
<id>default-compile</id>
<configuration>
<compilerArgument>-proc:none</compilerArgument>
<source>1.8</source>
<target>1.8</target>
</configuration>
</execution>
<execution>
<id>default-testCompile</id>
<configuration>
<source>1.8</source>
<target>1.8</target>
</configuration>
</execution>
</executions>
</plugin>
</plugins>
```