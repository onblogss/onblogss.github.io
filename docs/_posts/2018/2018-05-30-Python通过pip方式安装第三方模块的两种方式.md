---
layout: post
title: Python通过pip方式安装第三方模块的两种方式
date: 2018-05-30 15:21:00
author: 薛师兄
tags: Python
---
# 一：环境

*  python3.6
*  windows 10

---

# 二：常用命令

*  如果直接执行pip命令报错，说明pip不在path环境变量中
*  解决方法：

```javascript
python -m pip list
```

*  以下默认可直接使用pip

###### 1.查看已安装的模块

```javascript
pip list
```

###### 2.安装模块

```javascript
pip install 模块名
```

###### 3.卸载模块

```java
pip uninstall 模块名
```

##### Python第三方库地址

*  收藏本页 ： [https://pypi.org/](https://pypi.org/)

---

# 三：安装whl文件

*  如果pip安装编译失败，可以下载编译好的二进制文件再安装：

##### Python Whl库地址

[https://www.lfd.uci.edu/~gohlke/pythonlibs/](https://www.lfd.uci.edu/~gohlke/pythonlibs/)

![](./20180530Python通过pip方式安装第三方模块的两种方式/1136672-20190623141700863-1943886782.png)

*  下载第一个xxxxx-win32.whl，解压到本地。
*  执行

```java
pip install D:<span class="hljs-command">\本</span>地路径<span class="hljs-command">\xxx</span>.whl
```

*  失败试试第二个

---

![](./20180530Python通过pip方式安装第三方模块的两种方式/1136672-20190623141717895-943511979.png)