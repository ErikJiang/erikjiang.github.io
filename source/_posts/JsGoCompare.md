---
title: "JS 与 GO 差异比较"
date: 2018-10-25 12:32:00
categories: "技术" 
tags: 
  - go
  - js
type: "tags"

---

> `Javascript is an event driven, dynamically typed and interpreted language, while Go is a statically typed and compiled language.`

### 语言特性
#### js
事件驱动、动态类型、解释型语言

#### go
静态类型、编译型语言

<!--more-->

### 编译时安全

`Go is compiled. Javascript is not, though some Javascript runtimes use JIT compilation. From the developer experience perspective, the biggest effect of compiled languages is compile-time safety. You get compile-time safety with Go, while in Javascript you can use external code linters to ease the missing of this feature.`

### 顺序执行 与 并发执行

#### js
* 顺序执行
``` javascript
async function fetchSequential() {
    const a = await fetchA();
    console.log(a);
    const b = await fetchB();
    console.log(b);
    const c = await fetchC();
    console.log(c);
}
```
* 并发执行
``` javascript
async function fetchConcurrent() {
    const values = await Promise.all([fetchA(), fetchB(), fetchC()])
    console.log(values);
}
```

#### go
* 顺序执行
``` javascript
func fetchSequential() {
	a := fetchA()
	fmt.Println(a)
	b := fetchB()
	fmt.Println(b)
	c := fetchC()
	fmt.Println(c)
}
```
* 并发执行
``` javascript
func fetchConcurrent() {
	aChan := make(chan fetchResult, 0)
	bChan := make(chan fetchResult, 0)
	cChan := make(chan fetchResult, 0)

	go func(c chan fetchResult) {
		c <- fetchA()
	}(aChan)
	go func(c chan fetchResult) {
		c <- fetchB()
	}(bChan)
	go func(c chan fetchResult) {
		c <- fetchC()
	}(cChan)

	for i := 0; i < 3; i++ {
		select {
		case a := <-aChan:
			fmt.Println(a)
		case b := <-bChan:
			fmt.Println(b)
		case c := <-cChan:
			fmt.Println(c)

		}
	}
}
```

### this 关键字

#### js
this 是对象`Object`的引用，指向对象内部属性或内部方法

#### go
js 的 this 概念，近似于 go 中的接收器 receiver ，可以将接收器变量命名为 this;(但一般建议使用短变量，单个字母命名)
``` go
type Bar struct {
    foo string
}

func (this *Bar) Foo() string {
    return this.foo
}
```

### new 关键字

#### js
`new Foo()` 将从 Foo 返回一个实例化对象，而 Foo 一般被认为是`构造函数`或者`类`;

#### go
`new(T)` 会为类型`T`的新条目，分配零值存储，并返回指针`*T`;

在`JS`等其他语言，`new`被用于初始化对象，而`go`中则只是将对象设置为零值；

在 go 中，以`New`为前缀命名的方法，表示将会返回方法名`New`后面名字为类型名的指针；
``` go
timer := time.NewTimer(d) // timer is a *time.Timer
```

### setTimeout / timer 定时器延时执行

#### js
``` javascript
setTimeout(somefunction, 3*1000)
```

#### go
``` go
time.AfterFunc(3*time.Second, somefunction)
```

### setInterval / ticker 定时器周期执行
#### js
``` javascript
setInterval(somefunction, 3*1000)
```

#### go
``` go
ticker := time.NewTicker(3 * time.Second)
go func() {
	for t := range ticker.C {
		somefunction()
	}
}()
```

### String 字面量
不同在于 go 使用双引号`""`表示字符串, 而 js 既可以双引号`""`也可以单引号`''`

### 值类型Values, 指针类型Pointers, 引用类型References

#### js
既有`值类型`也有`引用类型`，值类型如`string`、`number`，引用类型像`object`、`array`、`function`
``` javascript
var a = {
    message: 'hello'
}

var b = a;

// mutate
b.message = 'goodbye';
console.log(a.message === b.message); // prints 'true'

// reassign
b = {
    message: 'galaxy'
}
console.log(a.message === b.message); // prints 'false'
```

#### go
既有`值类型`也有`引用类型`同时还有`指针类型`，
引用类型：切片slices、字典maps、通道channels，
其余的都是值类型，但是可以通过指针的方式获得被引用的能力；
指针和引用的虽然都可以对所被指向或被引用的底值进行修改，但是指针还可以做到重新分配底值（underlaying value）；
``` go
a := struct {
    message string
}{"hello"}

b := &a

// mutate
// note b.message is short for (*b).message
b.message = "goodbye"
fmt.Println(a.message == b.message) // prints "true"

// reassign
*b = struct {
    message string
}{"galaxy"}
fmt.Println(a.message == b.message) // prints "true"
```

### Switch 语句

#### js
`js` 中 `switch` 的 `case` 子句后面默认带有 `fallthrough`，即在匹配到对应 `case` 并执行后，会默认执行之后的 `case`，除非在 `case` 后添加 `break` 跳出整个 `switch`;
``` javascript
switch (favorite) {
    case "yellow":
        console.log("yellow");
        break;
    case "red":
        console.log("red");
    case "pruple":
        console.log("(and) purple");
    default:
        console.log("white");
}
```

#### go
`go` 则恰恰相反，`case` 子句后面默认带有 `break`，即在匹配到对应 `case` 并执行后会默认直接跳出整个 `switch`，除非在 `case` 后面添加 `fallthrough`，才能接着执行之后的 `case` 语句；
``` go
switch favorite {
case "yellow":
    fmt.Println("yellow")
case "red":
    fmt.Println("red")
    fallthrough
case "pruple":
    fmt.Println("(and) purple")
default:
    fmt.Println("white")
}
```
### 函数嵌套
函数 `function` 在 js 和 go 中都是一等公民，都被允许作为参数传递、作为值返回，可进行函数嵌套，可闭包；

但请注意，关于函数嵌套，js 能够使用命名函数或匿名函数完成，但 go 仅能使用匿名函数完成嵌套；

### 函数多值返回

#### js
js 原生并不支持函数多值返回，但是 `ES6` 中提供了解构赋值语法，可以实现类似多值返回的功能；
``` javascript
function hello() {
  return ["hello", "world"];
}

var [a, b] = hello();
console.log(a,b);
```

#### go
go 原生支持函数的多值返回；
``` go
func hello() (string, string) {
	return "hello", "world"
}

func main() {
	a, b := hello()
	fmt.Println(a, b)
}
```
### 错误处理流程控制

`Javascript`使用`throw` `catch` `finally`块，`Go`使用`panic` `recover` `defer`

---
参考：[go for javascript developers](https://github.com/pazams/go-for-javascript-developers)