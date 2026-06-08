## 4.第一个python 程序
### 直接运行py文件
在Mac和Linux上在`.py`文件的第一行加上一个特殊的注释：

```python
#!/usr/bin/env python3
```
后，通过命令给`hello.py`以执行权限：

```plain
$ chmod a+x hello.py
```

就可以直接运行`hello.py`了
```python
$ ./hello.py
hello, world
```
## 5.python基础
###  数据类型和变量
转义字符`\`可以转义很多字符，比如`\n`表示换行，`\t`表示制表符，字符`\`本身也要转义，所以`\\`表示的字符就是`\`
如果字符串里面有很多字符都需要转义，就需要加很多`\`，为了简化，Python还允许用`r''`表示`''`内部的字符串默认不转义
### 字符串编码

* Python的字符串类型是`str`，在内存中以Unicode表示，一个字符对应若干个字节。如果要在网络上传输，或者保存到磁盘上，就需要把`str`变为以字节为单位的`bytes`。
 * Python对`bytes`类型的数据用带`b`前缀的单引号或双引号表示：

```python
x = b'ABC'
```
 * 以Unicode表示的`str`通过`encode()`方法可以编码为指定的`bytes`，例如
```python
>>> 'ABC'.encode('ascii')
b'ABC'
>>> '中文'.encode('utf-8')
b'\xe4\xb8\xad\xe6\x96\x87'
>>> '中文'.encode('ascii')
```
* `len()`函数计算的是`str`的字符数，如果换成`bytes`，`len()`函数就计算字节数

#### f-string
```plain
>>> r = 2.5
>>> s = 3.14 * r ** 2
>>> print(f'The area of a circle with radius {r} is {s:.2f}')
The area of a circle with radius 2.5 is 19.62
```

上述代码中，`{r}`被变量`r`的值替换，`{s:.2f}`被变量`s`的值替换，并且`:`后面的`.2f`指定了格式化参数（即保留两位小数），因此，`{s:.2f}`的替换结果是`19.62`。
### 5.3使用list 和tuple 



### 1. 添加元素（增）

| 方法 | 语法 | 说明 | 示例 |
| :--- | :--- | :--- | :--- |
| `append()` | `list.append(x)` | 在列表**末尾**添加一个元素 `x` | `L.append('Adam')` |
| `insert()` | `list.insert(index, x)` | 在**指定索引** `index` 处插入元素 `x` | `L.insert(1, 'Jack')` |
| `extend()` | `list.extend(iterable)` | 将另一个可迭代对象的所有元素**追加**到当前列表末尾 | `L.extend(['a', 'b'])` |

---

#### 2. 删除元素（删）

| 方法 | 语法 | 说明 | 示例 |
| :--- | :--- | :--- | :--- |
| `pop()` | `list.pop([index])` | **删除并返回**指定索引的元素。默认删除**最后一个**元素 | `L.pop()` (删末尾)<br>`L.pop(1)` (删索引 1) |
| `remove()` | `list.remove(x)` | 删除列表中**第一个**值为 `x` 的元素。如果不存在会报错 | `L.remove('Bob')` |
| `clear()` | `list.clear()` | **清空**列表中的所有元素，使其变成空列表 `[]` | `L.clear()` |

---

#### 3. 查找与统计（查）

| 方法 / 函数 | 语法 | 说明 | 示例 |
| :--- | :--- | :--- | :--- |
| `len()` | `len(list)` | **内置函数**（非方法），返回列表的元素总个数 | `len(classmates)` |
| `index()` | `list.index(x)` | 返回第一个值为 `x` 的元素的**索引位置**，不存在则报错 | `L.index('Sarah')` |
| `count()` | `list.count(x)` | 统计元素 `x` 在列表中**出现的次数** | `L.count('Apple')` |
| `in` | `x in list` | 关键字，判断元素 `x` **是否在列表中**，返回布尔值 | `'Michael' in L` |

---

####  4. 排序与反转（改）

以下两个方法都会**直接修改原列表**，它们的返回值都是 `None`。

| 方法 | 语法 | 说明 | 示例 |
| :--- | :--- | :--- | :--- |
| `reverse()` | `list.reverse()` | **反转**列表中的所有元素顺序 | `L.reverse()` |
| `sort()` | `list.sort(reverse=False)` | 对列表进行**排序**。默认升序，若设置 `reverse=True` 则降序 | `L.sort(reverse=True)` |

---

要定义一个只有1个元素的tuple，如果你这么定义：
```python
>>> t = (1)
>>> t
1
```

定义的不是tuple，是`1`这个数！这是因为括号`()`既可以表示tuple，又可以表示数学公式中的小括号，这就产生了歧义，因此，Python规定，这种情况下，按小括号进行计算，计算结果自然是`1`。

所以，只有1个元素的tuple定义时必须加一个逗号`,`，来消除歧义：

```plain
>>> t = (1,)
>>> t
(1,)
```

### 5.4条件判断
input()返回的数据类型是str

### 5.5 模式匹配
如果用`match`语句改写，则改写如下：

```python
score = 'B'

match score:
    case 'A':
        print('score is A.')
    case 'B':
        print('score is B.')
    case 'C':
        print('score is C.')
    case _: # _表示匹配到其他任何情况
        print('score is ???.')
```

使用`match`语句时，我们依次用`case xxx`匹配，并且可以在最后（且仅能在最后）==加一个`case _`表示“任意值”，==代码较`if ... elif ... else ...`更易读。

### 5.7 使用dict 和set
正确使用dict非常重要，需要牢记的第一条就是dict的key必须是**不可变对象**
		这是因为dict根据key来计算value的存储位置，如果每次计算相同的key得出的结果不同，那dict内部就完全混乱了。这个通过key计算位置的算法称为哈希算法（Hash）。要保证hash的正确性，作为key的对象就不能变。在Python中，字符串、整数等都是不可变的，因此，可以放心地作为key。而list是可变的，就不能作为key


set 的常用方法
1. 通过`add(key)`方法可以添加元素到set中
2. set(list)作为输入集合构建一个set
3. `remove(key)`来删除元素
4. 
* set和dict的唯一区别仅在于没有存储对应的value，但是，set的原理和dict一样，所以，同样不可以放入可变对象，因为无法判断两个可变对象是否相等，也就无法保证set内部“不会有重复元素”。
==对于不变对象来说，调用对象自身的任意方法，也不会改变该对象自身的内容。相反，这些方法会创建新的对象并返回，这样，就保证了不可变对象本身永远是不可变的。==

## 函数
### 6.1 调用函数
可以在交互式命令行通过`help(函数名)` 查看函数的帮助信息

### 6.2定义函数
如果你已经把`my_abs()`的函数定义保存为`abstest.py`文件了，那么，可以在该文件的当前目录下启动Python解释器，用`from abstest import my_abs`来导入`my_abs()`函数，注意`abstest`是文件名

函数可以返回多个值，但其实这只是一种假象，Python函数返回的仍然是单一值，是将返回值作为一个tuple进行返回

传入了不恰当的参数时，内置函数`abs`会检查出参数错误，而我们定义的`my_abs`没有参数检查，会导致`if`语句出错，出错信息和`abs`不一样。所以，这个函数定义不够完善。

让我们修改一下`my_abs`的定义，对参数类型做检查，只允许整数和浮点数类型的参数。数据类型检查可以用内置函数`isinstance()`实现：

```python
def my_abs(x):
    if not isinstance(x, (int, float)):
        raise TypeError('bad operand type')
    if x >= 0:
        return x
    else:
        return -x
```

添加了参数检查后，如果传入错误的参数类型，函数就可以抛出一个错误：

```plain
>>> my_abs('A')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 3, in my_abs
TypeError: bad operand type
```
### 6.3函数的参数
#### 普通参数又叫位置参数
#### 默认参数有注意事项：必选参数在前，默认参数在后
==注意：定义默认参数要牢记一点：默认参数必须指向不变对象！==
```python
def add_end(L=[]):
    L.append('END')
    return L
```
当你使用默认参数调用时，一开始结果也是对的：

```python
>>> add_end()
['END']
```

但是，再次调用`add_end()`时，结果就不对了：

```python
>>> add_end()
['END', 'END']
>>> add_end()
['END', 'END', 'END']
```
***
原因是：Python函数在定义的时候，默认参数`L`的值就被计算出来了，即`[]`，因为默认参数`L`也是一个变量，它指向对象`[]`，每次调用该函数，如果改变了`L`的内容，则下次调用时，默认参数的内容就变了，不再是函数定义时的`[]`了。



#### 可变参数
定义可变参数即将行参前加一个`*`，此时函数接收到的是一个tuple，从而在调用该函数时可以传入任意个参数，包括0
如果想传入一个list或tuple 进去，可以将list 前一个`*`号从而将list里的元素传入
例如：`*nums`表示把`nums`这个list的所有元素作为可变参数传进去。这种写法相当有用，而且很常见。

#### 关键字参数
与可变参数不同，可变参数将传入的参数自动拼成一个tuple ，而关键字参数则是允许将参数名和参数值一起传入组成一个dic

#### 命名关键字参数
由于关键字参数可以传入不受限制的变量名，此时就到了命名关键字参数的优势地方
和关键字参数`**kw`不同，命名关键字参数需要一个特殊分隔符`*`，`*`后面的参数被视为命名关键字参数。
如果函数定义中已经有了一个可变参数，后面跟着的命名关键字参数就不再需要一个特殊分隔符`*`了：

```python
def person(name, age, *args, city, job):
    print(name, age, args, city, job)
```

## 7.高级特性
### 7.1切片
[begin:end:间隔]
左闭右开
什么都不写，只写`[:]`就可以原样复制一个list

### 7.2迭代
只要是可迭代对象，无论有无下标，都可以迭代，比如`dict`就可以迭代
判断一个对象是可迭代对象可以通过`collections.abc`模块的`Iterable`类型判断：

```plain
>>> from collections.abc import Iterable
>>> isinstance('abc', Iterable) # str是否可迭代
True
>>> isinstance([1,2,3], Iterable) # list是否可迭代
True
>>> isinstance(123, Iterable) # 整数是否可迭代
False
```

要对`list`实现类似Java那样的下标循环可以使用Python内置的`enumerate`函数可以把一个`list`变成索引-元素对，这样就可以在`for`循环中同时迭代索引和元素本身：

### 7.3 列表生成式
在一个列表生成式中，`for`前面的`if ... else`是表达式，而`for`后面的`if`是过滤条件，不能带`else`。

### 7.4生成器
如果列表元素可以按照某种算法推算出来，那我们是否可以在循环的过程中不断推算出后续的元素呢？这样就不必创建完整的list，从而节省大量的空间。在Python中，这种一边循环一边计算的机制，称为生成器：generator。
创建生成器的方法
	1. 将列表生成器的[]改为（）
	2. 如果一个函数中包含`yield` 关键字

generator保存的是算法
	可以通过`next()`函数获得generator的下一个返回值
	因为generator 也是可迭代对象，也可以使用for循环


不同点：
普通函数是顺序执行，遇到`return`语句或者最后一行函数语句就返回。
变成generator的函数，在每次调用`next()`的时候执行，遇到`yield`语句返回，再次执行时从上次返回的`yield`语句处继续执行。

==调用generator函数会创建一个generator对象，多次调用generator函数会创建多个相互独立的generator。==
```python
def odd():
    print('step 1')
    yield 1
    print('step 2')
    yield(3)
    print('step 3')
    yield(5)
```

```plain
>>> next(odd())
step 1
1
>>> next(odd())
step 1
1
>>> next(odd())
step 1
1
```

原因在于`odd()`会创建一个新的generator对象，上述代码实际上创建了3个完全独立的generator，对3个generator分别调用`next()`当然每个都会返回第一个值。

正确的写法是创建一个generator对象，然后不断对这一个generator对象调用`next()`：

```plain
>>> g = odd()
>>> next(g)
step 1
1
>>> next(g)
step 2
3
>>> next(g)
step 3
5
```


### 7.5迭代器
[参考链接](https://composingprograms.netlify.app/4/2)
可以直接作用于`for`循环的数据类型有以下几种：

一类是集合数据类型，如`list`、`tuple`、`dict`、`set`、`str`等；

一类是`generator`，包括生成器和带`yield`的generator function。
（同时generator也是Iterator 的子集）
这些可以直接作用于`for`循环的对象统称为可迭代对象：`Iterable`
可以被`next()`函数调用并不断返回下一个值的对象称为迭代器：`Iterator`、这是可迭代对象的子集
迭代器抽象有两个组件：

- 检索下一个元素的机制
- 到达序列末尾并且没有剩余元素，发出信号的机制
对于任何容器，例如 `list` 或 `range`，都可以通过调用内置的 `iter` 函数来获取迭代器。使用内置的 `next` 函数来访问迭代器的内容。

在迭代器上调用 `iter` 将返回该迭代器，而不是其副本。 Python 中包含此行为，以便程序员可以对某个值调用 `iter` 来获取迭代器，而不必担心它是迭代器还是容器

## 8.函数式编程
### 8.1 高阶函数
函数名其实就是指向函数的变量！对于`abs()`这个函数，完全可以把函数名`abs`看成变量，它指向一个可以计算绝对值的函数
所谓高阶函数，就是让函数的参数能够接收别的函数。
#### map函数
`map()`函数接收两个参数，一个是函数，一个是`Iterable`，`map`将传入的函数依次作用到序列的每个元素，并把结果作为新的`Iterator`返回。
为了让map里元素全部输出，可以使用list(map(f,s))来全部输出

#### reduce 函数
`reduce`把一个函数作用在一个序列`[x1, x2, x3, ...]`上，这个函数必须接收两个参数，`reduce`把结果继续和序列的下一个元素做累积计算
#### filter函数
`filter()`也接收一个函数和一个序列。和`map()`不同的是，`filter()`把传入的函数依次作用于每个元素，然后根据返回值是`True`还是`False`决定保留还是丢弃该元素。
```python
def is_odd(n):
    return n % 2 == 1

list(filter(is_odd, [1, 2, 4, 5, 6, 9, 10, 15]))
# 结果: [1, 5, 9, 15]
```
`filter()`这个高阶函数，关键在于正确实现一个“筛选”函数。

#### sorted函数
sorted(iterable, cmp=None, key=None, reverse=False)、
- iterable -- 可迭代对象。
- cmp -- 比较的函数，这个具有两个参数，参数的值都是从可迭代对象中取出，此函数必须遵守的规则为，大于则返回1，小于则返回-1，等于则返回0。
- key -- 主要是用来进行比较的元素，只有一个参数，具体的函数的参数就是取自于可迭代对象中，指定可迭代对象中的一个元素来进行排序。
- reverse -- 排序规则，reverse = True 降序 ， reverse = False 升序（默认）。

### 返回函数


### 8.3匿名函数
`lambda`表示匿名函数。冒号前面的`x`表示函数参数。匿名函数有个限制，就是只能有一个表达式，不用写`return`，==返回值就是该表达式的结果==
用匿名函数有个好处，因为函数没有名字，不必担心函数名冲突

### 装饰器
本质上，decorator就是一个返回函数的高阶函数。

### 偏函数
简单总结`functools.partial`的作用就是，把一个函数的某些参数给固定住（也就是设置默认值），返回一个新的函数，调用这个新函数会更简单。
```python
>>> import functools
>>> int2 = functools.partial(int, base=2)
>>> int2('1000000')
64
>>> int2('1010101')
85
```

## 9.模块
任何模块代码的第一个字符串都被视为模块的文档注释
使用`sys`模块的第一步，就是导入该模块：

```python
import sys
```

导入`sys`模块后，我们就有了变量`sys`指向该模块，利用`sys`这个变量，就可以访问`sys`模块的所有功能。

```python
if __name__=='__main__':
    test()
```

当我们在命令行运行`hello`模块文件时，Python解释器把一个特殊变量`__name__`置为`__main__`，而如果在其他地方导入该`hello`模块时，`if`判断将失败，因此，这种`if`测试可以让一个模块通过命令行运行时执行一些额外的代码，最常见的就是运行测试。
类似`__xxx__`这样的变量是特殊变量，可以被直接引用，但是有特殊用途
类似`_xxx`和`__xxx`这样的函数或变量就是非公开的（private），不应该被直接引用


## 面向对象编程
由于类可以起到模板的作用，因此，可以在创建实例的时候，把一些我们认为必须绑定的属性强制填写进去。通过定义一个特殊的`__init__`方法，在创建实例的时候，就把`name`，`score`等属性绑上去：

```python
class Student(object):
    def __init__(self, name, score):
        self.name = name
        self.score = score
```
注意到`__init__`方法的第一个参数永远是`self`，表示创建的实例本身，因此，在`__init__`方法内部，就可以把各种属性绑定到`self`，因为`self`就指向创建的实例本身。
和普通的函数相比，在类中定义的函数只有一点不同，就是第一个参数永远是实例变量`self`，并且，调用时，不用传递该参数。除此之外，类的方法和普通函数没有什么区别，所以，你仍然可以用默认参数、可变参数、关键字参数和命名关键字参数。

- 在Python中，实例的变量名如果以`__`开头，就变成了一个私有变量（private），只有内部可以访问，外部不能访问
- 双下划线开头的实例变量是不是一定不能从外部访问呢？其实也不是。不能直接访问`__name`是因为Python解释器对外把`__name`变量改成了`_Student__name`，所以，仍然可以通过`_Student__name`来访问`__name`变量：
- Python中，变量名类似`__xxx__`的，也就是以双下划线开头，并且以双下划线结尾的，是特殊变量，特殊变量是可以直接访问的

### 继承
优势
1. 最大的好处是子类获得了父类的全部功能
2. 当子类和父类都存在相同的方法时，子类的的方法就覆盖了父类的方法。这样，我们就获得了继承的另一个好处：多态。

	多态的优势：当我们需要传入多种子类时，我们只需要接收父类就可以了，因为`Dog`、`Cat`、`Tortoise`……都是`Animal`类型，然后，按照`Animal`类型进行操作即可。由于`Animal`类型有`run()`方法，因此，传入的任意类型，只要是`Animal`类或者子类，就会自动调用实际类型的`run()`方法，这就是多态的意思
对于一个变量，我们只需要知道它是`Animal`类型，无需确切地知道它的子类型，就可以放心地调用`run()`方法，而具体调用的`run()`方法是作用在`Animal`、`Dog`、`Cat`还是`Tortoise`对象上，由运行时该对象的确切类型决定，==这就是多态真正的威力：调用方只管调用，不管细节，而当我们新增一种`Animal`的子类时，只要确保`run()`方法编写正确，不用管原来的代码是如何调用的。这就是著名的“开闭”原则：

	对扩展开放：允许新增`Animal`子类；
	
	对修改封闭：不需要修改依赖`Animal`类型的`run_twice()`等函数。

### 获取对象信息
判断对象的类型可以使用`type()`函数
```python
>>> type(123)
<class 'int'>
>>> type('str')
<class 'str'>
>>> type(None)
<type(None) 'NoneType'>
```

对于继承关系来说，想要判断class类型可以使用`isinstance`函数
isinstance(值，类型)
返回true ｜｜false ，值如果是子类也返回true

果要获得一个对象的所有属性和方法，可以使用`dir()`函数，它返回一个包含字符串的list，比如，获得一个str对象的所有属性和方法：

```python
>>> dir('ABC')
['__add__', '__class__',..., '__subclasshook__', 'capitalize', 'casefold',..., 'zfill']
```

类似`__xxx__`的属性和方法在Python中都是有特殊用途的，比如`__len__`方法返回长度。在Python中，如果你调用`len()`函数试图获取一个对象的长度，实际上，在`len()`函数内部，它自动去调用该对象的`__len__()`方法，所以，下面的代码是等价的：

```plain
>>> len('ABC')
3
>>> 'ABC'.__len__()
3
```