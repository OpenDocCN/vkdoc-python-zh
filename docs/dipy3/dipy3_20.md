# Chapter A 使用`2to3`将代码移植到 Python 3

# Chapter A 使用`2to3`将代码移植到 Python 3

> " Life is pleasant. Death is peaceful. It’s the transition that’s troublesome. " — Isaac Asimov (attributed)

## 概述

几乎所有的 Python 2 程序都需要一些修改才能正常地运行在 Python 3 的环境下。为了简化这个转换过程，Python 3 自带了一个叫做`2to3`的实用脚本(Utility Script)，这个脚本会将你的 Python 2 程序源文件作为输入，然后自动将其转换到 Python 3 的形式。案例研究:将`chardet`移植到 Python 3(porting chardet to Python 3)描述了如何运行这个脚本，然后展示了一些它不能自动修复的情况。这篇附录描述了它*能够*自动修复的内容。

## `print`语句

在 Python 2 里，`print`是一个语句。无论你想输出什么，只要将它们放在`print`关键字后边就可以。在 Python 3 里，`print()`是一个函数。就像其他的函数一样，`print()`需要你将想要输出的东西作为参数传给它。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | `print` | `print()` |
| ② | `print 1` | `print(1)` |
| ③ | `print 1, 2` | `print(1, 2)` |
| ④ | `print 1, 2,` | `print(1, 2, end=' ')` |
| ⑤ | `print &gt;&gt;sys.stderr, 1, 2, 3` | `print(1, 2, 3, file=sys.stderr)` |

1.  为输出一个空白行，需要调用不带参数的`print()`。
2.  为输出一个单独的值，需要将这这个值作为`print()`的一个参数就可以了。
3.  为输出使用一个空格分隔的两个值，用两个参数调用`print()`即可。
4.  这个例子有一些技巧。在 Python 2 里，如果你使用一个逗号(,)作为`print`语句的结尾，它将会用空格分隔输出的结果，然后在输出一个尾随的空格(trailing space)，而不输出回车(carriage return)。在 Python 3 里，通过把`end=' '`作为一个关键字参数传给`print()`可以实现同样的效果。参数`end`的默认值为`'\n'`，所以通过重新指定`end`参数的值，可以取消在末尾输出回车符。
5.  在 Python 2 里，你可以通过使用`&gt;&gt;pipe_name`语法，把输出重定向到一个管道，比如`sys.stderr`。在 Python 3 里，你可以通过将管道作为关键字参数`file`的值传递给`print()`来完成同样的功能。参数`file`的默认值为`std.stdout`，所以重新指定它的值将会使`print()`输出到一个另外一个管道。

## Unicode 字符串

Python 2 有两种字符串类型：Unicode 字符串和非 Unicode 字符串。Python 3 只有一种类型：Unicode 字符串(Unicode strings)。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | `u'PapayaWhip'` | `'PapayaWhip'` |
| ② | `ur'PapayaWhip\foo'` | `r'PapayaWhip\foo'` |

1.  Python 2 里的 Unicode 字符串在 Python 3 里即普通字符串，因为在 Python 3 里字符串总是 Unicode 形式的。
2.  Unicode 原始字符串(raw string)(使用这种字符串，Python 不会自动转义反斜线"\")也被替换为普通的字符串，因为在 Python 3 里，所有原始字符串都是以 Unicode 编码的。

## 全局函数`unicode()`

Python 2 有两个全局函数可以把对象强制转换成字符串：`unicode()`把对象转换成 Unicode 字符串，还有`str()`把对象转换为非 Unicode 字符串。Python 3 只有一种字符串类型，Unicode 字符串，所以`str()`函数即可完成所有的功能。(`unicode()`函数在 Python 3 里不再存在了。)

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
|  | `unicode(anything)` | `str(anything)` |

## `long` 长整型

Python 2 有为非浮点数准备的`int`和`long`类型。`int`类型的最大值不能超过`sys.maxint`，而且这个最大值是平台相关的。可以通过在数字的末尾附上一个`L`来定义长整型，显然，它比`int`类型表示的数字范围更大。在 Python 3 里，只有一种整数类型`int`，大多数情况下，它很像 Python 2 里的长整型。由于已经不存在两种类型的整数，所以就没有必要使用特殊的语法去区别他们。

[进一步阅读：PEP 237：统一长整型和整型](http://www.python.org/dev/peps/pep-0237/)。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | `x = 1000000000000L` | `x = 1000000000000` |
| ② | `x = 0xFFFFFFFFFFFFL` | `x = 0xFFFFFFFFFFFF` |
| ③ | `long(x)` | `int(x)` |
| ④ | `type(x) is long` | `type(x) is int` |
| ⑤ | `isinstance(x, long)` | `isinstance(x, int)` |

1.  在 Python 2 里的十进制长整型在 Python 3 里被替换为十进制的普通整数。
2.  在 Python 2 里的十六进制长整型在 Python 3 里被替换为十六进制的普通整数。
3.  在 Python 3 里，由于长整型已经不存在了，自然原来的`long()`函数也没有了。为了强制转换一个变量到整型，可以使用`int()`函数。
4.  检查一个变量是否是整型，获得它的数据类型，并与一个`int`类型(不是`long`)的作比较。
5.  你也可以使用`isinstance()`函数来检查数据类型；再强调一次，使用`int`，而不是`long`，来检查整数类型。

## <> 比较运算符

Python 2 支持`&lt;&gt;`作为`!=`的同义词。Python 3 只支持`!=`，不再支持<>了。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | `if x &lt;&gt; y:` | `if x != y:` |
| ② | `if x &lt;&gt; y &lt;&gt; z:` | `if x != y != z:` |

1.  简单地比较。
2.  相对复杂的三个值之间的比较。

## 字典类方法`has_key()`

在 Python 2 里，字典对象的`has_key()`方法用来测试字典是否包含特定的键(key)。Python 3 不再支持这个方法了。你需要使用`in`运算符。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | `a_dictionary.has_key('PapayaWhip')` | `'PapayaWhip' in a_dictionary` |
| ② | `a_dictionary.has_key(x) or a_dictionary.has_key(y)` | `x in a_dictionary or y in a_dictionary` |
| ③ | `a_dictionary.has_key(x or y)` | `(x or y) in a_dictionary` |
| ④ | `a_dictionary.has_key(x + y)` | `(x + y) in a_dictionary` |
| ⑤ | `x + a_dictionary.has_key(y)` | `x + (y in a_dictionary)` |

1.  最简单的形式。
2.  运算符`or`的优先级高于运算符`in`，所以这里不需要添加括号。
3.  另一方面，出于同样的原因 — `or`的优先级大于`in`，这里需要添加括号。(注意：这里的代码与前面那行完全不同。Python 会先解释`x or y`，得到结果`x`(如果`x`在布尔上下文里的值是真)或者`y`。然后 Python 检查这个结果是不是`a_dictionary`的一个键。)
4.  运算符`in`的优先级大于运算符`+`，所以代码里的这种形式从技术上说不需要括号，但是`2to3`还是添加了。
5.  这种形式一定需要括号，因为`in`的优先级大于`+`。

## 返回列表的字典类方法

在 Python 2 里，许多字典类方法的返回值是列表。其中最常用方法的有`keys`，`items`和`values`。在 Python 3 里，所有以上方法的返回值改为动态视图(dynamic view)。在一些上下文环境里，这种改变并不会产生影响。如果这些方法的返回值被立即传递给另外一个函数，并且那个函数会遍历整个序列，那么以上方法的返回值是列表或者视图并不会产生什么不同。在另外一些情况下，Python 3 的这些改变干系重大。如果你期待一个能被独立寻址元素的列表，那么 Python 3 的这些改变将会使你的代码卡住(choke)，因为视图(view)不支持索引(indexing)。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | `a_dictionary.keys()` | `list(a_dictionary.keys())` |
| ② | `a_dictionary.items()` | `list(a_dictionary.items())` |
| ③ | `a_dictionary.iterkeys()` | `iter(a_dictionary.keys())` |
| ④ | `[i for i in a_dictionary.iterkeys()]` | `[i for i in a_dictionary.keys()]` |
| ⑤ | `min(a_dictionary.keys())` | *no change* |

1.  使用`list()`函数将`keys()`的返回值转换为一个静态列表，出于安全方面的考量，`2to3`可能会报错。这样的代码是有效的，但是对于使用视图来说，它的效率低一些。你应该检查转换后的代码，看看是否一定需要列表，也许视图也能完成同样的工作。
2.  这是另外一种视图(关于`items()`方法的)到列表的转换。`2to3`对`values()`方法返回值的转换也是一样的。
3.  Python 3 里不再支持`iterkeys()`了。如果必要，使用`iter()`将`keys()`的返回值转换成为一个迭代器。
4.  `2to3`能够识别出`iterkeys()`方法在列表解析里被使用，然后将它转换为 Python 3 里的`keys()`方法(不需要使用额外的`iter()`去包装其返回值)。这样是可行的，因为视图是可迭代的。
5.  `2to3`也能识别出`keys()`方法的返回值被立即传给另外一个会遍历整个序列的函数，所以也就没有必要先把`keys()`的返回值转换到一个列表。相反的，`min()`函数会很乐意遍历视图。这个过程对`min()`，`max()`，`sum()`，`list()`，`tuple()`，`set()`，`sorted()`，`any()`和`all()`同样有效。

## 被重命名或者重新组织的模块

从 Python 2 到 Python 3，标准库里的一些模块已经被重命名了。还有一些相互关联的模块也被组合或者重新组织，以使得这种关联更有逻辑性。

### `http`

在 Python 3 里，几个相关的 HTTP 模块被组合成一个单独的包，即`http`。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | `import httplib` | `import http.client` |
| ② | `import Cookie` | `import http.cookies` |
| ③ | `import cookielib` | `import http.cookiejar` |
| ④ | `import BaseHTTPServer` `import SimpleHTTPServer` `import CGIHttpServer` | `import http.server` |

1.  `http.client`模块实现了一个底层的库，可以用来请求 HTTP 资源，解析 HTTP 响应。
2.  `http.cookies`模块提供一个蟒样的(Pythonic)接口来获取通过 HTTP 头部(HTTP header)Set-Cookie 发送的 cookies
3.  常用的流行的浏览器会把 cookies 以文件形式存放在磁盘上，`http.cookiejar`模块可以操作这些文件。
4.  `http.server`模块实现了一个基本的 HTTP 服务器

### `urllib`

Python 2 有一些用来分析，编码和获取 URL 的模块，但是这些模块就像老鼠窝一样相互重叠。在 Python 3 里，这些模块被重构、组合成了一个单独的包，即`urllib`。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | `import urllib` | `import urllib.request, urllib.parse, urllib.error` |
| ② | `import urllib2` | `import urllib.request, urllib.error` |
| ③ | `import urlparse` | `import urllib.parse` |
| ④ | `import robotparser` | `import urllib.robotparser` |
| ⑤ | `from urllib import FancyURLopener` `from urllib import urlencode` | `from urllib.request import FancyURLopener` `from urllib.parse import urlencode` |
| ⑥ | `from urllib2 import Request` `from urllib2 import HTTPError` | `from urllib.request import Request` `from urllib.error import HTTPError` |

1.  以前，Python 2 里的`urllib`模块有各种各样的函数，包括用来获取数据的`urlopen()`，还有用来将 URL 分割成其组成部分的`splittype()`，`splithost()`和`splituser()`函数。在新的`urllib`包里，这些函数被组织得更有逻辑性。2to3 将会修改这些函数的调用以适应新的命名方案。
2.  在 Python 3 里，以前的`urllib2`模块被并入了`urllib`包。同时，以`urllib2`里各种你最喜爱的东西将会一个不缺地出现在 Python 3 的`urllib`模块里，比如`build_opener()`方法，`Request`对象，`HTTPBasicAuthHandler`和 friends。
3.  Python 3 里的`urllib.parse`模块包含了原来 Python 2 里`urlparse`模块所有的解析函数。
4.  `urllib.robotparse`模块解析[`robots.txt`文件](http://www.robotstxt.org/)。
5.  处理 HTTP 重定向和其他状态码的`FancyURLopener`类在 Python 3 里的`urllib.request`模块里依然有效。`urlencode()`函数已经被转移到了`urllib.parse`里。
6.  `Request`对象在`urllib.request`里依然有效，但是像`HTTPError`这样的常量已经被转移到了`urllib.error`里。

我是否有提到`2to3`也会重写你的函数调用？比如，如果你的 Python 2 代码里导入了`urllib`模块，调用了`urllib.urlopen()`函数获取数据，`2to3`会同时修改`import`语句和函数调用。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
|  | `import urllib` `print urllib.urlopen('http://diveintopython3.org/').read()` | `import urllib.request, urllib.parse, urllib.error``print(urllib.request.urlopen('http://diveintopython3.org/').read())` |

### `dbm`

所有的 DBM 克隆(DBM clone)现在在单独的一个包里，即`dbm`。如果你需要其中某个特定的变体，比如 GNU DBM，你可以导入`dbm`包中合适的模块。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
|  | `import dbm` | `import dbm.ndbm` |
|  | `import gdbm` | `import dbm.gnu` |
|  | `import dbhash` | `import dbm.bsd` |
|  | `import dumbdbm` | `import dbm.dumb` |
|  | `import anydbm` `import whichdb` | `import dbm` |

### `xmlrpc`

XML-RPC 是一个通过 HTTP 协议执行远程 RPC 调用的轻重级方法。一些 XML-RPC 客户端和 XML-RPC 服务端的实现库现在被组合到了独立的包，即`xmlrpc`。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
|  | `import xmlrpclib` | `import xmlrpc.client` |
|  | `import DocXMLRPCServer` `import SimpleXMLRPCServer` | `import xmlrpc.server` |

### 其他模块

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | [1] | `import io` |
| ② | [2] | `import pickle` |
| ③ | `import __builtin__` | `import builtins` |
| ④ | `import copy_reg` | `import copyreg` |
| ⑤ | `import Queue` | `import queue` |
| ⑥ | `import SocketServer` | `import socketserver` |
| ⑦ | `import ConfigParser` | `import configparser` |
| ⑧ | `import repr` | `import reprlib` |
| ⑨ | `import commands` | `import subprocess` |

[1]:

```py
try:
    import cStringIO as StringIO
except ImportError:
    import StringIO 
```

[2]:

```py
try:
    import cPickle as pickle
except ImportError:
    import pickle 
```

1.  在 Python 2 里，你通常会这样做，首先尝试把`cStringIO`导入作为`StringIO`的替代，如果失败了，再导入`StringIO`。不要在 Python 3 里这样做；`io`模块会帮你处理好这件事情。它会找出可用的最快实现方法，然后自动使用它。
2.  在 Python 2 里，导入最快的`pickle`实现也是一个与上边相似的能用方法。在 Python 3 里，`pickle`模块会自动为你处理，所以不要再这样做。
3.  `builtins`模块包含了在整个 Python 语言里都会使用的全局函数，类和常量。重新定义`builtins`模块里的某个函数意味着在每处都重定义了这个全局函数。这听起来很强大，但是同时也是很可怕的。
4.  `copyreg`模块为用 C 语言定义的用户自定义类型添加了`pickle`模块的支持。
5.  `queue`模块实现一个生产者消费者队列(multi-producer, multi-consumer queue)。
6.  `socketserver`模块为实现各种 socket server 提供了通用基础类。
7.  `configparser`模块用来解析 INI-style 配置文件。
8.  `reprlib`模块重新实现了内置函数`repr()`，并添加了对字符串表示被截断前长度的控制。
9.  `subprocess`模块允许你创建子进程，连接到他们的管道，然后获取他们的返回值。

## 包内的相对导入

包是由一组相关联的模块共同组成的单个实体。在 Python 2 的时候，为了实现同一个包内模块的相互引用，你会使用`import foo`或者`from foo import Bar`。Python 2 解释器会先在当前目录里搜索`foo.py`，然后再去 Python 搜索路径(`sys.path`)里搜索。在 Python 3 里这个过程有一点不同。Python 3 不会首先在当前路径搜索，它会直接在 Python 的搜索路径里寻找。如果你想要包里的一个模块导入包里的另外一个模块，你需要显式地提供两个模块的相对路径。

假设你有如下包，多个文件在同一个目录下：

```py
chardet/
|
+--__init__.py
|
+--constants.py
|
+--mbcharsetprober.py
|
+--universaldetector.py 
```

现在假设`universaldetector.py`需要整个导入`constants.py`，另外还需要导入`mbcharsetprober.py`的一个类。你会怎样做?

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | `import constants` | `from . import constants` |
| ② | `from mbcharsetprober import MultiByteCharSetProber` | `from .mbcharsetprober import MultiByteCharsetProber` |

1.  当你需要从包的其他地方导入整个模块，使用新的`from . import`语法。这里的句号(.)即表示当前文件(`universaldetector.py`)和你想要导入文件(`constants.py`)之间的相对路径。在这个样例中，这两个文件在同一个目录里，所以使用了单个句号。你也可以从父目录(`from .. import anothermodule`)或者子目录里导入。
2.  为了将一个特定的类或者函数从其他模块里直接导入到你的模块的名字空间里，在需要导入的模块名前加上相对路径，并且去掉最后一个斜线(slash)。在这个例子中，`mbcharsetprober.py`与`universaldetector.py`在同一个目录里，所以相对路径名就是一个句号。你也可以从父目录(from .. import anothermodule)或者子目录里导入。

## 迭代器方法`next()`

在 Python 2 里，迭代器有一个`next()`方法，用来返回序列里的下一项。在 Python 3 里这同样成立，但是现在有了一个新的全局的函数`next()`，它使用一个迭代器作为参数。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | `anIterator.next()` | `next(anIterator)` |
| ② | `a_function_that_returns_an_iterator().next()` | `next(a_function_that_returns_an_iterator())` |
| ③ | [1] | [2] |
| ④ | [3] | *no change* |
| ⑤ | [4] | [5] |

[1]:

```py
class A:
    def next(self):
        pass 
```

[2]:

```py
class A:
    def __next__(self):
        pass 
```

[3]:

```py
class A:
    def next(self, x, y):
        pass 
```

[4]:

```py
next = 42
for an_iterator in a_sequence_of_iterators:
    an_iterator.next() 
```

[5]:

```py
next = 42
for an_iterator in a_sequence_of_iterators:
    an_iterator.__next__() 
```

1.  最简单的例子，你不再调用一个迭代器的`next()`方法，现在你将迭代器自身作为参数传递给全局函数`next()`。
2.  假如你有一个返回值是迭代器的函数，调用这个函数然后把结果作为参数传递给`next()`函数。(`2to3`脚本足够智能以正确执行这种转换。)
3.  假如你想定义你自己的类，然后把它用作一个迭代器，在 Python 3 里，你可以通过定义特殊方法`__next__()`来实现。
4.  如果你定义的类里刚好有一个`next()`，它使用一个或者多个参数，`2to3`执行的时候不会动它。这个类不能被当作迭代器使用，因为它的`next()`方法带有参数。
5.  这一个有些复杂。如果你恰好有一个叫做`next`的本地变量，在 Python 3 里它的优先级会高于全局函数`next()`。在这种情况下，你需要调用迭代器的特别方法`__next__()`来获取序列里的下一个元素。(或者，你也可以重构代码以使这个本地变量的名字不叫`next`，但是 2to3 不会为你做这件事。)

## 全局函数`filter()`

在 Python 2 里，`filter()`方法返回一个列表，这个列表是通过一个返回值为`True`或者`False`的函数来检测序列里的每一项得到的。在 Python 3 里，`filter()`函数返回一个迭代器，不再是列表。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | `filter(a_function, a_sequence)` | `list(filter(a_function, a_sequence))` |
| ② | `list(filter(a_function, a_sequence))` | *no change* |
| ③ | `filter(None, a_sequence)` | `[i for i in a_sequence if i]` |
| ④ | `for i in filter(None, a_sequence):` | *no change* |
| ⑤ | `[i for i in filter(a_function, a_sequence)]` | *no change* |

1.  最简单的情况下，`2to3`会用一个`list()`函数来包装`filter()`，`list()`函数会遍历它的参数然后返回一个列表。
2.  然而，如果`filter()`调用已经被`list()`包裹，`2to3`不会再做处理，因为这种情况下`filter()`的返回值是否是一个迭代器是无关紧要的。
3.  为了处理`filter(None, ...)`这种特殊的语法，`2to3`会将这种调用从语法上等价地转换为列表解析。
4.  由于`for`循环会遍历整个序列，所以没有必要再做修改。
5.  与上面相同，不需要做修改，因为列表解析会遍历整个序列，即使`filter()`返回一个迭代器，它仍能像以前的`filter()`返回列表那样正常工作。

## 全局函数`map()`

跟`filter()`作的改变一样，`map()`函数现在返回一个迭代器。(在 Python 2 里，它返回一个列表。)

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | `map(a_function, 'PapayaWhip')` | `list(map(a_function, 'PapayaWhip'))` |
| ② | `map(None, 'PapayaWhip')` | `list('PapayaWhip')` |
| ③ | `map(lambda x: x+1, range(42))` | `[x+1 for x in range(42)]` |
| ④ | `for i in map(a_function, a_sequence):` | *no change* |
| ⑤ | `[i for i in map(a_function, a_sequence)]` | *no change* |

1.  类似对`filter()`的处理，在最简单的情况下，`2to3`会用一个`list()`函数来包装`map()`调用。
2.  对于特殊的`map(None, ...)`语法，跟`filter(None, ...)`类似，`2to3`会将其转换成一个使用`list()`的等价调用
3.  如果`map()`的第一个参数是一个 lambda 函数，`2to3`会将其等价地转换成列表解析。
4.  对于会遍历整个序列的`for`循环，不需要做改变。
5.  再一次地，这里不需要做修改，因为列表解析会遍历整个序列，即使`map()`的返回值是迭代器而不是列表它也能正常工作。

## 全局函数`reduce()`

在 Python 3 里，`reduce()`函数已经被从全局名字空间里移除了，它现在被放置在`fucntools`模块里。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
|  | `reduce(a, b, c)` | `from functools import reduce` `reduce(a, b, c)` |

## 全局函数`apply()`

Python 2 有一个叫做`apply()`的全局函数，它使用一个函数`f`和一个列表`[a, b, c]`作为参数，返回值是`f(a, b, c)`。你也可以通过直接调用这个函数，在列表前添加一个星号(*)作为参数传递给它来完成同样的事情。在 Python 3 里，`apply()`函数不再存在了；必须使用星号标记法。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | `apply(a_function, a_list_of_args)` | `a_function(*a_list_of_args)` |
| ② | `apply(a_function, a_list_of_args, a_dictionary_of_named_args)` | `a_function(*a_list_of_args, **a_dictionary_of_named_args)` |
| ③ | `apply(a_function, a_list_of_args + z)` | `a_function(*a_list_of_args + z)` |
| ④ | `apply(aModule.a_function, a_list_of_args)` | `aModule.a_function(*a_list_of_args)` |

1.  最简单的形式，可以通过在参数列表(就像`[a, b, c]`一样)前添加一个星号来调用函数。这跟 Python 2 里的`apply()`函数是等价的。
2.  在 Python 2 里，`apply()`函数实际上可以带 3 个参数：一个函数，一个参数列表，一个字典命名参数(dictionary of named arguments)。在 Python 3 里，你可以通过在参数列表前添加一个星号(`*`)，在字典命名参数前添加两个星号(`**`)来达到同样的效果。
3.  运算符`+`在这里用作连接列表的功能，它的优先级高于运算符`*`，所以没有必要在`a_list_of_args + z`周围添加额外的括号。
4.  `2to3`脚本足够智能来转换复杂的`apply()`调用，包括调用导入模块里的函数。

## 全局函数`intern()`

在 Python 2 里，你可以用`intern()`函数作用在一个字符串上来限定(intern)它以达到性能优化。在 Python 3 里，`intern()`函数被转移到`sys`模块里了。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
|  | `intern(aString)` | `sys.intern(aString)` |

## `exec`语句

就像`print`语句在 Python 3 里变成了一个函数一样，`exec`语句也是这样的。`exec()`函数使用一个包含任意 Python 代码的字符串作为参数，然后就像执行语句或者表达式一样执行它。`exec()`跟`eval()`是相似的，但是`exec()`更加强大并更具有技巧性。`eval()`函数只能执行单独一条表达式，但是``exec`()`能够执行多条语句，导入(import)，函数声明 — 实际上整个 Python 程序的字符串表示也可以。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | `exec codeString` | `exec(codeString)` |
| ② | `exec codeString in a_global_namespace` | `exec(codeString, a_global_namespace)` |
| ③ | `exec codeString in a_global_namespace, a_local_namespace` | `exec(codeString, a_global_namespace, a_local_namespace)` |

1.  在最简单的形式下，因为`exec()`现在是一个函数，而不是语句，`2to3`会把这个字符串形式的代码用括号围起来。
2.  Python 2 里的`exec`语句可以指定名字空间，代码将在这个由全局对象组成的私有空间里执行。Python 3 也有这样的功能；你只需要把这个名字空间作为第二个参数传递给`exec()`函数。
3.  更加神奇的是，Python 2 里的`exec`语句还可以指定一个本地名字空间(比如一个函数里声明的变量)。在 Python 3 里，`exec()`函数也有这样的功能。

## `execfile`语句

就像以前的`exec`语句，Python 2 里的`execfile`语句也可以像执行 Python 代码那样使用字符串。不同的是`exec`使用字符串，而`execfile`则使用文件。在 Python 3 里，`execfile`语句已经被去掉了。如果你真的想要执行一个文件里的 Python 代码(但是你不想导入它)，你可以通过打开这个文件，读取它的内容，然后调用`compile()`全局函数强制 Python 解释器编译代码，然后调用新的`exec()`函数。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
|  | `execfile('a_filename')` | `exec(compile(open('a_filename').read(), 'a_filename', 'exec'))` |

## `repr`(反引号)

在 Python 2 里，为了得到一个任意对象的字符串表示，有一种把对象包装在反引号里(比如`x`)的特殊语法。在 Python 3 里，这种能力仍然存在，但是你不能再使用反引号获得这种字符串表示了。你需要使用全局函数`repr()`。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | ``x`` | `repr(x)` |
| ② | ``'PapayaWhip' + 2`` | `repr('PapayaWhip' + repr(2))` |

1.  记住，`x`可以是任何东西 — 一个类，函数，模块，基本数据类型，等等。`repr()`函数可以使用任何类型的参数。
2.  在 Python 2 里，反引号可以嵌套，导致了这种令人费解的(但是有效的)表达式。`2to3`足够智能以将这种嵌套调用转换到`repr()`函数。

## `try...except`语句

从 Python 2 到 Python 3，捕获异常的语法有些许变化。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | [1] | [2] |
| ② | [3] | [4] |
| ③ | [5] | *no change* |
| ④ | [6] | *no change* |

[1]:

```py
try:
    import mymodule
except ImportError, e
    pass 
```

[2]:

```py
try:
    import mymodule
except ImportError as e:
    pass 
```

[3]:

```py
try:
    import mymodule
except (RuntimeError, ImportError), e
    pass 
```

[4]:

```py
try:
    import mymodule
except (RuntimeError, ImportError) as e:
    pass 
```

[5]:

```py
try:
    import mymodule
except ImportError:
    pass 
```

[6]:

```py
try:
    import mymodule
except:
    pass 
```

1.  相对于 Python 2 里在异常类型后添加逗号，Python 3 使用了一个新的关键字，`as`。
2.  关键字`as`也可以用在一次捕获多种类型异常的情况下。
3.  如果你捕获到一个异常，但是并不在意访问异常对象本身，Python 2 和 Python 3 的语法是一样的。
4.  类似地，如果你使用一个保险方法(fallback)来捕获*所有*异常，Python 2 和 Python 3 的语法是一样的。

> ☞在导入模块(或者其他大多数情况)的时候，你绝对不应该使用这种方法(指以上的 fallback)。不然的话，程序可能会捕获到像`KeyboardInterrupt`(如果用户按`Ctrl-C`来中断程序)这样的异常，从而使调试变得更加困难。

## `raise`语句

Python 3 里，抛出自定义异常的语法有细微的变化。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | `raise MyException` | *unchanged* |
| ② | `raise MyException, 'error message'` | `raise MyException('error message')` |
| ③ | `raise MyException, 'error message', a_traceback` | `raise MyException('error message').with_traceback(a_traceback)` |
| ④ | `raise 'error message'` | *unsupported* |

1.  抛出不带用户自定义错误信息的异常，这种最简单的形式下，语法没有改变。
2.  当你想要抛出一个带用户自定义错误信息的异常时，改变就显而易见了。Python 2 用一个逗号来分隔异常类和错误信息；Python 3 把错误信息作为参数传递给异常类。
3.  Python 2 支持一种更加复杂的语法来抛出一个带用户自定义回溯(stack trace，堆栈追踪)的异常。在 Python 3 里你也可以这样做，但是语法完全不同。
4.  在 Python 2 里，你可以抛出一个不带异常类的异常，仅仅只有一个异常信息。在 Python 3 里，这种形式不再被支持。`2to3`将会警告你它不能自动修复这种语法。

## 生成器的`throw`方法

在 Python 2 里，生成器有一个`throw()`方法。调用`a_generator.throw()`会在生成器被暂停的时候抛出一个异常，然后返回由生成器函数获取的下一个值。在 Python 3 里，这种功能仍然可用，但是语法上有一点不同。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | `a_generator.throw(MyException)` | *no change* |
| ② | `a_generator.throw(MyException, 'error message')` | `a_generator.throw(MyException('error message'))` |
| ③ | `a_generator.throw('error message')` | *unsupported* |

1.  最简单的形式下，生成器抛出不带用户自定义错误信息的异常。这种情况下，从 Python 2 到 Python 3 语法上没有变化 。
2.  如果生成器抛出一个带用户自定义错误信息的异常，你需要将这个错误信息字符串(error string)传递给异常类来以实例化它。
3.  Python 2 还支持抛出只有异常信息的异常。Python 3 不支持这种语法，并且`2to3`会显示一个警告信息，告诉你需要手动地来修复这处代码。

## 全局函数`xrange()`

在 Python 2 里，有两种方法来获得一定范围内的数字：`range()`，它返回一个列表，还有`range()`，它返回一个迭代器。在 Python 3 里，`range()`返回迭代器，`xrange()`不再存在了。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | `xrange(10)` | `range(10)` |
| ② | `a_list = range(10)` | `a_list = list(range(10))` |
| ③ | `[i for i in xrange(10)]` | `[i for i in range(10)]` |
| ④ | `for i in range(10):` | *no change* |
| ⑤ | `sum(range(10))` | *no change* |

1.  在最简单的情况下，`2to3`会简单地把`xrange()`转换为`range()`。
2.  如果你的 Python 2 代码使用`range()`，`2to3`不知道你是否需要一个列表，或者是否一个迭代器也行。出于谨慎，`2to3`可能会报错，然后使用`list()`把`range()`的返回值强制转换为列表类型。
3.  如果在列表解析里有`xrange()`函数，就没有必要将其返回值转换为一个列表，因为列表解析对迭代器同样有效。
4.  类似的，`for`循环也能作用于迭代器，所以这里也没有改变任何东西。
5.  函数`sum()`能作用于迭代器，所以`2to3`也没有在这里做出修改。就像返回值为视图(view)而不再是列表的字典类方法一样，这同样适用于`min()`，`max()`，`sum()`，list()，`tuple()`，`set()`，`sorted()`，`any()`，`all()`。

## 全局函数`raw_input()`和`input()`

Python 2 有两个全局函数，用来在命令行请求用户输入。第一个叫做`input()`，它等待用户输入一个 Python 表达式(然后返回结果)。第二个叫做`raw_input()`，用户输入什么它就返回什么。这让初学者非常困惑，并且这被广泛地看作是 Python 语言的一个“肉赘”(wart)。Python 3 通过重命名`raw_input()`为`input()`，从而切掉了这个肉赘，所以现在的`input()`就像每个人最初期待的那样工作。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | `raw_input()` | `input()` |
| ② | `raw_input('prompt')` | `input('prompt')` |
| ③ | `input()` | `eval(input())` |

1.  最简单的形式，`raw_input()`被替换成`input()`。
2.  在 Python 2 里，`raw_input()`函数可以指定一个提示符作为参数。Python 3 里保留了这个功能。
3.  如果你真的想要请求用户输入一个 Python 表达式，计算结果，可以通过调用`input()`函数然后把返回值传递给`eval()`。

## 函数属性`func_*`

在 Python 2 里，函数的里的代码可以访问到函数本身的特殊属性。在 Python 3 里，为了一致性，这些特殊属性被重新命名了。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | `a_function.func_name` | `a_function.__name__` |
| ② | `a_function.func_doc` | `a_function.__doc__` |
| ③ | `a_function.func_defaults` | `a_function.__defaults__` |
| ④ | `a_function.func_dict` | `a_function.__dict__` |
| ⑤ | `a_function.func_closure` | `a_function.__closure__` |
| ⑥ | `a_function.func_globals` | `a_function.__globals__` |
| ⑦ | `a_function.func_code` | `a_function.__code__` |

1.  `__name__`属性(原`func_name`)包含了函数的名字。
2.  `__doc__`属性(原`funcdoc`)包含了你在函数源代码里定义的文档字符串(*docstring*)
3.  `__defaults__`属性(原`func_defaults`)是一个保存参数默认值的元组。
4.  `__dict__`属性(原`func_dict`)是一个支持任意函数属性的名字空间。
5.  `__closure__`属性(原`func_closure`)是一个由 cell 对象组成的元组，它包含了函数对自由变量(free variable)的绑定。
6.  `__globals__`属性(原`func_globals`)是一个对模块全局名字空间的引用，函数本身在这个名字空间里被定义。
7.  `__code__`属性(原`func_code`)是一个代码对象，表示编译后的函数体。

## I/O 方法`xreadlines()`

在 Python 2 里，文件对象有一个`xreadlines()`方法，它返回一个迭代器，一次读取文件的一行。这在`for`循环中尤其有用。事实上，后来的 Python 2 版本给文件对象本身添加了这样的功能。

在 Python 3 里，`xreadlines()`方法不再可用了。`2to3`可以解决简单的情况，但是一些边缘案例则需要人工介入。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | `for line in a_file.xreadlines():` | `for line in a_file:` |
| ② | `for line in a_file.xreadlines(5):` | *no change (broken)* |

1.  如果你以前调用没有参数的`xreadlines()`，`2to3`会把它转换成文件对象本身。在 Python 3 里，这种转换后的代码可以完成前同样的工作：一次读取文件的一行，然后执行`for`循环的循环体。
2.  如果你以前使用一个参数(每次读取的行数)调用`xreadlines()`，`2to3`不能为你完成从 Python 2 到 Python 3 的转换，你的代码会以这样的方式失败：`AttributeError: '_io.TextIOWrapper' object has no attribute 'xreadlines'`。你可以手工的把`xreadlines()`改成`readlines()`以使代码能在 Python 3 下工作。(readline()方法在 Python 3 里返回迭代器，所以它跟 Python 2 里的`xreadlines()`效率是不相上下的。)

☃

## 使用元组而非多个参数的`lambda`函数

在 Python 2 里，你可以定义匿名`lambda`函数(anonymous `lambda` function)，通过指定作为参数的元组的元素个数，使这个函数实际上能够接收多个参数。事实上，Python 2 的解释器把这个元组“解开”(unpack)成命名参数(named arguments)，然后你可以在`lambda`函数里引用它们(通过名字)。在 Python 3 里，你仍然可以传递一个元组作为`lambda`函数的参数，但是 Python 解释器不会把它解析成命名参数。你需要通过位置索引(positional index)来引用每个参数。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | `lambda (x,): x + f(x)` | `lambda x1: x1[0] + f(x1[0])` |
| ② | `lambda (x, y): x + f(y)` | `lambda x_y: x_y[0] + f(x_y[1])` |
| ③ | `lambda (x, (y, z)): x + y + z` | `lambda x_y_z: x_y_z[0] + x_y_z[1][0] + x_y_z[1][1]` |
| ④ | `lambda x, y, z: x + y + z` | *unchanged* |

1.  如果你已经定义了一个`lambda`函数，它使用包含一个元素的元组作为参数，在 Python 3 里，它会被转换成一个包含到`x1[0]`的引用的`lambda`函数。`x1`是`2to3`脚本基于原来元组里的命名参数自动生成的。
2.  使用含有两个元素的元组`(x, y)`作为参数的`lambda`函数被转换为`x_y`，它有两个位置参数，即`x_y[0]`和`x_y[1]`。
3.  `2to3`脚本甚至可以处理使用嵌套命名参数的元组作为参数的`lambda`函数。产生的结果代码有点难以阅读，但是它在 Python 3 下跟原来的代码在 Python 2 下的效果是一样的。
4.  你可以定义使用多个参数的`lambda`函数。如果没有括号包围在参数周围，Python 2 会把它当作一个包含多个参数的`lambda`函数；在这个`lambda`函数体里，你通过名字引用这些参数，就像在其他类型的函数里所做的一样。这种语法在 Python 3 里仍然有效。

## 特殊的方法属性

在 Python 2 里，类方法可以访问到定义他们的类对象(class object)，也能访问方法对象(method object)本身。`im_self`是类的实例对象；`im_func`是函数对象，`im_class`是类本身。在 Python 3 里，这些属性被重新命名，以遵循其他属性的命名约定。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
|  | `aClassInstance.aClassMethod.im_func` | `aClassInstance.aClassMethod.__func__` |
|  | `aClassInstance.aClassMethod.im_self` | `aClassInstance.aClassMethod.__self__` |
|  | `aClassInstance.aClassMethod.im_class` | `aClassInstance.aClassMethod.__self__.__class__` |

## `__nonzero__`特殊方法

在 Python 2 里，你可以创建自己的类，并使他们能够在布尔上下文(boolean context)中使用。举例来说，你可以实例化这个类，并把这个实例对象用在一个`if`语句中。为了实现这个目的，你定义一个特别的`__nonzero__()`方法，它的返回值为`True`或者`False`，当实例对象处在布尔上下文中的时候这个方法就会被调用 。在 Python 3 里，你仍然可以完成同样的功能，但是这个特殊方法的名字变成了`__bool__()`。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | [1] | [2] |
| ② | [3] | *no change* |

[1]:

```py
class A:
    def __nonzero__(self):
        pass 
```

[2]:

```py
class A:
    def __bool__(self):
        pass 
```

[3]:

```py
class A:
    def __nonzero__(self, x, y):
        pass 
```

1.  当在布尔上下文使用一个类对象时，Python 3 会调用`__bool__()`，而非`__nonzero__()`。
2.  然而，如果你有定义了一个使用两个参数的`__nonzero__()`方法，`2to3`脚本会假设你定义的这个方法有其他用处，因此不会对代码做修改。

## 八进制类型

在 Python 2 和 Python 3 之间，定义八进制(octal)数的语法有轻微的改变。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
|  | `x = 0755` | `x = 0o755` |

## `sys.maxint`

由于长整型和整型被整合在一起了，`sys.maxint`常量不再精确。但是因为这个值对于检测特定平台的能力还是有用处的，所以它被 Python 3 保留，并且重命名为`sys.maxsize`。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | `from sys import maxint` | `from sys import maxsize` |
| ② | `a_function(sys.maxint)` | `a_function(sys.maxsize)` |

1.  `maxint`变成了`maxsize`。
2.  所有的`sys.maxint`都变成了`sys.maxsize`。

## 全局函数`callable()`

在 Python 2 里，你可以使用全局函数`callable()`来检查一个对象是否可调用(callable，比如函数)。在 Python 3 里，这个全局函数被取消了。为了检查一个对象是否可调用，可以检查特殊方法`__call__()`的存在性。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
|  | `callable(anything)` | `hasattr(anything, '__call__')` |

## 全局函数`zip()`

在 Python 2 里，全局函数`zip()`可以使用任意多个序列作为参数，它返回一个由元组构成的列表。第一个元组包含了每个序列的第一个元素；第二个元组包含了每个序列的第二个元素；依次递推下去。在 Python 3 里，`zip()`返回一个迭代器，而非列表。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | `zip(a, b, c)` | `list(zip(a, b, c))` |
| ② | `d.join(zip(a, b, c))` | *no change* |

1.  最简单的形式，你可以通过调用`list()`函数包装`zip()`的返回值来恢复`zip()`函数以前的功能，`list()`函数会遍历这个`zip()`函数返回的迭代器，然后返回结果的列表表示。
2.  在已经会遍历序列所有元素的上下文环境里(比如这里对`join()`方法的调用)，`zip()`返回的迭代器能够正常工作。`2to3`脚本会检测到这些情况，不会对你的代码作出改变。

## `StandardError`异常

在 Python 2 里，`StandardError`是除了`StopIteration`，`GeneratorExit`，`KeyboardInterrupt`，`SystemExit`之外所有其他内置异常的基类。在 Python 3 里，`StandardError`已经被取消了；使用`Exception`替代。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
|  | `x = StandardError()` | `x = Exception()` |
|  | `x = StandardError(a, b, c)` | `x = Exception(a, b, c)` |

## `types`模块中的常量

`types`模块里各种各样的常量能帮助你决定一个对象的类型。在 Python 2 里，它包含了代表所有基本数据类型的常量，如`dict`和`int`。在 Python 3 里，这些常量被已经取消了。只需要使用基础类型的名字来替代。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
|  | `types.UnicodeType` | `str` |
|  | `types.StringType` | `bytes` |
|  | `types.DictType` | `dict` |
|  | `types.IntType` | `int` |
|  | `types.LongType` | `int` |
|  | `types.ListType` | `list` |
|  | `types.NoneType` | `type(None)` |
|  | `types.BooleanType` | `bool` |
|  | `types.BufferType` | `memoryview` |
|  | `types.ClassType` | `type` |
|  | `types.ComplexType` | `complex` |
|  | `types.EllipsisType` | `type(Ellipsis)` |
|  | `types.FloatType` | `float` |
|  | `types.ObjectType` | `object` |
|  | `types.NotImplementedType` | `type(NotImplemented)` |
|  | `types.SliceType` | `slice` |
|  | `types.TupleType` | `tuple` |
|  | `types.TypeType` | `type` |
|  | `types.XRangeType` | `range` |

> ☞`types.StringType`被映射为`bytes`，而非`str`，因为 Python 2 里的“string”(非 Unicode 编码的字符串，即普通字符串)事实上只是一些使用某种字符编码的字节序列(a sequence of bytes)。

## 全局函数`isinstance()`

`isinstance()`函数检查一个对象是否是一个特定类(class)或者类型(type)的实例。在 Python 2 里，你可以传递一个由类型(types)构成的元组给`isinstance()`，如果该对象是元组里的任意一种类型，函数返回`True`。在 Python 3 里，你依然可以这样做，但是不推荐使用把一种类型作为参数传递两次。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
|  | `isinstance(x, (int, float, int))` | `isinstance(x, (int, float))` |

## `basestring`数据类型

Python 2 有两种字符串类型：Unicode 编码的字符串和非 Unicode 编码的字符串。但是其实还有另外 一种类型，即`basestring`。它是一个抽象数据类型，是`str`和`unicode`类型的超类(superclass)。它不能被直接调用或者实例化，但是你可以把它作为`isinstance()`的参数来检测一个对象是否是一个 Unicode 字符串或者非 Unicode 字符串。在 Python 3 里，只有一种字符串类型，所以`basestring`就没有必要再存在了。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
|  | `isinstance(x, basestring)` | `isinstance(x, str)` |

## `itertools`模块

Python 2.3 引入了`itertools`模块，它定义了全局函数`zip()`，`map()`，`filter()`的变体(variant)，这些变体的返回类型为迭代器，而非列表。在 Python 3 里，由于这些全局函数的返回类型本来就是迭代器，所以这些`itertools`里的这些变体函数就被取消了。(在`itertools`模块里仍然还有许多其他的有用的函数，而不仅仅是以上列出的这些。)

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | `itertools.izip(a, b)` | `zip(a, b)` |
| ② | `itertools.imap(a, b)` | `map(a, b)` |
| ③ | `itertools.ifilter(a, b)` | `filter(a, b)` |
| ④ | `from itertools import imap, izip, foo` | `from itertools import foo` |

1.  使用全局的`zip()`函数，而非`itertools.izip()`。
2.  使用`map()`而非`itertools.imap()`。
3.  `itertools.ifilter()`变成了`filter()`。
4.  `itertools`模块在 Python 3 里仍然存在，它只是不再包含那些已经转移到全局名字空间的函数。`2to3`脚本能够足够智能地去移除那些不再有用的导入语句，同时保持其他的导入语句的完整性。

## `sys.exc_type`, `sys.exc_value`, `sys.exc_traceback`

处理异常的时候，在`sys`模块里有三个你可以访问的变量：`sys.exc_type，`sys.exc_value，`sys.exc_traceback`。(实际上这些在 Python 1 的时代就有。)从 Python 1.5 开始，由于新出的`sys.exc_info`，不再推荐使用这三个变量了，这是一个包含所有以上三个元素的元组。在 Python 3 里，这三个变量终于不再存在了；这意味着，你必须使用`sys.exc_info`。``

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
|  | `sys.exc_type` | `sys.exc_info()[0]` |
|  | `sys.exc_value` | `sys.exc_info()[1]` |
|  | `sys.exc_traceback` | `sys.exc_info()[2]` |

## 对元组的列表解析

在 Python 2 里，如果你需要编写一个遍历元组的列表解析，你不需要在元组值的周围加上括号。在 Python 3 里，这些括号是必需的。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
|  | `[i for i in 1, 2]` | `[i for i in (1, 2)]` |

## `os.getcwdu()`函数

Python 2 有一个叫做`os.getcwd()`的函数，它将当前的工作目录作为一个(非 Unicode 编码的)字符串返回。由于现代的文件系统能够处理能何字符编码的目录名，Python 2.3 引入了`os.getcwdu()`函数。`os.getcwdu()`函数把当前工作目录用 Unicode 编码的字符串返回。在 Python 3 里，由于只有一种字符串类型(Unicode 类型的)，所以你只需要`os.getcwd()`就可以了。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
|  | `os.getcwdu()` | `os.getcwd()` |

## 元类(metaclass)

在 Python 2 里，你可以通过在类的声明中定义`metaclass`参数，或者定义一个特殊的类级别的(class-level)`__metaclass__`属性，来创建元类。在 Python 3 里，`__metaclass__`属性已经被取消了。

| Notes | Python 2 | Python 3 |
| --- | --- | --- |
| ① | [1] | *unchanged* |
| ② | [2] | [3] |
| ③ | [4] | [5] |

[1]:

```py
class C(metaclass=PapayaMeta):
    pass 
```

[2]:

```py
class Whip:
    __metaclass__ = PapayaMeta 
```

[3]:

```py
class Whip(metaclass=PapayaMeta):
    pass 
```

[4]:

```py
class C(Whipper, Beater):
    __metaclass__ = PapayaMeta 
```

[5]:

```py
class C(Whipper, Beater, metaclass=PapayaMeta):
    pass 
```

1.  在声明类的时候声明`metaclass`参数，这在 Python 2 和 Python 3 里都有效，它们是一样的。
2.  在类的定义里声明`__metaclass__`属性在 Python 2 里有效，但是在 Python 3 里不再有效。
3.  `2to3`能够构建一个有效的类声明，即使这个类继承自多个父类。

## 关于代码风格

以下所列的“修补”(fixes)实质上并不算真正的修补。意思就是，他们只是代码的风格上的事情，而不涉及到代码的本质。但是 Python 的开发者们在使得代码风格尽可能一致方面非常有兴趣(have a vested interest)。为此，有一个专门[o 描述 Python 代码风格的官方指导手册](http://www.python.org/dev/peps/pep-0008/) — 细致到能使人痛苦 — 都是一些你不太可能关心的在各种各样的细节上的挑剔。鉴于`2to3`为转换代码提供了一个这么好的条件，脚本的作者们添加了一些可选的特性以使你的代码更具可读性。

### `set()`字面值(literal)(显式的)

在 Python 2 城，定义一个字面值集合(literal set)的唯一方法就是调用`set(a_sequence)`。在 Python 3 里这仍然有效，但是使用新的标注记号(literal notation)：大括号({})是一种更清晰的方法。这种方法除了空集以外都有效，因为字典也用大括号标记，所以`{}`表示一个空的字典，而不是一个空集。

> ☞`2to3`脚本默认不会修复`set()`字面值。为了开启这个功能，在命令行调用`2to3`的时候指定`-f set_literal`参数。

| Notes | Before | After |
| --- | --- | --- |
|  | `set([1, 2, 3])` | `{1, 2, 3}` |
|  | `set((1, 2, 3))` | `{1, 2, 3}` |
|  | `set([i for i in a_sequence])` | `{i for i in a_sequence}` |

### 全局函数`buffer()`(显式的)

用 C 实现的 Python 对象可以导出一个“缓冲区接口”(buffer interface)，它允许其他的 Python 代码直接读写一块内存。(这听起来很强大，它也同样可怕。)在 Python 3 里，`buffer()`被重新命名为`memoryview()`。(实际的修改更加复杂，但是你几乎可以忽略掉这些不同之处。)

> ☞`2to3`脚本默认不会修复`buffer()`函数。为了开启这个功能，在命令行调用`2to3`的时候指定`-f buffer`参数。

| Notes | Before | After |
| --- | --- | --- |
|  | `x = buffer(y)` | `x = memoryview(y)` |

### 逗号周围的空格(显式的)

尽管 Python 对用于缩进和凸出(indenting and outdenting)的空格要求很严格，但是对于空格在其他方面的使用 Python 还是很自由的。在列表，元组，集合和字典里，空格可以出现在逗号的前面或者后面，这不会有什么坏影响。但是，Python 代码风格指导手册上指出，逗号前不能有空格，逗号后应该包含一个空格。尽管这纯粹只是一个美观上的考量(代码仍然可以正常工作，在 Python 2 和 Python 3 里都可以)，但是`2to3`脚本可以依据手册上的标准为你完成这个修复。

> ☞`2to3`脚本默认不会修复逗号周围的空格。为了开启这个功能，在命令行调用`2to3`的时候指定`-f wscomma`参数。

| Notes | Before | After |
| --- | --- | --- |
|  | `a ,b` | `a, b` |
|  | `{a :b}` | `{a: b}` |

### 惯例(Common idioms)(显式的)

在 Python 社区里建立起来了许多惯例。有一些比如`while 1:` loop，它可以追溯到 Python 1。(Python 直到 Python 2.3 才有真正意义上的布尔类型，所以开发者以前使用`1`和`0`替代。)当代的 Python 程序员应该锻炼他们的大脑以使用这些惯例的现代版。

> ☞`2to3`脚本默认不会为这些惯例做修复。为了开启这个功能，在命令行调用`2to3`的时候指定`-f idioms`参数。

| Notes | Before | After |
| --- | --- | --- |
|  | [1] | [2] |
|  | `type(x) == T` | `isinstance(x, T)` |
|  | `type(x) is T` | `isinstance(x, T)` |
|  | [3] | [4] |

[1]:

```py
while 1:
    do_stuff() 
```

[2]:

```py
while True:
    do_stuff() 
```

[3]:

```py
a_list = list(a_sequence)
a_list.sort()
do_stuff(a_list) 
```

[4]:

```py
a_list = sorted(a_sequence)
do_stuff(a_list) 
```