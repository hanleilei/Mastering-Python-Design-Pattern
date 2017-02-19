## 原型模式

有时，我们需要原原本本地为对象创建一个副本。举例来说，假设你想创建一个应用来存储、分享、编辑(比如，修改、添加注释及删除)食谱。用户Bob找到一份蛋糕食谱，在做了一些改变后，觉得自己做的蛋糕非常美味，想要与朋友Alice分享这个食谱。但是该如何分享食谱呢? 如果在与Alice分享之后，Bob想对食谱做进一步的试验，Alice手里的食谱也能跟着变化吗? Bob能够持有蛋糕食谱的两个副本吗? 对蛋糕食谱进行的试验性变更不应该对原本美味蛋糕的食谱造成影响。

**这样的问题可以通过让用户对同一份食谱持有多个独立的副本来解决。每个副本被称为一个克隆，是某个时间点原有对象的一个完全副本。这里时间是一个重要因素。因为它会影响克隆所包含的内容**。例如，如果Bob在对蛋糕食谱做改进以臻完美之前就与Alice分享了，那么Alice就绝不可能像Bob那样烘烤出自己的美味蛋糕，只能按照Bob原来找到的食谱烘烤蛋糕。

注意引用与副本之间的区别。如果Bob和Alice持有的是同一个蛋糕食谱对象的两个引用，那么Bob对食谱做的任何改变，对于Alice的食谱版本都是可见的，反之亦然。我们想要的是Bob和Alice各自持有自己的副本，这样他们可以各自做变更而不会影响对方的食谱。实际上Bob需要蛋糕食谱的两个副本: 美味版本和试验版本。

图略，其实就是引用和复制的差别。很简单。

下面是一个关于原型模式的示例，来自github：

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import copy

class Prototype:

    value = 'default'

    def clone(self， **attrs):
        """Clone a prototype and update inner attributes dictionary"""
        obj = copy.deepcopy(self)
        obj.__dict__.update(attrs)
        return obj


class PrototypeDispatcher:

    def __init__(self):
        self._objects = {}

    def get_objects(self):
        """Get all objects"""
        return self._objects

    def register_object(self， name， obj):
        """Register an object"""
        self._objects[name] = obj

    def unregister_object(self， name):
        """Unregister an object"""
        del self._objects[name]


def main():
    dispatcher = PrototypeDispatcher()
    prototype = Prototype()

    d = prototype.clone()
    a = prototype.clone(value='a-value'， category='a')
    b = prototype.clone(value='b-value'， is_checked=True)
    dispatcher.register_object('objecta'， a)
    dispatcher.register_object('objectb'， b)
    dispatcher.register_object('default'， d)
    print([{n: p.value} for n， p in dispatcher.get_objects().items()])

if __name__ == '__main__':
    main()

### OUTPUT ###
# [{'objectb': 'b-value'}， {'default': 'default'}， {'objecta': 'a-value'}]
```


```python
import copy

class Prototype:
    def __init__(self):
        self._objects = {}

    def register_object(self， name， obj):
        """Register an object"""
        self._objects[name] = obj

    def unregister_object(self， name):
        """Unregister an object"""
        del self._objects[name]

    def clone(self， name， **attr):
        """Clone a registered object and update inner attributes dictionary"""
        obj = copy.deepcopy(self._objects.get(name))
        obj.__dict__.update(attr)
        return obj


def main():
    class A:
        pass

    a = A()
    prototype = Prototype()
    prototype.register_object('a'， a)
    b = prototype.clone('a'， a=1， b=2， c=3)

    print(a)
    print(b.a， b.b， b.c)


if __name__ == '__main__':
    main()


```
运行得到的结果为：

    <__main__.main.<locals>.A object at 0x1066e3cc0>
    1 2 3


原型设计模式(Prototype design pattern)帮助我们创建对象的克隆，其最简单的形式就是一个clone()函数，接受一个对象作为输入参数，返回输入对象的一个副本。在Python中，这可以使用copy.deepcopy()函数来完成。来看一个例子，下面的代码中(文件clone.py)有两个类，A和B。A是父类，B是衍生类/子类。在主程序部分，我们创建一个类B的实例b，并使用deepcopy()创建b的一个克隆c。结果是所有成员都被复制到了克隆c，以下是代码演示。作为一个有趣的练习，你也可以尝试在对象的组合形式上使用deepcopy()。

左侧是两个引用。Bob和Alice参考的是同一个食谱，这本质上意味着两者共享食谱，并且所有变更两人皆可见。右侧是同一个食谱的两个不同副本，这样允许各自独立地变更，Alice做的改变不会影响Bob做的改变，反之亦然。


```python
import copy

class A:
    def __init__(self):
        self.x = 18
        self.msg = 'Hello'

class B(A):
    def __init__(self):
        A.__init__(self)
        self.y = 34
    def __str__(self):
        return '{}， {}， {}'.format(self.x， self.msg， self.y)


if __name__ == '__main__':
    b = B()
    c = copy.deepcopy(b)
    print([str(i) for i in (b， c)])
    print([i for i in (b， c)])
```
得到的输出为：

    ['18， Hello， 34'， '18， Hello， 34']
    [<__main__.B object at 0x1066f2668>， <__main__.B object at 0x1066f26a0>]


重要的是注意两个对象位于两个不同的内存地址(输出中的0x...部分)。这意味着两个对象是两个独立的副本。

在本章稍后的3.4节中，我们将看到为了保持一个被克隆对象的注册表，如何将copy.deepcopy与封装在某个类中的一些额外的样板代码一起使用。

## 现实生活中的示例

原型设计模式无非就是克隆一个对象。有丝分裂，即细胞分裂的过程，是生物克隆的一个例子。在这个过程中，细胞核分裂产生两个新的细胞核，其中每个都有与原来细胞完全相同的染色体和DNA内容(请参考网页[t.cn/RqBrOuM])。

## 软件中的示例

很多Python应用都使用了原型模式(请参考网页[t.cn/RqBruae])，但几乎都不称之为原型模式，因为对象克隆是编程语言的一个内置特性。

可视化工具套件(Visualization Toolkit，VTK)(请参考网页[t.cn/hCDIf])是原型模式的一个应用。VTK是一个开源的跨平台系统，用于三维计算机图形/图片处理以及可视化。VTK使用原型模式来创建几何元素(比如，点、线、六面体等，请参考网页[t.cn/RqBrecy])的克隆。

另一个使用原型模式的项目是music21。根据该项目页面所述，“music21是一组工具，帮助学者和其他积极的听众快速简便地得到音乐相关问题的答案”(请参考网页[t.cn/RGK8f1V])。 music21工具套件使用原型模式来复制音符和总谱(请参考网页[t.cn/RqBdhGK])。

## 应用案例

当我们已有一个对象，并希望创建该对象的一个完整副本时，原型模式就派上用场了。在我们知道对象的某些部分会被变更但又希望保持原有对象不变之时，通常需要对象的一个副本。在这样的案例中，重新创建原有对象是没有意义的(请参考网页[t.cn/RqBrOuM])。

另一个案例是，当我们想复制一个复杂对象时，使用原型模式会很方便。对于复制复杂对象，我们可以将对象当作是从数据库中获取的，并引用其他一些也是从数据库中获取的对象。若通过多次重复查询数据来创建一个对象，则要做很多工作。在这种场景下使用原型模式要方便得多。

至此，我们仅涉及了引用与副本的问题，而副本又可以进一步分为 **深副本与浅副本**。深副本就是我们在本章中到目前为止所看到的：原始对象的所有数据都被简单地复制到克隆对象中，没有例外。浅副本则依赖引用。我们可以引入数据共享和写时复制一类的技术来优化性能(例如，减小克隆对象的创建时间)和内存使用。如果可用资源有限(例如，嵌入式系统)或性能至关重要(例如，高性能计算)，那么使用浅副本可能更佳。

在Python中，可以使用copy.copy()函数进行浅复制。以下内容引用自Python官方文档，说明了浅副本(copy.copy())和深副本(copy.deepcopy())之间的区别(请参考网页[t.cn/RqBdSj1])。

* 浅副本构造一个新的复合对象后，(会尽可能地)将在原始对象中找到的对象的引用插入新对象中。
* 深副本构造一个新的复合对象后，会递归地将在原始对象中找到的对象的副本插入新对象中。

你能想到什么使用浅副本比深副本更好的例子吗?

## 实现

编程方面的书籍历经多个版本的情况并不多见。例如，Kernighan和Ritchie编著的C编程经典教材《C程序设计语言》(The C Programming Language)历经两个版本。第一版1978年出版，那时C语言还没被标准化。该书第二版10年后才出版，涵盖C语言标准(ANSI)版本。这两个版本之间的区别是什么?列举几个：价格、长度(页数)以及出版日期。但也有很多相似之处：作者、出版商以及描述该书的标签/关键词都是完全一样的。这表明从头创建一版新书并不总是最佳方式。如果知道两个版本之间的诸多相似之处，则可以先克隆一份，然后仅修改新版本与旧版本之间的不同之处。

来看看可以如何使用原型模式创建一个展示图书信息的应用。我们以一本书的描述开始。除了常规的初始化之外，Book类展示了一种有趣的技术可避免可伸缩构造器问题。**在__init__() 方法中，仅有三个形参是固定的:name、authors和price，但是使用rest变长列表，调用者能以关键词的形式(名称=值)传入更多的参数。self.__dict__.update(rest)一行将rest的内容添加到Book类的内部字典中，成为它的一部分。**

但这里有个问题。我们并不知道所有被添加参数的名称，但又需要访问内部字典将这些参数应用到__str__()中，并且字典的内容并不遵循任何特定的顺序，所以使用一个OrderedDict来强制元素有序，否则，每次程序执行都会产生不同的输出。当然，你不应该把我的话当作理所当然。作为一个练习，尝试删除使用的OrderedDict和sorted()，运行一下示例代码看看我说得对不对。


```python

class Book:
    def __init__(self， name， authors， price， **rest):
        '''rest的例子有: 出版商、长度、 标签、出版日期'''
        self.name = name
        self.authors = authors
        self.price = price
        self.__dict__.update(rest)

    def __str__(self):
        mylist = []
        ordered = OrderedDict(sorted(self.__dict__.items()))
        for i in ordered.keys():
            mylist.append('{}: {}'.format(i， ordered[i]))
            if i == 'price':
                mylist.append('$')
            mylist.append('\n')
        return ''.join(mylist)
```

Prototype类实现了原型设计模式。Prototype类的核心是clone()方法，该方法使用我们熟悉的copy.deepcopy()函数来完成真正的克隆工作。但Prototype类在支持克隆之外做了一点更多的事情，它包含了方法register()和unregister()，这两个方法用于在一个字典中追踪被克隆的对象。注意这仅是一个方便之举，并非必需。

此外，clone()方法和Book类中的__str__使用了相同的技巧，但这次是因为别的原因。使用变长列表attr，我们可以仅传递那些在克隆一个对象时真正需要变更的属性变量，如下所示。


```python
class Prototype:
    def __init__(self):
        self.objects = dict()
    def register(self， identifier， obj):
        self.objects[identifier] = obj
    def unregister(self， identifier):
        del self.objects[identifier]
    def clone(self， identifier， **attr):
        found = self.objects.get(identifier)
        if not found:
            raise ValueError('Incorrect object identifier: {}'.format(identifier))
        obj = copy.deepcopy(found)
        obj.__dict__.update(attr)
        return obj
```

main()函数以实践的方式展示了本节开头提到的《C程序设计语言》一书克隆的例子。克隆该书的第一个版本来创建第二个版本，我们仅需要传递已有参数中被修改参数的值，但也可以传递额外的参数。在这个案例中，edition就是一个新参数，在书的第一个版本中并不需要，但对于克隆版本却是很有用的信息。


```python
def main():
    b1 = Book('The C Programming Language', ('Brian W. Kernighan', 'Dennis M.Ritchie'),
                price=118, publisher='Prentice Hall', length=228, publication_date='1978-02-22',
                tags=('C', 'programming', 'algorithms', 'data structures'))
    prototype = Prototype()
    cid = 'k&r-first'
    prototype.register(cid, b1)
    b2 = prototype.clone(cid, name='The C Programming Language(ANSI)', price=48.99,
                         length=274, publication_date='1988-04-01', edition=2)
    for i in (b1， b2):
            print(i)
    print("ID b1 : {} != ID b2 : {}".format(id(b1)， id(b2)))
```

注意，代码中使用了函数id()返回对象的内存地址。当使用深副本来克隆一个对象时，克隆对象的内存地址必定与原始对象的内存地址不一样。文件prototype.py的完整内容如下所示。


```python
import copy
from collections import OrderedDict

class Book:
    def __init__(self， name， authors， price， **rest):
        '''rest的例子有: 出版商、长度、 标签、出版日期'''
        self.name = name
        self.authors = authors
        self.price = price
        self.__dict__.update(rest)

    def __str__(self):
        mylist = []
        ordered = OrderedDict(sorted(self.__dict__.items()))
        for i in ordered.keys():
            mylist.append('{}: {}'.format(i， ordered[i]))
            if i == 'price':
                mylist.append('$')
            mylist.append('\n')
        return ''.join(mylist)

class Prototype:
    def __init__(self):
        self.objects = dict()
    def register(self， identifier， obj):
        self.objects[identifier] = obj
    def unregister(self， identifier):
        del self.objects[identifier]
    def clone(self， identifier， **attr):
        found = self.objects.get(identifier)
        if not found:
            raise ValueError('Incorrect object identifier: {}'.format(identifier))
        obj = copy.deepcopy(found)
        obj.__dict__.update(attr)
        return obj

def main():
    b1 = Book('The C Programming Language'， ('Brian W. Kernighan'， 'Dennis M.Ritchie')，
                price=118， publisher='Prentice Hall'， length=228， publication_date='1978-02-22'，
                tags=('C'， 'programming'， 'algorithms'， 'data structures'))
    prototype = Prototype()
    cid = 'k&r-first'
    prototype.register(cid， b1)
    b2 = prototype.clone(cid， name='The C Programming Language(ANSI)'， price=48.99，
                         length=274， publication_date='1988-04-01'， edition=2)
    for i in (b1， b2):
            print(i)
    print("ID b1 : {} != ID b2 : {}".format(id(b1)， id(b2)))

main()
```
得到的输出为：

    authors: ('Brian W. Kernighan'， 'Dennis M.Ritchie')
    length: 228
    name: The C Programming Language
    price: 118$
    publication_date: 1978-02-22
    publisher: Prentice Hall
    tags: ('C'， 'programming'， 'algorithms'， 'data structures')

    authors: ('Brian W. Kernighan'， 'Dennis M.Ritchie')
    edition: 2
    length: 274
    name: The C Programming Language(ANSI)
    price: 48.99$
    publication_date: 1988-04-01
    publisher: Prentice Hall
    tags: ('C'， 'programming'， 'algorithms'， 'data structures')

    ID b1 : 4402992184 != ID b2 : 4402992016


id()的输出依赖计算机当前的内存分配情况，该输出在每次执行这个程序时都应该是不一样的。无论实际的内存地址是多少，原始对象与克隆对象的内存地址都绝无可能相同。

原型模式确实按预期生效了。《C程序设计语言》的第二版复用了第一版设置的所有信息，所有我们定义的不同之处仅应用于第二版，第一版不受影响。看到id()函数的输出显示两个内存地址不相同，我们就更加确信了。

作为练习，你可以提出自己的原型模式的例子。以下是一些想法。

* 本章提到的食谱例子
* 本章提到的从数据库获取数据填充对象的例子
* 复制一张图片，这样你可以做些自己的修改而不会影响原来的图片

## 小结

在本章中，我们学习了如何使用原型设计模式。原型模式用于创建对象的完全副本。确切地说，创建一个对象的副本可以指代以下两件事情。

* 当创建一个浅副本时，副本依赖引用
* 当创建一个深副本时，副本复制所有东西

第一种情况中，我们关注提升应用性能和优化内存使用，在对象之间引入数据共享，但需要小心地修改数据，因为所有变更对所有副本都是可见的。浅副本在本章中没有过多介绍，但也许你会想试验一下。

第二种情况中，我们希望能够对一个副本进行更改而不会影响其他对象。对于我们之前看到的蛋糕食谱示例这类案例，这一特性是很有用的。这里不会进行数据共享，所以需要关注因对象克隆而引入的资源耗用问题。

我们展示了一个深副本的简单示例，在Python中，深副本使用copy.deepcopy()函数来完成。我们也提到一些现实生活中发现的克隆例子，并着重讲述了有丝分裂的例子。

许多软件项目都使用了原型模式，但因为在Python中这是一个内置特性，所以并不称之为原型模式。这些软件项目包括VTK(它使用原型模式创建几何学元素的克隆)和music21(它使用 原型模式复制乐谱和音符)。

最后，我们讨论了原型模式的应用案例，并实现一个程序以支持克隆书籍对象。这样在新版本中所有信息无需改变即可复用，并且同时可以更新变更的信息，并添加新的信息。
