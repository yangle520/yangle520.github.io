---
layout: post
title:  Java定时任务 ScheduledExecutorService
subtitle:
author: YangLe
date:   2024-05-09 02:00:00
categories: 定时任务
tag: 定时任务
---





## 一、概述

`ScheduledExecutorService`有线程池的特性，也可以实现任务循环执行，可以看作是一个简单地[定时任务](https://so.csdn.net/so/search?q=定时任务&spm=1001.2101.3001.7020)组件，因为有线程池特性，所以任务之间可以多线程并发执行，互不影响，当任务来的时候，才会真正创建线程去执行。如果你的任务出现异常没有捕获，会导致后续的任务不再执行，所以一定要`try...catch`



## 二、应用场景

### 1、延迟不循环任务

```java
/**
 * 延迟不循环任务
 * @param command 任务 
 * @param delay 方法第一次执行的延迟时间
 * @param unit 延迟单位
 */
schedule(Runnable command, long delay, TimeUnit unit);
```

### 2、延迟且循环任务

到了间隔时间时，先检测上一个任务是否执行完毕，如果执行完成，则立即执行当前任务。如果上一个任务没有完成，则等待它完成后 再执行当前任务。因此不能严格保证任务按照固定间隔执行。

```java
/**
 * 延迟且循环任务
 * @param command 任务
 * @param initialDelay 初始化完成后延迟多长时间执行第一次任务
 * @param period 任务时间间隔
 * @param unit 单位
 */
cheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit);
```

### 3、严格按照一定时间间隔执行

以上一次任务执行结束时间为准，加上任务间隔时长 作为下一次任务的开始时间。

```java
/**
 * 严格按照一定时间间隔执行
 * @param command 任务
 * @param initialDelay 初始化完成后延迟多长时间执行第一次任务
 * @param delay 任务执行时间间隔
 * @param unit 单位
 */
scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit);
```



## 三、使用方法

```
@Component
@Slf4j
public class MineExecutors {

    private final static ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(5);

    private final static SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    @PostConstruct
    public void init() {
        scheduler.scheduleWithFixedDelay(() -> {
            try {
                log.info("开始执行...time {}", format.format(new Date()));
                Thread.sleep(5000);
                log.info("执行结束...time {}", format.format(new Date()));
            } catch (Exception e) {
                log.error("定时任务执行出错");
            }
        }, 0, 3, TimeUnit.SECONDS);
        log.info("初始化成功 {}", format.format(new Date()));
    }

}
```



## 四、优缺点

优点：

- 比 Timer 合理的利用了线程池资源，在一个任务失败时，不影响其他任务执行
- 可以设置周期定时执行，也可以设置触发时间

确定：

- 不支持 cron 这种灵活配置
- 修改需要重新启动项目
- 功能单一，没有可视化、监控、分布式等功能。