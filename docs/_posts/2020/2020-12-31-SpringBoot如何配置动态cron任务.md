---
layout: post
title: SpringBoot如何配置动态cron任务
date: 2020-12-31 13:00:00
author: 薛师兄
tags: SpringBoot
---

## 前情回顾

SpringBoot 通过 `@Scheduled`注解提供了cron、fixedRate、fixedDelay三种定时任务执行规则。

举个例子，比如每5秒执行一次任务，你可以使用下面三种方式：

- @Scheduled(fixedRate = 5000) 
- @Scheduled(fixedDelay = 5000)
- @Scheduled(cron = "0/5 * * * * *") 

下面我们来看一下每种方式的执行区别是什么：

- fixedRate 是按照一定的速率执行，是从上一次方法执行开始的时间算起，如果上一次方法阻塞住了，下一次也是不会执行，但是在阻塞这段时间内累计应该执行的次数，当不再阻塞时，一下子把这些全部执行掉，而后再按照固定速率继续执行。
- fixedDelay 控制方法执行的间隔时间，是以上一次方法执行完开始算起，如上一次方法执行阻塞住了，那么直到上一次执行完，并间隔给定的时间后，执行下一次。
- cron 表达式可以定制化执行任务，但是执行的方式是与 fixedDelay 相近的，也是会按照上一次方法结束时间开始算起。

## 进入正文

说一说我遇到的问题，`@Scheduled` 注解天生无法使用变量，如果你用了，会提升你 Attribute value must be constant。无奈只能使用编程的方式进行定时任务的开发。

SpringBoot 显然已经考虑到了，提供了一个 `SchedulingConfigurer` 接口用来在任务执行时进行动态修改。我已经封装好了一个实现类，使用时只需要注入该 Bean 即可。

代码如下：

```java
import org.springframework.scheduling.Trigger;
import org.springframework.scheduling.TriggerContext;
import org.springframework.scheduling.annotation.SchedulingConfigurer;
import org.springframework.scheduling.config.ScheduledTaskRegistrar;
import org.springframework.scheduling.config.TriggerTask;
import org.springframework.scheduling.support.CronTrigger;

import java.util.Date;

/**
 * 动态Cron任务
 * 注入Spring容器使用
 * @author github.com/onblog
 * @date 2020/12/30
 */
public class CronScheduledBean implements SchedulingConfigurer {
    private static final int HEART_BEAT = 60 * 1000;
    private String cron;
    private TriggerTask triggerTask;
    private ScheduledTaskRegistrar scheduledTaskRegistrar;
    private Date nextDate;

    /**
     * 配置任务逻辑和cron表达式
     * @param cronTask 任务逻辑
     * @param cron 表达式
     */
    public CronScheduledBean(Runnable cronTask, String cron) {
        this.cron = cron;
        this.triggerTask = new TriggerTask(doTask(cronTask), getTrigger());
    }


    @Override
    public void configureTasks(ScheduledTaskRegistrar scheduledTaskRegistrar) {
        this.scheduledTaskRegistrar = scheduledTaskRegistrar;
        scheduledTaskRegistrar.addTriggerTask(triggerTask);
    }

    /**
     * 动态修改cron表达式
     */
    public void updateCron(String cron) {
        this.cron = cron;
        this.scheduledTaskRegistrar.scheduleTriggerTask(triggerTask);
    }

    /**
     * 任务执行方法
     */
    private Runnable doTask(Runnable cronTask) {
        return new Runnable() {
            @Override
            public void run() {
                // 存在乱触发情况，需要判断是否在可执行时间内
                if (System.currentTimeMillis() >= nextDate.getTime()) {
                    cronTask.run();
                }
            }
        };
    }


    /**
     * 任务触发器
     */
    private Trigger getTrigger() {
        return new Trigger() {
            @Override
            public Date nextExecutionTime(TriggerContext triggerContext) {
                // 根据触发器上下文获取下一次执行的时间
                try {
                    nextDate = new CronTrigger(cron).nextExecutionTime(triggerContext);
                } catch (Exception e) {
                    // 出现异常过1分钟再检查,可以自行修改监测时间
                    nextDate = new Date(System.currentTimeMillis() + HEART_BEAT);
                }
                System.out.println("触发器："+nextDate);
                return nextDate;
            }
        };
    }


}
```

使用示例：

```java
@RestController
public class TaskController {

    @Autowired
    private CronScheduledBean cronScheduledBean;

    @Bean
    public CronScheduledBean scheduledUtil() {
        return new CronScheduledBean(new Runnable() {
            @Override
            public void run() {
                System.out.println("任务执行：" + new Date());
            }
        }, "*/5 * * * * ?");
    }

    @GetMapping("/editCron")
    public String editCron(String cron) {
        System.out.println("修改: " + cron + "   " + new Date());
        cronScheduledBean.updateCron(cron);
        return cron;
    }

}
```

注意，分布式应用慎用。
