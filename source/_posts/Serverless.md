---
title: "Serverless技术相关梳理"
date: 2017-12-14 17:46:18
categories: "技术" 
tags:
  - serverless 
type: "serverless"
---

在Serverless架构中，应用业务逻辑将基于FaaS（function as a service）结构形成多个相互独立的功能组件，并以API服务的形式向外提供服务；
serverless让开发者无需关注资源的获取和运维，按需调用付费,节省成本；

注：FaaS属于serverless的子集，也是实现serverless化的核心服务；FaaS关键特征：事件驱动、细粒度调用、实现弹性伸缩、无需管理服务器等底层资源；

<!--more-->

---

### 一、Serverless 优势：

1. 节约使用成本：根据调用次数计费，节省成本，业务高峰弹性扩容，将资源分享给其他客户；
2. 简化设备运维：底层硬件透明化，无需运维；
3. 提升可维护性：serverless架构采用多种第三方功能服务模块组合，结构清晰，维护性高；

### 二、Serverless 行业场景：

1. 低频请求场景：（物联网行业）
物联网中，设备传输数据量小，且往往数据传输频率固定，
所以采购serverless低频的资源即可满足计算需求；

2. 流量突发场景：（移动互联网行业）
互联网中，经常会遇到突发流量场景，如电商网站促销活动期间流量爆增，但平时的流量会回到均值，传统架构需要扩展硬件能力以支持突发情况，但平时这些资源会被浪费，采用serverless的弹性扩展，可以在高峰时扩容，高峰后释放不用的资源，节省成本；

3. 基于事件的数据处理：
在图片视频处理后端进行视频转码、图片处理等等任务时，可结合对象存储的触发器，执行severless的事件处理函数进行对应任务的处理；

### 三、Severless 与 微服务：

两者都强调系统的解耦，微服务是以业务的粒度进行拆分，而serverless则是以函数function的粒度进行拆分，serverless的粒度更加细化；

### 四、serverless 与 运维：

Serverless并不等于没有运维，只是运维的工作由机器设备运维转变为业务运维，运维不再关心过去的配置、操作系统、内核网络等问题，而去关心系统业务的合理性、业务间的依赖关系、业务的报警监控等，serverless将推动 DevOps 的发展；

### 五、Serverless 业内竞品:

1. 亚马逊 AWS Lambda
2. 微软 Azure Function
3. 谷歌 Google Cloud Functions
4. 阿里云 Function Compute

### 六、基于 serverless 的程序框架

* https://github.com/serverless/serverless

### 七、阿里云 Serverless 生态：

目前阿里云围绕着 Serverless 推出的平台服务：

#### 1. 函数计算(function computer)

事件驱动的全托管计算服务，
* 无需管理服务器基础设施：无需考虑服务器规格、配置、升级、宕机、负载均衡、等等因素；
* 弹性伸缩，按需付费
* 以事件驱动方式与对象存储、api网关无缝链接；


#### 2. 对象存储 (Object Storage Service)

* 图片、音视频、日志等海量文件存储；
* 网页或移动应用静态文件的分离；
* 可对OSS云端数据进行媒体转码或图片处理；
* 对象存储触发函数计算事件处理函数：

https://help.aliyun.com/document_detail/53097.html?spm=5176.doc51784.6.553.nvicSg


#### 3. API网关 (API Gateway)

* API 录入维护统一管理
* 按实际调用付费
* 采用分布式部署，自动扩展，能够满足大规模访问场景
* 提供权限管理、流量管控、访问监控、安全预警、计量统计等功能

* 以函数计算作为Api网关的后端服务:

流程：`创建事件处理函数 -> 将函数授权给API网关 -> 配置API`

https://help.aliyun.com/document_detail/54788.html?spm=5176.product29462.6.584.7YpH2m

#### 4. 表格存储 (Table Store)

一种NoSQL数据存储服务，提供海量结构化数据存储和实时访问；
特性：
* 表格存储以`实例`和`表`的形式组织数据，通过分片和负载均衡实现规模的无缝扩展；
* 屏蔽底层硬件故障及错误，可快速恢复，提供高可用性；
* 数据被存储在SSD中且有多个备份，提供快速访问性能及可靠性；

* 通过表格存储触发器，可实现table store stream 和函数计算的自动对接：

流程：`创建开通Stream流的数据表 -> 创建函数计算事件处理函数 ->在数据表下创建触发器`

https://help.aliyun.com/document_detail/60304.html?spm=5176.doc60304.6.593.IoKlsE

#### 5. 日志服务 (Log Service)
提供日志类数据的采集、消费、投递及查询分析等功能

* 函数计算访问日志服务：
https://help.aliyun.com/document_detail/61023.html?spm=5176.doc60984.6.558.fjRIfN

#### 6. 批量计算 (BatchCompute)

一种适用于大规模并行批处理作业的分布式云服务。可支持海量作业并发规模，系统自动完成资源管理、作业调度和数据加载，并按实际使用量计费；

* 批量计算采用对象存储服务(OSS)作为持久化存储，同时批量计算可以通过文件接口访问OSS中的资源；
* 批量计算的程序默认运行于VM中，也支持采用Docker容器；
* 批量计算中提交的作业(Job)包含多个任务(Task);

<!--7. MQTT 消息队列遥测传输-->

<!--8. RAM 访问控制（资源访问管理）-->


### 八、函数计算无状态服务实现有状态逻辑

#### 大体思路：
1. 在函数计算初始化阶段在其本地启动一个`Express`服务；
2. 创建一个函数计算事件处理函数；
3. 前置服务`API Gateway`将关联 该函数计算事件处理函数；
4. 当前端或移动端发送请求给`Api`网关时，网关会将请求以事件触发给函数计算；
5. 此时函数计算的事件处理函数接收到了`event`事件数据源；
6. 函数计算对此事件数据源进行解析，需要解析出`URL`、`Header`、`Method`、`queryParams`、`body`等请求信息，并向本地`Express`服务发送该`http`请求；
7. 函数计算中启动的`Express`服务，处理完请求后返回结果，事件处理函数会对该结果进行一次输出格式的封装，以便`Api`网关能够识别，最终结果通过callback返回给Api网关；

#### 代码实例：
```
'use strict'
require(__dirname + "/bin/www");    // 初始化阶段在本地启动Express服务
const request = require('request');
const Promise = require('bulebird');
const httpRequest = Promise.promisify(request);

// 函数计算 事件处理函数入口
module.exports.handler = (event, ctx, cb) => {
    // 1. 入参判空等校验容错处理
    ...
    
    // 2. 获取事件请求参数
    let options = {
        url: event.url,
        headers: event.headers,
        method: event.httpMethod,
        qs: event.queryParams,
        body: event.body
        ...
    };
    
    // 3. 发送请求给本地Express服务
    return httpRequest(options)
    .then(res => {
        ...
        // 4. 按照API网关识别格式返回结果
        cb(null, {
            statusCode: res.statusCode,
            isBase64Encoded: isBase64Encoded,
            headers: res.headers,
            body: res.body 
        });
    })
    .catch(err => {
        ...
        cb(null, {
            statusCode: 500,
            isBase64Encoded: false,
            headers: {duration: duration},
            body: JSON.stringify(err)
        });
    });
    
};

```

---

<!--1. 目前的程序使用到哪些资源（包括即将用到的）-->

<!--2. 这些资源分别起什么作用-->

<!--3. 相互如何配合-->

<!--4. 最终如何实现在没有任何固定服务器资源的情况下，通常需要买服务器和带宽等才能实现的效果-->