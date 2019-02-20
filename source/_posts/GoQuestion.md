---
title: "Go问题集萃"
date: 2018-04-25 21:15:12
categories: "技术" 
tags: 
  - go
type: "tags"

---


### 语言相关

1. 谈谈 进程、线程、协程，以及它们出现所要解决的问题，及相关技术的演进历程？

* [C10K问题](http://erikjiang.github.io/2018/02/15/NodeQuestion/)

c10k主要指单机并发连接在1万的情况下，硬件性能足够，但仍然无法正常提供服务的问题
c10k是计算机从PC时代到互联网时代过程中由于并发连接量剧增做产生的问题；
c10k解决的关键在于，降低cpu等核心资源的消耗

IO多路复用的种类：

select: 存在句柄上线、重复初始化、轮询低效问题
poll：解决了句柄上线、重复初始化问题，但依然需要轮询IO句柄
epoll: 采用仅排查当前状态变化的句柄，无需逐个轮询句柄的方式，效率提升, 但不能跨平台，不同平台的库不同 epoll\kqueque\IOCP；
nginx, libevent, libev, nodejs底层的libuv都是基于epoll;


发展三阶段：
1). 一个进程/线程处理一个连接
2). 一个进程/线程处理多条连接（IO多路复用）
3). 用户态异步协程处理连接

进程：
是系统进行资源分配和调度的独立单元，拥有独立的内存空间，不同进程通过IPC进行通信，进程较重，上下文进程间切换（栈、寄存器、虚拟存储、文件句柄）开销大，但稳定性好；

线程：
是进程内部的实体，是CPU调度和分派的基本执行单元，它与同属一个进程内的其他线程共享进程所拥有的资源，线程间通信主要通过共享内存，相比进程资源开销小，上下文切换快，但较不稳定；

协程：
是一种用户态的轻量级线程，调度完全由用户程序控制，没有内核开销，效率快；

进程与线程：
* 地址空间：线程是进程内独立的执行单元，一个进程至少有一个线程，进程有独立的地址空间， 且进程内的线程共享进程的地址空间；
* 资源共享：进程内的资源与其内部的线程共享；
* 调度处理：线程是处理器CPU调度的基本单位，进程不是；
* 都可以并发执行；

线程与协程：
* 一个线程或进程可以拥有多个协程；
* 线程进程是同步机制，协程是异步机制；

https://www.cnblogs.com/lxmhhy/p/6041001.html

2. channel 要记得 close，但 close 时候需要注意些什么？
* channel 不能被重复close,否则会panic;
* 已经关闭的 channel 不能向其发送数据，否则painc;
* 从已关闭的 channel 中读取数据，如果是无缓冲的，读出的是零值，如果有缓冲且有数据，可以继续读；
* 通过`i, ok := <- ch`的 ok 获取channel是否已关闭；
* 应该只在唯一或最后唯一剩下的生产者发送端协程中关闭channel；

3. channel 是通过注册相关 goroutine id 实现消息通知，具体阐述？
// todo

4. slice 相关问题， slice 与数组的区别，slice的底层结构？

array 属于值类型，长度固定；
slice 属于引用类型，长度不定，slice的底层引用了一个数组对象；

声明时，array要指明长度，slice则不需要；
作为函数参数时，数组传递的时其副本，而slice传递的是指针；


5. 谈谈 互斥锁，读写锁，死锁，及其数据竞争的概念？

互斥锁: 是并发程序对共享资源进行唯一性访问控制的主要手段;
读写锁: 即是并发时针对于读写操作的互斥锁；多个写操作之间是互斥的；写操作与读操作之间是互斥的；多个读操作之间不存在互斥；
死锁: 并发时所有协程都彼此等待的状态（goroutine间存在无缓冲channel只读不写或只写不读的情况、加锁但没解锁的情况）
数据竞争：并发情况下存在的多个协程读写相同数据的情况，至少有一个协程是写操作；若多个协程都是读操作，则不存在数据竞争；

6. 什么是channel，为什么它可以做到线程安全？

是golang协程间通信的方式，channel本质是通过内存队列，一次只处理一个数据，从而实现了访问的序列化，避免了数据竞争，从而并发安全；

7. 如何用channel实现一个令牌桶

// todo

8. 如何写单元测试和基准测试

* 使用 gotest 包，文件名格式 `*_test.go`
* 导入 `import testing`，单元测试函数命名格式：`func Test*(t *testing.T){}`
* 导入 `import testing`，基准测试函数命名格式：`func Benchmark*(b *testing.B){}`
* `go test`执行单元测试，`go test -test.bench=.*`执行所有基准测试；

9. go 及 goroutine 的调度是怎样的（为什么并发性能好），抢占式goroutine调用什么意思？

1). 线程池的缺陷
线程池中的woker线程获取任务队列中的任务并执行，但若任务中发生系统调用，该worker进程将处于阻塞状态，从而任务队列任务堆积，
解决方法是增加线程数,但会使得多线程争抢CPU,从而使得处理能力下降；

2). G-P-M调度模型
* G: Goroutine Go协程
* M: Machine 系统工作线程
* P: Processor 上下文调度器
调度策列：
* 队列轮询：P维护着一个G的队列，P周期性将G队列中的G调度到M中执行，执行一会儿，保存上下文放入队尾，再执行队列中下一个G;
* 系统调用: 当M在执行G1的过程中遇到系统调用时，M会释放P以及P所维护的G队列，此时空闲的M1获取到P及剩余的G队列继续执行，M1接替M的剩余工作
* 工作量窃取：通常当两个P中的G队列不均衡时，空闲的P会从忙碌的P的G队列中窃取部分G来执行；

抢占式goroutine调用：如果一个goroutine运行时间过长或则长时间阻塞于系统调用，它就会被剥夺运行权；


https://segmentfault.com/a/1190000015352983
https://my.oschina.net/renhc/blog/2221426

10. golang 的内存回收是如何做到的？
go 的 GC 采用了mark and sweep 标记清除模式，从根变量扫描遍历所有被引用的对象并进行标记，将没有标记的对象回收；
缺点：每次执行垃圾回收时会暂停所有正常运行的代码，系统响应能力降低，但后期加入了三色标记法与写屏障，又使得性能缓解；

三色标记法：
* 起初所有对象都是白色。
* 从根出发扫描所有可达对象，标记为灰色，放入待处理队列。
* 从队列取出灰色对象，将其引用对象标记为灰色放入队列，自身标记为黑色。
* 重复上一步，直到灰色对象队列为空。此时白色对象即为垃圾，进行回收。

http://legendtkl.com/2017/04/28/golang-gc/

11. cap和len分别获取的是什么, cap函数适用的类型有哪些？

len:
array: 指数组成员的数量
slice: 指切片成员的数量
channel: 指通道缓冲区未读的成员数量

cap:
array: 指数组成员的数量
slice: 指切片当前最大容量
channel: 指通道缓冲区的容量

https://studygolang.com/articles/8519

12. go build时，tag 中 netgo，netcgo有什么区别？
netgo 使用 /opt/go/pkg/tool/linux_amd64/compile 进行编译；
netcgo 使用 gcc 编译；


13. 什么是interface？

interface是一种抽象类型，是一组方法(method)的集合，
是鸭式（duck-type）编程的一种体现;

但凡不相关的对象实现了interface中的所有方法，
那这些不相关的对象都可以给interface变量赋值，从而完成一些共性的行为操作；

14. 使用go语言，编写并行计算的快速排序算法

// todo

15. 数据竞争（Data Race）问题怎么解决？能不能不加锁解决这个问题？

数据竞争指并发多个协程对同一共享资源进行读写操作时，产生的数据错乱问题；
解决的关键在于同一时间仅允许一个协程对共享资源进行读写操作；

通过 `go build -race` 命令，生成可执行文件并运行，即可检测资源竞争并输出检测报告；

加锁的方式处理：`sync/atomic`、`sync.Mutex`


https://software.intel.com/en-us/blogs/2013/01/06/benign-data-races-what-could-possibly-go-wrong

16. 常用的 golang 库有哪些？用到其中哪些方法？

* viper 配置管理
* zerolog 可配置日志管理
* redigo redis客户端操作
* gin web服务搭建
* cron 定时任务设置
* jwt-go 请求认证相关
* swaggo restful api 文档化

* crypto 常用的加解密算法库
* fmt 信息格式化及打印信息
* errors 设置自定义错误信息
* net/http 服务相关
* strings|strconv 字符串处理
* time 时间相关

17. go for range 的坑？

for range 遍历的出的是slice/array中每个元素的副本，并非引用
而将这些副本值传给新变量时，我们最终结果的每一项都存的时新变量的指针，
那么所有新变量的指针都同时指向遍历到的最后一个值，故多个结果值一样；

https://studygolang.com/articles/9701


18. Golang 的 GC 触发时机是什么
* 内存阈值触发：当前分配内存是否大于上次GC后内存的2倍
* 每两分钟定时主动触发：没2min触发一次


19. golang中的引用类型与值类型分别有哪些？

值类型：int、float、bool、string；

引用类型：slice、map、channel；

两者区别：拷贝操作和函数传参

20. go vet 的用途

用于检查golang源码中的静态错误；

21. select 的用途

select是一种通信开关，用来监听协程通过channel发送的信息

* 每次执行select，都会只执行其中1个case或者执行default语句。
* 当没有case或者default可以执行时，select则阻塞，等待直到有1个case可以执行。
* 当有多个case可以执行时，则随机选择1个case执行。
* case后面跟的必须是读或者写通道的操作，否则编译出错。

22. go 的 new 与 make 的区别？

new 和 make 都是用于分配内存；
make 仅适用于slice、map、channel引用类型的非零值初始化，且会返回类型本身；
new 用于类型的零值初始化，且会返回类型的指针；

23. 怎么实现协程完美退出
使用 `waitgroup`;
* 创建一个 waitgroup 实例`wg`
* 在每个协程启动时调用 `wg.Add(1)`，或者创建n个协程前调用`wg.Add(n)`
* 当协程任务完成后，调用`wg.Done()`
* 在等待所有协程的地方调用`wg.Wait()`



### 容器相关问题

1. 如何给容器添加映射端口?

提交一个运行中的容器为镜像:
``` bash
docker commit containerid foo/live
```

运行镜像并添加端口:
``` bash
docker run -d -p 8000:80  foo/live /bin/bash
```

### Linux 操作相关
1. 如何后台启动一个服务？
* 命令：`conmmand &`，通过增加一个（&）符号，将应用程序在后台启动，但此操作在关闭终端时服务会停止；
* 命令：`nohup conmmand &`， 该命令可以在你退出帐户之后继续运行相应的进程，nohup( no hang up)；
* `bg`: 将一个在后台暂停的命令，变成继续执行; `fg`: 将后台中的命令调至前台继续运行; `jobs`: 查看当前有多少在后台运行的命令; `ctrl+z`: 将一个正在前台执行的命令放到后台，并且暂停;

2. 说说孤儿进程、僵尸进程

孤儿进程与僵尸进程都是指子进程的状态；

孤儿进程：
当父进程退出时，并未通知子进程结束，使得子进程仍然在运行，这样的子进程就是孤儿进程，
它将被系统init进程接管，当孤儿进程生命周期结束，init进程将调用wait()处理孤儿进程；

僵尸进程：
当子进程退出时，父进程并未调用wait()释放子进程的进程描述符(进程号、退出状态、运行时间等)，此时的子进程状态为僵尸状态，进程即僵尸进程；

任何子进程在exit()后，并非立即释放所有该子进程的资源信息，而是会留下一个僵尸状态的数据结构，等待父进程处理；
由于系统所能使用的进程号是有限的，如果大量产生僵尸进程占用所有进程号资源，将使得新进程无法创建，危害极大；
故孤儿进程无危害，僵尸进程存在危害；

解决僵尸进程的方法：
即 kill 掉僵尸子进程的父进程，使得僵尸进程被系统init进程接管，这样init进程会使用wait()释放掉进程号资源；
 
https://blog.csdn.net/morgerton/article/details/69388694



### 数据库相关
1. 如何做 SQL 优化


### WEB 通信相关

1. 当打开浏览器输入url到打开网页，这当中发生了什么？
// DNS 解析、TCP连接、发送HTTP请求、服务器处理请求并返回HTTP报文、浏览器解析渲染页面、连接结束

2. 什么是 restful api, 有哪些方法，分别代表什么？

3. TCP 和 UDP 有什么区别? 描述一下 TCP 四次挥手的过程中?

4. HTTPS 和 HTTP 有什么区别? 状态码有哪些？301和302有什么区别？504和500有什么区别， 401和403有什么区别？

301是永久重定向，302是临时重定向

5. 如何实现一个URL短链接服务？

将长短链接映射关系存入数据库，访问短链时，查询获取到对应数据库中的长链，然后转发请求；

https://www.zhihu.com/question/29270034/answer/46446911

6. cookie/session验证的过程 以及 token 验证过程？

由于http协议是无状态的，服务器需要记录用户的状态，所以cookie和session都是用来保持状态的方案，session又依赖cookie。二者的区别主要是
1). session 在服务器端，cookie 在客户端（浏览器）
2). session 默认被存在在服务器的一个文件里（不是内存）
3). session 的运行依赖 session id，而 session id 存于 cookie 中，如果浏览器禁用 cookie ，则 session 也会失效（但是可以通过其它方式实现，比如在 url 中传递 session_id）
4). session 可以放在 文件、数据库、或内存中都可以。
5). 用户验证这种场合一般会用 session

// todo 服务端如何获取cookie

// todo token


7. 使用golang net/http 实现一个简易的http服务
``` go
package main

import (
	"fmt"
	"log"
	"net/http"
)

// w表示response对象，返回给客户端的内容都在对象里处理
// r表示客户端请求对象，包含了请求头，请求参数等等
func index(w http.ResponseWriter, r *http.Request) {
	// 往w里写入内容，就会在浏览器里输出
	fmt.Fprintf(w, "Hello golang http!")
}

func main() {
	// 设置路由，如果访问/，则调用index方法
	http.HandleFunc("/", index)

	// 启动web服务，监听9090端口
	err := http.ListenAndServe(":9090", nil)
	if err != nil {
		log.Fatal("ListenAndServe: ", err)
	}
}
```

8. 使用 golang request/response 常用的方法？



---

* [effective go](https://www.kancloud.cn/kancloud/effective/72199)





