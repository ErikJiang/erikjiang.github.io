title: "Go笔记之反射"
date: 2018-08-26 13:25:33
categories: "技术" 
tags: 
  - go
type: "tags"

---

Reflection（反射）在计算机中表示程序能够检查自身结构的能力，尤其是类型；它是元编程的一种形式；

<!--more-->

### interface与反射
* 变量由 (type,value) `类型`与`值`两部分组成;
* 类型Type包括`static type`静态类型与`concrete type`具体类型;
```
静态类型即指go语言编码中指定的类型，如int、string
具体类型指运行时runtime系统看见的类型
```
* 类型断言取决于`具体类型`而不是`静态类型`;
* 反射永远指的是interface类型，而非go语言指定的静态类型；
* 每个interface变量实现中都有一个对应的pair，即记录了实际变量的值与类型的键值对：`(value, type)`；
* `interface{}`包含两个指针，一个指向值的类型`concrete type`，一个指向实际的值`value`；
* `反射`就是用来检测存储在接口变量内部pair对`(value, concrete type)`的一种机制；

### 反射reflect基本功能TypeOf和ValueOf
* `reflect.ValueOf()`和`reflect.TypeOf()`用于访问接口变量pair中的value及type；
``` go
// ValueOf用来获取输入参数接口中的数据的值，如果接口为空则返回0
func ValueOf(i interface{}) Value {...}

// TypeOf用来动态获取输入参数接口中的值的类型，如果接口为空则返回nil
func TypeOf(i interface{}) Type {...}
```
* 反射类型变量`reflect.Type`与`reflect.Value`
```
通过执行reflect.ValueOf(interface)，将会返回一个类型为reflect.Value的变量
通过执行reflect.TypeOf(interface)，将会返回一个类型为reflect.Type的变量
```

* 反射类型转换为真实类型有两种情况
``` go
> 情况1. 已知原有类型（使用类型断言强制装换）

`reflect.Value`变量可以通过其本身`interface()`方法获取接口变量真实内容，然后再通过类型断言进行转换，转换为原来的真实类型；

大致过程即: 反射类型 -> 接口类型 -> 真实类型

realValue := reflectValue.Interface().(已知的类型)
注意：类型断言中的预转换类型应该与已知类型一致，否则会触发Panic；

> 情况2. 未知原有类型（遍历探测其Filed及Method）

> 2.1. 获取未知类型的interface的具体变量及其类型的步骤为：

1) 先获取interface的reflect.Type，然后通过NumField进行遍历
2) 再通过reflect.Type的Field获取其Field
3) 最后通过Field的Interface()得到对应的value

> 2.2. 获取未知类型的interface的所属方法（函数）的步骤为：

1) 先获取interface的reflect.Type，然后通过NumMethod进行遍历
2) 再分别通过reflect.Type的Method获取对应的真实的方法（函数）
3) 最后对结果取其Name和Type得知具体的方法名
```

### 通过reflec.Value设置实际变量的值
`reflect.Value`是通过`reflect.ValueOf(X)`获得的，只有当X是指针的时候，才可以通过`reflec.Value`修改实际变量X的值，即：要修改反射类型的对象就一定要保证其值是可寻址`addressable`的;
``` go 
var pi float64 = 3.1415
poiner := reflect.ValueOf(&pi)
newVal := poiner.Elem()
newVal.Type() // float64
newVal.CanSet() // true
newVal.SetFloat(1.618)
```
* 当修改实际变量值时，reflect.ValueOf的传参必须为指针；
* relect.Value通过Elem()方法获取原始值对应的反射对象, 如果传入的非指针调用Elem()会Panic;
* 通过CanSet()方法判断是否可修改设置，如果传入的非指针则返回false;

### 通过reflect.ValueOf来进行方法的调用

* 要通过反射来调用起对应的方法，必须先通过reflect.ValueOf(interface)获取reflect.Value，得到“反射类型对象”后才能做下一步处理；

* 通过reflect.Value.MethodByName(funcName)方法注册并获取调用方法，需要注意的是，必须传入准确真实的方法名称，如果错误将直接panic，MethodByName返回一个函数值对应的reflect.Value方法的名字。


* []reflect.Value作为最终需要调用的方法的参数，可以没有或者一个或者多个，根据实际参数来定。

* reflect.Value的Call()方法，通过反射方式最终调用真实的方法，参数务必保持一致；


### 反射reflect性能

golang的反射性能不理想，原因：
* reflect视线中存在大量的枚举，即for循环，如类型等；
* reflect涉及到内存分配及后续的垃圾回收GC;

### 小结

* 反射提高的程序的灵活性，使得interface{}发挥更大作用
反射必须结合interface类型，也即是说变量的类型Type必须是具体类型concrete type;

* 反射可以将`接口类型变量`转换为`反射类型变量`
通过反射的`TypeOf`及`ValueOf`方法将接口变量转为反射变量；

* 反射也可将`反射类型变量`转换为`接口类型变量`
在已知类型情况下通过`reflect.value.Interface().(已知的类型)`方式转换接口类型变量；
在未知类型情况下通过遍历reflect.Type的Field然后通过Field.Interface()获取接口类型变量；

* 反射可以修改反射类型变量，但其值必须是可寻址的addressable
* 通过反射可以动态的调用方法

---
参考阅读：
* [Golang的反射reflect深入理解和示例](https://juejin.im/post/5a75a4fb5188257a82110544)
* [反射三定律](https://segmentfault.com/a/1190000006190038)