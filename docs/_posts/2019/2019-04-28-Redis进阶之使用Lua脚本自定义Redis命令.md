---
layout: post
title: Redis进阶之使用Lua脚本自定义Redis命令
date: 2019-04-28 19:13:00
author: 薛师兄
tags:
  - Redis
  - Lua
---
## 1.在Redis中使用Lua

在Redis中执行Lua脚本有两种方法：eval和evalsha。

###1.1 eval

```java
eval 脚本内容 key个数 key列表 参数列表
```

下面例子使用了key列表和参数列表来为Lua脚本提供更多的灵活性：

```shell
127.0.0.1:6379> eval 'return "hello " .. KEYS[1] .. ARGV[1]' 1 redis world
"hello redisworld"
```


此时KEYS[1]="redis"，ARGV[1]="world"，所以最终的返回结果是"hello redisworld"。

如果Lua脚本较长，还可以使用redis-cli--eval直接执行文件。

```shell
$ redis-cli --eval hello.lua mykey , myargv
```

注意，这种方式不需要指定key的数量，用 , 号划分key和arg，注意逗号左右的空格。

eval命令和--eval参数本质是一样的，客户端如果想执行Lua脚本，首先在客户端编写好Lua脚本代码，然后把脚本作为字符串发送给服务端，服务端会将执行结果返回给客户端。

### 1.2 evalsha

除了使用eval，Redis还提供了evalsha命令来执行Lua脚本。

首先要将Lua脚本加载到Redis服务端，得到该脚本的SHA1校验和，evalsha命令使用SHA1作为参数可以直接执行对应Lua脚本，避免每次发送Lua脚本的开销。这样客户端就不需要每次执行脚本内容，而脚本也会常驻在服务端，脚本功能得到了复用。

#### 加载脚本

script load命令可以将脚本内容加载到Redis内存中，例如下面将lua_get.lua加载到Redis中，得到SHA1为："7413dc2440db1fea7c0a0bde841fa68eefaf149c"

```shell
$ redis-cli script load "$(cat lua_get.lua)"
"7413dc2440db1fea7c0a0bde841fa68eefaf149c"
```

#### 执行脚本

evalsha的使用方法如下，参数使用SHA1值，执行逻辑和eval一致。

```java
evalsha 脚本SHA1值 key个数 key列表 参数列表
```

所以只需要执行如下操作，就可以调用lua_get.lua脚本：

```java
127.0.0.1:6379> evalsha 7413dc2440db1fea7c0a0bde841fa68eefaf149c 1 redis world
"hello redisworld"
```

## 2.Lua的RedisAPI

Lua可以使用redis.call函数实现对Redis的访问，例如下面代码是Lua使用redis.call调用了Redis的set和get操作：

```java
redis.call("set", "hello", "world")
redis.call("get", "hello")
```

放在Redis的执行效果如下：

```shell
127.0.0.1:6379> eval 'return redis.call("get", KEYS[1])' 1 hello
"world"
```

除此之外Lua还可以使用redis.pcall函数实现对Redis的调用，redis.call和redis.pcall的不同在于，如果redis.call执行失败，那么脚本执行结束会直接返回错误，而redis.pcall会忽略错误继续执行脚本，所以在实际开发中要根据具体的应用场景进行函数的选择。

获取KEY可以通过 KEYS[1]，获取 Value 可以通过 ARGV[1] 。

## 3.开发提示

Lua可以使用redis.log函数将Lua脚本的日志输出到Redis的日志文件中，但是一定要控制日志级别。
Redis3.2提供了Lua Script Debugger功能用来调试复杂的Lua脚本，具体可以参考：http://redis.io/topics/ldb。

```java
redis.log(redis.LOG_DEBUG,key1)
```

redis.LOG_DEBUG

redis.LOG_VERBOSE

redis.LOG_NOTICE

redis.LOG_WARNING

> 本文并没有详细讲解如何读取复杂参数以及结合程序开发，详细可以参考我的另一篇文章：
> 
> [SpringBoot通过RedisTemplate执行Lua脚本](/)