# 解释器模式

对每个应用来说，至少有以下两种不同的用户分类。
* 基本用户：这类用户只希望能够凭直觉使用应用。他们不喜欢花太多时间配置或学习应用的内部。对他们来说，基本的用法就足够了。
* 高级用户：这些用户，实际上通常是少数，不介意花费额外的时间学习如何使用应用的高级特性。如果知道学会之后能得到以下好处，他们甚至会去学习一种配置（或脚本）语言。
    · 能够更好地控制一个应用
    · 以更好的方式表达想法
    · 提高生产力

解释器（Interpreter）模式仅能引起应用的高级用户的兴趣。这是因为解释器模式背后的主要思想是让非初级用户和领域专家使用一门简单的语言来表达想法。然而，什么是一种简单的语言？对于我们的需求来说，一种简单的语言就是没有像编程语言那么复杂的语言。

一般而言，我们想要创建的是一种领域特定语言（Domain Specific Language，DSL）。DSL是一种针对一个特定领域的有限表达能力的计算机语言。很多不同的事情都使用DSL，比如，战斗模拟、记账、可视化、配置、通信协议等。DSL分为内部DSL和外部DSL（请参考网页[t.cn/zHtEh5t]和网页[t.cn/hBfQ2Y]）。

内部DSL构建在一种宿主编程语言之上。内部DSL的一个例子是，使用Python解决线性方程组的一种语言。使用内部DSL的优势是我们不必担心创建、编译及解析语法，因为这些已经被宿主语言解决掉了。劣势是会受限于宿主语言的特性。如果宿主语言不具备这些特性，构建一种表达能力强、简沽而且优美的内部DSL是富有挑战性的（请参考网页[t.cn/Rqr3B12]）。

外部DSL不依赖某种宿主语言。DSL的创建者可以决定语言的方方面面（语法、旬法等），但也要负责为其创建一个解析器和编译器。为一种新语言创建解析器和编译器是一个非常复杂、长期而又痛苦的过程（请参考网页[t.cn/Rqr3B12]）。

解释器模式仅与内部DSL相关。因此，我们的H标是使用宿主语言提供的特性构建一种简单但有用的语言，在这里，宿主语言是Python。注意，解释器根本不处理语言解析，它假设我们已经有某种便利形式的解析好的数据，可以是抽象语法树（abstract syntax tree，AST）或任何其他好用的数据结构（请参考[GOF95，第276页]）。

## 现实生活的例子

音乐演奏者是现实中解释器模式的一个例子。五线谱图形化地表现了声音的音调和持续时间。音乐演奏者能根据五线谱的符号精确地重现声音。在某种意义上，五线谱是音乐的语言，音乐演奏者是这种语言的解释器。下图展示了音乐例子的图形化描述，经www.sourcemaking.com准许使用（请参考网页[t.cn/Rqr3Fqs]）。

## 软件的例子
内部DSL在软件方面的例子有很多。PyT是一个用于生成(X)HTML的Python DSL。PyT关注性能，并声称能与Jinja2的速度相娘美（请参考网页[t.cn/Rqr1vlP]）。当然，我们不能假定在PyT中必须使用解释器模式。然而，PyT是一种内部DSL，非常适合使用解释器模式。

Chromium是一个自由开源的浏览器软件，催生出了Google Chrome浏览器（请参考网页[t.cn/hMjLK]）。Chromium的Mesa库Python绑定的一部分使用解释器模式将C样板参数翻译成Python对象并执行相关的命令（请参考网页[t.cn/Rqr1zZP]）。

## 应用案例

在我们希望为领域专家和高级用户提供一种简单语言来解决他们的问题时，可以使用解释器模式。不过要强调的第一件事情是，解释器模式应仅用于实现简单的语言。如果语言具有外部DSL那样的要求，有更好的工具（yacc和lex、Bison、ANTLR等）来从头创建一种语言。

我们的目标是为专家提供恰当的编程抽象，使其生产力更高；这些专家通常不是程序员。理想情况下，他们使用我们的DSL并不需要了解高级Python知识，当然了解一点Python基础知识会更好，因为我们最终生成的是Python代码，但不应该要求了解Python高级概念。此外，DSL的性能通常不是一个重要的关注点。重点是提供一种语言，隐藏宿主语言的独特性，并提供人类更易读的语法。诚然，Python已经是一门可读性非常高的语言，与其他编程语言相比，其古怪的语法更少。

## 实现

我们来创建一种内部DSL控制一个智能屋。这个例子非常契合如今越来越受关注的物联网时代。用户能够使用一种非常简单的事件标记来控制他们的房子。一个事件的形式为command->receiver->arguments。参数部分是可选的。并不是所有事件都要求参数。不要求任何参数的事件例子如下所示。

open->gate

要求参数的事件例子如下所示：

increase -> boiler temperature -> 3 degrees

->符号用于标记事件一个部分的结束，并声明下一个部分的开始。实现一种内部DSL有多种方式。我们可以使用普通的正则表达式、字符串处理、操作符重载的组合以及元编程，或者一个能帮我们完成困难工作的库/工具。虽然在正规情况下，解释器不处理解析，但我觉得一个实战的例子也需要覆盖解析工作。因此，我决定使用一个工具来完成解析工作。该工具名为Pyparsing，是标准Python3发行版的一部分心。要想获取更多Pyparsing的信息，可参考Paul McGuire编写的迷你书Getting Started with Pyparsing。如果你的系统上还没安装Pyparsing，可以使用下面的命令来安装。

>>> pip3 install pyparsing

下面的时序图展示了用户执行开门事件时发生的事情。对于其他事件，情况也是相似的，不过有些命令更复杂些，因为它们要求参数。

在编写代码之前，为我们的语言定义一种简单语法是一个好做法。我们可以使用巴科斯-诺尔形式（Backus-Naur Form，BNF）表示法来定义语法（请参考网页[t.cn/Rqr1ZrO]）。

event ::= command token receiver token arguments command ::= word+
word ::= a collection of one or more alphanumeric characters token ::= ->
receiver ::= word+ arguments ::= word+

简单来说，这个语法告诉我们的是一个事件具有command -> receiver -> arguments 的形式，并且命令、接收者及参数也具有相同的形式，即一个或多个字母数字字符的组合。包含数字部分是为了让我们能够在命令increase -> boiler temperature -> 3 degrees中传递3 degrees这样的参数，所以不必怀疑数字部分的必要性。

既然定义了语法，那么接着将其转变成实际的代码。以下是代码的样子。


```python
word = Word(alphanums)
command = Group(OneOrMore(word)) token = Suppress("->")
device = Group(OneOrMore(word)) argument = Group(OneOrMore(word))
event = command + token + device + Optional(token + argument)
```

代码和语法定义基本的不同点是，代码需要以自底向上的方式编写。例如，如果不先为word赋一个值，那就不能使用它。Suppress用于声明我们希望解析结果中省略->符号。

这个例子的完整代码（文件interpreter.py）使用了很多占位类，但为了让你精力集中一点，我会先只展示一个类。书中也包含完整的代码列表，在仔细解说完这个类之后会展示。我们来看一下Boiler类。一个锅炉的默认温度为83摄氏度。类有两个方法来分别提高和降低当前的温度。


```python
class Boiler:
    def __init__(self):
        self.temperature = 83 #

    def __str__(self):
        return 'boiler temperature: {}'.format(self.temperature)

    def increase_temperature(self, amount):
        print("increasing the boiler's temperature by {} degrees".format(amount))
        self.temperature += amount

    def decrease_temperature(self, amount):
        print("decreasing the boiler's temperature by {} degrees".format(amount))
        self.temperature -= amount
```

下一步是添加语法，之前已学习过。我们也创建一个boiler实例，并输出其默认状态。


```python
word = Word(alphanums)
command = Group(OneOrMore(word))
token = Suppress("->")
device = Group(OneOrMore(word))
argument = Group(OneOrMore(word))
event = command + token + device + Optional(token + argument)

boiler = Boiler()
print(boiler)
```

获取Pyparsing解析结果的最简单方式是使用parseString()方法，该方法返回的结果是一个ParseResults实例，它实际上是一个可视为嵌套列表的解析树。例如，执行print(event. parseString('increase -> boiler temperature -> 3 degrees'))得到的结果如下所示。


```python
[['increase'], ['boiler', 'temperature'], ['3', 'degrees']]
```

因此，在这里，我们知道第一个子列表是命令（提高），第二个子列表是接收者（锅炉温度），第三个子列表是参数（3摄氏度）。实际上我们可以解开ParseResults实例，从而可以直接访问事件的这三个部分。可直接访问意味着我们可以匹配模式找到应该执行哪个方法(即使不可以直接访问，也只能这样做)。


```python
cmd, dev, arg = event.parseString('increase -> boiler temperature -> 3 degrees')
if 'increase' in ' '.join(cmd):
    if 'boiler' in ' '.join(dev):
        boiler.increase_temperature(int(arg[0]))
        print(boiler)
```

执行上面的代码片段会得到以下输出。

>>> python3 boiler.py
boiler temperature: 83
increasing the boiler's temperature by 3 degrees
boiler temperature: 86

interpreter.py的完整代码与我刚描述的没有什么大的不同，只是进行了扩展以支持更多的事件和设备。


```python
from pyparsing import Word, OneOrMore, Optional, Group, Suppress, alphanums

class Gate:
    def __init__(self):
        self.is_open = False

    def __str__(self):
        return 'open' if self.is_open else 'closed'

    def open(self):
        print('opening the gate')
        self.is_open = True

    def close(self):
        print('closing the gate')
        self.is_open = False

class Garage:
    def __init__(self):
        self.is_open = False

    def __str__(self):
        return 'open' if self.is_open else 'closed'

    def open(self):
        print('opening the garage')
        self.is_open = True

    def close(self):
        print('closing the garage')
        self.is_open = False

class Aircondition:
    def __init__(self):
        self.is_on = False

    def __str__(self):
        return 'on' if self.is_on else 'off'

    def turn_on(self):
        print('turning on the aircondition')
        self.is_on = True

    def turn_off(self):
        print('turning off the aircondition')
        self.is_on = False

class Heating:
    def __init__(self):
        self.is_on = False

    def __str__(self):
        return 'on' if self.is_on else 'off'

    def turn_on(self):
        print('turning on the heating')
        self.is_on = True

    def turn_off(self):
        print('turning off the heating')
        self.is_on = False

class Boiler:
    def __init__(self):
        self.temperature = 83 # in celsius

    def __str__(self):
        return 'boiler temperature: {}'.format(self.temperature)

    def increase_temperature(self, amount):
        print("increasing the boiler's temperature by {} degrees".format(amount))
        self.temperature += amount

    def decrease_temperature(self, amount):
        print("decreasing the boiler's temperature by {} degrees".format(amount))
        self.temperature -= amount

class Fridge:
    def __init__(self):
        self.temperature = 2 # in celsius

    def __str__(self):
        return 'fridge temperature: {}'.format(self.temperature)

    def increase_temperature(self, amount):
        print("increasing the fridge's temperature by {} degrees".format(amount))
        self.temperature += amount

    def decrease_temperature(self, amount):
        print("decreasing the fridge's temperature by {} degrees".format(amount))
        self.temperature -= amount


def main():
    word = Word(alphanums)
    command = Group(OneOrMore(word))
    token = Suppress("->")
    device = Group(OneOrMore(word))
    argument = Group(OneOrMore(word))
    event = command + token + device + Optional(token + argument)

    gate = Gate()
    garage = Garage()
    airco = Aircondition()
    heating = Heating()
    boiler = Boiler()
    fridge = Fridge()

    tests = ('open -> gate',
             'close -> garage',
             'turn on -> aircondition',
             'turn off -> heating',
             'increase -> boiler temperature -> 5 degrees',
             'decrease -> fridge temperature -> 2 degrees')

    open_actions = {'gate':gate.open, 'garage':garage.open, 'aircondition':airco.turn_on,
                  'heating':heating.turn_on, 'boiler temperature':boiler.increase_temperature,
                  'fridge temperature':fridge.increase_temperature}
    close_actions = {'gate':gate.close, 'garage':garage.close, 'aircondition':airco.turn_off,
                   'heating':heating.turn_off, 'boiler temperature':boiler.decrease_temperature,
                   'fridge temperature':fridge.decrease_temperature}

    for t in tests:
        if len(event.parseString(t)) == 2: # no argument
            cmd, dev = event.parseString(t)
            cmd_str, dev_str = ' '.join(cmd), ' '.join(dev)
            if 'open' in cmd_str or 'turn on' in cmd_str:
                open_actions[dev_str]()
            elif 'close' in cmd_str or 'turn off' in cmd_str:
                close_actions[dev_str]()
        elif len(event.parseString(t)) == 3: # argument
            cmd, dev, arg = event.parseString(t)
            cmd_str, dev_str, arg_str = ' '.join(cmd), ' '.join(dev), ' '.join(arg)
            num_arg = 0
            try:
                num_arg = int(arg_str.split()[0]) # extract the numeric part
            except ValueError as err:
                print("expected number but got: '{}'".format(arg_str[0]))
            if 'increase' in cmd_str and num_arg > 0:
                open_actions[dev_str](num_arg)
            elif 'decrease' in cmd_str and num_arg > 0:
                close_actions[dev_str](num_arg)

if __name__ == '__main__':
    main()
```

    opening the gate
    closing the garage
    turning on the aircondition
    turning off the heating
    increasing the boiler's temperature by 5 degrees
    decreasing the fridge's temperature by 2 degrees




执行上面的例子会得到以下输出。

>>> python3 interpreter.py opening the gate
    closing the garage
    turning on the aircondition
    turning off the heating
    increasing the boiler's temperature by 5 degrees
    decreasing the fridge's temperature by 2 degrees



如果你想针对这个例子进行更多的实验，我可以给你提一些建议。第一个会让例子更有意思的改变是让其变成交互式。目前，所有事件都是在tests元组中硬编码的。然而，用户希望能使用一个交互式提示符来激活命令。不要忘了Pyparsing对空格、Tab或意料之外的输出都是敏感的。例如，如果用户输入turn	off	-> heating 37，那会发生什么呢？

另一个可能的改进是，注意open_actions和close_actions映射是如何用于将一个接收者关联到一个方法的。使用一个映射而不是两个，可能吗？这样做有何优势？

## 小结
本章中，我们学习了解释器设计模式。解释器模式用于为高级用户和领域专家提供一个类编程的框架，但没有暴露出编程语言那样的复杂性。这是通过实现一个DSL来达到H的的。

DSL是一种针对特定领域、表达能力有限的计算机语言。DSL有两类，分别是内部DSL和外部DSL。内部DSL构建在一种宿主编程语言之上，依赖宿主编程语言，外部DSL则是从头实现，不依赖某种已有的编程语言。解释器模式仅与内部DSL相关。

乐谱是一个非软件DSL的例子。音乐演奏者像一个解释器那样，使用乐谱演奏出音乐。从软件的视角来看，许多Python模板引擎都使用了内部DSL。PyT是一个高性能的生成(X)HTML的 Python DSL。我们也看到Chromium的Mesa库是如何使用解释器模式将图形相关的C代码翻译成Python可执行对象的。

虽然一般来说解释器模式不处理解析的工作，但是在12.4节，我们使用了Pyparsing创建一种DSL来控制一个智能屋，并且看到使用一个好的解析工具以模式匹配来解释结果更加简单。

第13章将演示观察者模式。观察者模式用于在两个或多个对象之间创建一个发布—订阅通信类型。
