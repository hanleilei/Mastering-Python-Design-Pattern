# 代理模式

在某些应用中，我们想要在访问某个对象之前执行一个或多个重要的操作，例如，访问敏感信息——在允许用户访问敏感信息之前，我们希望确保用户具备足够的权限。操作系统中也存在类似的情况，用户必须具有管理员权限才能在系统中安装新程序。

上面提到的重要操作不一定与安全问题相关。延迟初始化（请参考网页[t.cn/Ryf47bV]）是另一个案例：我们想要把一个计算成本较高的对象的创建过程延迟到用户首次真正使用它时才进行。

这类操作通常使用代理设计模式（Proxy design pattern）来实现。该模式因使用代理（又名替代，surrogate）对象在访问实际对象之前执行重要操作而得其名。以下是四种不同的知名代理类型（请参考[GOF95，第234页]和网页[t.cn/RqrYEn9]）。

* 远程代理：实际存在于不同地址空间（例如，某个网络服务器）的对象在本地的代理者。
* 虚拟代理：用于惰性求值，将一个大计算量对象的创建延迟到真正需要的时候进行。
* 保护/防护代理：控制对敏感对象的访问。
* 智能（引用）代理：在对象被访问时执行额外的动作。此类代理的例子包括引用计数和线程安全检查。

我发现虚拟代理非常有用，所以现在通过一个例子来看看可以如何实现它。在9.4节中将学习如何创建防护代理。

使用Python来创建虚拟代理存在很多方式，但我始终喜欢地道的/符合Python风格的实现。这里展示的代码源自网站stackoverflow.com用户Cyclone的一个超赞回答(请参考网页[t.cn/RqrYudC])。为避免混淆，我先说明一下，在本节中，术语特性(property)、变量(variable)、属性（attribute）可相互替代使用。我们先创建一个LazyProperty类，用作一个修饰器。当它修饰某个特性时，LazyProperty惰性地（首次使用时）加载特性，而不是立即进行。init方法创建两个变量，用作初始化待修饰特性的方法的别名。method变量是一个实际方法的别名，method_name变量则是该方法名称的别名。为更好理解如何使用这两个别名，可以将其值输出到标准输出（取消注释下面代码中的两个注释行）。

```python
import time

class SalesManager:
    def work(self):                         
        print("Sales Manager working...")             

    def talk(self):                         
        print("Sales Manager ready to talk")


class Proxy:
    def __init__(self):                         
        self.busy = 'No'                         
        self.sales = None             

    def work(self):
        print("Proxy checking for Sales Manager availability")                         
        if self.busy == 'No':                                      
            self.sales = SalesManager()                                      
            time.sleep(2)
            self.sales.talk()                         
        else:                                      
            time.sleep(2)
            print("Sales Manager is busy")


if __name__ == '__main__':
    p = Proxy()
    p.work()
    p.busy = 'Yes'
    p.work()


```

    Proxy checking for Sales Manager availability
    Sales Manager ready to talk
    Proxy checking for Sales Manager availability
    Sales Manager is busy



```python
## 以下是来自于github的代码：

#!/usr/bin/env python
# -*- coding: utf-8 -*-

import time


class SalesManager:
    def talk(self):
        print("Sales Manager ready to talk")


class Proxy:
    def __init__(self):
        self.busy = 'No'
        self.sales = None

    def talk(self):
        print("Proxy checking for Sales Manager availability")
        if self.busy == 'No':
            self.sales = SalesManager()
            time.sleep(0.1)
            self.sales.talk()
        else:
            time.sleep(0.1)
            print("Sales Manager is busy")


class NoTalkProxy(Proxy):
    def talk(self):
        print("Proxy checking for Sales Manager availability")
        time.sleep(0.1)
        print("This Sales Manager will not talk to you whether he/she is busy or not")


if __name__ == '__main__':
    p = Proxy()
    p.talk()
    p.busy = 'Yes'
    p.talk()
    p = NoTalkProxy()
    p.talk()
    p.busy = 'Yes'
    p.talk()

### OUTPUT ###
# Proxy checking for Sales Manager availability
# Sales Manager ready to talk
# Proxy checking for Sales Manager availability
# Sales Manager is busy
# Proxy checking for Sales Manager availability
# This Sales Manager will not talk to you whether he/she is busy or not
# Proxy checking for Sales Manager availability
# This Sales Manager will not talk to you whether he/she is busy or not
```

    Proxy checking for Sales Manager availability
    Sales Manager ready to talk
    Proxy checking for Sales Manager availability
    Sales Manager is busy
    Proxy checking for Sales Manager availability
    This Sales Manager will not talk to you whether he/she is busy or not
    Proxy checking for Sales Manager availability
    This Sales Manager will not talk to you whether he/she is busy or not



```python
class LazyProperty:
    def __init__(self, method):
        self.method = method
        self.method_name = method.__name__
        # print('function overriden: {}'.format(self.fget))
        # print("function's name: {}".format(self.func_name))
```

LazyProperty类实际上是一个描述符(请参考网页[t.cn/RqrYBND])。描述符(descriptor)是Python中重写类属性访问方法(get()、set()和delete())的默认行为要使用的一种推荐机制。LazyProperty类仅重写了set()，因为这是其需要重写的唯一访问方法。换旬话说，我们尤其需重写所有访问方法。get()方法所访问的特性值，正是下层方法想要赋的值，并使用setattr()来手动赋值。get()实际做的事情非常简单，就是使用值来替代方法！这意味着不仅特性是惰性加载的，而且仅可以设置一次。我们马上就能看到这意味着什么。同样，取消注释以下代码的的注释行，以得到一些额外信息。


```python
def __get__(self, obj, cls):
    if not obj:
        return None
    value = self.method(obj)
    # print('value {}'.format(value))
    setattr(obj, self.method_name, value)
    return value
```

Test类演示了我们可以如何使用LazyProperty类。其中有三个属性，x、y和_resource。我们想懒加载_resource变量，因此将其初始化为None，如以下代码所示。


```python
class Test:
    def __init__(self):
        self.x = 'foo'
        self.y = 'bar'
        self._resource = None
```

resource() 方法是使用LazyProperty类修饰的。因演示H的，LazyProperty类将 \_resource属性初始化为一个tuple，如以下代码所示。通常来说这是一个缓慢/代价大的初始化过程（初始化数据库、图形等）。


```python
@LazyProperty
def resource(self):
    print('initializing self._resource which is: {}'.format(self._resource))
    self._resource = tuple(range(5)) # 假设这一行的计算成本比较大
    return self._resource
```

main()函数展示了惰性求值是如何进行的。注意，get()访问方法的重写使得可以将resource()方法当作一个变量（可以使用t.resource替代t.resource()）。


```python
def main():
    t = Test()
    print(t.x)
    print(t.y)
    # 做更多的事情……
    print(t.resource)
    print(t.resource)
```

从这个例子（文件lazy.py）的执行输出中，可以看出以下几点。

* \_resource变量实际不是在t实例创建时初始化的，而是在我们首次使用t.resource时。
* 第二次使用t.resource之时，并没有再次初始化变量。这就是为什么初始化字符串initializing self.\_resource which is:仅出现一次的原因。
* 下面显示的是lazy.py文件的执行。

>>> python3 lazy.py
    foo
    bar
    self.\_resource which is: None
    (0, 1, 2, 3, 4)
    (0, 1, 2, 3, 4)

在OOP中有两种基本的、不同类型的惰性求值，如下所示。
* 在实例级：这意味着会一个对象的特性进行惰性求值，但该特性有一个对象作用域。同一个类的每个实例（对象）都有自己的（不同的）特性副本。
* 在类级或模块级：在这种情况下，我们不希望每个实例都有一个不同的特性副本，而是所有实例共享同一个特性，而特性是惰性求值的。这一情况在本章不涉及。如果你觉得有意思，可以将其作为练习。

## 现实生活的例子

芯片（又名芯片密码）卡(请参考网页[t.cn/RqrYdYx])是现实生活中使用防护代理的一个好例子。借记/信用卡包含一个芯片，ATM机或读卡器需要先读取芯片；在芯片通过验证后，需要一个密码（PIN）才能完成交易。这意味着只有在物理地提供芯片卡并且知道密码时才能进行交易。

使用银行支票替代现金进行购买和交易是远程代理的一个例子。支票准许了对一个银行账户的访问。下图展示了支票如何用作一个远程代理(请参考网页[t.cn/RqrYEn9])，经sourcemaking.com允许使用。

## 软件的例子
Python的weakref模块包含一个proxy()方法，该方法接受一个输入对象并将一个智能代理返回给该对象。弱引用是为对象添加引用计数支持的一种推荐方式(请参考网页[t.cn/RqrT7cC])。
ZeroMQ(请参考网页[t.cn/zWzWCrR])是一组专注于分布式计算的自由开源软件项目。ZeroMQ的Python实现有一个代理模块，实现了一个远程代理。该模块允许Tornado(请参考网页[t.cn/RhFErfr])的处理程序在不同的远程进程中运行(请参考网页[t.cn/RqrTbY9])。

## 应用案例

因为存在至少四种常见的代理类型，所以代理设计模式有很多应用案例，如下所示。

* 在使用私有网络或云搭建一个分布式系统时。在分布式系统中，一些对象存在于本地内存中，一些对象存在于远程计算机的内存中。如果我们不想本地代码关心两者之间的区别，那么可以创建一个远程代理来隐藏/封装，使得应用的分布式性质透明化。
* 因过早创建计算成本较高的对象导致应用遭受性能问题之时。使用虚拟代理引入惰性求值，仅在真正需要对象之时才创建，能够明显提高性能。
* 用于检查一个用户是否有足够权限来访问某个信息片段。如果应用要处理敏感信息（例如，医疗数据），我们会希望确保用户在被准许之后才能访问/修改数据。一个保护/防护代理可以处理所有安全相关的行为。
* 应用（或库、工具集、框架等）使用多线程，而我们希望把线程安全的重任从客户端代码转移到应用。这种情况下，可以创建一个智能代理，对客户端隐藏线程安全的复杂性。
* 对象关系映射（Object-Relational Mapping，ORM）API也是一个如何使用远程代理的例子。包括Django在内的许多流行Web框架使用一个ORM来提供类OOP的关系型数据库访问。ORM是关系型数据库的代理，数据库可以部署在任意地方，本地或远程服务器都可以。

## 实现
为演示代理模式，我们将实现一个简单的保护代理来查看和添加用户。该服务提供以下两个选项。

* 查看用户列表：这一操作不要求特殊权限。
* 添加新用户：这一操作要求客户端提供一个特殊的密码。

SensitiveInfo类包含我们希望保护的信息。users变量是已有用户的列表。read()方法输出用户列表。add()方法将一个新用户添加到列表中。考虑一下下面的代码。


```python
class SensitiveInfo:
    def __init__(self):
        self.users = ['nick', 'tom', 'ben', 'mike']

    def read(self):
        print('There are {} users: {}'.format(len(self.users), ' '.join(self.users)))

    def add(self, user):
        self.users.append(user)
        print('Added user {}'.format(user))
```

Info类是SensitiveInfo的一个保护代理。secret变量值是客户端代码在添加新用户时被要求告知/提供的密码。注意，这只是一个例子。现实中，永远不要执行以下操作。

* 在源代码中存储密码
* 以明文形式存储密码
* 使用一种弱（例如，MD5）或自定义加密形式

read()方法是SensetiveInfo.read()的一个包装。add()方法确保仅当客户端代码知道密码时才能添加新用户。考虑一下下面的代码。


```python
class Info:
    def __init__(self):
        self.protected = SensitiveInfo()
        self.secret = '0xdeadbeef'

    def read(self):
        self.protected.read()

    def add(self, user):
        sec = input('what is the secret? ')
        self.protected.add(user) if sec == self.secret else print("That's wrong!")
```

main()函数展示了客户端代码可以如何使用代理模式。客户端代码创建一个Info类的实例，并使用菜单让用户选择来读取列表、添加新用户或退出应用。考虑一下下面的代码。


```python
def main():
    info = Info()
    while True:
        print('1. read list |==| 2. add user |==| 3. quit')
        key = input('choose option: ')
        if key == '1':
            info.read()
        elif key == '2':
            name = input('choose username: ')
            info.add(name)
        elif key == '3':
            exit()
        else:
            print('unknown option: {}'.format(key))
```

现在看一下proxy.py文件的完整代码。


```python
class SensitiveInfo:
    def __init__(self):
        self.users = ['nick', 'tom', 'ben', 'mike']

    def read(self):
        print('There are {} users: {}'.format(len(self.users), ' '.join(self.users)))

    def add(self, user):
        self.users.append(user)
        print('Added user {}'.format(user))

class Info:
    '''protection proxy to SensitiveInfo'''

    def __init__(self):
        self.protected = SensitiveInfo()
        self.secret = '0xdeadbeef'

    def read(self):
        self.protected.read()

    def add(self, user):
        sec = input('what is the secret? ')
        self.protected.add(user) if sec == self.secret else print("That's wrong!")

def main():
    info = Info()

    while True:
        print('1. read list |==| 2. add user |==| 3. quit')
        key = input('choose option: ')
        if key == '1':
            info.read()
        elif key == '2':
            name = input('choose username: ')
            info.add(name)
        elif key == '3':
            exit()

if __name__ == '__main__':
    main()
```

    1. read list |==| 2. add user |==| 3. quit
    choose option: 1
    There are 4 users: nick tom ben mike
    1. read list |==| 2. add user |==| 3. quit
    choose option: 2
    choose username: oliver
    what is the secret? 1
    That's wrong!
    1. read list |==| 2. add user |==| 3. quit
    choose option: 2
    choose username: oliver
    what is the secret? 3
    That's wrong!
    1. read list |==| 2. add user |==| 3. quit
    choose option: 3
    1. read list |==| 2. add user |==| 3. quit
    choose option: 3
    1. read list |==| 2. add user |==| 3. quit



    ---------------------------------------------------------------------------

    KeyboardInterrupt                         Traceback (most recent call last)

    /Users/hanlei/anaconda/lib/python3.5/site-packages/ipykernel/kernelbase.py in \_input_request(self, prompt, ident, parent, password)
        713             try:
    --> 714                 ident, reply = self.session.recv(self.stdin_socket, 0)
        715             except Exception:


    /Users/hanlei/anaconda/lib/python3.5/site-packages/jupyter_client/session.py in recv(self, socket, mode, content, copy)
        738         try:
    --> 739             msg_list = socket.recv_multipart(mode, copy=copy)
        740         except zmq.ZMQError as e:


    /Users/hanlei/anaconda/lib/python3.5/site-packages/zmq/sugar/socket.py in recv_multipart(self, flags, copy, track)
        357         """
    --> 358         parts = [self.recv(flags, copy=copy, track=track)]
        359         # have first part already, only loop while more to receive


    zmq/backend/cython/socket.pyx in zmq.backend.cython.socket.Socket.recv (zmq/backend/cython/socket.c:6971)()


    zmq/backend/cython/socket.pyx in zmq.backend.cython.socket.Socket.recv (zmq/backend/cython/socket.c:6763)()


    zmq/backend/cython/socket.pyx in zmq.backend.cython.socket.\_recv_copy (zmq/backend/cython/socket.c:1931)()


    /Users/hanlei/anaconda/lib/python3.5/site-packages/zmq/backend/cython/checkrc.pxd in zmq.backend.cython.checkrc.\_check_rc (zmq/backend/cython/socket.c:7222)()


    KeyboardInterrupt:


    During handling of the above exception, another exception occurred:


    KeyboardInterrupt                         Traceback (most recent call last)

    <ipython-input-1-cf1cc63045e1> in <module>()
         39
         40 if __name__ == '__main__':
    ---> 41     main()


    <ipython-input-1-cf1cc63045e1> in main()
         29     while True:
         30         print('1. read list |==| 2. add user |==| 3. quit')
    ---> 31         key = input('choose option: ')
         32         if key == '1':
         33             info.read()


    /Users/hanlei/anaconda/lib/python3.5/site-packages/ipykernel/kernelbase.py in raw_input(self, prompt)
        687             self.\_parent_ident,
        688             self.\_parent_header,
    --> 689             password=False,
        690         )
        691


    /Users/hanlei/anaconda/lib/python3.5/site-packages/ipykernel/kernelbase.py in \_input_request(self, prompt, ident, parent, password)
        717             except KeyboardInterrupt:
        718                 # re-raise KeyboardInterrupt, to truncate traceback
    --> 719                 raise KeyboardInterrupt
        720             else:
        721                 break


    KeyboardInterrupt:


下面是一个如何执行proxy.py的示例。

>>> python3 proxy.py
1. read list |==| 2. add user |==| 3. quit choose option: a
1. read list |==| 2. add user |==| 3. quit choose option: 4
1. read list |==| 2. add user |==| 3. quit choose option: 1
There are 4 users: nick tom ben mike
1. read list |==| 2. add user |==| 3. quit choose option: 2
choose username: pet what is the secret? blah That's wrong!
1. read list |==| 2. add user |==| 3. quit choose option: 2
choose username: bill
what is the secret? 0xdeadbeef Added user bill
1. read list |==| 2. add user |==| 3. quit choose option: 1
There are 5 users: nick tom ben mike bill
1. read list |==| 2. add user |==| 3. quit choose option: 3

你已经发现这个代理示例中可以改进的缺陷或缺失特性了吗？我有如下一些建议。

* 该示例有一个非常大的安全缺陷。没有什么能阻止客户端代码通过直接创建一个SensitveInfo实例来绕过应用的安全设置。优化示例来阻止这种情况。一种方式是使用abc模块来禁止直接实例化SensitiveInfo。在这种情况下，会要求进行其他哪些代码变更呢？
* 一个基本的安全原则是，我们绝不应该存储明文密码。只要我们知道使用哪个库，安全地存储密码并不是一件难事（请参考网页[t.cn/zQf0g3c]）。如果你对安全感兴趣，阅读这篇文章，并尝试实现一种外部存储密码的安全方式（例如，在一个文件或数据库中）。
* 应用仅支持添加新用户，那么如何删除一个已有用户呢？添加一个remove()方法。

remove()应该是一个特权操作吗？

## 小结

本章中，你学习了如何使用代理设计模式。我们使用代理模式实现一个实际类的替代品，这样可以在访问实际类之前（或之后）做一些额外的事情。存在四种不同的代理类型，如下所示。

* 远程代理，代表一个活跃于远程位置（例如，我们自己的远程服务器或云服务）的对象。
* 虚拟代理，将一个对象的初始化延迟到真正需要使用时进行。
* 保护/防护代理，用于对处理敏感信息的对象进行访问控制。
* 当我们希望通过添加帮助信息（比如，引用计数）来扩展一个对象的行为时，可以使用智能（引用）代理。

在第一个代码示例中，我们使用修饰器和描述符以地道的Python风格创建一个虚拟代理。这个代理允许我们以惰性方式初始化对象属性。

芯片卡和银行支票是人们每天都在使用的两个不同代理的例子。芯片卡是一个防护代理，而银行支票是一个远程代理。另外，一些流行软件中也使用代理。Python有一个weakref.proxy()方法，使得创建一个智能代理非常简单。ZeroMQ的Python实现则使用了远程代理。

我们讨论了几个代理模式的应用案例，包括性能、安全及向用户提供简单的API。在第二个代码示例中，我们实现一个保护代理来处理用户信息。这个例子可以以多种方式进行改进，特别是关于其安全缺陷和用户列表实际上未持久化（永久存储）的问题。希望你会发现推荐练习比较有意思。
