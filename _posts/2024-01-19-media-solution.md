---
layout: post
title:  视频直播解决方案
date:   2024-01-15 00:00:00 +0800
categories: 智慧矿山
tag: 解决方案
---

* content
{:toc}
## 第三方解决方案

### 华为云

**产品**：视频直播Live

**介绍**：https://support.huaweicloud.com/productdesc-live/live030001.html

**功能**：



| 功能     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| 推流     | 采集、编码、封装后，推流到华为云<br />推流协议：RTMP协议<br />推流工具：OBS、Xsplit、FMLE等工具 |
| 播放     | 可以播放推流的视频，及第三方源站的视频<br />播放协议：RTMP、HTTP-FLV、HLS、WebRTC <br />播放器：常见播放器VLC等、华为云SDK |
| 录制     | 将直播内容录制并存储到OBS                                    |
| 转码     | 转码成多种分辨率和码率规格的视频流                           |
| 截图     | 按配置的模板截取画面存在OBS                                  |
| 通知     | 收到/断开 推流有回调通知(发送HTTP请求)                       |
| 访问控制 | 鉴权、防盗链、黑白名单                                       |
| 辅助功能 | 用量统计、服务状态监控、日志管理、审计                       |



**产品架构**

![华为云_视频直播产品架构]({{'/styles/images/media/华为云_视频直播产品架构.png' | prepend: site.baseurl}})

------

### 阿里云

**产品**：视频直播

**介绍**：https://help.aliyun.com/zh/live/product-overview/what-is-apsaravideo-live

**功能**：

| 功能     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| 推流     | 标准、超低延时、互动                                         |
| 转码     | 普通转码支持流畅、标清、高清、超清多种码率格式、转码视频宽高比自适应。窄带高清转码在普通转码的基础上可极大降低码率，提升清晰度 |
| 录制     | 支持FLV、MP4、M3U8格式录制，支持自定义录制时长               |
| 播放     | 主备切换<br />播放协议：RTMP、HTTP-FLV、HLS等<br />提供SDK、实时调整推流参数、码流、帧率、水印等<br />支持时移 |
| 时移     | 直播时移可以回看从直播开始时间到当前时间之间的直播视频。     |
| 截图     | 支持实时覆盖截图，支持实时截图存储，可自定义截图频率。       |
| 水印     | 支持查询、添加、编辑、删除水印模板。                         |
| 监播     | 广目系统为各类直播项目提供实时监播功能，并对帧率码率变化、音视频同步、延迟和卡顿等异常情况时进行告警，为各类专业直播保障护航。 |
| 云导播台 | 节目制作全链路上云，对传统视频生产全面革新。融合实时媒体处理能力、纯幕和实景抠像合成、ASR语音转文本及实时翻译、视频AI及实时图文特效等多种直播、互动能力，可满足标准直播、广电级专业直播、轮播台、虚拟演播厅等各种直播场景，即开即用简单便捷。 |
| 审核     | 支持视频审核，对视频画面内容进行截帧，进行内容安全审核。支持音频审核，对视频语音内容进行内容安全审核。 |
| 加密     | 支持私有加密，对直播视频数据进行阿里云私有加密；支持DRM加密，对直播内容进行Widevine与Fairplay的DRM加密。 |
| 实时剪辑 | 直播流剪辑可在直播过程中实时剪辑并输出视频内容。             |
| 辅助功能 | 动作识别、用量统计、业务数据统计监控                         |



**产品架构**

![阿里云_视频直播产品架构]({{'/styles/images/media/阿里云_视频直播产品架构.png' | prepend: site.baseurl}})

------

### 腾讯云

**产品**：实时音视频、云直播

**实时音视频介绍**：https://cloud.tencent.com/document/product/647/16788

**云直播介绍**：https://cloud.tencent.com/document/product/267/59018

**功能**：增加了微信小程序的优势



**产品架构**

实时音视频

![腾讯云_实时音视频_产品架构]({{'/styles/images/media/腾讯云_实时音视频_产品架构.svg' | prepend: site.baseurl}})

云直播

![腾讯云_云直播_产品架构]({{'/styles/images/media/腾讯云_云直播_产品架构.png' | prepend: site.baseurl}})

------



## 流媒体服务器

流媒体服务器是搭建视频网站平台和各类在线视频应用系统的基础支撑系统，实现将视频存储、视频转码、视频播出、协议复用、终端适配、大并发播出等的工作集中处理，这样在搭建视频网站时就可以只关注业务细节而不用再去处理与视频相关的诸多技术细节，从而实现提高项目实施效率、降低项目实施风险的目标。



### NTV Media Server G3

单节点突破5000并发，直播、点播、转码、录播、防盗链一站解决。RTMP、HTTP、HLS等多协议播出，面向PC、智能手机、微信公众号和小程序等提供流畅的视频播出。成熟的商用系统，自主产权，稳定可靠，教育、传媒、科研、政府、企业，超过1000家客户的选择

Linux操作系统，性能极高，单节点并发超5000，系统成熟，接口和文档都比较完善，有较大的用户群，售后服务好

官网：http://www.ruiboyun.com/

说明文档：http://pro8eb17329.pic3.ysjianzhan.cn/upload/ntv_media_server_v3.pdf



### Adobe Flash Media Server

官网：https://business.adobe.com/cn/products/primetime/adobe-media-server.html



### Ti Top Streamer

官网：https://www.ttstream.com/

说明文档：https://www.ttstream.com/document





### 常见开源服务

| 产品     | Red5                                | jitsi                      | EasyDarwin                               | ZLMediaKit                                                   | SRS                             | Nginx_RTMP |
| -------- | ----------------------------------- | -------------------------- | ---------------------------------------- | ------------------------------------------------------------ | ------------------------------- | ---------- |
| 收费情况 | 免费、开源                          | 免费、开源                 | 免费、开源                               | 免费、开源                                                   | 免费、开源                      | 免费、开源 |
| 语言     | Java                                | Java                       | Go                                       | C++                                                          | C++                             |            |
| 支持平台 | Windows、OS X、Linux、Unix          |                            | Windows、Linux、OS X                     | Windows、Linux、OS X                                         | OS X、Linux                     |            |
| 支持协议 | HTTP、RTMP、HLS                     |                            | RTSP                                     | RTSP、RTMP、HLS、HTTP-FLV、WebSocket-FLV、GB28181、HTTP-TS、WebSocket-TS、HTTP-fMP4、WebSocket-fMP4、WebRTC |                                 |            |
| 官网     | http://red5.org/                    | https://desktop.jitsi.org/ |                                          | https://docs.zlmediakit.com/                                 | http://www.ossrs.net/lts/zh-cn/ |            |
| 代码     | https://github.com/Red5/red5-server | https://github.com/jitsi   | https://github.com/EasyDarwin/EasyDarwin | https://github.com/ZLMediaKit/ZLMediaKit                     | https://github.com/ossrs/srs    |            |

**开源的问题**：没发保障高QOS





## 自研系统

**核心功能**：对接 摄像头、三方监控系统等视频源采集数据，将数据处理后存储，并可实时查看视频，以及回放视频。

**辅助功能**：视频转码、截图、时移、加密、访问控制、用量监控、服务监控、动作识别



### 系统架构

#### 采集端

核心功能：音视频采集、视频处理、音视频编码压缩-、音视频封装

常用框架：AVFoundation、FFmpeg、X264、libremp



#### 服务器端

核心功能：数据分发CDN、音视频转码/录制/截屏

常用服务器：SNS、BMS、Nginx



#### 播放端

核心功能：读取音视频数据、音视频解码、播放、截图

常用框架：ijkplayer、FFmpeg、VideoToolbox/AudioToolbox



#### 存储端

NoSQL数据库：MongoDB、Redis、Cassandra

归档存储：文件服务器、FTP/NFS
