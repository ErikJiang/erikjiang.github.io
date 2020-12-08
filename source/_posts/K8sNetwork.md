---
title: "有关 Kubenetes 网络"
date: 2020-06-08 11:17:31
categories: "技术" 
tags: 
  - kube
type: "tags"
---

### K8s 四层网络模型

* 第0层 Node节点网络 (Node IP + Port): 保障节点主机互联互通
* 第1层 Pod网络 (Pod IP + Port): 保证Pod容器互联互通
* 第2层 Service网络 (Cluster IP + Port): 保证服务发现和负载均衡
* 第3层 外部接入网络 (NodePort / LoadBalancer / Ingress): 提供外部流量接入

<!--more-->

|          | 作用   |  实现  |
| --------   | :-----  | :---- |
| 节点网络 | Master/Worker 节点之间网络互通 | 路由器,交换机,网卡 |
| Pod 网络 | Pod 虚拟机容器之间互通 | 虚拟网卡,虚拟网桥,网卡,路由器or覆盖网络 |
| Service 网络 | 服务发现+负载均衡 | Kube-Proxy,Kubelet,Master,Kube-DNS |
| NodePort | 将Service暴露在节点网络上 | Kube-Proxy |
| LoadBalancer | 将服务暴露在公网上+负载均衡 | 公有云LB+NodePort |
| Ingress | 反向路由,安全,日志监控(类似反向代理或网关) | Nginx,Envoy,Traefik,Zuul,SpringCloudGateway... |


### 第一层: K8s Pod Network

#### 三种网络设备:

    1. veth0: 存在于 pod 容器内部, 支持pod单元内部多容器间网络互联互通, 每个 pod 单位中都存在一个 pause 容器, 起作用即为pod创建 veth0 设备;
    2. docker0: 存在于 node 节点内部, 作为虚拟网桥, 支持多个 pod 间网络互联互通
    3. eth0: 存在于 node节点上的真实设备, 提供节点互联互通访问

#### 不同节点上的 Pod 如何实现互联互通?

    1. 路由方案: 
        依赖于底层网络设备, 设置各节点路由转发策略, 性能开销较小;

    2. 覆盖网络(overlay)方案: 
        采用隧道封包技术, 将pod网络数据包在出节点之前封装成节点的网络数据包, 当数据包到达目标节点再解封数据包, 并转发到对应的Pod上;
        对于底层网络设备无任何依赖, 但是封包解包过程会有性能消耗;

由于不同节点Pod网络访问的技术方案众多, CNCF指定了统一的标准CNI, 简化及规范K8s不同节点Pod网络的技术实现;


### 第二层: K8s Service Network

Service 网络构建于 Pod 网络之上, 主要功能:
1. 服务发现 (Service Discovery)
2. 负载均衡 (Load Balancing)

#### K8s 服务发现原理
原理基于: DNS + Service Registry

概念:
1. K8s Master
2. Worker Kubelet
3. Worker Kube-Proxy 
4. Worker Kube-DNS

服务注册: 
* 首先 Kubelet 创建 Pod 实例, 并将 IP 等信息通过 Kubelet 注册到 K8s Master;

服务发现的两层映射:
* Kube-DNS 实时监听 K8s Master 变化, 获取 ServiceName->ClusterIP 映射列表;
* Kube-Proxy 实时监听 K8s Master 变化, 获取 ClusterIP->PodIP 映射列表;

客户端 Kube-Proxy + IPTables 转发机制, 无侵入不穿透;

当有消费者客户端请求服务时, 若是域名会到 Kube-DNS 获取对应的 ClusterIP, 然后再通过 Kube-Proxy 的 IPtables 进行负载均衡, 将请求由 ClusterIP 均衡分配到对应的 Pod 实例;


### 第三层: NodePort / LoadBalancer / Ingress

K8s 对外暴露服务的方式:

* Service NodePort
* Service LoadBalancer
* Ingress
* Kubectl Proxy & Port Forward

如何将 K8s 集群内部服务暴露到外部网络环境, 以便访问?

1. NodePort 方式: 
将 Service 设置为 NodePort 模式, 将会在各个节点的 Kube-Proxy 组件上开启一个外部访问端口, 当有请求访问此端口时, Kube-Proxy 会将请求转发给 Service, Service 再将请求转发给 Pod 实例;

2. LoadBalancer 方式:
首先 LoadBalancer 方式是基于 NodePort 方式实现的, 若将 Service 设置为 LoadBalancer 模式, 第一步会为每个节点分配一个 NodePort(即 NodePort 方式), 然后云平台会在此基础上前置一个LB负载均衡器并映射到后端的各节点端口上; (仅公有云云厂商提供此类访问方式), 缺点是每新增一个Service, 就需要为其新增一个LoadBalancer;

3. Ingress 方式:
可以理解成就是 K8s 集群中的反向代理服务, 比起 LoadBalancer 可以扩展更多服务, Ingress对内转发使用ClusterIP, 对外使用 HostPort 80/443 访问, 并可以前置一台 LB 负载均衡器;
其转发方式, 可以基于域名domainName, 或者基于访问地址URLPath;

4. 开发测试环境对外暴露: Kubectl Proxy & Port Forward
* Kubectl Proxy 代理方式直接访问集群的 API Server, 间接访问K8s各个服务, 限于七层的HTTP转发;
* Kubectl Port-Forwarding 在本地开启一个转发端口将请求间接转发到集群的某个Pod中去, 此方式为TCP转发;


### Kube-Proxy

kube-proxy 功能的实现主要依赖于 netfilter 与 iptables;

处于用户空间的 iptables 可以操作处于内核空间的 netfilter, 以截获底层网卡的网络包并修改其路由;

kube-proxy 三种工作模式:

1. Userspace Proxy Mode (基本淘汰)
由于要在用户空间kube-proxy中进行负载均衡, 需要频繁从内核空间netfilter切换到用户空间, 故上下文切换频繁, 性能较差;

2. IPTables Mode (适用于中小型生产环境)
在内核netfilter中负载均衡, 不会频繁切换上下文, 性能较好, 但是支持的负载均衡方式较少, 且只能适用于中小规模集群;

3. IPVS Mode (适用于大规模生产环境)
在内核ipvs中负载均衡, 负载均衡策略较多, 性能较iptables优秀, 但是配置比较复杂;

