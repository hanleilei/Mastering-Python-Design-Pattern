
# 享元模式

由于对象创建的开销，面向对象的系统可能会面临性能问题。性能问题通常在资源受限的嵌入式系统中出现，比如智能手机和平板电脑。大型复杂系统中也可能会出现同样的问题，因为要在其中创建大量对象(也可能是用户)，这些对象需要同时并存。

这个问题之所以会发生，是因为当我们创建一个新对象时，需要分配额外的内存。虽然虚拟内存理论上为我们提供了无限制的内存空间，但现实却并非如此。如果一个系统耗尽了所有的物理内存，就会开始将内存页替换到二级存储设备，通常是硬盘驱动器(Hard Disk Drive，HDD)。 在多数情况下，由于内存和硬盘之间的性能差异，这是不能接受的。固态硬盘(Solid State Drive, SSD)的性能一般比硬盘更好，但并非人人都使用SSD，SSD并不会很快全面替代硬盘(请参考 网页[t.cn/RqrjS0E])。

除内存使用之外，计算性能也是一个考虑点。图形软件，包括计算机游戏，应该能够极快地 渲染3D信息(例如，有成千上万棵树的森林或满是士兵的村庄)。如果一个3D地带的每个对象都 是单独创建，未使用数据共享，那么性能将是无法接受的(请参考网页[t.cn/Rqrj9qa])。

作为软件工程师，我们应该编写更好的软件来解决软件问题，而不是要求客户购买更多更好的硬件。**享元设计模式通过为相似对象引入数据共享来最小化内存使用，提升性能**(请参考网页[t.cn/RqrjNF3])。**一个享元(Flyweight)就是一个包含状态独立的不可变(又称固有的)数据的共享对象。依赖状态的可变(又称非固有的)数据不应是享元的一部分，因为每个对象的这种信息都不同，无法共享。如果享元需要非固有的数据，应该由客户端代码显式地提供**(请参考[GOF95，第219页]和网页[t.cn/RqrjOX3])。

用一个例子可能有助于解释实际应用场景中如何使用享元模式。假设我们正在设计一个性能关键的游戏，例如第一人称射击(First-Person Shooter，FPS)游戏。在FPS游戏中，玩家(士兵) 共享一些状态，如外在表现和行为。例如，在《反恐精英》游戏中，同一团队(反恐精英或恐怖分子)的所有士兵看起来都是一样的(外在表现)。同一个游戏中，(两个团队的)所有士兵都有一些共同的动作，比如，跳起、低头等(行为)。这意味着我们可以创建一个享元来包含所有共同的数据。当然，士兵也有许多因人而异的可变数据，这些数据不是享元的一部分，比如，枪支、健康状况和地理位置等。


```python
#以下是来自于github的示例

# -*- coding: utf-8 -*-

"""http://codesnipers.com/?q=python-flyweights"""

import weakref


class FlyweightMeta(type):
    def __new__(mcs, name, parents, dct):
        """

        :param name: class name
        :param parents: class parents
        :param dct: dict: includes class attributes, class methods,
        static methods, etc
        :return: new class
        """

        # set up instances pool
        dct['pool'] = weakref.WeakValueDictionary()
        return super(FlyweightMeta, mcs).__new__(mcs, name, parents, dct)

    @staticmethod
    def _serialize_params(cls, *args, **kwargs):
        """Serialize input parameters to a key.
        Simple implementation is just to serialize it as a string

        """
        args_list = map(str, args)
        args_list.extend([str(kwargs), cls.__name__])
        key = ''.join(args_list)
        return key

    def __call__(cls, *args, **kwargs):
        key = FlyweightMeta._serialize_params(cls, *args, **kwargs)
        pool = getattr(cls, 'pool', {})

        instance = pool.get(key)
        if not instance:
            instance = super(FlyweightMeta, cls).__call__(*args, **kwargs)
            pool[key] = instance
        return instance


class Card(object):

    """The object pool. Has builtin reference counting"""
    _CardPool = weakref.WeakValueDictionary()

    """Flyweight implementation. If the object exists in the
    pool just return it (instead of creating a new one)"""
    def __new__(cls, value, suit):
        obj = Card._CardPool.get(value + suit)
        if not obj:
            obj = object.__new__(cls)
            Card._CardPool[value + suit] = obj
            obj.value, obj.suit = value, suit
        return obj

    # def __init__(self, value, suit):
    #     self.value, self.suit = value, suit

    def __repr__(self):
        return "<Card: %s%s>" % (self.value, self.suit)


class Card2(object):
    __metaclass__ = FlyweightMeta

    def __init__(self, *args, **kwargs):
        # print('Init {}: {}'.format(self.__class__, (args, kwargs)))
        pass


if __name__ == '__main__':
    import sys
    if sys.version_info[0] > 2:
        sys.stderr.write("!!! This example is compatible only with Python 2 ATM !!!\n")
        raise SystemExit(0)

    # comment __new__ and uncomment __init__ to see the difference
    c1 = Card('9', 'h')
    c2 = Card('9', 'h')
    print(c1, c2)
    print(c1 == c2, c1 is c2)
    print(id(c1), id(c2))

    c1.temp = None
    c3 = Card('9', 'h')
    print(hasattr(c3, 'temp'))
    c1 = c2 = c3 = None
    c3 = Card('9', 'h')
    print(hasattr(c3, 'temp'))

    # Tests with metaclass
    instances_pool = getattr(Card2, 'pool')
    cm1 = Card2('10', 'h', a=1)
    cm2 = Card2('10', 'h', a=1)
    cm3 = Card2('10', 'h', a=2)

    assert (cm1 == cm2) != cm3
    assert (cm1 is cm2) is not cm3
    assert len(instances_pool) == 2

    del cm1
    assert len(instances_pool) == 2

    del cm2
    assert len(instances_pool) == 1

    del cm3
    assert len(instances_pool) == 0

### OUTPUT ###
# (<Card: 9h>, <Card: 9h>)
# (True, True)
# (31903856, 31903856)
# True
# False
```

    <Card: 9h> <Card: 9h>
    True True
    4405784136 4405784136
    True
    False



    ---------------------------------------------------------------------------

    AttributeError                            Traceback (most recent call last)

    <ipython-input-6-f0792005f54c> in <module>()
         96
         97     # Tests with metaclass
    ---> 98     instances_pool = getattr(Card2, 'pool')
         99     cm1 = Card2('10', 'h', a=1)
        100     cm2 = Card2('10', 'h', a=1)


    AttributeError: type object 'Card2' has no attribute 'pool'


上面程序的运行结果

root@localhost:~/software# python flyweight.py
(<Card: 9h>, <Card: 9h>)
(True, True)
(140061805283344, 140061805283344)
True
False


```python
"""http://codesnipers.com/?q=python-flyweights"""

import weakref  


class Card(object):
    """The object pool. Has builtin reference counting"""
    _CardPool = weakref.WeakValueDictionary()

    """Flyweight implementation. If the object exists in the
    pool just return it (instead of creating a new one)"""
    def __new__(cls, value, suit):         
        obj = Card._CardPool.get(value + suit, None)         
        if not obj:             
            obj = object.__new__(cls)             
            Card._CardPool[value + suit] = obj             
            obj.value, obj.suit = value, suit          
        return obj

    # def __init__(self, value, suit):         
    #     self.value, self.suit = value, suit      

    def __repr__(self):         
        return "<Card: %s%s>" % (self.value, self.suit)      


if __name__ == '__main__':
    # comment __new__ and uncomment __init__ to see the difference
    c1 = Card('9', 'h')
    c2 = Card('9', 'h')
    print(c1, c2)
    print(c1 == c2)
    print(id(c1), id(c2))


```

    <Card: 9h> <Card: 9h>
    True
    4405761024 4405761024


## 现实生活的例子

享元模式是一个用于优化的设计模式。因此，要找一个合适的现实生活的例子不太容易。我们可以把享元看作现实生活中的缓存区。例如，许多书店都有专用的书架来摆放最新和最流行的出版物。这就是一个缓存区，你可以先在这些专用书架上看看有没有正在找的书籍，如果没找到，则可以让图书管理员来帮你。

## 软件的例子

Exaile音乐播放器(请参考网页[t.cn/RqrjYHQ])使用享元来复用通过相同URL识别的对象 (在这里是指音乐歌曲)。创建一个与已有对象的URL相同的新对象是没有意义的，所以复用相同的对象来节约资源(请参考网页[http://t.cn/RqrjQWr])。

Peppy是一个用Python语言实现的类XEmacs编辑器(请参考网页[t.cn/hbhSda])，它使用享元模式存储major mode状态栏的状态。这是因为除非用户修改，否则所有状态栏共享相同的属性(请参考网页[t.cn/Rqrjm9y])。这个软件原作者2014年就放弃了。

## 应用案例

享元旨在优化性能和内存使用。所有嵌入式系统(手机、平板电脑、游戏终端和微控制器等)和性能关键的应用(游戏、3D图形处理和实时系统等)都能从其获益。若想要享元模式有效，需要满足GoF的《设计模式》一书罗列的以下几个条件。
* 应用需要使用大量的对象。
* 对象太多，存储/渲染它们的代价太大。一旦移除对象中的可变状态(因为在需要之时，应该由客户端代码显式地传递给享元)，多组不同的对象可被相对更少的共享对象所替代。
* 对象ID对于应用不重要。对象共享会造成ID比较的失败，所以不能依赖对象ID(那些在客户端代码看来不同的对象，最终具有相同的ID)。

## 实现
由于之前已提到树的例子，那么就来看看如何实现它。在这个例子中，我们将构造一小片水果树的森林，小到能确保在单个终端页面中阅读整个输出。然而，无论你构造的森林有多大，内存分配都保持相同。下面这个Enum类型变量描述三种不同种类的水果树。


```python
TreeType = Enum('TreeType', 'apple_tree cherry_tree peach_tree')
```

## 现实生活中的例子

在深入代码之前，我们稍稍解释一下memoization与享元模式之间的区别。memoization是一种优化技术，使用一个缓存来避免重复计算那些在更早的执行步骤中已经计算好的结果。memoization并不是只能应用于某种特定的编程方式，比如面向对象编程(Object-Oriented Programming，OOP)。在Python中，memoization可以应用于方法和简单的函数。享元则是一种特定于面向对象编程优化的设计模式，关注的是共享对象数据。

在Python中，享元可以以多种方式实现，但我发现这个例子中展示的实现非常简洁。pool变量是一个对象池(换句话说，是我们的缓存)。注意：pool是一个类属性(类的所有实例共享的一个变量，请参考网页[t.cn/zHwpgFe])。使用特殊方法__new__(这个方法在__init__之前被调用)，我们把Tree类变换成一个元类，元类支持自引用。这意味着cls引用的是Tree类(请参考[Lott14，第99页])。当客户端要创建Tree的一个实例时，会以tree_type参数传递树的种类。树的种类用于检查是否创建过相同种类的树。如果是，则返回之前创建的对象；否则，将这个新的树种添加到池中，并返回相应的新对象，如下所示。


```python
def __new__(cls, tree_type):
    obj = cls.pool.get(tree_type, None)
    if not obj:
        obj = object.__new__(cls)
        cls.pool[tree_type] = obj
        obj.tree_type = tree_type
    return obj
```

方法render()用于在屏幕上渲染一棵树。注意，享元不知道的所有可变(外部的)信息都需要由客户端代码显式地传递。在当前案例中，每棵树都用到一个随机的年龄和一个x, y形式的位置。为了让render()更加有用，有必要确保没有树会被渲染到另一个棵之上。你可以考虑把这个作为练习。如果你想让渲染更加有趣，可以使用一个图形工具包，比如Tkinter或Pygame。


```python
def render(self, age, x, y):
    print('render a tree of type {} and age {} at ({}, {})'.format(self.tree_type, age, x, y))
```

main()函数展示了我们可以如何使用享元模式。一棵树的年龄是1到30年之间的一个随机值。坐标使用1到100之间的随机值。虽然渲染了18棵树，但仅分配了3棵树的内存。输出的最后一行证明当使用享元时，我们不能依赖对象的ID。函数id()会返回对象的内存地址。Python规范并没有要求id()返回对象的内存地址，只是要求id()为每个对象返回一个唯一性ID，不过CPython(Python的官方实现)正好使用对象的内存地址作为对象唯一性ID。在我们的例子中，即使两个对象看起来不相同，但是如果它们属于同一个享元家族(在这里，家族由tree_type定义)，那么它们实际上有相同的ID。当然，不同ID的比较仍然可用于不同家族的对象，但这仅在客户端知道实现细节的情况下才可行(通常并非如此)。


```python
def main():
    rnd = random.Random()
    age_min, age_max = 1, 30 # 单位为年
    min_point, max_point = 0, 100
    tree_counter = 0
    for _ in range(10):
        t1 = Tree(TreeType.apple_tree)
        t1.render(rnd.randint(age_min, age_max),
                rnd.randint(min_point, max_point),
                rnd.randint(min_point, max_point))
        tree_counter += 1
    for _ in range(3):
        t2 = Tree(TreeType.cherry_tree)
        t2.render(rnd.randint(age_min, age_max),
                rnd.randint(min_point, max_point),
                rnd.randint(min_point, max_point))
        tree_counter += 1
    for _ in range(5):
        t3 = Tree(TreeType.peach_tree)
        t3.render(rnd.randint(age_min, age_max),
                    rnd.randint(min_point, max_point),
                    rnd.randint(min_point, max_point))
        tree_counter += 1

    print('trees rendered: {}'.format(tree_counter))
    print('trees actually created: {}'.format(len(Tree.pool)))
    t4 = Tree(TreeType.cherry_tree)
    t5 = Tree(TreeType.cherry_tree)
    t6 = Tree(TreeType.apple_tree)
    print('{} == {}? {}'.format(id(t4), id(t5), id(t4) == id(t5)))
    print('{} == {}? {}'.format(id(t5), id(t6), id(t5) == id(t6)))
```

下面完整的代码清单(文件flyweight.py)将给出享元模式如何实现及使用的完整描述。


```python
import random
from enum import Enum
TreeType = Enum('TreeType', 'apple_tree cherry_tree peach_tree')

class Tree:
    pool = dict()
    def __new__(cls, tree_type):
        obj = cls.pool.get(tree_type, None)
        if not obj:
            obj = object.__new__(cls)
            cls.pool[tree_type] = obj
            obj.tree_type = tree_type
        return obj

    def render(self, age, x, y):
        print('render a tree of type {} and age {} at ({}, {})'.format(self.tree_type, age, x, y))


def main():
    rnd = random.Random()
    age_min, age_max = 1, 30 # 单位为年
    min_point, max_point = 0, 100
    tree_counter = 0
    for _ in range(10):
        t1 = Tree(TreeType.apple_tree)
        t1.render(rnd.randint(age_min, age_max),
                rnd.randint(min_point, max_point),
                rnd.randint(min_point, max_point))
        tree_counter += 1
    for _ in range(3):
        t2 = Tree(TreeType.cherry_tree)
        t2.render(rnd.randint(age_min, age_max),
                rnd.randint(min_point, max_point),
                rnd.randint(min_point, max_point))
        tree_counter += 1
    for _ in range(5):
        t3 = Tree(TreeType.peach_tree)
        t3.render(rnd.randint(age_min, age_max),
                    rnd.randint(min_point, max_point),
                    rnd.randint(min_point, max_point))
        tree_counter += 1

    print('trees rendered: {}'.format(tree_counter))
    print('trees actually created: {}'.format(len(Tree.pool)))
    t4 = Tree(TreeType.cherry_tree)
    t5 = Tree(TreeType.cherry_tree)
    t6 = Tree(TreeType.apple_tree)
    print('{} == {}? {}'.format(id(t4), id(t5), id(t4) == id(t5)))
    print('{} == {}? {}'.format(id(t5), id(t6), id(t5) == id(t6)))

main()
```

    render a tree of type TreeType.apple_tree and age 29 at (13, 7)
    render a tree of type TreeType.apple_tree and age 21 at (40, 66)
    render a tree of type TreeType.apple_tree and age 3 at (53, 52)
    render a tree of type TreeType.apple_tree and age 19 at (100, 84)
    render a tree of type TreeType.apple_tree and age 11 at (57, 56)
    render a tree of type TreeType.apple_tree and age 22 at (20, 37)
    render a tree of type TreeType.apple_tree and age 11 at (7, 16)
    render a tree of type TreeType.apple_tree and age 3 at (10, 18)
    render a tree of type TreeType.apple_tree and age 17 at (85, 75)
    render a tree of type TreeType.apple_tree and age 4 at (97, 34)
    render a tree of type TreeType.cherry_tree and age 17 at (72, 29)
    render a tree of type TreeType.cherry_tree and age 10 at (26, 79)
    render a tree of type TreeType.cherry_tree and age 1 at (85, 56)
    render a tree of type TreeType.peach_tree and age 6 at (44, 71)
    render a tree of type TreeType.peach_tree and age 4 at (42, 50)
    render a tree of type TreeType.peach_tree and age 27 at (85, 24)
    render a tree of type TreeType.peach_tree and age 27 at (51, 22)
    render a tree of type TreeType.peach_tree and age 23 at (37, 55)
    trees rendered: 18
    trees actually created: 3
    4405186288 == 4405186288? True
    4405186288 == 4405186456? False


执行上面的示例程序会显示被渲染对象的类型、随机年龄以及坐标，还有相同/不同家族享元对象ID的比较结果。你在执行这个程序时别指望能看到与下面相同的输出，因为年龄和坐标是随机的，对象ID也依赖内存映射。

如果你想更多地练习一下享元模式，可以尝试实现本章提到的FPS士兵。思考一下哪些数据应该是享元的一部分(不可变的、内部的)，哪些数据不应该是(可变的、外部的)。

## 小结

本章中，我们学习了享元模式。在我们想要优化内存使用提高应用性能之时，可以使用享元。在所有内存受限(想一想嵌入式系统)或关注性能的系统(比如图形软件和电子游戏)中，这一点相当重要。基于GTK+的Exaile音乐播放器使用享元来避免对象复制，Peppy文本编辑器则使用享元来共享状态栏的属性。

一般来说，在应用需要创建大量的计算代价大但共享许多属性的对象时，可以使用享元。重点在于将不可变(可共享)的属性与可变的属性区分开。我们实现了一个树渲染器，支持三种不同的树家族。通过显式地向render()方法提供可变的年龄和x，y属性，我们成功地仅创建了3个不同的对象，而不是18个。虽然那看起来似乎没什么了不起，但是想象一下，如果是2000棵树而不是18棵树，那又会怎样呢?
