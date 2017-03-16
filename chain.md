# 责任链模式

开发一个应用时，多数时候我们都能预先知道哪个方法能处理某个特定请求。然而，情况并非总是如此。例如，想想任意一种广播计算机网络，例如最早的以太网实现（请参考网页[t.cn/RqrTp0Y]）。在广播计算机网络中，会将所有请求发送给所有节点（简单起见，不考虑广播域），但仅对所发送请求感兴趣的节点会处理请求。加入广播网络的所有计算机使用一种常见的媒介相互连接，比如，下图中的三个节点通过光缆连接起来。

如果一个节点对某个请求不感兴趣或者不知道如何处理这个请求，可以执行以下两个操作。
* 忽略这个请求，什么都不做
* 将请求转发给下一个节点

节点对一个请求的反应方式是实现的细节。然而，我们可以使用广播计算机网络的类比来理解责任链模式是什么。责任链（Chain of Responsibility）模式用于让多个对象来处理单个请求时，或者用于预先不知道应该由哪个对象（来自某个对象链）来处理某个特定请求时。其原则如下所示。

(1) 存在一个对象链（链表、树或任何其他便捷的数据结构）。
(2) 我们一开始将请求发送给链中的第一个对象。
(3) 对象决定其是否要处理该请求。
(4) 对象将请求转发给下一个对象。
(5) 重复该过程，直到到达链尾。

在应用级别，不用讨论光缆和网络节点，而是可以专注于对象以及请求的流程。下图展示了客户端代码如何将请求发送给应用的所有处理元素（又称为节点或处理程序），经www.sourcema-king.com允许使用（请参考网页[t.cn/RqrTYuB]）。


注意，客户端代码仅知道第一个处理元素，而非拥有对所有处理元素的引用；并且每个处理元素仅知道其直接的下一个邻居（称为后继），而不知道所有其他处理元素。这通常是一种单向关系，用编程术语来说是一个单向链表，与之相反的是双向链表。单向链表不允许双向地遍历元素，双向链表则是允许的。这种链式组织方式大有用处：可以解耦发送方（客户端）和接收方（处理元素）（请参考[GOF95，第254页]）。

以下例子来自于GitHub：

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
"""http://www.dabeaz.com/coroutines/"""

from contextlib import contextmanager
import os
import sys
import time


class Handler(object):

    def __init__(self, successor=None):
        self._successor = successor

    def handle(self, request):
        res = self._handle(request)
        if not res:
            self._successor.handle(request)

    def _handle(self, request):
        raise NotImplementedError('Must provide implementation in subclass.')


class ConcreteHandler1(Handler):

    def _handle(self, request):
        if 0 < request <= 10:
            print('request {} handled in handler 1'.format(request))
            return True


class ConcreteHandler2(Handler):

    def _handle(self, request):
        if 10 < request <= 20:
            print('request {} handled in handler 2'.format(request))
            return True


class ConcreteHandler3(Handler):

    def _handle(self, request):
        if 20 < request <= 30:
            print('request {} handled in handler 3'.format(request))
            return True


class DefaultHandler(Handler):

    def _handle(self, request):
        print('end of chain, no handler for {}'.format(request))
        return True


class Client(object):

    def __init__(self):
        self.handler = ConcreteHandler1(
            ConcreteHandler3(ConcreteHandler2(DefaultHandler())))

    def delegate(self, requests):
        for request in requests:
            self.handler.handle(request)


def coroutine(func):
    def start(*args, **kwargs):
        cr = func(*args, **kwargs)
        next(cr)
        return cr
    return start


@coroutine
def coroutine1(target):
    while True:
        request = yield
        if 0 < request <= 10:
            print('request {} handled in coroutine 1'.format(request))
        else:
            target.send(request)


@coroutine
def coroutine2(target):
    while True:
        request = yield
        if 10 < request <= 20:
            print('request {} handled in coroutine 2'.format(request))
        else:
            target.send(request)


@coroutine
def coroutine3(target):
    while True:
        request = yield
        if 20 < request <= 30:
            print('request {} handled in coroutine 3'.format(request))
        else:
            target.send(request)


@coroutine
def default_coroutine():
    while True:
        request = yield
        print('end of chain, no coroutine for {}'.format(request))


class ClientCoroutine:

    def __init__(self):
        self.target = coroutine1(coroutine3(coroutine2(default_coroutine())))

    def delegate(self, requests):
        for request in requests:
            self.target.send(request)


def timeit(func):

    def count(*args, **kwargs):
        start = time.time()
        res = func(*args, **kwargs)
        count._time = time.time() - start
        return res
    return count


@contextmanager
def suppress_stdout():
    try:
        stdout, sys.stdout = sys.stdout, open(os.devnull, 'w')
        yield
    finally:
        sys.stdout = stdout


if __name__ == "__main__":
    client1 = Client()
    client2 = ClientCoroutine()
    requests = [2, 5, 14, 22, 18, 3, 35, 27, 20]

    client1.delegate(requests)
    print('-' * 30)
    client2.delegate(requests)

    requests *= 10000
    client1_delegate = timeit(client1.delegate)
    client2_delegate = timeit(client2.delegate)
    with suppress_stdout():
        client1_delegate(requests)
        client2_delegate(requests)
    # lets check what is faster
    print(client1_delegate._time, client2_delegate._time)

### OUTPUT ###
# request 2 handled in handler 1
# request 5 handled in handler 1
# request 14 handled in handler 2
# request 22 handled in handler 3
# request 18 handled in handler 2
# request 3 handled in handler 1
# end of chain, no handler for 35
# request 27 handled in handler 3
# request 20 handled in handler 2
# ------------------------------
# request 2 handled in coroutine 1
# request 5 handled in coroutine 1
# request 14 handled in coroutine 2
# request 22 handled in coroutine 3
# request 18 handled in coroutine 2
# request 3 handled in coroutine 1
# end of chain, no coroutine for 35
# request 27 handled in coroutine 3
# request 20 handled in coroutine 2
# (0.2369999885559082, 0.16199994087219238)
```

## 现实生活的例子

ATM机以及及一般而言用于接收/返回钞票或硬币的任意类型机器（比如，零食自动贩卖机）都使用了责任链模式。机器上总会有一个放置各种钞票的槽口，如下图所示（经www.sourcemaking.com允许使用）。

钞票放入之后，会被传递到恰当的容器。钞票返回时，则是从恰当的容器中获取（请参考网页[t.cn/RqrTYuB]和网页[t.cn/RqrTnts]）。我们可以把这个槽口视为共享通信媒介，不同的容器则是处理元素。结果包含来自一个或多个容器的现金。例如，在上图中，我们看到在从ATM机取175美元时会发生什么。

## 软件的例子

我试过寻找一些使用责任链模式的Python应用的好例子，但是没找到，很可能是因为Python程序员不使用这个名称。因此，很抱歉，我将使用其他编程语言的例子作为参考。

Java的servlet过滤器是在一个HTTP请求到达H标处理程序之前执行的一些代码片段。在使用servlet过滤器时，有一个过滤器链，其中每个过滤器执行一个不同动作（用户身份验证、记H志、数据压缩等），并且将请求转发给下一个过滤器直到链结束；如果发生错误（例如，连续三次身份验证失败）则跳出处理流程（请参考网页[t.cn/RqrTukH]）。

Apple的Cocoa和Cocoa Touch框架使用责任链来处理事件。在某个视图接收到一个其并不知道如何处理的事件时，会将事件转发给其超视图，直到有个视图能够处理这个事件或者视图链结束（请参考网页[t.cn/RqrTrzK]）。

## 应用案例

通过使用责任链模式，我们能让许多不同对象来处理一个特定请求。在我们预先不知道应该由哪个对象来处理某个请求时，这是有用的。其中一个例子是采购系统。在采购系统中，有许多核准权限。某个核准权限可能可以核准在一定额度之内的订单，假设为100美元。如果订单超过了100美元，则会将订单发送给链中的下一个核准权限，比如能够核准在200美元以下的订单，等等。

另一个责任链可以派上用场的场景是，在我们知道可能会有多个对象都需要对同一个请求进行处理之时。这在基于事件的编程中是常有的事情。单个事件，比如一次鼠标左击，可被多个事件监听者捕获。

不过应该注意，如果所有请求都能被单个处理程序处理，责任链就没那么有用了，除非确实不知道会是哪个程序处理请求。这一模式的价值在于解耦。客户端与所有处理程序（一个处理程序与所有其他处理程序之间也是如此）之间不再是多对多关系，客户端仅需要知道如何与链的起始节点（标头）进行通信。

下图演示了紧耦合与松耦合之间的区别心。松耦合系统背后的考虑是简化维护，并让我们易于理解系统的工作原理（请参考网页https://infomgmt.wordpress.com/2010/02/18/a-visual-respresen-tation-of-coupling/）。

数据耦合（data coupling）、特征耦合（stamp coupling）、控制耦合（control coupling）、共用耦合（common coupling）和内容耦合（content coupling）这几个概念的含义可参考Wikipedia词条 https://en.wikipedia.org/wiki/Coupling_(computer_programming)。 ——译者注

## 实现
使用Python实现责任链模式有许多种方式，但是我最喜欢的实现是Vespe Savikko所提出的（请参考网页[t.cn/RqruSj1])。Vespe的实现以地道的Python风格使用动态分发来处理请求（请参考网页[t.cn/RqruWFp]）。

我们以Vespe的实现为参考实现一个简单的事件系统。下面是该系统的UML类图。

Event类描述一个事件。为了让它简单一点，在我们的案例中一个事件只有一个name属性。

```python
class Event:

    def __init__(self, name):
        self.name = name

    def __str__(self):
        return self.name
```

Widget类是应用的核心类。UML图中展示的parent聚合关系表明每个控件都有一个到父对象的引用。按照约定，我们假定父对象是一个Widget实例。然而，注意，根据继承的规则，任何Widget子类的实例（例如，MsgText的实例）也是Widget实例。parent的默认值为None。


```python
class Widget:
    def __init__(self, parent=None):
        self.parent = parent
```

handle()方法使用动态分发，通过hasattr()和getattr()决定一个特定请求（event）应该由谁来处理。如果被请求处理事件的控件并不支持该事件，则有两种回退机制。如果控件有parent，则执行parent的handle()方法。如果控件没有parent，但有handle_default()方法，则执行handle_default()。

```python
def handle(self, event):
    handler = 'handle_{}'.format(event)
    if hasattr(self, handler):
        method = getattr(self, handler)
        method(event)
    elif self.parent:
        self.parent.handle(event)
    elif hasattr(self, 'handle_default'):
        self.handle_default(event)
```

此时，你可能已明臼为什么UML类图中Widget与Event类仅是关联关系而已（不是聚合或组合关系）。关联关系用于表明Widget类知道Event类，但对其没有任何严格的引用，因为事件仅需要作为参数传递给handle()。

MainWindow、MsgText和SendDialog是具有不同行为的控件。我们并不期望这三个控件都能处理相同的事件，即使它们能处理相同事件，表现出来也可能是不同的。MainWindow仅能处理close和default事件。


```python
class MainWindow(Widget):
    def handle_close(self, event):
        print('MainWindow: {}'.format(event))
    def handle_default(self, event):
        print('MainWindow Default: {}'.format(event))
```

SendDialog仅能处理paint事件。


```python
class SendDialog(Widget):
        def handle_paint(self, event):
            print('SendDialog: {}'.format(event))
```

最后，MsgText仅能处理down事件。

```python
class MsgText(Widget):
    def handle_down(self, event):
        print('MsgText: {}'.format(event))
```

main()函数展示如何创建一些控件和事件，以及控件如何对那些事件作出反应。所有事件都会被发送给所有控件。注意其中每个控件的父子关系。sd对象（SendDialog的一个实例）的父对象是mw（MainWindow的一个实例）。然而，并不是所有对象都需要一个MainWindow实例的父对象。例如，msg对象（MsgText的一个实例）是以sd作为父对象。


```python
def main(): 5 mw = MainWindow()
    sd = SendDialog(mw)
    msg = MsgText(sd)
    for e in ('down', 'paint', 'unhandled', 'close'):
        evt = Event(e)
        print('\nSending event -{}- to MainWindow'.format(evt))
        mw.handle(evt)
        print('Sending event -{}- to SendDialog'.format(evt))
        sd.handle(evt)
        print('Sending event -{}- to MsgText'.format(evt))
        msg.handle(evt)
```

以下是示例的完整代码（chain.py）。

```python
class Event:
    def __init__(self, name):
        self.name = name

    def __str__(self):
        return self.name

class Widget:
    def __init__(self, parent=None):
        self.parent = parent

    def handle(self, event):
        handler = 'handle_{}'.format(event)
        if hasattr(self, handler):
            method = getattr(self, handler)
            method(event)
        elif self.parent:
            self.parent.handle(event)
        elif hasattr(self, 'handle_default'):
            self.handle_default(event)

class MainWindow(Widget):
    def handle_close(self, event):
        print('MainWindow: {}'.format(event))

    def handle_default(self, event):
        print('MainWindow Default: {}'.format(event))

class SendDialog(Widget):
    def handle_paint(self, event):
        print('SendDialog: {}'.format(event))

class MsgText(Widget):
    def handle_down(self, event):
        print('MsgText: {}'.format(event))

def main():
    mw = MainWindow()
    sd = SendDialog(mw)
    msg = MsgText(sd)

    for e in ('down', 'paint', 'unhandled', 'close'):
        evt = Event(e)
        print('\nSending event -{}- to MainWindow'.format(evt))
        mw.handle(evt)
        print('Sending event -{}- to SendDialog'.format(evt))
        sd.handle(evt)
        print('Sending event -{}- to MsgText'.format(evt))
        msg.handle(evt)

if __name__ == '__main__':
    main()
```


    Sending event -down- to MainWindow
    MainWindow Default: down
    Sending event -down- to SendDialog
    MainWindow Default: down
    Sending event -down- to MsgText
    MsgText: down

    Sending event -paint- to MainWindow
    MainWindow Default: paint
    Sending event -paint- to SendDialog
    SendDialog: paint
    Sending event -paint- to MsgText
    SendDialog: paint

    Sending event -unhandled- to MainWindow
    MainWindow Default: unhandled
    Sending event -unhandled- to SendDialog
    MainWindow Default: unhandled
    Sending event -unhandled- to MsgText
    MainWindow Default: unhandled

    Sending event -close- to MainWindow
    MainWindow: close
    Sending event -close- to SendDialog
    MainWindow: close
    Sending event -close- to MsgText
    MainWindow: close


从输出中我们能看到一些有趣的东西。例如，发送一个down事件给MainWindow，最终被MainWindow默认处理函数处理。另一个不错的用例是，虽然close事件不能被SendDialog和MsgText直接处理，但所有close事件最终都能被MainWindow正确处理。这正是使用父子关系作为一种回退机制的优美之处。

如果你想在这个事件例子上花费更多时间发挥自己的创意，可以替换这些愚蠢的print语旬，针对罗列出来的事件添加一些实际的行为。当然，并不限于罗列出来的事件。随意添加一些你喜欢的事件，做一些有用的事情！

另一个练习是在运行时添加一个MsgText实例，以MainWindow为其父。这个有难度吗？也挑个事件类型来试试（为一个已有控件添加一个新的事件），哪个更难？

## 小结

本章中，我们学习了责任链设计模式。在尤法预先知道处理程序的数量和类型时，该模式有助于对请求/处理事件进行建模。适合使用责任链模式的系统例子包括基于事件的系统、采购系统和运输系统。

在责任链模式中，发送方可直接访问链中的首个节点。若首个节点不能处理请求，则转发给下一个节点，如此直到请求被某个节点处理或者整个链遍历结束。这种设计用于实现发送方与接收方（多个）之间的解耦。

ATM机是责任链的一个例子。用于取放钞票的槽口可看作是链的头部。从这里开始，根据具体交易，一个或多个容器会被用于处理交易。这些容器可看作是链中的处理程序。

Java的servlet过滤器使用责任链模式对一个HTTP请求执行不同的动作（例如，压缩和身份验证）。Apple的Cocoa框架使用相同的模式来处理事件，比如，按钮和手势。
