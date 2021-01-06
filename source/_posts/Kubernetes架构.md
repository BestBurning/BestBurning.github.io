---
title: Kubernetes架构
comments: true
tags: 
- Kubernetes
categories: 
- technology
keywords: kubernetes,架构
toc: true
date: 2020-12-08 15:35:05
---

# 概述

> Kubernetes 是一个开源的容器编排引擎，用来对容器化应用进行自动化部署、扩缩和管理

K8s → 使容器化应用部署更加简洁和高效

## 特性

- 自动装箱
- 自动修复
- 水平扩展
- 滚动更新
- 版本回退
- 秘钥和配置管理
- 存储编排
- 批处理

# 架构

![](https://images.di1shuai.com/kubernetes.jpg)

分布式架构中的 Master - Node 模式

## Control Plane Components 控制平面组件


控制平面的组件对集群做全局决策

- 调度
- 检测和响应集群事件（例如，当不满足部署的 `replicas` 字段时，启动新的 [`pod`](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/)）。

控制平面组件实际上也可以在任何节点上启动，也就是说可以在 `Node`节点上启动

### **kube-apiserver**

集群统一入口

### **etcd**

存储系统

### **kube-scheduler**

节点调度

调度决策考虑的因素包括 Pod 的

- 资源需求
- 硬件/软件/策略约束
- 亲和性和反亲和性规范
- 数据位置
- 工作负载间的干扰
- 最后时限

### **kube-controller-manager**

处理集群的后台任务

控制器包括:

- 节点控制器（Node Controller）: 负责在节点出现故障时进行通知和响应。
- 副本控制器（Replication Controller）: 负责为系统中的每个副本控制器对象维护正确数量的 Pod。
- 端点控制器（Endpoints Controller）: 填充端点(Endpoints)对象(即加入 Service 与 Pod)。
- 服务帐户和令牌控制器（Service Account & Token Controllers）: 为新的命名空间创建默认帐户和 API 访问令牌.

### **cloud-controller-manager**

嵌入特定云的控制逻辑入口，链接聚合到云提供商的应用编程接口中， 并分离出相互作用的组件与我们的集群交互的组件

下面的控制器都包含对云平台驱动的依赖：

- 节点控制器（Node Controller）: 用于在节点终止响应后检查云提供商以确定节点是否已被删除
- 路由控制器（Route Controller）: 用于在底层云基础架构中设置路由
- 服务控制器（Service Controller）: 用于创建、更新和删除云提供商负载均衡器

## **Node 组件**

Node组件在每个节点上运行，维护运行的 Pod 并提供 Kubernetes 运行环境。

### **kubelet**

一个在集群中每个节点上运行的代理。 它保证容器都运行在 Pod 中。

kubelet 接收一组通过各类机制提供给它的 PodSpecs，确保这些 PodSpecs 中描述的容器处于运行状态且健康。 kubelet 不会管理不是由 Kubernetes 创建的容器。

### **kube-proxy**

[kube-proxy](https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kube-proxy/) 是集群中每个节点上运行的网络代理， 实现 Kubernetes [服务（Service）](https://kubernetes.io/zh/docs/concepts/services-networking/service/)概念的一部分。

kube-proxy 维护节点上的网络规则。这些网络规则允许从集群内部或外部的网络会话与 Pod 进行网络通信。

如果操作系统提供了数据包过滤层并可用的话，kube-proxy 会通过它来实现网络规则。否则， kube-proxy 仅转发流量本身。

### **容器运行时（Container Runtime）**

容器运行环境是负责运行容器的软件。

Kubernetes 支持多个容器运行环境: [Docker](https://kubernetes.io/zh/docs/reference/kubectl/docker-cli-to-kubectl/)、 [containerd](https://containerd.io/docs/)、[CRI-O](https://cri-o.io/#what-is-cri-o) 以及任何实现 [Kubernetes CRI (容器运行环境接口)](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md)。

## **Addons 插件**

插件使用 Kubernetes 资源（[DaemonSet](https://kubernetes.io/zh/docs/concepts/workloads/controllers/daemonset/)、 [Deployment](https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/)等）实现集群功能。 因为这些插件提供集群级别的功能，插件中命名空间域的资源属于 `kube-system` 命名空间。

下面描述众多插件中的几种。有关可用插件的完整列表，请参见 [插件（Addons）](https://kubernetes.io/zh/docs/concepts/cluster-administration/addons/)。

### **DNS**

尽管其他插件都并非严格意义上的必需组件，但几乎所有 Kubernetes 集群都应该 有[集群 DNS](https://kubernetes.io/zh/docs/concepts/services-networking/dns-pod-service/)， 因为很多示例都需要 DNS 服务。

集群 DNS 是一个 DNS 服务器，和环境中的其他 DNS 服务器一起工作，它为 Kubernetes 服务提供 DNS 记录。

Kubernetes 启动的容器自动将此 DNS 服务器包含在其 DNS 搜索列表中。

### **Web 界面（仪表盘）**

[Dashboard](https://kubernetes.io/zh/docs/tasks/access-application-cluster/web-ui-dashboard/) 是Kubernetes 集群的通用的、基于 Web 的用户界面。 它使用户可以管理集群中运行的应用程序以及集群本身并进行故障排除。

### **容器资源监控**

[容器资源监控](https://kubernetes.io/zh/docs/tasks/debug-application-cluster/resource-usage-monitoring/) 将关于容器的一些常见的时间序列度量值保存到一个集中的数据库中，并提供用于浏览这些数据的界面。

### **集群层面日志**

[集群层面日志](https://kubernetes.io/zh/docs/concepts/cluster-administration/logging/) 机制负责将容器的日志数据 保存到一个集中的日志存储中，该存储能够提供搜索和浏览接口。

# 参考

[Kubernetes 组件](https://kubernetes.io/zh/docs/concepts/overview/components/#control-plane-components)