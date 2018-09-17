title: "NodeJS问题集萃"
date: 2018-02-15 20:45:17
categories: "技术" 
tags: 
  - node
type: "tags"

---

这篇放上一些关于Node.JS的问题总结，有助于对NodeJS加深理解。

---

### node modules 中，exports 及 module.exports 的区别？
需要注意，模块导出返回的永远是`module.exports`
#### (1) 两者关系：
> exports 是 module.exports 的引用

#### (2) 何时使用 exports 何时使用 module.exports:
* module.exports 一般用于导出指定的对象类型:
如`module.exports = new Gift();`
* exports 一般导出的是模块实例:
如`exports.PI = 3.1415;`

---

### 描述一下 require 一个 module 的过程？
1. 检查 module._cacahe 是否已经缓存了模块.
2. 如果没有缓存，则创建一个新的模块实例并且存入缓存.
3. 根据所给的模块名调用 module.load() 方法，load 完毕后会调用 module.compile() 方法进行编译.
4. 如果装载/编译过程中出现了错误，则会抛出错误并且将错误的模块从缓存中移除.
5. 最后返回所依赖的模块: module.exports

---

### 哪些后缀扩展名的文件会被自动识别并require ？
require 默认会加载.js、.json、.node 格式的文件

---

### 解释一下 同步和异步 以及 阻塞与非阻塞？

* 同步异步关注的是: `消息通信机制` (发生调用是否立即返回)
`同步`即在发生调用时，在没有得到结果时，该调用不会返回，待得到结果后，调用才会返回；
`异步`即在发生调用时，在没有得到结果时，该调用直接返回，待得到结果后，通过状态、消息通知调用者，或通过回调函数处理该调用；

* 阻塞与非阻塞关注的是: `程序等待调用结果时的状态` （当前线程是否被挂起）
`阻塞`即指调用结果返回前，当前线程会被挂起，直到得到结果返回；
`非阻塞`即指调用结果返回前，该调用不会阻塞挂起当前线程；

* [怎样理解阻塞非阻塞与同步异步的区别](https://www.zhihu.com/question/19732473/answer/20851256)

---

### 什么是C10K问题，聊聊它的技术变革及演进方向？
`C10K`即单机1万个并发连接情况下，硬件性能足够，但依然无法提供正常服务的问题；
`C10K`问题是计算机从PC时代变迁到互联网时代的过程中因并发量剧增而逐渐出现问题；
`C10K问题特点`: 并发性能与硬件性能成非线性
`解决C10K问题的关键`: 尽可能减少CPU等核心计算资源消耗，防止榨干单台服务器的性能；
#### (1) 技术变革与演进方向
##### 1. 每个进程/线程处理一个连接

##### 2. 每个进程/线程处理多个连接（即：IO多路复用）

###### 2.1 Select 方案
同时监控多个文件句柄，逐个排查IO句柄状态，若准备好了就处理
缺点：有句柄上限、重复初始化、逐个排查所有文件句柄效率低；

###### 2.2 Poll 方案
Poll 方案在 Select 方案基础上解决了`句柄上线`和`重复初始化`的问题，效率有所提高；
缺点：仍然存在逐个排查所有文件句柄的问题，在句柄变多后，处理效率会变差；

###### 2.3 Epoll 方案
Epoll 方案采用仅排查当前发生状态变化的文件句柄的设计，使得排查句柄的效率得到提升；
Nginx、libevent、libev、NodeJS的libuv 都是基于Epoll方案；
缺点：不能跨平台，Linux 使用 epoll, FreeDSD 使用 kqueue, windows 使用 IOCP;

###### 2.4 Libevent库
考虑到跨平台和移植，libevent 库将不同平台的 epoll、kqueue、IOCP 等进行了封装整合；

##### 3. 采用协程的方式处理
使用进程/线程的方式处理并发请求时，在进行系统调度时切换系统上下文的代价是很高的，且进程/线程作为处理单元太厚重，于是诞生了一种轻量级的处理单元，协程；
如 Python、Lua 提供的 coroutine 模型，Go 提供的 goroutine 模型；
特点：占用资源少、避免系统调度过程中的上下文切换、用户代码实现无需内核参与；

[C10K 问题引发的技术变革](https://blog.csdn.net/yeasy/article/details/43152115)
[事件驱动与协程：基本概念介绍](https://zhuanlan.zhihu.com/p/31410589)
[上一个10年，著名的C10K并发连接问题](https://segmentfault.com/a/1190000007240744)

---

### 什么是libuv? 在NodeJS中的作用是什么？
`libuv`是一个高性能的事件驱动IO库，它封装了比`libevent`设计简练且性能更好的`libev`，
但是由于`libev`对于windows平台支持有限，所以libuv又在库中加入了windows的IOCP，以支持跨平台；
libuv 提供了非阻塞网络请求、各平台的事件轮询方法等；

libuv 是 NodeJS 实现事件循环 Event-Loop 的关键；

---

### 什么是事件驱动？什么是事件循环？

#### (1) 事件驱动
事件驱动指持续事务管理过程中决策的一种策略，即跟随当前时间点上出现的事件，调动可用资源，执行相关任务，使不断出现的问题得以解决，防止事务堆积;

事件驱动并非NodeJS独有的功能，它是作为一概念引入到NodeJS;

NodeJS主线程执行时，会启动事件循环（Event Loop）进行轮询监听和捕获事件，当一个IO事件触发时，会将其放入事件循环队列，同时会立即返回主线程而不会阻塞，这样保证了主线程可以处理其他事件而不会等待，此时，事件循环里的事件等待执行处理结果，若得到结果，则会调用相应的回调函数；

NodeJS的事件循环是有libuv处理的；

---

### 什么是调用栈？
调用堆栈是JS代码执行的基本机制。

当调用一个函数时，会将函数参数和返回值地址推送到堆栈。

这样允许运行时知道函数结束后继续执行代码的位置。

NodeJS的调用栈有V8处理；

由于调用栈的栈采用的是`后进先出`的方式，
故在语法解析完毕后，主函数首先被push到栈底，文件头的首个函数被存入栈顶；
而在执行过程中，则先执行栈顶的函数，然后依次执行后续函数，直到执行到最后的主函数；

---

### setImmediate 与 process.nextTick 的关系？
首先，在NodeJS中事件执行的顺序是，同步任务先于异步任务执行，异步中本轮循环先于次轮循环执行；

本轮循环的异步任务按先后阶段顺序包括两个队列：
1. nextTickQueue (如：process.nexTick())
2. microTaskQueue (如：Promise)

次轮循环的阶段顺序：
1. timers (setTimeout() setInterval())
2. I/O callback (其他类型的回调)
3. idle (libuv内部调用)
4. poll (等待没有返回的IO事件)
5. check (setImmdiate())
6. close callbacks (close关闭事件的回调)

setImmediate()在此轮循环的check阶段，而process.nextTick()位于本轮循环的nextTickQueue当中；

process.nextTick()先于setImmediate()执行；

---

### child_process 模块的 spawn、exec、fork主要区别是什么？
`child_process.spawn()`: 异步的衍生子进程，不会阻塞NodeJS事件循环；
`child_process.spawnSync()`: 同步的衍生子进程，会阻塞事件循环，直到子进程退出终止；
`child_process.exec()`: 衍生一个shell并执行命令，完成时会输出stdout和stderr到回调函数；
`child_process.fork()`: 衍生一个NodeJS进程，并通过IPC通道实现父子进程通信；

---

### cluster 模块的工作原理？
由于NodeJS是单进程的，无法充分利用多核，因此需要启动多个进程满足高负载的处理需求；

cluster 即可以创建并管理多个子进程并共享端口；

Cluster 基于`child_process.fork()`实现工作进程的复制；

cluster 通过一个主进程接收请求或连接，并通过循环的方式分发请求给创建的多个子进程进行处理，从而做到简单的负载均衡；

---

### 如何查看NodeJS进程占用的内存？

程序外，通过进程内存查看工具
程序内，通过`process.memoryUsage()`查看

---

### 如何在NodeJS进程退出前做最后的操作，操作是否可以异步执行？
可以注册一个`exit`事件监听，通过`process.on('exit',cb)`,该操作不可异步；

---

### NodeJS依赖有哪些？

* 库依赖：v8、libuv、http-parser、c-ares、openSSL、zlib
* 工具依赖：npm、gyp、gtest

---

### 同样是实现异步操作，回调 callback 与事件 emitter 有什么区别?
emitter 更加灵活，可以将执行的过程发送到别的地方，只要有方法监听这个事件就能够收到；
callback 则只能在本函数或本文件内执行，相对比较固定；

---

### koa2的中间件原理

// todo

---

### 对websocket的理解，及其在NodeJS中的应用

// todo

---

