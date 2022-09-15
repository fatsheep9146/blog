---
title: "003 Exemplar Research"
date: 2022-09-13T18:59:25+08:00
draft: true
---

# Exemplar 调研

# 什么是 exemplar，为什么要用 exemplar

# 如何使用 exemplar

目前一个 golang 程序，想要使用 exemplar 能力，需要做如下几件事情

- 监控埋点改造，加入 exemplar 信息
- 暴露 openmetrics 协议监控数据 

## 监控埋点改造，加入 exemplar 信息

TBD 阐述一下原来的方式

需要使用 1.4.0 以上版本的 prometheus golang library 进行埋点，并且调用专门为 exemplar feature 增加的埋点函数



TBD 
- ObserveWithExemplar
- ...

```
curl -H "Accept: application/openmetrics-text" 127.0.0.1:7777/metrics
```
