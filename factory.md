# 工厂模式

创建型设计模式处理对象创建相关的问题，目标是当直接创建对象(在Python中是通过__init__()函数实现的)不太方便时，提供更好的方式。

在工厂设计模式中，客户端1可以请求一个对象，而无需知道这个对象来自哪里；也就是说，无需知道使用哪个类来生成这个对象。工厂背后的思想是简化对象的创建。与客户端自己基于类实例化直接创建对象相比，基于一个中心化函数来实现，也更易于追踪创建了哪些对象。通过将创建对象的代码和使用对象的代码解耦，工厂能够降低应用维护的复杂度。

工厂通常有两种形式：一种是工厂方法(Factory Method)，它是一个方法(或以地道的Python 术语来说，是一个函数)，对不同的输入参数返回不同的对象；第二种是抽象工厂，它是一组用于创建一系列相关事物对象的工厂方法。

## 工厂方法

在工厂方法模式中，我们执行单个函数，传入一个参数(提供信息表明我们想要什么)，但并不要求知道任何关于对象如何实现以及对象来自哪里的细节。

如果因为应用创建对象的代码分布在多个不同的地方，而不是仅在一个函数/方法中，你发现没法跟踪这些对象，那么应该考虑使用工厂方法模式。工厂方法集中地在一个地方创建对象，使对象跟踪变得更容易。注意，创建多个工厂方法也完全没有问题，实践中通常也这么做，对相似的对象创建进行逻辑分组，每个工厂方法负责一个分组。例如，有一个工厂方法负责连接到不同的数据库(MySQL、SQLite)，另一个工厂方法负责创建要求的几何对象(圆形、三角形)，等等。

若需要将对象的创建和使用解耦，工厂方法也能派上用场。创建对象时，我们并没有与某个 12 特定类耦合/绑定到一起，而只是通过调用某个函数来提供关于我们想要什么的部分信息。这意味着修改这个函数比较容易，不需要同时修改使用这个函数的代码。

另外一个值得一提的应用案例与应用性能及内存使用相关。工厂方法可以在必要时创建新的对象，从而提高性能和内存使用率。若直接实例化类来创建对象，那么每次创建新对象就需要分配额外的内存(除非这个类内部使用了缓存，一般情况下不会这样)。用行动说话，下面的代码(文件id.py)对同一个类A创建了两个实例，并使用函数id()比较它们的内存地址。输出中也会包含地址，便于检查地址是否正确。内存地址不同就意味着创建了两个不同的对象。

首先，让我们先查看一下github上的代码：
```python
# 参考github上的代码：
#!/usr/bin/env python
# -*- coding: utf-8 -*-

"""http://ginstrom.com/scribbles/2007/10/08/design-patterns-python-style/"""


class GreekGetter:

    """A simple localizer a la gettext"""

    def __init__(self):
        self.trans = dict(dog="σκύλος", cat="γάτα")

    def get(self, msgid):
        """We'll punt if we don't have a translation"""
        return self.trans.get(msgid, str(msgid))

class EnglishGetter:

    """Simply echoes the msg ids"""

    def get(self, msgid):
        return str(msgid)

def get_localizer(language="English"):
    """The factory method"""
    languages = dict(English=EnglishGetter, Greek=GreekGetter)
    return languages[language]()

# Create our localizers
e, g = get_localizer(language="English"), get_localizer(language="Greek")
# Localize some text
for msgid in "dog parrot cat bear".split():
    print(e.get(msgid), g.get(msgid))

### OUTPUT ###
# dog σκύλος
# parrot parrot
# cat γάτα
# bear bear
```
通过一个函数工厂，返回得到相应的实例，所要传入的是一个参数，其他都不需要去管理，函数自动找到需要实例化的类。

我们将使用Python发行版自带的两个库(xml.etree.ElementTree和json)来处理 XML 和 JSON，如下所示：
```python
import xml.etree.ElementTree as etree
import json
```

类JSONConnector解析JSON文件，通过parsed_data()方法以一个字典(dict)的形式 返回数据。修饰器property使parsed_data()显得更像一个常规的变量，而不是一个方法，如下所示。

```python
class JSONConnector:

    def __init__(self, filepath):
        self.data = dict()
        # 比较奇怪的是这里会报错
        # with open(filepath, mode='r', encoding='utf-8') as f:
        with open(filepath, mode='r') as f:
            self.data = json.load(f)

    @property
    def parsed_data(self):
        return self.data
```
类XMLConnector解析 XML 文件，通过parsed_data()方法以xml.etree.Element列表的形式返回所有数据，如下所示：
```python
class XMLConnector:

    def __init__(self, filepath):
        self.tree = etree.parse(filepath)

    @property
    def parsed_data(self):
        return self.tree
```
函数connection_factory是一个工厂方法，基于输入文件路径的扩展名返回一个JSONConnector或XMLConnector的实例，如下所示:

```python
def connection_factory(filepath):
    if filepath.endswith('json'):
        connector = JSONConnector
    elif filepath.endswith('xml'):
        connector = XMLConnector
    else:
        raise ValueError('Cannot connect to {}'.format(filepath))
    return connector(filepath)
```

函数connect_to()对connection_factory()进行包装，添加了异常处理，如下所示:

```python
def connect_to(filepath):
    factory = None
    try:
        factory = connection_factory(filepath)
    except ValueError as ve:
        print(ve)
    return factory
```

演示如何使用工厂方法处理XML文件。XPath用于查找所有包含姓(last name) 为Liar的person元素。对于每个匹配到的元素，展示其基本的姓名和电话号码信息，如下所示。

```python
xml_factory = connect_to('data/person.xml')
xml_data = xml_factory.parsed_data
liars = xml_data.findall(".//{}[{}='{}']".format('person', 'lastName', 'Liar'))
print('found: {} persons'.format(len(liars)))
for liar in liars:
    print('first name: {}'.format(liar.find('firstName').text))
    print('last name: {}'.format(liar.find('lastName').text))
    [print('phone number ({})'.format(p.attrib['type']), p.text) for p in liar.find('phoneNumbers')]
```

最后一部分演示如何使用工厂方法处理JSON文件。这里没有模式匹配,因此所有甜甜圈的 name、price和topping,如下所示。

```python
json_factory = connect_to('data/donut.json')
json_data = json_factory.parsed_data
print('found: {} donuts'.format(len(json_data)))
for donut in json_data:
    print('name: {}'.format(donut['name']))
    print('price: ${}'.format(donut['ppu']))
    [print('topping: {} {}'.format(t['id'], t['type'])) for t in donut['topping']]
```

完整代码如下所示：
```python
import xml.etree.ElementTree as etree
import json

class JSONConnector:

    def __init__(self, filepath):
        self.data = dict()
        # 比较奇怪的是这里会报错
        # with open(filepath, mode='r', encoding='utf-8') as f:
        with open(filepath, mode='r') as f:
            self.data = json.load(f)

    @property
    def parsed_data(self):
        return self.data

class XMLConnector:

    def __init__(self, filepath):
        self.tree = etree.parse(filepath)

    @property
    def parsed_data(self):
        return self.tree

def connection_factory(filepath):
    if filepath.endswith('json'):
        connector = JSONConnector
    elif filepath.endswith('xml'):
        connector = XMLConnector
    else:
        raise ValueError('Cannot connect to {}'.format(filepath))
    return connector(filepath)

def connect_to(filepath):
    factory = None
    try:
        factory = connection_factory(filepath)
    except ValueError as ve:
        print(ve)
    return factory


sqlite_factory = connect_to('data/person.sq3')
print()
xml_factory = connect_to('data/person.xml')
xml_data = xml_factory.parsed_data
liars = xml_data.findall(".//{}[{}='{}']".format('person', 'lastName', 'Liar'))
print('found: {} persons'.format(len(liars)))
for liar in liars:
    print('first name: {}'.format(liar.find('firstName').text))
    print('last name: {}'.format(liar.find('lastName').text))
    [print('phone number ({})'.format(p.attrib['type']), p.text) for p in liar.find('phoneNumbers')]
print()
json_factory = connect_to('data/donut.json')
json_data = json_factory.parsed_data
print('found: {} donuts'.format(len(json_data)))
for donut in json_data:
    print('name: {}'.format(donut['name']))
    print('price: ${}'.format(donut['ppu']))
    [print('topping: {} {}'.format(t['id'], t['type'])) for t in donut['topping']]
```

注意，虽然JSONConnector和XMLConnector拥有相同的接口，但是对于parsed_data() 返回的数据并不是以统一的方式进行处理。对于每个连接器，需使用不同的Python代码来处理。 若能对所有连接器应用相同的代码当然最好，但是在多数时候这是不现实的，除非对数据使用某 种共同的映射，这种映射通常是由外部数据提供者提供。即使假设可以使用相同的代码来处理 XML和JSON文件，当需要支持第三种格式(例如，SQLite)时，又该对代码作哪些改变呢?找一个SQlite文件或者自己创建一个，尝试一下。

只需要通过connection_factory上加一个sqlite的文件格式，然后再添加一个对于sql操作的类就好。

代码并未禁止直接实例化一个连接器。如果要禁止直接实例化，是否可以实现？

可以，Python中允许class中嵌套class。
