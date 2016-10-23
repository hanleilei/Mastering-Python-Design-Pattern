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























1
