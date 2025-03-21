---
layout: post
title:  音视频处理
date:   2024-01-16 00:00:00 +0800
categories: 智慧矿山
tag: 学习
author: YangLe
---


## 音视频处理框架

### OpenGL

Open Graphics Library

定义了一个跨编程语言、跨平台的编程接口的规格，它用于三维图象（二维的亦可）。OpenGL是个专业的图形程序接口，是一个功能强大，调用方便的底层图形库



### OpenGL ES

OpenGL for Embedded Systems

是 OpenGL三维图形) API 的子集，针对手机、PDA和游戏主机等嵌入式设备而设计



### GPUImage

GPUImage是一个基于OpenGL ES的一个强大的图像/视频处理框架,封装好了各种滤镜同时也可以编写自定义的滤镜,其本身内置了多达120多种常见的滤镜效果



### FFmpeg

一个跨平台的开源视频框架,能实现如视频编码，解码，转码，串流，播放等丰富的功能。其支持的视频格式以及播放协议非常丰富，几乎包含了所有音视频编解码、封装格式以及播放协议

**库介绍**

| 库            | 介绍                                                         |
| ------------- | ------------------------------------------------------------ |
| Libswresample | 可以对音频进行重采样,rematrixing 以及转换采样格式等操作      |
| Libavcodec    | 提供了一个通用的编解码框架,包含了许多视频,音频,字幕流 等编码/解码器 |
| Libavformat   | 用于对视频进行封装/解封装                                    |
| Libavutil     | 包含一些共用的函数，如随机数生成，数据结构，数学运算等       |
| Libpostproc   | 用于进行视频的一些后期处理                                   |
| Libswscale    | 用于视频图像缩放，颜色空间转换等                             |
| Libavfilter   | 提供滤镜功能                                                 |




ffmpeg文档 https://ffmpeg.github.net.cn/

ffmpeg介绍 https://zhuanlan.zhihu.com/p/89872960

ffmpeg基础 https://ffmpeg.xianwaizhiyin.net/

ffmpeg与java结合 https://zhuanlan.zhihu.com/p/38961122

ffmpeg使用 https://www.zhihu.com/pub/reader/120332508/chapter/1545870486378266624