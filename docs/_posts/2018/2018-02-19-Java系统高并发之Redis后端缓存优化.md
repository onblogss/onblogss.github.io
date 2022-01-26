---
layout: post
title: Java系统高并发之Redis后端缓存优化
date: 2018-02-19 20:35:00
author: 薛师兄
tags:
  - Java
  - Redis
---
#### 一：前端优化

1. 暴露接口，按钮防重复（点击一次按钮后就变成禁用，禁止重复提交）
2. 采用CDN存储静态化的页面和一些静态资源（css，js等）

### 二：Redis后端缓存优化

1. Redis 是完全开源免费的，遵守BSD协议，是一个高性能的key-value数据库。
2. Redis支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。
3. 性能极高 &ndash; Redis能读的速度是110000次/s,写的速度是81000次/s 。
4. 原子 &ndash; Redis的所有操作都是原子性的，意思就是要么成功执行要么失败完全不执行。
5. 利用Redis可以减轻MySQL服务器的压力，减少了跟数据库服务器的通信次数。

#### 2.1 Redis服务端下载以及安装

详细步骤参考：[Redis服务端安装教程](http://www.runoob.com/redis/redis-install.html)

注：以下pom.xml为Maven项目配置文件，若非Maven项目，百度相应名称包导入即可。

#### 2.2 在pom.xml中配置Redis客户端

```xml
<!-- redis客户端:Jedis -->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.7.3</version>
</dependency>
```

由于Jedis并没有实现内部序列化操作，而Java内置的序列化机制性能又不高，我们需要考虑高并发优化，在这里我们采用开源社区提供的更高性能的自定义序列化工具protostuff。

#### 2.3 在pom.xml中配置protostuff依赖

```xml
<!-- protostuff序列化依赖 -->
<dependency>
    <groupId>com.dyuproject.protostuff</groupId>
    <artifactId>protostuff-core</artifactId>
    <version>1.0.8</version>
</dependency>
<dependency>
    <groupId>com.dyuproject.protostuff</groupId>
    <artifactId>protostuff-runtime</artifactId>
    <version>1.0.8</version>
</dependency>
```

#### 2.4 使用Redis优化数据库访问

1. 流程：先去Redis缓存中查询，以此降低数据库的压力。如果在缓存中查询不到数据再去数据库中查询，再将查询到的数据放入Redis缓存中，这样下次就可以直接去缓存中直接查询到。
2. 推荐：新建dao.cache包，实现RedisDao类。例子中缓存实体类名为Seckill.class。例子：
3. 使用protostuff序列化工具时，被序列化的对象必须是pojo对象（具备setter/getter）

```java
import com.dyuproject.protostuff.LinkedBuffer;
import com.dyuproject.protostuff.ProtostuffIOUtil;
import com.dyuproject.protostuff.runtime.RuntimeSchema;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
/*
 *  Redis可以近似理解为Map<Key,Value>对象
 */
public class RedisDao {
    private final Logger logger = LoggerFactory.getLogger(this.getClass());

    private final JedisPool jedisPool;

    public RedisDao(String ip, int port) {
        jedisPool = new JedisPool(ip, port);
    }

    private RuntimeSchema<Seckill> schema = RuntimeSchema.createFrom(Seckill.class);

    public Seckill getSeckill(long seckillId) {
        // redis操作逻辑
        try {
            Jedis jedis = jedisPool.getResource();
            try {
                String key = "seckill:" + seckillId;
                // 并没有实现内部序列化操作
                // get-> byte[] -> 反序列化 ->Object(Seckill)
                // 采用自定义序列化
                // protostuff : pojo.
                byte[] bytes = jedis.get(key.getBytes());//根据Key获取Value
                // 缓存中获取到bytes
                if (bytes != null) {
                    // 空对象
                    Seckill seckill = schema.newMessage();
                    ProtostuffIOUtil.mergeFrom(bytes, seckill, schema);
                    // seckill 被反序列化
                    return seckill;
                }
            } finally {
                jedis.close();
            }
        } catch (Exception e) {
            logger.error(e.getMessage(), e);
        }
        return null;
    }

    public String putSeckill(Seckill seckill) {
        // set Object(Seckill) -> 序列化 -> byte[]
        try {
            Jedis jedis = jedisPool.getResource();
            try {
                String key = "seckill:" + seckill.getSeckillId();//保存Value的Key
                byte[] bytes = ProtostuffIOUtil.toByteArray(seckill, schema,
                        LinkedBuffer.allocate(LinkedBuffer.DEFAULT_BUFFER_SIZE));
                // 超时缓存
                int timeout = 60 * 60;// 1小时
                String result = jedis.setex(key.getBytes(), timeout, bytes);
                return result;
            } finally {
                jedis.close();
            }
        } catch (Exception e) {
            logger.error(e.getMessage(), e);
        }

        return null;
    }

}
```

1. 在使用该RedisDao对象时，需要传入Ip地址和端口。
2. `new RedisDao("localhost","6379");`
3. 若使用Spring ICO容器，需配置：

```xml
<!--redisDao -->
<bean id="redisDao" class="换成你的包dao.cache.RedisDao">
    <constructor-arg index="0" value="localhost" />
    <constructor-arg index="1" value="6379" />
</bean>
```

测试Demo：

```java
Seckill seckill = redisDao.getSeckill(id);
if (seckill == null) {
    seckill = seckillDao.queryById(id);
    if (seckill != null) {
        String result = redisDao.putSeckill(seckill);
        System.out.pritln(result);
        seckill = redisDao.getSeckill(id);
        System.out.pritln(result);
    }
}
```