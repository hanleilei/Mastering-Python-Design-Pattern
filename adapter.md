# 适配器模式

结构型设计模式处理一个系统中不同实体(比如，类和对象)之间的关系，关注的是提供一种简单的对象组合方式来创造新功能(请参考[GOF95，第155页]和网页[t.cn/RqBdWzD])。

适配器模式(Adapter pattern)是一种结构型设计模式，**帮助我们实现两个不兼容接口之间的兼容**。首先，解释一下不兼容接口的真正含义。如果我们希望把一个老组件用于一个新系统中，或者把一个新组件用于一个老系统中，不对代码进行任何修改两者就能够通信的情况很少见。但又并非总是能修改代码，或因为我们无法访问这些代码(例如，组件以外部库的方式提供)，或因为修改代码本身就不切实际。在这些情况下，我们 **可以编写一个额外的代码层，该代码层包含让两个接口之间能够通信需要进行的所有修改。这个代码层就叫适配器**。

电子商务系统是这方面众所周知的例子。假设我们使用的一个电子商务系统中包含一个calculate_total(order)函数。这个函数计算一个订单的总金额，但货币单位为丹麦克朗(Danish Kroner，DKK)。顾客让我们支持更多的流行货币，比如美元(United States Dollar，USD) 和欧元(Euro，EUR)，这是很合理的要求。如果我们拥有系统的源代码，那么可以扩展系统，方法是添加一些新函数，将金额从DKK转换成USD，或者从DKK转换成EUR。但是如果应用仅以外部库的方式提供，我们无法访问其源代码，那又该怎么办呢？在这种情况下，我们仍然可以使用这个外部库(例如，调用它的方法)，但无法修改/扩展它。解决方案是编写一个包装器(又名适配器)将数据从给定的DKK格式转换成期望的USD或EUR格式。

适配器模式并不仅仅对数据转换有用。通常来说，如果你想使用一个接口，期望它是function_a()，但仅有function_b()可用，那么可以使用一个适配器把function_b()转换(适配)成function_a()(请参考[Eckel08，第207页]和网页[t.cn/RqBdTCD])。不仅对于函数可以这样做，对于函数参数也可以如此。其中一个例子是，有一个函数要求参数x、y、z，但你手头只有一个带参数x、y的函数。在4.4节我们将看到如何使用适配器模式。


```python
#下面是一个来自github的示例：
# http://ginstrom.com/scribbles/2008/11/06/generic-adapter-class-in-python/

import os


class Dog(object):
    def __init__(self):
        self.name = "Dog"

    def bark(self):
        return "woof!"


class Cat(object):
    def __init__(self):
        self.name = "Cat"

    def meow(self):
        return "meow!"


class Human(object):
    def __init__(self):
        self.name = "Human"

    def speak(self):
        return "'hello'"


class Car(object):
    def __init__(self):
        self.name = "Car"

    def make_noise(self, octane_level):
        return "vroom%s" % ("!" * octane_level)


class Adapter(object):
    """
    Adapts an object by replacing methods.
    Usage:
    dog = Dog
    dog = Adapter(dog, dict(make_noise=dog.bark))
    """
    def __init__(self, obj, adapted_methods):
        """We set the adapted methods in the object's dict"""
        self.obj = obj
        self.__dict__.update(adapted_methods)

    def __getattr__(self, attr):
        """All non-adapted calls are passed to the object"""
        return getattr(self.obj, attr)


def main():
    objects = []
    dog = Dog()
    objects.append(Adapter(dog, dict(make_noise=dog.bark)))
    cat = Cat()
    objects.append(Adapter(cat, dict(make_noise=cat.meow)))
    human = Human()
    objects.append(Adapter(human, dict(make_noise=human.speak)))
    car = Car()
    car_noise = lambda: car.make_noise(3)
    objects.append(Adapter(car, dict(make_noise=car_noise)))

    for obj in objects:
        print("A", obj.name, "goes", obj.make_noise())


if __name__ == "__main__":
    main()
```

    A Dog goes woof!
    A Cat goes meow!
    A Human goes 'hello'
    A Car goes vroom!!!


## 现实生活的例子

也许我们所有人每天都在使用适配器模式，只不过是硬件上的，而不是软件上的。如果你有一部智能手机或者一台平板电脑，在想把它(比如，iPhone手机的闪电接口)连接到你的电脑时，就需要使用一个USB适配器。如果你从大多数欧洲国家到英国旅行，在为你的笔记本电脑充电时，需要使用一个插头适配器。如果你从欧洲到美国旅行，同样如此；反之亦然。适配器无处不在!

下图展示了硬件适配器的若干例子(请参考网页[t.cn/RqBdTCD])，经sourcemaking.com允许使用。

## 软件的例子

Grok是一个Python框架，运行在Zope 3之上，专注于敏捷开发。Grok框架使用适配器，让已有对象无需变更就能符合指定API的标准(请参考网页[t.cn/RqBd1gM])。

Python第三方包Traits也使用了适配器模式，将没有实现某个指定接口(或一组接口)的对象转换成实现了接口的对象(请参考网页[t.cn/RqBdg28])。

## 应用案例

在某个产品制造出来之后，需要应对新的需求之时，如果希望其仍然有效，则可以使用适配器模式(请参考网页[t.cn/RqBdTCD])。通常两个不兼容接口中的一个是他方的或者是老旧的。如果一个接口是他方的，就意味着我们无法访问其源代码。如果是老旧的，那么对其重构通常是不切实际的。更进一步，我们可以说修改一个老旧组件的实现以满足我们的需求，不仅是不切实际的，而且也违反了开放/封闭原则(请参考网页[t.cn/RqBdFAO])。

**开放/封闭原则(open/close principle)是面向对象设计的基本原则之一(SOLID中的O)，声明一个软件实体应该对扩展是开放的，对修改则是封闭的**。本质上这意味着我们应该无需修改一个软件实体的源代码就能扩展其行为。适配器模式遵从开放/封闭原则(请参考网页[t.cn/RqBghbH])。

因此，在某个产品制造出来之后，需要应对新的需求之时，如果希望其仍然有效，使用适配器是一种更好的方式，原因如下所示。

* 不要求访问他方接口的源代码  
* 不违反开放/封闭原则

## 实现

使用Python实现适配器设计模式有多种方法(请参考[Eckel08，第207页]。Bruce Eckel演示的所有技巧都是使用继承，但是Python提供了一种替代方案，在我看来，这种实现适配器的方式更地道一些。你应该已熟悉这一替代技巧，因为它使用了类的内部字典，在第3章中我们就看到了如何使用类的内部字典。

先来看看“我们有什么”部分。我们的应用有一个Computer类，用来显示一台计算机的基本信息。这一例子中的所有类，包括Computer类，都非常简单，因为我们希望关注适配器模式，而不是如何尽可能完善一个类。


```python
class Computer:
        def __init__(self, name):
            self.name = name
        def __str__(self):
            return 'the {} computer'.format(self.name)
        def execute(self):
            return 'executes a program'
```

在这里，execute方法是计算机可以执行的主要动作。这一方法由客户端代码调用。

现在再来看看“我们想要什么”部分。我们决定用更多功能来丰富应用，并且幸运地在两个与我们应用无关的代码库中发现两个有意思的类，Synthesizer和Human。在Synthesizer类中，主要动作由play()方法执行。在Human类中，主要动作由speak()方法执行。为表明这两个类是外部的，将它们放在一个单独的模块中，如下所示。


```python
class Synthesizer:
        def __init__(self, name):
            self.name = name
        def __str__(self):
            return 'the {} synthesizer'.format(self.name)
        def play(self):
            return 'is playing an electronic song'

class Human:
        def __init__(self, name):
            self.name = name
        def __str__(self):
            return 'the {} Human'.format(self.name)
        def play(self):
            return 'say hello'
```

看起来不错，但是有一个问题：客户端仅知道如何调用execute()方法，并不知道play()和speak()。在不改变Synthesizer和Human类的前提下，我们该如何做才能让代码有效？适配器是救星！我们创建一个通用的Adapter类，将一些带不同接口的对象适配到一个统一接口中。\__init__()方法的obj参数是我们想要适配的对象，adapted_methods是一个字典，键值对中的键是客户端要调用的方法，值是应该被调用的方法。


```python
class Adapter:
    def __init__(self, obj, adapted_methods):
        self.obj = obj
        self.__dict__.update(adapted_methods)
    def __str__(self):
        return str(self.obj)
```

下面看看使用适配器模式的方法。列表objects容纳着所有对象。属于Computer类的可兼容对象不需要适配。可以直接将它们添加到列表中。不兼容的对象则不能直接添加。使用Adapter类来适配它们。结果是，对于所有对象，客户端代码都可以始终调用已知的execute()方法，而无需关心被使用的类之间的任何接口差别。


```python
def main():
    objects = [Computer('Asus')]
    synth = Synthesizer('moog')
    objects.append(Adapter(synth, dict(execute=synth.play)))
    human = Human('Bob')
    objects.append(Adapter(human, dict(execute=human.speak)))
    for i in objects:
        print('{} {}'.format(str(i), i.execute()))
```

现在来看看适配器模式例子的完整代码(文件external.py和adapter.py)，如下所示。


```python
class Synthesizer:
    def __init__(self, name):
        self.name = name

    def __str__(self):
        return 'the {} synthesizer'.format(self.name)

    def play(self):
        return 'is playing an electronic song'

class Human:
      def __init__(self, name):
          self.name = name
      def __str__(self):
          return '{} the human'.format(self.name)
      def speak(self):
          return 'says hello'

class Computer:
      def __init__(self, name):
          self.name = name
      def __str__(self):
          return 'the {} computer'.format(self.name)
      def execute(self):
          return 'executes a program'

class Adapter:
      def __init__(self, obj, adapted_methods):
          self.obj = obj
          self.__dict__.update(adapted_methods)
      def __str__(self):
          return str(self.obj)


objects = [Computer('Asus')]
synth = Synthesizer('moog')
objects.append(Adapter(synth, dict(execute=synth.play)))
human = Human('Bob')
objects.append(Adapter(human, dict(execute=human.speak)))
for i in objects:
    print('{} {}'.format(str(i), i.execute()))

```

    the Asus computer executes a program
    the moog synthesizer is playing an electronic song
    Bob the human says hello


我们设法使得Human和Synthesizer类与客户端所期望的接口兼容，且无需改变它们的源代码。这太棒了!

这里有一个为你准备的挑战性练习，当前的实现有一个问题，当所有类都有一个属性name时，以下代码会运行失败。


```python
for i in objects:
    print(i.name)
```

首先想想这段代码为什么会失败?虽然从编码的角度来看这是有意义的，但对于客户端代码来说毫无意义，客户端不应该关心“适配了什么”和“什么没有被适配”这类细节。我们只是想提供一个统一的接口。该如何做才能让这段代码生效?

思考一下如何将未适配部分委托给包含在适配器类中的对象。

## 小结

本章介绍了适配器设计模式。我们使用适配器模式让两个(或多个)不兼容接口兼容。为了引起读者的兴趣，本章先提到一个应该支持多种货币的电子商务系统1。我们每天都在为设备的互连、充电等使用适配器。

适配器让一件产品在制造出来之后需要应对新需求之时还能工作。Python框架Grok和第三方包Traits各自都使用了适配器模式来获得API一致性和接口兼容性。开放/封闭原则与这些方面密切相关。

我们看到了如何使用适配器模式，无需修改不兼容模型的源代码就能获得接口的一致性。这是通过让一个通用的适配器类完成相关工作而实现的。虽然在Python中我们可以沿袭传统方式使用子类(继承)来实现适配器模式，但这种技术是一种很棒的替代方案。
