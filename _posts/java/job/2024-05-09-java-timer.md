---
layout: post
title:  Java定时任务 Timer定时器
subtitle:
author: YangLe
date:   2024-05-09 01:00:00
categories: 定时任务
tag: 定时任务
---



## 一、概述

定时计划任务功能在Java中主要使用的就是Timer对象，它在内部使用多线程的方式进行处理，所以Timer对象一般又和多线程技术结合紧密。

Timer是Java提供的原生Scheduler(任务调度)工具类，不需要导入其他jar包，使用起来方便高效，非常快捷。



## 二、应用场景

1. 在指定的时间执行任务
2. 指定时间启动任务，执行后间隔指定时间重复执行任务
3. 启动任务之后，延迟多久后执行
4. 启动任务后，延迟多久时间执行,执行之后指定间隔多久重复执行任务

```java
// 在指定的时间执行任务
void schedule(TimerTask task, Date time);
// 指定时间启动任务，执行后间隔指定时间重复执行任务
void schedule(TimerTask task, Date firstTime, long period);
// 指定的延迟之后，执行指定的任务
void schedule(TimerTask task, long delay);
// 延迟多久时间执行,执行之后指定间隔多久重复执行任务
schedule(TimerTask task, long delay, long period);
```

## 三、使用方法

1. 继承 TimerTask 类 并实现其中的 run() 方法来自定义要执行的任务(也可以写成匿名类)
2. 创建一个 Timer 类定时器的对象，并通过 Timer.schedule(参数) 方法执行1中定义的任务

```java
public static void testTimer1() {
    // 创建定时器对象
    Timer timer = new Timer();
    // 配置任务执行 -> 指定延迟，指定重复周期
    timer.schedule(new TimerTask() {
        // 通过匿名类定义要执行的任务方法
	    public void run() {
	    	System.out.println("-------任务执行--------");
	    }
    }, 2000, 3500);
}
```



## 四、注意事项

1、创建一个 Timer 对象相当于新启动了一个线程，但它并不是守护线程。可以通过如下方法设置为守护线程。

```java
private static Timer timer = new Timer(true);
```

2、当计划时间早于当前时间，则任务立即被运行。

3、TimerTask 是以队列的方式一个一个被顺序运行的，所以执行的时间和你预期的时间可能不一致，因为前面的任务可能消耗的时间较长，则后面的任务运行的时间会被延迟。延迟的任务具体开始的时间，就是依据前面任务的结束时间

## 五、优缺点

优点：

- 简单

缺点：

- 只能支持传入触发时间和周期。不支持cron表达式等方式
- 每个任务都是一个线程，无线程池配置
- 修改配置需要重启工程
- 没有可视化界面、没有执行记录等