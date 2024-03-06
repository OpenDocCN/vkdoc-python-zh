# 第四章 自省的威力

# 第四章 自省的威力

*   4.1\. 概览
*   4.2\. 使用可选参数和命名参数
*   4.3\. 使用 type、str、dir 和其它内置函数
    *   4.3.1\. type 函数
    *   4.3.2\. str 函数
    *   4.3.3\. 内置函数
*   4.4\. 通过 getattr 获取对象引用
    *   4.4.1\. 用于模块的 getattr
    *   4.4.2\. getattr 作为一个分发者
*   4.5\. 过滤列表
*   4.6\. and 和 or 的特殊性质
    *   4.6.1\. 使用 and-or 技巧
*   4.7\. 使用 lambda 函数
    *   4.7.1\. 真实世界中的 lambda 函数
*   4.8\. 全部放在一起
*   4.9\. 小结

本章论述了 Python 众多强大功能之一：自省。正如你所知道的，Python 中万物皆对象，自省是指代码可以查看内存中以对象形式存在的其它模块和函数，获取它们的信息，并对它们进行操作。用这种方法，你可以定义没有名称的函数，不按函数声明的参数顺序调用函数，甚至引用事先并不知道名称的函数。

# 4.1\. 概览

# 4.1\. 概览

下面是一个完整可运行的 Python 程序。大概看一下这段程序，你应该可以理解不少了。用数字标出的行阐述了 第二章 *第一个 Python 程序* 中涉及的一些概念。如果剩下来的代码看起来有点奇怪，不用担心，通过阅读本章你将会理解所有这些。

## 例 4.1\. `apihelper.py`

如果您还没有下载本书附带的样例程序, 可以 [下载本程序和其他样例程序](http://www.woodpecker.org.cn/diveintopython/download/diveintopython-exampleszh-cn-5.4b.zip "Download example scripts")。

```py
 def info(object, spacing=10, collapse=1):   
    """Print methods and doc strings.
    Takes module, class, list, dictionary, or string."""
    methodList = [method for method in dir(object) if callable(getattr(object, method))]
    processFunc = collapse and (lambda s: " ".join(s.split())) or (lambda s: s)
    print "\n".join(["%s %s" %
                      (method.ljust(spacing),
                       processFunc(str(getattr(object, method).__doc__)))
                     for method in methodList])
if __name__ == "__main__":                 
    print info.__doc__ 
```

|  |  |
| --- | --- |
| [1] | 该模块有一个声明为 `info` 的函数。根据它的函数声明可知，它有三个参数： `object`、`spacing` 和 `collapse`。实际上后面两个参数都是可选参数，关于这点你很快就会看到。 |
| [2] | `info` 函数有一个多行的 `doc string`，简要地描述了函数的功能。注意这里并没有提到返回值；单独使用这个函数只是为了这个函数产生的效果，并不是为了它的返回值。 |
| [3] | 函数内的代码是缩进形式的。 |
| [4] | `if __name__` 技巧允许这个程序在自己独立运行时做些有用的事情，同时又不妨碍作为其它程序的模块使用。在这个例子中，程序只是简单地打印出 `info` 函数的 `doc string`。 |
| [5] | `if` 语句使用 `==` 进行比较，而且不需要括号。 |

`info` 函数的设计意图是提供给工作在 Python IDE 中的开发人员使用，它可以接受任何含有函数或者方法的对象 (比如模块，含有函数，又比如 list，含有方法) 作为参数，并打印出对象的所有函数和它们的 `doc string`。

## 例 4.2\. `apihelper.py` 的用法示例

```py
>>> from apihelper import info
>>> li = []
>>> info(li)
append     L.append(object) -- append object to end
count      L.count(value) -> integer -- return number of occurrences of value
extend     L.extend(list) -- extend list by appending list elements
index      L.index(value) -> integer -- return index of first occurrence of value
insert     L.insert(index, object) -- insert object before index
pop        L.pop([index]) -> item -- remove and return item at index (default last)
remove     L.remove(value) -- remove first occurrence of value
reverse    L.reverse() -- reverse *IN PLACE*
sort       L.sort([cmpfunc]) -- sort *IN PLACE*; if given, cmpfunc(x, y) -> -1, 0, 1 
```

缺省地，程序输出进行了格式化处理，以使其易于阅读。多行 `doc string` 被合并到单行中，要改变这个选项需要指定 *`collapse`* 参数的值为 `0`。如果函数名称长于 10 个字符，你可以将 *`spacing`* 参数的值指定为更大的值以使输出更容易阅读。

## 例 4.3\. `apihelper.py` 的高级用法

```py
>>> import odbchelper
>>> info(odbchelper)
buildConnectionString Build a connection string from a dictionary Returns string.
>>> info(odbchelper, 30)
buildConnectionString          Build a connection string from a dictionary Returns string.
>>> info(odbchelper, 30, 0)
buildConnectionString          Build a connection string from a dictionary
    Returns string. 
```

# 4.2\. 使用可选参数和命名参数

# 4.2\. 使用可选参数和命名参数

Python 允许函数参数有缺省值；如果调用函数时不使用参数，参数将获得它的缺省值。此外，通过使用命名参数还可以以任意顺序指定参数。SQL Server Transact/SQL 中的存储过程也可以做到这些；如果你是脚本高手，你可以略过这部分。

`info` 函数就是这样一个例子，它有两个可选参数。

```py
 def info(object, spacing=10, collapse=1): 
```

`spacing` 和 `collapse` 是可选参数，因为它们已经定义了缺省值。`object` 是必备参数，因为它没有指定缺省值。如果调用 `info` 时只指定一个参数，那么 `spacing` 缺省为 `10` ，`collapse` 缺省为 `1`。如果调用 `info` 时指定两个参数，`collapse` 依然默认为 `1`。

假如你要指定 `collapse` 的值，但是又想要接受 `spacing` 的缺省值。在绝大部分语言中，你可能运气就不太好了，因为你需要使用三个参数来调用函数，这势必要重新指定 `spacing` 的值。但是在 Python 中，参数可以通过名称以任意顺序指定。

## 例 4.4\. `info` 的有效调用

```py
info(odbchelper)                    
info(odbchelper, 12)                
info(odbchelper, collapse=0)        
info(spacing=15, object=odbchelper) 
```

|  |  |
| --- | --- |
| [1] | 只使用一个参数，`spacing` 使用缺省值 `10` ，`collapse` 使用缺省值 `1`。 |
| [2] | 使用两个参数，`collapse` 使用缺省值 `1`。 |
| [3] | 这里你显式命名了 `collapse` 并指定了它的值。`spacing` 将依然使用它的缺省值 `10`。 |
| [4] | 甚至必备参数 (例如 `object`，没有指定缺省值) 也可以采用命名参数的方式，而且命名参数可以以任意顺序出现。 |

这些看上去非常累，除非你意识到参数不过是一个字典。“通常” 不使用参数名称的函数调用只是一个简写的形式，Python 按照函数声明中定义的的参数顺序将参数值和参数名称匹配起来。大部分时间，你会使用“通常”方式调用函数，但是如果你需要，总是可以提供附加的灵活性。

> 注意
> 调用函数时唯一必须做的事情就是为每一个必备参数指定值 (以某种方式)；以何种具体的方式和顺序都取决于你。

## 进一步阅读

*   *Python Tutorial* 确切地讨论了[何时、如何进行缺省参数赋值](http://www.python.org/doc/current/tut/node6.html#SECTION006710000000000000000)，这都和缺省值是一个 list 还是一个具有副作用的表达式有关。

# 4.3\. 使用 `type`、`str`、`dir` 和其它内置函数

# 4.3\. 使用 `type`、`str`、`dir` 和其它内置函数

*   4.3.1\. type 函数
*   4.3.2\. str 函数
*   4.3.3\. 内置函数

Python 有小部分相当有用的内置函数。除这些函数之外，其它所有的函数都被分到了各个模块中。其实这是一个非常明智的设计策略，避免了核心语言变得像其它脚本语言一样臃肿 (咳 咳，Visual Basic)。

## 4.3.1\. `type` 函数

`type` 函数返回任意对象的数据类型。在 `types` 模块中列出了可能的数据类型。这对于处理多种数据类型的帮助者函数 [1] 非常有用。

## 例 4.5\. `type` 介绍

```py
>>> type(1)           
<type 'int'>
>>> li = []
>>> type(li)          
<type 'list'>
>>> import odbchelper
>>> type(odbchelper)  
<type 'module'>
>>> import types      
>>> type(odbchelper) == types.ModuleType
True 
```

|  |  |
| --- | --- |
| [1] | `type` 可以接收任何东西作为参数――我的意思是任何东西――并返回它的数据类型。整型、字符串、列表、字典、元组、函数、类、模块，甚至类型对象都可以作为参数被 `type` 函数接受。 |
| [2] | `type` 可以接收变量作为参数，并返回它的数据类型。 |
| [3] | `type` 还可以作用于模块。 |
| [4] | 你可以使用 `types` 模块中的常量来进行对象类型的比较。这就是 `info` 函数所做的，很快你就会看到。 |

## 4.3.2\. `str` 函数

`str` 将数据强制转换为字符串。每种数据类型都可以强制转换为字符串。

## 例 4.6\. `str` 介绍

```py
>>> str(1)          
'1'
>>> horsemen = ['war', 'pestilence', 'famine']
>>> horsemen
['war', 'pestilence', 'famine']
>>> horsemen.append('Powerbuilder')
>>> str(horsemen)   
"['war', 'pestilence', 'famine', 'Powerbuilder']"
>>> str(odbchelper) 
"<module 'odbchelper' from 'c:\\docbook\\dip\\py\\odbchelper.py'>"
>>> str(None)       
'None' 
```

|  |  |
| --- | --- |
| [1] | 对于简单的数据类型比如整型，你可以预料到 `str` 的正常工作，因为几乎每种语言都有一个将整型转化为字符串的函数。 |
| [2] | 然而 `str` 可以作用于任何数据类型的任何对象。这里它作用于一个零碎构建的列表。 |
| [3] | `str` 还允许作用于模块。注意模块的字符串形式表示包含了模块在磁盘上的路径名，所以你的显示结果将会有所不同。 |
| [4] | `str` 的一个细小但重要的行为是它可以作用于 `None`，`None` 是 Python 的 null 值。这个调用返回字符串 `'None'`。你将会使用这一点来改进你的 `info` 函数，这一点你很快就会看到。 |

`info` 函数的核心是强大的 `dir` 函数。`dir` 函数返回任意对象的属性和方法列表，包括模块对象、函数对象、字符串对象、列表对象、字典对象 …… 相当多的东西。

## 例 4.7\. `dir` 介绍

```py
>>> li = []
>>> dir(li)           
['append', 'count', 'extend', 'index', 'insert',
'pop', 'remove', 'reverse', 'sort']
>>> d = {}
>>> dir(d)            
['clear', 'copy', 'get', 'has_key', 'items', 'keys', 'setdefault', 'update', 'values']
>>> import odbchelper
>>> dir(odbchelper)   
['__builtins__', '__doc__', '__file__', '__name__', 'buildConnectionString'] 
```

|  |  |
| --- | --- |
| [1] | `li` 是一个列表，所以 ``dir`(`li`)` 返回一个包含所有列表方法的列表。注意返回的列表只包含了字符串形式的方法名称，而不是方法对象本身。 |
| [2] | `d` 是一个字典，所以 ``dir`(`d`)`返回字典方法的名称列表。其中至少有一个方法，`keys`，看起来还是挺熟悉的。 |
| [3] | 这里就是真正变得有趣的地方。`odbchelper` 是一个模块，所以 ``dir`(`odbchelper`)`返回模块中定义的所有部件的列表，包括内置的属性，例如 `**name**`、`**doc**`，以及其它你所定义的属性和方法。在这个例子中，`odbchelper`只有一个用户定义的方法，就是在第二章中论述的`buildConnectionString` 函数。 |

最后是 `callable` 函数，它接收任何对象作为参数，如果参数对象是可调用的，返回 `True`；否则返回 `False`。可调用对象包括函数、类方法，甚至类自身 (下一章将更多的关注类)。

## 例 4.8\. `callable` 介绍

```py
>>> import string
>>> string.punctuation           
'!"#$%&\'()*+,-./:;<=>?@[\\]^_`{|}~'
>>> string.join                  
<function join at 00C55A7C>
>>> callable(string.punctuation) 
False
>>> callable(string.join)        
True
>>> print string.join.__doc__    
join(list [,sep]) -> string
    Return a string composed of the words in list, with
    intervening occurrences of sep.  The default separator is a
    single space.
    (joinfields and join are synonymous) 
```

|  |  |
| --- | --- |
| [1] | `string` 模块中的函数现在已经不赞成使用了 (尽管很多人现在仍然还在使用 `join` 函数)，但是在这个模块中包含了许多有用的变量，例如 `string.punctuation`，这个字符串包含了所有标准的标点符号字符。 |
| [2] | `string.join` 是一个用于连接字符串列表的函数。 |
| [3] | `string.punctuation` 是不可调用的对象；它是一个字符串。(字符串确有可调用的方法，但是字符串本身不是可调用的。) |
| [4] | `string.join` 是可调用的；这个函数可以接受两个参数。 |
| [5] | 任何可调用的对象都有 `doc string`。通过将 `callable` 函数作用于一个对象的每个属性，可以确定哪些属性 (方法、函数、类) 是你要关注的，哪些属性 (常量等等) 是你可以忽略、之前不需要知道的。 |

## 4.3.3\. 内置函数

`type`、`str`、`dir` 和其它的 Python 内置函数都归组到了 `__builtin__` (前后分别是双下划线) 这个特殊的模块中。如果有帮助的话，你可以认为 Python 在启动时自动执行了 `from __builtin__ import *`，此语句将所有的 “内置” 函数导入该命名空间，所以在这个命名空间中可以直接使用这些内置函数。

像这样考虑的好处是，你是可以获取 `__builtin__` 模块信息的，并以组的形式访问所有的内置函数和属性。猜到什么了吗，现在我们的 Python 有一个称为 `info` 的函数。自己尝试一下，略看一下结果列表。后面我们将深入到一些更重要的函数。(一些内置的错误类，比如 `AttributeError`，应该看上去已经很熟悉了。)

## 例 4.9\. 内置属性和内置函数

```py
>>> from apihelper import info
>>> import __builtin__
>>> info(__builtin__, 20)
ArithmeticError      Base class for arithmetic errors.
AssertionError       Assertion failed.
AttributeError       Attribute not found.
EOFError             Read beyond end of file.
EnvironmentError     Base class for I/O related errors.
Exception            Common base class for all exceptions.
FloatingPointError   Floating point operation failed.
IOError              I/O operation failed.
[...snip...] 
```

> 注意
> Python 提供了很多出色的参考手册，你应该好好地精读一下所有 Python 提供的必备模块。对于其它大部分语言，你会发现自己要常常回头参考手册或者 man 页来提醒自己如何使用这些模块，但是 Python 不同于此，它很大程度上是自文档化的。

## 进一步阅读

*   *Python Library Reference* 对[所有的内置函数](http://www.python.org/doc/current/lib/built-in-funcs.html)和[所有的内置异常](http://www.python.org/doc/current/lib/module-exceptions.html)都进行了文档化。

## Footnotes

[1] 帮助者函数，原文是 helper function，也就是我们在前文所看到的诸如 `odbchelper`、`apihelper` 这样的函数。――译注

# 4.4\. 通过 `getattr` 获取对象引用

# 4.4\. 通过 `getattr` 获取对象引用

*   4.4.1\. 用于模块的 getattr
*   4.4.2\. getattr 作为一个分发者

你已经知道 Python 函数是对象。你不知道的是，使用 `getattr` 函数，可以得到一个直到运行时才知道名称的函数的引用。

## 例 4.10\. `getattr` 介绍

```py
>>> li = ["Larry", "Curly"]
>>> li.pop                       
<built-in method pop of list object at 010DF884>
>>> getattr(li, "pop")           
<built-in method pop of list object at 010DF884>
>>> getattr(li, "append")("Moe") 
>>> li
["Larry", "Curly", "Moe"]
>>> getattr({}, "clear")         
<built-in method clear of dictionary object at 00F113D4>
>>> getattr((), "pop")           
Traceback (innermost last):
  File "<interactive input>", line 1, in ?
AttributeError: 'tuple' object has no attribute 'pop' 
```

|  |  |
| --- | --- |
| [1] | 该语句获取列表的 `pop` 方法的引用。注意该语句并不是调用 `pop` 方法；调用 `pop` 方法的应该是 `li.pop()`。这里指的是方法对象本身。 |
| [2] | 该语句也是返回 `pop` 方法的引用，但是此时，方法名称是作为一个字符串参数传递给 `getattr` 函数的。`getattr` 是一个有用到令人无法致信的内置函数，可以返回任何对象的任何属性。在这个例子中，对象是一个 list，属性是 `pop` 方法。 |
| [3] | 如果不确信它是多么的有用，试试这个：`getattr` 的返回值*是* 方法，然后你就可以调用它，就像直接使用 `li.append("Moe")` 一样。但是实际上你没有直接调用函数；只是以字符串形式指定了函数名称。 |
| [4] | `getattr` 也可以作用于字典。 |
| [5] | 理论上，`getattr` 可以作用于元组，但是由于元组没有方法，所以不管你指定什么属性名称 `getattr` 都会引发一个异常。 |

## 4.4.1\. 用于模块的 `getattr`

`getattr` 不仅仅适用于内置数据类型，也可作用于模块。

## 例 4.11\. `apihelper.py` 中的 `getattr` 函数

```py
>>> import odbchelper
>>> odbchelper.buildConnectionString             
<function buildConnectionString at 00D18DD4>
>>> getattr(odbchelper, "buildConnectionString") 
<function buildConnectionString at 00D18DD4>
>>> object = odbchelper
>>> method = "buildConnectionString"
>>> getattr(object, method)                      
<function buildConnectionString at 00D18DD4>
>>> type(getattr(object, method))                
<type 'function'>
>>> import types
>>> type(getattr(object, method)) == types.FunctionType
True
>>> callable(getattr(object, method))            
True 
```

|  |  |
| --- | --- |
| [1] | 该语句返回 `odbchelper` 模块中 `buildConnectionString` 函数的引用，第二章 *第一个 Python 程序* 你已经研习过这个方法了。(你看到的这个十六进制地址是我机器上的；你的输出结果会有所不同。) |
| [2] | 使用 `getattr`，你能够获得同一函数的同一引用。通常，``getattr`(*object*, "*attribute*")`等价于`*object*.*attribute*`。如果 _`object`_ 是一个模块的话，那么 _`attribute`_ 可能是定义在模块中的任何东西：函数、类或者全局变量。 |
| [3] | 接下来的是你真正用在 `info` 函数中的东西。`object` 作为一个参数传递给函数； `method` 是方法或者函数的名称字符串。 |
| [4] | 在这个例子中，`method` 是函数的名称，通过获取 `type` 可以进行验证。 |
| [5] | 由于 `method` 是一个函数，所以它是可调用的。 |

## 4.4.2\. `getattr` 作为一个分发者

`getattr` 常见的使用模式是作为一个分发者。举个例子，如果你有一个程序可以以不同的格式输出数据，你可以为每种输出格式定义各自的格式输出函数，然后使用唯一的分发函数调用所需的格式输出函数。

例如，让我们假设有一个以 HTML、XML 和普通文本格式打印站点统计的程序。输出格式在命令行中指定，或者保存在配置文件中。`statsout` 模块定义了三个函数：`output_html`、`output_xml` 和 `output_text`。然后主程序定义了唯一的输出函数，如下：

## 例 4.12\. 使用`getattr` 创建分发者

```py
 import statsout
def output(data, format="text"):                              
    output_function = getattr(statsout, "output_%s" % format) 
    return output_function(data) 
```

|  |  |
| --- | --- |
| [1] | `output` 函数接收一个必备参数 `data`，和一个可选参数 `format`。如果没有指定 `format` 参数，其缺省值是 `text` 并完成普通文本输出函数的调用。 |
| [2] | 你可以连接 `format` 参数值和 "output_" 来创建一个函数名称作为参数值，然后从 `statsout` 模块中取得该函数。这种方式允许今后很容易地扩展程序以支持其它的输出格式，而且无需修改分发函数。所要做的仅仅是向 `statsout` 中添加一个函数，比如 `output_pdf`，之后只要将 “pdf” 作为 `format` 的参数值传递给 `output` 函数即可。 |
| [3] | 现在你可以简单地调用输出函数，就像调用其它函数一样。`output_function` 变量是指向 `statsout` 模块中相应函数的引用。 |

你是否发现前面示例的一个 Bug？即字符串和函数之间的松耦合，而且没有错误检查。如果用户传入一个格式参数，但是在 `statsout` 中没有定义相应的格式输出函数，会发生什么呢？还好，`getattr` 会返回 `None`，它会取代一个有效函数并被赋值给 `output_function`，然后下一行调用函数的语句将会失败并抛出一个异常。这种方式不好。

值得庆幸的是，`getattr` 能够使用可选的第三个参数，一个缺省返回值。

## 例 4.13\. `getattr` 缺省值

```py
 import statsout
def output(data, format="text"):
    output_function = getattr(statsout, "output_%s" % format, statsout.output_text)
    return output_function(data) 
```

|  |  |
| --- | --- |
| [1] | 这个函数调用一定可以工作，因为你在调用 `getattr` 时添加了第三个参数。第三个参数是一个缺省返回值，如果第二个参数指定的属性或者方法没能找到，则将返回这个缺省返回值。 |

正如你所看到，`getattr` 是相当强大的。它是自省的核心，在后面的章节中你将看到它更强大的示例。

# 4.5\. 过滤列表

# 4.5\. 过滤列表

如你所知，Python 具有通过列表解析 (第 3.6 节 “映射 list”) 将列表映射到其它列表的强大能力。这种能力同过滤机制结合使用，使列表中的有些元素被映射的同时跳过另外一些元素。

过滤列表语法：

```py
[`mapping-expression` for `element` in `source-list` if `filter-expression`] 
```

这是你所知所爱的列表解析的扩展。前三部分都是相同的；最后一部分，以 `if` 开头的是过滤器表达式。过滤器表达式可以是返回值为真或者假的任何表达式 (在 Python 中是几乎任何东西)。任何经过滤器表达式演算值为真的元素都可以包含在映射中。其它的元素都将忽略，它们不会进入映射表达式，更不会包含在输出列表中。

## 例 4.14\. 列表过滤介绍

```py
>>> li = ["a", "mpilgrim", "foo", "b", "c", "b", "d", "d"]
>>> [elem for elem in li if len(elem) > 1]       
['mpilgrim', 'foo']
>>> [elem for elem in li if elem != "b"]         
['a', 'mpilgrim', 'foo', 'c', 'd', 'd']
>>> [elem for elem in li if li.count(elem) == 1] 
['a', 'mpilgrim', 'foo', 'c'] 
```

|  |  |
| --- | --- |
| [1] | 这里的映射表达式很简单 (只是返回每个元素的值)，所以请把注意力集中到过滤器表达式上。由于 Python 会遍历整个列表，它将对每个元素执行过滤器表达式。如果过滤器表达式演算值为真，该元素就会被映射，同时映射表达式的结果将包含在返回的列表中。这里，你过滤掉了所有单字符的字符串，留下了一个由长字符串构成的列表。 |
| [2] | 这里你过滤掉了一个特定值 `b`。注意这个过滤器会过滤掉所有的 `b`，因为每次取出 `b`，过滤表达式都将为假。 |
| [3] | `count` 是一个列表方法，返回某个值在列表中出现的次数。你可以认为这个过滤器将从列表中剔除重复元素，返回一个只包含了在原始列表中有着唯一值拷贝的列表。但并非如此，因为在原始列表中出现两次的值 (在本例中，`b` 和 `d`) 被完全剔除了。从一个列表中排除重复值有多种方法，但过滤并不是其中的一种。 |

回到 `apihelper.py` 中的这一行：

```py
 methodList = [method for method in dir(object) if callable(getattr(object, method))] 
```

这行看上去挺复杂――确实也很复杂――但是基本结构都还是一样的。整个过滤表达式返回一个列表，并赋值给 `methodList` 变量。表达式的前半部分是列表映射部分。映射表达式是一个和遍历元素相同的表达式，因此它返回每个元素的值。``dir`(`object`)`返回`object`对象的属性和方法列表――你正在映射的列表。所以唯一新出现的部分就是在`if` 后面的过滤表达式。

过滤表达式看上去很恐怖，其实不是。你已经知道了 `callable`、`getattr` 和 `in`。正如你在前面的部分中看到的，如果 `object` 是一个模块，并且 `method` 是上述模块中某个函数的名称，那么表达式 `getattr(object, method)` 将返回一个函数对象。

所以这个表达式接收一个名为 `object` 的对象，然后得到它的属性、方法、函数和其他成员的名称列表，接着过滤掉我们不关心的成员。执行过滤行为是通过对每个属性/方法/函数的名称调用 `getattr` 函数取得实际成员的引用，然后检查这些成员对象是否是可调用的，当然这些可调用的成员对象可能是方法或者函数，同时也可能是内置的 (比如列表的 `pop` 方法) 或者用户自定义的 (比如 `odbchelper` 模块的 `buildConnectionString` 函数)。这里你不用关心其它的属性，如内置在每一个模块中的 `__name__` 属性。

## 进一步阅读

*   *Python Tutorial* 讨论了[使用内置 `filter` 函数](http://www.python.org/doc/current/tut/node7.html#SECTION007130000000000000000)过滤列表的另一种方式。

# 4.6\. `and` 和 `or` 的特殊性质

# 4.6\. `and` 和 `or` 的特殊性质

*   4.6.1\. 使用 and-or 技巧

在 Python 中，`and` 和 `or` 执行布尔逻辑演算，如你所期待的一样。但是它们并不返回布尔值，而是返回它们实际进行比较的值之一。

## 例 4.15\. `and` 介绍

```py
>>> 'a' and 'b'         
'b'
>>> '' and 'b'          
''
>>> 'a' and 'b' and 'c' 
'c' 
```

|  |  |
| --- | --- |
| [1] | 使用 `and` 时，在布尔环境中从左到右演算表达式的值。`0`、`''`、`[]`、`()`、`{}`、`None` 在布尔环境中为假；其它任何东西都为真。还好，几乎是所有东西。默认情况下，布尔环境中的类实例为真，但是你可以在类中定义特定的方法使得类实例的演算值为假。你将会在第五章中了解到类和这些特殊方法。如果布尔环境中的所有值都为真，那么 `and` 返回最后一个值。在这个例子中，`and` 演算 `'a'` 的值为真，然后是 `'b'` 的演算值为真，最终返回 `'b'`。 |
| [2] | 如果布尔环境中的某个值为假，则 `and` 返回第一个假值。在这个例子中，`''` 是第一个假值。 |
| [3] | 所有值都为真，所以 `and` 返回最后一个真值，`'c'`。 |

## 例 4.16\. `or` 介绍

```py
>>> 'a' or 'b'          
'a'
>>> '' or 'b'           
'b'
>>> '' or [] or {}      
{}
>>> def sidefx():
... print "in sidefx()"
... return 1
>>> 'a' or sidefx()     
'a' 
```

|  |  |
| --- | --- |
| [1] | 使用 `or` 时，在布尔环境中从左到右演算值，就像 `and` 一样。如果有一个值为真，`or` 立刻返回该值。本例中，`'a'` 是第一个真值。 |
| [2] | `or` 演算 `''` 的值为假，然后演算 `'b'` 的值为真，于是返回 `'b'` 。 |
| [3] | 如果所有的值都为假，`or` 返回最后一个假值。`or` 演算 `''` 的值为假，然后演算 `[]` 的值为假，依次演算 `{}` 的值为假，最终返回 `{}` 。 |
| [4] | 注意 `or` 在布尔环境中会一直进行表达式演算直到找到第一个真值，然后就会忽略剩余的比较值。如果某些值具有副作用，这种特性就非常重要了。在这里，函数 `sidefx` 永远都不会被调用，因为 `or` 演算 `'a'` 的值为真，所以紧接着就立刻返回 `'a'` 了。 |

如果你是一名 C 语言黑客，肯定很熟悉 `_bool_ ?`a`:`b`` 表达式，如果 _`bool`_ 为真，表达式演算值为`a`，否则为`b`。基于 Python 中`and`和`or` 的工作方式，你可以完成相同的事情。

## 4.6.1\. 使用 `and-or` 技巧

## 例 4.17\. `and-or` 技巧介绍

```py
>>> a = "first"
>>> b = "second"
>>> 1 and a or b 
'first'
>>> 0 and a or b 
'second' 
```

|  |  |
| --- | --- |
| [1] | 这个语法看起来类似于 C 语言中的 `_bool_ ?`a`:`b`` 表达式。整个表达式从左到右进行演算，所以先进行`and`表达式的演算。`1 and 'first'`演算值为`'first'`，然后`'first' or 'second'`的演算值为`'first'`。 |
| [2] | `0 and 'first'` 演算值为 `False`，然后 `0 or 'second'` 演算值为 `'second'`。 |

然而，由于这种 Python 表达式单单只是进行布尔逻辑运算，并不是语言的特定构成，这是 `and-or` 技巧和 C 语言中的 `_bool_ ?`a`:`b`` 语法非常重要的不同。如果`a` 为假，表达式就不会按你期望的那样工作了。(你能知道我被这个问题折腾过吗？不止一次？)

## 例 4.18\. `and-or` 技巧无效的场合

```py
>>> a = ""
>>> b = "second"
>>> 1 and a or b         
'second' 
```

|  |  |
| --- | --- |
| [1] | 由于 `a` 是一个空字符串，在 Python 的布尔环境中空字符串被认为是假的，`1 and ''` 的演算值为 `''`，最后 `'' or 'second'` 的演算值为 `'second'`。噢！这个值并不是你想要的。 |

`and-or` 技巧，也就是 `_bool_ and`a`or`b`表达式，当 `a` 在布尔环境中的值为假时，不会像 C 语言表达式 `_bool_ ? `a` : `b` 那样工作。

在 `and-or` 技巧后面真正的技巧是，确保 `a` 的值决不会为假。最常用的方式是使 `a` 成为 `[`a`]` 、 `b` 成为 `[`b`]`，然后使用返回值列表的第一个元素，应该是 `a` 或 `b`中的某一个。

## 例 4.19\. 安全使用 `and-or` 技巧

```py
>>> a = ""
>>> b = "second"
>>> (1 and [a] or [b])[0] 
'' 
```

|  |  |
| --- | --- |
| [1] | 由于 `[`a`]` 是一个非空列表，所以它决不会为假。即使 `a` 是 `0` 或者 `''` 或者其它假值，列表 `[`a`]` 也为真，因为它有一个元素。 |

到现在为止，这个技巧可能看上去问题超过了它的价值。毕竟，使用 `if` 语句可以完成相同的事情，那为什么要经历这些麻烦事呢？哦，在很多情况下，你要在两个常量值中进行选择，由于你知道 `a` 的值总是为真，所以你可以使用这种较为简单的语法而且不用担心。对于使用更为复杂的安全形式，依然有很好的理由要求这样做。例如，在 Python 语言的某些情况下 `if` 语句是不允许使用的，比如在 `lambda` 函数中。

## 进一步阅读

*   Python Cookbook 讨论了[其它的 `and-or` 技巧](http://www.activestate.com/ASPN/Python/Cookbook/Recipe/52310)。

# 4.7\. 使用 `lambda` 函数

# 4.7\. 使用 `lambda` 函数

*   4.7.1\. 真实世界中的 lambda 函数

Python 支持一种有趣的语法，它允许你快速定义单行的最小函数。这些叫做 `lambda` 的函数，是从 Lisp 借用来的，可以用在任何需要函数的地方。

## 例 4.20\. `lambda` 函数介绍

```py
>>> def f(x):
... return x*2
... 
>>> f(3)
6
>>> g = lambda x: x*2  
>>> g(3)
6
>>> (lambda x: x*2)(3) 
6 
```

|  |  |
| --- | --- |
| [1] | 这是一个 `lambda` 函数，完成同上面普通函数相同的事情。注意这里的简短的语法：在参数列表周围没有括号，而且忽略了 `return` 关键字 (隐含存在，因为整个函数只有一行)。而且，该函数没有函数名称，但是可以将它赋值给一个变量进行调用。 |
| [2] | 使用 `lambda` 函数时甚至不需要将它赋值给一个变量。这可能不是世上最有用的东西，它只是展示了 `lambda` 函数只是一个内联函数。 |

总的来说，`lambda` 函数可以接收任意多个参数 (包括可选参数) 并且返回单个表达式的值。`lambda` 函数不能包含命令，包含的表达式不能超过一个。不要试图向 `lambda` 函数中塞入太多的东西；如果你需要更复杂的东西，应该定义一个普通函数，然后想让它多长就多长。

> 注意
> `lambda` 函数是一种风格问题。不一定非要使用它们；任何能够使用它们的地方，都可以定义一个单独的普通函数来进行替换。我将它们用在需要封装特殊的、非重用代码上，避免令我的代码充斥着大量单行函数。

## 4.7.1\. 真实世界中的 `lambda` 函数

`apihelper.py` 中的 `lambda` 函数：

```py
 processFunc = collapse and (lambda s: " ".join(s.split())) or (lambda s: s) 
```

注意这里使用了 `and-or` 技巧的简单形式，它是没问题的，因为 `lambda` 函数在布尔环境中总是为真。(这并不意味这 `lambda` 函数不能返回假值。这个函数对象的布尔值为真；它的返回值可以是任何东西。)

还要注意的是使用了没有参数的 `split` 函数。你已经看到过它带一个或者两个参数的使用，但是不带参数它按空白进行分割。

## 例 4.21\. `split` 不带参数

```py
>>> s = "this   is\na\ttest"  
>>> print s
this   is
a    test
>>> print s.split()           
['this', 'is', 'a', 'test']
>>> print " ".join(s.split()) 
'this is a test' 
```

|  |  |
| --- | --- |
| [1] | 这是一个多行字符串，通过使用转义字符的定义代替了三重引号。`\n` 是一个回车，`\t` 是一个制表符。 |
| [2] | 不带参数的 `split` 按照空白进行分割。所以三个空格、一个回车和一个制表符都是一样的。 |
| [3] | 通过 `split` 分割字符串你可以将空格统一化；然后再以单个空格作为分隔符用 `join` 将其重新连接起来。这也就是 `info` 函数将多行 `doc string` 合并成单行所做的事情。 |

那么 `info` 函数到底用这些 `lambda` 函数、`split` 函数和 `and-or` 技巧做了些什么呢？

```py
 processFunc = collapse and (lambda s: " ".join(s.split())) or (lambda s: s) 
```

`processFunc` 现在是一个函数，但是它到底是哪一个函数还要取决于 `collapse` 变量。如果 `collapse` 为真，`processFunc`(_string_)` 将压缩空白；否则`processFunc`(_string_)` 将返回未改变的参数。

在一个不很健壮的语言中实现它，像 Visual Basic，你很有可能要创建一个函数，接受一个字符串参数和一个 *`collapse`* 参数，并使用 `if` 语句确定是否压缩空白，然后再返回相应的值。这种方式是低效的，因为函数可能需要处理每一种可能的情况。每次你调用它，它将不得不在给出你所想要的东西之前，判断是否要压缩空白。在 Python 中，你可以将决策逻辑拿到函数外面，而定义一个裁减过的 `lambda` 函数提供确切的 (唯一的) 你想要的。这种方式更为高效、更为优雅，而且很少引起那些令人讨厌 (哦，想到那些参数就头昏) 的错误。

## `lambda` 函数进一步阅读

*   Python Knowledge Base 讨论了使用 `lambda` 来[间接调用函数](http://www.faqts.com/knowledge-base/view.phtml/aid/6081/fid/241)。
*   *Python Tutorial* 演示了如何[从一个 `lambda` 函数内部访问外部变量](http://www.python.org/doc/current/tut/node6.html#SECTION006740000000000000000)。([PEP 227](http://python.sourceforge.net/peps/pep-0227.html) 解释了在 Python 的未来版本中将如何变化。)
*   *The Whole Python FAQ* 有关于[令人模糊的使用 `lambda` 单行语句](http://www.python.org/cgi-bin/faqw.py?query=4.15&querytype=simple&casefold=yes&req=search)的例子。

# 4.8\. 全部放在一起

# 4.8\. 全部放在一起

最后一行代码是唯一还没有解释过的，它完成全部的工作。但是现在工作已经简单了，因为所需要的每件事都已经按照需求建立好了。所有的多米诺骨牌已经就位，到了将它们推倒的时候了。

下面是 `apihelper.py` 的关键

```py
 print "\n".join(["%s %s" %
                      (method.ljust(spacing),
                       processFunc(str(getattr(object, method).__doc__)))
                     for method in methodList]) 
```

注意这是一条命令，被分隔成了多行，但是并没有使用续行符 (`\`)。还记得我说过一些表达式可以分割成多行而不需要使用反斜线吗？列表解析就是这些表达式之一，因为整个表达式包括在方括号里。

现在，让我们从后向前分析。

```py
 for method in methodList 
```

告诉我们这是一个列表解析。如你所知 `methodList` 是 `object` 中所有你关心的方法的一个列表。所以你正在使用 `method` 遍历列表。

## 例 4.22\. 动态得到 `doc string`

```py
>>> import odbchelper
>>> object = odbchelper                   
>>> method = 'buildConnectionString'      
>>> getattr(object, method)               
<function buildConnectionString at 010D6D74>
>>> print getattr(object, method).__doc__ 
Build a connection string from a dictionary of parameters.
    Returns string. 
```

|  |  |
| --- | --- |
| [1] | 在 `info` 函数中，`object` 是要得到帮助的对象，作为一个参数传入。 |
| [2] | 在你遍历 `methodList` 时，`method` 是当前方法的名称。 |
| [3] | 通过 `getattr` 函数，你可以得到 *`object`* 模块中 *`method`* 函数的引用。 |
| [4] | 现在，很容易就可以打印出方法的 `doc string` 。 |

接下来令人困惑的是 `doc string` 周围 `str` 的使用。你可能记得，`str` 是一个内置函数，它可以强制将数据转化为字符串。但是一个 `doc string` 应该总是一个字符串，为什么还要费事地使用 `str` 函数呢？答案就是：不是每个函数都有 `doc string` ，如果没有，这个 `__doc__` 属性为 `None`。

## 例 4.23\. 为什么对一个 `doc string` 使用 `str` ？

```py
>>> >>> def foo(): print 2
>>> >>> foo()
2
>>> >>> foo.__doc__     
>>> foo.__doc__ == None 
True
>>> str(foo.__doc__)    
'None' 
```

|  |  |
| --- | --- |
| [1] | 你可以很容易的定义一个没有 `doc string` 的函数，这种情况下它的 `__doc__` 属性为 `None`。令人迷惑的是，如果你直接演算 `__doc__` 属性的值，Python IDE 什么都不会打印。这是有意义的 (前提是你考虑了这个结果的来由)，但是却没有什么用。 |
| [2] | 你可以直接通过 `__doc__` 属性和 `None` 的比较验证 `__doc__` 属性的值。 |
| [3] | `str` 函数可以接收值为 null 的参数，然后返回它的字符串表示，`'None'`。 |

> 注意
> 在 SQL 中，你必须使用 `IS NULL` 代替 `= NULL` 进行 null 值比较。在 Python，你可以使用 `== None` 或者 `is None` 进行比较，但是 `is None` 更快。

现在你确保有了一个字符串，可以把这个字符串传给 `processFunc`，这个函数已经定义是一个既可以压缩空白也可以不压缩空白的函数。现在你看出来为什么使用 `str` 将 `None` 转化为一个字符串很重要了。`processFunc` 假设接收到一个字符串参数然后调用 `split` 方法，如果你传入 `None` ，将导致程序崩溃，因为 `None` 没有 `split` 方法。

再往回走一步，你再一次使用了字符串格式化来连接 `processFunc` 的返回值 和 `method` 的 `ljust` 方法的返回值。`ljust` 是一个你之前没有见过的新字符串方法。

## 例 4.24\. `ljust` 方法介绍

```py
>>> s = 'buildConnectionString'
>>> s.ljust(30) 
'buildConnectionString         '
>>> s.ljust(20) 
'buildConnectionString' 
```

|  |  |
| --- | --- |
| [1] | `ljust` 用空格填充字符串以符合指定的长度。`info` 函数使用它生成了两列输出并将所有在第二列的 `doc string` 纵向对齐。 |
| [2] | 如果指定的长度小于字符串的长度，`ljust` 将简单地返回未变化的字符串。它决不会截断字符串。 |

几乎已经完成了。有了 `ljust` 方法填充过的方法名称和来自调用 `processFunc` 方法得到的 `doc string` (可能压缩过)，你就可以将两者连接起来并得到单个字符串。因为对 `methodList` 进行了映射，最终你将获得一个字符串列表。利用 `"\n"` 的 `join` 函数，将这个列表连接为单个字符串，列表中每个元素独占一行，接着打印出结果。

## 例 4.25\. 打印列表

```py
>>> li = ['a', 'b', 'c']
>>> print "\n".join(li) 
a
b
c 
```

|  |  |
| --- | --- |
| [1] | 在你处理列表时，这确实是一个有用的调试技巧。在 Python 中，你会十分频繁地操作列表。 |

上述就是最后一个令人困惑的地方了。但是现在你应该已经理解这段代码了。

```py
 print "\n".join(["%s %s" %
                      (method.ljust(spacing),
                       processFunc(str(getattr(object, method).__doc__)))
                     for method in methodList]) 
```

# 4.9\. 小结

# 4.9\. 小结

`apihelper.py` 程序和它的输出现在应该非常清晰了。

```py
 def info(object, spacing=10, collapse=1):
    """Print methods and doc strings.
    Takes module, class, list, dictionary, or string."""
    methodList = [method for method in dir(object) if callable(getattr(object, method))]
    processFunc = collapse and (lambda s: " ".join(s.split())) or (lambda s: s)
    print "\n".join(["%s %s" %
                      (method.ljust(spacing),
                       processFunc(str(getattr(object, method).__doc__)))
                     for method in methodList])
if __name__ == "__main__":
    print info.__doc__ 
```

`apihelper.py` 的输出：

```py
>>> from apihelper import info
>>> li = []
>>> info(li)
append     L.append(object) -- append object to end
count      L.count(value) -> integer -- return number of occurrences of value
extend     L.extend(list) -- extend list by appending list elements
index      L.index(value) -> integer -- return index of first occurrence of value
insert     L.insert(index, object) -- insert object before index
pop        L.pop([index]) -> item -- remove and return item at index (default last)
remove     L.remove(value) -- remove first occurrence of value
reverse    L.reverse() -- reverse *IN PLACE*
sort       L.sort([cmpfunc]) -- sort *IN PLACE*; if given, cmpfunc(x, y) -> -1, 0, 1 
```

在研究下一章前，确保你可以无困难的完成下面这些事情：

*   用可选和命名参数定义和调用函数
*   用 `str` 强制转换任意值为字符串形式
*   用 `getattr` 动态得到函数和其它属性的引用
*   扩展列表解析语法实现列表过滤
*   识别 `and-or` 技巧并安全地使用它
*   定义 `lambda` 函数
*   将函数赋值给变量然后通过引用变量调用函数。我强调的已经够多了：这种思考方式对于提高对 Python 的理解力至关重要。在本书中你会随处可见这种技术的更复杂的应用。