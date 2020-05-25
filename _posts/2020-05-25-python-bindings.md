---
layout: post
title: Python Bindings - Calling C or C++ From Python
tags: 
    - python
    - python bindings
    - ctypes
    - CFFI
    - Cython
    - PyBind11
---
![image](../assets/images/python-bindings.png)
> python 最被人诟病的问题是什么？ 慢，这是被人诟病最多的问题，很少人知道具体原因，极少人愿意去深入了解并找到原因，更极少的人愿意付出时间去解决这个问题，很多人都是停留在抱怨吐槽阶段，知乎上有几个问题是跟这个问题相关的，里面有些答案很专业，很早以前我也去找寻过答案，总结下来：
>
> - 动态特性导致解释器低效（python是非常动态的语言，为了支持这些动态特性，付出的是解释器的低效）
> - 解释器（或者称 python VM）在 GC 方面的低效
> - 由于 [GIL](https://en.wikipedia.org/wiki/Global_interpreter_lock) 的存在，无法更好的支持 CPU 密集型的计算场景
> - 没有 JIT 和更好的 VM，这是相对其它语言来讲，比如：Java
>
> 所以也就可以围绕这几个方面来找到解决方案提升 python 程序的执行速度，还是有些人愿意贡献自己的时间从这些方面去提升 python 的性能，比如 GIL 的问题，我看到 [pycon 2019](https://www.youtube.com/watch?v=7RlqbHCCVyc) 这位小伙就分享了他尝试去解决这个问题。
>
> 这篇文章是翻译自 `realpython` 上题为 [Python Bindings: Calling C or C++ From Python]((https://realpython.com/python-bindings-overview/)) 的文章，怎么绕开 GIL 的限制，怎么避免解释器的低效，python bindings 是一个方案，也是最常用的方案。

您是要从 Python 使用 C 或 C++ 库的 Python 开发人员吗？ 如果是这样，则 Python bindings 允许您调用函数并将数据从 Python 传递到 C 或 C++，从而使您能够充分利用这两种语言的优势。 在本教程中，您会看到一些可用于创建 Python bindings 的工具的概述。

在本教程中，您将了解：

- 为什么要从 Python **调用 C 或 C++**
- 如何在 C 和 Python 之间**传递数据**
- 哪些**工具和方法**可以帮助您创建 Python 绑定

本教程针对中级 Python 开发人员。 它假定您具有 Python 的基础知识，并且对 C 或 C++ 中的函数和数据类型有所了解。 点击[链接](https://realpython.com/bonus/python-bindings-code/)，您可以获得本教程中将看到的所有示例代码。

让我们深入研究 Python bindings！

# Python Bindings 概述

在深入研究如何从 Python 调用 C 之前，最好花点时间了解为什么。 在几种情况下，创建 Python 绑定来调用 C 库是一个好主意：

1. 您已经拥有一个大型的，经过测试的，稳定的，用 C++ 编写的库，并且希望在 Python 中加以利用。 这可以是通信库，也可以是与特定硬件对话的库。 它的作用并不重要。
2. 您想通过将关键部分转换为 C 来加快 Python 代码的特定部分的速度。C 不仅执行速度更快，而且还允许您在小心的情况下摆脱 GIL 的限制。
3. 您想使用Python测试工具对其系统进行大规模测试。

以上所有都是学习创建 Python 绑定 C 库接口的重要原因。

> 注意：在本教程中，您将创建与 C 和 C++ 的 Python 绑定。 大多数通用概念都适用于两种语言，因此除非两种语言之间有特定区别，否则将使用 C。 通常，每种工具都支持 C 或 C++，但不能同时支持两者。

让我们开始吧！

## 编组数据类型（Marshalling Data Types）

等等！ 在开始编写 Python 绑定之前，请查看 Python 和 C 如何存储数据以及这将导致什么类型的问题。 首先，让我们定义编组（**marshalling**）。 Wikipedia 对此概念的定义如下：

> 将对象的内存表示形式转换为适合存储或传输的数据格式的过程。[原始链接](https://en.wikipedia.org/wiki/Marshalling_(computer_science))

出于您的目的，编组是 Python 绑定在准备将数据从 Python 移至 C 或反之时所做的工作。 Python 绑定需要进行编组，因为 Python 和 C 以不同的方式存储数据。 C 以尽可能紧凑的形式将数据存储在内存中。 如果使用 uint8_t，则它总共仅使用 8 bits 内存。

另一方面，在 Python 中，一切都是对象。 这意味着每个整数都会在内存中使用很多个字节。 有多少个取决于您正在运行的 Python 版本，您的操作系统以及其他因素。 这意味着您的 Python 绑定需要为跨边界传递的每个整数从 **C 整数**转换为 **Python 整数**。

其他数据类型在两种语言之间具有相似的关系。 让我们依次来看一下：

- [整数](https://realpython.com/python-data-types/#integers)存储计数数字。 Python 以[任意精度](https://mortada.net/can-integer-operations-overflow-in-python.html)存储整数，这意味着您可以存储非常非常大的数字。 C 指定整数的具体大小。 在语言之间切换时，您需要注意数据大小，以防止 Python 整数值溢出 C 整数变量。
- [浮点数](https://realpython.com/python-data-types/#floating-point-numbers)是带小数位的数字。 Python 可以存储比 C 大得多（小得多）的浮点数。这意味着您还必须注意这些值以确保它们在范围内。
- [复数](https://realpython.com/python-data-types/#complex-numbers)是具有虚部的数字。 尽管 Python 具有内置的复数，而 C 具有复杂的数，但没有内置的方法可在它们之间进行编组。 要编列复数，您需要在 C 代码中构建一个结构或类来对其进行管理。
- [字符串](https://realpython.com/python-data-types/#strings)是字符序列。 对于这种常见的数据类型，在创建 Python 绑定时，字符串会变得非常棘手。 与其他数据类型一样，Python 和 C 以完全不同的格式存储字符串。 （与其他数据类型不同，C 和 C++ 在这方面也有所不同，这很有趣！）您将研究的每个解决方案在处理字符串方面都有略有不同的方法。
- [布尔变量](https://realpython.com/python-data-types/#boolean-type-boolean-context-and-truthiness)只能有两个值。由于它们在 C 语言中得到支持，因此将它们编组将非常简单。

除了数据类型转换外，在构建 Python 绑定时还需要考虑其他问题。让我们继续探索它们。

## 了解可变值和不可变值

除了所有这些数据类型之外，您还必须了解 Python 对象可以是[可变和不可变的](https://realpython.com/courses/immutability-python/)。 在谈论**值传递**或**引用传递**时，C 具有与函数参数类似的概念。 在 C 语言中，所有参数都是传递值。 如果要允许函数在调用方中更改变量，则需要将指针传递给该变量。

您可能想知道是否可以通过使用指针简单地将不可变对象传递给 C 来解决不可变限制。 除非您进入丑陋且不可携带的极端，否则 Python 不会为您提供指向对象的指针，因此这是行不通的。 如果您要在 C 语言中修改 Python 对象，则需要采取额外的步骤来实现。 这些步骤将取决于您使用的工具，你后面会看到。

因此，您可以在项目清单中添加不变性，以便在创建 Python 绑定时考虑。创建此清单的最后一步是如何处理 Python 和 C 处理内存管理的不同方式。

## 管理内存

C 和 Python 对内存的管理方式不同。 在 C 语言中，开发人员必须管理所有内存分配，并确保一次只能释放一次。 Python 使用[垃圾收集器](https://realpython.com/python-memory-management/)为您解决此问题。

尽管每种方法都有其优势，但在创建 Python 绑定时确实增加了额外的麻烦。 您需要了解**每个对象的内存分配位置**，并确保仅在语言屏障的同一侧释放内存。

例如，当您设置`x = 3`时，将创建一个 Python 对象。该对象的内存在 Python 端分配，需要进行垃圾回收。 幸运的是，使用 Python 对象，很难做其他任何事情。 看一下 C 语言中的相反情况，您可以在其中直接分配一个内存块：

```c++
int* iPtr = (int*)malloc(sizeof(int));
```

执行此操作时，需要确保在 C 中释放此指针。这可能意味着需要手动将代码添加到 Python 绑定中。

这样就完善了您的常规主题清单。 让我们开始设置你的系统，以便您编写一些代码！

## 设置环境

在本教程中，您将使用 Real Python GitHub repo 中[已经存在的 C 和 C++ 库](https://github.com/realpython/materials/tree/master/python-bindings/overview_article)来演示每个工具的测试。 目的是您可以将这些构想用于任何 C 库。 要遵循此处的所有示例，您需要具备以下条件：

- 已安装 C++ 库并了解命令行调用的路径
- Python 开发工具：
  
  - 对于Linux，这是 python3-dev 或 python3-devel 软件包，具体取决于您的发行版。
  - 对于Windows，有[多个选项](https://wiki.python.org/moin/WindowsCompilers#Microsoft_Visual_C.2B-.2B-_10.0_standalone:_Windows_SDK_7.1_.28x86.2C_x64.2C_ia64.29)。

- Python 3.6或更高版本
- 一个[虚拟环境](https://realpython.com/courses/working-python-virtual-environments/)（推荐，但不是必需的）
- **invoke** 工具

最后一个可能对您来说是新手，所以让我们仔细看看。

## 使用 invoke 工具

[invoke](http://www.pyinvoke.org/)是本教程中用于构建和测试 Python 绑定的工具。 它具有类似的用途，但使用 Python 而不是 `Makefiles`。 您需要使用 pip 在虚拟环境中安装 `invoke`：

```shell
$ python3 -m pip install invoke
```

要运行它，请键入 `invoke`，然后键入要执行的任务：

```shell
$ invoke build-cmult
==================================================
= Building C Library
* Complete
```

要查看可用的任务，请使用 `--list` 选项：

```shell
$ invoke --list
Available tasks:

  all              Build and run all tests
  build-cffi       Build the CFFI Python bindings
  build-cmult      Build the shared library for the sample C code
  build-cppmult    Build the shared library for the sample C++ code
  build-cython     Build the cython extension module
  build-pybind11   Build the pybind11 wrapper library
  clean            Remove any built objects
  test-cffi        Run the script to test CFFI
  test-ctypes      Run the script to test ctypes
  test-cython      Run the script to test Cython
  test-pybind11    Run the script to test PyBind11
```

请注意，当您查看定义了调用任务的 `task.py` 文件时，您会看到列出的第二个任务的名称为 `build_cffi`。 但是，`--list` 的输出将其显示为 `build-cffi`。 减号（-）不能用作 Python 名称的一部分，因此该文件改用下划线（_）。

对于您要检查的每种工具，都会定义一个构建和测试任务。 例如，要运行 `CFFI` 的代码，可以键入 `invoke build-cffi test-cffi`。 `ctypes` 是一个例外，因为 `ctypes` 没有构建阶段。 此外，为方便起见还添加了两个特殊任务：

- `invoke all` 运行所有工具的构建和测试任务。
- `invoke clean` 删除所有生成的文件。

现在，您已经了解了如何运行代码，让我们先看一下要包装的 C 代码，然后再访问工具概述。

## C 或 C++ 源代码

在下面的每个示例部分中，您将为 C 或 C++ 中的同一函数创建 Python 绑定。 这些部分旨在让您了解每种方法的外观，而不是该工具的深入教程，因此您要包装的功能很小。 您将为其创建 Python 绑定的函数将一个整数和一个浮点数作为输入参数，并返回一个浮点数，该浮点数是两个数字的乘积：

```c++
// cmult.c
float cmult(int int_param, float float_param) {
    float return_value = int_param * float_param;
    printf("    In cmult : int: %d float %.1f returning  %.1f\n", int_param,
            float_param, return_value);
    return return_value;
}
```

C 和 C++ 函数几乎完全相同，它们之间的名称和[字符串](https://realpython.com/python-strings/)有所不同。 您可以通过单击[链接](https://realpython.com/bonus/python-bindings-code/)获得所有代码的副本：

现在，您已经克隆了库并安装了工具，您可以构建和测试工具。因此，让我们深入了解下面的每个部分！

# ctypes

您将从 [ctypes](https://docs.python.org/3.8/library/ctypes.html) 开始，这是标准库中用于创建 Python 绑定的工具。 它提供了一个低级工具集，用于在 Python 和 C 之间加载共享库并编组数据。

## 如何安装

ctypes 的一大优点是它是 Python 标准库的一部分。 它是在 Python 2.5 版中添加的，因此你很可能已经有了。 您可以像使用 `sys` 或 `time` 模块一样导入它。

## 调用方法

加载 C 库并调用该函数的所有代码都在您的 Python 程序中。 很好，因为您的过程没有多余的步骤。 您只需运行您的程序，一切都将得到照顾。 要在 ctypes 中创建 Python 绑定，您需要执行以下步骤：

1. **加载**库。
2. **包装**一些输入参数。
3. **告诉** ctypes 函数的返回类型。

您将依次查看每一个。

### 加载库

ctypes 提供了几种[加载共享库](https://docs.python.org/3.5/library/ctypes.html#loading-shared-libraries)的方法，其中一些是特定于平台的。 以您的示例为例，您将直接通过完整路径传递所需的共享库来创建 `ctypes.CDLL` 对象：

```python
# ctypes_test.py
import ctypes
import pathlib

if __name__ == "__main__":
    # Load the shared library into ctypes
    libname = pathlib.Path().absolute() / "libcmult.so"
    c_lib = ctypes.CDLL(libname)
```

这适用于共享库与 Python 脚本位于同一目录中的情况，但是当您尝试从 Python 绑定以外的包中加载库时要小心。 在 ctypes 文档中有许多有关平台和特定情况的加载库和查找路径的详细信息。

> 注意：库加载期间可能会出现许多特定于平台的问题。实例生效后，最好进行增量更改。

现在您已将库加载到 Python 中，可以尝试调用它了！

### 调用你编写的方法

请记住，您的C函数的函数原型如下：

```c++
// cmult.h
float cmult(int int_param, float float_param);
```

您需要传入一个整数和一个浮点数，并且可以期望返回一个浮点数。 整数和浮点数在 Python 和 C 中都具有原生支持，因此您希望这种情况适用于合理的值。

将库加载到 Python 绑定中后，该函数将成为 c_lib 的属性，c_lib 是您之前创建的 CDLL 对象。 您可以尝试这样称呼它：

```python
x, y = 6, 2.3
answer = c_lib.cmult(x, y)
```

糟糕！这行不通。在示例 repo 中，此行已被注释掉，因为它失败了。如果您尝试通过该调用运行，则 Python 会报错：

```shell
$ invoke test-ctypes
Traceback (most recent call last):
  File "ctypes_test.py", line 16, in <module>
    answer = c_lib.cmult(x, y)
ctypes.ArgumentError: argument 2: <class 'TypeError'>: Don't know how to convert parameter 2
```

看来您需要告诉 ctypes 任何不是整数的参数。 除非您明确告诉 ctypes，否则 ctypes 对该函数一无所知。 除非另有说明，否则任何参数均假定为整数。 ctypes 不知道如何将 y 中存储的值 2.3 转换为整数，因此失败。

要解决此问题，您需要根据数字创建一个 c_float。您可以在调用该函数的行中执行以下操作：

```python
# ctypes_test.py
answer = c_lib.cmult(x, ctypes.c_float(y))
print(f"    In Python: int: {x} float {y:.1f} return val {answer:.1f}")
```

现在，当您运行此代码时，它将返回您传入的两个数字的乘积：

```shell
$ invoke test-ctypes
    In cmult : int: 6 float 2.3 returning  13.8
    In Python: int: 6 float 2.3 return val 48.0
```

请稍等... 6 乘以 2.3 不是 48.0！

事实证明，与输入参数一样，ctypes 假定您的函数返回一个 int 值。 实际上，您的函数返回一个浮点数，该浮点数被错误地编组。 就像输入参数一样，您需要告诉 ctypes 使用其他类型。 这里的语法略有不同：

```python
# ctypes_test.py
c_lib.cmult.restype = ctypes.c_float
answer = c_lib.cmult(x, ctypes.c_float(y))
print(f"    In Python: int: {x} float {y:.1f} return val {answer:.1f}")
```

这应该够了吧。 让我们运行整个 test-ctypes 目标，看看有什么。 请记住，输出的第一部分是在将函数的 restype 固定为 float 之前：

```shell
$ invoke test-ctypes
==================================================
= Building C Library
* Complete
==================================================
= Testing ctypes Module
    In cmult : int: 6 float 2.3 returning  13.8
    In Python: int: 6 float 2.3 return val 48.0

    In cmult : int: 6 float 2.3 returning  13.8
    In Python: int: 6 float 2.3 return val 13.8
```

这样更好！ 当第一个未经更正的版本返回错误值时，您的固定版本同意 C 函数。 C 和 Python 都得到相同的结果！ 现在它可以正常工作了，看看为什么您可能不希望使用 ctypes。

## 长处和短处

与其他工具相比，ctypes 的最大优点是它内置在标准库中。它也不需要任何额外的步骤，因为所有工作都是在 Python 程序中完成的。

此外，所使用的概念是低级的，这使您刚进行的练习变得易于管理。 但是，由于缺乏自动化，更复杂的任务变得繁琐。 在下一节中，您将看到一个为流程增加一些自动化的工具。

# CFFI

[CFFI](https://cffi.readthedocs.io/en/latest/) 是 Python 的 **C 外部函数接口**。 它采用一种更加自动化的方法来生成 Python 绑定。 CFFI 有多种构建和使用 Python 绑定的方式。 有两个不同的选项可供选择，这为您提供了四种可能的模式：

- **ABI vs API**: API 模式使用 C 编译器生成完整的 Python 模块，而 ABI 模式加载共享库并直接与其交互。 如果不运行编译器，则正确构造结构和参数很容易出错。 文档强烈建议使用 API 模式。
- **in-line vs out-of-line**: 这两种模式之间的区别在于速度和便利性之间的权衡：

  - **In-line mode** 在每次脚本运行时，都会编译 Python 绑定。这很方便，因为您不需要额外的构建步骤。但是，它的确会使您的程序变慢。
  - **Out-of-line mode** 需要一个额外的步骤来一次生成 Python 绑定，然后在每次运行程序时都使用它们。这要快得多，但是对于您的应用程序可能并不重要。

在此示例中，您将使用 API​​ out-of-line mode，该模式会生成最快的代码，并且通常看起来与您将在本教程后面创建的其他 Python 绑定类似。

## 如何安装

由于 CFFI 不是标准库的一部分，因此您需要将其安装在计算机上。 建议您为此创建一个虚拟环境。 幸运的是，CFFI 使用 pip 进行安装：

```shell
$ python3 -m pip install cffi
```

这会将软件包安装到您的虚拟环境中。如果您已经从 requirements.txt 安装了该文件，则应注意这一点。您可以通过访问[链接](https://realpython.com/bonus/python-bindings-code/)中的 repo 来查看requirements.txt：

现在，您已经安装了 CFFI，现在该试一下了！

## 调用方法

与 ctypes 不同，使用 CFFI，您可以创建完整的 Python 模块。 您可以像导入标准库中的任何其他模块一样导入该模块。 您需要做一些额外的工作来构建 Python 模块。 要使用 CFFI Python 绑定，您需要执行以下步骤：

- **编写**一些描述绑定的 Python 代码。
- **运行**该代码以生成可加载模块。
- **修改**调用代码以导入和使用新创建的模块。

这似乎是一项艰巨的工作，但您将逐步完成这些步骤，并了解其工作原理。

### 编写绑定

CFFI 提供了一些方法，可以在生成 Python 绑定时读取 **C 头文件**来完成大部分工作。 在 CFFI 文档中，执行此操作的代码放置在单独的 Python 文件中。 在此示例中，您会将代码直接放入使用 Python 文件作为输入的构建工具调用中。 要使用 CFFI，首先创建一个 cffi.FFI 对象，该对象提供了所需的三种方法：

```python
# tasks.py
import cffi
...
""" Build the CFFI Python bindings """
print_banner("Building CFFI Module")
ffi = cffi.FFI()
```

获得FFI后，您将使用 `.cdef()` 自动处理头文件的内容。这将为您创建包装器方法，用来编组来自 Python 的数据：

```python
# tasks.py
this_dir = pathlib.Path().absolute()
h_file_name = this_dir / "cmult.h"
with open(h_file_name) as h_file:
    ffi.cdef(h_file.read())
```

读取和处理头文件是第一步。之后，您需要使用 `.set_source()` 来描述 CFFI 将生成的源文件：

```python
# tasks.py
ffi.set_source(
    "cffi_example",
    # Since you're calling a fully-built library directly, no custom source
    # is necessary. You need to include the .h files, though, because behind
    # the scenes cffi generates a .c file that contains a Python-friendly
    # wrapper around each of the functions.
    '#include "cmult.h"',
    # The important thing is to include the pre-built lib in the list of
    # libraries you're linking against:
    libraries=["cmult"],
    library_dirs=[this_dir.as_posix()],
    extra_link_args=["-Wl,-rpath,."],
)
```

以下是您要传递的参数的明细：

- `cffi_example` 是将在文件系统上创建的源文件的基本名称。 CFFI 将生成一个 `.c` 文件，将其编译为 `.o` 文件，并将其链接至 <system-description>.so 或 <system-description>.dll 文件。
- `#include "cmult.h"` 是自定义 C 源代码，在编译之前将包含在生成的源代码中。在这里，您仅包含要为其生成绑定的 `.h` 文件，但这可用于一些有趣的自定义。
- `library = ["cmult"]` 告诉链接器您先前存在的 C 库的名称。这是一个列表，因此您可以根据需要指定几个库。
- `library_dirs = [this_dir.as_posix(),]` 是目录列表，它告诉链接程序在何处查找上述库列表。
- `extra_link_args = ['-Wl，-rpath,.']` 是一组选项，它们生成一个共享库，该共享库将在当前路径（.）中查找需要加载的其他库。

### 构建 Python 绑定

调用 `.set_source()` 不会建立 Python 绑定。它仅设置元数据来描述将要生成的内容。要构建 Python 绑定，您需要调用 `.compile()`：

```python
# tasks.py
ffi.compile()
```

这通过生成 `.c` 文件，`.o` 文件和共享库来完成。您刚浏览过的 invoke 任务可以在命令行上运行以构建 Python 绑定：

```shell
$ invoke build-cffi
==================================================
= Building C Library
* Complete
==================================================
= Building CFFI Module
* Complete
```

您已经有了 CFFI Python 绑定，因此现在该运行该代码了！

### 调用你编写的方法

在完成所有工作之后，您配置并运行了 CFFI 编译器，使用生成的 Python 绑定看起来就像使用任何其他 Python 模块一样：

```python
# cffi_test.py
import cffi_example

if __name__ == "__main__":
    # Sample data for your call
    x, y = 6, 2.3

    answer = cffi_example.lib.cmult(x, y)
    print(f"    In Python: int: {x} float {y:.1f} return val {answer:.1f}")
```

导入新模块，然后可以直接调用 `cmult()`。要测试它，请使用 `test-cffi` 任务：

```shell
$ invoke test-cffi
==================================================
= Testing CFFI Module
    In cmult : int: 6 float 2.3 returning  13.8
    In Python: int: 6 float 2.3 return val 13.8
```

这将运行 `cffi_test.py` 程序，该程序将测试您使用 CFFI 创建的新 Python 绑定。这样就完成了有关编写和使用 CFFI Python 绑定的部分。

## 长处和短处

似乎 ctypes 比您刚刚看到的 CFFI 示例所需的工作更少。尽管在这种用例中确实如此，但由于许多功能包装的**自动化**，CFFI 可以比 ctypes 更好地扩展到较大的项目。

CFFI 还产生了完全不同的用户体验。 ctypes 允许您将预先存在的 C 库直接加载到 Python 程序中。另一方面，CFFI 创建了一个新的 Python 模块，该模块可以像其他 Python 模块一样加载。

更重要的是，通过上面使用的 out-of-line API 方法，创建 Python 绑定的时间代价是在构建它时就完成一次，而在每次运行代码时都不会发生。 对于小型程序，这可能没什么大不了的，但是 CFFI 也可以通过这种方式更好地扩展到较大的项目。

与 ctypes 一样，使用 CFFI 仅允许您直接与 C 库连接。 C++ 库需要大量工作才能使用。在下一节中，您将看到专注于 C++ 的 Python 绑定工具。

# PyBind11

[PyBind11](https://pybind11.readthedocs.io/en/master/) 采用了完全不同的方法来创建 Python 绑定。 除了将重点从 C 转移到 C++ 之外，它还使用 C++ 来指定和构建模块，从而使其能够利用 C++ 中的元编程工具。 与 CFFI 一样，从 PyBind11 生成的 Python 绑定是一个完整的 Python 模块，可以直接导入和使用。

PyBind11 是在 `Boost::Python` 库之后建模的，并具有类似的接口。 它将它的使用限制在 C++11 和更高版本中，但是，与支持所有功能的 Boost 相比，它可以简化并加快处理速度。

## 如何安装

PyBind11 文档的[第一步](https://pybind11.readthedocs.io/en/latest/basics.html)部分将引导您逐步了解如何下载和构建 PyBind11 的测试用例。 尽管似乎并非严格要求这样做，但按照以下步骤进行操作可以确保您设置正确的 C++ 和 Python 工具。

> 注意：PyBind11的大多数示例都使用 [cmake](https://cmake.org/)，这是构建 C 和 C++ 项目的好工具。 但是，对于此演示，您将继续使用调用工具，该工具将遵循文档的[手动构建](https://pybind11.readthedocs.io/en/latest/compiling.html#building-manually)部分中的说明。

您需要将此工具安装到[虚拟环境](https://realpython.com/python-virtual-environments-a-primer/)中：

```shell
$ python3 -m pip install pybind11
```

PyBind11 是一个全标题库，与 Boost 的大部分相似。 这允许 pip 将库的实际 C++ 源直接安装到您的虚拟环境中。

## 调用方法

在深入研究之前，请注意，**您使用的是其他 C++ 源文件 cppmult.cpp**，而不是先前示例中使用的 C 文件。 两种语言的功能基本相同。

### 编写绑定

与 CFFI 相似，您需要创建一些代码来告诉该工具如何构建 Python 绑定。 与 CFFI 不同，此代码将使用 C++ 而不是 Python。 幸运的是，所需代码最少：

```c++
// pybind11_wrapper.cpp
#include <pybind11/pybind11.h>
#include <cppmult.hpp>

PYBIND11_MODULE(pybind11_example, m) {
    m.doc() = "pybind11 example plugin"; // Optional module docstring
    m.def("cpp_function", &cppmult, "A function that multiplies two numbers");
}
```

由于 PyBind11 将大量信息打包到几行中，因此让我们一次来看一下。

前两行包括 pybind11.h 文件和 C++ 库 cppmult.hpp 的头文件。之后，您将拥有 PYBIND11_MODULE 宏。这扩展为 PyBind11 源代码中很好描述的 C++ 代码块：

> 这个宏创建入口点，当 Python 解释器导入扩展模块时，该入口点将被调用。 模块名称作为第一个参数给出，不应使用引号引起来。 第二个宏参数定义了 py::module 类型的变量，可用于初始化模块。[原始链接](https://github.com/pybind/pybind11/blob/1376eb0e518ff2b7b412c84a907dd1cd3f7f2dcd/include/pybind11/detail/common.h#L267)

对您而言，这意味着在本示例中，您正在创建一个名为 pybind11_example 的模块，其余代码将使用 m 作为 `py::module` 对象的名称。 在下一行中，在您定义的 C++ 函数中，为模块创建一个文档字符串。 虽然这是可选的，但可以使您的模块更加 [Pythonic](https://realpython.com/learning-paths/writing-pythonic-code/)。

最后，您有 `m.def()` 调用。 这将定义由您的新 Python 绑定导出的函数，这意味着它将在 Python 中可见。 在此示例中，您传递了三个参数：

- **cpp_function** 是您将在 Python 中使用的函数的导出名称。 如本例所示，它不需要匹配 C++ 函数的名称。
- **＆cppmult** 使用要导出的函数的地址。
- **"A function..."** 是该函数的可选文档字符串。

既然您已经有了 Python 绑定的代码，请看一下如何将其构建到 Python 模块中。

### 构建 python 绑定

在 PyBind11 中用于构建 Python 绑定的工具是 C++ 编译器本身。您可能需要修改编译器和操作系统的默认设置。

首先，您必须构建要为其创建绑定的 C++ 库。 例如，您可以将 cppmult 库直接构建到 Python 绑定库中。 但是，对于大多数实际示例，您将拥有一个要包装的预先存在的库，因此您将单独构建 cppmult 库。 该构建是对编译器的一个标准调用，以构建一个共享库：

```python
# tasks.py
invoke.run(
    "g++ -O3 -Wall -Werror -shared -std=c++11 -fPIC cppmult.cpp "
    "-o libcppmult.so "
)
```

使用 `invoke build-cppmult` 运行它会生成 `libcppmult.so`：

```shell
$ invoke build-cppmult
==================================================
= Building C++ Library
* Complete
```

另一方面，Python 绑定的构建需要一些特殊的细节：

```python
# tasks.py
invoke.run(
    "g++ -O3 -Wall -Werror -shared -std=c++11 -fPIC "
    "`python3 -m pybind11 --includes` "
    "-I /usr/include/python3.7 -I .  "
    "{0} "
    "-o {1}`python3.7-config --extension-suffix` "
    "-L. -lcppmult -Wl,-rpath,.".format(cpp_name, extension_name)
)
```

让我们逐行浏览。 第 3 行包含相当标准的 C++ 编译器标志，这些标志指示一些详细信息，包括您希望捕获所有警告并将其视为错误，需要共享库以及正在使用 C++11。

第 4 行是魔术的第一步。它调用 pybind11 模块，使其为 PyBind11 生成正确的包含路径。您可以直接在控制台上运行此命令以查看其作用：

```shell
$ python3 -m pybind11 --includes
-I/home/jima/.virtualenvs/realpython/include/python3.7m
-I/home/jima/.virtualenvs/realpython/include/site/python3.7
```

您的输出应该相似，但显示不同的路径。

在编译调用的第 5 行中，您可以看到您还在将路径添加到 Python 开发人员包含的文件中。 建议您不要链接到 Python 库本身，但是源代码需要 Python.h 中的一些代码才能发挥其魔力。 幸运的是，它使用的代码在 Python 版本之间相当稳定。

第5行还使用 `-I.` 将当前目录添加到包含路径列表中。这样可以解析包装代码中的 `#include <cppmult.hpp>`行。

第 6 行指定源文件的名称，即 `pybind11_wrapper.cpp`。 然后，在第 7 行上，您将看到更多的构建魔术正在发生。 此行指定输出文件的名称。 Python 在模块命名方面有一些特别的想法，包括 Python 版本，机器架构和其他细节。 Python 还提供了一个工具来帮助解决这个问题，称为 `python3.7-config`：

```python
$ python3.7-config --extension-suffix
.cpython-37m-x86_64-linux-gnu.so
```

如果您使用的是其他版本的 Python，则可能需要修改命令。 如果您使用其他版本的 Python 或使用其他操作系统，则结果可能会发生变化。

构建命令的最后一行（第 8 行）将链接器指向您之前构建的 libcppmult 库。 rpath 部分告诉链接器将信息添加到共享库，以帮助操作系统在运行时找到 libcppmult。 最后，您会注意到该字符串的格式为 cpp_name 和 extension_name。 在下一部分中使用 Cython 构建 Python 绑定模块时，将再次使用此功能。

```shell
$ invoke build-pybind11
==================================================
= Building C++ Library
* Complete
==================================================
= Building PyBind11 Module
* Complete
```

是的！您已经使用 PyBind11 构建了 Python 绑定。现在该进行测试了！

### 调用你编写的方法

与上面的 CFFI 示例类似，完成创建 Python 绑定的繁重工作后，调用函数看起来就像普通的 Python 代码：

```python
# pybind11_test.py
import pybind11_example

if __name__ == "__main__":
    # Sample data for your call
    x, y = 6, 2.3

    answer = pybind11_example.cpp_function(x, y)
    print(f"    In Python: int: {x} float {y:.1f} return val {answer:.1f}")
```

由于您在 PYBIND11_MODULE 宏中使用了 pybind11_example 作为模块的名称，因此，这就是您导入的名称。 在 `m.def()` 调用中，您告诉 PyBind11 将 cppmult 函数导出为 cpp_function，这就是从 Python 调用它的方法。

您也可以使用 invoke 对其进行测试：

```shell
$ invoke test-pybind11
==================================================
= Testing PyBind11 Module
    In cppmul: int: 6 float 2.3 returning  13.8
    In Python: int: 6 float 2.3 return val 13.8
```

这就是 PyBind11 的样子。接下来，您将了解何时以及为什么 PyBind11 是适合该工作的工具。

## 长处和短处

PyBind11 专注于 C++ 而不是 C，这使其不同于 ctypes 和 CFFI。它具有一些功能，使其对于 C++ 库非常有吸引力：

- 它支持**类**。
- 它处理**多态子类化**。
- 它使您可以从 Python 和许多其他工具向对象添加**动态属性**，而这对于您已经研究过的基于 C 的工具来说是很难做到的。

话虽这么说，您需要做一些设置和配置来启动和运行 PyBind11。 正确安装和构建可能有些棘手，但是一旦完成，它似乎就很牢固了。 另外，PyBind11 要求您至少使用 C++11 或更高版本。 对于大多数项目而言，这不太可能成为一个很大的限制，但您可能需要考虑。

最后，创建 Python 绑定所需编写的额外代码是 C++，而不是 Python。 这可能对您来说不是问题，但与您在此处看到的跟其他工具不同。 在下一节中，您将继续学习 Cython，它采用了完全不同的方法来解决此问题。

# Cython

[Cython](https://cython.org/) 用于创建 Python 绑定的方法，使用类似于 Python 的语言来定义绑定，然后生成可编译到模块中的 C 或 C++ 代码。 有多种使用 Cython 构建 Python 绑定的方法。 最常见的一种是使用 `distutils` 中的安装程序。 在此示例中，您将继续使用 `invoke` 工具，该工具将使您能够精确执行所运行的命令。

## 如何安装

Cython 是一个 Python 模块，可以从 [PyPI](https://realpython.com/courses/how-to-publish-your-own-python-package-pypi/) 安装到您的虚拟环境中：

```python
$ python3 -m pip install cython
```

同样，如果您已经将 requirements.txt 文件安装到虚拟环境中，则该文件已经存在。您可以通过单击[链接](https://realpython.com/bonus/python-bindings-code/)来获取requirements.txt 的副本。

那应该已经准备好与 Cython 合作！

## 调用方法

要使用 Cython 构建 Python 绑定，您将遵循与用于 CFFI 和 PyBind11 的步骤相似的步骤。 您将编写绑定，构建它们，然后运行 Python 代码来调用它们。 Cython 可以同时支持 C 和 C++。 在此示例中，您将使用在上面的 PyBind11 示例中使用的 cppmult 库。

### 编写绑定

在 Cython 中声明模块的最常见形式是使用 `.pyx` 文件：

```python
# cython_example.pyx
""" Example cython interface definition """

cdef extern from "cppmult.hpp":
    float cppmult(int int_param, float float_param)

def pymult( int_param, float_param ):
    return cppmult( int_param, float_param )
```

这里有两个部分：

- **第 3 行和第 4 行**告诉 Cython 您正在使用 cppmult.hpp 中的 cppmult()。
- **第 6 行和第 7 行**创建了包装函数 pymult()，以调用 cppmult()。

这里使用的语言是 C，C++ 和 Python 的特殊组合。 不过，对于 Python 开发人员来说，它看起来相当熟悉，因为其目标是使过程变得更容易。

使用 `cdef extern...` 的第一部分告诉 Cython，以下函数声明也在 `cppmult.hpp` 文件中找到。 这对于确保针对与 C++ 代码相同的声明构建 Python 绑定很有用。 第二部分看起来像是常规的 Python 函数-因为它是！ 本部分创建可访问 C++ 函数 cppmult 的 Python 函数。

现在您已经定义了 Python 绑定，是时候构建它们了！

### 构建 python 绑定

Cython 的构建过程与您用于 PyBind11 的过程相似。您首先在 .pyx 文件上运行 Cython 以生成一个 .cpp 文件。完成此操作后，可以使用与 PyBind11 相同的功能对其进行编译：

```python
# tasks.py
def compile_python_module(cpp_name, extension_name):
    invoke.run(
        "g++ -O3 -Wall -Werror -shared -std=c++11 -fPIC "
        "`python3 -m pybind11 --includes` "
        "-I /usr/include/python3.7 -I .  "
        "{0} "
        "-o {1}`python3.7-config --extension-suffix` "
        "-L. -lcppmult -Wl,-rpath,.".format(cpp_name, extension_name)
    )

def build_cython(c):
    """ Build the cython extension module """
    print_banner("Building Cython Module")
    # Run cython on the pyx file to create a .cpp file
    invoke.run("cython --cplus -3 cython_example.pyx -o cython_wrapper.cpp")

    # Compile and link the cython wrapper library
    compile_python_module("cython_wrapper.cpp", "cython_example")
    print("* Complete")
```

首先在 .pyx 文件上运行 cython。您可以在此命令上使用一些选项：

- **--cplus**告诉编译器生成 C++ 文件而不是 C 文件。
- **-3** 切换 Cython 生成 Python 3 语法，而不是 Python 2。
- **-o cython_wrapper.cpp** 指定要生成的文件的名称。

生成 C++ 文件后，就像使用 PyBind11 一样，您可以使用 C++ 编译器生成 Python 绑定。 请注意，使用 pybind11 工具生成额外包含路径的调用仍在该函数中。 在这里不会造成任何伤害，因为您的源码将不需要这些。

在 `invoke` 中运行此任务将产生以下输出：

```shell
$ invoke build-cython
==================================================
= Building C++ Library
* Complete
==================================================
= Building Cython Module
* Complete
```

您会看到它构建了 cppmult 库，然后构建了 cython 模块来包装它。现在您有了 Cython Python 绑定。（尝试快速地说出……），现在该进行测试了！

### 调用你编写的方法

调用新的 Python 绑定的 Python 代码与用于测试其他模块的代码非常相似：

```python
# cython_test.py
import cython_example

# Sample data for your call
x, y = 6, 2.3

answer = cython_example.pymult(x, y)
print(f"    In Python: int: {x} float {y:.1f} return val {answer:.1f}")
```

第 2 行将导入新的 Python 绑定模块，并在第 7 行调用 pymult()。请记住，.pyx 文件在 cppmult() 周围提供了 Python 包装器，并将其重命名为 pymult。使用 invoke 运行测试将产生以下结果：

```shell
$ invoke test-cython
==================================================
= Testing Cython Module
    In cppmul: int: 6 float 2.3 returning  13.8
    In Python: int: 6 float 2.3 return val 13.8
```

您得到与以前相同的结果！

## 长处和短处

Cython 是一个相对复杂的工具，在为 C 或 C++ 创建 Python 绑定时，可以为您提供更深层次的控制。 尽管这里没有详细介绍，但它提供了一种 Python 风格的方法来编写可手动控制 [GIL](https://realpython.com/python-gil/) 的代码，从而可以显着加快某些类型的问题。

但是，这种 Python 风格的语言并不完全是 Python，因此，当您逐渐确定 C 和 Python 的哪些部分适合您时，会有一条学习曲线。

# 其它一些解决方案

在研究本教程时，我遇到了几种用于创建 Python 绑定的工具和选项。 尽管我将本概述限制为一些更常见的选项，但我偶然发现了其他几种工具。 以下列表并不全面。 如果上述工具之一不适合您的项目，则仅是其他可能性的示例。

## PyBindGen

[PyBindGen](https://pybindgen.readthedocs.io/en/latest/tutorial/#supported-python-versions) 生成用于 C 或 C++ 的 Python 绑定，并用 Python 编写。 它的目标是产生可读的 C 或 C++ 代码，这将简化调试问题。 目前尚不清楚是否最近更新，因为该文档将 Python 3.4 列为最新测试版本。 但是，最近几年每年都有版本发布。

## Boost.Python

[Boost.Python](https://www.boost.org/doc/libs/1_66_0/libs/python/doc/html/index.html) 具有类似于您在上面看到的 PyBind11 的接口。 这不是巧合，因为 PyBind11 是基于该库的！ Boost.Python 是用完整的 C++ 编写的，并且在大多数平台上支持大多数（如果不是全部）C++ 版本。 相比之下，PyBind11 仅限于使用现代 C++。

## SIP

SIP 是为 PyQt 项目开发的用于生成 Python 绑定的工具集。 wxPython 项目也使用它来生成其绑定。 它具有代码生成工具和一个额外的 Python 模块，该模块为生成的代码提供支持功能。

## Cppyy

[cppyy](https://cppyy.readthedocs.io/en/latest/) 是一个有趣的工具，其设计目标与到目前为止所看到的略有不同。用包作者的话来说：

> “cppyy 的原始想法（可追溯到2001年）是允许生活在 C++ 世界中的 Python 程序员访问那些 C++ 程序包，而不必直接接触 C++（或等待 C++ 开发人员到来并提供绑定）。” [原始链接](https://news.ycombinator.com/item?id=15098764)

## Shiboken

[Shiboken](https://wiki.qt.io/Qt_for_Python/Shiboken) 是一种生成 Python 绑定的工具，该工具是为与 Qt 项目关联的 PySide 项目开发的。 虽然将其设计为该项目的工具，但文档显示它既不是 Qt 也不是 PySide专 用的，并且可以用于其他项目。

## SWIG

[SWIG](http://swig.org/) 是与此处列出的任何其他工具不同的工具。 这是一种通用工具，可用于创建与[许多其他语言](http://swig.org/compat.html#SupportedLanguages)（不仅限于 Python）的 C 和 C++ 程序的绑定。 在某些项目中，这种为不同语言生成绑定的功能可能会非常有用。 就复杂性而言，当然要付出代价。

# 结论

恭喜！ 现在，您已经对用于创建 Python 绑定的几种不同选项有了一定的概览。 您已经了解了编组数据和创建绑定时需要考虑的问题。 您已经了解了使用以下工具可以从 Python 调用 C 或 C++ 函数的过程：

- ctypes
- CFFI
- PyBind11
- Cython

您现在已经知道，尽管 ctypes 允许您直接加载 DLL 或共享库，但其他三个工具需要额外的步骤，但仍会创建完整的 Python 模块。 另外，您还可以使用 invoke 工具从 Python 侧运行命令行任务。 单击[链接](https://realpython.com/bonus/python-bindings-code/)可以获取在本教程中看到的所有代码。

现在选择您喜欢的工具并开始构建这些 Python 绑定吧！特别感谢 Loic Domaigne 对本教程的额外技术评论。
