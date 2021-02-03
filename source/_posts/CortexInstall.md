---
title: "Cortex安装操作笔记"
date: 2020-07-02 20:29:17
categories: "技术" 
tags: 
  - kube
  - cortex
type: "tags"
---


这篇安装 cortex, 使用的是官方提供的 helm 包: [cortex-helm-chart](https://github.com/cortexproject/cortex-helm-chart), 大致的安装步骤记录在此;

<!--more-->

#### 安装 Cassandra 数据库

cortex 需要一个存储后端, 这里选用:
``` sh
docker run -d --name cassandra -v /data/cassandra/:/var/lib/cassandra --rm -p 9042:9042 cassandra:3.11
```

进入容器内 Cassandra 查询语言(CQL) shell 环境:
``` sh
docker exec -it <container_id> cqlsh

```

为 Cortex metrics创建一个新的 Cassandra Keyspace:

``` sh
CREATE KEYSPACE cortex WITH replication = {'class':'SimpleStrategy', 'replication_factor' : 1};

```

因为存储后端采用本地部署, 且cortex数据量较大, 这里存在一个定期清理数据的问题, 这个有机会再议;


#### 安装 Consul 键值存储

cortex 需要一个外部的 key/value 存储, 官方支持的有`etcd` 以及 `consul`;

这里安装由于集群环境是 k3s, 所以没法复用 etcd, 就使用了 consul;

另外官网  [readme.key-value-store](https://github.com/cortexproject/cortex-helm-chart#key-value-store) 中也给出了helm的安装方式, 如果有兴趣可以去捣腾一下, 这里用 docker run 的方式简单搭建了一下:

```
docker run -d -p 8500:8500 -p 8600:8600/udp -v /data/consul/:/consul/data/ --name=badger consul agent -server -ui -node=server-1 -bootstrap-expect=1 -client=0.0.0.0
```

#### 安装 Cortex 

使用 helm 的方式安装, 由于默认的 values.yaml 并不满足配置, 所以需要自定义一些配置;

注意 consul 及 cassandra 配置;

custom.yaml
``` yaml
image:
  repository: quay.io/cortexproject/cortex
  tag: v1.6.0
  pullPolicy: IfNotPresent
config:
  ingester:
    max_transfer_retries: 0
    lifecycler:
      join_after: 0s
      final_sleep: 0s
      num_tokens: 512
      ring:
        replication_factor: 1
        kvstore:
          store: consul
          prefix: 'collectors/'
          consul:
            host: '192.168.0.7:8500'
            http_client_timeout: '20s'
            consistent_reads: true
  limits:
    ingestion_rate: 100000
    ingestion_burst_size: 500000
    max_series_per_metric: 500000
    enforce_metric_name: false
    reject_old_samples: true
    reject_old_samples_max_age: 168h
  schema:
    configs:
      - from: 2021-01-14
        store: cassandra
        object_store: cassandra
        schema: v10
        index:
          prefix: index_
          period: 168h
        chunks:
          prefix: chunks_
          period: 168h
  server:
    http_listen_port: 8080
    grpc_listen_port: 9095
    grpc_server_max_recv_msg_size: 104857600
    grpc_server_max_send_msg_size: 104857600
    grpc_server_max_concurrent_streams: 1000
  storage:
    cassandra:
      addresses: 192.168.0.8
      port: 9042
      keyspace: cortex
      consistency: "ONE"
ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: nginx
  hosts:
    - host: q5.wise2c.com
      paths:
        - /

```

接下来使用 helm 安装 cortex:

``` bash

# 添加源
$ helm repo add cortex-helm https://cortexproject.github.io/cortex-helm-chart

# 创建命名空间
$ kubectl create ns cortex

# 安装 cortex
$ helm install cortex --namespace cortex -f custom.yaml cortex-helm/cortex

```

到此 cortex 就部署完毕, 等待 pods 正常启动即可;

#### 关联 Prometheus 指标

这一步, 需要将 prometheus 的所有指标发送到 cortex;

prometheus 是采用 push 的方式将指标存入 cortex 的, 所以需要对 prometheus 配置 cortex 的 push 地址;

prometheus 的配置一般是存放在对应的 configmap 当中, 所以做如下操作:

``` sh
# 编辑 prometheus config
$ kubectl edit cm prometheus-config
```
添加 push 地址:
``` yaml
    remote_write:
      - url: http://192.168.0.3:30080/api/prom/push
```

注: 这里的端口 30080 是 service/cortex 的 NodePort;

配置添加完成, 重启一次 prometheus pod 即可使得配置生效;

此时访问: `http://${cluster_node_public_ip}:30080/all_user_stats` 就可以看到 cortex ingrester 指标采集的状态了;

#### Grafana 可视化

首先是数据源的添加:

* 进入: Configuration -> Data Sources -> Add data source 选择 Prometheus 插件, 没错 cortex 使用的是 Prometheus 数据源插件;
* HTTP URL 填写: `http://${cluster_node_ip}:30080/api/prom/`;
* 保存并测试;

检查指标数据:

* 进入: Explore -> 选择 cortex 数据源 -> 点击 Metrics 选择任意指标 -> 点击 Run Query;
* 查看是否有数据产生;

仪表盘图表:

* 进入: Configuration -> Data Sources -> Cortex data source -> Dashboards -> 点击 import;
* 进入: DashBoards -> Manage -> 任选一个dashboard;
* 查看图表展示;


---

* [cortex docs](https://cortexmetrics.io/docs/)
* [cortex helm](https://github.com/cortexproject/cortex-helm-chart)
