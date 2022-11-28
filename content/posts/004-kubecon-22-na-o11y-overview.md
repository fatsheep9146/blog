---
title: "kubeCon 2022 北美 observability talks 总览"
date: 2022-09-13T18:59:25+08:00
draft: true 
---

2022 KubeCon 北美站于十月在底特律召开。可观测性(observability)作为一个热门主题，有多达 7 个 talk。

### Customer Centric Observability: How Intuit Reduced Time To Detect Customer Impact From 30+ Minutes To Under 3 Minutes.

以客户为中心的可观测: Intuit 公司是如何将对客户体验有损的监测速度从平均30+分钟提升到不到3分钟


### SLO-Based Observability For All Kubernetes Cluster Components

直译：对所有 kubernetes 组件进行基于 SLO 可观测性建设。

在这个 talk 中，介绍了一个叫做 Pyrra 的开源项目，该项目的目标是构造一个 以 Prometheus 的 metrics 数据为基础，实现以更低成本对 SLO 进行管理的平台。

SLO 是 SRE 发现系统稳定性隐患的重要抓手，也是驱动系统稳定性提升的数据支撑。但是对于大多数用户来说，如何管理自己的 SLO，甚至是如何定义一个 SLO 都是一个不容易的问题。演讲者也特别在 talk 中特别提到在他们发起这个项目之前，他们特意找了一些客户做了一些调研和咨询，收集到了一些客户在使用 SLO 遇到的常见问题。

在这个背景下，Pyrra 项目也诞生了。

Pyrra 项目