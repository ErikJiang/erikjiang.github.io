---
title: "Go问题集萃"
date: 2018-12-25 21:15:12
categories: "技术" 
tags: 
  - go
type: "tags"

---


1. 谈谈 进程、线程、协程，以及它们出现所要解决的问题，及相关技术的演进历程？

* [C10K问题](http://erikjiang.github.io/2018/02/15/NodeQuestion/)

2. channel 要记得 close，但 close 时候需要注意些什么？

3. channel 是通过注册相关 goroutine id 实现消息通知，具体阐述？

4. slice 相关问题

5. 携程的模型是怎样的？抢占式goroutine调用是什么意思？

6. 谈谈 互斥锁，读写锁，死锁，及其数据竞争的概念？

7. 什么是channel，为什么它可以做到线程安全？

8. 如何用channel实现一个令牌桶

9. 如何写单元测试和基准测试

10. goroutine 的调度是怎样的？

11. golang 的内存回收是如何做到的？

12. cap和len分别获取的是什么？

13. netgo，cgo有什么区别

14. 什么是interface？

15. 使用go语言，编写并行计算的快速排序算法

16. 数据竞争（Data Race）问题怎么解决？能不能不加锁解决这个问题？

17. 常用的 golang 库有哪些？用到其中哪些方法？

18. go for range 的坑？

https://studygolang.com/articles/9701


19. Golang 的 GC 触发时机是什么
阈值触发；主动触发；两分钟定时触发；

20. TCP 和 UDP 有什么区别? 描述一下 TCP 四次挥手的过程中?

21. HTTPS 和 HTTP 有什么区别? 状态码有哪些？301和302有什么区别？504和500有什么区别？







---

* [effective go](https://www.kancloud.cn/kancloud/effective/72199)





