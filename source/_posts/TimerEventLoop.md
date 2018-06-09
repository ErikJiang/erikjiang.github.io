---
title: "NodeJS定时器与事件循环"
date: 2018-06-09 15:53:51
categories: "技术" 
tags:
  - timer
  - eventloop
  - nodejs
type: "tags"
---

> 一直以为事件循环是一个独立于NodeJS主线程的单独工作线程，其实并非如此；
> 事件循环是在NodeJS主线程中完成的；

<!--more-->

---


### NodeJS提供四种定时器

为了协调异步任务，使任务按时间顺序执行，NodeJS提供了四种定时器；

* setTimeout()
* setInterval()
* setImmediate()
* process.nextTick()

其中`setTimeout`、`setInterval`存在于JS语言标准，
而`setImmediate`、`process.nextTick`则为NodeJS独有；


### 任务的执行顺序

> * `同步`任务先于`异步`任务执行
> * `本轮`循环先于`次轮`循环执行


### 本轮事件循环
 
本轮循环中存在两个队列：

1. nextTick队列(nextTickQueue)
2. 微任务队列(microTaskQueue)

在本轮循环中`process.nextTick`是优先执行的，所有的`nextTick`定时器执行完成后，
才轮到微任务队列定时器执行，常见的`Promise`就属于微任务定时器；


### 次轮事件循环

```flow
op0=>operation: Timer
op1=>operation: I/O callbacks
op2=>operation: Idle, Prepare
op3=>operation: Poll
op4=>operation: Check
op5=>operation: Close callbacks
op0->op1->op1->op2->op3->op4->op5
```

事件循环的六个阶段：

1. `timers阶段`: 处理setTimeout、setInterval的回调函数
2. `i/o callback阶段`: 处理除setTimeout、setInterval、setImmediate、close callback以外的回调函数
3. `idle、prepare阶段`: 供LibUV内部使用
4. `poll阶段`: 等待未返回的IO事件，直到返回结果
5. `check阶段`: 处理setImmediate的回调函数
6. `close callback阶段`: 处理关闭事件的回调，如socket.on('close')

---

* 参考：[定时器详解](http://www.ruanyifeng.com/blog/2018/02/node-event-loop.html)

