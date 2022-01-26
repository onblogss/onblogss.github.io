---
layout: post
title: 警告:Establishing SSL connection without server's identity verification is not recommended
date: 2018-06-04 00:59:00
author: 薛师兄
tags: MySQL
---
SpringBoot启东时红色警告：

*Mon Jun 04 00:53:48 CST 2018 WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification*

翻译：

>请注意:不建议在没有服务器身份验证的情况下建立SSL连接。根据MySQL 5.5.45+、5.6.26+和5.7.6+的要求，如果不设置显式选项，则必须建立默认的SSL连接。您需要通过设置useSSL=false显式地禁用SSL，或者设置useSSL=true并为服务器证书验证提供信任存储

**那问题来了，SSL是什么？**

**SSL**（Secure Socket Layer：安全套接字层）利用数据加密、身份验证和消息完整性验证机制，为基于TCP等可靠连接的应用层协议提供安全性保证。

SSL协议提供的功能主要有：

- 1、 数据传输的机密性：利用对称密钥算法对传输的数据进行加密。    
- 2、 身份验证机制：基于证书利用数字签名方法对服务器和客户端进行身份验证，其中客户端的身份验证是可选的。     
- 3、 消息完整性验证：消息传输过程中使用MAC算法来检验消息的完整性。

如果用户的传输不是通过SSL的方式，那么其在网络中数据都是以明文进行传输的，而这给别有用心的人带来了可乘之机。所以，现在很多大型网站都开启了SSL功能。同样地，在我们数据库方面，如果客户端连接服务器获取数据不是使用SSL连接，那么在传输过程中，数据就有可能被窃取。

这里我推荐阅读这篇博客：https://www.cnblogs.com/mysql-dba/p/7061300.html

**我们可以去查看我们的mysql是否开启了SSL：**



```shell
mysql> show global variables like '%ssl%';
+---------------+----------+
| Variable_name | Value    |
+---------------+----------+
| have_openssl  | DISABLED |
| have_ssl      | DISABLED |
| ssl_ca        |          |
| ssl_capath    |          |
| ssl_cert      |          |
| ssl_cipher    |          |
| ssl_crl       |          |
| ssl_crlpath   |          |
| ssl_key       |          |
+---------------+----------+
9 rows in set
```

我没开启，所以我的解决方法是：


```shell
jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=utf-8&useSSL=false
```

再次推荐大家阅读这篇博客：https://www.cnblogs.com/mysql-dba/p/7061300.html

深入了解SSL。