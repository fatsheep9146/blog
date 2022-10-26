---
title: "Prometheus Histogram"
date: 2022-10-06T20:07:47+08:00
draft: true
---

# histogram 简介

A histogram samples observations (usually things like request durations or response sizes) and counts them in configurable buckets. It also provides a sum of all observed values.

A histogram with a base metric name of <basename> exposes multiple time series during a scrape:

- cumulative counters for the observation buckets, exposed as <basename>_bucket{le="<upper inclusive bound>"}
- the total sum of all observed values, exposed as <basename>_sum
- the count of events that have been observed, exposed as <basename>_count (identical to <basename>_bucket{le="+Inf"} above)

# histogram 基本格式

TBD 对 golang client 代码理解一下，看如何生产 histogram 类型的原始数据

client_golang/prometheus/examples_test.go

# histogram 的常用功能

- 计算平均值[1]

## 计算平均值

To calculate the average request duration during the last 5 minutes from a histogram or summary called http_request_duration_seconds, use the following expression[1]:
```
  rate(http_request_duration_seconds_sum[5m])
/
  rate(http_request_duration_seconds_count[5m])

```

## quantiles

Using histograms, the aggregation is perfectly possible with the histogram_quantile() function.
```
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le)) // GOOD.
```

TBD 理解 histogram_quantile function 的内部逻辑

# 新版 native histogram [3]

## 老版本 histogram 的弊端

## 新版本实现思路

# Ref
1. https://prometheus.io/docs/practices/histograms/
2. Apdex Score https://en.wikipedia.org/wiki/Apdex
3. https://github.com/prometheus/prometheus/milestone/10