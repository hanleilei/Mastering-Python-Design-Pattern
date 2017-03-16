# 模型－视图－控制器模式

关注点分离(Separation of Concerns,SoC)原则是软件工程相关的设计原则之一。SoC原则背后的思想是将一个应用切分成不同的部分，每个部分解决一个单独的关注点。分层设计中的层次(数据访问层、业务逻辑层和表示层等)即是关注点的例子。使用SoC原则能简化软件应用的开发和维护(请参考网页[t.cn/RqrjewK])。

模型—视图—控制器(Model-View-Controller，MVC)模式是应用到面向对象编程的Soc原则。模式的名称来自用来切分软件应用的三个主要部分，即模型部分、视图部分和控制器部分。MVC被认为是一种架构模式而不是一种设计模式。架构模式与设计模式之间的区别在于前者比后者的范畴更广。然而，MVC太重要了，不能仅因为这个原因就跳过不说。即使我们从不需要从头实现它，也需要熟悉它，因为所有常见框架都使用了MVC或者是其略微不同的版本(之后会详述)。

模型是核心的部分，代表着应用的信息本源，包含和管理(业务)逻辑、数据、状态以及应用的规则。视图是模型的可视化表现。视图的例子有，计算机图形用户界面、计算机终端的文本输出、智能手机的应用图形界面、PDF文档、饼图和柱状图等。视图只是展示数据，并不处理数据。控制器是模型与视图之间的链接/粘附。模型与视图之间的所有通信都通过控制器进行(请参考[GOF95,第14页]、网页[t.cn/RqrjF4G]和网页[t.cn/RPrOUPr])。

对于将初始屏幕渲染给用户之后使用MVC的应用，其典型使用方式如下所示：

* 用户通过单击(键入、触摸等)某个按钮触发一个视图
* 视图把用户操作告知控制器
* 控制器处理用户输入，并与模型交互
* 模型执行所有必要的校验和状态改变,并通知控制器应该做什么
* 控制器按照模型给出的指令，指导视图适当地更新和显示输出

与3-tier相比，MVC 是一种设计模式，包括模型(Model),视图(View)，控制Controller)：

1. Model（模型）是应用程序中用于处理应用程序数据逻辑的部分。通常模型对象负责在数据库中存取数据。

2. View（视图）是应用程序中处理数据显示的部分。通常视图是通过控制器操作模型数据创建的。

3. Controller（控制器）作用于模型和视图之间，处理业务逻辑层的操作。通常控制器负责从视图读取数据，控制用户输入，并向模型发送数据。

```python
## 如下所示为来自于github的示例代码：

#!/usr/bin/env python
# -*- coding: utf-8 -*-

class Model(object):
    def __iter__(self):
        raise NotImplementedError

    def get(self, item):
        """Returns an object with a .items() call method
        that iterates over key,value pairs of its information."""
        raise NotImplementedError

    @property
    def item_type(self):
        raise NotImplementedError


class ProductModel(Model):

    class Price(float):
        """A polymorphic way to pass a float with a particular __str__ functionality."""
        def __str__(self):
            first_digits_str = str(round(self,2))
            try:
                dot_location = first_digits_str.index('.')
            except ValueError:
                return (first_digits_str + '.00')
            else:
                return (first_digits_str +
                               '0'*(3 + dot_location - len(first_digits_str)))

    products = {
        'milk': {'price': Price(1.50), 'quantity': 10},
        'eggs': {'price': Price(0.20), 'quantity': 100},
        'cheese': {'price': Price(2.00), 'quantity': 10}
    }

    item_type = 'product'

    def __iter__(self):
        for item in self.products:
            yield item

    def get(self, product):
        try:
            return self.products[product]
        except KeyError as e:
            raise KeyError((str(e) + " not in the model's item list."))

class View(object):
    def show_item_list(self, item_type, item_list):
        raise NotImplementedError

    def show_item_information(self, item_type, item_name, item_info):
        """Will look for item information by iterating over key,value pairs
        yielded by item_info.items()"""
        raise NotImplementedError

    def item_not_found(self, item_type, item_name):
        raise NotImplementedError

class ConsoleView(View):

    def show_item_list(self, item_type, item_list):
        print(item_type.upper() + ' LIST:')
        for item in item_list:
            print(item)
        print('')

    @staticmethod
    def capitalizer(string):
        return string[0].upper() + string[1:].lower()

    def show_item_information(self, item_type, item_name, item_info):
        print(item_type.upper() + ' INFORMATION:')
        printout = 'Name: %s' % item_name
        for key, value in item_info.items():
            printout += (', ' + self.capitalizer(str(key)) + ': ' + str(value))
        printout += '\n'
        print(printout)

    def item_not_found(self, item_type, item_name):
        print('That %s "%s" does not exist in the records' % (item_type, item_name))


class Controller(object):

    def __init__(self, model, view):
        self.model = model
        self.view = view

    def show_items(self):
        items = list(self.model)
        item_type = self.model.item_type
        self.view.show_item_list(item_type, items)

    def show_item_information(self, item_name):
        try:
            item_info = self.model.get(item_name)
        except:
            item_type = self.model.item_type
            self.view.item_not_found(item_type, item_name)
        else:
            item_type = self.model.item_type
            self.view.show_item_information(item_type, item_name, item_info)


if __name__ == '__main__':
    model = ProductModel()
    view = ConsoleView()
    controller = Controller(model, view)
    controller.show_items()
    controller.show_item_information('cheese')
    controller.show_item_information('eggs')
    controller.show_item_information('milk')
    controller.show_item_information('arepas')

### OUTPUT ###
# PRODUCT LIST:
# cheese
# eggs
# milk
#
# PRODUCT INFORMATION:
# Name: Cheese, Price: 2.00, Quantity: 10
#
# PRODUCT INFORMATION:
# Name: Eggs, Price: 0.20, Quantity: 100
#
# PRODUCT INFORMATION:
# Name: Milk, Price: 1.50, Quantity: 10
#
# That product "arepas" does not exist in the records
```

    PRODUCT LIST:
    cheese
    milk
    eggs

    PRODUCT INFORMATION:
    Name: cheese, Quantity: 10, Price: 2.00

    PRODUCT INFORMATION:
    Name: eggs, Quantity: 100, Price: 0.20

    PRODUCT INFORMATION:
    Name: milk, Quantity: 10, Price: 1.50

    That product "arepas" does not exist in the records


你可能想知道为什么控制器部分是必要的?我们能跳过它吗？能，但那样我们将失去MVC提供的一大优势：无需修改模型就能使用多个视图的能力(甚至可以根据需要同时使用多个视图)。为了实现模型与其表现之间的解耦，每个视图通常都需要属于它的控制器。如果模型直接与特定视图通信，我们将无法对同一个模型使用多个视图(或者至少无法以简洁模块化的方式实现)。

## 现实生活的例子

MVC是应用于面向对象编程的SoC原则。SoC原则在现实生活中的应用有很多。例如，如果你造一栋新房子，通常会请不同的专业人员来完成以下工作。

* 安装管道和电路
* 粉刷房子

另一个例子是餐馆。在一个餐馆中，服务员接收点菜单并为顾客上菜，但是饭菜由厨师烹饪(请参考网页[t.cn/RqrYh1I])。

## 软件的例子

Web框架web2py(请参考网页[t.cn/RqrYZwy])是一个支持MVC模式的轻量级Python框架。若你还未尝试过web2py，我推荐你试用一下，安装过程极其简单，你要做的就是下载安装包并执行一个Python文件(web2py.py)。在该项目的网页上有很多例子演示了在web2py中如何使用MVC(请参考网页[t.cn/RqrYADU])。

Django也是一个MVC框架，但是它使用了不同的命名约定。在此约定下，控制器被称为视图，视图被称为模板。Django使用名称模型—模板—视图(Model-Template-View,MTV)来替代MVC。依据Django的设计者所言，视图是描述哪些数据对用户可见。因此，Django把对应一个特定URL的Python回调函数称为视图。Django中的“模板”用于把内容与其展现分开，其描述的是用户看到数据的方式而不是哪些数据可见(请参考网页[t.cn/RwRJZ87])。

## 应用案例

MVC是一个非常通用且大有用处的设计模式。实际上,所有流行的Web框架(Django、Rails 和Yii)和应用框架(iPhone SDK、Android和QT)都使用了MVC或者其变种，其变种包括模式—视图—适配器(Model-View-Adapter,MVA)、模型—视图—演示者(Model-View-Presenter,MVP) 等。然而，即使我们不使用这些框架，凭自己实现这一模式也是有意义的，因为这一模式提供了以下这些好处。

* 视图与模型的分离允许美工一心搞UI部分，程序员一心搞开发，不会相互干扰。
* 由于视图与模型之间的松耦合,每个部分可以单独修改/扩展，不会相互影响。例如，添加一个新视图的成本很小，只要为其实现一个控制器就可以了。
* 因为职责明晰，维护每个部分也更简单。

在从头开始实现MVC时，请确保创建的模型很智能，控制器很瘦，视图很傻瓜(请参考[Zlobin13,第9页])。

可以将具有以下功能的模型视为智能模型。

* 包含所有的校验/业务规则/逻辑
* 处理应用的状态
* 访问应用数据(数据库、云或其他)
* 不依赖UI

可以将符合以下条件的控制器视为瘦控制器。

* 在用户与视图交互时,更新模型
* 在模型改变时，更新视图
* 如果需要，在数据传递给模型/视图之前进行处理不展示数据
* 不直接访问应用数据
* 不包含校验/业务规则/逻辑

可以将符合以下条件的视图视为傻瓜视图。

* 展示数据
* 允许用户与其交互
* 仅做最小的数据处理，通常由一种模板语言提供处理能力(例如，使用简单的变量和循环控制)
* 不存储任何数据
* 不直接访问应用数据
* 不包含校验/业务规则/逻辑

如果你正在从头实现MVC，并且想弄清自己做得对不对，可以尝试回答以下两个关键问题。

* 如果你的应用有GUI，那它可以换肤吗？易于改变它的皮肤/外观以及给人的感受吗？可以为用户提供运行期间改变应用皮肤的能力吗？如果这做起来并不简单，那就意味着你的MVC实现在某些地方存在问题(请参考网页[t.cn/RqrjF4G])。
* 如果你的应用没有GUI(例如，是一个终端应用),为其添加GUI支持有多难？或者，如果添加GUI没什么用，那么是否易于添加视图从而以图表(饼图、柱状图等)或文档(PDF、电子表格等)形式展示结果？如果因此而作出的变更不小(小的变更是，在不变更模型的情况下创建控制器并绑定到视图)，那你的MVC实现就有些不对了。

如果你确信这两个条件都已满足，那么与未使用MVC模式的应用相比，你的应用会更灵活、更好维护。

## 实现

我可以使用任意常见框架来演示如何使用MVC，但觉得那样的话，读者对MVC的理解会不完整。因此我决定使用一个非常简单的示例来展示如何从头实现MVC，这个示例是名人名言打印机。想法极其简单：用户输入一个数字，然后就能看到与这个数字相关的名人名言。名人名言存储在一个quotes元组中。这种数据通常是存储在数据库、文件或其他地方，只有模型能够直接访问它。

我们从下面的代码开始考虑这个例子。


```python
quotes = ('A man is not complete until he is married. Then he is finished.',
              'As I said before, I never repeat myself.',
              'Behind a successful man is an exhausted woman.',
              'Black holes really suck...', 'Facts are stubborn things.')
```

模型极为简约，只有一个get_quote()方法，基于索引n从quotes元组中返回对应的名人名言（字符串）。注意，n可以小于等于0，因为这种索引方式在Python中是有效的。本节末尾准备了练习，供你改进这个方法的行为。


```python
class QuoteModel:
    def get_quote(self, n):
        try:
            value = quotes[n]
        except IndexError as err:
            value = 'Not found!'
        return value
```

视图有三个方法，分别是show()、error()和select_quote()。show()用于在屏幕上输出一旬名人名言（或者输出提示信息Not found!）；error()用于在屏幕上输出一条错误消息；select_quote()用于读取用户的选择，如以下代码所示。


```python
class QuoteTerminalView:
    def show(self, quote):
        print('And the quote is: "{}"'.format(quote))
    def error(self, msg):
        print('Error: {}'.format(msg))
    def select_quote(self):
        return input('Which quote number would you like to see? ')
```

控制器负责协调。init()方法初始化模型和视图。run()方法校验用户提供的名言索引，然后从模型中获取名言，并返回给视图展示，如以下代码所示。


```python
class QuoteTerminalController:
    def init (self):
        self.model = QuoteModel()
        self.view = QuoteTerminalView()

    def run(self):
        valid_input = False
        while not valid_input:
            n = self.view.select_quote()
            try:
                n = int(n)
            except ValueError as err:
                self.view.error("Incorrect index '{}'".format(n)) else:
            valid_input = True
        quote = self.model.get_quote(n)
        self.view.show(quote)
```

最后，但同样重要的是，main()函数初始化并触发控制器，如以下代码所示。

```python
def main():
    controller = QuoteTerminalController()
    while True:
        controller.run()
```


以下是该示例的完整代码（文件mvc.py）。


```python
quotes = ('A man is not complete until he is married. Then he is finished.',
          'As I said before, I never repeat myself.',
          'Behind a successful man is an exhausted woman.',
          'Black holes really suck...',
          'Facts are stubborn things.')

class QuoteModel:
    def get_quote(self, n):
        try:
            value = quotes[n]
        except IndexError as err:
            value = 'Not found!'
        return value

class QuoteTerminalView:
    def show(self, quote):
        print('And the quote is: "{}"'.format(quote))

    def error(self, msg):
        print('Error: {}'.format(msg))

    def select_quote(self):
        return input('Which quote number would you like to see? ')

class QuoteTerminalController:
    def __init__(self):
        self.model = QuoteModel()
        self.view = QuoteTerminalView()

    def run(self):
        valid_input = False
        while not valid_input:
            n = self.view.select_quote()
            try:
                n = int(n)
            except ValueError as err:
                self.view.error("Incorrect index '{}'".format(n))
            else:
                valid_input = True
        quote = self.model.get_quote(n)
        self.view.show(quote)

def main():
    controller = QuoteTerminalController()
    while True:
        controller.run()

if __name__ == '__main__':
    main()

```


    ---------------------------------------------------------------------------
```python
    KeyboardInterrupt                         Traceback (most recent call last)

    /Users/hanlei/anaconda/lib/python3.5/site-packages/ipykernel/kernelbase.py in _input_request(self, prompt, ident, parent, password)
        713             try:
    --> 714                 ident, reply = self.session.recv(self.stdin_socket, 0)
        715             except Exception:


    /Users/hanlei/anaconda/lib/python3.5/site-packages/jupyter_client/session.py in recv(self, socket, mode, content, copy)
        738         try:
    --> 739             msg_list = socket.recv_multipart(mode, copy=copy)
        740         except zmq.ZMQError as e:

    /Users/hanlei/anaconda/lib/python3.5/site-packages/zmq/sugar/socket.py in recv_multipart(self, flags, copy, track)

    --> 358         parts = [self.recv(flags, copy=copy, track=track)]
        359         # have first part already, only loop while more to receive


    zmq/backend/cython/socket.pyx in zmq.backend.cython.socket.Socket.recv (zmq/backend/cython/socket.c:6971)()


    zmq/backend/cython/socket.pyx in zmq.backend.cython.socket.Socket.recv (zmq/backend/cython/socket.c:6763)()


    zmq/backend/cython/socket.pyx in zmq.backend.cython.socket._recv_copy (zmq/backend/cython/socket.c:1931)()


    /Users/hanlei/anaconda/lib/python3.5/site-packages/zmq/backend/cython/checkrc.pxd in zmq.backend.cython.checkrc._check_rc (zmq/backend/cython/socket.c:7222)()


    KeyboardInterrupt:


    During handling of the above exception, another exception occurred:


    KeyboardInterrupt                         Traceback (most recent call last)

    <ipython-input-11-ea7696f29195> in <module>()
         47
         48 if __name__ == '__main__':
    ---> 49     main()


    <ipython-input-11-ea7696f29195> in main()
         44     controller = QuoteTerminalController()
         45     while True:
    ---> 46         controller.run()
         47
         48 if __name__ == '__main__':


    <ipython-input-11-ea7696f29195> in run(self)
         31         valid_input = False
         32         while not valid_input:
    ---> 33             n = self.view.select_quote()
         34             try:
         35                 n = int(n)


    <ipython-input-11-ea7696f29195> in select_quote(self)
         21
         22     def select_quote(self):
    ---> 23         return input('Which quote number would you like to see? ')
         24
         25 class QuoteTerminalController:


    /Users/hanlei/anaconda/lib/python3.5/site-packages/ipykernel/kernelbase.py in raw_input(self, prompt)
        687             self._parent_ident,
        688             self._parent_header,
    --> 689             password=False,
        690         )
        691


    /Users/hanlei/anaconda/lib/python3.5/site-packages/ipykernel/kernelbase.py in _input_request(self, prompt, ident, parent, password)
        717             except KeyboardInterrupt:
        718                 # re-raise KeyboardInterrupt, to truncate traceback
    --> 719                 raise KeyboardInterrupt
        720             else:
        721                 break


    KeyboardInterrupt:
```

当然，你不会（也不应该）就此止步。坚持多写代码，还有很多有意思的想法可以试验，比如以下这些。

* 仅允许用户使用大于或等于1的索引，程序会显得更加友好。为此，你也需要修改get_quote()。
* 使用Tkinter、Pygame或Kivy之类的GUI框架来添加一个图形化视图。程序如何模块化？可以在程序运行期间决定使用哪个视图吗？
* 让用户可以选择键入某个键（例如，r键）随机地看一旬名言。
* 索引校验H前是在控制器中完成的。这个方式好吗？如果你编写了另一个视图，需要它自己的控制器，那又该怎么办呢？试想一下，为了让索引校验的代码被所有控制/视图复用，将索引校验移到模型中进行，需要做哪些变更？
* 对这个例子进行扩展，使其变得像一个创建、读取、更新、删除（Create, Read, Update, Delete，CURD）应用。你应该能够输入新的名言，删除已有的名言，以及修改名言。

## 小结

本章中，我们学习了MVC模式。MVC是一个非常重要的设计模式，用于将应用组织成三个部分：模型、视图和控制器。

每个部分都有明确的职责。模型负责访问数据，管理应用的状态。视图是模型的外在表现。视图并非必须是图形化的；文本输出也是一种好视图。控制器是模型与视图之间的连接。MVC的恰当使用能确保最终产出的应用易于维护、易于扩展。

MVC模式是应用到面向对象编程的SoC原则。这一原则类似于一栋新房子如何建造，或一个餐馆如何运营。

Python框架web2py使用MVC作为核心架构理念。即使是最简单的web2py例子也使用了MVC来实现模块化和可维护性。Django也是一个MVC框架，但它使用的名称是MTV。

使用MVC时，请确保创建智能的模型（核心功能）、瘦控制器（实现视图与模型之间通信的能力）以及傻瓜式的视图（外在表现，最小化逻辑处理）。

在8.4节中，我们学习了如何从零开始实现MVC，为用户展示有趣的名人名言。这与罗列一个RSS源的所有文章所要求的功能没什么两样，如果你对其他推荐练习不感兴趣，可以练习实现这个。
