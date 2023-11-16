---
title: "Kubernetes Tracing 能力介绍"
date: 2023-01-24T18:59:25+08:00
draft: true 
tags: ["kubernetes", "trace", "observability", "o11y"]
---

# Trace 和 OpenTelemetry

Trace 是分布式系统下用于定位问题的利器，相比于非结构化的日志数据，trace 可以更加详细的展示一个事务的完整链路。通过 trace 内详细的上下文信息，可以更快捷的确认问题的根因。

OpenTelemetry 是目前可观测领域对于 Trace/Log/Metric 三种可观测数据类型定义的统一标准，并且也为这套数据标准提供了一系列针对各种编程语言（Go，Java 等）的标准 SDK。通过这套统一的数据标准，用户再也不用担心被某种可观测服务提供商（比如 datadog，splunk）绑定，每次切换监控方案都要对代码进行伤筋动骨的改造。因为目前大部分可观测服务提供商都已经兼容了 OpenTelemetry 的数据协议，只要用户的代码使用的是 OpenTelemetry 的 SDK 输出数据，那就可以无需修改代码输出到任意一种可观测服务提供商中。

# Kubernetes 对 Trace 能力的支持

Kubernetes 对于 Trace 的需求由来已久。在 2018 年，社区就已经提出了希望为 Kubernetes 实现标准的 Tracing 能力的期望，KEP 链接 https://github.com/kubernetes/enhancements/issues/647（BTW 当时 k8s 的版本还是 1.14）。

但是整整过了 2 年，这个 feature 一直没有明显进展。在 2020 年底，Google 的 dashpole 终于开始启动了正式的设计开发工作，这个时间恰好和 OpenTelemetry 项目中 Trace 数据协议正式 Release 时间比较吻合，https://medium.com/opentelemetry/tracing-specification-release-candidate-ga-p-eec434d220f2。盲猜一下，正是 OpenTelemetry 的真正落地让社区也可以更有信心的把 Kubernetes 的 Tracing 能力以标准的 OpenTelemetry 协议向前演进。

那么到今天为止，kubernetes 的 tracing 能力发展到什么程度了呢？我通过下面的这张 kubernetes 的架构图来为大家阐述。

TBD 架构图

社区对于 Trace 能力的支持也是按照组件的类型，逐步进行覆盖，目前覆盖情况如图中所示

- [x] APIServer，
- [x] etcd
- [x] webhook
- [x] kubelet
- [] controllers(kube-scheduler/kube-controller-manager)

为什么 controllers 还没有被覆盖，这个会在下面专门说明。

# kubernetes 现有的 Tracing 能力能解决什么问题

虽然现在 tracing 能力还没有注入到每一个组件中，但是现在已有的能力，已经能够帮助我们解决一些问题了。

下面我将结合一些实际的 demo 样例，让读者可以体验这些问题

## APIServer 

APIServer 作为一个典型的 http server 类型的服务，是非常适合通过 trace 埋点来确认他自身的状态的。所以我梳理了3个比较典型的使用场景

### APIServer 组件黄金 SLO 指标透出

通过 trace 的原始数据，结合 opentelemetry 的 span 转换为 metrics 的能力（spanmetrics connector）。我们可以直接绘制出 APIServer 的成功率，APIServer 写入延时，读取延时

按照 demo 中的指导启动 kind 集群，并且查看大盘即可看到

### APIServer 异常请求基于 trace 分析

针对个别的异常请求，我们可以直接在 trace 的分析系统中进行检索，从而获取异常 trace 的完整调用链

### APIServer 基于 exemplar 的异常快速定位

通过 exemplar 的能力，我们还可以实现通过异常 metrics 波动快速跳转到引起波动的异常 trace 的能力。

## kubelet 

### kubelet pod 创建成功率

## controller

### 针对 pod 的全链路

### controller 目前有什么问题


# 参考文档

1. https://kubernetes.io/blog/2021/09/03/api-server-tracing/
2. https://kubernetes.io/docs/concepts/cluster-administration/system-traces/


# TODO
- 是不是可以借由 otel 的强大能力，进一步缩减 apiserver 需要的监控量，预先计算一些 p99 指标