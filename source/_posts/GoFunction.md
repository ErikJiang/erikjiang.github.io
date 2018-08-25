title: "Go笔记之函数"
date: 2018-08-19 10:40:01
categories: "技术" 
tags: 
  - go
type: "tags"

---

这篇关于Go的函数特性相关内容，如`递归函数`、`匿名函数`、`延迟函数`、`错误机制`等

<!--more-->

### 函数声明

* 函数声明的构成
包括函数名、形式参数列表、返回值列表（可省略）以及函数体。
``` go
func name(parameter-list) (result-list) {
    body
}
```
* 函数声明时若包含返回值列表，则该函数必须以`return`语句结尾
* 函数标识符
函数的类型被称为函数的`标识符`。如果两个函数形式参数列表和返回值列表中的变量类型一一对应，那么这两个函数被认为有相同的类型或标识符。
* go函数没有默认参数及命名关键字参数
* 函数的形参是实参的拷贝
实参通过值的方式传递，因此函数的形参是实参的拷贝，对形参进行修改不会影响实参；
但是，`如果实参包括引用类型`，如指针，slice(切片)、map、function、channel等类型，`实参可能会由于函数的间接引用被修改`。

* 没有函数体的函数声明,表示该函数不是以Go实现,仅仅是定义了函数标识符

### 递归

* 函数可变调用栈
大部分编程语言使用固定大小的函数调用栈，常见的大小从64KB到2MB不等；
固定大小栈会限制递归的深度，当你用递归处理大量数据时，需要避免栈溢出；
Go语言使用可变栈，栈的大小按需增加(初始时很小)。这使得我们使用递归时不必考虑溢出和安全问题；

### 多返回值

* Go的垃圾回收机制
垃圾回收机制会回收不被使用的内存，但是这不包括操作系统层面的资源，比如打开的文件、网络连接。所以操作系统层面的资源需要通过代码实现回收；

* 多返回之裸返回
如果一个函数所有的返回值都有显式的变量名，那么该函数的return语句可以省略操作数。这称之为bare return。
当一个函数有多处return语句以及许多返回值时，bare return 可以减少代码的重复，但是使得代码难以被理解。所以不宜过度使用；
``` go
func CountWordsAndImages(url string) (words, images int, err error) {
    resp, err := http.Get(url)
    if err != nil {
        return
    }
    doc, err := html.Parse(resp.Body)
    resp.Body.Close()
    if err != nil {
        err = fmt.Errorf("parsing HTML: %s", err)
        return
    }
    words, images = countWordsAndImages(doc)
    return
}
```

### 错误
* 一个良好的程序永远不应该发生panic异常
* 函数的错误返回参数
对于那些将运行失败看作是预期结果的函数，它们会返回一个额外的返回值，通常是最后一个，来传递错误信息。如果导致失败的原因只有一个，额外的返回值可以是一个布尔值，通常被命名为ok。
```
value, ok := cache.Lookup(key)
```
* 错误处理策略
```
> 1. 传播错误
函数中出现的失败，传播给父级函数，传播前编写错误信息时，要确保错误信息对问题细节的描述是详尽的；

> 2. 重新尝试失败的操作
限制重试的时间间隔或重试的次数，防止无限制的重试;

> 3. 输出错误信息并结束程序
这种策略只应在main中执行。对库函数而言，应仅向上传播错误，除非该错误意味着程序内部包含不一致性，即遇到了bug，才能在库函数中结束程序;

> 4. 仅输出错误信息，无需中断程序运行

> 5. 可以直接忽略掉错误
```
* 文件结尾错误（EOF）

### 函数值
在Go中，函数被看作第一类值（first-class values）：函数像其他值一样，拥有类型，可以被赋值给其他变量，传递给函数，从函数返回。对函数值（function value）的调用类似函数调用;

* 函数类型的零值是nil。调用值为nil的函数值会引起panic错误
* 函数间不可比较，函数与nil可以比较，不能用函数值作为map的key;

### 匿名函数
拥有函数名的函数只能在包级语法块中被声明，通过函数字面量（function literal），我们可绕过这一限制，在任何表达式中表示一个函数值。函数字面量的语法和函数声明相似，区别在于func关键字后没有函数名。函数值字面量是一种表达式，它的值被称为匿名函数（anonymous function）。
``` go
strings.Map(func(r rune) rune { return r + 1 }, "hello world")
```

* Go使用闭包（closures）技术实现函数值，Go程序员也把函数值叫做闭包。

### 可变参数
参数数量可变的函数称为可变参数函数。
* 声明可变参数函数
在声明可变参数函数时，需要在参数列表的最后一个参数类型之前加上省略符号“...”，这表示该函数会接收任意数量的该类型参数。
``` go
// 可变参数vals被视为slice切片
func sum(vals...int) int {
    total := 0
    for _, val := range vals {
        total += val
    }
    return total
}
```
* interface{}表示可以接收任意类型

### Deferred延迟函数
当执行到`defer`语句时，函数和参数表达式得到计算，但直到包含该defer语句的函数执行完毕时，defer后的函数才会被执行，不论包含defer语句的函数是通过return正常结束，还是由于panic导致的异常结束。你可以在一个函数中执行多条defer语句，它们的`执行顺序与声明顺序相反`;

defer语句经常被用于处理成对的操作，如打开、关闭、连接、断开连接、加锁、释放锁。通过defer机制，不论函数逻辑多复杂，都能保证在任何执行路径下，资源被释放。释放资源的defer应该直接跟在请求资源的语句后;

调试复杂程序时，defer机制也常被用于记录何时进入和退出函数。

* defer控制函数的出入口
我们可以只通过一条defer语句控制函数的入口和所有的出口，甚至可以记录函数的运行时间，需要注意一点：不要忘记defer语句后的圆括号，否则本该在进入时执行的操作会在退出时执行，而本该在退出时执行的，永远不会被执行。
``` go
func bigSlowOperation() {
    defer trace("bigSlowOperation")() // don't forget the extra parentheses
    // ...lots of work…
    time.Sleep(10 * time.Second) // simulate slow operation by sleeping
}
func trace(msg string) func() {
    start := time.Now()
    log.Printf("enter %s", msg)
    return func() { 
        log.Printf("exit %s (%s)", msg,time.Since(start)) 
    }
}
```

* 循环体中的defer
在循环体中的defer语句需要特别注意，因为只有在函数执行完毕后，这些被延迟的函数才会执行。解决方法是将循环体中的defer语句移至另外一个函数。在每次循环时，调用这个函数;
``` go
for _, filename := range filenames {
    if err := doFile(filename); err != nil {
        return err
    }
    // defer f.Close()
}
func doFile(filename string) error {
    f, err := os.Open(filename)
    if err != nil {
        return err
    }
    defer f.Close()
}
```

### Panic异常
Go的类型系统会在编译时捕获很多错误，但有些错误只能在运行时检查，如数组访问越界、空指针引用等。这些运行时错误会引起painc异常。
一般而言，当panic异常发生时，程序会中断运行，并立即执行在该goroutine中被延迟(defer)的函数。随后，程序崩溃并输出日志信息。

* panic函数
不是所有的panic异常都来自运行时，直接调用内置的panic函数也会引发panic异常；panic函数接受任何值作为参数。

* panic用于严重错误,一般性问题应采用错误机制
虽然Go的panic机制类似于其他语言的异常，但panic的适用场景有一些不同。由于panic会引起程序的崩溃，因此panic一般用于严重错误，如程序内部的逻辑不一致；

* 为方便诊断问题，runtime包允许输出堆栈信息
``` go
func main() {
    defer printStack()
    f(3)
}
func printStack() {
    var buf [4096]byte
    n := runtime.Stack(buf[:], false)
    os.Stdout.Write(buf[:n])
}
```

### Recover捕获异常
通常来说，不应该对panic异常做任何处理，但有时，也许我们可以从异常中恢复，至少我们可以在程序崩溃前，做一些操作。

如果在deferred函数中调用了内置函数recover，并且定义该defer语句的函数发生了panic异常，recover会使程序从panic中恢复，并返回panic value。导致panic异常的函数不会继续运行，但能正常返回。在未发生panic时调用recover，recover会返回nil。
``` go
func Parse(input string) (s *Syntax, err error) {
    defer func() {
        if p := recover(); p != nil {
            err = fmt.Errorf("internal error: %v", p)
        }
    }()
    // ...parser...
}
```
* 不加区分的恢复所有的panic异常，不是可取的做法；
* 不应该试图去恢复其他包引起的panic;
* 公有的API应该将函数的运行失败作为error返回，而不是panic。
* 安全的做法是有选择性的recover
只恢复应该被恢复的panic异常，此外，这些异常所占的比例应该尽可能的低。为了标识某个panic是否应该被恢复，我们可以将panic value设置成特殊类型。在recover时对panic value进行检查，如果发现panic value是特殊类型，就将这个panic作为errror处理，如果不是，则按照正常的panic进行处理;
``` go
func soleTitle(doc *html.Node) (title string, err error) {
    type bailout struct{}
    defer func() {
        switch p := recover(); p {
        case nil:       // no panic
        case bailout{}: // "expected" panic
            err = fmt.Errorf("multiple title elements")
        default:
            panic(p) // unexpected panic; carry on panicking
        }
    }()
    // Bail out of recursion if we find more than one nonempty title.
    forEachNode(doc, func(n *html.Node) {
        if n.Type == html.ElementNode && n.Data == "title" &&
            n.FirstChild != nil {
            if title != "" {
                panic(bailout{}) // multiple titleelements
            }
            title = n.FirstChild.Data
        }
    }, nil)
    if title == "" {
        return "", fmt.Errorf("no title element")
    }
    return title, nil
}
```


---


