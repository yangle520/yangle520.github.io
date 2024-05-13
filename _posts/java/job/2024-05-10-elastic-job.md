---
layout: post
title:  定时任务框架 Elastic-job
subtitle:
author: YangLe
date:   2024-05-10 02:00:00
categories: 定时任务
tag: 定时任务
---





## 一、概述

Elastic job是当当网架构师建基于Zookepper、Quartz开发并开源的一个Java分布式定时任务，解决了Quartz不支持分布式的弊端。Elastic job主要的功能有支持弹性扩容，通过Zookepper集中管理和监控job，支持失效转移等。

项目由两个相互独立的子项目 Elastic-Job-Lite 和 Elastic-Job-Cloud 组成。Elastic-Job-Lite 定位为轻量级无中心化解决方案，使用 jar 的形式提供分布式任务的协调服务。ElasticJob Cloud 采用自研 Mesos Framework 的解决方案，额外提供资源治理、应用分发以及进程隔离等功能。它通过弹性调度、资源管控、以及作业治理的功能，打造一个适用于互联网场景的分布式调度解决方案，并通过开放的架构设计，提供多元化的作业生态。

Elastic-Job并不直接提供数据处理的功能，框架只会将分片项分配至各个运行中的作业服务器，开发者需要自行处理分片项与真实数据的对应关系。

仓库地址：https://github.com/apache/shardingsphere-elasticjob

官方文档：https://shardingsphere.apache.org/elasticjob/

### 1、作用：

- 分布式调度协调
- 弹性扩容缩容
- 失效转移
- 错过执行作业重触发
- 作业分片一致性，保证同一分片在分布式环境中仅一个执行实例
- 自诊断并修复分布式不稳定造成的问题
- 支持并行调度
- 支持作业生命周期操作
- 丰富的作业类型
- Spring整合以及命名空间提供
- 运维平台



### 2、版本

#### 2.1 Elastic-Job-Lite

Elastic-Job-Lite 是面向进程内的线程级调度框架。通过 ElasticJob ，作业能够透明化的与业务应用系统相结合。它能够方便的与 Spring 、Dubbo 等 Java 框架配合使用，在作业中可自由使用 Spring 注入的 Bean，如数据源连接池、Dubbo 远程服务等，更加方便的贴合业务开发。

Elastic Job Lite 与业务应用部署在一起，其生命周期与业务应用保持一致，是典型的**嵌入式轻量级架构**。Elastic Job Lite 非常适合于资源使用稳定、部署架构简单的普通 Java 应用，可以理解为 Java 开发框架。

ElasticJob Lite 本身是无中心化架构，无需独立的中心化调度节点，分布式下的每个任务节点均是以自调度的方式适时的调度作业。任务之间需要一个注册中心来对分布式场景下的任务状态进行协调即可，目前支持 ZooKeeper 和 ETCD 作为注册中心。

架构图如下：

![图片]({{ '/images/job/Elastic-Job-Lite架构.png' | prepend: site.baseurl }})

ElasticJob Lite 的分布式作业节点通过选举获取主节点，并通过主节点进行分片。分片完毕后，主节点与从节点并无二致，均以自我调度的方式执行任务。



#### 2.2 Elastic-Job-Cloud

ElasticJob Cloud 拥有进程内调度和进程级别调度两种方式。由于 ElasticJob Cloud 能够对作业服务器的资源进行控制，因此其作业类型可划分为常驻任务和瞬时任务。常驻任务类似于 ElasticJob Lite，是进程内调度；瞬时任务则完全不同，它充分的利用了资源分配的削峰填谷能力，是进程级的调度，每次任务的会启动全新的进程处理。

ElasticJob Cloud 需要通过 Mesos 对资源进行控制，并且通过部署在 Mesos Master 的调度器进行任务和资源的分配。Cloud 采用中心化架构，将调度中心的高可用交由 Mesos 管理。

架构图如下：

![图片]({{ '/images/job/elastic_job_cloud架构图.png' | prepend: site.baseurl }})

ElasticJob Cloud 除了拥有 Lite 的全部能力之外，还拥有资源分配和任务分发的能力。它将作业的开发、打包、分发、调度、治理、分片等一些列的生命周期完全托管，是真正的作业云调度系统。

相比于 ElasticJob Lite 的简单易用，ElasticJob Cloud 对 Mesos 的强依赖增加了系统部署的复杂度，因此更加适合大规模的作业系统。



## 二、核心功能

ElasticJob 功能主要有弹性调度、资源分配、作业治理和可视化管控。

### 1、弹性调度

#### 1.1 自动分片

弹性调度是 ElasticJob 最重要的功能，也是这款产品名称的由来。它是一款能够让任务通过分片进行水平扩展的任务处理系统。

ElasticJob 中任务分片项的概念，使得任务可以在分布式的环境下运行，每台任务服务器只运行分配给该服务器的分片。随着服务器的增加或宕机，ElasticJob 会近乎实时的感知服务器数量的变更，从而重新为分布式的任务服务器分配更加合理的任务分片项，使得任务可以随着资源的增加而提升效率。

如果作业分为 4 片，用两台服务器执行，则每个服务器分到 2 片，如下图所示：

![图片]({{ '/images/job/elastic-job-sharding.png' | prepend: site.baseurl }})

#### 1.2 水平拓展

当增加新作业服务器时，ElasticJob 会通过注册中心的临时节点的变化感知到新服务器的存在，并在下次任务调度的时候重新分片，新的服务器会承载一部分作业分片，分片如下图所示：

![图片]({{ '/images/job/elastic-job-sacle-out.png' | prepend: site.baseurl }})

#### 1.3 高可用

当作业服务器在运行中宕机时，注册中心同样会通过临时节点感知，并将在下次运行时将分片转移至仍存活的服务器，以达到作业高可用的效果。本次由于服务器宕机而未执行完的作业，则可以通过失效转移的方式继续执行。作业高可用如下图所示：

![图片]({{ '/images/job/elastic-job-ha.png' | prepend: site.baseurl }})

### 2、资源分配

调度是指在适合的时间将适合的资源分配给任务，并使其生效。ElasticJob 具备资源分配的能力，它能够像分布式的操作系统一样调度任务。资源分配是借由 Mesos 实现的，由 Mesos 负责分配任务声明的所需资源（CPU 和内存），并将分配出去的资源进行隔离。ElasticJob 在获取到资源之后才会执行任务。

考虑到 Mesos 系统部署相对复杂，因此 ElasticJob 将这部分拆分至 ElasticJob cloud 部分，供高级用户使用。随着 Kubernetes 的强劲发展，ElasticJob 未来也会完成 cloud 部分与它的对接。

### 3、作业治理

作业在分布式场景下的高可用、失效转移、错过作业重新执行等行为的治理和协调。

### 4、可视化管控

主要包括作业的增删改查管控端、执行历史记录查询、配置中心的管理等。



## 三、应用场景

ElasticJob 着重解决与复杂任务、资源导向任务和业务应用任务这几个方面的问题。

### 1、复杂任务

数据迁移。如果将百亿的数据从一组数据库集群迁移至另一组数据库集群，单线程的作业可能需要几天到几周不等。通过 ElasticJob 的弹性分片能力，可以大幅减少海量数据迁移所需要的时间。

### 2、资源导向任务

占用大量计算资源的报表作业。如果每天凌晨需要花费数小时计算 T+1 的业务报表，没有资源的管控，则无论报表作业是否启动，都要为其分配足够的资源。ElasticJob 将作业分为常驻作业和瞬时作业，对于报表类作业，瞬时作业是非常适合的。它能否在作业启动时获取资源，在作业结束后归还资源，做到真正的削峰填谷，更加合理的利用资源。

### 3、业务应用

订单拉取作业。订单系统大多采用消息中间件或作业的方式实现订单拉取，用于将订单生成系统和后端履约系统解耦，以便于前后端流量分离。采用作业实现的订单系统，可以通过 ElasticJob 实现订单相关业务逻辑，可以方便的利用外围系统所提供的依赖注入服务，无缝的融入业务端研发。

## 四、使用方法

### 1、pom 依赖

```xml
<dependency>
    <groupId>com.dangdang</groupId>
    <artifactId>elastic-job-lite-spring</artifactId>
    <version>2.1.5</version>
</dependency>
```

### 2、编写任务

```java
@Slf4j
public class MyElasticJob implements SimpleJob {
    @Override
    public void execute(ShardingContext context) {
        log.info("当前任务分片" + context.getShardingItem());
    }
}
```

### 3、任务配置

基于yml配置

```yml
elasticjob:
  reg-center:
    max-sleep-time-milliseconds: 30000
    namespace: xxxx
    server-lists: 10.188.179.11:2181
  jobs:
    MyElasticJob:
      elasticJobClass: com.demo.job.MyElasticJob
      overwrite: true
      cron: 0 0 0 * * ?
      shardingTotalCount: 1
    dataflowJob:
      elasticJobClass: org.apache.shardingsphere.elasticjob.dataflow.job.DataflowJob
      cron: 0/5 * * * * ?
      shardingTotalCount: 3
      shardingItemParameters: 0=Beijing,1=Shanghai,2=Guangzhou
    scriptJob:
      elasticJobType: SCRIPT
      cron: 0/10 * * * * ?
      shardingTotalCount: 3
      props:
        script.command.line: "echo SCRIPT Job: "
```

也可以基于xml配置

```xml
<!--配置作业注册中心 -->
    <reg:zookeeper id="regCenter" server-lists="10.188.179.11:2181" namespace="xxxx" base-sleep-time-milliseconds="1000" max-sleep-time-milliseconds="3000" max-retries="3"/>

    <!-- 配置作业-->
    <job:simple id="myElasticJob" class="com.demo.job.MyElasticJob" registry-center-ref="regCenter" cron="0/10 * * * * ?" sharding-total-count="3" sharding-item-parameters="0=A,1=B,2=C"/>
```



## 五、作业类型

### 1、SimpleJob

与 Quartz 原生接口相似，但是提供了弹性扩容和分片等功能。示例：

```java
public class MyElasticJob implements SimpleJob {
    
    @Override
    public void execute(ShardingContext context) {
        switch (context.getShardingItem()) {
            case 0: 
                // do something by sharding item 0
                break;
            case 1: 
                // do something by sharding item 1
                break;
            case 2: 
                // do something by sharding item 2
                break;
            // case n: ...
        }
    }
}
```



### 2、DataflowJob

Dataflow类型用于处理数据流，该接口有2个方法需要实现，分别用于抓取(fetchData)和处理(processData)数据。

可通过 DataflowJobConfiguration 配置是否流式处理。

流式处理数据只有fetchData方法的返回值为null或集合长度为空时，作业才停止抓取，否则作业将一直运行下去； 非流式处理数据则只会在每次作业执行过程中执行一次fetchData方法和processData方法，随即完成本次作业。如果采用流式作业处理方式，建议processData 处理数据后更新其状态，避免 fetchData 再次抓取到，从而使得作业永不停止。 流式数据处理参照 TbSchedule 设计，适用于不间歇的数据处理。

```java
public class MyElasticJob implements DataflowJob<Foo> {
    
    @Override
    public List<Foo> fetchData(ShardingContext context) {
        switch (context.getShardingItem()) {
            case 0: 
                List<Foo> data = // get data from database by sharding item 0
                return data;
            case 1: 
                List<Foo> data = // get data from database by sharding item 1
                return data;
            case 2: 
                List<Foo> data = // get data from database by sharding item 2
                return data;
            // case n: ...
        }
    }
    
    @Override
    public void processData(ShardingContext shardingContext, List<Foo> data) {
        // process data
        // ...
    }
}
```



### 3、Script

Script类型作业意为脚本类型作业，支持shell，python，perl等所有类型脚本。只需通过控制台或代码配置scriptCommandLine即可，无需编码。执行脚本路径可包含参数，参数传递完毕后，作业框架会自动追加最后一个参数为作业运行时信息。

示例：

```shell
#!/bin/bash
echo sharding execution context is $*
```

输出：

```shell
sharding execution context is {"jobName":"scriptElasticDemoJob","shardingTotalCount":10,"jobParameter":"","shardingItem":0,"shardingParameter":"A"}
```

以上方法中的参数`shardingContext`包含作业配置、片和运行时信息。可通过`getShardingTotalCount()`, `getShardingItem()`等方法分别获取分片总数，运行在本作业服务器的分片序列号等。



## 六、配置介绍

Elastic-Job配置分为3个层级，分别是Core, Type和Root。每个层级使用相似于装饰者模式的方式装配。

- Core：对应JobCoreConfiguration，用于提供作业核心配置信息，如：作业名称、分片总数、CRON表达式等。
- Type：对应JobTypeConfiguration，有3个子类分别对应SIMPLE, DATAFLOW和SCRIPT类型作业，提供3种作业需要的不同配置，如：DATAFLOW类型是否流式处理或SCRIPT类型的命令行等。
- Root：对应JobRootConfiguration，有2个子类分别对应Lite和Cloud部署类型，提供不同部署类型所需的配置，如：Lite类型的是否需要覆盖本地配置或Cloud占用CPU或Memory数量等。

详细配置说明参考官方文档：https://shardingsphere.apache.org/elasticjob/legacy/lite-2.x/02-guide/config-manual/
