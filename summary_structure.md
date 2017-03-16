# 结构型模式

结构型模式的用途：将一种对象改装为另一种对象，或者将小对象合并为大的对象。有三个主题：适配借口（adapt interface），增加功能（add functionality）和处理对象集群（handle collections of object)。

Structural Patterns:

Pattern	                Description
3-tier	                data<->business logic<->presentation separation (strict relationships)
adapter	                adapt one interface to another using a white-list
bridge	                a client-provider middleman to soften interface changes
composite	            encapsulate and provide access to a number of different objects
decorator	            wrap functionality with other functionality in order to affect outputs
facade	                use one class as an API to a number of others
flyweight	            transparently reuse existing instances of objects with similar/identical state
front_controller	    single handler requests coming to the application
mvc	                    model<->view<->controller (non-strict relationships)
proxy	                an object funnels operations to something else


已经介绍过适配器（adapter），装饰器（decorator），外观（facade），享元（flyweight），mvc和代理（proxy）模式，没有介绍的有3-tier，桥接（bridge），组合（composite）和front_controller模式。下面将分别介绍一些这些没有介绍过的内容。

## 3-tier模式：

对于3-tier和mvc的区别，可以参考这个[链接](http://stackoverflow.com/questions/3129633/model-view-presenter-and-three-tier/8291398):

MVC is an UI pattern, three tier is an application architecture pattern. That is you can design your application with 3 tiers - UI, BL, data. And than use MVC in the UI tier

三层架构(3-tier Architecture )通常意义上的三层架构就是将整个业务应用划分为：表现层（UI）、业务逻辑层（BLL）、数据访问层（DAL）。区分层次的目的即为了 “高内聚，低耦合”的思想。

1.表现层（UI）：通俗讲就是展现给用户的界面，即用户在使用一个系统的时候他的所见所得。  
2.业务逻辑层（BLL）：针对具体问题的操作，也可以说是对数据层的操作，对数据业务逻辑处理。  
3.数据访问层（DAL）：该层所做事务直接操作数据库，针对数据的增、删、改、查。

所以，严格的讲，3-tier不能算得上是一个设计模式，只是一种设计架构。

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-


class Data(object):
    """ Data Store Class """

    products = {
        'milk': {'price': 1.50, 'quantity': 10},
        'eggs': {'price': 0.20, 'quantity': 100},
        'cheese': {'price': 2.00, 'quantity': 10}
    }

    def __get__(self, obj, klas):
        print("(Fetching from Data Store)")
        return {'products': self.products}


class BusinessLogic(object):
    """ Business logic holding data store instances """

    data = Data()

    def product_list(self):
        return self.data['products'].keys()

    def product_information(self, product):
        return self.data['products'].get(product, None)


class Ui(object):
    """ UI interaction class """

    def __init__(self):
        self.business_logic = BusinessLogic()

    def get_product_list(self):
        print('PRODUCT LIST:')
        for product in self.business_logic.product_list():
            print(product)
        print('')

    def get_product_information(self, product):
        product_info = self.business_logic.product_information(product)
        if product_info:
            print('PRODUCT INFORMATION:')
            print('Name: {0}, Price: {1:.2f}, Quantity: {2:}'.format(
                product.title(), product_info.get('price', 0),
                product_info.get('quantity', 0)))
        else:
            print('That product "{0}" does not exist in the records'.format(
                product))


def main():
    ui = Ui()
    ui.get_product_list()
    ui.get_product_information('cheese')
    ui.get_product_information('eggs')
    ui.get_product_information('milk')
    ui.get_product_information('arepas')

if __name__ == '__main__':
    main()

### OUTPUT ###
# PRODUCT LIST:
# (Fetching from Data Store)
# cheese
# eggs
# milk
#
# (Fetching from Data Store)
# PRODUCT INFORMATION:
# Name: Cheese, Price: 2.00, Quantity: 10
# (Fetching from Data Store)
# PRODUCT INFORMATION:
# Name: Eggs, Price: 0.20, Quantity: 100
# (Fetching from Data Store)
# PRODUCT INFORMATION:
# Name: Milk, Price: 1.50, Quantity: 10
# (Fetching from Data Store)
# That product "arepas" does not exist in the records
```

## 桥接模式（bridge）

桥接模式用于将抽象（abstraction，比如接口或者算法）与实现方式相分离。

如果不用桥接模式，通常的方法就是创建若干个基类，用于表示各种抽象方式，然后从每个基类中继承出两个或者多个子类，用于
表示这种抽象方式的不同实现方法。使用桥接模式，需要定义两套独立的“类体系”：抽象体系定义了我们所要执行的操作：接口
和高层算法，实现体系包含具体的实现方式；抽象体系要调用实现体系以完成其所想要的操作。抽象体系中的类会把实现体系中的
某个实例聚合起来，而这个实例将充当抽象接口与具体实现之间的桥梁。

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

"""http://en.wikibooks.org/wiki/Computer_Science_Design_Patterns/Bridge_Pattern#Python"""


# ConcreteImplementor 1/2
class DrawingAPI1(object):

    def draw_circle(self, x, y, radius):
        print('API1.circle at {}:{} radius {}'.format(x, y, radius))


# ConcreteImplementor 2/2
class DrawingAPI2(object):

    def draw_circle(self, x, y, radius):
        print('API2.circle at {}:{} radius {}'.format(x, y, radius))


# Refined Abstraction
class CircleShape(object):

    def __init__(self, x, y, radius, drawing_api):
        self._x = x
        self._y = y
        self._radius = radius
        self._drawing_api = drawing_api

    # low-level i.e. Implementation specific
    def draw(self):
        self._drawing_api.draw_circle(self._x, self._y, self._radius)

    # high-level i.e. Abstraction specific
    def scale(self, pct):
        self._radius *= pct


def main():
    shapes = (
        CircleShape(1, 2, 3, DrawingAPI1()),
        CircleShape(5, 7, 11, DrawingAPI2())
    )

    for shape in shapes:
        shape.scale(2.5)
        shape.draw()


if __name__ == '__main__':
    main()

### OUTPUT ###
# API1.circle at 1:2 radius 7.5
# API2.circle at 5:7 radius 27.5
```

## composite

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

"""
A class which defines a composite object which can store
hieararchical dictionaries with names.
This class is same as a hiearchical dictionary, but it
provides methods to add/access/modify children by name,
like a Composite.
Created Anand B Pillai     <abpillai@gmail.com>
"""
__author__ = "Anand B Pillai"
__maintainer__ = "Anand B Pillai"
__version__ = "0.2"


def normalize(val):
    """ Normalize a string so that it can be used as an attribute
    to a Python object """

    if val.find('-') != -1:
        val = val.replace('-', '_')

    return val


def denormalize(val):
    """ De-normalize a string """

    if val.find('_') != -1:
        val = val.replace('_', '-')

    return val


class SpecialDict(dict):

    """ A dictionary type which allows direct attribute
    access to its keys """

    def __getattr__(self, name):

        if name in self.__dict__:
            return self.__dict__[name]
        elif name in self:
            return self.get(name)
        else:
            # Check for denormalized name
            name = denormalize(name)
            if name in self:
                return self.get(name)
            else:
                raise AttributeError('no attribute named %s' % name)

    def __setattr__(self, name, value):

        if name in self.__dict__:
            self.__dict__[name] = value
        elif name in self:
            self[name] = value
        else:
            # Check for denormalized name
            name2 = denormalize(name)
            if name2 in self:
                self[name2] = value
            else:
                # New attribute
                self[name] = value


class CompositeDict(SpecialDict):

    """ A class which works like a hierarchical dictionary.
    This class is based on the Composite design-pattern """

    ID = 0

    def __init__(self, name=''):

        if name:
            self._name = name
        else:
            self._name = ''.join(('id#', str(self.__class__.ID)))
            self.__class__.ID += 1

        self._children = []
        # Link  back to father
        self._father = None
        self[self._name] = SpecialDict()

    def __getattr__(self, name):

        if name in self.__dict__:
            return self.__dict__[name]
        elif name in self:
            return self.get(name)
        else:
            # Check for denormalized name
            name = denormalize(name)
            if name in self:
                return self.get(name)
            else:
                # Look in children list
                child = self.findChild(name)
                if child:
                    return child
                else:
                    attr = getattr(self[self._name], name)
                    if attr:
                        return attr

                    raise AttributeError('no attribute named %s' % name)

    def isRoot(self):
        """ Return whether I am a root component or not """

        # If I don't have a parent, I am root
        return not self._father

    def isLeaf(self):
        """ Return whether I am a leaf component or not """

        # I am a leaf if I have no children
        return not self._children

    def getName(self):
        """ Return the name of this ConfigInfo object """

        return self._name

    def getIndex(self, child):
        """ Return the index of the child ConfigInfo object 'child' """

        if child in self._children:
            return self._children.index(child)
        else:
            return -1

    def getDict(self):
        """ Return the contained dictionary """

        return self[self._name]

    def getProperty(self, child, key):
        """ Return the value for the property for child
        'child' with key 'key' """

        # First get the child's dictionary
        childDict = self.getInfoDict(child)
        if childDict:
            return childDict.get(key, None)

    def setProperty(self, child, key, value):
        """ Set the value for the property 'key' for
        the child 'child' to 'value' """

        # First get the child's dictionary
        childDict = self.getInfoDict(child)
        if childDict:
            childDict[key] = value

    def getChildren(self):
        """ Return the list of immediate children of this object """

        return self._children

    def getAllChildren(self):
        """ Return the list of all children of this object """

        l = []
        for child in self._children:
            l.append(child)
            l.extend(child.getAllChildren())

        return l

    def getChild(self, name):
        """ Return the immediate child object with the given name """

        for child in self._children:
            if child.getName() == name:
                return child

    def findChild(self, name):
        """ Return the child with the given name from the tree """

        # Note - this returns the first child of the given name
        # any other children with similar names down the tree
        # is not considered.

        for child in self.getAllChildren():
            if child.getName() == name:
                return child

    def findChildren(self, name):
        """ Return a list of children with the given name from the tree """

        # Note: this returns a list of all the children of a given
        # name, irrespective of the depth of look-up.

        children = []

        for child in self.getAllChildren():
            if child.getName() == name:
                children.append(child)

        return children

    def getPropertyDict(self):
        """ Return the property dictionary """

        d = self.getChild('__properties')
        if d:
            return d.getDict()
        else:
            return {}

    def getParent(self):
        """ Return the person who created me """

        return self._father

    def __setChildDict(self, child):
        """ Private method to set the dictionary of the child
        object 'child' in the internal dictionary """

        d = self[self._name]
        d[child.getName()] = child.getDict()

    def setParent(self, father):
        """ Set the parent object of myself """

        # This should be ideally called only once
        # by the father when creating the child :-)
        # though it is possible to change parenthood
        # when a new child is adopted in the place
        # of an existing one - in that case the existing
        # child is orphaned - see addChild and addChild2
        # methods !
        self._father = father

    def setName(self, name):
        """ Set the name of this ConfigInfo object to 'name' """

        self._name = name

    def setDict(self, d):
        """ Set the contained dictionary """

        self[self._name] = d.copy()

    def setAttribute(self, name, value):
        """ Set a name value pair in the contained dictionary """

        self[self._name][name] = value

    def getAttribute(self, name):
        """ Return value of an attribute from the contained dictionary """

        return self[self._name][name]

    def addChild(self, name, force=False):
        """ Add a new child 'child' with the name 'name'.
        If the optional flag 'force' is set to True, the
        child object is overwritten if it is already there.
        This function returns the child object, whether
        new or existing """

        if type(name) != str:
            raise ValueError('Argument should be a string!')

        child = self.getChild(name)
        if child:
            # print('Child %s present!' % name)
            # Replace it if force==True
            if force:
                index = self.getIndex(child)
                if index != -1:
                    child = self.__class__(name)
                    self._children[index] = child
                    child.setParent(self)

                    self.__setChildDict(child)
            return child
        else:
            child = self.__class__(name)
            child.setParent(self)

            self._children.append(child)
            self.__setChildDict(child)

            return child

    def addChild2(self, child):
        """ Add the child object 'child'. If it is already present,
        it is overwritten by default """

        currChild = self.getChild(child.getName())
        if currChild:
            index = self.getIndex(currChild)
            if index != -1:
                self._children[index] = child
                child.setParent(self)
                # Unset the existing child's parent
                currChild.setParent(None)
                del currChild

                self.__setChildDict(child)
        else:
            child.setParent(self)
            self._children.append(child)
            self.__setChildDict(child)


if __name__ == "__main__":
    window = CompositeDict('Window')
    frame = window.addChild('Frame')
    tfield = frame.addChild('Text Field')
    tfield.setAttribute('size', '20')

    btn = frame.addChild('Button1')
    btn.setAttribute('label', 'Submit')

    btn = frame.addChild('Button2')
    btn.setAttribute('label', 'Browse')

    # print(window)
    # print(window.Frame)
    # print(window.Frame.Button1)
    # print(window.Frame.Button2)
    print(window.Frame.Button1.label)
    print(window.Frame.Button2.label)

### OUTPUT ###
# Submit
# Browse
```

## front_controller

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

"""
@author: Gordeev Andrey <gordeev.and.and@gmail.com>
The controller provides a centralized entry point that controls and manages
request handling.
"""


class MobileView(object):
    def show_index_page(self):
        print('Displaying mobile index page')


class TabletView(object):
    def show_index_page(self):
        print('Displaying tablet index page')


class Dispatcher(object):
    def __init__(self):
        self.mobile_view = MobileView()
        self.tablet_view = TabletView()

    def dispatch(self, request):
        if request.type == Request.mobile_type:
            self.mobile_view.show_index_page()
        elif request.type == Request.tablet_type:
            self.tablet_view.show_index_page()
        else:
            print('cant dispatch the request')


class RequestController(object):
    """ front controller """
    def __init__(self):
        self.dispatcher = Dispatcher()

    def dispatch_request(self, request):
        if isinstance(request, Request):
            self.dispatcher.dispatch(request)
        else:
            print('request must be a Request object')


class Request(object):
    """ request """

    mobile_type = 'mobile'
    tablet_type = 'tablet'

    def __init__(self, request):
        self.type = None
        request = request.lower()
        if request == self.mobile_type:
            self.type = self.mobile_type
        elif request == self.tablet_type:
            self.type = self.tablet_type


if __name__ == '__main__':
    front_controller = RequestController()
    front_controller.dispatch_request(Request('mobile'))
    front_controller.dispatch_request(Request('tablet'))

    front_controller.dispatch_request(Request('desktop'))
    front_controller.dispatch_request('mobile')


### OUTPUT ###
# Displaying mobile index page
# Displaying tablet index page
# cant dispatch the request
# request must be a Request object
```











1
