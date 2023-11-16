---
title: "如何将 OpenTelemetry Metric 数据导入 Prometheus 中?"
date: 2023-08-06T18:59:25+08:00
draft: false 
tags: ["opentelemetry", "metric", "observability", "o11y", "opentelemetry-collector", "prometheus"]
---

# 一句话总结

本文介绍如何将 OpenTelemetry Metric 数据导入 Prometheus 中，并且通过可运行的 demo 样例介绍几种常见的使用模式。

-------------

# 什么是 OpenTelemetry Metric 数据？

OpenTelemetry 协议中目前包含 3 类可观测数据 —— Trace，Metric，Log。其中 Metric 类型的数据通常用于衡量系统在一段时间内表现的统计值，比如请求成功率；或者系统表现的瞬时值，比如 CPU 利用率。

Prometheus 作为云原生领域 Metric 技术的事实标准，显然也是最适合作为 OpenTelemery Metric 数据的后端存储方案。但是 Prometheus 原生的 Metric 数据协议同 OpenTelemetry Metric 数据协议还是有诸多不一致的地方，如何才能将 OpenTelemetry Metric 数据写入 Prometheus 中呢？

# 如何生产 OpenTelemetry Metric 数据？

OpenTelemetry 提供了有关 OpenTelemetry Metric 的 API，以及基于这套协议，针对各种编程语言实现的 SDK。基于官方提供的 SDK，我们就可以生产 OpenTelemetry Metric 数据了。

参考如下代码，是通过 golang 实现的输出一个最简单的 Counter 类型指标的程序

https://github.com/fatsheep9146/awesome-observability/blob/main/services/otel-metric-producer/main.go

```golang
package main
...

var mode string

func init() {
	flag.StringVar(&mode, "mode", "otlphttp", "the metric reader mode, support otlphttp and prometheus")
}

func main() {
	flag.Parse()

	var reader metricsdk.Reader
	switch mode {
	case "otlphttp":
		httpexporter, _ := otlpmetrichttp.New(context.TODO())
		reader = metricsdk.NewPeriodicReader(httpexporter)
	case "prometheus":
		reader, _ = prometheus.New()
		go serveMetrics()
	}

	provider := metricsdk.NewMeterProvider(metricsdk.WithReader(reader))

	meter := provider.Meter("github.com/open-telemetry/opentelemetry-go/example/prometheus")

	testInt64Counter, err := meter.Int64Counter("test_int64_counter", metricapi.WithDescription("the counter of http requests"))
    ...

	for i := 0; i < 10; i++ {
		testInt64Counter.Add(context.TODO(), int64(i), metricapi.WithAttributes(
			attribute.Key("testkey").String("testvalue")))
		time.Sleep(20 * time.Second)
		log.Printf("add %v\n", i)
	}
}
...
```

上面的代码中，整体包括 4 个步骤：

- 根据输入的 mode 参数，构造一个 metricsdk.Reader 对象 reader
- 基于 reader 对象构造一个 metricsdk.MeterProvider 对象 provider
- 通过 provider 构造一个 metricsdk.Meter 对象 meter
- 通过 meter 构造一个 count 类型的 metric 名字叫做 test_int64_counter，一般也把它叫做 Instrument 类型

通过官方提供的示意图，我们可以大体理解 MeterProvider，Meter，Instrument 三者之间的层次关系

```
+-- MeterProvider(default)
    |
    +-- Meter(name='io.opentelemetry.runtime', version='1.0.0')
    |   |
    |   +-- Instrument<Asynchronous Gauge, int>(name='cpython.gc', attributes=['generation'], unit='kB')
    |   |
    |   +-- instruments...
    |
    +-- Meter(name='io.opentelemetry.contrib.mongodb.client', version='2.3.0')
        |
        +-- Instrument<Counter, int>(name='client.exception', attributes=['type'], unit='1')
		|
		+-- Instrument<Histogram, double>(name='client.duration', attributes=['server.address', 'server.port'], unit='ms')
```

其中 
- MeterProvider 是用来生产 Meter 对象的工厂
- Meter 通常代表一个应用程序内，用来生产一组属于同一范畴的 Metric 的工厂类。比如你的程序中，既引用了 net/http 库去构造一个 http server，也引用了 grpc 库，和其他 grpc server 进行通讯。那我们需要为这两个库分别构造不同的 Meter 对象。分别构造 http server 相关的 Metric 和 grpc 相关的 metric。
- Instrument 就是一个个具体的指标了


而 Metric Reader 对象主要负责最终将 Metric 的数据透出，目前 go 语言官方 SDK 支持 OTLP 和 Prometheus 两种数据透出方式

- otlp exporter：这种 exporter 代表的就是以 OTLP 官方协议输出 metric 数据的模式，是一种主动 push 的模式。
- prometheus exporter：这种 exporter 顾名思义，就是将数据转换为 prometheus 协议的数据，然后再通过监控一个端口，以 prometheus 指标抓取协议，是被动 pull 的模式。

# 如何将 OpenTelemetry Metric 传入 prometheus? 

从上一部分，我们知道，通过 OpenTelemetry SDK，我们可以将 Metric 通过 OpenTelemery 和 Prometheus 两种协议透出。

在此基础之上，我们可以进一步讨论，如何将数据传入 Prometheus，目前整体支持两种模式
- 一是直接将数据传入 Prometheus，
- 二是以 OpenTelemetry Collector 作为中继，导入 Prometheus。

这两种使用模式各自有什么好处？OpenTelemetry 官方文档已经为我们列举了 2 者之间的优劣处

- [直接传入 Prometheus，无 Collector](https://opentelemetry.io/docs/collector/deployment/no-collector/)：优势是使用简单，无需运维部署 collector，很适合本地测试调试；缺陷是新的数据加工需求需要代码层面改动来支持，并且不同编程语言对某种 backend 支持程度不一。
- [采用 collector](https://opentelemetry.io/docs/collector/deployment/gateway/)：优势是可以通过 collector 丰富的插件，通过配置即可满足各种数据加工需求，并且同后端 backend 解耦；缺陷是需要运维 collector，所以非常适合生产环境的运行模式。

## 直接传入 Prometheus

无论 app 通过 otlp，还是 prometheus 透出 metric 数据，prometheus 目前都支持直接传入。

### 针对 prometheus 格式数据，通过 prometheus 默认 pull 模式采集

这是 prometheus 最传统的使用方法，Prometheus 通过 Pull 的方式，直接访问应用程序获取指标。如下图所示

我在 github 上创建了一个模拟这种模式的 demo，可以通过 docker compose 来体验，通过如下命令
```
git clone git@github.com:fatsheep9146/awesome-observability.git
cd awesome-observability/demos/otel-to-prometheus/demo-1
docker compose up --no-build
```

启动后过一段时间，即可通过页面查看到具体的指标

### 针对 otlp 格式数据，通过 prometheus otlp receiver 以 push 模式导入

prometheus 目前也开始积极支持 opentelemetry 协议了。当然目前还处于 experiment 阶段。应用程序可以直接通过 otlp 格式将数据 push 到 prometheus 中。

你可以通过如下命令体验这种模式

```
git clone git@github.com:fatsheep9146/awesome-observability.git
cd awesome-observability/demos/otel-to-prometheus/demo-2
docker compose up --no-build
```

## 以 OpenTelemetry Collector 作为中继导入 Prometheus

采用 OpenTelemetry Collector 作为中继，我们就需要考虑使用什么种类的 receiver 和 exporter。

### receiver 选择

针对不同的数据透出需求，我们需要采用不同的 receiver 做支持，

- 针对 otlp 格式数据，需要采用 otlp receiver
- 针对 prometheus 格式数据，需要采用 prometheus receiver

### exporter 选择

无论通过 otlp receiver 还是 prometheus receiver，metric 在 collector 内部都会被转换为 otlp 格式，理论上都可以对接任意一个支持 metric 数据类型的 exporter。

那都有哪些 exporter 可以最终将数据输出到 prometheus 中呢？目前官方有如下几种
- prometheus exporter
- prometheus remote write exporter
- otlp exporter

所以我们可以根据自己的诉求，随意组合 receiver 和 exporter，并且也可以根据需求随意在中间加上满足自己需求的 processor，对数据进行二次处理。

下面我为大家提供一个全面的 demo 样例，其中包括 2 个 app，2 种 receiver 和 3 种 exporter，并且 3 种 exporter 都采用一个 prometheus 作为 backend 存储数据。并且为了方便区分，我们为每种模式都加了独有的 label，最终可以在查询时，用于区分。

