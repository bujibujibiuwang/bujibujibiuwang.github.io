---
title: Python对象引用机制
date: 2024-11-14 20:00:00 +0800
categories: [Python, FluentPython]
tags: [blog]
---

> 参考：《FluentPython》

## Chapter 8 Object References, Mutability,and Recycling

### (1) Variables Are Not Boxes

下面代码中，变量a和b指向的是同一个列表，而不是列表的复制。修改a实际上是在修改存放在某个地址的列表，而b也指向这个地址，因此b也会发现变化

```python
>>> a = [1, 2, 3]
>>> b = a
>>> a.append(4)
>>> b
[1, 2, 3, 4]
```

![Variables Are Not Boxes](/assets/img/python/fluentpython-p1.png)

在python中，变量不是一个单独的盒子，而是便利贴，是reference variable。对于一个赋值assignment语句，首先读取右边的内容，创建相应的对象object，然后左边的变量variable绑定到这个对象object，就像贴了一张标签

### (2) Identity, Equality, and Aliases

lewis可以看成是charles的别名，从id和is两个函数可以看出，两者绑定的是一个对象。id会返回一个object的内存地址

```python
>>> charles = {'name': 'Charles L. Dodgson', 'born': 1832}
>>> lewis = charles
>>> lewis is charles
True
>>> id(charles), id(lewis)
(140215650472960, 140215650472960)
>>> lewis['balance'] = 950
>>> charles
{'name': 'Charles L. Dodgson', 'born': 1832, 'balance': 950}
```

如果新建alex，即便内容和charles相同，两者也不是同一个object。alex指向的是charles绑定的object的副本，通过==可以判断两者内容相同，通过字典里的__eq__判断的，但是两者是不同的object，通过is判断

```python
>>> alex = {'name': 'Charles L. Dodgson', 'born': 1832}
>>> alex == charles
False
>>> alex is not charles
True
```

#### Choosing Between == and is

```
==: 比较object的值
is：比较object的身份identity
```

大部分情况下，我们更关心object的值，而不是身份，因此python代码里==比is用的更频繁，is比较常用的场景是检查变量是否绑定到None

```python
x is None
```

is操作符比==更快，==要加载object的属性值进行比较，对于大规模的结构比较可能会增加很多处理过程

#### The Relative Immutability of Tuples

元组的不可变性是相对的，元组一旦创建，结构（元素数量和顺序）就不能改变，但是如果元组包含可变对象（如列表或字典），这些可变对象的内容可以被修改，从而导致元组引用的内容发生变化，但元组本身并未改变

```python
>>> t1 = (1, 2, [30, 40])
>>> t2 = (1, 2, [30, 40])
>>> t1 == t2
True
>>> id(t1[-1])
140215650336448
>>> t1[-1].append(99)
>>> id(t1[-1])
140215650336448
>>> t1 == t2
False
>>> t1[-1] = 0
Traceback (most recent call last):
  File "<pyshell#27>", line 1, in <module>
    t1[-1] = 0
TypeError: 'tuple' object does not support item assignment
```

t1和t2是值相同的两个object，t1的第三个元素指向的是可变列表，因此这个列表内容可以变化，变化后t1还是原来的那个object，id没有变化，但是和t2值不同。元组的相对不可变性是指元组绑定的对象不可变，但是如果这个对象本身是可变的，那这个对象的值也可以改变，就会体现在元组的值也改变，而元组本身未改变

### (3) Copies Are Shallow by Default

python中的copy分成两种，一种是浅拷贝shadllow copy，一种是深拷贝deep copy

- 浅拷贝：最外层的容器被复制，但是这个新容器中的元素仍然是对原始容器中元素的引用，因此对浅拷贝中的元素进行修改也会影响到原始容器中的相应元素

- 深拷贝：创建一个新容器的同时，也递归地复制原始容器中的所有元素，新容器及其内部的所有对象都是独立的副本

（1）外层和内层容器的区别

```python
>>> l1 = [3, [66, 55, 44], (7, 8, 9)]
>>> l2 = list(l1)
>>> l1.append(100)
>>> l1[1].remove(55)
>>> l1
[3, [66, 44], (7, 8, 9), 100]
>>> l2
[3, [66, 44], (7, 8, 9)]
```

如代码所示，l2是l1的浅拷贝，最外层的容器是[3，列表，元组]，被复制创建，因此l1.append(100)时只会在l1中改变，这是添加到最外层容器里。而内层的[66, 55, 44]和(7, 8, 9)没有被复制，l1和l2引用的是相同的对象，由于列表是可变的，因此l1[1].remove(55)时l1和l2都会改变

![外层和内层容器的区别](/assets/img/python/fluentpython-p2.png)

（2）内容容器可变和不可变的区别

```python
>>> l1[1] += [33, 22]
>>> l2[2] += (10, 11)
>>> l1
[3, [66, 44, 33, 22], (7, 8, 9), 100]
>>> l2
[3, [66, 44, 33, 22], (7, 8, 9, 10, 11)]
```

列表是可变的，因此l1[1] += [33, 22]后l1和l2都会变化，但是元组不可变，l2[2] += (10, 11)相当于创建了一个新元组(7, 8, 9, 10, 11)，此时l1和l2第三个元素引用的就不是同一个对象，因此l1不会变化

![内容容器可变和不可变的区别](/assets/img/python/fluentpython-p3.png)

#### Deep and Shallow Copies of Arbitrary Objects

python中的copy模块提供了深拷贝deepcopy和浅拷贝copy的实现

```python
>>> import copy
>>> name = [['Alice', 'Bill'], 'Claire', 'David']
>>> name1 = copy.copy(name)
>>> name2 = copy.deepcopy(name)
>>> name[0].append('Mike')
>>> name1
[['Alice', 'Bill', 'Mike'], 'Claire', 'David']
>>> name2
[['Alice', 'Bill'], 'Claire', 'David']
```

### (4) Function Parameters as References

python中的参数传递称为call by sharing（共享传参），参数传递机制有以下方式

- **Call by Value**：函数接收实参的副本，修改副本不会影响原实参

```python
def modify(x):
    x = 10

a = 5
modify(a)
a
Out[5]: 5  # 输出 5，a 并没有被修改
```

- **Call by Reference**：函数接收实参的引用，修改引用会影响原实参

```python
def modify(lst):
    lst.append(10)

a = [1, 2, 3]
modify(a)
a
Out[9]: [1, 2, 3, 10]  # 输出 [1, 2, 3, 10],a被修改
```

- **Call by Sharing**：类似 `call by reference`，但修改的是对象的内容，不会改变引用本身的指向

```python
def modify(lst):
    lst.append(10)   # 修改原对象
    lst = [1, 2, 3]  # 创建新对象，原引用不变

a = [1, 2, 3]
modify(a)
a
Out[13]: [1, 2, 3, 10]. # 输出 [1, 2, 3, 10]，修改了原对象，但引用没有变
```

对于python的call by sharing，如果传入的引用是不可变的，即便函数内部修改了原对象，在外部也保持不变

```python
def f(a, b):
    a += b
    return a
t = (10, 20)
u = (30, 40)
f(t, u)
Out[17]: (10, 20, 30, 40)
t, u
Out[18]: ((10, 20), (30, 40))
```

#### Mutable Types as Parameter Defaults: Bad Idea

由于python中函数参数是共享传递，因此使用可变对象作为参数默认值可能会造成问题

```python
# 定义一个Bus类，参数passengers默认为空列表
class Bus:
    def __init__(self, passengers=[]):
        self.passengers = passengers
    def pick(self, name):
        self.passengers.append(name)
    def drop(self, name):
        self.passengers.remove(name)

# 定义bus，增加成员
bus = Bus()
bus.pick('Alice')
bus.passengers
Out[23]: ['Alice']

# 定义一个新的bus，passengers不为空，与默认设置不符
bus2 = Bus()
bus2.passengers
Out[25]: ['Alice']
```

此时查看Bus初始化对象的默认属性，产生了变化，不再是[]，bus和bus2都没有传入passengers，因此它们的passengers指向的是同一个对象。如果一个默认值是可变对象，只要修改了这个值，这个变化会影响函数未来的所有调用

```python
Bus.__init__.__defaults__
Out[38]: (['Alice'],)
```

#### Defensive Programming with Mutable Parameters

在编写函数时，如果参数是一个可变参数，要认真考虑是否希望该参数被改变，比如传入了一个字典，在函数内部需要修改这个字典，那么修改变化是否需要在外部可见？

- 防止修改类的初始化属性，但是对传入的passengers进行修改，在外部可见

```python
def __init__(self, passengers=None):
    if passengers is None:
        self.passengers = []
    else:
        self.passengers = passengers
```

- 防止修改类的初始化属性，对传入的passengers进行修改，在外部不可见

```python
def __init__(self, passengers=None):
    if passengers is None:
        self.passengers = []
    else:
        self.passengers = list(passengers)  # 从内存上新建一个对象
```

### (5) del and Garbage Collection

> 对象永远不会被显式销毁;但是，当它们变得无法访问时，可能会被垃圾回收

del语句只是删除了名字，而不是对象，但是使用del语句后可能导致某个对象没有引用，或者说该对象无法访问，因此进行了垃圾回收。

在CPython中，垃圾回收garbage collection的主要算法是引用计数reference counting，每个对象都在记录指向它的引用有多少，只要计数为0，该对象就会被立即销毁，CPython调用对象的_del_方法，释放分配在该对象上的内存。

垃圾回收有一个问题是循环引用，即两个对象都指向对方，这样基于引用计数无法销毁这两个对象，造成内存泄露，因此也有些Python实现采用了其它垃圾回收算法。

```python
import weakref
def bye():
    print('garbage collection')
s1 = {1, 2, 3}
s2 = s1
ender = weakref.finalize(s1, bye)
ender.alive
Out[53]: True
del s1
ender.alive
Out[55]: True
s2 = 'spam'
garbage collection
ender.alive
Out[57]: False
```

### (6) Weak References

当某个对象的引用计数为0时，垃圾回收机制会处理该对象，但是有时我们希望引用一个不会让它停留超过必要时间的对象，比如cache。

对某个对象的弱引用(weak reference)不会增加它的引用计数，这种引用称为referent，因此弱引用不会阻止垃圾回收，弱引用在缓存应用程序中很有用，这样缓存对象不会因为被缓存引用而一直保持活动状态

- **不会增加引用计数**：普通的引用（强引用）会增加对象的引用计数，导致对象不会被垃圾回收。而弱引用不会增加引用计数，它允许对象在没有强引用指向它时被垃圾回收。
- **适用于缓存和代理**：弱引用常用于实现缓存和代理模式，当对象不再被使用时，可以自动清除这些对象，而不需要手动干预。

Python 提供了 `weakref` 模块来创建弱引用，此外还有WeakKeyDictionary, WeakValueDictionary, WeakSet等模块，用于存储字典，集合等弱引用对象，不是所有的python对象都可以使用弱引用

```python
import weakref
class MyClass:
    def __init__(self, name):
        self.name = name
    def __repr__(self):
        return f'MyClass {self.name}'

# 创建一个 MyClass 实例   
obj = MyClass('zoey')

# 创建一个弱引用
weak_ref = weakref.ref(obj)

# 弱引用输出obj
print(weak_ref())
MyClass zoey

# 删除强引用
del obj

# 弱引用也删除
print(weak_ref())
None
```

如果想要构建一个类，同时存储所有该类的实例，可以使用WeakSet来存储对实例的弱引用，避免造成循环引用

### (7) Tricks Python Plays with Immutables

对于list, tuple, dict, string等创建别名或者复制，相同的操作有不同的效果

```python
# tuple创建引用
t1 = (1, 2, 3)
t2 = tuple(t1)
t2 is t1
Out[82]: True

# list创建复制
l1 = [1, 2, 3]
l2 = list(l1)
l2 is l1
Out[85]: False

# dict创建复制
d1 = {1: 'a', 2: 'b', 3:'c'}
d2 = dict(d1)
d2 is d1
Out[90]: False
```

```python
# 元组的[:]创建引用，类似的还有str, bytes和frozenset
t1 = (1, 2, 3)
t2 = t1[:]
t2 is t1
Out[93]: True

# 列表的[:]创建复制
l1 = [1, 2, 3]
l2 = l1[:]
l2 is l1
Out[96]: False
```

对于字符串，有一个优化技术称为字符串驻留(string interning)，Cpython使用该技术处理一些小的整数（0，-1等）的内存分配，通过减少字符串对象的重复创建来提高内存使用效率和程序性能

```python
# t1和t2值相同，但是对象不同
t1 = (1, 2, 3)
t2 = (1, 2, 3)
t2 is t1
Out[107]: False

# s2和s1值相同，对象也相同
s1 = 'abc'
s2 = 'abc'
s2 is s1
Out[104]: True
```

### Summary

> Every Python object has an identity, a type, and a value. Only the value of an object changes over time

- 简单的赋值没有创建副本

- 如果左边变量时不可变对象，增强赋值(+=, *=)会创建新的对象

- 为一个已经存在的变量赋新值，不会改变绑定到这个变量的对象，称为rebinding

- 函数参数作为引用传入

- 将可变对象作为函数参数的默认值传入是危险的

- python垃圾回收机制是基于引用计数

- 如果需要循环引用，可以使用弱引用
