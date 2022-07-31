---
title: "opentelemetry-collector 代码解析"
date: 2022-07-25T09:07:26+08:00
draft: true
---

TBD 对 opentelmetry collector 的功能，定位的基本介绍

首先，通过对官方文档中有关[配置文件](https://opentelemetry.io/docs/collector/configuration/)的介绍，可以了解到，collector 内部包含几类重要的组件，

- exporter
- processor
- receiver
- extension

每一类组件都有自己的定位，然后最终再通过 pipeline 最终将上述组件串联成一个数据流。

举个例子，下面的配置文件定义了

- 一个 类型为 otlp，名称为 r 的 receiver 对象
- 一个 类型为 batch，名称为空 的 processor
- 一个 类型为 otlp，名称为空的 exporter

并且最终由一个类型为 traces 的 pipeline 将上述 3 者串联了起来

```yaml
receivers:
  otlp/r:
    protocols:
      grpc:

processors:
  batch:

exporters:
  otlp:
    endpoint: otelcol:4317

service:
  pipelines:
    traces:
      receivers: [otlp/r]
      processors: [batch]
      exporters: [otlp]
```

##  如何实现各种不同的组件？

由于 collector 中包含了非常多

- 通用一致性
  - 接口
- 可扩展性
  - Factory 类

为了更好的管理，组织并且实现这些组件，collector 的代码实现中，定义了几个通用概念/接口，从而为各种组件

- Component: 暴露了如下两个方法
  - Start: 启动组件
  - Shutdown: 停止组件运行
- consumer.*: 这里指的是 consumer package 中定义的针对不同数据类型的接口
  - consumer.Traces: 定义了 `ConsumeTraces(ctx context.Context, td ptrace.Traces) error` 方法，从方法名中就可以看出是在消费 traces 类型数据
  - consumer.Metrics: 定义了 `ConsumeMetrics(ctx context.Context, md pmetric.Metrics) error`，类似 consumer.Traces
  - consumer.Logs: 定义了 `ConsumeLogs(ctx context.Context, ld plog.Logs) error`，类似 consumer.Traces

每一种类型的组件（exporter，receiver 等）都是以 Component, consumer.* 的 interface 为基础进一步构造自己的接口

## 如何实现 pipeline

- 复用