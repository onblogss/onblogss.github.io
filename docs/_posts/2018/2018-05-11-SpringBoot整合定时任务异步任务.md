---
layout: post
title: SpringBoot整合定时任务异步任务
date: 2018-05-11 11:00:00
author: 薛师兄
tags: SpringBoot
---
# 1.定时任务

1.开启定时任务

```java
@SpringBootApplication
//开启定时任务
@EnableScheduling
public class SpringBootDemoApplication{

    public static void main(String[] args) {
        SpringApplication.run(SpringBootDemoApplication.class, args);
    }
}
```

2.使用定时任务

```java
@Component
public class TestTask {
    private static final SimpleDateFormat dateFormat = new SimpleDateFormat("HH:mm:ss");

    //定义每三秒执行任务
//  @Scheduled(fixedRate=3000)
    @Scheduled(cron="4-10 * * * * ?")
    public void reportCurrentTime() {
        System.out.println(dateFormat.format(new Date()));
    }

}
```

# 2.异步任务

1.开启异步任务

```java
@SpringBootApplication
//开启异步调用方法
@EnableAsync
public class SpringBootDemoApplication{

    public static void main(String[] args) {
        SpringApplication.run(SpringBootDemoApplication.class, args);
    }
}
```

2.定义异步任务

```java
@Component
public class AsyncTask {

    @Async
    public Future<Boolean> dotask(){
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("执行异步任务结束");
        return new AsyncResult<>(true);
    }

}
```

3.使用异步任务

```java
@Autowired
private AsyncTask async;

async.dotask();
```

>就是这么简洁不墨迹~~