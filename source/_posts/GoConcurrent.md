title: "Go笔记之基于共享变量的并发"
date: 2018-09-02 10:40:11
categories: "技术" 
tags: 
  - go
type: "tags"

---

这篇我们了解下并发机制；
尤其是多goroutine之间的共享变量，
并发问题的分析手段，以及解决这些问题的基本模式；

<!--more-->


### 竞争条件
竞争条件指的是程序在多个goroutine交叉执行操作时，没有给出正确的结果。
竞争条件带来的问题非常难以复现而且难以分析诊断。

* 并发安全

如果一个程序或函数或类型在线性程序中能够正常操作运行，却当在并发情况下依然能够正常操作运行，那么我们认为该程序或函数或类型是并发安全的；

* 导出包级别的函数一般情况都是并发安全的；而包级变量必须采用互斥条件；

* 数据竞争

数据竞争会在两个以上的goroutine并发访问相同的变量且至少其中一个为写操作时发生。
避免数据竞争的三种方式：
```
> 不要对变量进行写操作；
> 避免从多个goroutine访问变量，使用channel；
> 多goroutine访问变量，采用互斥方式，同一时刻最多仅有一个goroutine访问；
```

### sync.Mutex 互斥锁

* 通过容量为1的缓存通道实现互斥

一个只能为1和0的信号量叫做二元信号量；
``` go
var (
    sema    = make(chan struct{}, 1) // a binary semaphore guarding balance
    balance int
)
func Deposit(amount int) {
    sema <- struct{}{} // acquire token
    balance = balance + amount
    <-sema // release token
}
```

* 通过sync.Mutex进行互斥操作
``` go
var (
    mu      sync.Mutex // guards balance
    balance int
)

func Deposit(amount int) {
    mu.Lock()
    balance = balance + amount // 临界区代码段
    mu.Unlock()
}

```

* 共享变量与临界区

mutex 所保护的共享变量一般在mutex变量声明之后立刻声明；
在Lock和Unlock之间包含的代码段可以进行正常读写操作，这之间的代码段称为临界区；

* Lock与Unlock必须成对调用

* 临界区逻辑复杂时，使用`defer Unlock()`

在临界区代码段逻辑复杂情况下，可使用`defer Unlock()`以简化代码逻辑分支中重复的`Unlock()`调用
``` go
func Balance() int {
    mu.Lock()
    defer mu.Unlock()
    return balance
}
```

* 互斥所无法重入

go中互斥锁无法重入，即无法对已经上锁的mutex再次上锁，这将会导致程序死锁，没法继续执行，从而使得程序一直保持阻塞状态；

遇到重入情况，可以将代码拆分封装为多个函数，消除锁重入情况；

### sync.RWMutex 读写锁
有时会存在一个接口要求写操作完全互斥，而读操作希望是并行执行的，这种时候的锁称为“多读单写”锁，go提供`sync.RWMutex`支持该锁行为；
``` go
var mu sync.RWMutex
var balance int
func Balance() int {
    mu.RLock() // readers lock
    defer mu.RUnlock()
    return balance
}
```
RLock只能在临界区共享变量没有任何写入操作时可用。一般来说，我们不应该假设逻辑上的只读函数/方法也不会去更新某一些变量。如果对这类函数/方法的读写行为不确定，请使用互斥锁；

### 内存同步
由于现代计算机常见为多处理器，每个处理器都会有其本地缓存，出于效率，对内存的写入操作一般会在每个处理器中缓存，并在必要时一并flush到主存。
而在多任务并行时，多个goroutine会在不同的CPU上运行，每个CPU存在独立的本地缓存，各goroutine的写入操作首先仅能作用于CPU本地缓存，在没有同步到主存之前，各CPU的写入操作均为不可见的；

所以在设计到内存同步问题时，建议如下：
1. 将变量限定于goroutine内部；
2. 若为多goroutine都需要访问的变量，采用互斥条件来访问；

### sync.Once 初始化
在多goroutine并发情况下进行初始化操作，可能会存在goroutine重复初始化操作的情况，这时，可以使用`sync.Once`进行初始化操作，它将锁定mutex,并检测是否已初始化的布尔变量，若为初始化，则执行初始化函数并设置布尔变量为true；当其他goroutine执行时，会检测该布尔值，为false代表未初始化，为true表示已执行完成初始化操作；其中`Do`方法传参为初始化函数；
``` go
func loadIcons() {
    icons = make(map[string]image.Image)
    icons["spades.png"] = loadIcon("spades.png")
    icons["hearts.png"] = loadIcon("hearts.png")
    icons["diamonds.png"] = loadIcon("diamonds.png")
    icons["clubs.png"] = loadIcon("clubs.png")
}
var loadIconsOnce sync.Once
var icons map[string]image.Image
// Concurrency-safe.
func Icon(name string) image.Image {
    loadIconsOnce.Do(loadIcons)
    return icons[name]
}
```

### 竞争条件检测
go的运行时环境和工具链提供了一套动态分析工具，专用于并发程序的竞态检测（the race detector）。

* go命令`-race`选项
只要在`go build`，`go run`或者`go test`命令后面加上-race的flag，
就会使编译器创建一个你的应用的“修改”版或者一个附带了能够记录所有运行期对共享变量访问工具的test，并且会记录下每一个读或者写共享变量的goroutine的身份信息。

* race仅能检测到运行时的竞争条件
确保并发测试能够尽可能的覆盖；

* 添加race的程序运行会慢一些
由于需要额外的记录，因此构建时加了竞争检测的程序跑起来会慢一些，且需要更大的内存，即使是这样，这些代价对于很多生产环境的工作来说还是可以接受的；

### Goroutines与线程之间比较

* 栈：

goroutine: 栈大小动态伸缩（2KB~1GB）
thread: 栈大小固定（一般2MB）

* 调度方式：

goroutine: 语言本身特性实现调度
协程的切换步骤：
1. 休眠当前goroutine
2. 唤醒执行下一个goroutine
整个过程不需要进入内核上下文，开销成本低；

thread: 操作系统内核实现调度
线程的切换步骤：
1. 挂起当前线程
2. 保存挂起线程的寄存器信息
2. 检查线程列表选出下一个执行线程
3. 恢复执行线程的寄存器信息
4. 恢复执行线程环境并执行
整个步骤需要完整的上下文切换，切换效率很慢；

* 操作系统CPU多核利用

goroutine: 
可以使用`GOMAXPROCS`变量来决定同时执行Go代码的操作系统CPU核心数；天然支持多核利用；

thread: 
各语言线程利用多核方式不同；多进程方式较多；

* 程序单元身份ID

goroutine: 
goroutine设计上取消掉了身份ID,简化了多任务并行处理复杂度；

thread: 
线程存在身份ID,可以使用线程本地存储（thread-local storage）进行管理,但是线程本地存储总被滥用，是的函数行为别的不可预知，过于复杂化；

