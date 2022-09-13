---
title: "opentelemetry-collector 代码解析"
date: 2022-07-25T09:07:26+08:00
draft: true
---

TBD 对 opentelemetry collector 的功能，定位的基本介绍

这篇文章尝试对 opentelemetry collector 的代码实现基于自己的理解做一个简单的介绍。

首先，通过对官方文档中有关 collector 的[配置文件](https://opentelemetry.io/docs/collector/configuration/)的介绍，可以了解到，collector 内部包含几类重要的组件(component)，

- exporter
- processor
- receiver
- extension

每一类组件都有自己的定位，最终再通过 pipeline 对象将上述组件串联成一个数据的流。

举个例子，下面的配置文件定义了

- 一个 receiver 对象，类型为 otlp，名称为 r1 
- 一个 processor 对象，类型为 batch，名称为空
- 一个 exporter 对象，类型为 otlp，名称为 e1

并且最终由一个处理 traces 数据的 pipeline 对象将上述 3 者串联了起来

```yaml
receivers:
  otlp/r1:
    protocols:
      grpc:

processors:
  batch:

exporters:
  otlp/e1:
    endpoint: otelcol:4317

service:
  pipelines:
    traces:
      receivers: [otlp/r1]
      processors: [batch]
      exporters: [otlp/e1]
```

当然这个只是 collector 最基本的一种使用样例，在这基础之上，还有更多更灵活的用法

- 组件复用：比如同一个组件，可以被多个 pipeline 所共享，比如定义了 2 种 receiver ，都希望最终导入到一个 exporter 中
- 自定义组件扩展：用户可以根据自己的需求，开发自己想要的组件，满足自己的需求。TBD 简单举几个例子

所以通过对代码的研读，我认为 collector 代码设计的核心思想，就是为了在实现基本功能的基础之上，还能更优雅，高效的实现上面提到的组件复用，自定义组件扩展的能力。

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