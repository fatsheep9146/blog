---
title: "Operator SDK: Helm Operator 介绍"
date: 2022-12-13T18:59:25+08:00
draft: true 
tags: ["kubernetes", "operator", "operator framework", "operator sdk"]
---

### 摘要



### 什么是 helm？什么是 operator？

想必深入对 k8s 做过开发工作的同学都应该知道 helm 和 operator 这两个概念。

- helm：helm 可以帮助用户可以管理，安装，卸载，升级，在 kubernetes 上运行的复杂应用[1]。这种应用通常是一组 k8s 资源的集合，比如 1 个 deployment 加上一个 service。用户通过 charts 描述部署应用的模板，再通过 helm cli 以 chart 为输入安装具体的应用实例。

- operator：operator 是一种针对 kubernetes 进行扩展的编程模式，利用 kubernetes 提供的 custom resource 能力，管理运行在 kubernetes 中的复杂应用。用户在创建具体的 custom resource 资源时，operator 即可基于 custom resource 中的信息，构造具体的应用。

那么，helm 和 operator 各自适合什么场景呢？

helm 特别适合架构简单的无状态应用，或者是只需要考虑服务的安装和删除，或者简单升级操作的场景下去使用。比如部署一套 nginx 服务，牵涉到的就是一个 deployment 和 service 的组合。

operator 特别适合在需要更复杂的，更定制的运维操作的场景，比如 etcd 应用，在对一个 etcd 集群扩容时，不仅仅要创建一个 pod 这么简单，还要先访问当前 etcd 集群的 master，通过 http 请求，将这个新 pod 的基本配置信息通过命令登记到 etcd 集群后，才能去启动新的 pod，加入这个 etcd 集群。

### 什么是 helm operator

helm operator 是 operator sdk 。。。。

operator sdk 是 。。。

其中他对 operator 的周期也定义了 。。。

所以 helm operator 可以特别好的解决一个场景的问题。。早期，中期，后期

### 如何从 0 构造一个 helm operator


# 参考

1. https://helm.sh/
2. https://kubernetes.io/docs/concepts/extend-kubernetes/operator/
3. https://sdk.operatorframework.io/docs/overview/
4. https://sdk.operatorframework.io/docs/building-operators/helm/tutorial/
5. https://github.com/operator-framework/operator-sdk
6. https://www.techtarget.com/searchitoperations/tip/When-to-use-Kubernetes-operators-vs-Helm-charts
7. https://www.velotio.com/engineering-blog/getting-started-with-kubernetes-operators-helm-based-part-1