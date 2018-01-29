
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
```
# map
<Iterator迭代器对象> = map(<function>, <Iterable可迭代对象>)

# reduce
reduce(f, [x1, x2, x3, x4]) = f(f(f(x1, x2), x3), x4)
```
2. filter
```
filter()把传入的函数依次作用于每个元素，然后根据返回值是True还是False决定保留还是丢弃该元素。
```
3. sorted
```
sorted(['bob','about','Zoo','Credit'], key=str.lower, reverse=True)
# key指定的函数将作用于list的每一个元素上，并根据key函数返回的结果进行排序
# 要进行反向排序，可以传入第三个参数reverse=True
```

### 返回函数
```
* 当一个函数返回了一个函数后，其内部的局部变量会被新函数引用，这样的程序结构称为闭包；
* 返回闭包时牢记一点：返回函数不要引用任何循环变量，或者后续会发生变化的变量。
* 如果一定要引用循环变量,方法是再创建一个函数，用该函数的参数绑定循环变量当前的值，无论该循环变量后续如何更改，已绑定到函数参数的值不变：
```
### 匿名函数
```
* 关键字 lambda 表示匿名函数，冒号:前面表示函数的参数；lambda函数只能有一个表达式且不需要return;
* 匿名函数不需要函数名称，且是一个函数对象可以赋值给一个变量；
```
### 装饰器(Decorator)
在面向对象（OOP）的设计模式中，decorator被称为装饰模式。OOP的装饰模式需要通过继承和组合来实现，而Python除了能支持OOP的decorator外，直接从语法层次支持decorator。Python的decorator可以用函数实现，也可以用类实现。
```
# 本质上，decorator就是一个返回函数的高阶函数
import functools

def log(text):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kw):
            print('%s %s():' % (text, func.__name__))
            return func(*args, **kw)
        return wrapper
    return decorator

```

### 偏函数(Partial function)
```
当函数的参数个数太多，需要简化时，使用functools.partial可以创建一个新的函数，这个新函数可以固定住原函数的部分参数，从而在调用时更简单。
import functools
functools.partial(<func name>, arg=<设定值>)
```

## 模块
### 作用域
* 正常的函数和变量名是公开的（public）
* 类似_xxx和__xxx这样的函数或变量就是非公开的（private），不应该被直接引用
* 类似__xxx__这样的变量是特殊变量，可以被直接引用，但是有特殊用途,比如__author__，__name__

### 第三方模块
* 在Python中，安装第三方模块，是通过包管理工具pip完成的
* 可以直接安装Anaconda，内置了数十个常用的第三方模块

## 面向对象编程
### 类和实例
```
# 类的定义 类名首字母大写
class ClassName(object):
    pass

# 类的实例化
instance = ClassName()

# 类的属性参数绑定
class Student(object):
    # __init__方法类似于JS中的constructor,且第一个参数永远是self
    # self指向创建的实例本身，调用时self不用传
    def __init__(self, name, score):
        self.name = name
        self.score = score
    # 类中定义的函数第一个参数永远是self

# 类中数据的封装
通过在类中定义函数来实现访问类中的数据
```

### 访问权限
```
# 通过`__`定义的内部变量会被识别为类的私有变量(private)
# 私有变量只能在内部访问，外部不能访问
class Student(object):
    def __init__(self, name, score):
        self.__name = name
        self.__score = score
    def print_score(self):
        print('%s: %s' % (self.__name, self.__score))
    def get_name(self):
        return self.__name
    def set_name(self, name):
        return self.__name
# 私有变量并非完全无法访问，私有变量其实可以通过如下方式访问
<instance>._<classname>__<propname>
# 例如：
studentInst = Student()
studentInst._Student__name

# Python 类的机制不能完全的防范开发的行为，所以需要自觉遵守正常的方式

```

### 继承与多态
在OOP程序设计中，当定义Class时，可以从现有的Class继承，新的Class称为子类（Subclass）,被继承的类称为基类、父类和超类（BaseClass、SuperClass）;
```
# 继承
class BasicClass(object):
    def run(self):
        pass
class SubClass(BasicClass):
    pass

subinst = SubClass()
subinst.run()

# 静态类型语言的继承必须传入规定类型的父类
# 动态语言"鸭子类型" 不严格要求继承体系，只要父类存在子类所需的属性方法即可

```

### 获取对象信息
* 判断对象的类型可以使用`type()`函数, 返回对应的Class类型;
* `isinstance()`可以判断对象是否为该类型本身或者是否位于该类型父级继承链上;
* 使用`dir()`可以列出对象的属性及方法名称，返回一个包含str的list;
* 获取对象属性`getattr()`,设置对象属性`setattr()`,判断对象属性是否存在`hasattr()`

```
# 判断对象是否为函数类型
def fn():
    pass
type(fn) == types.FunctionType
type(abs) == types.BuiltinFunctionType
type(lambda x:x) == types.LambdaType
type((x for x in range(10))) == types.GeneratorType

# 判断是否是 list or tuple
isinstance([1, 2, 3], (list, tuple))

```

### 实例属性与类属性
* 实例属性属于各个实例所有，互不干扰；
* 类属性属于类所有，所有实例共享一个属性；
* 不要对实例属性和类属性使用相同的名字，否则将产生难以发现的错误。

## 面向对象高级编程
### 使用__slots__
* 为实例对象绑定方法，通常可以使用`MethodType`,但仅对当前作用的实例生效，对其他实例无效;
* 当需要对实例对象绑定的属性作出限制时，可以使用`__slots__`;
* `__slots__`仅对当前类的实例起到属性限制作用，对于继承其的子类没有效果;

```
# 通过MethodType为实例对象绑定一个方法
s = Student()
def set_age(self, age):
    self.age = age
from types import MethodType
s.set_age = MethodType(set_age, s)

# 限制对象实例中定义的属性
class Student(object):
    __slots__ = ('name', 'age') # 用tuple定义允许绑定的属性名称

```
### 使用@property
* `@property`是Python内置的装饰器,其作用是将类中的方法转换为属性的调用方式;
* 将一个属性的`getter`方法变成属性，仅需要在其方法上添加`@property`;
* 将一个属性的`setter`方法变成属性，必须先使用`@property`创建`getter`方法，然后在其`setter`方法之上添加`@<funcName>.setter`即可;
* 如果一个方法仅仅定义了`getter`方法，没有定义`setter`,那么所转换的属性将是一个只读属性;
```
class Student(object):
    # setter 方法
    @property
    def name(self):
        return self.__name

    # getter 方法
    @name.setter
    def name(self, name):
        if not isinstance(name, str):
            raise ValueError('name must be a string!')
        self.__name = name

```

### 多重继承
* 多重继承: 类名后面括号中的继承类参数可以传入多个继承类;
* 通过多重继承，可以混入额外的功能，我们将这种设计称为`MixIn`;
```
# 多重继承
class ClassName(Class1,Class2...):
    pass

```
### 定制类
为方便生成特定的类，Python提供类用于定制类的多个方法：
* __str__() 设置打印类实例的信息，针对于用户展示;
* __repr__() 设置显示类实例的信息，针对于开发者调试;
* __iter__() 设置对象为可迭代对象，设置后可用于`for in`循环;
* __next__() 当设置__iter__()后返回迭代对象，`for in`调用是会执行__next__()方法返回下一个值;
* __getitem__() 将类实例设置为一个具有list tuple dict类型属性的实例，可通过数组下标访问;
* __setitem__() 将类实例设置为能够具有list tuple dict类型的赋值能力
* __delitem__() 删除某个元素
* __getattr__() 在没有找到属性时，动态返回属性或方法
* __call__() 将类的实例本身作为函数方法调用，可调用的对象被称为`Callable`对象


### 使用枚举类
* python 提供了`Enum`枚举类，用于定义同种类型属性的常量
```
# 通过枚举类定义月份常量
from enum import Enum
Month = Enum('Month', (
    'Jan', 'Feb', 'Mar', 'Apr', 
    'May', 'Jun', 'Jul', 'Aug', 
    'Sep', 'Oct', 'Nov', 'Dec'
    ))

# 通过枚举类进行自定义类 @unique 可以帮助检测是否存在重复
from enum import Enum, unique
@unique
class Weekday(Enum):
    Sun=0
    Mon=1
    Tue=2
    Wed=3
    Thu=4
    Fri=5
    Sat=6

```

### 使用元类
* 使用`type()`方式可以创建类，通常的class定义方式其实仅仅是扫描了定义的语法然后调用type()创建class而已;
* 除了使用`type()`创建类，还可以使用`metaclass`,其使用顺序：`先定义metaclass`->`创建类`->`创建实例`;

```
# 使用type()动态创建类的方式
def fn(self, name='world'):
    print('Hello, %s' % name)
Hello = type('Hello', (Object,), dict(hello=fn))
# type(<类名称>,<继承的父类元组集合>,<绑定函数的字典>)

```

## 错误、调试与测试

### 错误处理
* `try...except...finally...`:
    
    1. 当try中的代码块执行错误，则会进入except中去执行异常处理,执行完成后如果存在finally,则执行finally中代码;
    2. 当try中的代码块正常执行完成，则不会进入except,然后若存在finally,则执行finally中代码;
    3. 当try中存在的错误类型并非一种，则except可以针对不同的错误类型定义多个except异常处理代码块;
    4. 可以在except之后添加else,这样当try中的代码块没有错误发生，则else会被执行;

* Python的错误同样是类，所有的错误都从`BaseException`类中派生而来;
* Python内置`logging`模块可以清晰大打印错误栈信息

    ```
    import logging
    logging.exception()
    ```
* 使用`raise`抛出一个错误
    ```
    raise ValueError('invalid value')

    ```

### 调试

* 调试手段1: 通过打印`print()`
* 调试手段2: 通过断言`assert()`,同时python启动器可以通过`-O`参数关闭断言
* 调试手段3: 通过日志`import logging`,可以设置记录日志信息的级别
* 调试手段4: 通过Python调试器pdb,可以进行单步运行调试
* 调试手段5: 同样通过Python调试器pdb,使用pdb.set_trace()打断点,无需单步执行
* 调试手段6: 通过IDE,如`VSCode`、`PyCharm`

推荐：logging + IDE 配合方式

### 单元测试
* 单元测试：即对单一模块、函数、类进行正确性验证的测试
* python自带的单元测试模块`unittest`
    1. 编写单元测试时，要定义一个测试类，并继承unittest.TestCase
    2. 编写具体测试方法时，方法名称要以`test`开头
    ```
    # unittest常见的断言类型
    # 1. 判断是否相等
    self.assertEqual()
    # 2. 判断是否与预期的错误类型一致
    self.assertRaises(KeyError):
        value = dict1['empty']
    # 3. 判断是否为True
    self.assertTrue()

    ```
* 推荐命令行使用`-m unittest`直接运行单元测试脚本，这样可以批量运行测试脚本；
* setup 与 tearDown 方法会在每个测试方法前后运行
    用于添加预定义操作和执行完成后的内存释放等工作
 

### 文档测试
* Python 内置了文档测试(doctest)模块，通过用`'''`的方式编写注释文档，如下：
```
class Dict(dict):
    '''
    Simple dict but also support access as x.y style.

    >>> d1 = Dict()
    >>> d1['x'] = 100
    >>> d1.x
    100
    >>> d1.y = 200
    >>> d1['y']
    200
    >>> d2 = Dict(a=1, b=2, c='3')
    >>> d2.c
    '3'
    >>> d2['empty']
    Traceback (most recent call last):
        ...
    KeyError: 'empty'
    >>> d2.empty
    Traceback (most recent call last):
        ...
    AttributeError: 'Dict' object has no attribute 'empty'
    '''
    def __init__(self, **kw):
        super(Dict, self).__init__(**kw)

    def __getattr__(self, key):
        try:
            return self[key]
        except KeyError:
            raise AttributeError(r"'Dict' object has no attribute '%s'" % key)

    def __setattr__(self, key, value):
        self[key] = value

if __name__=='__main__':
    import doctest
    doctest.testmod()

```
* 当在命令行执行脚本时，会调用doctest进行文档测试，当脚本作为模块导出时，不会进行文档测试

## IO编程
* I/O 即计算机中的Input输入(读)/Output输出(写)
* IO Stream 中数据从外部(磁盘网络)流入内存称为`Input Stream`；
* IO Stream 中数据从内存流入外部(磁盘网络)称为`Output Stream`；
* 由于内存速度远远高于外设的速度，所以围绕着运行性能效率，IO编程中出现了`同步IO`与`异步IO`两种不同方式；
* 异步IO性能高于同步IO，但异步IO编程模型复杂，异步的消息通知机制有`回调`及`轮询`等模式；

### 文件读写
* 如果文件很小，read()一次性读取最方便；
* 如果不能确定文件大小，反复调用read(size)比较保险；
* 如果是配置文件，调用readlines()最方便
```
# 写法一 使用try...finally
try:
    f = open('/path/file', r)
    print(f.read())
finally:
    if f:
        f.close()
# 写法二 使用with无需每次调用f.close()
with open('/path/file', r) as f:
    print(f.read())
with open('/Users/michael/test.txt', 'w') as f:
    f.write('Hello, world!')
    
```
### StringIO和BytesIO

### StringIO 在内存中读写str
* getvalue() 用于获取写入的str

```
from io import StringIO
f = StringIO()  # 实例化
f.write('hello world') # 写入操作
print(f.getvalue()) => 'hello world'
f.readline() # 读取操作
```
### BytesIO 用于内存中读写二进制数据
```
from io import BytesIO
f = BytesIO()
f.write('您好 世界'.encode('utf-8')) # 写入经过UTF8编码得到Bytes
print(f.getvalue()) => b'\xe4\xb8\xad\xe6\x96\x87'
f.read() # 读取二进制数据

```

### 操作文件和目录
> 通过Python内置的`os`模块可以直接调用操作系统提供的接口函数
* 除了os内置模块外，还有shutil模块提供了更多的实用函数

```
import os
os.name # 获取操作系统类型 posix or nt
os.uname # 获取详细系统信息 windows环境不提供
os.environ # 获取操作系统定义的环境变量
os.environ.get('PATH') # 获取环境变量PATH的内容
os.path.abspath('.') # 查看当前目录的绝对路径 如：'/user/src'
os.path.join('/user/src', 'testdir') # 在某个目录下创建新目录 如：'/user/src/testdir'
os.mkdir('/user/src/testdir') # 创建目录
os.rmdir('/user/src/testdir') # 删除目录
os.path.split('/user/src/testdir/file.txt') # 拆分目录 如：('/user/src/testdir', 'file.txt')
os.path.splitext() # 获取文件后缀名
os.rename('test.txt', 'test.py') # # 对文件重命名
os.remove('test.py') # 删掉文件

```
### 序列化
> 变量从内存中变成可被存储或利于传输的过程称为序列化，在Python中成为Pickling，其他语言称为serialization

> 通过序列化之后，即可以将结果写入磁盘或通过网络传输到其他终端

> 反之，变量内容由序列化对象重新读入内存的过程成为 反序列化 即unpickling

> Python 中提供 pickle 模块实现序列化

> Python 中提供 json 模块实现JSON标准序列化，提供的方法`dump()`&`load()`与pickle类似

```
import pickle

# 序列化
d = dict(name='Bob', age=20, score=88)
pickle.dumps(d) # pickle.dumps() 将任何对象序列化为bytes

f = open('dump.txt', 'wb')
pickle.dump(d) # pickle.dumps() 将任何对象序列化后写入`类文件对象(file-like Object)`
f.close()

# 反序列化
pickle.loads() # 将bytes反序列化为对象

f = open('dump.txt', 'rb')
d = pickle.load(f) # 从`类文件对象`中反序列化出对象
f.close()

```
> 将Python Class转换为JSON序列化，需要编写转换函数，不同直接进行转换
```
# 类的序列化与反序列化
import json
class Student(object):
    def __init__(self, name, age, score):
        self.name = name
        self.age = age
        self.score = score

s = Student('Bob', 20, 88)

def student2dict(std):
    return {
        'name': std.name,
        'age': std.age,
        'score': std.score
    }
print(json.dumps(s, default=student2dict))

def dict2student(d):
    return Student(d['name'], d['age'], d['score'])
json_str = '{"age": 20, "score": 88, "name": "Bob"}'
print(json.loads(json_str, object_hook=dict2student))

```

## 进程与线程
* 多任务的实现方式：
    1. 多进程模式
    2. 多线程模式
    3. 多进程+多线程模式（相比前两种模式复杂度更高，所以很少采用）

### 多进程（multiprocessing）
* multiprocessing模块 fork()函数
* 由于windows平台不支持fork(),故除了使用内置的fork()外，还可以使用跨平台的`multiprocessing`，可以兼容Windows
* multiprocessing模块 Process类创建单进程实例
```
from multiprocessing import Process
import os

# 子进程要执行的代码
def run_proc(name):
    print('Run child process %s (%s)...' % (name, os.getpid()))

if __name__=='__main__':
    print('Parent process %s.' % os.getpid())
    p = Process(target=run_proc, args=('test',))
    print('Child process will start.')
    p.start()
    p.join()    # 等待子进程执行完成后才继续往下执行
    print('Child process end.')
```
* multiprocessing模块 Pool进程池批量创建子进程
```
from multiprocessing import Pool
import os, time, random

def long_time_task(name):
    print('Run task %s (%s)...' % (name, os.getpid()))
    start = time.time()
    time.sleep(random.random() * 3)
    end = time.time()
    print('Task %s runs %0.2f seconds.' % (name, (end - start)))

if __name__=='__main__':
    print('Parent process %s.' % os.getpid())
    p = Pool(4)
    for i in range(5):
        p.apply_async(long_time_task, args=(i,))
    print('Waiting for all subprocesses done...')
    p.close() # 在join()前调用close(),在调用close()之后不能添加创建新的进程实例
    p.join()  # 等待所有子进程执行完毕后才能继续往下执行
    print('All subprocesses done.')
```
* subprocess 模块可以方便的启动子进程并控制其输入输出
```
import subprocess

print('$ nslookup')
p = subprocess.Popen(['nslookup'], stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
output, err = p.communicate(b'set q=mx\npython.org\nexit\n')
print(output.decode('utf-8'))
print('Exit code:', p.returncode)

```
* 进程间通信，multiprocessing模块中提供了`Queue`、`Pipes`等多种方式进行进程间数据交换
```
from multiprocessing import Process, Queue
import os, time, random

# 写数据进程执行的代码:
def write(q):
    print('Process to write: %s' % os.getpid())
    for value in ['A', 'B', 'C']:
        print('Put %s to queue...' % value)
        q.put(value)
        time.sleep(random.random())

# 读数据进程执行的代码:
def read(q):
    print('Process to read: %s' % os.getpid())
    while True:
        value = q.get(True)
        print('Get %s from queue.' % value)

if __name__=='__main__':
    # 父进程创建Queue，并传给各个子进程：
    q = Queue()
    pw = Process(target=write, args=(q,))
    pr = Process(target=read, args=(q,))
    # 启动子进程pw，写入:
    pw.start()
    # 启动子进程pr，读取:
    pr.start()
    # 等待pw结束:
    pw.join()
    # pr进程里是死循环，无法等待其结束，只能强行终止:
    pr.terminate()
```

### 多线程
* 一个进程默认会启动一个线程，这个默认线程称为`主线程`（MainThread）;
* Python标准库内置两个线程模块：`_thread`和`threading`，其中`threading`是对`_thread`的高级封装，一般情况使用`threading`
* 多线程与多进程最大的不同是，多进程的变量，各自的进程会拷贝一份，互不影响；多线程的变量则可以在所有线程中共享，任何一个变量都可以被任何一个线程修改；
* 多线程锁机制，使用`threading.Lock()`
    1. 锁的好处：确保某段代码从头至尾只能由一个线程执行完成；
    2. 锁的坏处：不能进行多线程并发，使用锁后仅能单线程执行，效率下降；另外存在多线程死锁问题；
* 由于Python解析器存在GIL(Global Interpreter Lock)全局解析器锁，使得多线程只能在单核上执行，所以要实现多核多线程任务，只能通过多个进程来实现；
```
import time, threading

# 新线程执行的代码:
def loop():
    # current_thread()返回值永远是当前线程的实例
    print('thread %s is running...' % threading.current_thread().name)
    n = 0
    while n < 5:
        n = n + 1
        print('thread %s >>> %s' % (threading.current_thread().name, n))
        time.sleep(1)
    print('thread %s ended.' % threading.current_thread().name)

print('thread %s is running...' % threading.current_thread().name)
t = threading.Thread(target=loop, name='LoopThread') # 指定线程执行函数及线程的名称
t.start()
t.join()
print('thread %s ended.' % threading.current_thread().name)

# 线程锁
balance = 0
lock = threading.Lock()

def change_it(n):
    # 先存后取，结果应该为0:
    global balance
    balance = balance + n
    balance = balance - n

def run_thread(n):
    for i in range(100000):
        # 先要获取锁:
        lock.acquire()
        try:
            # 放心地改吧:
            change_it(n)
        finally:
            # 改完了一定要释放锁:
            lock.release()

t1 = threading.Thread(target=run_thread, args=(5,))
t2 = threading.Thread(target=run_thread, args=(8,))
t1.start()
t2.start()
t1.join()
t2.join()
print(balance)

```


### ThreadLocal
* 我们知道多线程中的全局变量是可以共享的，导致会出现全局变量污染的问题，为了解决变量在线程间可被随意篡改的问题，出现了`ThreadLocal`;
* ThreadLocal可被理解为单个线程内的全局变量，但不会被其他线程所干扰篡改；
```
import threading

# 创建全局ThreadLocal对象:
local_school = threading.local()

def process_student():
    # 获取当前线程关联的student:
    std = local_school.student
    print('Hello, %s (in %s)' % (std, threading.current_thread().name))

def process_thread(name):
    # 绑定ThreadLocal的student:
    local_school.student = name
    process_student()

t1 = threading.Thread(target= process_thread, args=('Alice',), name='Thread-A')
t2 = threading.Thread(target= process_thread, args=('Bob',), name='Thread-B')
t1.start()
t2.start()
t1.join()
t2.join()
```

### 进程 vs. 线程

* 多进程：稳定性高，子进程崩溃不会影响到主进程和其他子进程，但创建进程的资源开销大；
* 多线程：稳定性较弱，任何一个线程挂掉都可能造成整个进程的崩溃，因为所有线程共享进程的内存，但创建线程开销小；

> 除了多进程与多线程，python还提供了基于事件驱动，采用单线程异步编程模型的多任务处理方式`协程`；

* thread 与 process中，应该优选process，因为考虑到稳定性，且Process可分布到多台设备当中，而thread仅能在单台设备的多个CPU中； 

### 分布式进程
`multiprocessing`不但支持多进程，同时其子模块`managers`也提供将多进程分布到多台设备当中
``` python
# task_master.py

import random, time, queue
from multiprocessing.managers import BaseManager

# 发送任务的队列:
task_queue = queue.Queue()
# 接收结果的队列:
result_queue = queue.Queue()

# 从BaseManager继承的QueueManager:
class QueueManager(BaseManager):
    pass

# 把两个Queue都注册到网络上, callable参数关联了Queue对象:
QueueManager.register('get_task_queue', callable=lambda: task_queue)
QueueManager.register('get_result_queue', callable=lambda: result_queue)
# 绑定端口5000, 设置验证码'abc':
manager = QueueManager(address=('', 5000), authkey=b'abc')
# 启动Queue:
manager.start()
# 获得通过网络访问的Queue对象:
task = manager.get_task_queue()
result = manager.get_result_queue()
# 放几个任务进去:
for i in range(10):
    n = random.randint(0, 10000)
    print('Put task %d...' % n)
    task.put(n)
# 从result队列读取结果:
print('Try get results...')
for i in range(10):
    r = result.get(timeout=10)
    print('Result: %s' % r)
# 关闭:
manager.shutdown()
print('master exit.')
```
``` python
# task_worker.py

import time, sys, queue
from multiprocessing.managers import BaseManager

# 创建类似的QueueManager:
class QueueManager(BaseManager):
    pass

# 由于这个QueueManager只从网络上获取Queue，所以注册时只提供名字:
QueueManager.register('get_task_queue')
QueueManager.register('get_result_queue')

# 连接到服务器，也就是运行task_master.py的机器:
server_addr = '127.0.0.1'
print('Connect to server %s...' % server_addr)
# 端口和验证码注意保持与task_master.py设置的完全一致:
m = QueueManager(address=(server_addr, 5000), authkey=b'abc')
# 从网络连接:
m.connect()
# 获取Queue的对象:
task = m.get_task_queue()
result = m.get_result_queue()
# 从task队列取任务,并把结果写入result队列:
for i in range(10):
    try:
        n = task.get(timeout=1)
        print('run task %d * %d...' % (n, n))
        r = '%d * %d = %d' % (n, n, n*n)
        time.sleep(1)
        result.put(r)
    except Queue.Empty:
        print('task queue is empty.')
# 处理结束:
print('worker exit.')

```

## 正则表达式
Python 提供`re`模块，包含所有正则表达式的功能；
```
import re
# 判断是否匹配
re.match(<正则>, <str>)
# 切割字符串
re.split(<正则>, <str>)
# 分组
m = re.match(r'^(\d{3})-(\d{3,8})$', '010-12345')
m.group(0) # 原始字符串‘010-12345’
m.group(1) # 第一组‘010’
m.group(2) # 第二组‘12345’

```
## 常用内建模块

### datetime
```
from datetime import datetime

# 获取当前日期
now = datetime.now()

# 获取制定日期及时间
dt = datetime(2015, 4, 19, 12, 20) => 2015-04-19 12:20:00

# datetime转换timestamp
dt.timestamp()

# timestamp转换datetime
datetime.fromtimestamp(14294172200.00) => 2015-04-19 12:20:00

# str转换datetime
datetime.strptime('2015-6-1 18:19:59', '%Y-%m-%d %H:%M:%S')

# datetime转换str
now = datetime.now()
now.strftime('%a, %b %d %H:%M')

# datetime加减
now = datetime.now()
now + timedelta(hours=10) # 加10小时
now - timedelta(days=1)   # 减1天
now + timedelta(days=2, hours=12) # 加2天12个小时

```
### collections
集合模块，提供了许多有用的集合类
* namedtuple 命名元组

`namedtuple`本身是个函数，用于创建自定义tuple对象，规定了tuple元素的个数，
并可以使用属性而非索引的方式引用tuple中的元素;
```
from collections import namedtuple
Point = namedtuple('Point', ['x', 'y'])
p = Point(1,2)
p.x => 1
p.y => 2
```
* deque 双向列表

`list`对于查询效率高，但是对于删除及插入等操作效率很低;
`deque`可以实现插入和删除操作双向列表，适合队列和栈;
* defaultdict 默认值字典

使用`dict`时，当引用不存在的key时，会抛出keyError;
使用`defaultdict`可以在引用不存在Key时，提供一个默认值;
```
from collections import defaultdict
dd = defaultdict(lambda: 'N/A')
dd['key1'] = 'abc'
dd['key1']
dd['key2'] => 'N/A'

```
* OrderedDict 有序字典

普通的`dict`的key是无序的，`OrderedDict`是按照插入的顺序排列的，不是按Key本身排序;

* Counter 计数器

```
from collections import Counter
c = Counter
for ch in 'programming':
    c[ch] = c[ch]+1

c => Counter({'g': 2, 'm': 2, 'r': 2, 'a': 1, 'i': 1, 'o': 1, 'n': 1, 'p': 1})
```

### base64

可以将任意二进制转换为文本字符串的编码方式;

```
import base64

base64.b64encode(b'binary\x00string') => b'YmluYXJ5AHN0cmluZw=='
base64.b64decode(b'YmluYXJ5AHN0cmluZw==') => b'binary\x00string'

```
### struct
* Python没有专门处理字节的数据类型，但可以通过b'str'表示字节;
* `struct`模块用于解决`bytes`和其他二进制数据类型的转换;
* struct.pack()可以将任意数据类型转变为`bytes`;
* unpack()把`byte`变成相应的数据类型;
``` py
import struct
# >表示字节顺序是big-endian，也就是网络序，I表示4字节无符号整数
struct.pack('>I', 10240099) => b'\x00\x9c@c'
struct.unpack('>IH', b'\xf0\xf0\xf0\xf0\x80\x80')
```

### hashlib
提供常用的摘要算法，如MD5、SHA1等;
``` py
import hashlib

md5 = hashlib.md5()
md5.update('hello world!'.encode('utf-8'))
md5.hexdigest()
```
### hmac
hmac是在计算哈希摘要的过程中，混入key的算法，这样可以防止通过彩虹表反推原始口令;
``` py
import hmac
msg = b'hello, world!'
key = b'secret'
h = hmac.new(key, msg, digestmod='MD5')
h.hexdigest() => 'fa4ee7d173f2d97ee79022d1a7355bcf'

```

### itertools
主要提供可用于操作迭代对象的工具函数
```
import itertools
# 无限迭代器
natuals = intertools.count(1) # 连续输出自然数
cs = intertools.cycle('ABC')  # 无限循环迭代
ns = itertools.repeat('A', 3) # 将A重复迭代3次

# takewhile()可通过条件判断截取有限个序列
natuals = itertools.count(1)
itertools.takewhile(lambda x: x <= 10, natuals)

# chain()可以将一组迭代对象串联起来，形成一个更大的迭代器
for c in itertools.chain('ABC', 'XYZ'):
    print(c) => 'A' 'B' 'C' 'X' 'Y' 'Z'

# groupby()把迭代器中相邻的重复元素挑出来放在一起
for key, group in itertools.groupby('AAABBBCCAAA'):
```
### contextlib

在关闭资源等操作时, 会用到`try...finally`, 但该方法繁琐, 可以使用`with`;
通过with实现上下文管理是通过`__enter__`和`__exit__`两个方法实现

### urllib
提供了操作URL的一系列功能
### xml
提供XML格式数据的解析
### htmlParser
提供解析HTML格式的文件

## 常用第三方模块
第三方模块可以用pip安装，同时建议安装anaconda,其中包含数十个常用第三方呢模块

### Pillow
图形处理标准库
### requests
由于内置的urllib,提供更加方便的请求访问URL资源的工具
### chardet
在遇到未知的字符编码时，可以使用chardet对编码进行检测确认;
### psutil
用于获取系统信息
``` py
import psutil
psutil.cpu_count() # CPU逻辑数
psutil.cpu_times() # 统计CPU用户以及系统空余时间
psutil.virtual_memory() # 获取物理内存空间
psutil.swap_memory() # 获取交换内存空间
psutil.disk_partitions() # 获取磁盘分区信息
psutil.disk_usage('/') # 磁盘使用情况
psutil.net_io_counters() # 获取网络读写字节／包的个数
psutil.net_if_addrs() # 获取网络接口信息
psutil.net_connections() # 获取当前网络连接信息
psutil.pids() # 所有进程ID
p = psutil.Process(3776) # 获取指定进程ID=3776，其实就是当前Python交互环境
```

## virtualenv 虚拟环境

在开发多个应用程序时，每个应用的运行环境会有不同，
为每个应用创建一套独立且隔离的运行环境, 就可以使用virtualenv

1. 在项目根目录运行虚拟环境,命令：`virtualenv --no-site-packages venv`
2. 在虚拟环境中安装所需要的第三方包（这些包都被安装到venv目录下，不受pyhon环境影响）
3. 退出环境使用`deactivate`命令;


### 技术栈
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




