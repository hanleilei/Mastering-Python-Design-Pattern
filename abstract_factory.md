# 抽象工厂
抽象工厂设计模式是抽象方法的一种泛化。概括来说，一个抽象工厂是(逻辑上的)一组工厂方法，其中的每个工厂方法负责产生不同种类的对象。

## 真实的例子

抽象工厂用在车辆织造中。相同的机械装置用来冲压不同的车辆模型的部件（门，面板，车身罩体，挡泥板，以及各种镜子）。聚合了不同机械装置的模型是可配置的，而且在任何时候都可以轻松变更。在[这个链接](https://sourcemaking.com/design_patterns/abstract_factory)中我们可以看到一个车辆制造的抽象工厂的例子。

如下所示的示例可以作为抽象工厂实现的参考：

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

# http://ginstrom.com/scribbles/2007/10/08/design-patterns-python-style/

"""Implementation of the abstract factory pattern"""

import random

class PetShop:

    """A pet shop"""

    def __init__(self, animal_factory=None):
        """pet_factory is our abstract factory.  We can set it at will."""

        self.pet_factory = animal_factory

    def show_pet(self):
        """Creates and shows a pet using the abstract factory"""

        pet = self.pet_factory.get_pet()
        print("We have a lovely {}".format(pet))
        print("It says {}".format(pet.speak()))
        print("We also have {}".format(self.pet_factory.get_food()))

# Stuff that our factory makes

class Dog:

    def speak(self):
        return "woof"

    def __str__(self):
        return "Dog"


class Cat:

    def speak(self):
        return "meow"

    def __str__(self):
        return "Cat"


# Factory classes

class DogFactory:

    def get_pet(self):
        return Dog()

    def get_food(self):
        return "dog food"


class CatFactory:

    def get_pet(self):
        return Cat()

    def get_food(self):
        return "cat food"


# Create the proper family
def get_factory():
    """Let's be dynamic!"""
    return random.choice([DogFactory, CatFactory])()


# Show pets with various factories

for i in range(3):
    shop = PetShop(get_factory())
    shop.show_pet()
    print("=" * 20)

### OUTPUT ###
# We have a lovely Dog
# It says woof
# We also have dog food
# ====================
# We have a lovely Dog
# It says woof
# We also have dog food
# ====================
# We have a lovely Cat
# It says meow
# We also have cat food
# ====================


```

## 软件例子

程序包django_factory是一个用于在测试中创建Django模型的抽象工厂实现，可用来为支持测试专有属性的模型创建实例。这能让测试代码的可读性更高，且能避免共享不必要的代码，故有其存在的价值（请参考网页［t.cn/RqBBvcw］）。

## 使用案例

因为抽象工模式是工厂方法模式的归纳，它具有同样的**优点：使得追踪一个对象创建更容易，它让对象的使用和创建分离开，并且给我们内存使用以及应用的性能提高的可能**。

这样会产生一个问题：我们怎么知道何时该使用工厂方法，何时又该使用抽象工厂？答案是，通常一开始时使用工厂方法，因为它更简单。如果后来发现应用需要许多工厂方法，那么将创建一系列对象的过程合并在一起更合理，从而最终引入抽象工厂。

抽象工厂有一个优点，在使用工厂方法时从用户视角通常是看不到的，那就是抽象工厂能够通过改变激活的工厂方法动态地(运行时)改变应用行为。一个经典例子是能够让用户在使用应用时改变应用的观感(比如，Apple风格和Windows风格等),而不需要终止应用然后重新启动。


## 实现

为演示抽象工厂模式，我将重新使用Python 3 Patterns & Idioms（Bruce Eckel著）一书中的一个例子，它是我个人最喜欢的例子之一。想象一下，我们正在创造一个游戏，或者想在应用中包含一个迷你游戏让用户娱乐娱乐。我们希望至少包含两个游戏, 一个面向孩子，一个面向成人。在运行时，基于用户输入，决定该创建哪个游戏并运行。游戏的创建部分由一个抽象工厂维护。

从孩子的游戏说起，我们将该游戏命名为 FrogWorld。主人公是一只青蛙，喜欢吃虫子。每个英雄都需要一个好名字，在我们的例子中，这个名字在运行时由用户给定。方法 interact_with()用于描述青蛙与障碍物(比如,虫子、迷宫或其他青蛙)之间的交互，如下所示：

```python
Class Frog:
    def __init__(self, name):
        self.name = name

    def __str__(self):
        return self.name

    def interact_with(self, obstacle):
        print('{} the Frog encounters {} and {}!'.format(self, obstacle, obstacle.action()))
```

可以有很多种类的障碍，不过对于我们的这个例子来说可以只有一个Bug。青蛙遇到一个虫子，只有一个行为被支持：吃掉它！

```python
Class Bug:
    def __str__(self):
        return 'a bug'

    def action(self):
        return 'eats it'
```

FrogWorld类是一个抽象工厂。它的主要责任是游戏的主角和障碍物。保持创建方法的独立和方法名称的普通（例如，make_character()，make_obstacle()）让我们可以动态的改变活动工厂（以及活动的游戏）而不设计任何的代码变更。如下所示，在一个静态类型语言中，抽象工厂可以是一个有着空方法的抽象类/接口，但是在Python中不要求这么做，因为类型在运行时才被检查。

```python
Class FrogWorld:
    def __init__(self, name):
        print(self)
        self.player_name = name

    def __str__(self):
        return '\n\n\t------ Frog World ------'

    def make_character(self):
        return Forg(self.player_name)

    def make_obstacle(self):
        return Bug()
```

游戏WizarWorld也类似。唯一的不同是巫师不吃虫子而是与半兽人这样的怪物战斗！

```python
Class Wizard:
    def __init__(self, name):
        self.name = name

    def __str__(self):
        return self.name

    def interact_with(self, obstacle):
        print('{} the Wizard batteles against {} and {}!'.format(
            self, obstacle, obstacle.action()))


Class Ork:
    def __str__(self):
        return 'an evial ork'

    def action(self):
        return 'kills it'



Class WizardWorld:
    def __init__(self, name):
        print(self)
        self.player_name = name

    def __str__(self):
        return '\n\n\t------ Wizard World ------'

    def make_character(self):
        return Wizard(self.player_name)

    def make_obstacle(self):
        return Ork()


```
GameEnviroment是我们游戏的主要入口。它接受factory作为输入，然后用它创建游戏的世界。play()方法发起创造的hero和obstacle之间的交互如下：

```python
Class GameEnviroment:
        def __init__(self, factory):
            self.hero = factory.make_character()
            self.obstacle = factory.make_obstacle()

        def play(self):
            self.hero.interact_with(self.obstacle)
```

validate_age()函数提示用户输入一个有效的年龄。若年龄无效，则返回一个第一个元素设置为False的元组。如下所示，若输入的年龄没问题，元组的第一个元素设置为True，以及我们所关心的该元组的第二个元素，即用户给定的年龄：

```python
def validate_age(name):
        try:
            age = input('Welcome {}. How old are you？'.format(name))
            age = int(age)
        except ValueError as err:
            print("Age {} is valid, plz try again...".format(age))
            return (False, age)
        return (True, age)
```

最后，尤其是main()函数的出现。它询问了用户的名字和年龄，然后由用户的年龄来决定哪一个游戏应该被运行：

```python
def main():
        name = input("Hello. What's your name?")
        valid_input = False
        while not valid_input:
            valid_input, age = validate_age(name)
            game = FrogWorld if age < 18 else WizardWorld
            enviroment = GameEnviroment(game(name))
            enviroment.play()

```
抽象工厂实现（abstrac_factory.py）的完整代码如下：

```python
class Frog:

    def __init__(self, name):
        self.name = name

    def __str__(self):
        return self.name

    def interact_with(self, obstacle):
        print('{} the Frog encounters {} and {}!'.format(self,
        obstacle, obstacle.action()))


class Bug:

    def __str__(self):
        return 'a bug'

    def action(self):
        return 'eats it'


class FrogWorld:

    def __init__(self, name):
        print(self)
        self.player_name = name

    def __str__(self):
        return '\n\n\t------ Frog World -------'

    def make_character(self):
        return Frog(self.player_name)

    def make_obstacle(self):
        return Bug()


class Wizard:

    def __init__(self, name):
        self.name = name

    def __str__(self):
        return self.name


    def interact_with(self, obstacle):
        print(
            '{} the Wizard battles against {} and {}!'.format(
            self,
            obstacle,
            obstacle.action()))


class Ork:

    def __str__(self):
        return 'an evil ork'

    def action(self):
        return 'kills it'


class WizardWorld:

    def __init__(self, name):
        print(self)
        self.player_name = name

    def __str__(self):
        return '\n\n\t------ Wizard World -------'

    def make_character(self):
        return Wizard(self.player_name)

    def make_obstacle(self):
        return Ork()

class GameEnvironment:

    def __init__(self, factory):
        self.hero = factory.make_character()
        self.obstacle = factory.make_obstacle()

    def play(self):
        self.hero.interact_with(self.obstacle)

def validate_age(name):
    try:
        age = input('Welcome {}. How old are you? '.format(name))
        age = int(age)
    except ValueError as err:
        print("Age {} is invalid, please try again...".format(age))
        return (False, age)
    return (True, age)

def main():
    name = input("Hello. What's your name? ")
    valid_input = False
    while not valid_input:
        valid_input, age = validate_age(name)
    game = FrogWorld if age < 18 else WizardWorld
    environment = GameEnvironment(game(name))
    environment.play()

main()


# Hello. What's your name? Nick
# Welcome Nick. How old are you? 17


# 	------ Frog World -------
# Nick the Frog encounters a bug and eats it!


```
试着扩展游戏使它更加复杂。你的思想有多远就可以走多远：更多的障碍物，更多的敌人，以及任何你喜欢的东西。

##Abstract Factory versus Factory Method

Let's look at the instances where we need to use the Abstract Factory or Factory Method.

Use the Factory Method pattern when there is a need to decouple a client from a particular product it uses. Use the Factory Method to relieve a client of the responsibility of creating and configuring instances of a product.

Use the Abstract Factory pattern when clients must be decoupled from the product classes. The Abstract Factory pattern can also enforce constraints specifying which classes must be used with others, creating independent families of objects.”

## 总结

本章中,我们学习了如何使用工厂方法和抽象工厂设计模式。两种模式都可以用于以下几种场景:(a)想要追踪对象的创建时,(b)想要将对象的创建与使用解耦时，(c)想要优化应用的性能和资源占用时。场景(c)在本章中并未详细说明，你也许可以将其作为一个练习。

工厂方法设计模式的实现是一个不属于任何类的单一函数，负责单一种类对象(一个形状、一个连接点或者其他对象)的创建。我们看到工厂方法是如何与玩具制造相关联的，提到Django是如何将其用于创建不同表单字段的,并讨论了其他可能的应用案例。作为示例，我们实现了一个工厂方法，提供了访问XML和JSON文件的能力。

抽象工厂设计模式的实现是同属于单个类的许多个工厂方法用于创建一系列种类的相关对象(一辆车的部件、一个游戏的环境,或者其他对象)。我们提到抽象工厂如何与汽车制造业相关联，Django程序包django_factory是如何利用抽象工厂创建干净的测试用例，并学习了抽象工厂的应用案例。作为抽象工厂实现的示例，我们完成了一个迷你游戏，演示了如何在单个类中使用多个相关工厂。








1
