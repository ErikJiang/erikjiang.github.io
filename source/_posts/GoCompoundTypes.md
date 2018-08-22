title: "Go笔记之复合数据类型"
date: 2018-08-11 10:25:05
categories: "技术" 
tags: 
  - go
type: "tags"

---

这篇主要记录Go的复合数据类型，包含`数组`、`切片`、`映射`以及`结构体`等相关记录；

<!--more-->


### Array 数组
数组是一个由固定长度的特定类型元素组成的序列，一个数组可以由零个或多个元素组成。因为数组的长度是固定的，因此在Go语言中很少直接使用数组。和数组对应的类型是Slice（切片），它是可以增长和收缩的动态序列，slice功能也更灵活；

* 数组的长度是数组类型的组成部分
数组的长度必须是常量表达式，因为数组的长度需要在编译阶段确定；
```
// [3]int 与 [4]int 为不同的两种数组类型
q := [3]int{1, 2, 3}
q = [4]int{1, 2, 3, 4} // compile error: cannot assign [4]int to [3]int
```

* 指定索引和对应值的数组初始化
```
type Currency int

const (
    USD Currency = iota // 美元
    EUR                 // 欧元
    GBP                 // 英镑
    RMB                 // 人民币
)

symbol := [...]string{USD: "$", EUR: "€", GBP: "￡", RMB: "￥"}

fmt.Println(RMB, symbol[RMB]) // "3 ￥"

// 含有100个元素的数组r,除索引99为-1外其余元素为0
r := [...]int{99: -1}
```
* 数组作为函数传参的用法
当调用函数时，函数的每个入参将作为函数内部的参数变量，所以`函数参数变量接收的是一个复制的副本`，数组作为参数传递时也是如此，这样的`函数传参机制在遇到大的数组参数时往往是低效的`，在其它编程语言中可能会隐式地将数组作为引用或指针对象传入被调用的函数，而在go中则是`显式的传入数组指针`；

虽然通过指针来传递数组参数是高效的，而且也允许在函数内部修改数组的值，但是数组依然是僵化的类型，因为数组的类型包含了僵化的长度信息。

除了像需要处理特定大小数组的特例外，`数组依然很少用作函数参数`；相反，我们`一般使用slice来替代数组`;

### Slice 切片

Slice（切片）代表`变长的序列`，序列中`每个元素都有相同的类型`。一个slice类型一般写作[]T，其中T代表slice中元素的类型；slice的语法和数组很像，只是没有固定长度而已。
一个slice是一个轻量级的数据结构，提供了访问数组子序列（或者全部）元素的功能，而且slice的底层确实引用一个数组对象。

* slice的构成
```
# slice由 指针、长度、容量三个部分构成；

> 指针: 
指向第一个slice元素对应的底层数组元素的地址
（注：slice的第一个元素并不一定就是数组的第一个元素）

> 长度:
对应slice中元素的数目;
长度不能超过容量;
内建函数len()测量长度；

> 容量:
一般是从slice的开始位置到底层数据的结尾位置;
内建函数cap()测量容量;

---

如果切片操作超出cap(s)的上限将导致一个panic异常，
但是超出len(s)则是意味着扩展了slice；

```

* slice作为函数传参的使用
因为slice值包含指向第一个slice元素的指针，因此向函数传递slice将允许在函数内部修改底层数组的元素。

* slice 不支持比较
唯一合法的比较操作是与nil比较，零值的slice等于nil;
```
if summer == nil {/* ... */}
```
* 使用make创建指定元素类型、长度、容量的切片slice
```
make([]T, len)  // len num == cap num
make([]T, len, cap) // same as make([]T, cap)[:len]
```

* append 函数
内置的append函数用于向slice追加元素；

* slice内存操作技巧
一个slice可以用来模拟一个stack。最初给定的空slice对应一个空的stack，然后可以使用append函数将新的值压入stack：
```
stack = append(stack, v) // push v
```
stack的顶部位置对应slice的最后一个元素：
```
top := stack[len(stack)-1] // top of stack
```
通过收缩stack可以弹出栈顶的元素
```
stack = stack[:len(stack)-1] // pop
```

### Map 映射
哈希表（映射）是一种巧妙并且实用的数据结构。它是一个无序的key/value对的集合，其中所有的key都是不同的，然后通过给定的key可以在常数时间复杂度内检索、更新或删除对应的value。

* map的创建、修改、删除操作：
``` go
// 通过make函数创建map
ages := make(map[string]int)

// 通过map字面量语法创建
ages := map[string]int{
    "Sam": 25,
    "Lisa": 32
}

// 创建空map
ages := map[string]int{}

// 修改map
ages["Sam"] = 20

// 通过delete()函数删除元素
delete(ages, "Sam")

// 累加：简短赋值语法
ages["Sam"] += 1
ages["Sam"]++
```
* map 遍历的顺序是随机的
Map的迭代顺序是不确定的，并且不同的哈希函数实现可能导致不同的遍历顺序。在实践中，遍历的顺序是随机的，每一次遍历的顺序都不相同。

* 向一个nil值的map存入元素将导致一个panic异常
所以需要在map存数据前通过`map字面量`或者`make()`创建map
``` go
ages["carol"] = 21 // panic: assignment to entry in nil map
```
* 判断元素是否存在于map
```
// 返回的ok用于判断元素是否存在
if age, ok := ages["bob"]; !ok { /* ... */ }
```

* map可作为集合set使用
Go语言中并没有提供一个set类型，但是map中的key也是不相同的，可以用map实现类似set的功能。 

### Struct 结构体
结构体是一种聚合的数据类型，是由零个或多个任意类型的值聚合成的实体。每个值称为结构体的成员。

* 通常一行对应一个结构体成员，成员的名字在前，类型在后，如果相邻的成员的类型相同，则可以合并成一行；
```
type Employee struct {
    ID            int
    Name, Address string
    DoB           time.Time
    Position      string
    Salary        int
    ManagerID     int
}
```
* struct 结构体成员输入顺序是有意义的
成员顺序改变即会将原结构体改变成为新的结构体；

* 结构体成员名字是以大写字母开头，则该成员可以导出；
这是Go语言导出规则决定的。一个结构体可能同时包含导出和未导出的成员。

* 聚合类型struct及array的成员类型不能是其自身；
一个命名为S的结构体类型将不能再包含S类型的成员，但是S类型的结构体可以包含*S指针类型的成员，这可以让我们创建递归的数据结构，比如链表和树结构等。

* 结构体类型的零值是每个成员都为零值；

* 如果结构体没有任何成员的话就是空结构体，写作struct{}。它的大小为0；

* 结构体字面值
结构体有两种字面值语法
```
// 1. 要求以结构体成员定义的顺序为每个结构体成员指定一个字面值；
// 缺点：结构体成员有细微的调整就可能导致不能编译
// 适用范围：只在定义结构体的包内部使用，或者是在较小的结构体中使用且这些结构体的成员排列比较规则；
type Point struct{ X, Y int }
p := Point{1, 2}

// 2. 以成员名字和相应的值来初始化，可以包含部分或全部的成员，成员被忽略将默认用零值；
type Gif struct{ X, Y, W, H int }
anim := Gif{ X: 1, W: 2 }
```
* 结构体作为函数传参
较大的结构体建议使用结构体指针作为入参，考虑到效率；
如果要在函数内部修改结构体成员的话，用指针传入是必须的；

* 结构体嵌入与匿名成员
`结构体嵌入机制`让一个命名的结构体包含另一个结构体类型的成员;
```
type Point struct {
    X, Y int
}

type Circle struct {
    Center Point
    Radius int
}

type Wheel struct {
    Circle Circle
    Spokes int
}
var w Wheel
w.Circle.Center.X = 8
w.Circle.Center.Y = 8
w.Circle.Radius = 5
w.Spokes = 20
```
但由于修改成员字面量需要逐层访问子成员太过于繁琐，故go语法提供了匿名成员嵌套的方式，这样之后再修改时，就可以通过简短形式访问匿名成员下的嵌套成员；
```
type Circle struct {
    Point
    Radius int
}

type Wheel struct {
    Circle
    Spokes int
}
var w Wheel
w.X = 8            // equivalent to w.Circle.Point.X = 8
w.Y = 8            // equivalent to w.Circle.Point.Y = 8
w.Radius = 5       // equivalent to w.Circle.Radius = 5
w.Spokes = 20
```
在声明了潜入匿名成员的结构体后，如何进行结构体成员字面量赋值？
> 结构体字面值必须遵循形状类型声明时的结构

```
w = Wheel{Circle{Point{8, 8}, 5}, 20}

w = Wheel{
    Circle: Circle{
        Point:  Point{X: 8, Y: 8},
        Radius: 5,
    },
    Spokes: 20, // NOTE: trailing comma necessary here (and at Radius)
}
```

* 匿名方法集
匿名成员并不要求是结构体类型；其实任何命名的类型都可以作为结构体的匿名成员，比如匿名类型的方法集，这个机制可以用于将一些有简单行为的对象组合成有复杂行为的对象。组合是Go语言中面向对象编程的核心；

### JSON

* 结构体slice转为JSON的过程叫编组，编组通过调用json.Marshal函数完成
```
type Movie struct {
    Title  string
    Year   int  `json:"released"`
    Color  bool `json:"color,omitempty"`
    Actors []string
}

var movies = []Movie{
    {Title: "Casablanca", Year: 1942, Color: false,
        Actors: []string{"Humphrey Bogart", "Ingrid Bergman"}},
    {Title: "Cool Hand Luke", Year: 1967, Color: true,
        Actors: []string{"Paul Newman"}},
    {Title: "Bullitt", Year: 1968, Color: true,
        Actors: []string{"Steve McQueen", "Jacqueline Bisset"}},
    // ...
}
// 将结构体编码为json
data, err := json.Marshal(movies)
// 将结构体编码为json,并进行格式化整理
data, err := json.MarshalIndent(movies, "", "    ")
```

* 只有导出的结构体成员才会被编码，所以编码的结构体应选择用大写字母开头的成员名称

* 结构体成员Tag
结构体成员Tag是和在编译阶段关联到该成员的元信息字符串；
结构体的成员Tag可以是任意的字符串面值，但是通常是一系列用空格分隔的key:"value"键值对序列；因为值中含有双引号字符，因此成员Tag一般用原生字符串面值的形式书写。
```
Year  int  `json:"released"`
Color bool `json:"color,omitempty"`
```

* json解码为结构体
编码的逆操作是解码，对应将JSON数据解码为Go语言的数据结构，Go语言中一般叫unmarshaling，通过json.Unmarshal函数完成；

### 文本与HTML模板
一个模板是一个字符串或一个文件，里面包含了一个或多个由双花括号包含的{{action}}对象。

* 文本模板
`text/template`

* HTML模板
`html/template`


---
