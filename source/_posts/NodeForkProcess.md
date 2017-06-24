---
title: "Node 多进程初探"
date: 2017-05-12 22:49:18
categories: "技术"
tags:
  - node
  - fork
  - process
type: "tags"
---

前段时间在使用 Node 实现一个系统的定时任务功能，定时任务数量较多，且其中存在一些需要 CPU 计算的任务，一开始这些任务被放置在 Node 单线程中运行，可是随着任务数的增长，单一线程渐渐吃不消，任务运行高发时段，CPU 会飙至百分之九十以上，同时造成 Web 页面查询请求卡顿甚者请求无响应；

<!--more-->

## 问题分析
众所周知，Node.JS 的事件循环对于处理 I/O 密集型任务十分擅长，但是当遇到 CPU 密集型任务，由于事件循环是按照队列顺序执行，若队列中任何一个事件任务没有被完成，那么其他待进入队列的事件任务将不被执行，换句话说，CPU 密集型任务的耗时操作阻塞了事件队列后，将会使得事件任务的处理效率下降，严重时会使得事件循环停滞不动；

## 处理方案

Node 作为一种的单线程语言，先天的无法发挥服务器多核 CPU 的性能，而要处理上述问题，一般有两种方式：
1. 多进程: `child_process`、 `cluster`
2. 多线程: `threads-a-gogo`

作为初探，先来讲讲多进程的通用方案 `child_process`，我们这里通过 fork 的方式创建一个独立负责定时任务处理的子进程，这样一来将任务处理的逻辑从主进程中分离，然后主进程要做的只是运用 IPC 进程间通讯管理分发要执行的任务给子进程；

### 目录概述
项目大致目录结构如下：
```
.
├── app.js
├── modules
│   └── childProcess.js
├── swagger
│   └── controller.js
├── usecase
│   └── worker
│       └── index.js
├── datasource
│   ├── worker
│   │   ├── task1Handler.js
│   │   ├── task2Handler.js
│   │   ├── task3Handler.js
│   │   ├── task4Handler.js
│   │   └── task5Handler.js
│   └──     

```

### 子进程的创建
* 创建工作子进程，并将子进程实例带入对应入口的 req 属性，便于获取；
```js
    // modules/childProcess.js
    var child_process = require('child_process');
    var forkChild = child_process.fork(process.cwd() + '/usecase/task/wokerIndex.js');
    module.exports = function(req, res, next) {
        var urlPath = /^\/api\/v\d+\/subProcessHandler/;
        if (urlPath.test(req.path) && (req.method === 'POST')) {
            req.childProcess = forkChild;
        }
        return next();
    };
```
### 中间件添加
* 将子进程创建的中间件加入到服务应用的实例中；
```js
    // app.js
    var childProcess = require(process.cwd() + '/modules/childProcess');
    app.use(childProcess);
```

### 应用层接口
* 任务处理入口，运用进程间通信将执行参数传入子进程；
```js
    // swagger/controller.js
    exports.postSubProcessHandler = (req, res, next) => {
        let childProcess = req.childProcess;
        childProcess.send(req.body);
        res.end();
    }
```
### 定时任务分发
* 工作子进程通过接收主进程执行参数，分发执行任务；
```js
    // usecase/worker/index.js
    let workerTaskList = {
        taskType1: require('../../datasource/worker/task1Handler'),
        taskType2: require('../../datasource/worker/task2Handler'),
        taskType3: require('../../datasource/worker/task3Handler'),
        taskType4: require('../../datasource/worker/task4Handler'),
        taskType5: require('../../datasource/worker/task5Handler')
    };

    process.on('message', msg => {
        workerTaskList[msg.taskType](msg.params);
    });
```

## 多进程的集群管理
采用 fork 单独子进程处理任务，以上述的方式即可满足，可若要充分利用 CPU 多核性能资源，就需要 fork 多个工作进程，但是子进程数量变多，就存在诸如进程协调管理、负载均衡等问题；

此时就需要考虑使用 [Node Cluster](https://nodejs.org/dist/latest-v7.x/docs/api/cluster.html) 模块，该模块为 Node 自身内置模块，用于构建多进程并行化程序，实现用于负载均衡的集群；

当然使用基于 Cluster 模块的 [PM2](http://pm2.keymetrics.io/) 集群控制工具也是一种很好的选择；

关于 Cluster 与 PM2 ，待后续文章有机会讲解，本文就此告一段落，如有不妥之处，望指出，感谢。


