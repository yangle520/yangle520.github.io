---

layout: post
title:  Sentinal接入配置
date:   2024-02-27 00:00:00 +0800
categories: 微服务
tag: 微服务组件
author: YangLe
---



## 快速开始

Sentinel 的使用可以分为两个部分:

1. 核心库（Java 客户端）：不依赖任何框架/库，能够运行于 Java 8 及以上的版本的运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持
2. 控制台（Dashboard）：Dashboard 主要负责管理推送规则、监控、管理机器信息等

### 1、引入依赖

在 pom.xml 中加入依赖

```xml
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
	<version>2.1.1.RELEASE</version>
</dependency>
```

### 2、定义资源

通过Sentinel API将代码块定义成资源

使用 `SphU.entry("HelloWorld")` 和 `entry.exit()` 包围起来即可

```java
public static void main(String[] args) {
    // 配置规则.
    initFlowRules();

    while (true) {
        // 1.5.0 版本开始可以直接利用 try-with-resources 特性
        try (Entry entry = SphU.entry("HelloWorld")) {
            // 被保护的逻辑
            System.out.println("hello world");
		} catch (BlockException ex) {
            // 处理被流控的逻辑
	    	System.out.println("blocked!");
		}
    }
}
```

通过注解将一个方法定义成资源，并配置 `blockHandler` 和 `fallback` 函数来进行限流之后的处理。

注意 `blockHandler` 函数会在原方法被限流/降级/系统保护的时候调用，而 `fallback` 函数会针对所有类型的异常。

```java
@SentinelResource(value = "HelloWorld", blockHandler = "blockHandlerForXXX")
public String helloWorld(String id) {
	// 资源中的逻辑*    
	return "hello world"; 
}

// blockHandler 函数，原方法调用被限流/降级/系统保护的时候调用
public String blockHandlerForXXX(String id, BlockException ex) {
    return "blockHandlerForXXX";
}
```



### 3、定义规则

限制资源的访问规则

例如下面的代码定义了资源 `HelloWorld` 每秒最多只能通过 20 个请求

```java
private static void initFlowRules(){
    List<FlowRule> rules = new ArrayList<>();
    FlowRule rule = new FlowRule();
    rule.setResource("HelloWorld");
    rule.setGrade(RuleConstant.FLOW_GRADE_QPS);
    // Set limit QPS to 20.
    rule.setCount(20);
    rules.add(rule);
    FlowRuleManager.loadRules(rules);
}
```

完成这3步，sentinel就能够正常工作了

### 4、检查效果

运行后可以在日志里面看到输出

```
|--timestamp-|------date time----|--resource-|p |block|s |e|rt
1709003250000|2024-02-27 11:07:30|yangle|5|0|5|0|0|0|0|0
```

其中 `p` 代表通过的请求, `block` 代表被阻止的请求, `s` 代表成功执行完成的请求个数, `e` 代表用户自定义的异常, `rt` 代表平均响应时长。

### 5、启动控制台

Sentinel 开源控制台支持实时监控和规则管理

**启动命令**

java -Dserver.port=9000 -Dcsp.sentinel.dashboard.server=localhost:9000 -Dproject.name=sentinel-dashboard -jar sentinel-dashboard-1.8.7.jar

访问控制台：http://localhost:9000
用户名：sentinel
密码：sentinel



### 6、客户端接入控制台

客户端需要引入 Transport 模块来与 Sentinel 控制台进行通信。您可以通过 `pom.xml` 引入 JAR 包

```xml
<dependency>    
	<groupId>com.alibaba.csp</groupId>    
    <artifactId>sentinel-transport-simple-http</artifactId>
    <version>1.8.6</version> 
</dependency>
```

配置文件中写入

```yml
spring:
  cloud:
    sentinel:
      transport:
        port: 8719                  # 应用与Sentinel控制台交互的端口
        dashboard: 127.0.0.1:9000   # Sentinel 控制台地址
        heartbeat-interval-ms: 1000 # 应用与Sentinel控制台的心跳间隔时间
```



## 详细说明

### 定义资源

#### 1、@SentinelResource

注意：注解方式埋点不支持 private 方法

`@SentinelResource` 用于定义资源，并提供可选的异常处理和 fallback 配置项。 `@SentinelResource` 注解包含以下属性：

| 属性                             | 说明                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| value                            | 资源名称，必需项                                             |
| entryType                        | entry 类型，可选项（默认为 `EntryType.OUT`）                 |
| blockHandler / blockHandlerClass | `blockHandler` 对应处理 `BlockException` 的函数名称，可选项。blockHandler 函数访问范围需要是 `public`，返回类型需要与原方法相匹配，参数类型需要和原方法相匹配并且最后加一个额外的参数，类型为 `BlockException`。blockHandler 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 `blockHandlerClass` 为对应的类的 `Class` 对象，注意对应的函数必需为 static 函数，否则无法解析。 |
| fallback                         | fallback 函数名称，可选项，用于在抛出异常的时候提供 fallback 处理逻辑。fallback 函数可以针对所有类型的异常（除了 `exceptionsToIgnore` 里面排除掉的异常类型）进行处理。fallback 函数签名和位置要求：<br />1、返回值类型必须与原函数返回值类型一致；<br />2、 方法参数列表需要和原函数一致，或者可以额外多一个 `Throwable` 类型的参数用于接收对应的异常。<br />3、 fallback 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 `fallbackClass` 为对应的类的 `Class` 对象，注意对应的函数必需为 static 函数，否则无法解析 |
| defaultFallback                  | 默认的 fallback 函数名称，可选项，通常用于通用的 fallback 逻辑（即可以用于很多服务或方法）。默认 fallback 函数可以针对所以类型的异常（除了 `exceptionsToIgnore` 里面排除掉的异常类型）进行处理。若同时配置了 fallback 和 defaultFallback，则只有 fallback 会生效。defaultFallback 函数签名要求：<br />1、返回值类型必须与原函数返回值类型一致；<br />2、方法参数列表需要为空，或者可以额外多一个 `Throwable` 类型的参数用于接收对应的异常<br />3、defaultFallback 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 `fallbackClass` 为对应的类的 `Class` 对象，注意对应的函数必需为 static 函数，否则无法解析 |
| exceptionsToIgnore               | 用于指定哪些异常被排除掉，不会计入异常统计中，也不会进入 fallback 逻辑中，而是会原样抛出。 |

若 blockHandler 和 fallback 都进行了配置，则被限流降级而抛出 `BlockException` 时只会进入 `blockHandler` 处理逻辑。若未配置 `blockHandler`、`fallback` 和 `defaultFallback`，则被限流降级时会将 `BlockException` **直接抛出**

代码示例：

```java
@Operation(summary = "Sentinel demo")
@GetMapping("/demo")
@SentinelResource(value = "sentinelDemo", blockHandler = "blockHandlerForDemo", fallback = "fallbackForDemo")
public Result<String> demo(String id) {
   return Result.success(id + ":success");
}
```



### 定义规则

Sentinel 的所有规则都可以在内存态中动态地查询及修改，修改之后立即生效。同时 Sentinel 也提供相关 API，供您来定制自己的规则策略

Sentinel 支持以下几种规则：**流量控制规则**、**熔断降级规则**、**系统保护规则**、**来源访问控制规则** 和 **热点参数规则**

#### 1、流量控制规则 (FlowRule)

`FlowSlot` 会根据预设的规则，结合前面 `NodeSelectorSlot`、`ClusterNodeBuilderSlot`、`StatistcSlot` 统计出来的实时信息进行流量控制。同一个资源可以同时有多个限流规则。

| 属性            | 说明                                                         | 默认值                      |
| --------------- | ------------------------------------------------------------ | --------------------------- |
| resource        | 资源名，资源名是限流规则的作用对象                           |                             |
| count           | 限流阈值                                                     |                             |
| grade           | 限流阈值类型，QPS 或线程数模式                               | QPS                         |
| limitApp        | 流控针对的调用来源                                           | default，代表不区分调用来源 |
| strategy        | 调用关系限流策略：直接、链路、关联                           | 直接                        |
| controlBehavior | 流控效果（直接拒绝 / 匀速器 /冷启动），不支持按调用关系限流<br />1、直接拒绝：CONTROL_BEHAVIOR_DEFAULT，当QPS超过任意规则的阈值后，新的请求就会被立即拒绝，拒绝方式为抛出`FlowException`<br />2、冷启动：CONTROL_BEHAVIOR_RATE_LIMITER，让通过的流量缓慢增加，在一定时间内逐渐增加到阈值上限，给冷系统一个预热的时间，避免冷系统被压垮的情况<br />3、匀速器：CONTROL_BEHAVIOR_WARM_UP，严格控制了请求通过的间隔时间，也即是让请求以均匀的速度通过，对应的是漏桶算法。例如当QPS=2时，每隔500ms才允许通过下一个请求，削峰填谷。 | 直接拒绝                    |

代码示例：

```java
private static void initFlowQpsRule() {
   List<FlowRule> rules = new ArrayList<>();
   FlowRule rule1 = new FlowRule();
   rule1.setResource("sentinelDemo");          // 资源名
   rule1.setCount(1);                          // 限流阈值
   rule1.setGrade(RuleConstant.FLOW_GRADE_QPS);// 限流阈值类型，QPS
   rule1.setLimitApp("default");               // 调用来源
   rules.add(rule1);
   FlowRuleManager.loadRules(rules);
}
```



#### 2、熔断降级规则 (DegradeRule)

| 属性               | 说明                                                         | 默认值     |
| ------------------ | ------------------------------------------------------------ | ---------- |
| resource           | 资源名，即规则的作用对象                                     |            |
| grade              | 熔断策略，支持慢调用比例/异常比例/异常数策略<br />1、慢调用比例：SLOW_REQUEST_RATIO，选择以慢调用比例作为阈值，需要设置允许的慢调用 RT（即最大的响应时间），请求的响应时间大于该值则统计为慢调用。当单位统计时长（`statIntervalMs`）内请求数目大于设置的最小请求数目，并且慢调用的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求响应时间小于设置的慢调用 RT 则结束熔断，若大于设置的慢调用 RT 则会再次被熔断<br />2、异常比例：ERROR_RATIO，当单位统计时长（`statIntervalMs`）内请求数目大于设置的最小请求数目，并且异常的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断。异常比率的阈值范围是 `[0.0, 1.0]`，代表 0% - 100%<br />3、异常数：ERROR_COUNT，当单位统计时长内的异常数目超过阈值之后会自动进行熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断 | 慢调用比例 |
| count              | 慢调用比例模式下：慢调用临界 RT（超出该值计为慢调用）<br />异常比例和异常数模式模式下： 对应的阈值 |            |
| timeWindow         | 熔断时长，单位为 s                                           |            |
| minRequestAmount   | 熔断触发的最小请求数，请求数小于该值时即使异常比率超出阈值也不会熔断 | 5          |
| statIntervalMs     | 统计时长（单位为 ms）                                        | 1000 ms    |
| slowRatioThreshold | 慢调用比例阈值，仅慢调用比例模式有效                         |            |

代码示例

```java
private static void initDegradeRule() {
   List<DegradeRule> rules = new ArrayList<>();
   DegradeRule rule = new DegradeRule("sentinelDemo");
   rule.setGrade(CircuitBreakerStrategy.ERROR_RATIO.getType())
         .setCount(0.7)// Threshold is 70% error ratio
         .setMinRequestAmount(100)
         .setStatIntervalMs(30000) // 30s
         .setTimeWindow(10);
   rules.add(rule);
   DegradeRuleManager.loadRules(rules);
}
```



#### 3、系统保护规则 (SystemRule)

Sentinel 系统自适应限流从整体维度对应用入口流量进行控制，结合应用的 Load、CPU 使用率、总体平均 RT、入口 QPS 和并发线程数等几个维度的监控指标，通过自适应的流控策略，让系统的入口流量和系统的负载达到一个平衡，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性

| 属性              | 说明                                   | 默认值      |
| ----------------- | -------------------------------------- | ----------- |
| highestSystemLoad | `load1` 触发值，用于触发自适应控制阶段 | -1 (不生效) |
| avgRt             | 所有入口流量的平均响应时间             | -1 (不生效) |
| maxThread         | 入口流量的最大并发数                   | -1 (不生效) |
| qps               | 所有入口资源的 QPS                     | -1 (不生效) |
| highestCpuUsage   | 当前系统的 CPU 使用率（0.0-1.0）       | -1 (不生效) |

代码示例

```java
private void initSystemProtectionRule() {
    List<SystemRule> rules = new ArrayList<>();
    SystemRule rule = new SystemRule();
    rule.setHighestSystemLoad(10);
    rule.setHighestCpuUsage(0.6);
    rules.add(rule);
    SystemRuleManager.loadRules(rules);
}
```

#### 4、访问控制规则 (AuthorityRule)

黑白名单根据资源的请求来源（`origin`）限制资源是否通过，若配置白名单则只有请求来源位于白名单内时才可通过；若配置黑名单则请求来源位于黑名单时不通过，其余的请求通过

调用方信息通过 `ContextUtil.enter(resourceName, origin)` 方法中的 `origin` 参数传入。

| 属性     | 说明                                                         | 默认值 |
| -------- | ------------------------------------------------------------ | ------ |
| resource | 资源名，即限流规则的作用对象                                 |        |
| limitApp | 对应的黑/白名单内容，多个用逗号分隔，如 `appA,appB`          |        |
| strategy | 限制模式，`AUTHORITY_WHITE` 为白名单模式，`AUTHORITY_BLACK` 为黑名单模式 | 白名单 |

代码示例

```java
private static void initWhiteRules() {
   AuthorityRule rule = new AuthorityRule();
   rule.setResource("sentinelDemo");
   rule.setStrategy(RuleConstant.AUTHORITY_WHITE);
   rule.setLimitApp("appA,appB");
   AuthorityRuleManager.loadRules(Collections.singletonList(rule));
}
```



#### 5、热点规则 (ParamFlowRule)

很多时候我们希望统计某个热点数据中访问频次最高的 Top K 数据，并对其访问进行限制。比如：

- 商品 ID 为参数，统计一段时间内最常购买的商品 ID 并进行限制
- 用户 ID 为参数，针对一段时间内频繁访问的用户 ID 进行限制

热点参数限流会统计传入参数中的热点参数，并根据配置的限流阈值与模式，对包含热点参数的资源调用进行限流。热点参数限流可以看做是一种特殊的流量控制，仅对包含热点参数的资源调用生效。

Sentinel 利用 LRU 策略统计最近最常访问的热点参数，结合令牌桶算法来进行参数级别的流控。

使用热点参数限流，需要引入下面依赖：

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-parameter-flow-control</artifactId>
    <version>1.8.7</version> 
</dependency>
```

| 属性              | 说明                                                         | 默认值   |
| ----------------- | ------------------------------------------------------------ | -------- |
| resource          | 资源名                                                       |          |
| count             | 限流阈值                                                     |          |
| grade             | 限流模式                                                     | QPS      |
| durationInSec     | 统计窗口时间长度（单位为秒）                                 | 1s       |
| controlBehavior   | 流控效果（支持快速失败和匀速排队模式）                       | 快速失败 |
| maxQueueingTimeMs | 最大排队等待时长（仅在匀速排队模式生效）                     | 0        |
| paramIdx          | 热点参数的索引，必填                                         |          |
| paramFlowItemList | 参数例外项，可以针对指定的参数值单独设置限流阈值，不受前面 `count` 阈值的限制。**仅支持基本类型和字符串类型** |          |
| clusterMode       | 是否是集群参数流控规则                                       | false    |
| clusterConfig     | 集群流控相关配置                                             |          |

代码示例

```java
private static void initParamFlowRules() {
   // 使用QPS模式,针对第一个参数，限流为5
   ParamFlowRule rule = new ParamFlowRule("sentinelDemo")
         .setParamIdx(0)
         .setGrade(RuleConstant.FLOW_GRADE_QPS)
         .setCount(5);

   // 如果输入的参数值是 PARAM_B ，单独设置限流 QPS 阈值为 10，而不是全局的阈值 5.
   ParamFlowItem item = new ParamFlowItem().setObject(PARAM_B)
         .setClassType(int.class.getName())
         .setCount(10);
   rule.setParamFlowItemList(Collections.singletonList(item));

   ParamFlowRuleManager.loadRules(Collections.singletonList(rule));
}
```



### 配置规则

#### 1、Nacos配置

添加以下依赖

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
    <version>1.8.7</version>
</dependency>
```

yaml配置

```yml
spring:
  cloud:
    sentinel:
      datasource:
        flow-rule:   # 这个名称可以自定义
          nacos:
            # nacos 地址
            server-addr: http://10.188.179.204:8848
            # 命名空间
            namespace: 790a96ba-a7f7-4ae6-812d-a1413289aa3a
            # 不同规则可以设置不同group id
            group-id: SENTINEL_FLOW_GROUP
            # 仅支持json和xml类型
            data-id: ${spring.application.name}-flow-rule.json
            # 规则类型：flow,degrade,param-flow,system,authority
            rule-type: flow
            # nacos如果开启了认证需要配置账户密码
            # username: nacos
            # password: nacos
```

Nacos中增加sentinel配置内容

```json
[
    {
        "clusterMode": false,        // 是否是集群模式
        "controlBehavior": 0,        // 流量控制效果 0：快速失败，1：warm up,2:排队等待
        "count": 5,                  // 阈值
        "grade": 1,                  // 1：QPS，0：并发线程数
        "limitApp": "default",       // 针对来源
        "resource": "sentinelNacos",  // 资源名
        "strategy": 0                // 0：直接，1：关联，2：链路
    }
]
```

java代码调用

```java
@Operation(summary = "Sentinel nacos")
@GetMapping("/nacos")
@SentinelResource(value = "sentinelNacos", blockHandler = "blockHandlerForDemo", fallback = "fallbackForDemo")
public Result<String> nacos(String id) {
   return Result.success(id + ":success");
}
```

#### 2、其他配置

推模式：ZooKeeper, Redis, Nacos, Apollo, etcd

拉模式：动态文件数据源、Consul, Eureka



### transport 模块

引入了 transport 模块后，可以通过以下的 HTTP API 来获取所有已加载的规则

| 方法                   | 链接                                | 取值范围                |
| ---------------------- | ----------------------------------- | ----------------------- |
| 查询所有已经加载的规则 | http://ip:port/getRules?type=<XXXX> | flow / degrade / system |
| 获取所有热点规则       | http://ip:port/getParamRules        |                         |
| 查看调用链             | http://ip:port/tree                 |                         |








## Sentinel配置选项

Spring Cloud Alibaba Sentinel 提供了这些配置选项:

| 配置项                                                  | 含义                                                         | 默认值            |
| ------------------------------------------------------- | ------------------------------------------------------------ | ----------------- |
| `spring.application.name` or `project.name`             | Sentinel项目名                                               |                   |
| `spring.cloud.sentinel.enabled`                         | Sentinel自动化配置是否生效                                   | true              |
| `spring.cloud.sentinel.eager`                           | 是否提前触发 Sentinel 初始化                                 | false             |
| `spring.cloud.sentinel.transport.port`                  | 应用与Sentinel控制台交互的端口，应用本地会起一个该端口占用的HttpServer | 8719              |
| `spring.cloud.sentinel.transport.dashboard`             | Sentinel 控制台地址                                          |                   |
| `spring.cloud.sentinel.transport.heartbeat-interval-ms` | 应用与Sentinel控制台的心跳间隔时间                           |                   |
| `spring.cloud.sentinel.transport.client-ip`             | 此配置的客户端IP将被注册到 Sentinel Server 端                |                   |
| `spring.cloud.sentinel.filter.order`                    | Servlet Filter的加载顺序。Starter内部会构造这个filter        | Integer.MIN_VALUE |
| `spring.cloud.sentinel.filter.url-patterns`             | 数据类型是数组。表示Servlet Filter的url pattern集合          | /*                |
| `spring.cloud.sentinel.filter.enabled`                  | Enable to instance CommonFilter                              | true              |
| `spring.cloud.sentinel.metric.charset`                  | metric文件字符集                                             | UTF-8             |
| `spring.cloud.sentinel.metric.file-single-size`         | Sentinel metric 单个文件的大小                               |                   |
| `spring.cloud.sentinel.metric.file-total-count`         | Sentinel metric 总文件数量                                   |                   |
| `spring.cloud.sentinel.log.dir`                         | Sentinel 日志文件所在的目录                                  |                   |
| `spring.cloud.sentinel.log.switch-pid`                  | Sentinel 日志文件名是否需要带上 pid                          | false             |
| `spring.cloud.sentinel.servlet.block-page`              | 自定义的跳转 URL，当请求被限流时会自动跳转至设定好的 URL     |                   |
| `spring.cloud.sentinel.flow.cold-factor`                | WarmUp 模式中的 冷启动因子                                   | 3                 |
| `spring.cloud.sentinel.zuul.order.pre`                  | SentinelZuulPreFilter 的 order                               | 10000             |
| `spring.cloud.sentinel.zuul.order.post`                 | SentinelZuulPostFilter 的 order                              | 1000              |
| `spring.cloud.sentinel.zuul.order.error`                | SentinelZuulErrorFilter 的 order                             | -1                |
| `spring.cloud.sentinel.scg.fallback.mode`               | Spring Cloud Gateway 流控处理逻辑 (选择 `redirect` or `response`) |                   |
| `spring.cloud.sentinel.scg.fallback.redirect`           | Spring Cloud Gateway 响应模式为 'redirect' 模式对应的重定向 URL |                   |
| `spring.cloud.sentinel.scg.fallback.response-body`      | Spring Cloud Gateway 响应模式为 'response' 模式对应的响应内容 |                   |
| `spring.cloud.sentinel.scg.fallback.response-status`    | Spring Cloud Gateway 响应模式为 'response' 模式对应的响应码  | 429               |
| `spring.cloud.sentinel.scg.fallback.content-type`       | Spring Cloud Gateway 响应模式为 'response' 模式对应的 content-type | application/json  |

