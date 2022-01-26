---
layout: post
title: JSOUP如何优秀的下载JPEG等二进制图像
date: 2018-07-26 21:27:00
author: 薛师兄
tags: Jsoup
---
## 引言

*  JSOUP默认是不支持解析JPEG等二进制图像的，解决方法也很简单，只需要加上`Jsoup.ignoreContentType(true)`这一行代码就可以。关于这一点的原因，来看看官方API说明。
*  [Connection (jsoup Java HTML Parser 1.11.3 API)](https://jsoup.org/apidocs/org/jsoup/Connection.html)

## 解释

*  连接ignoreContentType(boolean ignoreContentType) 在解析响应时忽略文档的内容类型。默认情况下，这是错误的，未识别的内容类型将导致抛出IOException。(例如，通过尝试解析JPEG二进制映像来防止产生垃圾)。设置为true以强制执行解析尝试，而不考虑内容类型。
*  参数: ignoreContentType&mdash;如果您想要将响应解析为文档时忽略的内容类型设置为true。
*  返回: 这个连接,链接

## 完整示例

这个示例是完整下载一张 [图片](http://sjbz.fd.zol-img.com.cn/t_s640x960c/g5/M00/0F/09/ChMkJlfJQcWIDXJEAAN5CfxwAOYAAU7hwBVxTQAA3kh337.jpg) 的所有步骤。

```java
@Test
public void test() throws IOException {
    Response response = Jsoup.connect("http://sjbz.fd.zol-img.com.cn/t_s640x960c/g5/M00/0F/09/ChMkJlfJQcWIDXJEAAN5CfxwAOYAAU7hwBVxTQAA3kh337.jpg")
            .ignoreContentType(true)
            .method(Method.GET)
            .execute();
    byte[] bytes = response.bodyAsBytes();
    File file = new File("D:\\img.png");
    FileOutputStream fileOutputStream = new FileOutputStream(file);
    fileOutputStream.write(bytes);
    fileOutputStream.flush();
    fileOutputStream.close();
}
```

上面的方法很简单，但是我并不推荐使用。

>*  原因：
>*  我们有必要知道不带缓冲的操作，每读一个字节就要写入一个字节，由于涉及磁盘的IO操作相比内存的操作要慢很多，所以不带缓冲的流效率很低。带缓冲的流，可以一次读很多字节，但不向磁盘中写入，只是先放到内存里。等凑够了缓冲区大小的时候一次性写入磁盘，这种方式可以减少磁盘操作次数，速度就会提高很多！

## 完美方案

```java
@Test
public void test() throws IOException {
    Response response = Jsoup.connect("http://sjbz.fd.zol-img.com.cn/t_s640x960c/g5/M00/0F/09/ChMkJlfJQcWIDXJEAAN5CfxwAOYAAU7hwBVxTQAA3kh337.jpg")
            .ignoreContentType(true)
            .method(Method.GET)
            .execute();
    //声明缓冲字节输入流
    BufferedInputStream bufferedInputStream = response.bodyStream();
    //缓冲字节输出流-》文件字节输出流-》文件
    File file = new File("D:\\img.png");
    FileOutputStream fileOutputStream = new FileOutputStream(file);
    BufferedOutputStream bufferedOutputStream = new BufferedOutputStream(fileOutputStream);
    //把缓冲字节输入流写入到输出流
    byte[] b = new byte[1024]; //每次最多读1KB的大小
    int length; //实际读入的字节
    while ((length = bufferedInputStream.read(b))!=-1){
        //写入到输出流
        bufferedOutputStream.write(b,0,length);
    }
    //刷新缓冲的输出流。这将强制将任何缓冲的输出字节写入底层输出流。
    bufferedOutputStream.flush();
    bufferedInputStream.close();
}
```