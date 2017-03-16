# 状态模式

面向对象编程着力于在对象交互时改变它们的状态。在很多问题中，有限状态机（通常名为状态机）是一个非常方便的状态转换建模（并在必要时以数学方式形式化）工具。首先，什么是状态机？状态机是一个抽象机器，有两个关键部分，状态和转换。状态是指系统的当前（激活）状况。例如，假设我们有一个收音机，其两个可能的状态是在调频波段（FM）或调幅波段（AM）上调节。另一个可能的状态是从一个FM/AM尤线电台切换到另一个。转换是指从一个状态切换到另一个状态，因某个事件或条件的触发而开始。通常，在一次转换发生之前或之后会执行一个或一组动作。假设我们的收音机被调到107 FM尤线电台，一次状态转换的例子是收听人按下按钮切换到107.5 FM。

状态机的一个不错的特性是可以用图来表现（称为状态图），其中每个状态都是一个节点，每个转换都是两个节点之间的边。下图展示了一个典型操作系统进程的状态图（不是针对特定的系统），经Wikipedia允许使用（请参考网页[t.cn/Rqr1CDd]）。进程一开始由用户创建好，就进入“已创建/新建”状态。这个状态只能切换到“等待”状态，这个状态转换发生在调度器将进程加载进内存并添加到“等待/预备执行”的进程队列之时。一个“等待”进程有两个可能的状态转换：可被选择而执行（切换到“运行”状态），或被更高优先级的进程所替代（切换到“换出并等待”状态）。

进程的其他典型状态还包括“终止”（已完成或已终止）、“阻塞”（例如，等待一个I/O操作完成）等。需要注意，一个状态机在一个特定时间点只能有一个激活状态。例如，一个进程不可能同时处于“已创建”状态和“运行”状态。

状态机可用于解决多种不同的问题，包括非计算机的问题。非计算机的例子包括自动售货机、电梯、交通灯、暗码锁、停车计时器、自动加油泵及自然语言文法描述。计算机方面的例子包括游戏编程和计算机编程的其他领域、硬件设计、协议设计，以及编程语言解析（请参考网页[t.cn/RUFNdYt]和网页[t.cn/zY5jPeH]）。

好了，听起来很美好。但是状态机如何关联到状态设计模式（State design pattern）呢？其实状态模式就是应用到一个特定软件工程问题的状态机（请参考[GOF95，第342页]和[Eckel08，第151页]）。

以下示例来自于github：

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
"""Implementation of the state pattern"""

# http://ginstrom.com/scribbles/2007/10/08/design-patterns-python-style/
from __future__ import print_function


class State(object):

    """Base state. This is to share functionality"""

    def scan(self):
        """Scan the dial to the next station"""
        self.pos += 1
        if self.pos == len(self.stations):
            self.pos = 0
        print(u"Scanning... Station is %s %s" %
              (self.stations[self.pos], self.name))


class AmState(State):

    def __init__(self, radio):
        self.radio = radio
        self.stations = ["1250", "1380", "1510"]
        self.pos = 0
        self.name = "AM"

    def toggle_amfm(self):
        print(u"Switching to FM")
        self.radio.state = self.radio.fmstate


class FmState(State):

    def __init__(self, radio):
        self.radio = radio
        self.stations = ["81.3", "89.1", "103.9"]
        self.pos = 0
        self.name = "FM"

    def toggle_amfm(self):
        print(u"Switching to AM")
        self.radio.state = self.radio.amstate


class Radio(object):

    """A radio.     It has a scan button, and an AM/FM toggle switch."""

    def __init__(self):
        """We have an AM state and an FM state"""
        self.amstate = AmState(self)
        self.fmstate = FmState(self)
        self.state = self.amstate

    def toggle_amfm(self):
        self.state.toggle_amfm()

    def scan(self):
        self.state.scan()


# Test our radio out
if __name__ == '__main__':
    radio = Radio()
    actions = [radio.scan] * 2 + [radio.toggle_amfm] + [radio.scan] * 2
    actions *= 2

    for action in actions:
        action()

### OUTPUT ###
# Scanning... Station is 1380 AM
# Scanning... Station is 1510 AM
# Switching to FM
# Scanning... Station is 89.1 FM
# Scanning... Station is 103.9 FM
# Scanning... Station is 81.3 FM
# Scanning... Station is 89.1 FM
# Switching to AM
# Scanning... Station is 1250 AM
# Scanning... Station is 1380 AM
```

## 现实生活的例子
这里再一次提到零食自动售货机（在之前的第10章中见过），它也是正常生活中状态模式的一个例子。自动售货机有不同的状态，并根据我们放入的钱币数量作出不同反应。根据我们的选择和放入的钱币，机器会执行以下操作。

* 拒绝我们的选择，因为请求的货物已售罄。
* 拒绝我们的选择，因为放入的钱币不足。
* 递送货物，且不找零，因为放入的钱币恰好足够。
* 递送货物，并找零。

当然还有更多可能的状态，但你明臼要点就好。下图由www.sourcemaking.com提供（请参考网页[t.cn/RqBS3o0]），展示了使用继承实现售货机不同状态的一种可能方案。

## 软件的例子
使用状态模式本质上相当于实现一个状态机来解决特定领域的一个软件问题。django-fsm程序包是一个第三方程序包，用于 Django 框架中简化状态机的实现和使用（请参考网页[t.cn/Rqr1Tgb]）。

Python提供不止一个第三方包/模块来使用和实现状态机（请参考网页[t.cn/Rqr1Qdn]）。我们将在14.4节中看到如何使用其中的一个。

另一个值得一提的项目是状态机编译器（State Machine Compiler，SMC）。使用SMC，你可以使用一种简单的领域特定语言在文本文件中描述你的状态机，SMC会自动生成状态机的代码。该项目声称这种DSL非常简单，写起来就像一对一地翻译一个状态图。我没试过，但听起来非常有意思。SMC可以生成多种编程语言的代码，包括Python（请参考网页[t.cn/RwDrn4v]）。

## 应用案例

状态模式适用于许多问题。所有可以使用状态机解决的问题都是不错的状态模式应用案例。我们已经见过的一个例子是操作系统/嵌入式系统的进程模型。

编程语言的编译器实现是另一个好例子。词法和旬法分析可使用状态来构建抽象语法树（请参考网页[t.cn/RUFNdYt]）。

事件驱动系统也是另一个例子。在一个事件驱动系统中，从一个状态转换到另一个状态会触发一个事件/消息。许多计算机游戏都使用这一技术。例如，怪兽会在主人公接近时从防御状态转换到攻击状态（请参考网页[t.cn/Rqr13Lr] 和网页[t.cn/Rqr1BW4]）。

这里引用Thomas Jaeger说过的一旬话：“状态设计模式解决的是一定上下文中尤限数量状态的完全封装，从而实现更好的可维护性和灵活性。”（请参考网页[t.cn/8sZrLP0]。）

## 实现

下面编写必需的Python代码，演示一下如何基于本章之前提到的状态图创建一个状态机。我们的状态机应该覆盖一个进程的不同状态以及它们之间的转换。

状态设计模式通常使用一个父State类和许多派生的ConcreteState类来实现，父类包含所有状态共同的功能，每个派生类则仅包含特定状态要求的功能。可在网页[t.cn/h47Rs9]上找到一个样例实现。然而在我看来，这些是实现细节。状态模式关注的是实现一个状态机，状态机的核心部分是状态和状态之间的转换。每个部分具体如何实现并不重要。

为避免重复造轮子，可以利用已有的Python模块。它们不仅能帮助我们创建状态机，而且还是地道的Python方式。我发现state_machine这个模块非常有用（请参考网页[t.cn/RqrBvQG]）。在进一步学习之前，如果你的系统上尚未安装state_machine，请使用下面的命令进行安装。


> pip3 install state_machine

state_machine相当简单，不需要特别的介绍。我们将通过示例代码覆盖该模块的大部分内容。

首先从Process类开始。每个创建好的进程都有自己的状态机。使用state_machine模块创建状态机的第一个步骤是使用@acts_as_state_machine修饰器。


```python
@acts_as_state_machine
class Process:
```

下一步，定义状态机的状态。这是我们在状态图中看到的节点的映射。唯一的区别是应指定状态机的初始状态。这可通过设置inital=True来指定。

```python
created = State(initial=True)
waiting = State()
runnig = State()
terminated = State()
blocked = State()
swapped_out_waiting = State()
swapped_out_blocked = State()
```

接着定义状态转换。在state_machine模块中，一个状态转换就是一个Event。我们使用参数from_states和to_state来定义一个可能的转换。from_states可以是单个状态或一组状态（元组）。


```python
wait = Event(from_states=(created, running, blocked,swapped_out_waiting), to_state=waiting)
run = Event(from_states=waiting, to_state=running)
terminate = Event(from_states=running, to_state=terminated)
block = Event(from_states=(running, swapped_out_blocked),to_state=blocked)
swap_wait = Event(from_states=waiting, to_state=swapped_out_waiting)
swap_block = Event(from_states=blocked, to_state=swapped_out_blocked)
```

每个进程都有一个名称。正式的应用场景中，一个进程需要多得多的信息才能发挥其作用（例如，ID、优先级和状态等），但为了专注于模式本身，我们进行一些简化。


```python
def init (self, name):
    self.name = name
```

在发生状态转换时，如果什么影响都没有，那转换就没什么用了。state_machine模块提供@before和@after修饰器，用于在状态转换之前或之后执行动作。为了达到示例的H的，这里的动作限于输出进程状态转换的信息。

```python
        @after('wait')
        def wait_info(self):
            print('{} entered waiting mode'.format(self.name))

        @after('run')
        def run_info(self):
            print('{} is running'.format(self.name))

        @before('terminate')
        def terminate_info(self):
            print('{} terminated'.format(self.name))
        @after('block')
        def block_info(self):
            print('{} is blocked'.format(self.name))

        @after('swap_wait')
        def swap_wait_info(self):
            print('{} is swapped out and waiting'.format(self.name))

        @after('swap_block')
        def swap_block_info(self):
            print('{} is swapped out and blocked'.format(self.name))

```

transition()函数接受三个参数：process、event和event_name。process是一个Process类实例，event是一个Event类（wait、run和terminate等）实例，而event_name是事件的名称。在尝试执行event时，如果发生错误，则会输出事件的名称。


```python
def transition(process, event, event_name):
    try:
        event()
    except InvalidStateTransition as err:
        print('Error: transition of {} from {} to {} failed'.format(process.name, process.current_state, event_name))
```

state_info()函数展示进程当前（激活）状态的一些基本信息。

```python
def state_info(process):
    print('state of {}: {}'.format(process.name, process.current_state))
```

在main()函数的开始，我们定义了一些字符串常量，作为event_name参数值传递。

```python
def main():
    RUNNING = 'running'
    WAITING = 'waiting'
    BLOCKED = 'blocked'
    TERMINATED = 'terminated'
```

接着，我们创建两个Process实例，并输出它们的初始状态信息。

```python
p1, p2 = Process('process1'), Process('process2')
[state_info(p) for p in (p1, p2)]
```

函数的其余部分将尝试不同的状态转换。回忆一下本章之前提到的状态图。允许的状态转换应与状态图一致。例如，从状态“运行”转换到状态“阻塞”是可能的，但从状态“阻塞”转换到状态“运行”则是不可能的。


```python
print()
transition(p1, p1.wait, WAITING)
transition(p2, p2.terminate, TERMINATED)
[state_info(p) for p in (p1, p2)]
print()
transition(p1, p1.run, RUNNING)
transition(p2, p2.wait, WAITING) 11 [state_info(p) for p in (p1, p2)]
print()
transition(p2, p2.run, RUNNING)
[state_info(p) for p in (p1, p2)]

print() 13 [transition(p, p.block, BLOCKED) for p in (p1, p2)]
[state_info(p) for p in (p1, p2)]
print()
[transition(p, p.terminate, TERMINATED) for p in (p1, p2)]
[state_info(p) for p in (p1, p2)]
```

下面是示例的完整代码（文件state.py）。

```python
from state_machine import State, Event, acts_as_state_machine, after, before, InvalidStateTransition

@acts_as_state_machine
class Process:
    created = State(initial=True)
    waiting = State()
    running = State()
    terminated = State()
    blocked = State()
    swapped_out_waiting = State()
    swapped_out_blocked = State()

    wait = Event(from_states=(created, running, blocked, swapped_out_waiting), to_state=waiting)
    run = Event(from_states=waiting, to_state=running)
    terminate = Event(from_states=running, to_state=terminated)
    block = Event(from_states=(running, swapped_out_blocked), to_state=blocked)
    swap_wait = Event(from_states=waiting, to_state=swapped_out_waiting)
    swap_block = Event(from_states=blocked, to_state=swapped_out_blocked)

    def __init__(self, name):
        self.name = name

    @after('wait')
    def wait_info(self):
        print('{} entered waiting mode'.format(self.name))

    @after('run')
    def run_info(self):
        print('{} is running'.format(self.name))

    @before('terminate')
    def terminate_info(self):
        print('{} terminated'.format(self.name))

    @after('block')
    def block_info(self):
        print('{} is blocked'.format(self.name))

    @after('swap_wait')
    def swap_wait_info(self):
        print('{} is swapped out and waiting'.format(self.name))

    @after('swap_block')
    def swap_block_info(self):
        print('{} is swapped out and blocked'.format(self.name))

def transition(process, event, event_name):
    try:
        event()
    except InvalidStateTransition as err:
        print('Error: transition of {} from {} to {} failed'.format(process.name, process.current_state, event_name))

def state_info(process):
    print('state of {}: {}'.format(process.name, process.current_state))

def main():
    RUNNING = 'running'
    WAITING = 'waiting'
    BLOCKED = 'blocked'
    TERMINATED = 'terminated'

    p1, p2 = Process('process1'), Process('process2')
    [state_info(p) for p in (p1, p2)]

    print()
    transition(p1, p1.wait, WAITING)
    transition(p2, p2.terminate, TERMINATED)
    [state_info(p) for p in (p1, p2)]

    print()
    transition(p1, p1.run, RUNNING)
    transition(p2, p2.wait, WAITING)
    [state_info(p) for p in (p1, p2)]

    print()
    transition(p2, p2.run, RUNNING)
    [state_info(p) for p in (p1, p2)]

    print()
    [transition(p, p.block, BLOCKED) for p in (p1, p2)]
    [state_info(p) for p in (p1, p2)]

    print()
    [transition(p, p.terminate, TERMINATED) for p in (p1, p2)]
    [state_info(p) for p in (p1, p2)]

if __name__ == '__main__':
    main()
```

    state of process1: created
    state of process2: created

    process1 entered waiting mode
    Error: transition of process2 from created to terminated failed
    state of process1: waiting
    state of process2: created

    process1 is running
    process2 entered waiting mode
    state of process1: running
    state of process2: waiting

    process2 is running
    state of process1: running
    state of process2: running

    process1 is blocked
    process2 is blocked
    state of process1: blocked
    state of process2: blocked

    Error: transition of process1 from blocked to terminated failed
    Error: transition of process2 from blocked to terminated failed
    state of process1: blocked
    state of process2: blocked


确实，输出内容显示，非法的状态转换（比如，“已创建”→“终止”和“阻塞”→“终止”）都失败了。我们不希望应用在请求一个非法转换时崩溃，而except代码块能正确地处理这一点。

注意如何使用stat_machine这样的一个好模块来消除条件式逻辑。没有必要使用冗长易错的if-else语旬来检测每个状态转换并作出反应。

为了更好地理解状态模式和状态机，我强烈推荐你实现你自己的例子。可以是任何东西，比如一个简单的电子游戏（你可以使用状态机来处理主人公和敌人的状态）、电梯、解析器或其他任何可以使用状态机来建模的系统。

## 小结

本章中，我们学习了状态设计模式。状态模式是一个或多个有限状态机（简称状态机）的实现，用于解决一个特定的软件工程问题。

状态机是一个抽象机器，具有两个主要部分：状态和转换。状态是指一个系统的当前状况。一个状态机在任意时间点只会有一个激活状态。转换是指从当前状态到一个新状态的切换。在一个转换发生之前或之后通常会执行一个或多个动作。状态机可以使用状态图进行视觉上的展现。

状态机用于解决许多计算机问题和非计算机问题，其中包括交通灯、停车计时器、硬件设计和编程语言解析等。我们也看到零食自动贩卖机是如何与状态机的工作方式相关联的。

许多现代软件提供库/模块来简化状态机的实现与使用。Django提供第三方包django-fsm，Python也有许多大家贡献的模块。实际上，在14.4节就使用了其中的一个模块（state_machine）。状态机编译器是另一个有前景的项H，提供许多编程语言的绑定（包括Python）。

我们学习了如何使用state_machine模块为一个计算机系统的进程实现状态机。state_machine模块简化了状态机的创建以及状态转换之前/之后动作的定义。第15章将学习如何使用策略设计模式实现（在许多候选算法中）动态地选择算法。
