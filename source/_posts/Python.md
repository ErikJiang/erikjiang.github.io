
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
* 

### 文档测试


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




