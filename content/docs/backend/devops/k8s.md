---
weight: 3
title: k8s
---

# kubernetes

官网：[https://kubernetes.io/zh-cn](https://kubernetes.io/zh-cn)

## K8s组件

- Control Plane：控制平面组件，为集群做全局决策。一般部署在同一台独立的服务器上。
  - kube-apiserver：公开Kubernetes API并接受请求。
  - etcd：保存k8s所有集群元数据的键值数据库。
  - kube-scheduler：监视并调度Pods。
  - kube-controller-manager：负责运行Controller进程。
  - cloud-controller-manager：连接集群到云提供商的 API。
- Node：工作节点，可以是虚拟机或者物理机。生产级别的 Kubernetes 集群至少应具有三个 Node。
  - kubelet：负责管理Node和由k8s创建的容器，并通过Kubernetes API 与 Master 通信。
  - kube-proxy：Node上的网络代理，维护节点上的一些网络规则。
- Master：管理集群，协调集群中的所有活动，如调度应用、维护状态、扩容等。
- Deployment：创建和管理Pod。Deployment会持续监视Pod，当Pod所在在Node出现问题后，将Pod转移到其它Node上（HA）。
  - ReplicaSet ：确保任何时间都有指定数量的 Pod 副本在运行。Deployment 管理 ReplicaSet。
  - Pod：Pod为一组共享资源的容器。这些资源包括：共享存储，网络和有关每个容器如何运行的信息。Pod 中的容器共享 IP 地址和端口，始终位于同一位置并且共同调度，并在同一工作节点上的共享上下文中运行。
- Service：抽象层，定义了 Pod 的逻辑集和访问 Pod 的协议。 Service通过LabelSelector来匹配Pod。