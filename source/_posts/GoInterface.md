title: "Go笔记之接口"
date: 2018-08-24 22:10:05
categories: "技术" 
tags: 
  - go
type: "tags"

---

接口类型是对其它类型行为的抽象和概括；
因为接口类型不会和特定的实现细节绑定在一起，通过这种抽象的方式可以让我们的函数更加灵活和更具有适应能力；

<!--more-->


### 接口约定
接口类型是一种抽象的类型(不同于数组、映射、切片等具体类型)。它不会暴露出它所代表的对象的内部值的结构和这个对象支持的基础操作的集合；它们只会表现出它们自己的方法。也就是说当你有看到一个接口类型的值时，你不知道它是什么，唯一知道的就是可以通过它的方法来做什么。


### 接口类型
接口类型具体描述了一系列方法的集合，一个实现了这些方法的具体类型是这个接口类型的实例。

* 接口类型可以通过组合已有的接口来定义
``` go
type Reader interface {
    Read(p []byte) (n int, err error)
}
type Closer interface {
    Close() error
}
type ReadWriter interface {
    Reader
    Writer
}
type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}
```
以上接口类型的组合类似于结构内嵌，可以称为接口内嵌

### 实现接口的条件

* 接口指定规则
表达一个类型属于某个接口只要这个类型实现这个接口

* 即使具体类型有其它的方法，也只有接口类型暴露出来的方法能被调用到

* 空接口类型
空接口类型`interface{}`
空接口类型对实现它的类型没有要求，所以我们可以将任意一个值赋给空接口类型;
interface{}代表空接口，任何类型都是它的实现类型；
将非接口类型标识符x转换为接口类型：`interface{}(x)`;

* 空花括号
`{}`: 一对不包裹任何东西的花括号，除了可以代表空的代码块之外，还可以用于表示不包含任何内容的数据结构（或者说数据类型）
``` go
struct{} // 代表了不包含任何字段和方法的、空的结构体类型
interface{} // 代表了不包含任何方法定义的、空的接口类型
[]string{}  // 代表空切片，值不包含任何元素
map[int]string{} // 空字典
...
```

* 类型转换
语法形式是`T(x)`，其中的x可以是一个变量，也可以是一个代表值的字面量，还可以是一个表达式。注意，如果是表达式，那么该表达式的结果只能是一个值，而不能是多个值。在这个上下文中，x可以被叫做源值，它的类型就是源类型，而那个T代表的类型就是目标类型。




* 并非仅指针类型才能实现接口类型
接口类型也能被其他引用类型实现，如slice、map、function等，甚至基本类型也可以实现一些接口；

### flag.Value接口

### 接口值
接口值，由两个部分组成，一个具体的类型和那个类型的值。它们被称为接口的动态类型和动态值;

* 空接口值即其类型type和值value都为nil

* 调用一个空接口值上的任意方法都会产生panic

* 接口值可以使用==和!＝来进行比较
两个接口值相等仅当它们都是nil值，或者它们的动态类型相同并且动态值也根据这个动态类型的==操作相等。因为接口值是可比较的，所以它们可以用在map的键或者作为switch语句的操作数。

* 通过`%T`获取接口值类型
在fmt包内部，使用反射来获取接口动态类型的名称
`fmt.Printf("%T\n", w)`

* 一个包含nil指针的接口不是nil接口
一个不包含任何值的nil接口值和一个刚好包含nil指针的接口值是不同的

### sort.Interface接口
内置的排序算法需要知道三个东西：
1. 序列的长度
2. 表示两个元素比较的结果
3. 一种交换两个元素的方式

```
package sort

type Interface interface {
    Len() int
    Less(i, j int) bool // i, j are indices of sequence elements
    Swap(i, j int)
}
```
为了对序列进行排序，我们需要定义一个实现了这三个方法的类型，然后对这个类型的一个实例应用sort.Sort函数。

### http.Handler接口
``` go
package http

type Handler interface {
    ServeHTTP(w ResponseWriter, r *Request)
}

func ListenAndServe(address string, h Handler) error
```

### error 接口
error类型实际上是interface类型
```
type error interface {
    Error() string
}
```
* 创建error方式

``` go
// 三种
import "fmt"
import "errors"
errors.New()    // 创建error
fmt.Errorf()    // 创建error并格式化，是errors.New()的封装
syscall.Errno(1)// 创建err的数字类型，满足标准错误接口
```

### 类型断言
类型断言是一个使用在接口值上的操作(非接口类型需要通过`interface{}`进行转换后再进行断言操作)。语法上它看起来像`x.(T)`被称为断言类型，这里`x`表示一个接口的类型而`T`表示一个类型。一个类型断言检查它操作对象的动态类型是否和断言的类型匹配。

* 断言的两种可能
```
> 1. 断言的类型T为具体类型
> 2. 断言的类型T为接口类型
```
* 断言时返回两个参数可以避免发生panic
```
var w io.Writer = os.Stdout
f, ok := w.(*os.File)      // success:  ok, f == os.Stdout
b, ok := w.(*bytes.Buffer) // failure: !ok, b == nil
```

### 基于类型断言识别错误类型
``` go
import (
    "errors"
    "syscall"
)

var ErrNotExist = errors.New("file does not exist")

// IsNotExist returns a boolean indicating whether the error is known to
// report that a file or directory does not exist. It is satisfied by
// ErrNotExist as well as some syscall errors.
func IsNotExist(err error) bool {
    // 错误类型断言
    if pe, ok := err.(*PathError); ok {
        err = pe.Err
    }
    return err == syscall.ENOENT || err == ErrNotExist
}
_, err := os.Open("/no/such/file")
fmt.Println(os.IsNotExist(err)) // "true"
```

### 通过类型断言查询接口
我们不能对任意io.Writer类型的变量，假设它也拥有WriteString方法。但是我们可以定义一个只有这个方法的新接口并且使用类型断言来检测是否变量的动态类型满足这个新接口。
``` go
func writeString(w io.Writer, s string) (n int, err error) {
    // 定义新接口
    type stringWriter interface {
        WriteString(string) (n int, err error)
    }
    // 类型断言检测接口方法是否存在
    if sw, ok := w.(stringWriter); ok {
        return sw.WriteString(s) // avoid a copy
    }
    return w.Write([]byte(s)) // allocate temporary copy
}

func writeHeader(w io.Writer, contentType string) error {
    if _, err := writeString(w, "Content-Type: "); err != nil {
        return err
    }
    if _, err := writeString(w, contentType); err != nil {
        return err
    }
    // ...
}
```

### 类型分支
```
func sqlQuote(x interface{}) string {
    switch x := x.(type) {
    case nil:
        return "NULL"
    case int, uint:
        return fmt.Sprintf("%d", x) // x has type interface{} here.
    case bool:
        if x {
            return "TRUE"
        }
        return "FALSE"
    case string:
        return sqlQuoteString(x) // (not shown)
    default:
        panic(fmt.Sprintf("unexpected type %T: %v", x, x))
    }
}
```
虽然x的类型是interface{}，但是我们把它认为是一个int，uint，bool，string，和nil值的discriminated union（可识别联合）

### 补充

* 设计新包时，并不一定要用到接口
只有当有两个或两个以上的具体类型，且都必须以相同的方式进行处理时，才需要使用接口;

---
