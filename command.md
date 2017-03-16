# 命令模式

现在多数应用都有撤销操作。虽然难以想象，但在很多年里，任何软件中确实都不存在撤销操作。撤销操作是在1974年引入的（请参考网页[t.cn/Rqr3N22]），但Fortran和Lisp分别早在1957年和1958年就已创建了撤销操作（请参考网页[t.cn/Rqr3067]），这两门语言仍在被人广泛使用。在那些年里，我真心不想使用应用软件。犯了一个错误，用户也没什么便捷方式能修正它。

历史就讲到这里。我们想知道如何在应用中实现撤销功能。你已读过本章的标题，所以知道应该推荐哪个设计模式来实现撤销，那就是命令模式（Command pattern）。

命令设计模式帮助我们将一个操作（撤销、重做、复制、粘贴等）封装成一个对象。简而言之，这意味着创建一个类，包含实现该操作所需要的所有逻辑和方法。这样做的优势如下所述（请参考[GOF95，第265页]和网页[t.cn/Rqr3tfQ]）。

* 我们并不需要直接执行一个命令。命令可以按照希望执行。
* 调用命令的对象与知道如何执行命令的对象解耦。调用者不需知道命令的任何实现细节。
* 如果有意义，可以把多个命令组织起来，这样调用者能够按顺序执行它们。例如，在实现一个多层撤销命令时，这是很有用的。

以下的例子来自Github：

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
from __future__ import print_function
import os
from os.path import lexists


class MoveFileCommand(object):

    def __init__(self, src, dest):
        self.src = src
        self.dest = dest

    def execute(self):
        self.rename(self.src, self.dest)

    def undo(self):
        self.rename(self.dest, self.src)

    def rename(self, src, dest):
        print(u"renaming %s to %s" % (src, dest))
        os.rename(src, dest)


def main():
    command_stack = []

    # commands are just pushed into the command stack
    command_stack.append(MoveFileCommand('foo.txt', 'bar.txt'))
    command_stack.append(MoveFileCommand('bar.txt', 'baz.txt'))

    # verify that none of the target files exist
    assert(not lexists("foo.txt"))
    assert(not lexists("bar.txt"))
    assert(not lexists("baz.txt"))
    try:
        with open("foo.txt", "w"):  # Creating the file
            pass

        # they can be executed later on
        for cmd in command_stack:
            cmd.execute()

        # and can also be undone at will
        for cmd in reversed(command_stack):
            cmd.undo()
    finally:
        os.unlink("foo.txt")

if __name__ == "__main__":
    main()

### OUTPUT ###
# renaming foo.txt to bar.txt
# renaming bar.txt to baz.txt
# renaming baz.txt to bar.txt
# renaming bar.txt to foo.txt
```


## 现实生活的例子

当我们去餐馆吃饭时，会叫服务员来点单。他们用来做记录的账单（通常是纸质的）就是命令模式的一个例子。在记录好订单后，服务员将其放入账单队列，厨师会照着单子去做。每个账单都是独立的，并且可用来执行许多不同命令，例如，一个命令对应一个将要烹任的菜品。下图展示了一个样例订单的时序图，经www.sourcemaking.com允许使用（请参考网页[t.cn/Rqr3tfQ]）。

## 软件的例子

PyQt是QT工具包的Python绑定。PyQt包含一个QAction类，将一个动作建模为一个命令。对每个动作都支持额外的可选信息，比如，描述、工具提示、快捷键和其他（请参考网页[t.cn/Rqr3VQU]）。

git-cola（请参考网页[t.cn/Rqr3IWK]）是使用Python语言编写的一个Git GUI，它使用命令模式来修改模型、变更一次提交、应用一个差异选择、签出，等等（请参考网页[t.cn/Rqr3JVz]）。

## 应用案例

许多开发人员以为撤销例子是命令模式的唯一应用案例。撤销操作确实是命令模式的杀手级特性，然而命令模式能做的实际上还有很多（请参考[GOF95，第265页]和网页[t.cn/R4a50r2]）。

* GUI按钮和菜单项：前面提过的PyQt例子使用命令模式来实现按钮和菜单项上的动作。
* 其他操作：除了撤销，命令模式可用于实现任何操作。其中一些例子包括剪切、复制、粘贴、重做和文本大写。
* 事务型行为和日志记录：事务型行为和H志记录对于为变更记录一份持久化H志是很重要的。操作系统用它来从系统崩溃中恢复，关系型数据库用它来实现事务，文件系统用它来实现快照，而安装程序（向导程序）用它来恢复取消的安装。
* 宏：在这里，宏是指一个动作序列，可在任意时间点按要求进行录制和执行。流行的编辑器（比如，Emacs和Vim）都支持宏。

## 实现

本节中，我们将使用命令模式实现最基本的文件操作工具。

* 创建一个文件，并随意写入一个字符串
* 读取一个文件的内容
* 重命名一个文件
* 删除一个文件

我们并不从头实现这些工具程序，因为Python在os模块中已提供了良好的实现。我们想做的是在已有实现之上添加一个额外的抽象层，这样可以当作命令来使用。这样，我们就能获得命令提供的所有优势。

下面的用例图展示了实现将支持的用户可执行操作。从展示的操作可以看出，重命名文件和创建文件支持撤销。删除一个文件和读取文件内容不支持撤销。对于文件删除操作实际上是可以实现撤销的，一种技术是使用一个特殊的垃圾箱/废物目录来存储所有被删除文件，这样在用户请求时可以恢复出来。这是所有现代桌面环境使用的默认行为，就留作练习吧。

每个命令都包括两个部分，初始化部分和执行部分。初始化部分由init()方法完成，包含该命令发挥作用所要求的所有信息（文件路径和将写入文件的内容等）。执行部分由execute()方法完成。在我们想真正地运行命令时才调用其execute()方法。该方法并不需要在命令初始化之后立即调用。

我们从重命名工具开始，使用RenameFile类来实现。init()方法接受源文件路径（path_src）和H标文件路径（path_dest）作为参数。如果文件路径未使用路径分隔符，则在当前H录下创建文件。使用路径分隔符的一个例子是传递字符串/tmp/file1作为path_src，字符串/home/user/file2作为path_dest。不使用路径的例子则是传递file1作为path_src，file2作为path_dest。


```python
class RenameFile:
    def __init__(self, path_src, path_dest):
        self.src, self.dest = path_src, path_dest
```

execute()方法使用os.rename()完成实际的重命名。verbose是一个全局标记，被激活时（默认是激活的），能向用户反馈执行的操作。如果你倾向于静默地执行命令，则可以取消激活状态。注意，虽然对于示例来说print()足够好了，但通常会使用更成熟更强大的方式，例如，志模块（请参考网页[t.cn/Rqr3SXw]）。


```python
def execute(self):
    if verbose:
        print("[renaming '{}' to '{}']".format(self.src, self.dest))
    os.rename(self.src, self.dest)
```

我们的重命名工具通过undo()方法支持撤销操作。在这里，撤销操作再次使用os.rename()将文件名恢复为原始值。


```python
def undo(self):
    if verbose:
        print("[renaming '{}' back to '{}']".format(self.dest, self.src))
    os.rename(self.dest, self.src)
```

文件删除功能实现为单个函数，而不是一个类。我想让你明臼并不一定要为想要添加的每个命令（之后会涉及更多）都创建一个新类。delete_file()函数接受一个字符串类型的文件路径，并使用os.remove()来删除它。


```python
def delete_file(path):
    if verbose:
        print("deleting file '{}'".format(path))
    os.remove(path)
```

再次回到使用类的方式。CreateFile类用于创建一个文件。init()函数接受熟悉的path参数和一个txt字符串，默认向文件写入hello world文本。通常来说，合理的默认行为是创建一个空文件，但因这个例子的需要，我决定向文件写个一个默认字符串。可以根据需要更改它。


```python
def __init__(self, path, txt='hello world\n'):
    self.path, self.txt = path, txt
```

execute()方法使用with语旬和open()来打开文件（mode='w'意味着写模式），并使用write()来写入txt字符串。


```python
def execute(self):
    if verbose:
        print("[Creating file '{}']".format(self.path))
    with open(self.path, mode='w', encoding='utf-8') as out_file:
        out_file.write(self.txt)
```

创建一个文件的撤销操作是删除它。因此，undo()简单地使用delete_file()来实现目的。


```python
def undo(self):
    delete_file(self.path)
```

最后一个工具让我们能够读取文件内容。ReadFile类的execute()方法再次使用with()语旬配合open()，这次是读模式，并且只是使用print()来输出文件内容。


```python
def execute(self):
    if verbose:
        print("[reading file '{}']".format(self.path))
    with open(self.path, mode='r', encoding='utf-8') as in_file:
        print(in_file.read(), end='')
```

main()函数使用这些工具类/方法。参数orig_name和new_name是待创建文件的原始名称以及重命名后的新名称。commands列表用于添加（并配置）所有我们之后想要执行的命令。注意，命令不会被执行，除非我们显式地调用每个命令的execute()。


```python
orig_name, new_name = 'file1', 'file2'
commands = []
for cmd in CreateFile(orig_name), ReadFile(orig_name), RenameFile(orig_name, new_name):
    commands.append(cmd)
[c.execute() for c in commands]
```

下一步是询问用户是否需要撤销执行过的命令。用户选择撤销命令或不撤销。如果选择撤销，则执行commands列表中所有命令的undo()。然而，由于并不是所有命令都支持撤销，因此在undo()方法不存在时产生的AttributeError异常要使用异常处理来捕获。如果你不喜欢对这种情况使用异常处理，可以通过添加一个布尔方法（例如，supports_undo() 或can_be_undone()）来显式地检测命令是否支持撤销操作。


```python
answer = input('reverse the executed commands? [y/n] ')

if answer not in 'yY':
    print("the result is {}".format(new_name))
    exit()

for c in reversed(commands):
    try:
        c.undo()
    except AttributeError as e:
        pass
```

以下是该示例的完整代码（command.py）。

```python
import os

verbose = True

class RenameFile:
    def __init__(self, path_src, path_dest):
        self.src, self.dest = path_src, path_dest

    def execute(self):
        if verbose:
            print("[renaming '{}' to '{}']".format(self.src, self.dest))
        os.rename(self.src, self.dest)

    def undo(self):
        if verbose:
            print("[renaming '{}' back to '{}']".format(self.dest, self.src))
        os.rename(self.dest, self.src)

class CreateFile:
    def __init__(self, path, txt='hello world\n'):
        self.path, self.txt = path, txt

    def execute(self):
        if verbose:
            print("[creating file '{}']".format(self.path))
        with open(self.path, mode='w', encoding='utf-8') as out_file:
            out_file.write(self.txt)

    def undo(self):
        delete_file(self.path)

class ReadFile:
    def __init__(self, path):
        self.path = path

    def execute(self):
        if verbose:
            print("[reading file '{}']".format(self.path))
        with open(self.path, mode='r', encoding='utf-8') as in_file:
            print(in_file.read(), end='')

def delete_file(path):
    if verbose:
        print("deleting file '{}".format(path))
    os.remove(path)

def main():
    orig_name, new_name = 'file1', 'file2'

    commands = []
    for cmd in CreateFile(orig_name), ReadFile(orig_name), RenameFile(orig_name, new_name):
        commands.append(cmd)

    [c.execute() for c in commands]

    answer = input('reverse the executed commands? [y/n] ')

    if answer not in 'yY':
        print("the result is {}".format(new_name))
        exit()

    for c in reversed(commands):
        try:
            c.undo()
        except AttributeError as e:
            pass

if __name__ == "__main__":
    main()
```

    [creating file 'file1']
    [reading file 'file1']
    hello world
    [renaming 'file1' to 'file2']
    reverse the executed commands? [y/n] y
    [renaming 'file2' back to 'file1']
    deleting file 'file1


这个命令模式的例子可以从多个方面进行改进。首先，这些工具程序都未遵从防御性编程风格（请参考网页[t.cn/Rqr3KHR]）。如果尝试重命名的文件并不存在，那么会发生什么？文件存在但不能对其重命名，因为没有正确的文件系统权限，此时会怎么样？所有工具都存在同样的问题。例如，如果尝试读取一个不存在的文件会发生什么？通过添加一些错误处理逻辑尝试改进这些工具程序。检查os模块方法的返回状态是否必要？

文件创建功能使用默认文件权限来创建文件，默认文件权限具体什么样由文件系统决定。例如，在POSIX系统中，这个权限为 -rw-rw-r--。你也许想通过向CreateFile传递恰当的参数让用户能够提供自己的权限设置。可以怎样实现呢？提示，一种方式是通过使用os.fdopen()。

现在，这里有一些东西需要你思考一下。之前我提到过，一个命令并不一定是一个类。文件删除功能就是那样实现的；仅有一个delete_file()函数。这种方式的优缺点是什么？这里有一个提示，把删除命令放入commands列表，像其余命令那样去执行，可能吗？我们知道在Python中函数是一等公民，因此我们可以执行某些操作，如以下代码所示（文件first-class.py）。


```python
orig_name = 'file1'
df = delete_file

commands = []
commands.append(df)

for c in commands:
    try:
        c.execute()
    except AttributeError as e:
        df(orig_name)

for c in reversed(commands):
    try:
        c.undo()
    except AttributeError as e:
        pass
```

虽然这个示例可以工作，但存在以下这些问题。

* 代码不统一。我们过于依赖异常处理，异常处理不是一个程序的常规流程。在这里，所有其他命令都有一个execute()方法，但删除命令没有execute()。
* 目前，文件删除功能还不支持撤销。如果我们最终决定要为其添加撤销支持，那会怎么样呢？通常，我们会为代表命令的那个类添加一个undo()方法。然而，这里的文件删除功能不是类。我们可以创建另一个函数来处理撤销操作，但创建一个类是更好的方式。

## 小结

本章中，我们学习了命令模式。使用这种设计模式，可以将一个操作（比如，复制/粘贴）封装为一个对象。这样能提供很多好处，如下所述。

* 我们可以在任何时候执行一个命令，而并不一定是在命令创建时。
* 执行一个命令的客户端代码并不需要知道命令的任何实现细节。
* 可以对命令进行分组，并按一定的顺序执行。

执行一个命令就像在餐馆里点单。每个顾客的订单都是一个独立的命令，分多个阶段，最终由厨师来执行。

许多GUI框架，包括PyQt，使用命令模式来建模动作，动作可被一个或多个事件触发，也可以自定义。然而，命令模式并不仅限于在框架中使用，普通应用（比如git-cola）也会因其而获益。

虽然至今命令模式最广为人知的特性是撤销操作，但它还有更多用处。一般而言，要在运行时按照用户意愿执行的任何操作都适合使用命令模式。命令模式也适用于组合多个命令。这有助于实现宏、多级撤销以及事务。一个事务应该：要么成功，这意味着事务中所有操作应该都成功（提交操作）；要么如果至少一个操作失败，则全部失败（回滚操作）。如果希望进一步使用命令模式，可以实现一个例子，涉及将多个命令组合成一个事务。

为演示命令模式，我们在Python的os模块之上实现了一些基本的文件操作工具。我们的工具程序支持撤销，并具有统一的接口，便于组合命令。

第12章将学习解释器模式，该模式可用于创建一种专注于某个特定领域的计算机语言。这种语言被称为领域特定语言（Domain Specific Language，DSL）。
