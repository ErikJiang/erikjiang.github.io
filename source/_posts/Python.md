
## Python环境搭建
[Pyenv](https://github.com/pyenv/pyenv): python 版本管理工具;

可安装不同版本的`Python`并在不同版本间任意切换。有些类似于`Node.JS`中的`NVM`命令行工具；

## Python 数据类型
* 整数
* 浮点数
* 字符串
```
字符编码问题：
1. ASCII编码为1个字节
2. Unicode编码为2个字节
3. ASCII编码仅包含英文字母、数字及一些符号，好处是存储小，缺点是仅适合英文国家使用；
4. Unicode编码包含全球大部分语言字符，好处是国际通用性高且不易出现乱码，缺点是存储是ASCII的2倍；
5. UTF8解决了Unicode与ASCII码的问题，在保证兼容所有语言的同时实现了可变长度编码，对英文采用1字节，汉字采用3字节，按需分配字符空间，节省了存储量；

Python3采用的是Unicode编码，但为了保证编码统一及避免乱码问题，
在Python文件中开头部分请添加字符编码的设定：
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
```
* 布尔（True、False）
* 空值None
* 变量 (可任意类型)
* 常量 (不可变的变量，首字母大写)
* 列表list (有序集合) => "[]"
* 元组tuple (有序集合,初始化后元素成员的指向不变) => "()"
* 字典dict (其他语言中称Map或者Hash,属于键值存储) => "{k:v}"
```
1. Dict特点：
查找和插入的速度极快，不会随着key的增加而变慢；
需要占用大量的内存，内存浪费多。
2. List特点：
查找和插入的时间随着元素的增加而增加；
占用空间小，浪费内存很少。

因此，Dict是用空间来换取时间的一种方法;
```
* 集合set (一组key的无序集合，且元素成员不可重复) => "{k1,k2...}"
* 不可变对象及可变对象？？？？


## Python 条件判断及循环
```
# 1. 条件判断
if <条件判断1>:
    <执行1>
elif <条件判断2>:
    <执行2>
elif <条件判断3>:
    <执行3>
else:
    <执行4>

# 2. 循环有两种 for in 和 while
for item in <List or Tuple>:
    <执行>

while <条件>:
    <执行>
    continue    # 循环过程中跳出本次循环进入下一次循环
    break   # 提前结束循环
    
```

## Python 函数

* 命令行下查看内置函数使用说明：`help(func_name)`；
* 内置数据类型转换函数：int、float、str、bool...
* 函数的定义
```
def 函数名(参数...):
    函数体
    pass # 函数中的占位符类似TODO的性质
    return 结果 or None
```
* 函数的参数检查
```
# 参数arg1如果不是整数或浮点数则抛异常告知参数类型错误
if not isinstance(arg1, (int, float)):
    raise TypeError('param type error.')
```
* 函数中的多个值的返回
    函数可返回多个值，其实返回的就是一个元组tuple类型的参数
* 函数的参数种类：
```
1. 位置参数
函数执行中必传的基本参数
power(x, n)

2. 默认参数
在函数定义的时候添加参数默认值,默认参数必须指向不可变对象；
def power(x, n=1):
    ...

3. 可变参数
定义在函数中的参数个数可变，定义方式如下：
def power(*args):
    ...

可变参数在函数内部得到的是一个元组tuple类型参数；
函数外部调用含有可变参数的函数时，传入的是list or tuple 时，调用方式如下：
list_num = [1,2,3]
tuple_num = (1,2,3)
power(*list_num) # 传入时需要带星号
power(*tuple_num)

4. 关键字参数
关键字参数在函数内部得到的是一个字典dict类型的参数;
关键字参数在函数中的定义如下：
def func(arg1, **arg2): # 其中arg2为关键字参数，arg1为必选的位置参数
    ...
关键字参数当遇到字典类型的入参变量时，调用方式如下：
dict_num = {a:1,b:2};
func('hello', **dict_num); ## 传入时需要双星号

5. 命名关键字参数
由于关键字参数的关键字不受限制，当我们遇到需要在关键字参数中检测必传关键字或限定关键字名称时，就需要在函数内对关键字参数做关键字的检测，这种方式会相当不便；

于是出现了命名关键字参数，它在函数声明时即限定关键字的名称及数量，对关键字参数进行了有效的约束控制；

命名关键字参数的定义：
(1) 当其前面没有定义可变参数类型时，需要以"*"号分割，"*"后的参数即为命名关键字参数：
def func(arg1, *, arg2, arg3): # arg2及arg3即为命名关键字参数
    ...

(2) 当其前面存在可变参数类型时，则在*可变参数之后，**关键字参数之前的的参数即为命名关键字参数：
def func(arg1, *arg2, arg3=1, arg4, **arg5): # arg3及arg4即为命名关键字参数
    ...

6. 参数组合

参数定义的顺序必须是：
`必选参数 > 默认参数 > 可变参数 > 命名关键字参数 > 关键字参数`
参数的类型有多达5种，但不建议使用过多的参数组合，否则函数接口的理解性下降；

7. 递归函数
在函数内部可以调用其自身的函数称为递归函数；

栈溢出：在计算机中，函数调用是通过栈（stack）数据结构实现的，
每当一个函数调用，栈就会加一层栈帧，每当函数返回，栈就会减一层栈帧，
由于栈的大小是有限的，所以递归调用次数过多，即会造成栈溢出；

针对尾递归优化的语言可以通过尾递归来做优化，防止栈溢出；

但Python并不存在尾递归优化，故依然会造成栈溢出；

```
---

## Python 高级特性
高级特性使得Python的代码量减少且简单易读，开发效率将会提高；
### 切片(Slice)
针对于：list tuple string
```
L = ['Michael', 'Sarah', 'Tracy', 'Bob', 'Jack']
# 从索引0开始直到索引3,但不包含索引3; 其中索引0可省略，即亦可写成L[:3]
L[0:3] => ['Michael', 'Sarah', 'Tracy']
# 使用倒数索引取值,索引-1是取倒数第一个成员
L[-2:] => ['Bob', 'Jack']

# 创建0～99的数列
L = list(range(100)) => [0,1,2,3,...99]
# 取前10个 or 取后10个
L[:10] or L[-10:]
# 取前10个，每两个取一个
L[:10:2] => [0,2,4,6,8]
# 每5个取一个
L[::5]

# 元组与列表的切片操作一致
(0,1,2,3,4,5)[:3] => (0,1,2)

# 字符串也可做切片，进行单个字符的切分
'ABCDEFGH'[:3] => 'ABC'
'ABCDEFGH'[::2] => 'ACEG'

```
### 迭代(Iteration)
针对于：可被`for in`执行的数据类型; dict、 list、 string、 tuple、 set、 generator function; 
```
# dict 字典类型的迭代方式
d = {'a':1, 'b':2, 'c':3}
# 1. 字典key的迭代
for key in d:
    print(key)
# 2. 字典value的迭代
for value in d.values():
    print(value)
# 3. 字典 key value 的迭代
for key, val in d.items():
    print(key, val)

# list 列表的迭代方式,注：带列表索引下标
for index, value in enumerate(['A','B','C']):
    print(index, value)

# string 字符串的迭代方式,即将字符串遍历为单个字符
for ch in 'ABCDEF':
    print(ch)

# 元组类型成员的列表集合坐标迭代
for x,y in [(1,1),(2,4),(3,9)]:
    print(x,y)

# 如何判断对象是否为`可迭代对象`？
from collections import Iterable
isinstance(<待检测的对象>, Iterable) # 返回的是boolean类型
```

### 列表生成式(List Comprehensions)
针对于：list dict
```
# 列表生成式 生成`[1x1, 2x2, 3x3, ..., 10x10]`
[x * x for x in range(1, 11)] => [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]

[x * x for x in range(1, 11) if x % 2 == 0] => [4, 16, 36, 64, 100]

# 两层for循环
[m + n for m in 'ABC' for n in 'XYZ'] => ['AX', 'AY', 'AZ', 'BX', 'BY', 'BZ', 'CX', 'CY', 'CZ']

# 字典dict类型 也可使用列表生成式
d = {'x': 'A', 'y': 'B', 'z': 'C' }
[k + '=' + v for k, v in d.items()] => ['y=B', 'x=A', 'z=C']

# 将大写转为小写
L = ['Hello', 'World', 'IBM', 'Apple']
[s.lower() for s in L] => ['hello', 'world', 'ibm', 'apple']

```

### 生成器(Generator)
针对于：generator对象
```
# 将`列表生成式`的`[]`改为`()`，就可以创建一个生成器
g = (x * x for x in range(10))
next(g) # next()方式可以获取g生成器的下一个值

# 如果一个函数定义中包含`yield`关键字，则可成为Generator生成器函数
def gFunc():
    print('step 1')
    yield 1
    print('step 2')
    yield 2
    print('step 3')
    yield 3
g = gFunc()
next(g)

# 生成器只记录集合元素推演的算法，而不存储所有计算完成的成员集合

```

### 迭代器(Iterator)
可被next()函数调用并不断返回下一个值的对象成为迭代器;

```
# 如何判断对象是否为迭代器？
from collections import Iterator
isinstance(<待检测的对象>, Interator)

# 迭代器Iterator 与 可迭代Iterable 区分
* 可被next()函数调用并不断返回下一个值的对象成为迭代器;(生成器是Iterator对象)
* 可以被for in执行的对象成为可迭代对象;(list dict str 等则是Interable对象)

# 可以将 Iterable 可迭代对象通过 iter() 函数 转换为Iterator迭代器对象
isinstance(iter([]), Iterator) => True

# Iterator 迭代器表示的是一个数据流，它的计算是惰性的，只有通过next()才会计算下一个值

```

---

## 函数式编程(Functional Programming)
* 函数式编程的函数指的是数学意义上的函数
* 函数式编程是一种抽象程度很高的编程范式
* 纯粹的函数式编程没有变量，输入确定不变的自变量，其输出即是不变的，我们称作没有副作用
* 函数式编程起源于范畴论
* 函数式编程允许函数作为参数传入另一个函数，同时也运行函数作为另一个函数的返回值;

### 高阶函数(Higher-order function)
* 变量可指向函数
* 函数名也是变量
* 函数可以接收另一个函数作为参数传入，这样的函数成为高阶函数

1. map/reduce

2. filter

3. sorted

### 返回函数

### 匿名函数

### 装饰器

### 偏函数

## 模块

### 需要掌握的技术栈
* html/css/js
* vue.js
* docker
* mongodb
* redis
* python
* django
* flask
* linux
* git
* mysql
* nginx




