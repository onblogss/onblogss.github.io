---
layout: post
title: SpringBoot通过RedisTemplate执行Lua脚本
date: 2019-04-28 19:50:00
author: 薛师兄
tags:
  - SpringBoot
  - Lua
---
如果你对Redis和Lua的关系不太清楚，请先阅读：[Redis进阶之使用Lua脚本开发](/)

## 1.RedisScript

首先你得引入spring-boot-starter-data-redis依赖，其次把lua脚本放在resources目录下。

```java
@Bean
public DefaultRedisScript<List> defaultRedisScript() {
    DefaultRedisScript<List> defaultRedisScript = new DefaultRedisScript<>();
    defaultRedisScript.setResultType(List.class);
//   defaultRedisScript.setScriptText("");
    defaultRedisScript.setScriptSource(new ResourceScriptSource(new ClassPathResource("redis/demo.lua")));
    return defaultRedisScript;
}
```

在Spring Boot2.0的时候，上述配置没有问题，但在Spring Boot1.5测试会出错，需要将List.class改为具体的返回类型（如Long.class）。

RedisScript的getSha1()方法可以获取脚本摘要。 

## 2.调用脚本

```java
/**
* List设置lua的KEYS
*/
List<String> keyList = new ArrayList();
keyList.add("count");
keyList.add("rate.limiting:127.0.0.1");

/**
* 用Mpa设置Lua的ARGV[1]
*/
Map<String, Object> argvMap = new HashMap<String, Object>();
argvMap.put("expire", 10000);
argvMap.put("times", 10);

/**
* 调用脚本并执行
*/
List result = redisTemplate1.execute(redisScript, keyList, argvMap);
System.out.println(result);
```

若是出现序列化问题，可以指定序列化方式。

```java
public <T> T execute(RedisScript<T> script, RedisSerializer<?> argsSerializer, RedisSerializer<T> resultSerializer,
		List<K> keys, Object... args) {
	return scriptExecutor.execute(script, argsSerializer, resultSerializer, keys, args);
}
```

## 3.Lua脚本

```java
--获取KEY
local key1 = KEYS[1]
local key2 = KEYS[2]
 
-- 获取ARGV[1],这里对应到应用端是一个List<Map>.
--  注意，这里接收到是的字符串，所以需要用csjon库解码成table类型
local receive_arg_json =  cjson.decode(ARGV[1])
 
--获取ARGV内的参数并打印
local expire = receive_arg_json.expire
local times = receive_arg_json.times
```

> 参考：https://blog.csdn.net/fsw4848438/article/details/81540495