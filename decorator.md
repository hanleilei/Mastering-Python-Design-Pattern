# 装饰器模式

无论何时我们想对一个对象添加额外的功能，都有下面这些不同的可选方法。

* 如果合理，可以直接将功能添加到对象所属的类（例如，添加一个新的方法）
* 使用组合
* 使用继承

注意，本文中的Decorator可以为装饰器或者修饰器。

与继承相比，通常应该优先选择组合，因为继承使得代码更难复用，继承关系是静态的，并且应用于整个类以及这个类的所有实例（请参考[GOF95，第31页]和网页[t.cn/RqrC8Yo]）。

设计模式为我们提供第四种可选方法，以支持动态地（运行时）扩展一个对象的功能，这种方法就是修饰器。修饰器（Decorator）模式能够以透明的方式（不会影响其他对象）动态地将功能添加到一个对象中（请参考[GOF95，第196页]）。

在许多编程语言中，使用子类化（继承）来实现修饰器模式（请参考[GOF95，第198页]）。在Python中，我们可以（并且应该）使用内置的修饰器特性。一个Python修饰器就是对Python语法的一个特定改变，用于扩展一个类、方法或函数的行为，而无需使用继承。从实现的角度来说，Python修饰器是一个可调用对象（函数、方法、类），接受一个函数对象fin作为输入，并返回另一个函数对象fout（请参考[网页]( https://pythonconquerstheuniverse.wordpress.com/2012/04/29/python-decorators/)）。这意味着可以将任何具有这些属性的可调用对象当作一个修饰器。在第1章和第2章中已经看到如何使用内置的property修饰器让一个方法表现为一个变量。在5.4节，我们将学习如何实现及使用我们自己的修饰器。

修饰器模式和Python修饰器之间并不是一对一的等价关系。Python修饰器能做的实际上比修饰器模式多得多，其中之一就是实现修饰器模式（请参考[Eckel08，第59页]和网页[t.cn/RqrlLcQ]）。


```python
#!/usr/bin/env python
"""https://docs.python.org/2/library/functools.html#functools.wraps"""
"""https://stackoverflow.com/questions/739654/how-can-i-make-a-chain-of-function-decorators-in-python/739665#739665"""

from functools import wraps


def makebold(fn):
    return getwrapped(fn, "b")


def makeitalic(fn):
    return getwrapped(fn, "i")


def getwrapped(fn, tag):
    @wraps(fn)
    def wrapped():
        return "<%s>%s</%s>" % (tag, fn(), tag)
    return wrapped


@makebold
@makeitalic
def hello():
    """a decorated hello world"""
    return "hello world"

if __name__ == '__main__':
    print('result:{}   name:{}   doc:{}'.format(hello(), hello.__name__, hello.__doc__))

### OUTPUT ###
# result:<b><i>hello world</i></b>   name:hello   doc:a decorated hello world
```

    result:<b><i>hello world</i></b>   name:hello   doc:a decorated hello world



```python
# http://stackoverflow.com/questions/3118929/implementing-the-decorator-pattern-in-python


class foo(object):
    def f1(self):
        print("original f1")

    def f2(self):
        print("original f2")


class foo_decorator(object):
    def __init__(self, decoratee):
        self._decoratee = decoratee

    def f1(self):
        print("decorated f1")
        self._decoratee.f1()

    def __getattr__(self, name):
        return getattr(self._decoratee, name) # 这个不是delegation么

u = foo()
v = foo_decorator(u)
v.f1()
v.f2()


```

    decorated f1
    original f1
    original f2


## 现实中的例子

该模式虽名为修饰器，但这并不意味着它应该只用于让产品看起来更漂亮。修饰器模式通常用于扩展一个对象的功能。这类扩展的实际例子有，给枪加一个消音器、使用不同的照相机镜头（在可拆卸镜头的照相机上）等。

下图由sourcemaking.com提供，展示了我们可以如何使用一些专用配件来修饰一把枪，使其 无声、更准以及更具破坏力（请参考网页[t.cn/RqrC8Yo]）。注意，图中使用了子类化，但是在 Python中，这并不是必需的，因为可以使用语言内置的修饰器特性。

## 软件中的例子

Django框架大量地使用修饰器，其中一个例子是视图修饰器。Django的视图（View）修饰器 可用于以下几种用途（请参考网页[t.cn/RqrlJbA]）。

* 限制某些HTTP请求对视图的访问控制特定视图上的缓存行为
* 按单个视图控制压缩
* 基于特定HTTP请求头控制缓存

Grok框架也使用修饰器来实现不同的目标，比如下面几种情况。
* 将一个函数注册为事件订阅者
* 以特定权限保护一个方法
* 实现适配器模式

## 应用案例

当用于实现横切关注点（cross-cutting concerns）时，修饰器模式会大显神威（请参考[Lott14，第223页]和网页[t.cn/Rqrl6O0]）。以下是横切关注点的一些例子。
* 数据校验
* 事务处理（这里的事务类似于数据库事务，意味着要么所有步骤都成功完成，要么事务失败） 缓存
* 日志
* 监控
* 调试
* 业务规则
* 压缩
* 加密

一般来说，应用中有些部件是通用的，可应用于其他部件，这样的部件被看作横切关注点。

使用修饰器模式的另一个常见例子是图形用户界面（Graphical User Interface，GUI）工具集。在一个GUI工具集中，我们希望能够将一些特性，比如边框、阴影、颜色以及滚屏，添加到单个组件/部件。

## 实现

Python修饰器通用并且非常强大。你可以在Python官网python.org的修饰器代码库页面（请参考网页[t.cn/zRHPIq4]）中找到许多修饰器的使用样例。本节中，我们将学习如何实现一个memoization修饰器（请参考网页[t.cn/zQi9AET]）。所有递归函数都能因memoization而提速，那么来试试常用的斐波那契数列例子。使用递归算法实现斐波那契数列，直接了当，但性能问题较大，即使对于很小的数值也是如此。首先来看看朴素的实现方法（文件fibonacci_naive.py）。


```python
def fibonacci(n):
    assert(n >= 0), 'n must be >= 0'
    return n if n in (0, 1) else fibonacci(n-1) + fibonacci(n-2)

if __name__ == '__main__':
    from timeit import Timer
    t = Timer('fibonacci(8)', 'from __main__ import fibonacci')
    print(t.timeit())
```

    15.40320448600687


执行一下这个例子就知道这种实现的速度有多慢了。计算第8个斐波那契数要花费运行的样例输出如上所示。

使用memoization方法看看能否改善。在下面的代码中，我们使用一个dict来缓存斐波那契 数列中已经计算好的数值，同时也修改传给fabonacci()函数的参数，计算第100个斐波那契数， 而不是第8个。


```python
known = {0:0, 1:1}
def fibonacci(n):
    assert(n >= 0), 'n must be >= 0'
    if n in known:
        return known[n]
    res = fibonacci(n-1) + fibonacci(n-2)
    known[n] = res
    return res

if __name__ == '__main__':
    from timeit import Timer
    t = Timer('fibonacci(100)', 'from __main__ import fibonacci')
    print(t.timeit())
```

    0.30129148002015427


执行基于memoization的代码实现，可以看到性能得到了极大的提升，甚至对于计算大的数 值性能也是可接受的。运行的样例输出如上所示。

但这种方法有一些问题。虽然性能不再是一个问题，但代码也没有不使用memoization时那 样简洁。如果我们决定扩展代码，加入更多的数学函数，并将其转变成一个模块，那又会是什么 样的呢?假设决定加入的下一个函数是nsum()，该函数返回前n个数字的和。注意这个函数已存 在于math模块中，名为fsum()，但我们也能很容易就能想到标准库中还没有、但是对我们模块 有用的其他函数（例如，帕斯卡三角形、埃拉托斯特尼筛法等）。所以暂且不必在意示例函数是 否已存在。使用memoization实现nsum()函数的代码如下所示。


```python
known_sum = {0:0}
def nsum(n):
    assert(n >= 0), 'n must be >= 0'
    if n in known_sum:
        return known_sum[n]
    res = n + nsum(n-1)
    known_sum[n] = res
    return res
```

你有没有注意到其中的问题?多了一个名为known_sum的新字典，为nsum提供缓存作用， 并且函数本身也比不使用memoization时的更复杂。这个模块逐步变得不必要地复杂。保持递归 函数与朴素版本的一样简单，但在性能上又能与使用memoization的函数相近，这可能吗?幸运 的是，确实可能，解决方案就是使用修饰器模式。

首先创建一个如下面的例子所示的memoize()函数。这个修饰器接受一个需要使用 memoization的函数fn作为输入，使用一个名为known的dict作为缓存。函数functools.wraps() 是一个为创建修饰器提供便利的函数;虽不强制，但推荐使用，因为它能保留被修饰函数的文档字符串和签名（请参考网页[t.cn/Rqrl0K5]）。这种情况要求参数列表*args，因为被修饰的函数可能有输入参数。如果fibonacci()和nsum()不需要任何参数，那么使用*args确实是多余的，但它 们是需要参数n的。


```python
from functools import wraps
def memoize(fn):
    known = dict()
    @wraps(fn)
    def memoizer(*args):
        if args not in known:
            known[args] = fn(*args)
        return known[args]
    return memoizer
```

现在，对朴素版本的函数应用memoize()修饰器。这样既能保持代码的可读性又不影响性能。 我们通过修饰（或修饰行）来应用一个修饰器。修饰使用@name语法，其中name是指我们想要使 用的修饰器的名称。这其实只不过是一个简化修饰器使用的语法糖。我们甚至可以绕过这个语法 手动执行修饰器，留给你作为练习吧。来看看下面的例子中如何对我们的递归函数使用memoize() 修饰器。


```python
@memoize
def nsum(n):
    '''返回前n个数字的和'''
    assert(n >= 0), 'n must be <= 0'
    return 0 if n == 0 else n + nsum(n-1)

@memoize
def fibonacci(n):
    '''返回斐波那契数列的第n个数'''
    assert(n >= 0), 'n must be >= 0'
    return n if n in (0, 1) else fibonacci(n-1) + fibonacci(n-2)
```

代码的最后一部分展示如何使用被修饰的函数，并测量其性能。measure是一个字典列表，用于避免代码重复。注意__name__和__doc__分别是如何展示正确的函数名称和文档字符串值的。尝试从memoize()中删除@functools.wraps(fn)修饰，看看是否仍旧如此。


```python
if __name__ == '__main__':
    from timeit import Timer
    measure = [ {'exec':'fibonacci(100)', 'import':'fibonacci', 'func':fibonacci},{'exec':'nsum(200)', 'import':'nsum', 'func':nsum} ]
    for m in measure:
            t = Timer('{}'.format(m['exec']), 'from __main__ import {}'.format(m['import']))
            print('name: {}, doc: {}, executing: {}, time: {}'.format(m['func'].__name__, m['func'].__doc__, m['exec'], t.timeit()))
```

    name: fibonacci, doc: 返回斐波那契数列的第n个数, executing: fibonacci(100), time: 0.29140055197058246
    name: nsum, doc: 返回前n个数字的和, executing: nsum(200), time: 0.3004333569551818


看看我们数学模块的完整代码（文件mymath.py）和执行时的样例输出。


```python
from functools import wraps

def memoize(fn):
    known = dict()
    @wraps(fn)
    def memoizer(*args):
        if args not in known:
            known[args] = fn(*args)
        return known[args]
    return memoizer

@memoize
def nsum(n):
    '''返回前n个数字的和'''
    assert(n >= 0), 'n must be <= 0'
    return 0 if n == 0 else n + nsum(n-1)

@memoize
def fibonacci(n):
    '''返回斐波那契数列的第n个数'''
    assert(n >= 0), 'n must be >= 0'
    return n if n in (0, 1) else fibonacci(n-1) + fibonacci(n-2)

if __name__ == '__main__':
    from timeit import Timer
    measure = [ {'exec':'fibonacci(100)', 'import':'fibonacci', 'func':fibonacci},{'exec':'nsum(200)', 'import':'nsum', 'func':nsum} ]
    for m in measure:
            t = Timer('{}'.format(m['exec']), 'from __main__ import {}'.format(m['import']))
            print('name: {}, doc: {}, executing: {}, time: {}'.format(m['func'].__name__, m['func'].__doc__, m['exec'], t.timeit()))
```

    name: fibonacci, doc: 返回斐波那契数列的第n个数, executing: fibonacci(100), time: 0.272907609003596
    name: nsum, doc: 返回前n个数字的和, executing: nsum(200), time: 0.2719842789811082


不错!这一方案同时具备可读的代码和可接受的性能。此时，你可能想争论说这不是修饰器 模式，因为我们并不是在运行时应用它。被修饰的函数确实无法取消修饰，但仍然可以在运行时 决定是否执行修饰器。这个有趣的练习就留给你来完成吧。

使用修饰器进行一层额外的封装，基于某个条件来决定是否执行真正的修 饰器。

修饰器的另一个有趣的特性是可以使用多个修饰器来修饰一个函数。本章没有涉及这一特 性，因此这是另一个练习，创建一个修饰器来帮助你调试递归函数，并将其应用于nsum()和 fibonacci()。多个修饰器会以什么次序执行?

如果你仍未充分理解修饰器，那么我有最后一个练习留给你。修饰器memoize()无法修饰接 受多个参数的函数。我们如何可以验证这一点?验证之后，尝试找到一种方法解决这个问题: 经测试，memoize()对多参函数仍然有效。（此处可能有误）

## 小结

本章介绍了修饰器模式及其与Python编程语言的关联。我们使用修饰器模式来扩展一个对象的行为，无需使用继承，非常方便。Python进一步扩展了修饰器的概念，允许我们无需使用继承或组 合就能扩展任意可调用对象（函数、方法或类）的行为。我们可以使用Python内置的修饰器特性。

我们看了现实中一些被修饰对象的例子，比如枪和照相机。从软件的视角来看，Django和Grok都使用了修饰器来达到不同的目标，比如控制HTTP压缩和缓存。
修饰器模式是实现横切关注点的绝佳方案，因为横切关注点通用但不太适合使用面向对象编 程范式来实现。在5.3节中我们提到很多种横切关注点。事实上，5.4节演示了一个横切关注点， memoization。我们看到修饰器如何可以帮助我们保持函数简洁，同时不牺牲性能。

本章中推荐的练习可以帮助你更好地理解修饰器，这样你就能将这一强大工具用于解决许多 常见的（或许不太常见的）编程问题。第6章将介绍外观模式，一种简化复杂系统访问的方式。


```python

```

个人读后感，好烂的一章，完全就是凑字数，还不如干脆挑明了直接解释传统意义上的装饰器模式和python的装饰器之间的差别，还有自己造了一个轮子：memorize，其实我们完全可以使用现有的轮子： from functools import lru_cache，还是别自己造轮子了。。

我后续会补充完整这方面的内容。
