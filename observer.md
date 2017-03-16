# 观察者模式

有时，我们希望在一个对象的状态改变时更新另外一组对象。在MVC模式中有这样一个非常常见的例子，假设在两个视图（例如，一个饼图和一个电子表格）中使用同一个模型的数据，无论何时更改了模型，都需要更新两个视图。这就是观察者设计模式要处理的问题（请参考[Eckel08，第213页]）。

观察者模式描述单个对象（发布者，又称为主持者或可观察者）与一个或多个对象（订阅者，又称为观察者）之间的发布—订阅关系。在MVC例子中，发布者是模型，订阅者是视图。然而，MVC并非是仅有的发布—订阅例子。信息聚合订阅（比如，RSS或Atom）是另一种例子。许多读者通常会使用一个信息聚合阅读器订阅信息流，每当增加一条新信息时，他们就能自动地获取到更新。

观察者模式背后的思想等同于MVC和关注点分离原则背后的思想，即降低发布者与订阅者之间的耦合度，从而易于在运行时添加/删除订阅者。此外，发布者不关心它的订阅者是谁。它只是将通知发送给所有订阅者（请参考[GOF95，第327页]）。

以下为来自于github的示例：

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
from __future__ import print_function
"""http://code.activestate.com/recipes/131499-observer-pattern/"""


class Subject(object):

    def __init__(self):
        self._observers = []

    def attach(self, observer):
        if observer not in self._observers:
            self._observers.append(observer)

    def detach(self, observer):
        try:
            self._observers.remove(observer)
        except ValueError:
            pass

    def notify(self, modifier=None):
        for observer in self._observers:
            if modifier != observer:
                observer.update(self)


# Example usage
class Data(Subject):

    def __init__(self, name=''):
        Subject.__init__(self)
        self.name = name
        self._data = 0

    @property
    def data(self):
        return self._data

    @data.setter
    def data(self, value):
        self._data = value
        self.notify()


class HexViewer:

    def update(self, subject):
        print(u'HexViewer: Subject %s has data 0x%x' %
              (subject.name, subject.data))


class DecimalViewer:

    def update(self, subject):
        print(u'DecimalViewer: Subject %s has data %d' %
              (subject.name, subject.data))


# Example usage...
def main():
    data1 = Data('Data 1')
    data2 = Data('Data 2')
    view1 = DecimalViewer()
    view2 = HexViewer()
    data1.attach(view1)
    data1.attach(view2)
    data2.attach(view2)
    data2.attach(view1)

    print(u"Setting Data 1 = 10")
    data1.data = 10
    print(u"Setting Data 2 = 15")
    data2.data = 15
    print(u"Setting Data 1 = 3")
    data1.data = 3
    print(u"Setting Data 2 = 5")
    data2.data = 5
    print(u"Detach HexViewer from data1 and data2.")
    data1.detach(view2)
    data2.detach(view2)
    print(u"Setting Data 1 = 10")
    data1.data = 10
    print(u"Setting Data 2 = 15")
    data2.data = 15


if __name__ == '__main__':
    main()

### OUTPUT ###
# Setting Data 1 = 10
# DecimalViewer: Subject Data 1 has data 10
# HexViewer: Subject Data 1 has data 0xa
# Setting Data 2 = 15
# HexViewer: Subject Data 2 has data 0xf
# DecimalViewer: Subject Data 2 has data 15
# Setting Data 1 = 3
# DecimalViewer: Subject Data 1 has data 3
# HexViewer: Subject Data 1 has data 0x3
# Setting Data 2 = 5
# HexViewer: Subject Data 2 has data 0x5
# DecimalViewer: Subject Data 2 has data 5
# Detach HexViewer from data1 and data2.
# Setting Data 1 = 10
# DecimalViewer: Subject Data 1 has data 10
# Setting Data 2 = 15
# DecimalViewer: Subject Data 2 has data 15
```

## 现实生活的例子

现实中，拍卖会类似于观察者模式。每个拍卖出价人都有一些拍牌，在他们想出价时就可以举起来。不论出价人在何时举起一块拍牌，拍卖师都会像主持者那样更新报价，并将新的价格广播给所有出价人（订阅者）。

下图展示了观察者模式与拍卖会的关联，经www.sourcemaking.com 允许使用（请参考网页[t.cn/rqr1yxo]）。

## 软件的例子
django-observer源代码包（请参考网页[t.cn/rqr14oz]）是一个第三方django包，可用于注册回调函数，之后在某些django模型字段发生变化时执行。它支持许多不同类型的模型字段（charfield、integerfield等）。

rabbitmq可用于为应用添加异步消息支持，支持多种消息协议（比如，http和amqp），可在python应用中用于实现发布—订阅模式，也就是观察者设计模式（请参考网页[t.cn/rqr1iix]）。

## 应用案例

当我们希望在一个对象（主持者/发布者/可观察者）发生变化时通知/更新另一个或多个对象的时候，通常会使用观察者模式。观察者的数量以及谁是观察者可能会有所不同，也可以（在运行时）动态地改变。

可以想到许多观察者模式在其中有用武之地的案例。本章开头已提过这样的一个案例，就是信息聚合。尤论格式为RSS、Atom还是其他，思想都一样：你追随某个信息源，当它每次更新时，你都会收到关于更新的一个通知（请参考[Zlobin13，第60页]）。

同样的概念也存在于社交网络。如果你使用社交网络服务关联了另一个人，在关联的人更新某些内容时，你能收到相关通知，不论这个关联的人是你关注的一个Twitter用户，Facebook上的一个真实朋友，还是LinkdIn上的一位同事。

事件驱动系统是另一个可以使用（通常也会使用）观察者模式的例子。在这种系统中，监听者被用于监听特定事件。监听者正在监听的事件被创建出来时，就会触发它们。这个事件可以是键入（键盘的）某个特定键、移动鼠标或者其他。事件扮演发布者的角色，监听者则扮演观察者的角色。在这里，关键点是单个事件（发布者）可以关联多个监听者（观察者），请参考网页[t.cn/Rqr1Xgj]。

## 实现
本节中，我们将实现一个数据格式化程序。这里描述的想法来源于ActiveState网站上观察者模式用法的Python代码实现（请参考网页[t.cn/Rqr1SDO]）。默认格式化程序是以十进制格式展示一个数值。然而，我们可以添加/注册更多的格式化程序。这个例子中将添加一个十六进制格式化程序和一个二进制格式化程序。每次更新默认格式化程序的值时，已注册的格式化程序就会收到通知，并采取行动。在这里，行动就是以相关的格式展示新的值。

在一些模式中，继承能体现自身价值，观察者模式是这些模式中的一个。我们可以实现一个基类Publisher，包括添加、删除及通知观察者这些公用功能。DefaultFormatter类继承自Publisher，并添加格式化程序特定的功能。我们可以按需动态地添加删除观察者。下面的类图展示了一个使用两个观察者（HexFormatter和BinaryFormatter）的示例。注意，因为类图是静态的，所以尤法展示系统的整个生命周期，只能展示某个特定时间点的系统状态。

从Publisher类开始说起。观察者们保存在列表observers中。add()方法注册一个新的观察者，或者在该观察者已存在时引发一个错误。remove()方法注销一个已有观察者，或者在该观察者尚未存在时引发一个错误。最后，notify()方法则在变化发生时通知所有观察者。


```python
class Publisher:
    def __init__(self):
        self.observers = []

    def add(self, observer):
        if observer not in self.observers:
            self.observers.append(observer)
        else:
            print('Failed to add: {}'.format(observer))

    def remove(self, observer):
        try:
            self.observers.remove(observer)
        except ValueError:
            print('Failed to remove: {}'.format(observer))

    def notify(self):
        [o.notify(self) for o in self.observers]
```

接着是DefaultFormatter类。init()做的第一件事情就是调用基类的init()方法，因为这在Python中没法自动完成。DefaultFormatter实例有自己的名字，这样便于我们跟踪其状态。对于_data变量，我们使用了名称改编来声明不能直接访问该变量。注意，Python中直接访问一个变量始终是可能的（请参考[Lott14，第54页]），不过资深开发人员没有借口这样做，因为代码已经声明不应该这样做。这里使用名称改编是有一个严肃理由的。请继续往下看。DefaultFormatter把_data变量用作一个整数，默认值为零。


```python
class DefaultFormatter(Publisher):
    def __init__(self, name):
        Publisher.__init__(self)
        self.name = name
        self._data = 0
```

\__str\__()方法返回关于发布者名称和_data值的信息。type(self). \__name是一种获取类名的方便技巧，避免硬编码类名。这降低了代码的可读性，却提高了可维护性。是否喜欢，要看你的选择。


```python
def str (self):
    return "{}: '{}' has data = {}".format(type(self).__name__, self.name, self._data)
```

类中有两个data()方法。第一个使用@property修饰器来提供_data变量的读访问方式。这样，我们就能使用object.data来替代object.data()。


```python
@property
def data(self):
    return self._data
```

第二个data()更有意思。它使用了@setter修饰器，该修饰器会在每次使用赋值操作符（=）为_data变量赋新值时被调用。该方法也会尝试把新值强制类型转换为一个整数，并在类型转换失败时处理异常。


```python
@data.setter
def data(self, new_value):
    try:
        self._data = int(new_value)
    except ValueError as e:
        print('Error: {}'.format(e))
    else:
        self.notify()
```

下一步是添加观察者。HexFormatter和BinaryFormatter的功能非常相似。唯一的不同在于如何格式化从发布者那获取到的数据值，即分别以十六进制和二进制进行格式化。


```python
class HexFormatter:
    def notify(self, publisher):
        print("{}: '{}' has now hex data = {}".format(type(self).__name__, publisher.name, hex(publisher.data)))

class BinaryFormatter:
    def notify(self, publisher):
        print("{}: '{}' has now bin data = {}".format(type(self).__name__, publisher.name, bin(publisher.data)))
```

如果没有测试数据，示例就不好玩了。main()函数一开始创建一个名为test1的Default-Formatter实例，并在之后关联了两个可用的观察者。也使用了异常处理来确保在用户输入问题数据时应用不会崩溃。此外，诸如两次添加相同的观察者或删除尚不存在的观察者之类的事情也不应该导致崩溃。


```python
def main():
    df = DefaultFormatter('test1')
    print(df)
    print()
    hf = HexFormatter()
    df.add(hf)
    df.data = 3
    print(df)
    print()
    bf = BinaryFormatter()
    df.add(bf)
    df.data = 21
    print(df)
    print()
    df.remove(hf)
    df.data = 40
    print(df)
    print()
    df.remove(hf)
    df.add(bf)
    df.data = 'hello'
    print(df)
    print()
    df.data = 15.8
    print(df)
```


示例的完整代码（observer.py）如下所示。

```python
class Publisher:
    def __init__(self):
        self.observers = []

    def add(self, observer):
        if observer not in self.observers:
            self.observers.append(observer)
        else:
            print('Failed to add: {}'.format(observer))

    def remove(self, observer):
        try:
            self.observers.remove(observer)
        except ValueError:
            print('Failed to remove: {}'.format(observer))

    def notify(self):
        [o.notify(self) for o in self.observers]

class DefaultFormatter(Publisher):
    def __init__(self, name):
        Publisher.__init__(self)
        self.name = name
        self._data = 0

    def __str__(self):
        return "{}: '{}' has data = {}".format(type(self).__name__, self.name, self._data)

    @property
    def data(self):
        return self._data

    @data.setter
    def data(self, new_value):
        try:
            self._data = int(new_value)
        except ValueError as e:
            print('Error: {}'.format(e))
        else:
            self.notify()

class HexFormatter:
    def notify(self, publisher):
        print("{}: '{}' has now hex data = {}".format(type(self).__name__, publisher.name, hex(publisher.data)))

class BinaryFormatter:
    def notify(self, publisher):
        print("{}: '{}' has now bin data = {}".format(type(self).__name__, publisher.name, bin(publisher.data)))

def main():
    df = DefaultFormatter('test1')
    print(df)

    print()
    hf = HexFormatter()
    df.add(hf)
    df.data = 3
    print(df)

    print()
    bf = BinaryFormatter()
    df.add(bf)
    df.data = 21
    print(df)

    print()
    df.remove(hf)
    df.data = 40
    print(df)

    print()
    df.remove(hf)
    df.add(bf)

    df.data = 'hello'
    print(df)

    print()
    df.data = 15.8
    print(df)

if __name__ == '__main__':
    main()
```

    DefaultFormatter: 'test1' has data = 0

    HexFormatter: 'test1' has now hex data = 0x3
    DefaultFormatter: 'test1' has data = 3

    HexFormatter: 'test1' has now hex data = 0x15
    BinaryFormatter: 'test1' has now bin data = 0b10101
    DefaultFormatter: 'test1' has data = 21

    BinaryFormatter: 'test1' has now bin data = 0b101000
    DefaultFormatter: 'test1' has data = 40

    Failed to remove: <__main__.HexFormatter object at 0x1068730f0>
    Failed to add: <__main__.BinaryFormatter object at 0x106873160>
    Error: invalid literal for int() with base 10: 'hello'
    DefaultFormatter: 'test1' has data = 40

    BinaryFormatter: 'test1' has now bin data = 0b1111
    DefaultFormatter: 'test1' has data = 15


执行observer.py会输出以下内容。
>>> python3 observer.py

DefaultFormatter: 'test1' has data = 0
HexFormatter: 'test1' has now hex data = 0x3
DefaultFormatter: 'test1' has data = 3
HexFormatter: 'test1' has now hex data = 0x15
BinaryFormatter: 'test1' has now bin data = 0b10101
DefaultFormatter: 'test1' has data = 21
BinaryFormatter: 'test1' has now bin data = 0b101000
DefaultFormatter: 'test1' has data = 40

Failed to remove: < main .HexFormatter object at 0x7f30a2fb82e8>
Failed to add: < main .BinaryFormatter object at 0x7f30a2fb8320>
Error: invalid literal for int() with base 10: 'hello'
BinaryFormatter: 'test1' has now bin data = 0b101000
DefaultFormatter: 'test1' has data = 40

BinaryFormatter: 'test1' has now bin data = 0b1111
DefaultFormatter: 'test1' has data = 15

在输出中我们看到，添加额外的观察者，就会出现更多（相关的）输出；一个观察者被删除后，就再也不会被通知到。这正是我们想要的，能够按需启用/禁用运行时通知。

应用的防护性编程方面看起来也工作得不错。尝试玩一些花样都是不会被允许的，比如，删除一个不存在的观察者或者两次添加相同的观察者。不过，显示的信息还不太友好，就留给你作为练习吧。在API要求一个数字参数时输出一个字符串所导致的运行时失败，也能得到正确处理，不会造成应用崩溃/终止。

如果是交互式的，这个例子会有趣得多。即使只是以一个简单的菜单形式允许用户在运行时绑定/解绑观察者或修改DefaultFormatter的值，也是不错的，因为这样能看到更多的运行时方面的信息。请随意来做吧。

另一个不错的练习是添加更多的观察者。例如，可以添加一个八进制格式化程序、罗马数字格式化程序或使用你最爱展现形式的任何其他观察者。发挥你的创意，享受乐趣吧！

## 小结

本章中，我们学习了观察者设计模式。若希望在一个对象的状态变化时能够通知/提醒所有相关者（一个对象或一组对象），则可以使用观察者模式。观察者模式的一个重要特性是，在运行时，订阅者/观察者的数量以及观察者是谁可能会变化，也可以改变。

为理解观察者模式，你可以想一想拍卖会的场景，出价人是订阅者，拍卖师是发布者。这一模式在软件领域的应用非常多。大体上，所有利用MVC模式的系统都是基于事件的。作为具体的例子，我们提到了以下两项。

* django-observer，一个第三方Django库，用于注册在模型字段变更时执行的观察者。
* RabbitMQ的Python绑定。我们介绍了一个RabbitMQ的具体例子，用于实现发布—订阅（即观察者）模式。

在实现例子中，我们看到了如何使用观察者模式创建可在运行时绑定/解绑的数据格式化程序，以此增强对象的行为。希望你会觉得推荐的练习比较有趣。

第14章介绍状态设计模式，该模式可用于实现一个核心的计算机科学概念：状态机。
