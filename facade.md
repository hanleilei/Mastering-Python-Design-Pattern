# 外观模式

系统会随着演化变得非常复杂，最终形成大量的(并且有时是令人迷惑的)类和交互，这种情况并不少见。许多情况下，我们并不想把这种复杂性暴露给客户端。外观设计模式有助于隐藏系统的内部复杂性，并通过一个简化的接口向客户端暴露必要的部分(请参考[Eckel08，第209 页])。本质上，外观(Facade)是在已有复杂系统之上实现的一个抽象层。

下图演示了外观的角色。这张图是Wikipedia上外观模式Java语言示例的类图表示(请参考网页[t.cn/Rqrl38m])。计算机是一个复杂的机器，全功能运行依赖多个部件。为简化表述，这里所说的计算机是指IBM衍生的那一类，使用冯·诺依曼架构。启动一台计算机是一个相当复杂的过程。CPU、内存以及硬盘都需要加电运行；引导加载程序需要从硬盘加载到内存，CPU则必须启动操作系统内核，等等。我们不会把这些复杂性暴露给客户端，而是创造一个外观来封装整个过程，并保证所有步骤按照正确的次序运行。

从图中展示的类可知，仅Computer类需要暴露给客户端代码。客户端仅执行Computer的Start()方法。所有其他复杂部件都由外观类Computer来维护。


```python
# 请看如下所示，来自于github的例子：

#!/usr/bin/env python
# -*- coding: utf-8 -*-

import time

SLEEP = 0.1


# Complex Parts
class TC1:

    def run(self):
        print("###### In Test 1 ######")
        time.sleep(SLEEP)
        print("Setting up")
        time.sleep(SLEEP)
        print("Running test")
        time.sleep(SLEEP)
        print("Tearing down")
        time.sleep(SLEEP)
        print("Test Finished\n")


class TC2:

    def run(self):
        print("###### In Test 2 ######")
        time.sleep(SLEEP)
        print("Setting up")
        time.sleep(SLEEP)
        print("Running test")
        time.sleep(SLEEP)
        print("Tearing down")
        time.sleep(SLEEP)
        print("Test Finished\n")


class TC3:

    def run(self):
        print("###### In Test 3 ######")
        time.sleep(SLEEP)
        print("Setting up")
        time.sleep(SLEEP)
        print("Running test")
        time.sleep(SLEEP)
        print("Tearing down")
        time.sleep(SLEEP)
        print("Test Finished\n")


# Facade
class TestRunner:

    def __init__(self):
        self.tc1 = TC1()
        self.tc2 = TC2()
        self.tc3 = TC3()
        self.tests = [self.tc1， self.tc2， self.tc3]

    def runAll(self):
        [i.run() for i in self.tests]


# Client
if __name__ == '__main__':
    testrunner = TestRunner()
    testrunner.runAll()

### OUTPUT ###
# ###### In Test 1 ######
# Setting up
# Running test
# Tearing down
# Test Finished
#
# ###### In Test 2 ######
# Setting up
# Running test
# Tearing down
# Test Finished
#
# ###### In Test 3 ######
# Setting up
# Running test
# Tearing down
# Test Finished
#
```

    ###### In Test 1 ######
    Setting up
    Running test
    Tearing down
    Test Finished

    ###### In Test 2 ######
    Setting up
    Running test
    Tearing down
    Test Finished

    ###### In Test 3 ######
    Setting up
    Running test
    Tearing down
    Test Finished



## 现实生活中的例子

在现实中，外观模式相当常见。当你致电一个银行或公司，通常是先被连线到客服部门，客服职员在你和业务部门(结算、技术支持、一般援助等)及帮你解决具体问题的职员之间充当一个外观的角色。下图由sourcemaking.com提供，以图表形式展示了这个例子(请参考网页[t.cn/RqrlrtI])。

也可以将汽车或摩托车的启动钥匙视为一个外观。外观是激活一个系统的便捷方式，系统的内部则非常复杂。当然，对于其他可以通过一个简单按钮就能激活的复杂电子设备，同样可以如此看待，比如计算机。

## 软件中的例子

django-oscar-datacash模块是Django的一个第三方组件，用于集成DataCash支付网关。该组件有一个Gateway类，提供对多种DataCash API的细粒度访问。在那之上，它也包含一个Facade类，提供粗粒度API(提供给那些不需要处理细节的人)，并针对审计目的提供保存事务的能力(请参考网页[t.cn/RqrlgCG])。

Caliendo是一个用于模拟Python API的的接口，它包含一个facade模块。该模块使用外观模式来完成许多不同但有用的事情(比如缓存方法)，并基于传给顶层Facade方法的输入对象决定返回什么方法(请参考网页[t.cn/RqrlkiU])。

## 应用案例

使用外观模式的最常见理由是为一个复杂系统提供单个简单的入口点。引入外观之后，客户端代码通过简单地调用一个方法/函数就能使用一个系统。同时，内部系统并不会丢失任何功能，外观只是封装了内部系统。

**不把系统的内部功能暴露给客户端代码有一个额外的好处：我们可以改变系统内部，但客户端代码不用关心这个改变，也不会受这个改变的影响**。客户端代码不需要进行任何改变(请参考[Zlobin13，第44页])。

如果你的系统包含多层，外观模式也能派上用场。你可以为每一层引入一个外观入口点，并让所有层级通过它们的外观相互通信。这提高了层级之间的松耦合性，尽可能保持层级独立(请参考[GOF95，第209页])。

## 实现

假设我们想使用多服务进程方式实现一个操作系统，类似于MINIX 3(请参考网页 [t.cn/h5mI2X])或GNU Hurd(请参考网页[t.cn/RqrjZA1])那样。多服务进程的操作系统有一个极小的内核，称为微内核(microkernel)，它在特权模式下运行。系统的所有其他服务都遵从一种服务架构(驱动程序服务器、进程服务器、文件服务器等)。每个服务进程属于一个不同的内存地址空间，以用户模式在微内核之上运行。这种方式的优势是操作系统更能容错、更加可靠、更加安全。例如，由于所有驱动程序都以用户模式在一个驱动服务进程之上运行，所以某个驱动程序中的一个bug并不能让整个系统崩溃，也无法影响到其他服务进程。其劣势则是性能开销和系统编程的复杂性，因为服务进程和微内核之间，还有独立的服务进程之间，使用消息传递方式进行通信。消息传递比宏内核(如Linux)所使用的共享内存模型更加复杂(请参考网页[t.cn/RqrjAK8])。

我们从Server接口(这里的“接口”并非指语法上的interface，而是指一个不能直接实例化的类)开始实现，使用一个Enum类型变量来描述一个服务进程的不同状态，使用abc模块来禁止对Server接口直接进行初始化，并强制子类实现关键的boot()和kill() 方法。这里假设每个服务进程的启动、关闭及重启都相应地需要不同的动作。如果你以前没用过abc模块，请记住以下几个重要事项。

* 我们需要使用metaclass关键字来继承ABCMeta。
* 使用@abstractmethod修饰器来声明Server的所有子类都应(强制性地)实现哪些方法。

尝试移除一个子类的boot()或kill()方法，看看会发生什么。移除@abstractmethod修饰器之后再试试。一切如你所料吗?

我们来思考以下这段代码。


```python
from enum import Enum
from abc import ABCMeta， abstractmethod

State = Enum('State'， 'new running sleeping restart zombie')

class server(metaclass=ABCMeta):

    @abstractmethod
    def __init__(self):
        pass

    def __str__(self):
        return self.name

    @abstractmethod
    def boot(self):
        pass

    @abstractmethod
    def kill(self， restart=True):
        pass
```

一个模块化的操作系统可以有很多有意思的服务进程，其中包括文件服务进程、进程服务进程、身份验证服务进程、网络服务进程和图形/窗口服务进程等。下面这个例子包含两个存根服务进程(FileServer和ProcessServer)。除了Server接口要求实现的方法之外，每个服务进程还可以具有自己特有的方法。例如，FileServer有一个create_file()方法用于创建文件，ProcessServer有一个create_process()方法用于创建进程。


```python
class FileServer(server):

    def __init__(self):
        '''初始化文件服务进程要求的操作'''
        self.name = 'FileServer'
        self.state = State.new

    def boot(self):
        print('booting the {}'.format(self))
        '''启动文件服务进程要求的操作'''
        self.state = State.running

    def kill(self， restart=True):
        print('Killing {}'.format(self))
        '''终止文件服务进程要求的操作'''
        self.state = State.restart if restart else State.zombie

    def create_file(self， user， name， permissions):
        '''检查访问权限的有效性和用户权限等'''
        print("trying to create the file '{}' for user '{}' with permissions {}".format(name， user， permissions))

class ProcessServer(server):
    def __init__(self):
        '''初始化进程服务进程要求的操作'''
        self.name = 'ProcessServer'
        self.state = State.new

    def boot(self):
        print('booting the {}'.format(self))
        '''启动进程服务进程要求的操作'''
        self.state = State.running

    def kill(self， restart=True):
        print('Killing {}'.format(self))
        '''终止进程服务进程要求的操作'''
        self.state = State.restart if restart else State.zombie

    def create_process(self， user， name):
        '''检查用户权限和生成PID等'''
        print("trying to create the process '{}' for user '{}'".format(name， user))
```

OperatingSystem类是一个外观。__init__()中创建所有需要的服务进程实例。start()方法是系统的入口点，供客户端代码使用。如果需要，可以添加更多的包装方法作为服务的访问点，比如包装方法create_file()和create_process()。从客户端的角度来看，所有服务都是由OperatingSystem类提供的。客户端并不应该被不必要的细节所干扰，比如，服务进程的存在和每个服务进程的责任。


```python
class OperatingSystem:
    '''外观'''
    def __init__(self):
        self.fs = FileServer()
        self.ps = ProcessServer()
    def start(self):
        [i.boot() for i in (self.fs， self.ps)]
    def create_file(self， user， name， permissions):
        return self.fs.create_file(user， name， permissions)
    def create_process(self， user， name):
        return self.ps.create_process(user， name)
```

在下面的完整代码清单中(文件facade.py)，可以看到许多模拟的类和服务进程，它们的存在是为了让读者了解系统运转要求哪些抽象(User、Process和File等)和服务进程(WindowServer和NetworkServer等)。推荐至少实现系统的一个服务来练习一下(例如，文件创建)。可随意改变接口和方法签名来满足你的需求，但要确保在改变之后，客户端代码不需要知道OperatingSystem外观类之外的任何对象。


```python
from enum import Enum
from abc import ABCMeta， abstractmethod

State = Enum('State'， 'new running sleeping restart zombie')

class User:
    pass

class Process:
    pass

class File:
    pass

class Server(metaclass=ABCMeta):

    @abstractmethod
    def __init__(self):
        pass

    def __str__(self):
        return self.name

    @abstractmethod
    def boot(self):
        pass

    @abstractmethod
    def kill(self， restart=True):
        pass

class FileServer(Server):
    def __init__(self):
        '''初始化文件服务进程要求的操作'''
        self.name = 'FileServer'
        self.state = State.new

    def boot(self):
        print('booting the {}'.format(self))
        '''启动文件服务进程要求的操作'''
        self.state = State.running

    def kill(self， restart=True):
        print('Killing {}'.format(self))
        '''终止文件服务进程要求的操作'''
        self.state = State.restart if restart else State.zombie

    def create_file(self， user， name， permissions):
        '''检查访问权限的有效性、用户权限等'''
        print("trying to create the file '{}' for user '{}' with permissions{}".format(name， user， permissions))

class ProcessServer(Server):
    def __init__(self):
        '''初始化进程服务进程要求的操作'''
        self.name = 'ProcessServer'
        self.state = State.new

    def boot(self):
        print('booting the {}'.format(self))
        '''启动进程服务进程要求的操作'''
        self.state = State.running

    def kill(self， restart=True):
        print('Killing {}'.format(self))
        '''终止进程服务进程要求的操作'''
        self.state = State.restart if restart else State.zombie

    def create_process(self， user， name):
        '''检查用户权限和生成PID等'''
        print("trying to create the process '{}' for user '{}'".format(name， user))

class WindowsServer:
    pass

class NetworkServer:
    pass

class OperatingSystem:
    '''外观'''
    def __init__(self):
        self.fs = FileServer()
        self.ps = ProcessServer()

    def start(self):
        [i.boot() for i in (self.fs， self.ps)]

    def create_file(self， user， name， permissions):
        return self.fs.create_file(user， name， permissions)

    def create_process(self， user， name):
        return self.ps.create_process(user， name)

def main():
    os = OperatingSystem()
    os.start()
    os.create_file('foo'， 'hello'， '-rw-r-r')
    os.create_process('bar'， 'ls /tmp')

if __name__ == '__main__':
    main()
```

    booting the FileServer
    booting the ProcessServer
    trying to create the file 'hello' for user 'foo' with permissions-rw-r-r
    trying to create the process 'ls /tmp' for user 'bar'



```python
如上所示：执行这个例子会显示两个存根服务进程的启动信息。
```

外观类OperatingSystem起到了很好的作用。客户端代码可以创建文件和进程，而无需知道操作系统的内部细节，比如，多个服务进程的存在。准确点说是客户端可以调用方法来创建文件和进程，但是目前它们是模拟的。如果感兴趣，你可以实现这两个方法之一作为练习，或者两个都实现。

## 小结

本章中，我们学习了如何使用外观模式。在客户端代码想要使用一个复杂系统但又不关心系统复杂性之时，这种模式是为复杂系统提供一个简单接口的理想方式。一台计算机是一个外观，因为当我们使用它时需要做的事情仅是按一个按钮来启动它；其余的所有硬件复杂性都用户无感知地交由BIOS、引导加载程序以及其他系统软件来处理。现实生活中外观的例子更多，比如，我们所致电的银行或公司客服部门，还有启动机动车所使用的钥匙。

我们讨论了两个使用外观的Django第三方组件:django-oscar-datacash和Caliendo。前者使用外观模式来提供一个简单的DataCash API以及保存事务的能力，后者为多种目的使用了外观，比如，缓存、基于输入对象的类型决定应该返回什么。

我们讲解了外观基本的应用案例，并以多服务进程操作系统使用的接口实现来结束本章内容。外观是一种隐藏系统复杂性的优雅方式，因为多数情况下客户端代码并不应该关心系统的这些细节。
