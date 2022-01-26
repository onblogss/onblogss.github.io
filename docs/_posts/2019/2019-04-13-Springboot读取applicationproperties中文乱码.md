---
layout: post
title: SpringBoot读取application.properties中文乱码
date: 2019-04-13 23:32:00
author: 薛师兄
tags: SpringBoot
---
解决方案

在IDEA环境下：

File -> Settings -> Editor -> File Encodings

将Properties Files (*.properties)下的Default encoding for properties files设置为UTF-8，将Transparent native-to-ascii conversion前的勾选上。如图所示：

![](./20190413Springboot读取applicationproperties中文乱码/1136672-20190413233056897-401978446.jpg)

> 注意：做了上面操作后，一定要重新创建 application.properties 才有效！

*原文：https://www.cnblogs.com/diffx/p/9866717.html*