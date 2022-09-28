---
title: "003 Exemplar Research"
date: 2022-09-13T18:59:25+08:00
draft: true
---

# Exemplar 调研

# 什么是 exemplar，为什么要用 exemplar

在可观测领域，有三种经典数据类型

- metric
- trace
- log

metric 适合用来描述一个应用的整体表现；trace 则适合用于探究某次事务的完整流程。那么我们经常遇到的一个场景就是：某个 metric 发生了波动，代表这个系统可能产生了某种问题，但是我们通过 metric 上的信息，很难确认到底是哪些请求导致了这种波动，所以就需要再通过 trace 的系统去大海捞针，捞出那些异常请求再进行分析，非常低效。

所以为了解决这个问题，exemplar 诞生了。它相当于为 metric 和 trace 之间建立起了一座桥梁，使用户在遇到上面的问题时，可以直接一键跳转到导致异常波动的请求的 trace 详情中，大幅提升排障的速度和精度。

# 如何使用 exemplar

目前一个 golang 程序，想要使用 exemplar 能力，需要做如下几件事情

- 监控埋点改造，加入 exemplar 信息
- 暴露 openmetrics 协议监控数据 
- to collect the metrics & exemplars using Prometheus
- to configure Grafana to visualize the time series with the exemplars 

## 监控埋点改造，加入 exemplar 信息

1.4.0 以上版本的 prometheus golang library 专门为 exemplar feature 增加新的埋点函数，使用样例如下

```go
	requestDurationsHistogram = prometheus.NewHistogram(prometheus.HistogramOpts{
		Name:    "http_request_durations_histogram_seconds",
		Help:    "HTTP request latency distributions.",
		Buckets: []float64{.005, .01, .025, .05, .1, .25, .5, 1, 2.5, 5, 10, 25, 50, 100},
	})

    requestDurationsHistogram.(prometheus.ExemplarObserver).ObserveWithExemplar(float64(time.Since(starttime)), prometheus.Labels{
        "TraceID": traceId.String(),
    })

```

## 暴露 openmetrics 协议监控数据 

除了采用 exemplar 专用的埋点函数以外，还需要通过 prometheus client vendor 开启 openmetrics 协议的数据

```go
	prometheus.Register(requestDurationsHistogram)
	mux.Handle("/metrics", promhttp.HandlerFor(
		prometheus.DefaultGatherer,
		promhttp.HandlerOpts{
			EnableOpenMetrics: true,
		},
	))
```

通过下面的命令可以看到带有 exemplar 信息的数据
```
$ curl -H "Accept: application/openmetrics-text" 127.0.0.1:7777/metrics
http_request_durations_histogram_seconds_bucket{le="0.005"} 0
....
http_request_durations_histogram_seconds_bucket{le="50.0"} 0
http_request_durations_histogram_seconds_bucket{le="100.0"} 0
http_request_durations_histogram_seconds_bucket{le="+Inf"} 1 # {TraceID="4730c8f4b45465a501b2b42e0589b891"} 6.1782157e+07 1.664375433305092e+09
http_request_durations_histogram_seconds_sum 6.1782157e+07
```

# 参考
1. https://vbehar.medium.com/using-prometheus-exemplars-to-jump-from-metrics-to-traces-in-grafana-249e721d4192
2. 