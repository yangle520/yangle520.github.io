---
layout: post
title:  Java定时任务 Spring-Task
subtitle:
author: YangLe
date:   2024-05-09 03:00:00
categories: 定时任务
tag: 定时任务
---





## 一、概述

Spring 3.0 之后，提供了一种简单的定时任务 Spring Task。



## 二、使用方法

1、在入口处添加注解 @EnableScheduling

```java
@SpringBootApplication
@EnableScheduling
public class SystemApplication {
    public static void main(String[] args) {
        SpringApplication.run(SystemApplication.class, args);
    }
}
```

2、编写任务实现

```java
@Component
public class HelloJob {
    @Scheduled(cron = "0/3 * * * * ?")
    public void hi(){
        System.err.println("定时任务-"+System.currentTimeMillis());
    }
}
```



## 三、核心介绍

### 1、cron表达式

表达式组成：

| 组成部分 | 字段       | 是否必填 | 允许值              | 可用特殊字符    |
| -------- | ---------- | -------- | ------------------- | --------------- |
| 第一部分 | 秒         | 是       | 0~59                | , - * /         |
| 第二部分 | 分钟       | 是       | 0~59                | , - * /         |
| 第三部分 | 小时       | 是       | 0~23                | , - * /         |
| 第四部分 | 月中的哪天 | 是       | 1~31                | , - * / ? L W C |
| 第五部分 | 月         | 是       | 1~12 或 JAN~DEC     | , - * /         |
| 第六部分 | 周中的哪天 | 是       | 1~7 或 SUN~SAT      | , - * / ? L C # |
| 第七部分 | 年         | 否       | 不填写 或 1970~2099 | , - * /         |

符号含义：

| 符号      | 含义                                                         |
| --------- | ------------------------------------------------------------ |
| ?  (问号) | 表示不确定的值，任意的一天                                   |
| *  (星号) | 表示整个时段                                                 |
| ,  (逗号) | 设置多个值。例如：26,29,33  表示26分，29分，33分各运行一次   |
| -  (减号) | 设置取值范围。例如：5-20  表示从5分到20分钟 每分钟执行一次   |
| /  (斜线) | 设置频率。例如：1/15  表示从1分开始，每隔15分钟运行一次      |
| L         | 用于每月或每周，表示 每月的最后一天 或 每月的最后星期几。例如 6L 表示每月最后一个星期五 |
| W         | 离给定日期最近的工作日。例如：15W 表示离本月15号最近的工作日 |
| #         | 该月第几个周几。例如：6#3  表示该月第3个周五                 |

cron公式在线生成/解析：https://cron.qqe2.com/

### 2、核心注解

#### @EnableScheduling

修饰的是类，主要作用是用于启动定时任务，时刻监控我们所写的任务该不该触发。

#### @Scheduled

修饰的是方法，主要用来标记哪个方法需要定时触发，同时通过内部属性cron实现定时任务的触发规则，其实就是编写cron表达式。



## 四、优缺点

优点：

- @Scheduled注解方法执行时抛出异常，不会导致线程终止，不影响任务的下一次执行
- Spring-Task扩展了对CRON的支持，其对任务耗时超过间隔时间的处理方式与固定延时相同

缺点：

- 集群环境下多个节点都会重复执行任务
- @Scheduled不支持动态修改执行计划，需要重新部署项目

