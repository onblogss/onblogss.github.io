---
layout: post
title: Java获取文件Content-Type的四种方法
date: 2018-09-08 00:22:00
author: 薛师兄
tags: Java
---
> [HTTP Content-Type在线工具](http://tool.oschina.net/commons) 

有时候我们需要获取本地文件的Content-Type，已知 Jdk 自带了三种方式来获取文件类型。

另外还有第三方包 Magic 也提供了API。Magic依赖：

```java
<dependency>
    <groupId>net.sf.jmimemagic</groupId>
    <artifactId>jmimemagic</artifactId>
    <version>0.1.4</version>
</dependency>
```

下面我们来通过单元测试看下这四种方式的效果。主要代码：

```java
@Test
public void test() {
    String pathname = "D:\\...";

    try {
        Magic parser = new Magic() ;
        MagicMatch match = parser.getMagicMatch(new File(pathname),false);
        System.out.println("第一种Magic: " + match.getMimeType()) ;
    } catch (MagicParseException e) {
        e.printStackTrace();
    } catch (MagicMatchNotFoundException e) {
        e.printStackTrace();
    } catch (MagicException e) {
        e.printStackTrace();
    }

    String type = new MimetypesFileTypeMap().getContentType(new File(pathname));
    System.out.println("第二种javax.activation: "+type);

    try {
        String s = Files.probeContentType(new File(pathname).toPath());
        System.out.println("第三种java.nio: "+s);
    } catch (IOException e) {
        e.printStackTrace();
    }

    FileNameMap fileNameMap = URLConnection.getFileNameMap();
    String contentType = fileNameMap.getContentTypeFor(pathname);
    System.out.println("第四种java.net: "+contentType);
}
```

首先，新建文本文件更名为new.json，测试。

```java
log4j:WARN No appenders could be found for logger (net.sf.jmimemagic.Magic).
log4j:WARN Please initialize the log4j system properly.
net.sf.jmimemagic.MagicMatchNotFoundException
....
第二种javax.activation: application/octet-stream
第三种java.nio: null
第四种java.net: null
```

可以看到，Magic直接抛了异常。javax.activation提示不知道的二进制流。nio 和 net 报null。就此次来说，第2，3, 4种方法对未知类型的处理都可以。唯有第一种不令人满意。

接下来随便写入字符串到new.json文件。然后运行。

```java
第一种Magic: text/plain
第二种javax.activation: application/octet-stream
第三种java.nio: null
第四种java.net: null
```

Magic提示的是文本类型，javax.activation提示不知道的二进制流。nio 和 net 方式直接为null。Magic胜出。

接下来把new.json文件改名为new.xml文件。再次运行。

```java
第一种Magic: text/plain
第二种javax.activation: application/octet-stream
第三种java.nio: text/xml
第四种java.net: application/xml
```

javax.activation又提示不知道的二进制流。其它几个提示的都差不多是文本类型，还算靠谱。

再把文件更名为new.html，运行。

```java
第一种Magic: text/plain
第二种javax.activation: text/html
第三种java.nio: text/html
第四种java.net: text/html
```

可以看到，除了Magic提示文本类型，其它都是html，很准确。

再次把文件改为new.png，运行。

```java
第一种Magic: text/plain
第二种javax.activation: application/octet-stream
第三种java.nio: image/png
第四种java.net: image/png
```

我们再把pathname改为一张真实图片的地址，运行。

```java
第一种Magic: image/png
第二种javax.activation: application/octet-stream
第三种java.nio: image/png
第四种java.net: image/png
```

这时Magic的运行结果和上次又不相同。可见，Magic并不是通过文件扩展名来对文件类型进行判定的。

## 总结

nio 和 net的方式区别不大。Magic的异常需要注意。javax.activation大部分只报application/octet-stream。

建议使用nio 与javax.activation 结合的方法。代码如下：

```java
public String getContentType() {
    //利用nio提供的类判断文件ContentType
    Path path = Paths.get(getUri());
    String content_type = null;
    try {
        content_type = Files.probeContentType(path);
    } catch (IOException e) {
        logger.error("Read File ContentType Error");
    }
    //若失败则调用另一个方法进行判断
    if (content_type == null) {
        content_type = new MimetypesFileTypeMap().getContentType(new File(getUri()));
    }
    return content_type;
}
```