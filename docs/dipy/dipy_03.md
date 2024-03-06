# 第二章 第一个 Python 程序

# 第二章 第一个 Python 程序

*   2.1\. 概览
*   2.2\. 函数声明
    *   2.2.1\. Python 和其他编程语言数据类型的比较
*   2.3\. 文档化函数
*   2.4\. 万物皆对象
    *   2.4.1\. 模块导入的搜索路径
    *   2.4.2\. 何谓对象？
*   2.5\. 代码缩进
*   2.6\. 测试模块

大家都很清楚，其他书籍是如何一步步从编程基础讲述到构建完整的可运行程序的，但还是让我们跳过这个部分吧！

# 2.1\. 概览

# 2.1\. 概览

这是一个完整的、可执行的 Python 程序。

它可能对您来说根本无法理解。别着急，我们将逐行地进行剖析。不过首先把代码通读一遍，看一看是否有些可以理解的内容。

## 例 2.1\. `odbchelper.py`

如果您还没有下载本书附带的样例程序, 可以 [下载本程序和其他样例程序](http://www.woodpecker.org.cn/diveintopython/download/diveintopython-exampleszh-cn-5.4b.zip "Download example scripts")。

```py
 def buildConnectionString(params):
    """Build a connection string from a dictionary of parameters.
    Returns string."""
    return ";".join(["%s=%s" % (k, v) for k, v in params.items()])
if __name__ == "__main__":
    myParams = {"server":"mpilgrim", \
                "database":"master", \
                "uid":"sa", \
                "pwd":"secret" \
                }
    print buildConnectionString(myParams) 
```

现在运行一下这个程序，看一看结果是什么。

> 提示
> 在 Windows 的 ActivePython IDE 中，可以选择 File->Run... (****Ctrl**-R**) 来运行 Python 程序。输出结果将显示在交互窗口中。
> 
> 提示
> 在 Mac OS 的 Python IDE 中，可以选择 Python->Run window... (****Cmd**-R**) 来运行 Python 程序，但首先要设置一个重要的选项。在 IDE 中打开 `.py` 模块，点击窗口右上角的黑色三角，弹出这个模块的选项菜单，然后将 Run as **main** 选中。 这个设置是同模块一同保存的，所以对于每个模块您都需要这样做。
> 
> 提示
> 在 UNIX 兼容的操作系统中 (包括 Mac OS X)，可以通过命令行：**`python`odbchelper.py``** 运行模块。

`odbchelper.py` 的输出结果：

```py
server=mpilgrim;uid=sa;database=master;pwd=secret 
```

# 2.2\. 函数声明

# 2.2\. 函数声明

*   2.2.1\. Python 和其他编程语言数据类型的比较

与其它大多数语言一样 Python 有函数，但是它没有像 C++ 一样的独立的头文件；或者像 Pascal 一样的分离的 `interface`/`implementation` 段。在需要函数时，像下面这样声明即可：

```py
 def buildConnectionString(params): 
```

首先，函数声明以关键字 `def` 开始，接着为函数名，再往后为参数，参数放在小括号里。多个参数之间 (这里没有演示)用逗号分隔。

其次，函数没有定义返回的数据类型。Python 不需要指定返回值的数据类型；甚至不需要指定是否有返回值。实际上，每个 Python 函数都返回一个值；如果函数执行过 `return` 语句，它将返回指定的值，否则将返回 `None` (Python 的空值)。

> 注意
> 在 Visual Basic 中，函数 (有返回值) 以 `function` 开始，而子程序 (无返回值) 以 `sub` 开始。在 Python 中没有子程序。只有函数，所有的函数都有返回值 (尽管可能为 `None`)，并且所有的函数都以 `def` 开始。

最后需要指出的是，在 Python 中参数，`params` 不需要指定数据类型。Python 会判定一个变量是什么类型，并在内部将其记录下来。

> 注意
> 在 Java、C++ 和其他静态类型语言中，必须要指定函数返回值和每个函数参数的数据类型。在 Python 中，永远也不需要明确指定任何东西的数据类型。Python 会根据赋给它的值在内部将其数据类型记录下来。

## 2.2.1\. Python 和其他编程语言数据类型的比较

一位博学的读者发给我 Python 如何与其它编程语言的比较的解释：

静态类型语言

一种在编译期间就确定数据类型的语言。大多数静态类型语言是通过要求在使用任一变量之前声明其数据类型来保证这一点的。Java 和 C 是静态类型语言。

动态类型语言

一种在运行期间才去确定数据类型的语言，与静态类型相反。VBScript 和 Python 是动态类型的，因为它们确定一个变量的类型是在您第一次给它赋值的时候。

强类型语言

一种总是强制类型定义的语言。Java 和 Python 是强制类型定义的。您有一个整数，如果不明确地进行转换 ，不能将把它当成一个字符串。

弱类型语言

一种类型可以被忽略的语言，与强类型相反。VBScript 是弱类型的。在 VBScript 中，您可以将字符串 `'12'` 和整数 `3` 进行连接得到字符串`'123'`，然后可以把它看成整数 `123` ，所有这些都不需要任何的显示转换。

所以说 Python 既是*动态类型语言* (因为它不使用显示数据类型声明)，又是*强类型语言* (因为只要一个变量获得了一个数据类型，它实际上就一直是这个类型了)。

# 2.3\. 文档化函数

# 2.3\. 文档化函数

可以通过给出一个 `doc string` (文档字符串) 来文档化一个 Python 函数。

## 例 2.2\. 定义 `buildConnectionString` 函数的 `doc string`

```py
 def buildConnectionString(params):
    """Build a connection string from a dictionary of parameters.
    Returns string.""" 
```

三重引号表示一个多行字符串。在开始与结束引号间的所有东西都被视为单个字符串的一部分，包括硬回车和其它的引号字符。您可以在任何地方使用它们，但是您可能会发现，它们经常被用于定义 `doc string`。

> 注意
> 三重引号也是一种定义既包含单引号又包含双引号的字符串的简单方法，就像 Perl 中的 `qq/.../` 。

在三重引号中的任何东西都是这个函数的 `doc string`，它们用来说明函数可以做什么。如果存在 `doc string`，它必须是一个函数要定义的第一个内容 (也就是说，在冒号后面的第一个内容)。在技术上不要求给出函数的 `doc string`，但是您应该这样做。我相信在您上过的每一种编程课上都听到过这一点，但是 Python 带给您一些额外的动机：`doc string` 在运行时可作为函数的属性。

> 注意
> 许多 Python IDE 使用 `doc string` 来提供上下文敏感的文档信息，所以当键入一个函数名时，它的 `doc string` 显示为一个工具提示。这一点可以说非常有用，但是它的好坏取决于您书写的 `doc string` 的好坏。

## 进一步阅读

*   PEP 257 定义了 `doc string` 规范。
*   *Python Style Guide* 讨论了如何编写一个好的 `doc string`。
*   *Python Tutorial* 讨论了[在 `doc string` 中如何使用空白](http://www.python.org/doc/current/tut/node6.html#SECTION006750000000000000000)。

# 2.4\. 万物皆对象

# 2.4\. 万物皆对象

*   2.4.1\. 模块导入的搜索路径
*   2.4.2\. 何谓对象？

也许您没在意，我刚才的意思是 Python 函数有属性，并且这些属性在运行时是可用的。

在 Python 中，函数同其它东西一样也是对象。

打开您习惯使用的 Python IDE 执行如下的操作：

## 例 2.3\. 访问 `buildConnectionString` 函数的 `doc string`

```py
>>> import odbchelper                              
>>> params = {"server":"mpilgrim", "database":"master", "uid":"sa", "pwd":"secret"}
>>> print odbchelper.buildConnectionString(params) 
server=mpilgrim;uid=sa;database=master;pwd=secret
>>> print odbchelper.buildConnectionString.__doc__ 
Build a connection string from a dictionary
Returns string. 
```

|  |  |
| --- | --- |
| [1] | 第一行将 `odbchelper` 程序作为模块导入。模块是指一个可以交互使用，或者从另一 Python 程序访问的代码段。(您在 第四章 将会看到多模块 Python 程序的许多例子。) 只要导入了一个模块，就可以引用它的任何公共的函数、类或属性。模块可以通过这种方法来使用其它模块的功能，您也可以在 IDE 中这样做。这是一个很重要的概念，在后面我们将谈得更多。 |
| [2] | 当使用在被导入模块中定义的函数时，必须包含模块的名字。所以不能只使用 `buildConnectionString`，而应该使用 `odbchelper.buildConnectionString`。如果您用过 Java 的类，对此应该不感到陌生。 |
| [3] | 访问函数的 `__doc__` 属性不像您想象的那样是通过函数调用。 |

> 注意
> 在 Python 中的 `import` 就像 Perl 中的 `require`。`import` 一个 Python 模块后，您就可以使用 `_module_._function_` 来访问它的函数；`require` 一个 Perl 模块后，您就可以使用 `_module_::_function_` 来访问它的函数。

## 2.4.1\. 模块导入的搜索路径

在我们继续之前，我想简要地提一下库的搜索路径。当导入一个模块时，Python 在几个地方进行搜索。明确地，它会对定义在 `sys.path` 中的目录逐个进行搜索。它只是一个 list (列表)，您可以容易地查看它或通过标准的 list 方法来修改它。(在本章的后面我们将学习更多关于 list 的知识。)

## 例 2.4\. 模块导入的搜索路径

```py
>>> import sys                 
>>> sys.path                   
['', '/usr/local/lib/python2.2', '/usr/local/lib/python2.2/plat-linux2',
'/usr/local/lib/python2.2/lib-dynload', '/usr/local/lib/python2.2/site-packages',
'/usr/local/lib/python2.2/site-packages/PIL', '/usr/local/lib/python2.2/site-packages/piddle']
>>> sys                        
<module 'sys' (built-in)>
>>> sys.path.append('/my/new/path') 
```

|  |  |
| --- | --- |
| [1] | 导入 `sys` 模块，使得它的所有函数和属性都有效。 |
| [2] | `sys.path` 是一个指定当前搜索路径的目录列表。(您的输出结果可能有所不同，这取决于您的操作系统、正在运行的 Python 版本和初始安装的位置。)Python 将搜索这些目录 (按顺序) 来查找一个与您正试着导入的模块名相匹配的 `.py` 文件。 |
| [3] | 实际上，我没说实话。真实情况要比这更复杂，因为不是所有的模块都保存为 `.py` 文件。有一些模块 (像 `sys`)，是“内置模块”，它们实际上是置于 Python 内部的。内置模块的行为如同一般的模块，但是它们的 Python 源代码是不可用的，因为它们不是用 Python 写的！(`sys` 模块是用 C 写的。) |
| [4] | 在运行时，通过向 `sys.path` 追加目录名，就可以在 Python 的搜索路径中增加新的目录，然后当您导入模块时，Python 也会在那个目录中进行搜索。这个作用在 Python 运行时一直生效。(在 第三章 我们将讨论更多的关于 `append` 和其它的 list 方法。) |

## 2.4.2\. 何谓对象？

在 Python 中一切都是对象，并且几乎一切都有属性和方法。所有的函数都有一个内置的 `__doc__` 属性，它会返回在函数源代码中定义的 `doc string`；`sys` 模块是一个对象，它有一个叫作 `path` 的属性；等等。

我们仍然在回避问题的实质，究竟何谓对象？不同的编程语言以不同的方式定义 “对象” 。 某些语言中，它意味着*所有* 对象*必须* 有属性和方法；另一些语言中，它意味着所有的对象都可以子类化。在 Python 中，定义是松散的；某些对象既没有属性也没有方法 (关于这一点的说明在 第三章)，而且不是所有的对象都可以子类化 (关于这一点的说明在第五章)。但是万物皆对象从感性上可以解释为：一切都可以赋值给变量或作为参数传递给函数 (关于这一点的说明在第四章)。

这一点太重要了，所以我会在刚开始就不止一次地反复强调它，以免您没注意到：在 Python 中*万物皆对象*。字符串是对象。列表是对象。函数是对象。甚至模块也是对象，这一点我们很快会看到。

## 进一步阅读

*   *Python Reference Manual* 确切解释了[在 Python 中万物皆对象的含义](http://www.python.org/doc/current/ref/objects.html)，因为有些书生气十足的人，喜欢花时间讨论这类的问题。
*   eff-bot 总结了 [Python 对象](http://www.effbot.org/guides/python-objects.htm).

# 2.5\. 代码缩进

# 2.5\. 代码缩进

Python 函数没有明显的 `begin` 和 `end`，没有标明函数的开始和结束的花括号。唯一的分隔符是一个冒号 (`:`)，接着代码本身是缩进的。

## 例 2.5\. 缩进 `buildConnectionString` 函数

```py
 def buildConnectionString(params):
    """Build a connection string from a dictionary of parameters.
    Returns string."""
    return ";".join(["%s=%s" % (k, v) for k, v in params.items()]) 
```

代码块是通过它们的缩进来定义的。我所说的“代码块”是指：函数、`if` 语句、`for` 循环、`while` 循环，等等。开始缩进表示块的开始，取消缩进表示块的结束。不存在明显的括号，大括号或关键字。这就意味着空白是重要的，并且要一致。在这个例子中，函数代码 (包括 `doc string`) 缩进了 4 个空格。不一定非要是 4 个，只要一致就可以了。没有缩进的第一行则被视为在函数体之外。

例 2.6 “if 语句” 展示了一个 `if` 语句缩进的例子。

## 例 2.6\. `if` 语句

```py
 def fib(n):                   
    print 'n =', n            
    if n > 1:                 
        return n * fib(n - 1)
    else:                     
        print 'end of the line'
        return 1 
```

|  |  |
| --- | --- |
| [1] | 这是一个名为 `fib` 的函数，有一个参数 `n`。在函数内的所有代码都是缩进的。 |
| [2] | 在 Python 中向屏幕输出内容非常容易，只要使用 `print` 即可。`print` 语句可以接受任何数据类型，包括字符串、整数和其它类型，如字典和列表 (我们将在下一章学习)。甚至可以混在一起输出，只需用逗号隔开。所有值都输出到同一行，用空格隔开 (逗号并不打印出来)。所以当用 `5` 来调用 `fib` 时，将输出“n = 5”。 |
| [3] | `if` 语句是一种的代码块。如果 `if` 表达式计算为 true，紧跟着的缩进块会被执行，否则进入 `else` 块执行。 |
| [4] | 当然 `if` 和 `else` 块可以包含许多行，只要它们都同样缩进。这个 `else` 块中有两行代码。对于多行代码块没有其它特殊的语法，只要缩进就行了。 |

在经过一些最初的抗议和几个与 Fortran 的嘲讽的类比之后，您会心平气和地对待代码缩进，并且开始看到它的好处。一个主要的好处就是所有的 Python 程序看上去都差不多，因为缩进是一种语言的要求而不是一种风格。这样就使得阅读和理解他人的 Python 代码容易得多。

> 注意
> Python 使用硬回车来分割语句，冒号和缩进来分割代码块。C++ 和 Java 使用分号来分割语句，花括号来分割代码块。

## 进一步阅读

*   *Python Reference Manual* 讨论了交叉缩进问题，并且[演示了各种各样的缩进错误](http://www.python.org/doc/current/ref/indentation.html)。
*   *Python Style Guide* 讨论了良好的缩进风格。

# 2.6\. 测试模块

# 2.6\. 测试模块

所有的 Python 模块都是对象，并且有几个有用的属性。您可以使用这些属性方便地测试您所编写的模块。下面是一个使用 `if` `__name__` 的技巧。

```py
 if __name__ == "__main__": 
```

在继续学习新东西之前，有几个重要的观察结果。首先，`if` 表达式无需使用圆括号括起来。其次，`if` 语句以冒号结束，紧跟其后的是缩进代码。

> 注意
> 与 C 一样，Python 使用 `==` 做比较，使用 `=` 做赋值。与 C 不一样，Python 不支持行内赋值，所以不会出现想要进行比较却意外地出现赋值的情况。

那么为什么说这个特殊的 `if` 语句是一个技巧呢？模块是对象，并且所有的模块都有一个内置属性 `__name__`。一个模块的 `__name__` 的值取决于您如何应用模块。如果 `import` 模块，那么 `__name__` 的值通常为模块的文件名，不带路径或者文件扩展名。但是您也可以像一个标准的程序一样直接运行模块，在这种情况下 `__name__` 的值将是一个特别的缺省值，`__main__`。

```py
>>> import odbchelper
>>> odbchelper.`__name__`
'odbchelper' 
```

只要了解到这一点，您就可以在模块内部为您的模块设计一个测试套件，在其中加入这个 `if` 语句。当您直接运行模块，`__name__` 的值是 `__main__`，所以测试套件执行。当您导入模块，`__name__` 的值就是别的东西了，所以测试套件被忽略。这样使得在将新的模块集成到一个大程序之前开发和调试容易多了。

> 提示
> 在 MacPython 上，需要一个额外的步聚来使得 `if` `__name__` 技巧有效。点击窗口右上角的黑色三角，弹出模块的属性菜单，确认 Run as **main** 被选中。

## 进一步阅读

*   *Python Reference Manual* 讨论了[导入模块](http://www.python.org/doc/current/ref/import.html)的底层细节。