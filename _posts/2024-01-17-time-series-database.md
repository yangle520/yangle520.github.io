---
layout: post
title:  时序数据库
date:   2024-01-17 00:00:00 +0800
categories: 基础知识
tag: 智慧矿山
---



## 介绍

时间序列数据库(Time Series Database)是用于存储和管理时间序列数据的专业化数据库，具备写多读少、冷热分明、[高并发](https://www.zhihu.com/search?q=高并发&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A"2324622450"})写入、无事务要求、海量数据持续写入等特点，可以基于时间区间[聚合分析](https://www.zhihu.com/search?q=聚合分析&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A"2324622450"})和高效检索，广泛应用在物联网、[经济金融](https://www.zhihu.com/search?q=经济金融&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A"2324622450"})、环境监控、工业制造、农业生产、硬件和软件系统监控等场景。







## 常见时序数据库

### InfluxDB

InfluxDB是自研存储，通过hash分片的方式来存储海量数据

通过LSM方式来实现数据的快速写入

### Kdb+



### Prometheus



### Graphite



### OpenTSDB

OpenTSDB自身并不实现分布式存储，而是借助于HBase或者Cassandra来存储海量的数据



### Apache Druid

开源

分布式的、支持实时多维 OLAP 分析的数据处理系统。既支持高速的数据实时摄入处理，也支持实时且灵活的多维数据分析查询，支持根据时间戳对数据进行预聚合摄入和聚合分析。

数据结构：时间、维度、指标

