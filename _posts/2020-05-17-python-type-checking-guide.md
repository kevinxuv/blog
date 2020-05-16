---
layout: post
title: Python Type Checking (Guide)
tags: 
    - python
    - type system
    - type checking
---
> Python 作为一种动态语言，在 PEP484（3.5） 才支持 Type Hints，且类型申明是 optional 的，对于从静态语言（比如：Java，国内大学专业cs or se的教学语言也是以 C/C++、Java 为主）转过来的人来讲，变量以及函数的申明没有类型，会让他们很困惑（不知道方法返回什么，IDE 没有提示，不能自动生成代码，不能像 Java 那样点点操作就可以了），要他们理解 duck typing 就更难，因为这些都是动态语言领域的常识，这样的动态特性，也导致了 IDE 无法很好的支持 python type checking，就很难提示错误，对于小白来讲的确是一种困扰，本文翻译至 realpython 上的 [Python Type Checking (Guide)](https://realpython.com/python-type-checking/#duck-types-and-protocols) 这篇文章，为什么不自己写一篇：1. 个人对 python type system 的设计没有很好的掌握 2. 别人写的比我好太多了。 所以我就做一个搬运工吧。注：本文主要通过 Google translate 翻译，手工矫正，如果对翻译质量有问题可以具体指出。

在本指南中，您将了解 Python 类型检查。 传统上，类型由 Python 解释器以灵活但隐式的方式处理。 Python 的最新版本允许您指定显式类型提示（type hints），不同的工具可以使用这些提示来帮助您更有效地开发代码。

在这个指南里面，你将会学会如下：

- 类型注解和类型提示
- 在你的代码和其它代码添加静态类型
- 执行一个静态代码检查工具
- 在运行时强制类型

这是一本全面的指南，内容涉及很多领域。 如果您只是想快速了解类型提示（type hints）在 Python 中的工作方式，并查看是否将类型检查包含在代码中，则无需阅读所有内容。 [Hello Types](https://realpython.com/python-type-checking/#hello-types) 和 [Pros and Cons](https://realpython.com/python-type-checking/#pros-and-cons) 这两个部分将带您领略类型检查的工作原理以及有关何时使用的建议。

# 类型系统（Type Systems）

所有编程语言都包含某种类型的[类型系统](https://en.wikipedia.org/wiki/Type_system)，该系统形式化了可以使用的对象类别以及如何处理这些类别。 例如，类型系统可以定义数字类型，其中 42 是数字类型对象的一个示例。

## 动态类型（Dynamic Typing）

Python 是一种动态类型的语言。 这意味着 Python 解释器仅在代码运行时才进行类型检查，并且允许变量的类型在其生命周期内进行更改。 以下虚拟示例演示 Python 具有动态类型：

```python
>>> if False:
...     1 + "two"  # This line never runs, so no TypeError is raised
... else:
...     1 + 2
...
3

>>> 1 + "two"  # Now this is type checked, and a TypeError is raised
TypeError: unsupported operand type(s) for +: 'int' and 'str'
```

在第一个示例中，分支 `1 + "two"` 从不运行，因此永远不会经过类型检查。 第二个示例显示，当评估 `1 + "two"` 时，它会引发 TypeError，因为您无法在 Python 中使一个整数和字符串相加。

接下来，让我们看看变量是否可以更改类型：

```python
>>> thing = "Hello"
>>> type(thing)
<class 'str'>

>>> thing = 28.1
>>> type(thing)
<class 'float'>
```

`type()` 方法返回对象的类型。这些示例证实了变量的类型可以被允许更改，并且 Python 在更改时正确地推断了该类型。

## 动态类型（Static Typing）

动态类型化的反面是静态类型化。 在不运行程序的情况下执行静态类型检查。 在大多数静态类型的语言中，例如 C 和 Java，这是在编译程序时完成的。

对于静态类型，尽管可能存在将变量转换为其他类型的机制(强制转换)，但通常不允许变量更改类型。

让我们看一个来自静态类型语言的简单示例。考虑以下 Java 代码段：

```java
String thing;
thing = "hello";
```

第一行声明在编译时将变量名 thing 绑定到 String 类型。 这个名字永远不会反绑到另一种类型。 在第二行中，为事物分配了一个值。 永远不能为它分配一个非 String 对象的值。 例如，如果您稍后要说 thing = 28.1f，则编译器会由于类型不兼容而引发错误(静态类型检查也会提示错误)。

Python将始终是[动态类型的语言](https://www.python.org/dev/peps/pep-0484/#non-goals)。 但是，[PEP 484](https://www.python.org/dev/peps/pep-0484/) 引入了类型提示，这使得还可以对 Python 代码进行静态类型检查。

与大多数其他静态类型语言类型工作方式不同，类型提示本身不会导致 Python 强制执行类型。 顾名思义，类型提示只是建议类型。 还有其他工具，您将在本文后面看到，它们使用类型提示执行静态类型检查。

## 鸭子类型（Duck Typing）

在谈论 Python 时，经常使用的另一个术语是[鸭子类型(Duck Typing)](https://en.wikipedia.org/wiki/Duck_typing)。这个绰号来自：“如果它像鸭子一样行走，而像鸭子一样嘎嘎叫，那么它一定是鸭子”（或其[任何变体](https://en.wikipedia.org/wiki/Duck_test#History)）。

鸭子类型是动态类型相关的概念，其中对象的类型或类比其定义的方法重要。 使用鸭子类型您根本不需要检查类型。 而是检查给定方法或属性的存在。

例如，您可以在定义了 `.__len__()` 方法的任何 Python 对象上调用 `len()`：

```python
>>> class TheHobbit:
...     def __len__(self):
...         return 95022
...
>>> the_hobbit = TheHobbit()
>>> len(the_hobbit)
95022
```

请注意，对 `len()` 的调用给出了 `.__len__()` 方法的返回值。实际上，`len()` 的实现基本上等同于以下内容：

```python
def len(obj):
    return obj.__len__()
```

为了调用 `len(obj)`，对 `obj` 的唯一真正限制是它必须定义 `.__len__()` 方法。否则，对象的类型可以与 `str`，`list`，`dict` 或 `TheHobbit` 一样不同。

使用[结构子类型()](https://en.wikipedia.org/wiki/Structural_type_system) 对 Python 代码进行静态类型检查时，在某种程度上支持鸭子类型。稍后，您将详细了解鸭的分类。


# Hello Types

在本部分中，您将看到如何向函数添加类型提示。 以下函数通过添加适当的大写字母和装饰线将文本字符串转换为标题：

```python
def headline(text, align=True):
    if align:
        return f"{text.title()}\n{'-' * len(text)}"
    else:
        return f" {text.title()} ".center(50, "o")
```

默认情况下，该函数返回与下划线对齐的左侧标题。通过将 `align` 标志设置为 `False`，您可以使标题以 `o` 的环绕线为中心：

```python
>>> print(headline("python type checking"))
Python Type Checking
--------------------

>>> print(headline("python type checking", align=False))
oooooooooooooo Python Type Checking oooooooooooooo
```

现在该是我们的第一类提示了！要将有关类型的信息添加到函数，只需注解其参数并返回值，如下所示：

```python
def headline(text: str, align: bool = True) -> str:
    ...
```

`text` 参数：`str` 语法表示 `text` 参数应为 `str` 类型。同样，可选的 `align` 参数应具有默认值为 `True` 的 `bool` 类型。最后，`-> str` 表示法指定 `headline()` 将返回一个字符串。

在样式方面，[PEP 8](https://www.python.org/dev/peps/pep-0008/#other-recommendations) 建议以下内容：

- 对冒号使用正常规则，即冒号前没有空格，冒号后没有空格：`text: str`。
- 将参数注解与默认值组合时，请在 `=` 号周围使用空格：`align: bool = True`。
- 在 `->` 箭头周围使用空格：`def headline（...）-> str`。

像这样添加类型提示不会对运行时间产生影响：它们只是提示，不能单独执行。例如，如果我们为 `align` 参数（公认的错误命名）使用了错误的类型，则代码仍然可以运行，没有任何问题或警告：

```python
>>> print(headline("python type checking", align="left"))
Python Type Checking
--------------------
```

> 这似乎可行的原因是字符串 `left` 被[解释成 True](https://realpython.com/python-operators-expressions/#evaluation-of-non-boolean-values-in-boolean-context)。使用  `align = "center"` 不会产生预期的效果，因为 `center` 也是被解释成 `True`。

要捕获此类错误，可以使用静态类型检查器。也就是说，该工具可以检查代码的类型，而无需实际运行传统意义上的代码。

您可能已经在编辑器中内置了这种类型检查器。例如，PyCharm 立即向您发出警告：

![image](https://files.realpython.com/media/pycharm_type_error.76a49b9d4ff1.png)

不过，进行类型检查的最常用工具是 [Mypy](http://mypy-lang.org/)。稍后会给您将简短地讲解下 Mypy，您将了解更多有关 Mypy 的工作原理。

如果您的系统上还没有Mypy，则可以使用pip安装它：

```shell
$ pip install mypy
```

将以下代码放在一个名为 `headlines.py` 的文件中：

```python
# headlines.py
def headline(text: str, align: bool = True) -> str:
    if align:
        return f"{text.title()}\n{'-' * len(text)}"
    else:
        return f" {text.title()} ".center(50, "o")

print(headline("python type checking"))
print(headline("use mypy", align="center"))
```

这基本上与您之前看到的代码相同：`headline()` 的定义和两个使用它的示例。

现在在以下代码上运行Mypy：

```shell
$ mypy headlines.py
headlines.py:10: error: Argument "align" to "headline" has incompatible
                        type "str"; expected "bool"
```

根据类型提示，Mypy 可以告诉我们在第10行上使用了错误的类型。

要解决代码中的问题，您应该更改传入的align参数的值。您还可以将align标志重命名为不太混乱的名称：

```python
# headlines.py
def headline(text: str, centered: bool = False) -> str:
    if not centered:
        return f"{text.title()}\n{'-' * len(text)}"
    else:
        return f" {text.title()} ".center(50, "o")

print(headline("python type checking"))
print(headline("use mypy", centered=True))
```

在这里，您已将 `align` 更改为居中，并在调用 `headline()` 时正确使用了布尔值来居中。现在，代码通过了 Mypy：

```shell
$ mypy headlines.py
$ 
```

Mypy没有输出意味着没有检测到类型错误。此外，在运行代码时，您将看到预期的输出：

```shell
$ python headlines.py
Python Type Checking
--------------------
oooooooooooooooooooo Use Mypy oooooooooooooooooooo
```

第一个标题向左对齐，第二个标题居中。

# 利与弊（Pros and Cons）

上一节向您介绍了 Python 中的类型检查的外观。您还看到了一个向代码添加类型的优点之一的示例：类型提示有助于**捕获某些错误**。其他优点包括：

- 输入提示可帮助丰富**你的代码文档**。 传统上，如果要记录函数参数的预期类型，则应使用 [docstrings](https://realpython.com/documenting-python-code/)。 此方法有效，但是由于没有文档字符串标准（尽管有 [PEP 257](https://www.python.org/dev/peps/pep-0257/)，它们不能轻易用于自动检查）。
- 类型提示可**改善 IDE 和 linter**。 它们使静态推理代码变得容易得多。 反过来，这使 IDE 可以提供更好的代码完成和类似的功能。 通过类型注解，PyCharm 知道文本是字符串，并可以基于此给出具体建议：

![2](https://files.realpython.com/media/pycharm_code_completion.82857c2750f6.png)

- 类型提示可帮助您**构建和维护更简洁的代码架构**。 类型提示的行为迫使您考虑程序中的类型。 尽管 Python 的动态特性是它的一大优势，但是意识到依赖鸭子类型，重载方法或多种返回类型是一件好事。

当然，静态类型检查并非全部都是桃子和奶油。 您还应考虑以下缺点：

- 类型提示需要开发人员花费时间和精力进行添加。即使花费较少的调试时间可能会有所回报，但是您将花费更多的时间输入代码。
- 类型提示在现代 Python 中效果最好。 注解是在 Python 3.0 中引入的，并且可以在 Python 2.7 中使用类型注解。 尽管如此，诸如变量注解和类型提示的延迟评估等改进仍意味着您将拥有使用 Python 3.6 甚至 Python 3.7 进行类型检查的更好体验。
- 类型提示会在启动时间上带来一些损失。如果需要使用键入模块，则导入时间可能很长，尤其是在短脚本中。

因此，您应该在自己的代码中使用静态类型检查吗？ 好吧，这不是一个全有或全无的问题。 幸运的是，Python 支持渐进式输入的概念。 这意味着您可以逐步将类型引入代码中。 没有类型提示的代码将被静态类型检查器忽略。 因此，您可以开始向关键组件添加类型，并继续添加，只要它能为您增加价值即可。

查看上面的利弊清单，您会发现添加类型对您正在运行的程序或程序用户没有影响。 类型检查旨在使您作为开发人员的生活更美好，更便捷。

关于是否向项目中添加类型的一些经验法则是：

- 如果您刚刚开始学习 Python，则可以放心使用类型提示，直到您有更多经验为止。
- 在[简短的一次性脚本](https://www.youtube.com/watch?v=Jd8ulMb6_ls)中，类型提示几乎没有任何价值。
- 在其他人会使用的库中，尤其是在 `PyPI` 上发布的库中，类型提示会增加很多价值。 使用您的库的其他代码需要这些类型提示才能正确进行类型检查。 有关使用类型提示的项目的示例，请参见 [cursive_re](https://github.com/Bogdanp/cursive_re/blob/master/cursive_re/exprs.py)，[black](https://github.com/ambv/black/blob/master/black.py)，我们自己的 [Real Python Reader](https://github.com/realpython/reader/blob/master/reader/feed.py) 和 [Mypy](https://github.com/python/mypy/blob/master/mypy/build.py) 本身。
- 在较大的项目中，类型提示可帮助您了解类型在代码中的流动方式，因此强烈建议使用。在与他人合作的项目中更是如此。

BernátGábor 在他的出色文章[《Python中的类型提示的状态》](https://www.bernat.tech/the-state-of-type-hints-in-python/) 中建议“只要值得编写单元测试，都应使用类型提示。” 实际上，类型提示在代码中的作用与测试相似：它们可以帮助您作为开发人员编写更好的代码。

希望您现在对Python中类型检查的工作方式以及是否要在自己的项目中使用它有所了解。

在本指南的其余部分，我们将详细介绍 Python 类型系统，包括如何运行静态类型检查器（特别关注Mypy），如何在不带类型提示的库检查代码，以及在运行时使用注解。

# 注解（Annotations）

注解是在 [Python 3.0](https://www.python.org/dev/peps/pep-3107/) 中引入的，最初没有任何特定目的。它们只是将任意表达式与函数参数和返回值关联的一种方式。

多年后，[PEP 484](https://www.python.org/dev/peps/pep-0484/) 根据 Jukka Lehtosalo 在其博士学位上所做的工作，定义了如何在您的Python代码中添加类型提示：项目-Mypy。 添加类型提示的主要方法是使用注解。 随着类型检查变得越来越普遍，这也意味着注解应主要保留给类型提示。

## 函数注解

对于函数，您可以注解参数和返回值。这样做如下：

```python
def func(arg: arg_type, optarg: arg_type = default) -> return_type:
    ...
```

对于参数，语法为 `arguments: 注解`，而返回类型使用 `->` 注解进行注解。请注意，注解必须是有效的Python表达式。

下面的简单示例将注解添加到计算圆的周长的函数中：

```python
import math
def circumference(radius: float) -> float:
    return 2 * math.pi * radius
```

运行代码时，您也可以检查注解。它们存储在函数的特殊 `.__annotations__` 属性中：

```python
>>> circumference(1.23)
7.728317927830891

>>> circumference.__annotations__
{'radius': <class 'float'>, 'return': <class 'float'>}
```

有时，您可能会对 Mypy 如何解释您的类型提示感到困惑。 对于这些情况，有特殊的 Mypy 表达式：`reveal_type()` 和`reveal_locals()`。 您可以在运行 Mypy 之前将它们添加到代码中，然后 Mypy 将尽职地报告其推断出的类型。 举例来说，保存以下代码以 `reveal.py`：

```python
# reveal.py

import math
reveal_type(math.pi)

radius = 1
circumference = 2 * math.pi * radius
reveal_locals()
```

接下来，通过 Mypy 运行以下代码：

```shell
$ mypy reveal.py
reveal.py:4: error: Revealed type is 'builtins.float'

reveal.py:8: error: Revealed local types are:
reveal.py:8: error: circumference: builtins.float
reveal.py:8: error: radius: builtins.int
```

即使没有任何注解，Mypy 仍可以正确推断出内置 math.pi 的类型，以及我们的局部变量“半径”和“周长”。

> 注意：显示表达式仅作为帮助您添加类型和调试类型提示的工具。 如果您尝试以 Python 脚本的形式运行 reveal.py 文件，它将由于 NameError 崩溃，因为 reveal_type() 不是 Python 解释器已知的函数。

如果 Mypy 提示 `Name 'reveal_locals'is not defined`，则可能需要更新 Mypy 安装。 Mypy 0.610 及更高版本中提供了 reveal_locals() 表达式。

## 变量注解

在上一节的 circumference() 的定义中，您仅注解了参数和返回值。您没有在函数体内添加任何注解。通常这足够了。

但是，有时类型检查器在确定变量类型时也需要帮助。变量注解在 [PEP 526](https://www.python.org/dev/peps/pep-0526/) 中定义，并在 Python 3.6 中引入。其语法与函数参数注解的语法相同：

```python
pi: float = 3.142

def circumference(radius: float) -> float:
    return 2 * pi * radius
```

变量 pi 已使用 float 类型提示进行注解。

> 注意：静态类型检查器不仅仅能够确定 3.142 是浮点数，因此在此示例中，pi 的注解不是必需的。当您了解有关 Python 类型系统的更多信息时，您会看到更多相关的变量注解示例。

变量的注解存储在模块级别的 `__annotations__` 字典中：

```python
>>> circumference(1)
6.284

>>> __annotations__
{'pi': <class 'float'>}
```

您可以在不给变量值的情况下对其进行注解。这会将注解添加到 `__annotations__` 字典中，而变量仍未定义：

```python
>>> nothing: str
>>> nothing
NameError: name 'nothing' is not defined

>>> __annotations__
{'nothing': <class 'str'>}
```

由于没有为任何值分配任何值，因此尚未定义名称 nothing。

## 类型注释(Type Comments)

如前所述，注解是在 Python 3 中引入的，并且尚未向后移植到 Python2。这意味着，如果要编写需要支持旧版 Python 的代码，则不能使用注解。

相反，您可以使用类型注释。这些是特殊格式的注释，可用于添加与旧代码兼容的类型提示。要将类型注释添加到函数中，请执行以下操作：

```python
import math

def circumference(radius):
    # type: (float) -> float
    return 2 * math.pi * radius
```

类型注释只是注释，因此可以在任何版本的Python中使用。

类型注释由类型检查器直接处理，因此这些类型在 `__annotations__` 词典中不可用：

```python
>>> circumference.__annotations__
{}
```

类型注释必须以类型：文字开头，并且与函数定义在同一行或下一行。如果要用几个参数注释一个函数，请编写每个用逗号分隔的类型：

```python
def headline(text, width=80, fill_char="-"):
    # type: (str, int, str) -> str
    return f" {text.title()} ".center(width, fill_char)

print(headline("type comments work", width=40))
```

您还可以将每个参数写在带有自己注释的单独一行上：

```python
# headlines.py

def headline(
    text,           # type: str
    width=80,       # type: int
    fill_char="-",  # type: str
):                  # type: (...) -> str
    return f" {text.title()} ".center(width, fill_char)

print(headline("type comments work", width=40))
```

通过 Python 和 Mypy 运行示例：

```python
$  python headlines.py
---------- Type Comments Work ----------

$ mypy headline.py
$
```

如果您遇到错误，例如，如果您碰巧在第 10 行上调用了 `headline()` 并且 `width = "full"`，Mypy 将告诉您：

```python
$ mypy headline.py
headline.py:10: error: Argument "width" to "headline" has incompatible
                       type "str"; expected "int"
```

您还可以将类型注释添加到变量。类似于将类型注释添加到参数的方法：

```python
pi = 3.142  # type: float
```

在此示例中，将 pi 作为浮点变量进行类型检查。

## 那么，选择类型注解（Type Annotaions）或类型注释（Type Comments）呢?

在将类型提示添加到自己的代码中时，应该使用注解（annotations）还是键入注释（comments）？简而言之：如果可以，请使用注解（annotations），如果需要，请使用类型注释（comments）。

注解（Annotations） 提供了更简洁的语法，使类型信息更接近您的代码。它们也是书写类型提示的[官方推荐方式](https://www.python.org/dev/peps/pep-0484/)，并且将来会得到进一步开发和适当维护。

类型注释（Type Comments）更冗长，并且可能与代码中的其他类型的注释（例如 [linter 指令](https://realpython.com/python-code-quality/)）冲突。但是，它们可以在不支持注解（annotations）的代码库中使用。

还有第三个隐藏的选项：[存根文件（stub files）](https://github.com/python/mypy/wiki/Creating-Stubs-For-Python-Modules)。稍后，当我们讨论向第三方库添加类型时，您将学到这些。

# 玩转 Python 类型第二部分

到目前为止，您仅在类型提示中使用了 str，float 和 bool 等基本类型。 Python 类型系统非常强大，并且支持许多更复杂的类型。 这是必需的，因为它需要能够合理地建模 Python 的动态 duck typing 特性。

在本节中，您将学习有关这种类型系统的更多信息，同时实现一个简单的纸牌游戏。您将看到如何指定：

- 序列和映射的类型，例如元组，列表和字典
- 类型别名，使代码更易于阅读
- 函数和方法不返回任何东西
- 对象可能是 `any` 类型

## 示例：一副纸牌

以下示例显示了常规（法语）纸牌的实现：

```python
# game.py

import random

SUITS = "♠ ♡ ♢ ♣".split()
RANKS = "2 3 4 5 6 7 8 9 10 J Q K A".split()

def create_deck(shuffle=False):
    """Create a new deck of 52 cards"""
    deck = [(s, r) for r in RANKS for s in SUITS]
    if shuffle:
        random.shuffle(deck)
    return deck

def deal_hands(deck):
    """Deal the cards in the deck into four hands"""
    return (deck[0::4], deck[1::4], deck[2::4], deck[3::4])

def play():
    """Play a 4-player card game"""
    deck = create_deck(shuffle=True)
    names = "P1 P2 P3 P4".split()
    hands = {n: h for n, h in zip(names, deal_hands(deck))}

    for name, cards in hands.items():
        card_str = " ".join(f"{s}{r}" for (s, r) in cards)
        print(f"{name}: {card_str}")

if __name__ == "__main__":
    play()
```

每张卡都由表示 suit 和 rank 的字符串元组表示。 牌组表示为卡片列表。 `create_deck()` 创建一个由 52 张扑克牌组成的常规牌组，并有选择地随机播放纸牌。 `deal_hands()` 将一副纸牌发给四个玩家。

```shell
$ python game.py
P4: ♣9 ♢9 ♡2 ♢7 ♡7 ♣A ♠6 ♡K ♡5 ♢6 ♢3 ♣3 ♣Q
P1: ♡A ♠2 ♠10 ♢J ♣10 ♣4 ♠5 ♡Q ♢5 ♣6 ♠A ♣5 ♢4
P2: ♢2 ♠7 ♡8 ♢K ♠3 ♡3 ♣K ♠J ♢A ♣7 ♡6 ♡10 ♠K
P3: ♣2 ♣8 ♠8 ♣J ♢Q ♡9 ♡J ♠4 ♢8 ♢10 ♠9 ♡4 ♠Q
```

随着我们的前进，您将看到如何将此示例扩展为更有趣的游戏。

## 序列和映射（Sequences and Mappings）

让我们在纸牌游戏中添加类型提示。 换句话说，让我们注解一下函数 `create_deck()`，`deal_hands()` 和 `play()`。 第一个挑战是您需要注释复合类型，例如用于表示纸牌的列表和用于表示纸牌本身的元组。

使用 str，float 和 bool 等简单类型，添加类型提示与使用类型本身一样容易：

```python
>>> name: str = "Guido"
>>> pi: float = 3.142
>>> centered: bool = False
```

对于复合类型，您可以执行以下操作：

```python
>>> names: list = ["Guido", "Jukka", "Ivan"]
>>> version: tuple = (3, 7, 1)
>>> options: dict = {"centered": False, "capitalize": True}
```

但是，这并不能真正说明全部情况。 name[2]，version[0] 和 option["centered"] 的类型是什么？ 在这种具体情况下，您可以看到它们分别是 str，int 和 bool。 但是，类型提示本身对此不提供任何信息。

相反，您应该使用在键入模块中定义的特殊类型。这些类型添加了用于指定复合类型的元素类型的语法。您可以编写以下内容：

```python
>>> from typing import Dict, List, Tuple

>>> names: List[str] = ["Guido", "Jukka", "Ivan"]
>>> version: Tuple[int, int, int] = (3, 7, 1)
>>> options: Dict[str, bool] = {"centered": False, "capitalize": True}
```

请注意，以下每种类型均以大写字母开头，并且都使用方括号定义项目类型：

- **name** 是一个字符串列表
- **version** 是由3个整数组成的3元组
- **options** 是将字符串映射为布尔值的字典

typing 模块包含更多复合类型，包括 Counter，Deque，FrozenSet，NamedTuple 和 Set。此外，该模块还包含其他种类的类型，您将在后面的部分中看到。

让我们回到纸牌游戏。卡片由两个字符串的元组表示。您可以将其写为 `Tuple [str, str]`，因此卡片组的类型为 `List [Tuple [str, str]]`。因此，您可以如下注解 `create_deck()`：

```python
def create_deck(shuffle: bool = False) -> List[Tuple[str, str]]:
    """Create a new deck of 52 cards"""
    deck = [(s, r) for r in RANKS for s in SUITS]
    if shuffle:
        random.shuffle(deck)
    return deck
```

除了返回值外，您还向可选的 shuffle 参数中添加了 bool 类型。

> 注意：元组和列表的注释不同。
> 元组是一个不变的序列，通常由固定数量的可能不同类型的元素组成。例如，我们将卡表示为 suit 和 rank 的元组。通常，您将n个元组写入Tuple [t_1, t_2, ..., t_n]。
> 列表是可变序列，通常由未知数量的相同类型的元素组成，例如卡片列表。无论列表中有多少个元素，注释中都只有一种类型：List [t]。

在许多情况下，您的函数会期望某种[sequence](https://docs.python.org/glossary.html#term-sequence)，而实际上并不关心它是列表还是元组。在这些情况下，在对函数参数进行注释时应使用`type.Sequence`：

```python
from typing import List, Sequence

def square(elems: Sequence[float]) -> List[float]:
    return [x**2 for x in elems]
```

使用序列是使用 duck typing 的示例。序列是支持 `len()` 和 `.__getitem__()` 的任何事物，而与它的实际类型无关。

## 类型别名（Type Aliases）

当使用嵌套类型（如纸牌组）时，类型提示可能会变得非常倾斜。您可能需要先凝视 List [Tuple [str, str]]，然后才能确定它与我们对一副纸牌的表示相匹配。

```python
def deal_hands(
    deck: List[Tuple[str, str]]
) -> Tuple[
    List[Tuple[str, str]],
    List[Tuple[str, str]],
    List[Tuple[str, str]],
    List[Tuple[str, str]],
]:
    """Deal the cards in the deck into four hands"""
    return (deck[0::4], deck[1::4], deck[2::4], deck[3::4])
```

这太可怕了！

回想一下，类型注解是常规的 Python 表达式。这意味着您可以通过将它们分配给新变量来定义自己的类型别名。例如，您可以创建Card 和 Deck 类型别名：

```python
from typing import List, Tuple

Card = Tuple[str, str]
Deck = List[Card]
```

现在，可以在类型提示中或在新类型别名的定义中使用 Card，例如上面示例中的 Deck。

```python
def deal_hands(deck: Deck) -> Tuple[Deck, Deck, Deck, Deck]:
    """Deal the cards in the deck into four hands"""
    return (deck[0::4], deck[1::4], deck[2::4], deck[3::4])
```

类型别名非常适合使代码及其意图更清晰。同时，可以检查这些别名以查看其代表的含义：

```python
>>> from typing import List, Tuple
>>> Card = Tuple[str, str]
>>> Deck = List[Card]

>>> Deck
typing.List[typing.Tuple[str, str]]
```

请注意，在打印 Deck 时，它表示这是2元组字符串列表的别名。

## 没有返回值的函数

您可能知道没有显式返回的函数仍然返回 None：

```python
>>> def play(player_name):
...     print(f"{player_name} plays")
...

>>> ret_val = play("Jacob")
Jacob plays

>>> print(ret_val)
None
```

尽管此类函数从技术上讲会返回某些内容，但该返回值没有用。您还应该通过使用 None 作为返回类型来添加说出尽可能多的类型提示：

```python
# play.py

def play(player_name: str) -> None:
    print(f"{player_name} plays")

ret_val = play("Filip")
```

注解可帮助您捕获那些试图使用无意义的返回值的细微错误。 Mypy 将给您一个有用的警告：

```shell
$ mypy play.py
play.py:6: error: "play" does not return a value
```

请注意，明确声明函数不返回任何内容与不添加有关返回值的类型提示不同：

```python
# play.py

def play(player_name: str):
    print(f"{player_name} plays")

ret_val = play("Henrik")
```

在后一种情况下，Mypy没有有关返回值的信息，因此不会生成任何警告：

```shell
$ mypy play.py
$
```

作为更特殊的情况，请注意，您还可以注解那些永远不会正常返回的函数。这是使用 `NoReturn` 完成的：

```python
from typing import NoReturn

def black_hole() -> NoReturn:
    raise Exception("There is no going back ...")
```

由于 `black_hole()` 总是引发异常，因此它将永远不会正确返回。

## 示例：打一些牌

回到我们的纸牌游戏示例。 在游戏的第二版中，我们像以前一样向每位玩家分发一手纸牌。 然后选择开始玩家，然后玩家轮流玩自己的纸牌。 游戏中实际上没有任何规则，因此玩家只能玩随机纸牌：

```python
# game.py

import random
from typing import List, Tuple

SUITS = "♠ ♡ ♢ ♣".split()
RANKS = "2 3 4 5 6 7 8 9 10 J Q K A".split()

Card = Tuple[str, str]
Deck = List[Card]

def create_deck(shuffle: bool = False) -> Deck:
    """Create a new deck of 52 cards"""
    deck = [(s, r) for r in RANKS for s in SUITS]
    if shuffle:
        random.shuffle(deck)
    return deck

def deal_hands(deck: Deck) -> Tuple[Deck, Deck, Deck, Deck]:
    """Deal the cards in the deck into four hands"""
    return (deck[0::4], deck[1::4], deck[2::4], deck[3::4])

def choose(items):
    """Choose and return a random item"""
    return random.choice(items)

def player_order(names, start=None):
    """Rotate player order so that start goes first"""
    if start is None:
        start = choose(names)
    start_idx = names.index(start)
    return names[start_idx:] + names[:start_idx]

def play() -> None:
    """Play a 4-player card game"""
    deck = create_deck(shuffle=True)
    names = "P1 P2 P3 P4".split()
    hands = {n: h for n, h in zip(names, deal_hands(deck))}
    start_player = choose(names)
    turn_order = player_order(names, start=start_player)

    # Randomly play cards from each player's hand until empty
    while hands[start_player]:
        for name in turn_order:
            card = choose(hands[name])
            hands[name].remove(card)
            print(f"{name}: {card[0] + card[1]:<3}  ", end="")
        print()

if __name__ == "__main__":
    play()
```

请注意，除了更改 `play()` 之外，我们还添加了两个需要类型提示的新函数：`choice()` 和 `player_order()`。在讨论我们如何向其中添加类型提示之前，这是运行游戏的示例输出：

```shell
$ python game.py
P3: ♢10  P4: ♣4   P1: ♡8   P2: ♡Q
P3: ♣8   P4: ♠6   P1: ♠5   P2: ♡K
P3: ♢9   P4: ♡J   P1: ♣A   P2: ♡A
P3: ♠Q   P4: ♠3   P1: ♠7   P2: ♠A
P3: ♡4   P4: ♡6   P1: ♣2   P2: ♠K
P3: ♣K   P4: ♣7   P1: ♡7   P2: ♠2
P3: ♣10  P4: ♠4   P1: ♢5   P2: ♡3
P3: ♣Q   P4: ♢K   P1: ♣J   P2: ♡9
P3: ♢2   P4: ♢4   P1: ♠9   P2: ♠10
P3: ♢A   P4: ♡5   P1: ♠J   P2: ♢Q
P3: ♠8   P4: ♢7   P1: ♢3   P2: ♢J
P3: ♣3   P4: ♡10  P1: ♣9   P2: ♡2
P3: ♢6   P4: ♣6   P1: ♣5   P2: ♢8
```

在此示例中，玩家 P3 被随机选择为起始玩家。反过来，每个玩家都玩牌：首先是 P3，然后是 P4，然后是 P1，最后是 P2。玩家只要手中有手牌，就会继续打牌。

## Any 类型

`choice()` 适用于名称列表和卡列表（以及与此相关的任何其他顺序）。为此添加类型提示的一种方法是：

```python
import random
from typing import Any, Sequence

def choose(items: Sequence[Any]) -> Any:
    return random.choice(items)
```

这或多或少意味着它的含义：`items` 是一个可以包含任何类型的项目的序列，而 `choice()` 将返回一个任何类型的此类项目。不幸的是，这不是那么有用。考虑以下示例：

```python
# choose.py

import random
from typing import Any, Sequence

def choose(items: Sequence[Any]) -> Any:
    return random.choice(items)

names = ["Guido", "Jukka", "Ivan"]
reveal_type(names)

name = choose(names)
reveal_type(name)
```

尽管 Mypy 可以正确推断出名称是字符串列表，但是由于使用 Any 类型，因此在调用 `select()` 之后，该信息会丢失：

```python
$ mypy choose.py
choose.py:10: error: Revealed type is 'builtins.list[builtins.str*]'
choose.py:13: error: Revealed type is 'Any'
```

您很快就会看到更好的方法。首先，让我们从理论上更深入地了解 Python 类型系统，以及 Any 扮演的特殊角色。

# 类型理论

本教程主要是一本实用指南，我们仅会介绍支持 Python 类型提示的理论表面。 有关更多详细信息，[PEP 483](https://www.python.org/dev/peps/pep-0483/) 是一个很好的起点。 如果您想返回到实际示例，请随时跳到下一部分。

## 子类型（Subtypes）

一个重要的概念是子类型。正式地说，如果满足以下两个条件，则类型 T 是 U 的子类型：

- 来自T的每个值也属于U类型的值的集合。
- U 型的每个功能也都属于 T 型的功能。

这两个条件保证即使类型 T 与 U 不同，类型 T 的变量也总是可以假装为 U。

举一个具体的例子，考虑 T = bool和 U = int。bool 类型仅采用两个值。通常，它们分别表示为 True 和 False，但这些名称分别只是整数值 1 和 0 的别名：

```python
>>> int(False)
0

>>> int(True)
1

>>> True + True
2

>>> issubclass(bool, int)
True
```

由于 0 和 1 都是整数，因此第一个条件成立。在上面可以看到布尔值可以加在一起，但是它们也可以做整数可以做的其他事情。这是上面的第二个条件。换句话说，bool 是 int 的子类型。

子类型的重要性在于，子类型总是可以假装为其父类型。例如，以下代码类型检查是否正确：

```python
def double(number: int) -> int:
    return number * 2

print(double(True))  # Passing in bool instead of int
```

subtypes 与 subclasses 有些相关。 实际上，所有 subclasses 都对应于 subtypes，并且 bool 是 int 的 subtype，因为 bool 是 int 的 subclass。 但是，也有一些子 subtype 与 subclass 不对应。 例如，int 是 float 的 subtype，但int 不是 float 的 subclass。

## 协变，逆变和不变（Covariant, Contravariant, and Invariant）

在复合类型中使用子类型时会发生什么？ 例如，`Tuple[bool]` 是 `Tuple[int]` 的子类型吗？ 答案取决于复合类型，以及该类型是协变，逆变还是不变。 这可以快速获得技术，因此让我们举几个例子：

- 元组是协变的。这意味着它将保留其项类型的类型层次结构：`Tuple[bool]` 是 `Tuple[int]` 的子类型，因为 bool 是 int 的子类型。
- 列表是不变的。 不变类型不能保证子类型。 虽然 `List[bool]` 的所有值都是 `List[int]` 的值，但是可以将int附加到`List[int]` 而不是 `List[bool]`。 换句话说，子类型的第二个条件不成立，并且 `List[bool]` 不是 `List[int]` 的子类型。
- Callable 在其参数上是相反的。 这意味着它将反转类型层次结构。 您将在稍后看到 Callable 的工作方式，但现在将`Callable[[T], ...]` 视为一个函数，其唯一参数为 T 类型。`Callable[[int], ...]` 的示例为 上面定义的 `double()` 函数。 变数意味着如果期望对布尔操作的函数，那么对整数进行操作的函数将是可以接受的。

通常，您不需要保持直白的表达。但是，您应该意识到，子类型和复合类型可能不是简单直观的。

## 渐进式输入和一致类型

之前我们提到 Python 支持[gradual typing](http://wphomes.soic.indiana.edu/jsiek/what-is-gradual-typing/)，您可以在其中逐步将类型提示添加到 Python 代码中。本质上，通过 Any 类型可以进行渐变键入。

无论如何，Any 都位于子类型的类型层次结构的顶部和底部。 任何类型的行为就好像它是 Any 的子类型，而 Any 行为就好像它是任何其他类型的子类型。 从上面的子类型的定义来看，这实际上是不可能的。 相反，我们谈论下 **consistent types**。

如果T是U的子类型，或者 T 或 U 是 Any，则类型 T 与类型 U 一致。

类型检查器仅抱怨类型不一致。因此，总的来说，您将永远不会看到Any类型引起的类型错误。

这意味着您可以使用 Any 显式地退回到动态类型，描述太复杂而无法在 Python 类型系统中描述的类型，或描述复合类型的项。 例如，带有字符串键的字典可以采用任何类型，因为其值可以标注 `Dict[str, Any]`。

不过请记住，如果您使用Any，则静态类型检查器实际上将不会进行任何类型的任何检查。

# 玩转 Python 类型，第2部分

让我们回到实际的例子。回想一下您试图注释一般的 `select()` 函数：

```python
import random
from typing import Any, Sequence

def choose(items: Sequence[Any]) -> Any:
    return random.choice(items)
```

使用 Any 的问题在于，您不必要地丢失了类型信息。您知道，如果将字符串列表传递给 `choice()`，它将返回一个字符串。在下面，您将看到如何使用类型变量来表达这一点，以及如何使用它：

- Duck Types and Protocols
- 无默认值的参数
- 类方法
- 你的类的类型
- 可变数量的参数

## 类型变量（Type Variables）

类型变量是一种特殊的变量，可以根据情况采用任何类型。

让我们创建一个类型变量，该变量将有效地封装 `choice()` 的行为：

```python
# choose.py

import random
from typing import Sequence, TypeVar

Choosable = TypeVar("Chooseable")

def choose(items: Sequence[Choosable]) -> Choosable:
    return random.choice(items)

names = ["Guido", "Jukka", "Ivan"]
reveal_type(names)

name = choose(names)
reveal_type(name)
```

必须使用 typing 模块中的 TypeVar 定义类型变量。当使用类型变量时，类型变量将覆盖所有可能的类型，并采用最具体的类型。在示例中，名称现在是一个 str：

```shell
choose.py:12: error: Revealed type is 'builtins.list[builtins.str*]'
choose.py:15: error: Revealed type is 'builtins.str*'
```

考虑其他一些示例：

```python
# choose_examples.py

from choose import choose

reveal_type(choose(["Guido", "Jukka", "Ivan"]))
reveal_type(choose([1, 2, 3]))
reveal_type(choose([True, 42, 3.14]))
reveal_type(choose(["Python", 3, 7])
```

前两个示例应该具有 str 和 int 类型，但是后两个示例呢？各个列表项具有不同的类型，在这种情况下，Choosable type 变量会尽最大努力适应以下情况：

```shell
$ mypy choose_examples.py
choose_examples.py:5: error: Revealed type is 'builtins.str*'
choose_examples.py:6: error: Revealed type is 'builtins.int*'
choose_examples.py:7: error: Revealed type is 'builtins.float*'
choose_examples.py:8: error: Revealed type is 'builtins.object*'
```

您已经看到 bool 是 int 的子类型，再次是 float 的子类型。 因此，在第三个示例中，可以保证 `select()` 的返回值可以看作是 float。 在最后一个示例中，str 和 int 之间没有子类型关系，因此关于返回值的最好说法就是它是一个对象。

您可以通过列出可接受的类型来约束类型变量：

```python
# choose.py

import random
from typing import Sequence, TypeVar

Choosable = TypeVar("Choosable", str, float)

def choose(items: Sequence[Choosable]) -> Choosable:
    return random.choice(items)

reveal_type(choose(["Guido", "Jukka", "Ivan"]))
reveal_type(choose([1, 2, 3]))
reveal_type(choose([True, 42, 3.14]))
reveal_type(choose(["Python", 3, 7]))
```

现在 Choosable 只能是 str 或 float，而 Mypy 将注意到最后一个示例是一个错误：

```shell
$ mypy choose.py
choose.py:11: error: Revealed type is 'builtins.str*'
choose.py:12: error: Revealed type is 'builtins.float*'
choose.py:13: error: Revealed type is 'builtins.float*'
choose.py:14: error: Revealed type is 'builtins.object*'
choose.py:14: error: Value of type variable "Choosable" of "choose"
                     cannot be "object"
```

还要注意，在第二个示例中，即使输入列表仅包含 int 对象，该类型也被认为是 float 类型。这是因为 Choosable 仅限于字符串和浮点数，而 int 是 float的子类型。

在我们的纸牌游戏中，我们希望限制将 `select()` 用于 str 和 Card：

```python
Choosable = TypeVar("Choosable", str, Card)

def choose(items: Sequence[Choosable]) -> Choosable:
    ...
```

我们简要提到了 Sequence 既代表列表又代表元组。如前所述，序列可以被认为是鸭子类型，因为它可以是实现了 `.__len__()` 和 `.__getitem__()` 的任何对象。

# Duck Types and Protocols

回顾一下引言中的以下示例：

```python
def len(obj):
    return obj.__len__()
```

`len()` 可以返回已实现 `.__len__()` 方法的任何对象的长度。我们如何向 `len()` 尤其是 obj 参数添加类型提示？

答案隐藏在学术用语 [structural subtyping](https://en.wikipedia.org/wiki/Structural_type_system) 的背后。对类型系统进行分类的一种方法是通过 **nominal** 还是 **structural**：

- 在 **nominal** 类型系统中，类型之间的比较基于名称和声明。 Python 类型系统通常是 nominal 上的，由于其子类型关系，可以使用 int 代替 float。
- 在 **structural** 类型系统中，类型之间的比较是基于 structure 的。您可以定义结构类型 Sized，包括所有定义 `.__len__()` 的实例，无论它的 nominal 类型是什么。

目前正在进行通过 [PEP 544](https://www.python.org/dev/peps/pep-0544/) 将成熟的 structural 类型系统引入 Python 的工作，该系统旨在添加称为 protocols 的概念。不过，大多数 PEP 544 已在 Mypy 中实现。

一个 protocol 指定必须实施的一种或多种方法。例如，所有定义 `.__len__()` 的类均满足 `type.Sized` 协议。因此，我们可以如下注释 `len()`：

```python
from typing import Sized

def len(obj: Sized) -> int:
    return obj.__len__()
```

在 typing 模块中定义的协议的其他示例包括 Container，Iterable，Awaitable 和 ContextManager。

您也可以定义自己的 protocols。 这是通过从继承 Protocol 并定义协议期望的功能签名（带有空功能体）来完成的。 以下示例显示了如何实现 `len()` 和 `Sized`：

```python
from typing_extensions import Protocol

class Sized(Protocol):
    def __len__(self) -> int: ...

def len(obj: Sized) -> int:
    return obj.__len__()
```

在撰写本文时，对自定义协议的支持仍处于试验阶段，只能通过 `Typing_extensions` 模块获得。必须通过执行 `pip install typing-extensions`，从 PyPI 显式安装此模块。

## Optioanl Type

Python中的常见模式是将 None 用作参数的默认值。通常这样做是为了避免[可变默认值](https://docs.quantifiedcode.com/python-anti-patterns/correctness/mutable_default_value_as_argument.html)出现问题，或者是用标记特殊行为的前哨值。

在纸牌示例中，`player_order()` 函数使用 None 作为开始的哨兵值，如果没有给出 start 的玩家，则应随机选择：

```python
def player_order(names, start=None):
    """Rotate player order so that start goes first"""
    if start is None:
        start = choose(names)
    start_idx = names.index(start)
    return names[start_idx:] + names[:start_idx]
```

这给类型提示带来了挑战，即一般而言，开始应为字符串。但是，它也可以采用特殊的非字符串值 None。

为了注解此类参数，您可以使用 Optional 类型：

```python
from typing import Sequence, Optional

def player_order(
    names: Sequence[str], start: Optional[str] = None
) -> Sequence[str]:
    ...
```

Optional 类型仅表示变量具有指定的类型或为 None。一种等效的指定方法是使用 `Union` 类型：`Union[None, str]`

请注意，在使用 Optional 或 Union 时，必须注意变量在操作时具有正确的类型。 在示例中，这是通过测试 start 是否为 None 来完成的。 不这样做会导致静态类型错误以及可能的运行时错误：

```python
# player_order.py

from typing import Sequence, Optional

def player_order(
    names: Sequence[str], start: Optional[str] = None
) -> Sequence[str]:
    start_idx = names.index(start)
    return names[start_idx:] + names[:start_idx]
```

Mypy告诉您您没有处理 start 为 None 的情况：

```shell
$ mypy player_order.py
player_order.py:8: error: Argument 1 to "index" of "list" has incompatible
                          type "Optional[str]"; expected "str"
```

> 注意：对可选参数使用 None 非常普遍，Mypy 会自动对其进行处理。 Mypy 假定即使类型提示没有明确指出，默认参数 None 仍指示可选参数。您可能已经使用了以下内容：
>
> ```python
> def player_order(names: Sequence[str], start: str = None) -> Sequence[str]:
>    ...
> ```
>
> 如果您不希望 Mypy 做出此假设，则可以使用 `--no-implicit-optional` 命令行选项将其关闭。

## 示例：游戏的对象化

让我们将纸牌游戏改写为[面向对象](https://realpython.com/python3-object-oriented-programming/)的游戏。这将使我们讨论如何正确注释类和方法。

将纸牌游戏或多或少直接转换为使用 Card，Deck，Player 和 Game 类的代码，如下所示：

```python
# game.py

import random
import sys

class Card:
    SUITS = "♠ ♡ ♢ ♣".split()
    RANKS = "2 3 4 5 6 7 8 9 10 J Q K A".split()

    def __init__(self, suit, rank):
        self.suit = suit
        self.rank = rank

    def __repr__(self):
        return f"{self.suit}{self.rank}"

class Deck:
    def __init__(self, cards):
        self.cards = cards

    @classmethod
    def create(cls, shuffle=False):
        """Create a new deck of 52 cards"""
        cards = [Card(s, r) for r in Card.RANKS for s in Card.SUITS]
        if shuffle:
            random.shuffle(cards)
        return cls(cards)

    def deal(self, num_hands):
        """Deal the cards in the deck into a number of hands"""
        cls = self.__class__
        return tuple(cls(self.cards[i::num_hands]) for i in range(num_hands))

class Player:
    def __init__(self, name, hand):
        self.name = name
        self.hand = hand

    def play_card(self):
        """Play a card from the player's hand"""
        card = random.choice(self.hand.cards)
        self.hand.cards.remove(card)
        print(f"{self.name}: {card!r:<3}  ", end="")
        return card

class Game:
    def __init__(self, *names):
        """Set up the deck and deal cards to 4 players"""
        deck = Deck.create(shuffle=True)
        self.names = (list(names) + "P1 P2 P3 P4".split())[:4]
        self.hands = {
            n: Player(n, h) for n, h in zip(self.names, deck.deal(4))
        }

    def play(self):
        """Play a card game"""
        start_player = random.choice(self.names)
        turn_order = self.player_order(start=start_player)

        # Play cards from each player's hand until empty
        while self.hands[start_player].hand.cards:
            for name in turn_order:
                self.hands[name].play_card()
            print()

    def player_order(self, start=None):
        """Rotate player order so that start goes first"""
        if start is None:
            start = random.choice(self.names)
        start_idx = self.names.index(start)
        return self.names[start_idx:] + self.names[:start_idx]

if __name__ == "__main__":
    # Read player names from command line
    player_names = sys.argv[1:]
    game = Game(*player_names)
    game.play()
```

现在，我们将类型添加到此代码中。

## 方法的类型提示

首先，方法的类型提示与函数的类型提示工作原理大致相同。唯一的区别是self参数不需要注释，因为它总是一个类实例。 Card类的类型很容易添加：

```python
class Card:
    SUITS = "♠ ♡ ♢ ♣".split()
    RANKS = "2 3 4 5 6 7 8 9 10 J Q K A".split()

    def __init__(self, suit: str, rank: str) -> None:
        self.suit = suit
        self.rank = rank

    def __repr__(self) -> str:
        return f"{self.suit}{self.rank}"
```

请注意，`.__init__()` 方法始终应将 None 作为其返回类型。

## 把类作为类型

类（classes）和类型（types）之间存在对应关系。例如，Card类的所有实例一起构成Card类型。要将类用作类型，您只需使用类的名称。

```python
class Deck:
    def __init__(self, cards: List[Card]) -> None:
        self.cards = cards
```

Mypy 可以将您在 Card 中的使用与 Card 类的定义联系起来。

当您需要引用当前正在定义的类时，这种方法就无法正常工作了。 例如，`Deck.create()` 类方法返回一个 Deck 类型的对象。 但是，您不能简单地添加 `-> Deck`，因为 Deck 类尚未完全定义。

相反，允许您在注解中使用字符串文字。 这些字符串将仅在以后由类型检查器评估，因此可以包含自身和正向引用。 `.create()` 方法的类型应使用以下字符串文字：

```python
class Deck:
    @classmethod
    def create(cls, shuffle: bool = False) -> "Deck":
        """Create a new deck of 52 cards"""
        cards = [Card(s, r) for r in Card.RANKS for s in Card.SUITS]
        if shuffle:
            random.shuffle(cards)
        return cls(cards)
```

请注意，Player 类也将引用 Deck 类。但是，这没问题，因为 Deck 是在 Player 之前定义的：

```python
class Player:
    def __init__(self, name: str, hand: Deck) -> None:
        self.name = name
        self.hand = hand
```

通常在运行时不使用注解。 这为[推迟注解](https://www.python.org/dev/peps/pep-0563/)的评估提供了思路。 建议不是存储注解作为 Python 表达式并存储其值，而是建议存储注解的字符串表示形式，并仅在需要时对其进行求值。

计划将这种功能在神话般的 [Python 4.0](http://www.curiousefficiency.org/posts/2014/08/python-4000.html) 中成为标准配置。但是，在 Python 3.7 和更高版本中，可以通过 `__future__` 导入获得前向引用：

```python
from __future__ import annotations

class Deck:
    @classmethod
    def create(cls, shuffle: bool = False) -> Deck:
        ...
```

通过 `__future__` 导入，即使在定义 Deck 之前，也可以使用 Deck 代替 "Deck"。

## 返回 self 或者 cls

如前所述，通常不应注解 self 或 cls 参数。 部分地，由于自身指向该类的实例，因此这不是必需的，因此它将具有该类的类型。 在 Card 示例中，self 具有隐式类型 Card。 另外，显式添加此类型将很麻烦，因为尚未定义该类。 您将必须使用字符串文字语法 `self: "Card"`。

不过，在某些情况下，您可能想注解 self 或 cls。考虑一下，如果您有一个其他类继承的超类，并且该类具有返回 self 或 cls 的方法，那么会发生什么：

```python
# dogs.py

from datetime import date

class Animal:
    def __init__(self, name: str, birthday: date) -> None:
        self.name = name
        self.birthday = birthday

    @classmethod
    def newborn(cls, name: str) -> "Animal":
        return cls(name, date.today())

    def twin(self, name: str) -> "Animal":
        cls = self.__class__
        return cls(name, self.birthday)

class Dog(Animal):
    def bark(self) -> None:
        print(f"{self.name} says woof!")

fido = Dog.newborn("Fido")
pluto = fido.twin("Pluto")
fido.bark()
pluto.bark()
```

虽然码运行没有问题，Mypy将标记问题：

```shell
$ mypy dogs.py
dogs.py:24: error: "Animal" has no attribute "bark"
dogs.py:25: error: "Animal" has no attribute "bark"
```

问题是，即使继承的 `Dog.newborn()` 和 `Dog.twin()` 方法将返回 Dog，注释也指出它们返回了 Animal。

在这种情况下，您需要更加小心以确保注解正确。 返回类型应匹配 self 的类型或 cls 的实例类型。 这可以通过使用类型变量来完成，该变量跟踪实际传递给 self 和 cls 的内容：

```python
# dogs.py

from datetime import date
from typing import Type, TypeVar

TAnimal = TypeVar("TAnimal", bound="Animal")

class Animal:
    def __init__(self, name: str, birthday: date) -> None:
        self.name = name
        self.birthday = birthday

    @classmethod
    def newborn(cls: Type[TAnimal], name: str) -> TAnimal:
        return cls(name, date.today())

    def twin(self: TAnimal, name: str) -> TAnimal:
        cls = self.__class__
        return cls(name, self.birthday)

class Dog(Animal):
    def bark(self) -> None:
        print(f"{self.name} says woof!")

fido = Dog.newborn("Fido")
pluto = fido.twin("Pluto")
fido.bark()
pluto.bark()
```

此示例中有几件事要注意：

- 类型变量 TAnimal 用于表示返回值可能是 Animal 子类的实例。
- 我们指定 Animal 是 TAnimal 的上限。指定 bound 意味着 TAnimal 将仅是 Animal 或其子类之一。这是正确限制允许的类型所必需的。
- `typing.Type[]` 构造与 `type()` 的 typing 是等效。您需要注意类方法需要一个类并返回该类的实例。

## 注解 *args 和 **kwargs

在游戏的面向对象版本中，我们添加了在命令行上为玩家命名的选项。这是通过在程序名称之后列出播放器名称来完成的：

```shell
$ python game.py GeirArne Dan Joanna
Dan: ♢A   Joanna: ♡9   P1: ♣A   GeirArne: ♣2
Dan: ♡A   Joanna: ♡6   P1: ♠4   GeirArne: ♢8
Dan: ♢K   Joanna: ♢Q   P1: ♣K   GeirArne: ♠5
Dan: ♡2   Joanna: ♡J   P1: ♠7   GeirArne: ♡K
Dan: ♢10  Joanna: ♣3   P1: ♢4   GeirArne: ♠8
Dan: ♣6   Joanna: ♡Q   P1: ♣Q   GeirArne: ♢J
Dan: ♢2   Joanna: ♡4   P1: ♣8   GeirArne: ♡7
Dan: ♡10  Joanna: ♢3   P1: ♡3   GeirArne: ♠2
Dan: ♠K   Joanna: ♣5   P1: ♣7   GeirArne: ♠J
Dan: ♠6   Joanna: ♢9   P1: ♣J   GeirArne: ♣10
Dan: ♠3   Joanna: ♡5   P1: ♣9   GeirArne: ♠Q
Dan: ♠A   Joanna: ♠9   P1: ♠10  GeirArne: ♡8
Dan: ♢6   Joanna: ♢5   P1: ♢7   GeirArne: ♣4
```

这是通过实例化 `sys.argv` 并将其传递给 `Game()` 来实现的。 `.__init__()` 方法使用 `*names` 将给定名称打包为元组。

关于类型注解：即使名称将是字符串的元组，您也应仅注解每个名称的类型。换句话说，您应该使用 `str` 而不是 `Tuple[str]`：

同样，如果您有接受 `**kwargs` 的函数或方法，则应仅注解每个可能的关键字参数的类型。

## Callables

函数是 Python 中的[一等对象](https://dbader.org/blog/python-first-class-functions)。这意味着您可以将函数用作其他函数的参数。这也意味着您需要能够添加表示函数的类型提示。

函数以及 lambda，方法和类均通过键入 `.Callable` 表示。 通常还表示参数的类型和返回值。 例如，`Callable[[[A1, A2, A3]，Rt]` 表示一个函数，该函数具有三个分别为 A1，A2 和 A3 类型的参数。 该函数的返回类型为 Rt。

在下面的示例中，函数 `do_twice()` 调用给定函数两次并输出返回值：

```python
# do_twice.py

from typing import Callable

def do_twice(func: Callable[[str], str], argument: str) -> None:
    print(func(argument))
    print(func(argument))

def create_greeting(name: str) -> str:
    return f"Hello {name}"

do_twice(create_greeting, "Jekyll")
```

注意第 5 行上 `do_twice()` 的 `func` 参数的注解。它表示 `func` 应该是一个带有一个字符串参数的可调用对象，该参数还返回一个字符串。第 9 行中定义的 `create_greeting()` 就是这样的可调用示例。

大多数可调用类型都可以用类似的方式注解。但是，如果需要更大的灵活性，请查看[回调协议](https://mypy.readthedocs.io/en/latest/protocols.html#callback-protocols)和[扩展的可调用类型](https://mypy.readthedocs.io/en/latest/additional_features.html#extended-callable-types)。

## 示例: Hearts

让我们以 [Hearts](https://en.wikipedia.org/wiki/Hearts) 游戏的完整示例结尾。您可能已经从其他计算机模拟中了解了这款游戏。以下是规则的简要介绍：

- 四名玩家各有13张牌。
- 持有♣2的玩家在第一轮开始，必须下注♣2。
- 玩家轮流玩纸牌，如果可能的话跟随领队。
- 领先的一组中玩最高牌的玩家将赢得花样，并在下一轮中成为开始玩家。
- 除非在较早的技巧中玩过♡，否则玩家无法带领♡。
- 玩完所有纸牌后，如果玩家持有某些纸牌，则可获得积分：
    - ♠Q 13分
    - 每个 ♡ 一分
- 一场游戏持续数轮，直到一位玩家获得100分以上。得分最少的玩家获胜。

可以在[网上](https://www.bicyclecards.com/how-to-play/hearts)找到更多详细信息。

在此示例中，您还没有看到很多新的 typing 概念。因此，我们将不详细介绍此[代码](https://github.com/realpython/materials/tree/master/python-type-checking)，而将其作为带注解的代码的示例。

以下是代码中需要注意的几点：

- 对于使用联合或类型变量难以表达的类型关系，可以使用 `@overload` 装饰器。有关示例，请参见 `Deck.__getitem__()`。有关更多信息，请参见[文档](https://mypy.readthedocs.io/en/latest/more_types.html#function-overloading)。
- 子类与子类型相对应，因此可以在需要播放器的任何地方使用 `HumanPlayer`。
- 当子类从超类重新实现方法时，类型注释必须匹配。有关示例，请参见 `HumanPlayer.play_card()`。

开始游戏时，您控制第一个玩家。输入数字以选择要玩的卡。以下是游戏的示例，突出显示的行显示了玩家做出选择的位置：

```shell
$ python hearts.py GeirArne Aldren Joanna Brad

Starting new round:
Brad -> ♣2
  0: ♣5  1: ♣Q  2: ♣K  (Rest: ♢6 ♡10 ♡6 ♠J ♡3 ♡9 ♢10 ♠7 ♠K ♠4)
  GeirArne, choose card: 2
GeirArne => ♣K
Aldren -> ♣10
Joanna -> ♣9
GeirArne wins the trick

  0: ♠4  1: ♣5  2: ♢6  3: ♠7  4: ♢10  5: ♠J  6: ♣Q  7: ♠K  (Rest: ♡10 ♡6 ♡3 ♡9)
  GeirArne, choose card: 0
GeirArne => ♠4
Aldren -> ♠5
Joanna -> ♠3
Brad -> ♠2
Aldren wins the trick

...

Joanna -> ♡J
Brad -> ♡2
  0: ♡6  1: ♡9  (Rest: )
  GeirArne, choose card: 1
GeirArne => ♡9
Aldren -> ♡A
Aldren wins the trick

Aldren -> ♣A
Joanna -> ♡Q
Brad -> ♣J
  0: ♡6  (Rest: )
  GeirArne, choose card: 0
GeirArne => ♡6
Aldren wins the trick

Scores:
Brad             14  14
Aldren           10  10
GeirArne          1   1
Joanna            1   1
```

# 静态类型检查（Static Type Checking）

到目前为止，您已经了解了如何在代码中添加类型提示。在本部分中，您将了解有关如何实际执行 Python 代码的静态类型检查的更多信息。

## Mypy 项目

Mypy 由 Jukka Lehtosalo 于 2012 年左右在剑桥学习其博士学位期间创立。Mypy 最初被设想为具有无缝动态和静态类型的 Python 变量。有关Mypy最初愿景的示例，请参见[Jukka在PyCon Finland 2012](https://www.slideshare.net/jukkaleh/mypy-pyconfi2012)上的幻灯片。

大多数原始创意在 Mypy 项目中仍然发挥着重要作用。实际上，[Mypy 主页](http://mypy-lang.org/)上仍显着标语“无缝动态和静态类型”，并很好地描述了在 Python 中使用类型提示的动机。

自2012年以来最大的变化是 Mypy 不再是 Python 的变体。 在 Mypy 的第一个版本中，它是一种独立的语言，除了类型声明外，它都与 Python 兼容。 根据 Guido van Rossum 的建议，Mypy被改写为使用注解。 今天，Mypy 是常规 Python 代码的静态类型检查器。

## 运行 Mypy

首次运行Mypy之前，必须安装该程序。使用pip最容易做到这一点：

```shell
$ pip install mypy
```

安装Mypy后，您可以将其作为常规命令行程序运行：

```shell
$ mypy my_program.py
```

在 my_program.py Python文件上运行 Mypy 将检查它是否存在类型错误，而无需实际执行代码。

类型检查代码时，有许多可用选项。 由于 Mypy 仍在积极开发中，因此命令行选项可能会在版本之间进行更改。 您应该参考 Mypy 的帮助，以查看您的版本默认的设置：

```shell
$ mypy --help
usage: mypy [-h] [-v] [-V] [more options; see below]
            [-m MODULE] [-p PACKAGE] [-c PROGRAM_TEXT] [files ...]

Mypy is a program that will type check your Python code.

[... The rest of the help hidden for brevity ...]
```

此外，在线[Mypy命令行文档](https://mypy.readthedocs.io/en/stable/command_line.html#command-line)中有很多信息。

让我们看一些最常见的选项。首先，如果您使用的是没有类型提示的第三方程序包，则可能要使 Mypy 关于这些程序的警告保持沉默。这可以通过 --ignore-missing-imports 选项来完成。

下面的示例使用 Numpy 来计算和打印多个数字的余弦：

```python
# cosine.py

import numpy as np

def print_cosine(x: np.ndarray) -> None:
    with np.printoptions(precision=3, suppress=True):
        print(np.cos(x))

x = np.linspace(0, 2 * np.pi, 9)
print_cosine(x)
```

请注意，`np.printoptions()` 仅在 Numpy 的 1.15 版和更高版本中可用。运行此示例会将一些数字输出到控制台：

```shell
$ python cosine.py
[ 1.     0.707  0.    -0.707 -1.    -0.707 -0.     0.707  1.   ]
```

此示例的实际输出并不重要。但是，您应该注意，参数 x 在第 5 行上用 np.ndarray 进行了注解，因为我们要打印完整数字数组的余弦值。

您可以照常在此文件上运行Mypy：

```shell
$ mypy cosine.py 
cosine.py:3: error: No library stub file for module 'numpy'
cosine.py:3: note: (Stub files are from https://github.com/python/typeshed)
```

这些警告可能对您而言并没有立即意义，但是您将很快了解 stubs 和 typeshed。从本质上讲，您可以将警告读为 Mypy 提示 Numpy 包不包含类型提示。

在大多数情况下，不需要烦扰第三方程序包中的类型提示，因此可以使这些消息静音：

```shell
$ mypy --ignore-missing-imports cosine.py 
$
```

如果您使用 --ignore-missing-import 命令行选项，Mypy 将不会尝试跟踪或警告任何丢失的导入。但是，这可能有点笨拙，因为它也忽略了实际错误，例如拼写错误的软件包名称。

处理第三方软件包的两种不那么麻烦的方法是使用类型注解或配置文件。

在上面的一个简单示例中，您可以通过在包含导入的行中添加类型注释来使 numpy 警告静音：

```python
import numpy as np  # type: ignore
```

文字 `＃type：ignore` 告诉Mypy忽略Numpy的导入。

如果您有多个文件，则可能更容易跟踪配置文件中要忽略的导入。 如果存在，Mypy 会在当前目录中读取一个名为 mypy.ini 的文件。 该配置文件必须包含名为 [mypy] 的部分，并且可能包含格式为 [mypy-module] 的模块特定部分。

以下配置文件将忽略Numpy缺少类型提示：

```ini
# mypy.ini

[mypy]

[mypy-numpy]
ignore_missing_imports = True
```

在配置文件中可以指定许多选项。也可以指定一个全局配置文件。请参阅[文档](https://mypy.readthedocs.io/en/stable/config_file.html)以获取更多信息。

## 添加 Stubs

类型提示可用于Python标准库中的所有软件包。但是，如果您使用的是第三方软件包，您已经发现情况可能有所不同。

下面的示例使用 [parse 包](https://pypi.org/project/parse/) 进行简单的文本解析。要继续进行，您应该首先安装 Parse：

```shell
$ pip install parse
```

Parse 可用于识别简单模式。这是一个小程序，它会尽力找出您的名字：

```python
# parse_name.py

import parse

def parse_name(text: str) -> str:
    patterns = (
        "my name is {name}",
        "i'm {name}",
        "i am {name}",
        "call me {name}",
        "{name}",
    )
    for pattern in patterns:
        result = parse.parse(pattern, text)
        if result:
            return result["name"]
    return ""

answer = input("What is your name? ")
name = parse_name(answer)
print(f"Hi {name}, nice to meet you!")
```

最后三行定义了主要流程：询问您的姓名，解析答案并打印问候语。在第 14 行调用了 parse 包，以便尝试根据第 7-11 行列出的模式之一查找名称。

该程序可以如下使用：

```shell
$ python parse_name.py
What is your name? I am Geir Arne
Hi Geir Arne, nice to meet you!
```

请注意，即使我回答 `I am Geir Arne`，该程序也会指出我不是我的名字的一部分。

让我们在程序中添加一个小错误，看看Mypy是否能够帮助我们检测到它。将第16行从 `return result["name"]` 更改为 `return result`。这将返回一个 `parse.Result` 对象，而不是包含名称的字符串。

接下来在程序上运行Mypy：

```shell
$ mypy parse_name.py 
parse_name.py:3: error: Cannot find module named 'parse'
parse_name.py:3: note: (Perhaps setting MYPYPATH or using the
                       "--ignore-missing-imports" flag would help)
```

Mypy会输出与上一节中看到的错误类似的错误：它不知道解析包。您可以尝试忽略导入：

```shell
$ mypy parse_name.py --ignore-missing-imports
$
```

不幸的是，忽略导入意味着 Mypy 无法在程序中发现错误。 更好的解决方案是将类型提示添加到 Parse 包本身。 由于 Parse 是开源的，因此您实际上可以在源代码中添加类型然后发一个PR。

或者，您可以在存根文件中添加类型。[存根文件](https://mypy.readthedocs.io/en/latest/stubs.html)是一个文本文件，其中包含方法和函数的签名，但不包含其实现。 它们的主要功能是在代码中添加出于某种原因您无法更改的类型提示。 为了展示其工作原理，我们将为 Parse 包添加一些存根。

首先，应将所有存根文件放在一个公共目录中，并将 MYPYPATH 环境变量设置为指向该目录。在 Mac 和 Linux 上，您可以如下设置 MYPYPATH：

```shell
$ export MYPYPATH=/home/gahjelle/python/stubs
```

您可以通过将行添加到 .bashrc 文件中来永久设置变量。在 Windows 上，您可以单击开始菜单并搜索环境变量以设置 MYPYPATH。

接下来，在存根目录中创建一个名为 parse.pyi 的文件。它必须为要为其添加类型提示的包命名，后缀为 .pyi。现在将此文件留空。然后再次运行 Mypy：

```shell
$ mypy parse_name.py
parse_name.py:14: error: Module has no attribute "parse"
```

如果正确设置了所有内容，则应该看到此新错误消息。 Mypy 使用新的 parse.pyi 文件确定解析包中可用的功能。 由于存根文件为空，Mypy 假定 parse.parse() 不存在，然后给出您在上面看到的错误。

以下示例未为整个解析包添加类型。相反，它显示了您需要添加的类型提示，以便 Mypy 类型检查您对 parse.parse() 的使用：

```python
# parse.pyi

from typing import Any, Mapping, Optional, Sequence, Tuple, Union

class Result:
    def __init__(
        self,
        fixed: Sequence[str],
        named: Mapping[str, str],
        spans: Mapping[int, Tuple[int, int]],
    ) -> None: ...
    def __getitem__(self, item: Union[int, str]) -> str: ...
    def __repr__(self) -> str: ...

def parse(
    format: str,
    string: str,
    evaluate_result: bool = ...,
    case_sensitive: bool = ...,
) -> Optional[Result]: ...
```

省略号...是文件的一部分，应完全按照上面的说明编写。存根文件应仅包含变量，属性，函数和方法的类型提示，因此应省略实现并用...标记代替。

最终Mypy能够发现我们引入的错误：

```shell
$ mypy parse_name.py
parse_name.py:16: error: Incompatible return value type (got
                         "Result", expected "str")
```

这直接指向第 16 行，是我们返回 Result 对象而不是名称字符串的事实。将返回结果改回 `return result["name"]`，然后再次运行 Mypy 感到很高兴。

## Typeshed

您已经了解了如何使用存根添加类型提示，而无需更改源代码本身。 在上一节中，我们向第三方 Parse 包中添加了一些类型提示。 现在，如果每个人都需要为他们使用的所有第三方程序包创建自己的存根文件，那将不会很有效。

[Typeshed](https://github.com/python/typeshed) 是一个 Github 存储库，其中包含 Python 标准库的类型提示以及许多第三方软件包。 Mypy 附带有 Typeshed，因此，如果您使用的包装中已经在 Typeshed 中定义了类型提示，则类型检查将正常进行。

您还可以为 Typeshed 提供类型提示。不过，请确保首先获得软件包所有者的许可，尤其是因为他们可能正在努力在源代码本身中添加类型提示-这是[首选方法](https://github.com/python/typeshed/blob/master/CONTRIBUTING.md#adding-a-new-library)。

## 其他静态类型检查器

在本教程中，我们主要集中在使用Mypy进行类型检查。但是，Python生态系统中还有其他静态类型检查器。

[PyCharm](https://realpython.com/python-ides-code-editors-guide/#pycharm) 编辑器附带了自己的类型检查器。如果您使用 PyCharm 编写 Python 代码，则会自动进行类型检查。

Facebook 开发了 [Pyre](https://pyre-check.org/)。其既定目标之一是要快速且高效。尽管存在一些差异，但是 Pyre 的功能大部分类似于 Mypy。如果您有兴趣尝试使用 Pyre，请参阅[文档](https://pyre-check.org/docs/overview.html)。

此外，谷歌创建了 [Pytype](https://github.com/google/pytype)。 此类型检查器也与 Mypy 大致相同。 除了检查带注解的代码外，Pytype还支持对未注解的代码运行类型检查，甚至自动为代码添加注解。 有关更多信息，请参见[快速入门文档](https://github.com/google/pytype/blob/master/docs/quickstart.md)。

## 在运行时使用类型

最后一点，在 Python 程序执行期间，也可以在运行时使用类型提示。 Python 可能永远不会原生支持运行时类型检查。

但是，类型提示可以在运行时在 `__annotations__` 词典中找到，并且可以根据需要使用这些提示进行类型检查。 在开始编写自己的用于执行类型的程序包之前，您应该知道已经有多个程序包为您执行此操作。 请看一些示例的[Enforce](https://pypi.org/project/enforce/)，[Pydantic](https://pypi.org/project/pydantic/) 或 [Pytype](https://pypi.org/project/pytypes/)。

类型提示的另一种用法是将 Python 代码转换为 C 并进行编译以进行优化。 受欢迎的 [Cython 项目](https://cython.org/)使用混合C / Python 语言编写静态类型的 Python 代码。 但是，自版本 0.27 以来，Cython还支持类型注释。 最近，[Mypyc项目](https://github.com/mypyc/mypyc)已经可用。 虽然尚未准备好通用，但它可以将一些带有类型注释的 Python 代码编译为C扩展。


# 结论

Python 中的类型提示是一项非常有用的功能，您可以愉快地使用它。 类型提示无法使您能够编写任何不使用输入提示就无法编写的代码。 相反，使用类型提示可以使您更轻松地进行代码推理，发现细微的错误并维护干净的体系结构。

在本教程中，您学习了类型提示在 Python 中的工作方式，以及渐进式输入如何使 Python 中的类型检查比许多其他语言更灵活。 您已经了解了使用类型提示的利弊，以及如何使用注解或类型注释将它们添加到代码中。 最终，您了解了 Python 支持的许多不同类型，以及如何执行静态类型检查。

有很多资源可以了解有关 Python 中静态类型检查的更多信息。 [PEP 483](https://www.python.org/dev/peps/pep-0483/) 和 [PEP 484](https://www.python.org/dev/peps/pep-0484/) 提供了许多有关如何在 Python 中实现类型检查的背景知识。 [Mypy 文档](https://mypy.readthedocs.io/)有一个很好的[参考部分](https://mypy.readthedocs.io/en/stable/builtin_types.html)，详细介绍了所有可用的类型。
