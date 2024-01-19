---
layout: post
title:  物联网平台
date:   2024-01-17 00:00:00 +0800
categories: 智慧矿山
tag: 学习
---



* content
{:toc}
## 物联网平台 

**物联网平台 (IOT)** 是一种用于构建和管理物联网解决方案的数字平台，是实现万物互联的基础平台，也是帮助人工智能以更好的方式控制和理解事物的技术。通过物联网平台可以远程对接的各种设备，收集设备数据，并在平台端监控、联动、分析和管理所有互联网连接的设备，还能为整个系统或者运营提供决策。



## 开源框架

### 1、EdgeX 

边缘计算框架

官网：https://wiki.edgexfoundry.org/

代码：https://github.com/edgexfoundry



**边缘侧与云端环境对比**

| 特点     | 边缘侧                                     | 云端                       |
| -------- | ------------------------------------------ | -------------------------- |
| 通讯协议 | 几百种设备协议                             | 标准化IT协议               |
| 网络连接 | 有基于IP的，也有非IP的                     | 全部IP化网络               |
| 网络环境 | 分布各地的计算节点、运行环境恶劣、安全性低 | 数据中心内、安全性高       |
| 实时性   | 不管是否连接到云端都是要实时响应           | 实时性低                   |
| 运行环境 | 各种操作系统                               | Windows、Linux等标准服务器 |



**EdgeX架构**

![EdgeXArchitecture-6-1-19]({{ '/styles/images/monitor-design/EdgeXArchitecture-6-1-19.png' | prepend: site.baseurl  }})



### 2、KubeEdge

KubeEdge 是一个开源系统，可将本机容器化的业务流程和设备管理扩展到Edge上的主机。它基于Kubernetes构建，并为网络、应用程序部署以及云与边缘之间的元数据同步提供核心基础架构支持。它还支持MQTT，并允许开发人员编写自定义逻辑并在Edge上启用资源受限的设备通信。KubeEdge由云部分和边缘部分组成，边缘和云部分现已开源

官网：https://kubesphere.io/zh/docs/v3.3/pluggable-components/kubeedge/

代码：https://gitee.com/soxueren/KubeEdge







## 开源产品



### 1、DC3

DC3 是一个基于SpringCloud的开源的、分布式的物联网(IOT)平台，用于快速开发物联网项目和管理物联设备，是一整套物联系统解决方案。支持MQTT、Modbus_TCP、UDP、OPC-DA、OPC-UA、LWM2M、CoAP等协议驱动接入，同时支持快速拓展。

**官网**：https://doc.dc3.site/

**代码**：https://gitee.com/pnoker

**架构设计**

![DC3 架构设计]({{ '/styles/images/monitor-design/architecture.jpg' | prepend: site.baseurl  }})



### 2、FastBee

更适合中小企业和个人使用的物联网平台，后端采用Spring boot，前端采用Vue，移动端支持微信小程序、安卓、苹果和H5采用Uniapp；数据库采用Mysql、TDengine和Redis；设备端支持ESP32、ESP8266、树莓派、合宙等

官网：https://fastbee.cn/

代码：https://gitee.com/kerwincui/wumei-smart

https://github.com/kerwincui/FastBee

系统架构

![FastBee系统架构]({{ '/styles/images/monitor-design/FastBee系统架构.webp' | prepend: site.baseurl  }})



### 3、OPENIITA

包含了品类、物模型、消息转换、通讯组件（mqtt/EMQX通讯组件、小度音箱接入组件、onenet Studio接入组件）、modbus透传接入、modbus虚拟网关、云端低代码设备开发、设备管理、设备分组、规则引擎、第三方平台接入、数据流转（http/mqtt/kafka）、数据可视化、报警中心等模块和智能家居APP（小程序）

**官网**：https://iotkit-open-source.gitee.io/document/

**代码**：https://gitee.com/open-iita

**系统架构**

![OPENIITA架构]({{ '/styles/images/monitor-design/OPENIITA架构.jpg' | prepend: site.baseurl  }})



### 4、IoTOS

官网：http://www.iotos.top/

代码：https://gitee.com/chinaiot



### 5、JetLinks-IOT

官网：https://www.jetlinks.cn/#/

代码：https://gitee.com/jetlinks/jetlinks-community/tree/2.0/



### 6、Apache StreamPipes

官网：https://streampipes.apache.org/

代码：https://github.com/apache/streampipes/discussions



### 7、ThingLinks

官网：https://www.mqttsnet.com/index.html

代码：https://gitee.com/mqttsnet/thinglinks
