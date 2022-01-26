---
layout: post
title: eclipse不能运行Struts2项目
date: 2017-11-20 13:32:00
author: 薛师兄
tags:
  - eclipse
  - Struts2
---
刚接触Struts2项目，本想写个HelloWorld上手，谁知道光eclipse配置tomcat就鼓捣一晚上，查阅各种资料。

项目刚开始报错：

"java.lang.ClassNotFoundException:org.apache.commons.lang.StringUtils"

页面404

提示少包，于是我去检查包，没少。于是我又去把项目打包成war，关掉eclispe单独启动Tomcat7，运行没问题。

所以，代码没问题。那就是eclipse和tomcat两个在一起除了问题。

**解决办法：**

**删除eclipse工作空间workspace目录下的.metadata文件夹，这样eclipse就初始化了，然后在eclipse中配置tomcat服务器，运行Struts2项目，OK！**

备注：

eclipse删除配置后，工作空间还是改回原来的，项目也可以重新导入。如果运行tomcat不启动导入的项目，右击eclipse下方Servers视图的Tomcat，点击AddAndRemove，把项目添加进去就可以！