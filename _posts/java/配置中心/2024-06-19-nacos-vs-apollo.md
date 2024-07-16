---
layout: post
title: 配置中心设计 - nacos VS apollo
date:   2024-06-19 00:00:00
categories: 配置中心 nacos
tag: 配置中心
author: YangLe
---



nacos 和 Apollo 一样，也是一款配置中心，同样可以实现配置的集中管理、分环境管理、即时生效 等等。并且 nacos 还具备了服务发现的功能。

本文主要分析 nacos 和 Apollo 在设计上的差异。基于 Apollo 1.8.0 和 nacos 2.1.0



## 一、安全性的差异

### 1 方案差异

针对安全问题，一般有两个解决方案：

1. **不同环境使用不同的配置中心**。Apollo 用的就是这一种，当客户端需要获取生产配置时，运维需要在项目的启动参数中指定生产环境配的配置中心。这种方案要想可靠，生产环境的 config server 地址绝对不能泄漏。
2. **不同环境使用同一个配置中心，做好环境隔离**。Nacos 采用这一种，隔离方案是 命名空间 + 鉴权 。和 Apollo 不同，客户端去读 Nacos 是需要账号密码的，当客户端需要获取生产配置时，运维需要在项目的启动参数中指定生产环境的 namespace 以及对应的账号密码。



### 2 Namespace

Apollo 和 Nacos 都有这个概念，不过，在 Apollo 里，namespace 可以看成是一个具体的配置文件，而 Nacos 里，namespace 表示具体的环境。

- Apollo 是通过连接不同的 config server 来区分环境
- Nacos 通过指定 namespace 来区分

![图片]({{ '/images/java/config/nacos_vs_apollo_namespace.png' | prepend: site.baseurl }})



综上，要确保安全，使用 Apollo 时不能泄漏 config server 生产环境的地址。使用 Nacos 时不能泄漏对应生产环境 namespace 的账号密码。



## 二、系统复杂度的差异

### 1 Apollo 架构

![图片]({{ '/images/java/config/Apollo架构图.png' | prepend: site.baseurl }})

Apollo 虽然组件过多（config service、admin service、portal、SLB、meta server、eureka等），但是 Apollo 把 config service 、eureka 和 meta server 打包在一起部署。



### 2 Nacos 架构

![图片]({{ '/images/java/config/Nacos部署架构图.png' | prepend: site.baseurl }})



| 对比项   | Apollo                               | Nacos                                  |
| -------- | ------------------------------------ | -------------------------------------- |
| 功能     | ✅ 功能全面，能满足大部分的需求       | ❌ 功能比较简洁，满足的场景有限         |
| 维护     | ❌维护费用高，需要管理的服务比较多    | ✅ 相比Apollo维护费用低                 |
| 开发     | ✅ 支持其他语言开发，但是第三方客户端 | ✅ 支持其他语言客户端，不是第三方客户端 |
| 版本管理 | ✅ 支持版本管理、回滚、灰度发布       | ✅ 支持版本管理、回滚、灰度发布         |
| 性能     | ❌ 读写性能比Nacos低，各节点都有缓存  | ✅ 读写性能比 Apollo 好                 |



- Apollo 相对于 Nacos 在配置管理做的更加全面，管理流程上做的更好。使用起来也要麻烦一些
- Nacos 使用起来相对比较简洁，在对性能要求比较高的大规模场景更适合



