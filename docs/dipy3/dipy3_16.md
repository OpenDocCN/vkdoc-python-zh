# Chapter 13 序列化 Python 对象

> " Every Saturday since we’ve lived in this apartment, I have awakened at 6:15, poured myself a bowl of cereal, added a quarter-cup of 2% milk, sat on **this** end of **this** couch, turned on BBC America, and watched Doctor Who. " — Sheldon, [The Big Bang Theory](http://en.wikiquote.org/wiki/The_Big_Bang_Theory#The_Dumpling_Paradox_.5B1.07.5D)

## 深入

序列化的概念很简单。内存里面有一个数据结构，你希望将它保存下来，重用，或者发送给其他人。你会怎么做？嗯, 这取决于你想要怎么保存，怎么重用，发送给谁。很多游戏允许你在退出的时候保存进度，然后你再次启动的时候回到上次退出的地方。(实际上, 很多非游戏程序也会这么干。) 在这个情况下, 一个捕获了当前进度的数据结构需要在你退出的时候保存到磁盘上，接着在你重新启动的时候从磁盘上加载进来。这个数据只会被创建它的程序使用，不会发送到网络上，也不会被其它程序读取。因此，互操作的问题被限制在保证新版本的程序能够读取以前版本的程序创建的数据。

在这种情况下，`pickle` 模块是理想的。它是 Python 标准库的一部分, 所以它总是可用的。它很快; 它的大部分同 Python 解释器本身一样是用 C 写的。 它可以存储任意复杂的 Python 数据结构。

什么东西能用`pickle`模块存储?

*   所有 Python 支持的 原生类型 : 布尔, 整数, 浮点数, 复数, 字符串, `bytes`(字节串)对象, 字节数组, 以及 `None`.
*   由任何原生类型组成的列表，元组，字典和集合。
*   由任何原生类型组成的列表，元组，字典和集合组成的列表，元组，字典和集合(可以一直嵌套下去，直至[Python 支持的最大递归层数](http://docs.python.org/3.1/library/sys.html#sys.getrecursionlimit "sys.getrecursionlimit()")).
*   函数，类，和类的实例(带警告)。

如果这还不够用，`pickle`模块也是可扩展的。如果你对可扩展性有兴趣，请查看本章最后的进一步阅读小节中的链接。

### 本章例子的快速笔记

本章会使用两个 Python Shell 来讲故事。本章的例子都是一个单独的故事的一部分。当我演示`pickle` 和 `json` 模块时，你会被要求在两个 Python Shell 中来回切换。

为了让事情简单一点，打开 Python Shell 并定义下面的变量:

```py
>>> shell = 1 
```

保持该窗口打开。 现在打开另一个 Python Shell 并定义下面下面的变量:

```py
>>> shell = 2 
```

贯穿整个章节, 在每个例子中我会使用`shell`变量来标识使用的是哪个 Python Shell。

## 保存数据到 Pickle 文件

`pickle`模块的工作对象是数据结构。让我们来创建一个：

```py
 1

>>> entry['title'] = 'Dive into history, 2009 edition'
>>> entry['article_link'] = 'http://diveintomark.org/archives/2009/03/27/dive-into-history-2009-edition'
>>> entry['comments_link'] = None
>>> entry['internal_id'] = b'\xDE\xD5\xB4\xF8'
>>> entry['tags'] = ('diveintopython', 'docbook', 'html')
>>> entry['published'] = True
>>> import time

>>> entry['published_date']
time.struct_time(tm_year=2009, tm_mon=3, tm_mday=27, tm_hour=22, tm_min=20, tm_sec=42, tm_wday=4, tm_yday=86, tm_isdst=-1) 
```

1.  在 Python Shell #1 里面。
2.  想法是建立一个 Python 字典来表示一些有用的东西，比如一个 Atom 供稿的 entry。但是为了炫耀一下`pickle`模块我也想保证里面包含了多种不同的数据类型。不需要太关心这些值。
3.  `time` 模块包含一个表示时间点(精确到 1 毫秒)的数据结构(`time_struct`)以及操作时间结构的函数。`strptime()`函数接受一个格式化过的字符串并将其转化成一个`time_struct`。这个字符串使用的是默认格式，但你可以通过格式化代码来控制它。查看[`time`模块](http://docs.python.org/3.1/library/time.html)来获得更多细节。

这是一个很帅的 Python 字典。让我们把它保存到文件。

```py
 1
>>> import pickle

... 
```

1.  仍然在 Python Shell #1 中。
2.  使用`open()` 函数来打开一个文件。设置文件模式为`'wb'`来以二进制写模式打开文件。把它放入`with` 语句中来保证在你完成的时候文件自动被关闭。
3.  `pickle`模块中的`dump()`函数接受一个可序列化的 Python 数据结构, 使用最新版本的 pickle 协议将其序列化为一个二进制的，Python 特定的格式， 并且保存到一个打开的文件里。

最后一句话很重要。

*   `pickle`模块接受一个 Python 数据结构并将其保存的一个文件。
*   要做到这样，它使用一个被称为“pickle 协议”的东西*序列化*该数据结构。
*   pickle 协议是 Python 特定的，没有任何跨语言兼容的保证。你很可能不能使用 Perl, PHP, Java, 或者其他语言来对你刚刚创建的`entry.pickle`文件做任何有用的事情。
*   并非所有的 Python 数据结构都可以通过`pickle`模块序列化。随着新的数据类型被加入到 Python 语言中，pickle 协议已经被修改过很多次了，但是它还是有一些限制。
*   由于这些变化，不同版本的 Python 的兼容性也没有保证。新的版本的 Python 支持旧的序列化格式，但是旧版本的 Python 不支持新的格式(因为它们不支持新的数据类型)。
*   除非你指定，`pickle`模块中的函数将使用最新版本的 pickle 协议。这保证了你对可以被序列化的数据类型有最大的灵活度，但这也意味着生成的文件不能被不支持新版 pickle 协议的旧版本的 Python 读取。
*   最新版本的 pickle 协议是二进制格式的。请确认使用二进制模式来打开你的 pickle 文件,否则当你写入的时候数据会被损坏。

## 从 Pickle 文件读取数据

现在切换到你的第二个 Python Shell — *即*不是你创建`entry`字典的那个。

```py
 2

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'entry' is not defined
>>> import pickle

... 

{'comments_link': None,
 'internal_id': b'\xDE\xD5\xB4\xF8',
 'title': 'Dive into history, 2009 edition',
 'tags': ('diveintopython', 'docbook', 'html'),
 'article_link':
 'http://diveintomark.org/archives/2009/03/27/dive-into-history-2009-edition',
 'published_date': time.struct_time(tm_year=2009, tm_mon=3, tm_mday=27, tm_hour=22, tm_min=20, tm_sec=42, tm_wday=4, tm_yday=86, tm_isdst=-1),
 'published': True} 
```

1.  这是 Python Shell #2.
2.  这里没有`entry` 变量被定义过。你在 Python Shell #1 中定义了`entry`变量, 但是那是另一个拥有自己状态的完全不同的环境。
3.  打开你在 Python Shell #1 中创建的`entry.pickle`文件。`pickle`模块使用二进制数据格式，所以你总是应该使用二进制模式打开 pickle 文件。
4.  `pickle.load()`函数接受一个流对象, 从流中读取序列化后的数据，创建一个新的 Python 对象，在新的 Python 对象中重建被序列化的数据，然后返回新建的 Python 对象。
5.  现在`entry`变量是一个键和值看起来都很熟悉的字典。

`pickle.dump() / pickle.load()`循环的结果是一个和原始数据结构等同的新的数据结构。

```py
 1

... 

True

False

('diveintopython', 'docbook', 'html')
>>> entry2['internal_id']
b'\xDE\xD5\xB4\xF8' 
```

1.  切换回 Python Shell #1。
2.  打开`entry.pickle`文件。
3.  将序列化后的数据装载到一个新的变量, `entry2`。
4.  Python 确认两个字典, `entry` 和 `entry2` 是相等的。在这个 shell 里, 你从零开始构造了`entry`, 从一个空字典开始然后手工给各个键赋值。你序列化了这个字典并将其保存在`entry.pickle`文件中。现在你从文件中读取序列化后的数据并创建了原始数据结构的一个完美复制品。
5.  相等和相同是不一样的。我说的是你创建了原始数据结构的一个*完美复制品*, 这没错。但它仅仅是一个复制品。
6.  我要指出`'tags'`键对应的值是一个元组，而`'internal_id'`键对应的值是一个`bytes`对象。原因在这章的后面就会清楚了。

## 不使用文件来进行序列化

前一节中的例子展示了如果将一个 Python 对象序列化到磁盘文件。但如果你不想或不需要文件呢？你也可以序列化到一个内存中的`bytes`对象。

```py
>>> shell
1

<class 'bytes'>

True 
```

1.  `pickle.dumps()`函数(注意函数名最后的`'s'`)执行和`pickle.dump()`函数相同的序列化。取代接受流对象并将序列化后的数据保存到磁盘文件，这个函数简单的返回序列化的数据。
2.  由于 pickle 协议使用一个二进制数据格式，所以`pickle.dumps()`函数返回`bytes`对象。
3.  `pickle.loads()`函数(再一次, 注意函数名最后的`'s'`) 执行和`pickle.load()`函数一样的反序列化。取代接受一个流对象并去文件读取序列化后的数据，它接受包含序列化后的数据的`bytes`对象, 比如`pickle.dumps()`函数返回的对象。
4.  最终结果是一样的: 原始字典的完美复制。

## 字节串和字符串又一次抬起了它们丑陋的头。

pickle 协议已经存在好多年了，它随着 Python 本身的成熟也不断成熟。现在存在[四个不同版本](http://docs.python.org/3.1/library/pickle.html#data-stream-format) 的 pickle 协议。

*   Python 1.x 有两个 pickle 协议，一个基于文本的格式(“版本 0”) 以及一个二进制格式(“版本 1”).
*   Python 2.3 引入了一个新的 pickle 协议(“版本 2”) 来处理 Python 类对象的新功能。它是一个二进制格式。
*   Python 3.0 引入了另一个 pickle 协议 (“版本 3”) ，显式的支持`bytes` 对象和字节数组。它是一个二进制格式。

你看, 字节串和字符串的区别又一次抬起了它们丑陋的头。 (如果你觉得惊奇，你肯定开小差了。) 在实践中这意味着, 尽管 Python 3 可以读取版本 2 的 pickle 协议生成的数据, Python 2 不能读取版本 3 的协议生成的数据.

## 调试 Pickle 文件

pickle 协议是长什么样的呢？让我们离开 Python Shell 一会会，来看一下我们创建的`entry.pickle`文件。

```py
you@localhost:~/diveintopython3/examples$ ls -l entry.pickle
-rw-r--r-- 1 you  you  358 Aug  3 13:34 entry.pickle
you@localhost:~/diveintopython3/examples$ cat entry.pickle
comments_linkqNXtagsqXdiveintopythonqXdocbookqXhtmlq?qX publishedq?
XlinkXJhttp://diveintomark.org/archives/2009/03/27/dive-into-history-2009-edition
q   Xpublished_dateq
ctime
struct_time
?qRqXtitleqXDive into history, 2009 editionqu. 
```

这不是很有用。你可以看见字符串，但是其他数据类型显示为不可打印的(或者至少是不可读的)字符。域之间没有明显的分隔符(比如跳格符或空格)。你肯定不希望来调试这样一个格式。

```py
>>> shell
1
>>> import pickletools
>>> with open('entry.pickle', 'rb') as f:
...     pickletools.dis(f)
 0: \x80 PROTO      3
    2: }    EMPTY_DICT
    3: q    BINPUT     0
    5: (    MARK
    6: X        BINUNICODE 'published_date'
   25: q        BINPUT     1
   27: c        GLOBAL     'time struct_time'
   45: q        BINPUT     2
   47: (        MARK
   48: M            BININT2    2009
   51: K            BININT1    3
   53: K            BININT1    27
   55: K            BININT1    22
   57: K            BININT1    20
   59: K            BININT1    42
   61: K            BININT1    4
   63: K            BININT1    86
   65: J            BININT     -1
   70: t            TUPLE      (MARK at 47)
   71: q        BINPUT     3
   73: }        EMPTY_DICT
   74: q        BINPUT     4
   76: \x86     TUPLE2
   77: q        BINPUT     5
   79: R        REDUCE
   80: q        BINPUT     6
   82: X        BINUNICODE 'comments_link'
  100: q        BINPUT     7
  102: N        NONE
  103: X        BINUNICODE 'internal_id'
  119: q        BINPUT     8
  121: C        SHORT_BINBYTES '脼脮麓酶'
  127: q        BINPUT     9
  129: X        BINUNICODE 'tags'
  138: q        BINPUT     10
  140: X        BINUNICODE 'diveintopython'
  159: q        BINPUT     11
  161: X        BINUNICODE 'docbook'
  173: q        BINPUT     12
  175: X        BINUNICODE 'html'
  184: q        BINPUT     13
  186: \x87     TUPLE3
  187: q        BINPUT     14
  189: X        BINUNICODE 'title'
  199: q        BINPUT     15
  201: X        BINUNICODE 'Dive into history, 2009 edition'
  237: q        BINPUT     16
  239: X        BINUNICODE 'article_link'
  256: q        BINPUT     17
  258: X        BINUNICODE 'http://diveintomark.org/archives/2009/03/27/dive-into-history-2009-edition'
  337: q        BINPUT     18
  339: X        BINUNICODE 'published'
  353: q        BINPUT     19
  355: \x88     NEWTRUE
  356: u        SETITEMS   (MARK at 5)
  357: .    STOP
<mark>highest protocol among opcodes = 3</mark> 
```

这个反汇编中最有趣的信息是最后一行, 因为它包含了文件保存时使用的 pickle 协议的版本号。在 pickle 协议里面没有明确的版本标志。为了确定保存 pickle 文件时使用的协议版本，你需要查看序列化后的数据的标记(“opcodes”)并且使用硬编码的哪个版本的协议引入了哪些标记的知识(来确定版本号)。`pickle.dis()`函数正是这么干的，并且它在反汇编的输出的最后一行打印出结果。下面是一个不打印，仅仅返回版本号的函数:

[下载 `pickleversion.py`]

```py
import pickletools

def protocol_version(file_object):
    maxproto = -1
    for opcode, arg, pos in pickletools.genops(file_object):
        maxproto = max(maxproto, opcode.proto)
    return maxproto 
```

实际使用它:

```py
>>> import pickleversion
>>> with open('entry.pickle', 'rb') as f:
...     v = pickleversion.protocol_version(f)
>>> v
3 
```

## 序列化 Python 对象以供其它语言读取

`pickle`模块使用的数据格式是 Python 特定的。它没有做任何兼容其它编程语言的努力。如果跨语言兼容是你的需求之一，你得去寻找其它的序列化格式。一个这样的格式是[JSON](http://json.org/)。 “JSON” 代表 “JavaScript Object Notation,” 但是不要让名字糊弄你。 — JSON 是被设计为跨语言使用的。

Python 3 在标准库中包含了一个 `json`模块。同 `pickle`模块类似, `json`模块包含一些函数，可以序列化数据结构，保存序列化后的数据至磁盘，从磁盘上读取序列化后的数据，将数据反序列化成新的 Pythone 对象。但两者也有一些很重要的区别。 首先, JSON 数据格式是基于文本的, 不是二进制的。[RFC 4627](http://www.ietf.org/rfc/rfc4627.txt) 定义了 JSON 格式以及怎样将各种类型的数据编码成文本。比如，一个布尔值要么存储为 5 个字符的字符串`'false'`，要么存储为 4 个字符的字符串 `'true'`。 所有的 JSON 值都是大小写敏感的。

第二，由于是文本格式, 存在空白(whitespaces)的问题。 JSON 允许在值之间有任意数目的空白(空格, 跳格, 回车，换行)。空白是“无关紧要的”，这意味着 JSON 编码器可以按它们的喜好添加任意多或任意少的空白, 而 JSON 解码器被要求忽略值之间的任意空白。这允许你“美观的打印（pretty-print）” 你的 JSON 数据, 通过不同的缩进层次嵌套值，这样你就可以在标准浏览器或文本编辑器中阅读它。Python 的 `json` 模块有在编码时执行美观打印（pretty-printing）的选项。

第三, 字符编码的问题是长期存在的。JSON 用纯文本编码数据, 但是你知道, “不存在纯文本这种东西。” JSON 必须以 Unicode 编码(UTF-32, UTF-16, 或者默认的, UTF-8)方式存储, [RFC 4627 的第 3 节](http://www.ietf.org/rfc/rfc4627.txt) 定义了如何区分使用的是哪种编码。

## 将数据保存至 JSON 文件

JSON 看起来非常像你在 Javascript 中手工定义的数据结构。这不是意外; 实际上你可以使用 JavaScript 的`eval()`函数来“解码” JSON 序列化过的数据。(通常的对非信任输入的警告 也适用, 但关键点是 JSON *是* 合法的 JavaScript。) 因此, 你可能已经熟悉 JSON 了。

```py
>>> shell
1

>>> basic_entry['id'] = 256
>>> basic_entry['title'] = 'Dive into history, 2009 edition'
>>> basic_entry['tags'] = ('diveintopython', 'docbook', 'html')
>>> basic_entry['published'] = True
>>> basic_entry['comments_link'] = None
>>> import json 
```

1.  我们将创建一个新的数据结构，而不是重用现存的`entry`数据结构。在这章的后面, 我们将会看见当我们试图用 JSON 编码更复杂的数据结构的时候会发生什么。
2.  JSON 是一个基于文本的格式， 这意味你可以以文本模式打开文件，并给定一个字符编码。用 UTF-8 总是没错的。
3.  同`pickle`模块一样, `json` 模块定义了`dump()`函数，它接受一个 Python 数据结构和一个可写的流对象。`dump()` 函数将 Python 数据结构序列化并写入到流对象中。在`with`语句内工作保证当我们完成的时候正确的关闭文件。

那么生成的 JSON 序列化数据是什么样的呢？

```py
you@localhost:~/diveintopython3/examples$ cat basic.json
{"published": true, "tags": ["diveintopython", "docbook", "html"], "comments_link": null,
"id": 256, "title": "Dive into history, 2009 edition"} 
```

这肯定比 pickle 文件更可读。然而 JSON 的值之间可以包含任意数目的空把, 并且`json`模块提供了一个方便的途径来利用这一点生成更可读的 JSON 文件。

```py
>>> shell
1
>>> with open('basic-pretty.json', mode='w', encoding='utf-8') as f: 
```

1.  如果你给`json.dump()`函数传入`indent`参数, 它以文件变大为代价使生成的 JSON 文件更可读。`indent` 参数是一个整数。0 意味着“每个值单独一行。” 大于 0 的数字意味着“每个值单独一行并且使用这个数目的空格来缩进嵌套的数据结构。”

这是结果:

```py
you@localhost:~/diveintopython3/examples$ cat basic-pretty.json
{
  "published": true, 
  "tags": [
    "diveintopython", 
    "docbook", 
    "html"
  ], 
  "comments_link": null, 
  "id": 256, 
  "title": "Dive into history, 2009 edition"
} 
```

## 将 Python 数据类型映射到 JSON

由于 JSON 不是 Python 特定的，对应到 Python 的数据类型的时候有很多不匹配。有一些仅仅是名字不同，但是有两个 Python 数据类型完全缺少。看看你能能把它们指出来:

| 笔记 | JSON | Python 3 |
| --- | --- | --- |
|  | object | dictionary |
|  | array | list |
|  | string | string |
|  | integer | integer |
|  | real number | float |
| * | `true` | `True` |
| * | `false` | `False` |
| * | `null` | `None` |
| * | 所有的 JSON 值都是大小写敏感的。 |

注意到什么被遗漏了吗？元组和 *&* 字节串（bytes）! JSON 有数组类型, `json` 模块将其映射到 Python 的列表, 但是它没有一个单独的类型对应 “冻结数组(frozen arrays)” (元组)。而且尽管 JSON 非常好的支持字符串，但是它没有对`bytes` 对象或字节数组的支持。

## 序列化 JSON 不支持的数据类型

即使 JSON 没有内建的字节流支持, 并不意味着你不能序列化`bytes`对象。`json`模块提供了编解码未知数据类型的扩展接口。(“未知”的意思是≴JSON 没有定义”。很显然`json` 模块认识字节数组, 但是它被 JSON 规范的限制束缚住了。) 如果你希望编码字节串或者其它 JSON 没有原生支持的数据类型，你需要给这些类型提供定制的编码和解码器。

```py
>>> shell
1

{'comments_link': None,
 'internal_id': b'\xDE\xD5\xB4\xF8',
 'title': 'Dive into history, 2009 edition',
 'tags': ('diveintopython', 'docbook', 'html'),
 'article_link': 'http://diveintomark.org/archives/2009/03/27/dive-into-history-2009-edition',
 'published_date': time.struct_time(tm_year=2009, tm_mon=3, tm_mday=27, tm_hour=22, tm_min=20, tm_sec=42, tm_wday=4, tm_yday=86, tm_isdst=-1),
 'published': True}
>>> import json

... 
Traceback (most recent call last):
  File "<stdin>", line 5, in <module>
  File "C:\Python31\lib\json\__init__.py", line 178, in dump
    for chunk in iterable:
  File "C:\Python31\lib\json\encoder.py", line 408, in _iterencode
    for chunk in _iterencode_dict(o, _current_indent_level):
  File "C:\Python31\lib\json\encoder.py", line 382, in _iterencode_dict
    for chunk in chunks:
  File "C:\Python31\lib\json\encoder.py", line 416, in _iterencode
    o = _default(o)
  File "C:\Python31\lib\json\encoder.py", line 170, in default
    raise TypeError(repr(o) + " is not JSON serializable")
<mark>TypeError: b'\xDE\xD5\xB4\xF8' is not JSON serializable</mark> 
```

1.  好的, 是时间再看看`entry` 数据结构了。它包含了所有的东西: 布尔值，`None`值，字符串，字符串元组, `bytes`对象, 以及`time`结构体。
2.  我知道我已经说过了，但是这值得再重复一次：JSON 是一个基于文本的格式。总是应使用 UTF-8 字符编码以文本模式打开 JSON 文件。
3.  嗯，*这*可不好。发生什么了？

情况是这样的: `json.dump()` 函数试图序列化`bytes`对象 `b'\xDE\xD5\xB4\xF8'`，但是它失败了，原因是 JSON 不支持`bytes`对象。然而, 如果保存字节串对你来说很重要，你可以定义自己的“迷你序列化格式。”

1.  为了给一个 JSON 没有原生支持的数据类型定义你自己的“迷你序列化格式”, 只要定义一个接受一个 Python 对象为参数的函数。这个对象将会是`json.dump()`函数无法自己序列化的实际对象 — 这个例子里是`bytes` 对象 `b'\xDE\xD5\xB4\xF8'`。
2.  你的自定义序列化函数应该检查`json.dump()`函数传给它的对象的类型。当你的函数只序列化一个类型的时候这不是必须的，但是它使你的函数的覆盖的内容清楚明白，并且在你需要序列化更多类型的时候更容易扩展。
3.  在这个例子里面, 我将`bytes` 对象转换成字典。`__class__` 键持有原始的数据类型(以字符串的形式, `'bytes'`), 而 `__value__` 键持有实际的数据。当然它不能是`bytes`对象; 大体的想法是将其转换成某些可以被 JSON 序列化的东西! `bytes`对象就是一个范围在 0–255 的整数的序列。 我们可以使用`list()` 函数将`bytes`对象转换成整数列表。所以`b'\xDE\xD5\xB4\xF8'` 变成 `[222, 213, 180, 248]`. (算一下! 这是对的! 16 进制的字节 `\xDE` 是十进制的 222, `\xD5` 是 213, 以此类推。)
4.  这一行很重要。你序列化的数据结构可能包含 JSON 内建的可序列化类型和你的定制序列化器支持的类型之外的东西。在这种情况下，你的定制序列化器抛出一个`TypeError`，那样`json.dump()` 函数就可以知道你的定制序列化函数不认识该类型。

就这么多；你不需要其它的东西。特别是, 这个定制序列化函数*返回 Python 字典*，不是字符串。你不是自己做所有序列化到 JSON 的工作; 你仅仅在做转换成被支持的类型那部分工作。`json.dump()` 函数做剩下的事情。

```py
>>> shell
1

... 
Traceback (most recent call last):
  File "<stdin>", line 9, in <module>
    json.dump(entry, f, default=customserializer.to_json)
  File "C:\Python31\lib\json\__init__.py", line 178, in dump
    for chunk in iterable:
  File "C:\Python31\lib\json\encoder.py", line 408, in _iterencode
    for chunk in _iterencode_dict(o, _current_indent_level):
  File "C:\Python31\lib\json\encoder.py", line 382, in _iterencode_dict
    for chunk in chunks:
  File "C:\Python31\lib\json\encoder.py", line 416, in _iterencode
    o = _default(o)
  File "/Users/pilgrim/diveintopython3/examples/customserializer.py", line 12, in to_json

TypeError: time.struct_time(tm_year=2009, tm_mon=3, tm_mday=27, tm_hour=22, tm_min=20, tm_sec=42, tm_wday=4, tm_yday=86, tm_isdst=-1) is not JSON serializable 
```

1.  `customserializer` 模块是你在前一个例子中定义`to_json()`函数的地方。
2.  文本模式, UTF-8 编码, yadda yadda。(你很可能会忘记这一点! 我就忘记过好几次! 事情一切正常直到它失败的时刻, 而它的失败很令人瞩目。)
3.  这是重点: 为了将定制转换函数钩子嵌入`json.dump()`函数, 只要将你的函数以`default`参数传入`json.dump()`函数。(万岁, Python 里一切皆对象!)
4.  好吧, 实际上还是不能工作。但是看一下异常。`json.dump()` 函数不再抱怨无法序列化`bytes`对象了。现在它在抱怨另一个完全不同的对象: `time.struct_time` 对象。

尽管得到另一个不同的异常看起来不是什么进步, 但它确实是个进步! 再调整一下就可以解决这个问题。

```py
 import time

def to_json(python_object):

    if isinstance(python_object, bytes):
        return {'__class__': 'bytes',
                '__value__': list(python_object)}
    raise TypeError(repr(python_object) + ' is not JSON serializable') 
```

1.  在现存的`customserializer.to_json()`函数里面, 我们加入了 Python 对象 (`json.dump()` 处理不了的那些) 是不是 `time.struct_time`的判断。
2.  如果是的，我们做一些同处理`bytes`对象时类似的事情来转换: 将`time.struct_time` 结构转化成一个只包含 JSON 可序列化值的字典。在这个例子里, 最简单的将日期时间转换成 JSON 可序列化值的方法是使用`time.asctime()`函数将其转换成字符串。`time.asctime()` 函数将难看的`time.struct_time` 转换成字符串 `'Fri Mar 27 22:20:42 2009'`。

有了两个定制的转换, 整个`entry` 数据结构序列化到 JSON 应该没有进一步的问题了。

```py
>>> shell
1
>>> with open('entry.json', 'w', encoding='utf-8') as f:
...     json.dump(entry, f, default=customserializer.to_json)
... 
```

```py
you@localhost:~/diveintopython3/examples$ ls -l example.json
-rw-r--r-- 1 you  you  391 Aug  3 13:34 entry.json
you@localhost:~/diveintopython3/examples$ cat example.json
{"published_date": {"__class__": "time.asctime", "__value__": "Fri Mar 27 22:20:42 2009"},
"comments_link": null, "internal_id": {"__class__": "bytes", "__value__": [222, 213, 180, 248]},
"tags": ["diveintopython", "docbook", "html"], "title": "Dive into history, 2009 edition",
"article_link": "http://diveintomark.org/archives/2009/03/27/dive-into-history-2009-edition",
"published": true} 
```

## 从 JSON 文件加载数据

类似`pickle` 模块，`json`模块有一个`load()`函数接受一个流对象，从中读取 JSON 编码过的数据, 并且创建该 JSON 数据结构的 Python 对象的镜像。

```py
>>> shell
2

>>> entry
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'entry' is not defined
>>> import json
>>> with open('entry.json', 'r', encoding='utf-8') as f:

... 

{'comments_link': None,
 'internal_id': {'__class__': 'bytes', '__value__': [222, 213, 180, 248]},
 'title': 'Dive into history, 2009 edition',
 'tags': ['diveintopython', 'docbook', 'html'],
 'article_link': 'http://diveintomark.org/archives/2009/03/27/dive-into-history-2009-edition',
 'published_date': {'__class__': 'time.asctime', '__value__': 'Fri Mar 27 22:20:42 2009'},
 'published': True} 
```

1.  为了演示目的，切换到 Python Shell #2 并且删除在这一章前面使用`pickle`模块创建的`entry`数据结构。
2.  最简单的情况下，`json.load()`函数同`pickle.load()`函数的结果一模一样。你传入一个流对象，它返回一个新的 Python 对象。
3.  有好消息也有坏消息。好消息先来: `json.load()` 函数成功的读取了你在 Python Shell #1 中创建的`entry.json`文件并且生成了一个包含那些数据的新的 Python 对象。接着是坏消息: 它没有重建原始的 `entry` 数据结构。`'internal_id'` 和 `'published_date'` 这两个值被重建为字典 — 具体来说, 你在`to_json()`转换函数中使用 JSON 兼容的值创建的字典。

`json.load()` 并不知道你可能传给`json.dump()`的任何转换函数的任何信息。你需要的是`to_json()`函数的逆函数 — 一个接受定制转换出的 JSON 对象并将其转换回原始的 Python 数据类型。

```py
# add this to customserializer.py

        if json_object['__class__'] == 'time.asctime':

        if json_object['__class__'] == 'bytes':

    return json_object 
```

1.  这函数也同样接受一个参数返回一个值。但是参数不是字符串，而是一个 Python 对象 — 反序列化一个 JSON 编码的字符串为 Python 的结果。
2.  你只需要检查这个对象是否包含`to_json()`函数创建的`'__class__'`键。如果是的，`'__class__'`键对应的值将告诉你如何将值解码成原来的 Python 数据类型。
3.  为了解码由`time.asctime()`函数返回的字符串，你要使用`time.strptime()`函数。这个函数接受一个格式化过的时间字符串(格式可以自定义，但默认值同`time.asctime()`函数的默认值相同) 并且返回`time.struct_time`.
4.  为了将整数列表转换回`bytes` 对象, 你可以使用 `bytes()` 函数。

就是这样; `to_json()`函数处理了两种数据类型，现在这两个数据类型也在`from_json()`函数里面处理了。下面是结果:

```py
>>> shell
2
>>> import customserializer
>>> with open('entry.json', 'r', encoding='utf-8') as f:

... 

{'comments_link': None,
 'internal_id': b'\xDE\xD5\xB4\xF8',
 'title': 'Dive into history, 2009 edition',
 'tags': ['diveintopython', 'docbook', 'html'],
 'article_link': 'http://diveintomark.org/archives/2009/03/27/dive-into-history-2009-edition',
 'published_date': time.struct_time(tm_year=2009, tm_mon=3, tm_mday=27, tm_hour=22, tm_min=20, tm_sec=42, tm_wday=4, tm_yday=86, tm_isdst=-1),
 'published': True} 
```

1.  为了将`from_json()`函数嵌入到反序列化过程中，把它作为`object_hook` 参数传入到`json.load()`函数中。接受函数作为参数的函数; 真方便!
2.  `entry` 数据结构现在有一个值为`bytes`对象的`'internal_id'`键。它也包含一个`'published_date'`键，其值为`time.struct_time`对象。

然而，还有最后一个缺陷。

```py
>>> shell
1
>>> import customserializer
>>> with open('entry.json', 'r', encoding='utf-8') as f:
...     entry2 = json.load(f, object_hook=customserializer.from_json)
... 

False

('diveintopython', 'docbook', 'html')

['diveintopython', 'docbook', 'html'] 
```

1.  即使在序列化过程中加入了`to_json()`钩子函数, 也在反序列化过程中加入`from_json()`钩子函数, 我们仍然没有重新创建原始数据结构的完美复制品。为什么没有？
2.  在原始的`entry` 数据结构中, `'tags'`键的值为一个三个字符串组成的元组。
3.  但是重现创建的`entry2` 数据结构中, `'tags'` 键的值是一个三个字符串组成的*列表*。JSON 并不区分元组和列表；它只有一个类似列表的数据类型，数组，并且`json`模块在序列化过程中会安静的将元组和列表两个都转换成 JSON 数组。大多数情况下，你可以忽略元组和列表的区别，但是在使用`json` 模块时应记得有这么一回使。

## 进一步阅读

> ☞很多关于`pickle`模块的文章提到了`cPickle`。在 Python 2 中, `pickle` 模块有两个实现, 一个由纯 Python 写的而另一个用 C 写的(但仍然可以在 Python 中调用)。在 Python 3 中, 这两个模块已经合并, 所以你总是简单的`import pickle`就可以。你可能会发现这些文章很有用，但是你应该忽略已过时的关于的`cPickle`的信息.

使用`pickle`模块打包:

*   [`pickle` module](http://docs.python.org/3.1/library/pickle.html)
*   [`pickle` and `cPickle` — Python object serialization](http://www.doughellmann.com/PyMOTW/pickle/)
*   [Using `pickle`](http://wiki.python.org/moin/UsingPickle)
*   [Python persistence management](http://www.ibm.com/developerworks/library/l-pypers.html)

使用 JSON 和 `json` 模块:

*   [`json` — JavaScript Object Notation Serializer](http://www.doughellmann.com/PyMOTW/json/)
*   [JSON encoding and ecoding with custom objects in Python](http://blog.quaternio.net/2009/07/16/json-encoding-and-decoding-with-custom-objects-in-python/)

扩展打包:

*   [Pickling class instances](http://docs.python.org/3.1/library/pickle.html#pickling-class-instances)
*   [Persistence of external objects](http://docs.python.org/3.1/library/pickle.html#persistence-of-external-objects)
*   [Handling stateful objects](http://docs.python.org/3.1/library/pickle.html#handling-stateful-objects)