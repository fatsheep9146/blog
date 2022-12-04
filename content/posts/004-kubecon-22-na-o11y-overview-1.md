---
title: "kubeCon 2022 北美 observability talks 总览"
date: 2022-09-13T18:59:25+08:00
draft: true 
---

2022 KubeCon 北美站于十月在底特律召开。可观测性(observability)作为一个热门主题，有多达 7 个 talk。

最近 kubecon 官方也把所有 talk 的录像都放出了，所以在这篇文章主要就是记录了一下我对这几个 talk 录像的观后感，以及引发的思考。

### Customer Centric Observability: How Intuit Reduced Time To Detect Customer Impact From 30+ Minutes To Under 3 Minutes.

直译：以客户为中心的可观测: Intuit 公司是如何将对客户体验有损的发现速度从30分钟+提速到3分钟

Intuit 公司是一家以财务软件为主的高科技公司，包含多款面向终端用户的 SAAS 产品。而且根据演讲者的介绍所有产品都是在 Intuit 的 5 大基础平台之上所构建出来的。

![Intuit的5个基础平台(图片来自kubecon视频演讲截图)](/img/004-kubecon-22-na-o11y-overview-1/intuit-overview.jpg)

而演讲者所在的负责的部门，就是5个基础平台之一，开发平台。他们的目标就是：让研发人员能更快更稳定的进行开发迭代。从他所提供的平台规模数据（百万核，2000+的微服务，900+团队，6000+开发人员，16000+命名空间）来看，要达成这个目标是非常有挑战的。

讲着总结了在优化前，他们的稳定性建设的思路是“以服务为中心进行监控”(system centric monitoring)，而这种思路也确实让他们遇到很多问题。

- 缺乏

所以该团队重新思考，从用户体验出发，实现以用户为中心的可观测性建设，从而更加直观的判断对于用户体验的影响。

当然团队并不是要把产品中每一个用户可能的动作都视作为核心用户体验，他们为核心用户体验也制定了自己的定义：

FCI 平台



### SLO-Based Observability For All Kubernetes Cluster Components

直译：对所有 kubernetes 组件进行基于 SLO 可观测性建设。

在这个 talk 中，介绍了一个叫做 Pyrra 的开源项目，该项目的目标是构造一个，以 Prometheus 为数据源，以更低成本对 SLO 进行管理的平台。

SLO 是 SRE 发现系统稳定性隐患的重要抓手，也是驱动系统稳定性提升的数据支撑。但是对于大多数用户来说，如何管理自己的 SLO，甚至是如何定义一个 SLO 都是一个不容易的问题。演讲者也特别在 talk 中特别提到在他们发起这个项目之前，他们特意找了一些客户做了一些调研和咨询，收集到了一些客户在使用 SLO 遇到的常见问题。

在这个背景下，Pyrra 项目也诞生了。作为一个 SLO 管理平台，架构还是比较简单的，就是一个前端 ui 和后端 server 的标准架构，当然还依赖于一个 prometheus 作为数据源。部署方式支持 kubernetes, docker 和 systemd。

而这套平台为用户提供的能力是

- 快速，灵活的创建 SLO 规则
- 功能齐全，开箱即用的 SLO 管理 UI 界面

# 参考
1. https://kccncna2022.sched.com/event/182Fz/customer-centric-observability-how-intuit-reduced-time-to-detect-customer-impact-from-30-minutes-to-under-3-minutes-vinit-samel-intuit-nagaraja-tantry-intui?iframe=no&w=100%&sidebar=yes&bg=no
2. https://events.linuxfoundation.org/kubecon-cloudnativecon-north-america/program/schedule/