title: "Go笔记之协程及通道"
date: 2018-09-01 16:13:20
categories: "技术" 
tags: 
  - go
type: "tags"

---

协程与通道是实现go多任务并发程序的方式；
goroutine和channel，其支持“顺序通信进程”(communicating sequential processes)或被简称为CSP。CSP是一种现代的并发编程模型，在这种编程模型中值会在不同的运行实例(goroutine)中传递，尽管大多数情况下仍然是被限制在单一实例中。

<!--more-->

### goroutines 协程
在Go语言中，每一个并发的执行单元叫作一个goroutine。
当程序启动时，main函数即在一个单独的goroutine中运行，我们叫它`main goroutine`;

在一个普通的函数或方法调用前添加关键字go，即为goroutine函数：`go f()`;

### channel 通道
goroutine是go语言程序的`并发体`，而channels是并发体之间的`通信机制`；channel是引用类型，其零值为nil；

* 通过make函数创建channel
channel使用make函数创建将对应一个底层的数据结构引用；
```
// 创建一个无缓存int类型的通道
ch := make(chan int)
// 创建一个缓存容量为10的int类型的通道
ch1 := make(chan int, 10)
```

* channel的运算符比较
两个相同类型的channel可以使用==运算符比较；
一个channel也可以和nil进行比较。

* channel主要的两种通信行为
channel主要有`发送`和`接收`两种通信操作行为;
通过`<-`运算符表示发送与接收操作，当该运算符位于channel标识符右侧表示发送，当该运算符位于channel标识符左侧表示接收
```
ch := make(chan in)
ch <- x // 发送x到通道ch
<- ch   // 接收通道中的信息
```

* channel的关闭Close操作
当对一个已经关闭的channel进行接收操作依然可以收到之前已经成功发送的数据；
当对一个已经关闭的channel进行接收操作且该channel已经没有数据时将会产生一个零值数据；
当试图重复关闭或者关闭一个nil值的channel都会导致panic异常；
内置close函数即可关闭channel:`close(ch)`;
不管channel是否被关闭，当它没有被引用时将会被Go语言的垃圾自动回收器回收。

* 无缓存channels
无缓存Channels也被称为同步Channels。

一个基于无缓存Channels的发送或接收操作都会导致当前操作的goroutine阻塞，直到另一个goroutine在相同的channels上执行接收或发送操作；

* 串联的channels即pipeline
Channels也可以用于将多个goroutine连接在一起，一个Channel的输出作为下一个Channel的输入。这种串联的Channels就是所谓的管道（pipeline）

* 判断channel是否关闭且无值可接收
通道的接收操作可返回两个值，其中ok可以判断当前channel是否已关闭且无值可接收；
```
x, ok := <- ch
if !ok {
    // todo ...
}
```

* 使用 range channel代替`<-`接收
使用range循环依次从channel接收数据，当channel被关闭并且没有值可接收时将跳出循环。

* 单向的通道
通道有时的运用场景是典型单一的；可能会被专门用于只发送或者只接收；

为了表明这种意图并防止被滥用，Go语言的类型系统提供了单方向的channel类型，分别用于只发送或只接收的channel。

类型`chan<- int`表示一个只发送int的channel，只能发送不能接收。相反，类型`<-chan int`表示一个只接收int的channel，只能接收不能发送。（箭头<-和关键字chan的相对位置表明了channel的方向。）这种限制将在编译期检测;
```
func squarer(out chan<- int, in <-chan int) {
    for v := range in {
        out <- v * v
    }
    close(out)
}
```

* 只有在发送者所在的goroutine才会调用close函数
因为关闭操作只用于断言不再向channel发送新的数据，所以只有在发送者所在的goroutine才会调用close函数，因此对一个只接收的channel调用close将是一个编译错误。

* 缓存通道
make函数创建channel时，第二个参数可指定缓存容量；

向缓存Channel的发送操作就是向内部缓存队列的尾部插入元素，接收操作则是从队列的头部删除元素。

如果内部缓存队列是满的，那么发送操作将阻塞直到因另一个goroutine执行接收操作而释放了新的队列空间。相反，如果channel是空的，接收操作将阻塞直到有另一个goroutine执行发送操作而向队列插入元素；

* cap()及len()在通道channel中的使用
cap()函数用于获取通道的缓存容量；
len()函数用于获取通道缓存队列中有效元素个数；

### 并发的循环

### 基于select多路复用
当我们希望能够从channel中发送或者接收值，并避免因为发送或者接收导致的阻塞，尤其是当channel没有准备好写或者读时。select语句就可以实现这样的功能。select会有一个default来设置当其它的操作都不能够马上被处理时程序需要执行的逻辑。
``` go
select {
case <-ch1:
    // ...
case x := <-ch2:
    // ...use x...
case ch3 <- y:
    // ...
default:
    // ...
}
```
* Tick()函数
time.Tick函数的表现就像创建了一个在循环中调用time.Sleep的goroutine，每次被唤醒时发送一个事件。当select多路复用结束时，它会停止从tick中接收事件，但是ticker这个goroutine还依然存活，继续徒劳地尝试向channel中发送值，然而这时候已经没有其它的goroutine会从该channel中接收值了，这种情况被称为goroutine泄露；

Tick()函数只有当程序整个生命周期都需要这个时间时我们使用它才比较合适；否则应该采用如下方式：
``` go
ticker := time.NewTicker(1 * time.Second)
<-ticker.C    // receive from the ticker's channel
ticker.Stop() // cause the ticker's goroutine to terminate
```

* select 多路复用中case激活禁用方式
由于当channel为零值nil时，其发送和接收都会被阻塞，而在多路复用中channel如果为nil,将不会被select到；因此在select中可以通过为channel赋nil值，作为开关激活或禁用通道；


### 并发的退出

通过关闭一个channel来进行广播，然后每个goroutine通过select轮询检测channel的关闭状态，一旦检测到就自动退出当前goroutine函数；
