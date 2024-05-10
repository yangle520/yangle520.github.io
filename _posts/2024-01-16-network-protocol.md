---
layout: post
title:  网络协议
date:   2024-01-16 00:00:00 +0800
categories: 网络
tag: 学习
author: YangLe
---


## 协议介绍

协议定义了在两个或多个通信实体之间进行交换的报文格式和次序，以及报文发送和/或接收一条报文或其他事件所采取的行动



## 视频直播相关

### RTMP

**介绍**：RTMP是Adobe Systems公司为Flash播放器和服务器之间音频、视频和数据传输开发的开放协议。这个协议建立在TCP协议或者轮询HTTP协议之上。RTMP协议就像一个用来装数据包的容器，这些数据既可以是AMF格式的数据，也可以是FLV中的视音频数据。一个单一的连接可以通过不同的通道传输多路网络流，这些通道中的包都是按照固定大小的包传输的

| 属性 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| 全称 | Real-Time Messaging Protocol，实时消息传送协议               |
| 地址 | rtmp://                                                      |
| 传输 | 属于应用层，基于TCP传输                                      |
| 用途 | 可以推流，也可以拉流。<br />浏览器摒弃了Flash播放器，所以一般只用作直播源推流、推流到CDN等场景 |
| 原理 | 建立在TCP长连接通道上将数据切片发送，封装音视频数据时会强制切片，限制每个数据包的大小 |

RTMP及其变种

| 类型     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| 基本协议 | 建立在TCP长连接通道上，明文传输，默认端口1935                |
| RTMPT    | 通过HTTP隧道传输，将数据包封装在HTTP中，可穿越防火墙         |
| RTMPS    | 基于SSL/TLS加密的安全版本的RTMP协议，使用的是HTTPS隧道传输，提供了完整的数据加密和身份验证功能 |
| RTMPE    | 基于SSL/TLS加密的安全版本的RTMP协议，提供端到端的加密通信    |
| RTMFP    | 基于UDP传输，用于P2P通信                                     |



### RTSP

**介绍**：RTSP定义了一对多应用程序如何有效地通过IP网络传送多媒体数据。RTSP提供了一个可扩展框架，数据源可以包括实时数据与已有的存储的数据。该协议目的在于控制多个数据发送连接，为选择发送通道如UDP、组播UDP与TCP提供途径，并为选择基于RTP上发送机制提供方法。

RTSP语法和运作跟HTTP/1.1类似，但并不特别强调时间同步，所以比较能容忍网络延迟。代理服务器的缓存功能也同样适用于RTSP，并且因为RTSP具有重新导向功能，可根据实际负载情况来切换提供服务的服务器，以避免过大的负载集中于同一服务器而造成延迟。

| 属性    | 说明                                                   |
| ------- | ------------------------------------------------------ |
| 全称    | Real-Time Streaming Protocol，实时流传输协议           |
| 用途    | 一般用作摄像头、监控等硬件设备的实时视频流观看与推送上 |
| web支持 | 不支持                                                 |





#### RTP

**全称**：Real-Time Transport Protocol，实时传输协议

**介绍**：RTP是针对多媒体数据流的一种传输层协议，详细说明了在互联网上传递音频和视频的标准数据包格式。RTP协议常用于流媒体系统（配合RTCP协议），视频会议和一键通系统（配合H.323或SIP），使它成为IP电话产业的技术基础。

RTP是建立在UDP协议上的，常与RTCP一起使用，其本身并没有提供按时发送机制或其它服务质量（QoS）保证，它依赖于低层服务去实现这一过程。

RTP 并不保证传送或防止无序传送，也不确定底层网络的可靠性，只管发送，不管传输是否丢包，也不管接收方是否有收到包。RTP 实行有序传送，RTP中的序列号允许接收方重组发送方的包序列，同时序列号也能用于决定适当的包位置，如在视频解码中，就不需要顺序解码。



#### RTCP

**全称**：Real-Time Transport Control Protocol，实时传输控制协议

**介绍**：RTCP是RTP的配套协议，为RTP媒体流提供信道外的控制。RTCP和RTP一起协作将多媒体数据打包和发送，定期在多媒体流会话参与者之间传输控制数据。

RTCP的主要功能是为RTP所提供的服务质量（QoS）提供反馈，收集相关媒体连接的统计信息，例如传输字节数，传输分组数，丢失分组数，单向和双向网络延迟等等。网络应用程序可以利用RTCP所提供的信息来提高服务质量，比如限制流量或改用压缩比小的编解码器。



### HTTP-FLV

| 属性    | 说明                                                     |
| ------- | -------------------------------------------------------- |
| 地址    | http://                                                  |
| 用途    | 一般用作直播观看/拉流                                    |
| 原理    | 将流媒体数据封装成 FLV格式，然后通过HTTP协议传输给客户端 |
| web支持 | H5需要使用插件                                           |
| 其他    | 主流浏览器摒弃flash，不适合多音轨                        |



### HLS

| 属性       | 说明 |
| ------- | ---------------------------------------------------------- |
| 全称    | HTTP Living Streaming，基于HTTP的自适应码率流媒体传输协议  |
| 地址 | http:// 开头  .m3u8 结尾 |
| 基于    | HTTP短连接                                                 |
| 用途 | 一般用作拉流观看 |
| 原理    | 集合一段时间的数据，生成TS切片文件（三片），并更新m3u8索引。本质是通过HTTP协议下载静态文件。 |
| web支持 | 支持H5                                                     |
| 其他    | 播放时需要多次请求，对于网络质量要求高。CDN加速效果明显           |



### WebRTC

| 属性 | 说明                                      |
| ---- | ----------------------------------------- |
| 全称 | Web Real-Time Communication，网页实时通信 |
| 地址 | webrtc://                                 |
| 基于 | UDP                                       |
| 用途 | 点对点的视频/语音通话协议                 |
|      |                                           |







------



## 其他协议

### NFS

Network File System 网络文件系统

由SUN公司研制的UNIX表示层协议（presentation layer protocol）**文件共享协议**，能使使用者访问网络上别处的文件就像在使用自己的计算机一样。

1、NFS只提供基本的文件处理功能，而**不提供任何四层TCP/IP与OSI七层数据传输功能**，需要借助 RPC 协议才能实现 TCP/IP 数据传输功能

2、NFS 默认没有加密，对客户端来说是完全透明的，且仅依靠 IP 地址或主机名来决定是否允许客户端挂载指定的共享目录【**明文传输**】，需要可通过 Kerberos 进行认证及加密

3、NFS与其他文件共享协议共同点：使用C/S 架构

4、NFS实现原理：共享资源的属主、属组和权限。

5、NFS 服务器和客户端通过 UID 和 GID 来识别共享资源的所有者信息。当客户端挂载 NFS 共享目录时，共享目录中资源的 UID 和 GID 将与服务器上面的保持一致；而客户端会将 UID 和 GID 映射到客户端上所对应的用户名和组名。NFS 服务器与客户端上共享资源的权限及 ACL 信息（若支持）将保持一致。

### SMB协议

网络文件共享系统协议、（Server Message Block）  服务器消息块

允许应用程序和终端用户从远端的文件服务器访问文件资源，CIFS（通用互联网文件系统 Common Internet File System）是SMB的衍生协议。

1、SMB与NFS的区别：操作系统OS不同，NFS适配linux操作系统，SMB适配windows/linux，但是兼容linux时相关存储性能会收到影响。

2、SMB协议运用过程：SMB协议协商（Negotiate）> 建立SMB会话（Session Setup）> 连接一个文件分享（Tree Connect）> 文件系统操作 > 断开文件分享连接（Tree Disconnect）>  终止SMB会话（Logoff）

### POSIX协议

可移植操作系统接口POSIX（Portable Operating System Interface）是IEEE为要在各种UNIX操作系统上运行软件，而定义API的一系列互相关联的标准的总称。

### RESTful API

RESTful是一种[网络应用程序](https://www.zhihu.com/search?q=网络应用程序&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A"644349955"})的设计风格和开发方式，基于HTTP做传输协议，可以使用XML格式定义或JSON格式定义，跟[编程语言](https://www.zhihu.com/search?q=编程语言&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A"644349955"})、平台都无关。RESTFUL适用于移动互联网厂商作为业务接口的场景，实现第三方OTT调用移动网络资源的功能，动作类型为新增、变更、删除所调用资源。





网络协议 https://blog.csdn.net/qq_35512802/article/details/129433099



## 物联网

### Modbus

应用于电子控制器上的一种通用语言，通过此协议，可以实现控制器相互之间或通过网络实现通信



### CoAP

全称：Constrained Application Protocol，约束应用协议

基于 请求/响应 模型，默认运行在UDP

CoAP协议是HTTP协议的简化版，通过4个请求方法（GET, PUT, POST, DELETE）对服务器端资源进行操作



### MQTT

全称：Message Queuing Telemetry Transport，消息队列遥测传输协议

基于 发布/订阅 模式的“轻量级”通讯协议，该协议构建于TCP/IP协议上

MQTT最大优点在于，用极少的代码和有限的带宽，为连接远程设备提供实时可靠的消息服务。作为一种低开销、低带宽占用的即时通讯协议，使其在物联网、小型设备、移动应用等方面有较广泛的应用

![MQTT消息发送]({{ '/images/network/MQTT消息发送.jpg' | prepend: site.baseurl  }})



### AMQP

全称：Advanced Message Queuing Protocol，高级消息队列协议

AMQP的主要特征是面向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全。

RabbitMQ是一个开源的AMQP实现，服务器端用Erlang语言编写，支持多种客户端

![AMQP模型]({{ '/images/network/AMQP模型.png' | prepend: site.baseurl  }})
