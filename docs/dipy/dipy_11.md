# 第十章 脚本和流

# 第十章 脚本和流

*   10.1\. 抽象输入源
*   10.2\. 标准输入、输出和错误
*   10.3\. 查询缓冲节点
*   10.4\. 查找节点的直接子节点
*   10.5\. 根据节点类型创建不同的处理器
*   10.6\. 处理命令行参数
*   10.7\. 全部放在一起
*   10.8\. 小结

# 10.1\. 抽象输入源

# 10.1\. 抽象输入源

Python 的最强大力量之一是它的动态绑定，而动态绑定最强大的用法之一是*类文件(file-like)对象*。

许多需要输入源的函数可以只接收一个文件名，并以读方式打开文件，读取文件，处理完成后关闭它。其实它们不是这样的，而是接收一个*类文件对象*。

在最简单的例子中，*类文件对象* 是任意一个带有 `read` 方法的对象，这个方法带有一个可选的 `size` 参数，并返回一个字符串。调用时如果没有 `size` 参数，它从输入源中读取所有东西并将所有数据作为单个字符串返回；调用时如果指定了 `size` 参数，它将从输入源中读取 `size` 大小的数据并返回这些数据；再次调用的时候，它从余下的地方开始并返回下一块数据。

这就是从真实文件读取数据的工作方式；区别在于你不用把自己局限于真实的文件。输入源可以是任何东西：磁盘上的文件，甚至是一个硬编码的字符串。只要你将一个类文件对象传递给函数，函数只是调用对象的 `read` 方法，就可以处理任何类型的输入源，而不需要为处理每种类型分别编码。

你可能会纳闷，这和 XML 处理有什么关系。其实 `minidom.parse` 就是一个可以接收类文件对象的函数。

## 例 10.1\. 从文件中解析 XML

```py
>>> from xml.dom import minidom
>>> fsock = open('binary.xml')    
>>> xmldoc = minidom.parse(fsock) 
>>> fsock.close()                 
>>> print xmldoc.toxml()          
<?xml version="1.0" ?>
<grammar>
<ref id="bit">
  <p>0</p>
  <p>1</p>
</ref>
<ref id="byte">
  <p><xref id="bit"/><xref id="bit"/><xref id="bit"/><xref id="bit"/>\
<xref id="bit"/><xref id="bit"/><xref id="bit"/><xref id="bit"/></p>
</ref>
</grammar> 
```

|  |  |
| --- | --- |
| [1] | 首先，你要打开一个磁盘上的文件。这会提供给你一个文件对象。 |
| [2] | 将文件对象传递给 `minidom.parse`，它调用 `fsock` 的 `read` 方法并从磁盘上的文件读取 XML 文档。 |
| [3] | 确保处理完文件后调用 `close` 方法。`minidom.parse`不会替你做这件事。 |
| [4] | 在返回的 XML 文档上调用 `toxml()` 方法，打印出整个文档的内容。 |

哦，所有这些看上去像是在浪费大量的时间。毕竟，你已经看到，`minidom.parse` 可以只接收文件名，并自动执行所有打开文件和关闭无用文件的行为。不错，如果你知道正要解析的是一个本地文件，你可以传递文件名而且 `minidom.parse` 可以足够聪明地做正确的事情 (Do The Right Thing?[10])，这一切都不会有问题。但是请注意，使用类文件，会使分析直接从 Internet 上来的 XML 文档变得多么相似和容易！

## 例 10.2\. 解析来自 URL 的 XML

```py
>>> import urllib
>>> usock = urllib.urlopen('http://slashdot.org/slashdot.rdf') 
>>> xmldoc = minidom.parse(usock)                              
>>> usock.close()                                              
>>> print xmldoc.toxml()                                       
<?xml version="1.0" ?>
<rdf:RDF 
 >
<channel>
<title>Slashdot</title>
<link>http://slashdot.org/</link>
<description>News for nerds, stuff that matters</description>
</channel>
<image>
<title>Slashdot</title>
<url>http://images.slashdot.org/topics/topicslashdot.gif</url>
<link>http://slashdot.org/</link>
</image>
<item>
<title>To HDTV or Not to HDTV?</title>
<link>http://slashdot.org/article.pl?sid=01/12/28/0421241</link>
</item>
[...snip...] 
```

|  |  |
| --- | --- |
| [1] | 正如在前一章中所看到的，`urlopen` 接收一个 web 页面的 URL 作为参数并返回一个类文件对象。最重要的是，这个对象有一个 `read` 方法，它可以返回 web 页面的 HTML 源代码。 |
| [2] | 现在把类文件对象传递给 `minidom.parse`，它顺从地调用对象的 `read` 方法并解析 `read` 方法返回的 XML 数据。这与 XML 数据现在直接来源于 web 页面的事实毫不相干。`minidom.parse` 并不知道 web 页面，它也不关心 web 页面；它只知道类文件对象。 |
| [3] | 到这里已经处理完毕了，确保将 `urlopen` 提供给你的类文件对象关闭。 |
| [4] | 顺便提一句，这个 URL 是真实的，它真的是一个 XML。它是 [Slashdot](http://slashdot.org/) 站点 (一个技术新闻和随笔站点) 上当前新闻提要的 XML 表示。 |

## 例 10.3\. 解析字符串 XML (容易但不灵活的方式)

```py
>>> contents = "<grammar><ref id='bit'><p>0</p><p>1</p></ref></grammar>"
>>> xmldoc = minidom.parseString(contents) 
>>> print xmldoc.toxml()
<?xml version="1.0" ?>
<grammar><ref id="bit"><p>0</p><p>1</p></ref></grammar> 
```

|  |  |
| --- | --- |
| [1] | `minidom` 有一个方法，`parseString`，它接收一个字符串形式的完整 XML 文档作为参数并解析这个参数。如果你已经将整个 XML 文档放入一个字符串，你可以使用它代替 `minidom.parse`。 |

好吧，所以你可以使用 `minidom.parse` 函数来解析本地文件和远端 URL，但对于解析字符串，你使用……另一个函数。这就是说，如果你要从文件、URL 或者字符串接收输入，就需要特别的逻辑来判断参数是否是字符串，然后调用 `parseString`。多不让人满意。

如果有一个方法可以把字符串转换成类文件对象，那么你只要这个对象传递给 `minidom.parse` 就可以了。事实上，有一个模块专门设计用来做这件事：`StringIO`。

## 例 10.4\. `StringIO` 介绍

```py
>>> contents = "<grammar><ref id='bit'><p>0</p><p>1</p></ref></grammar>"
>>> import StringIO
>>> ssock = StringIO.StringIO(contents)   
>>> ssock.read()                          
"<grammar><ref id='bit'><p>0</p><p>1</p></ref></grammar>"
>>> ssock.read()                          
''
>>> ssock.seek(0)                         
>>> ssock.read(15)                        
'<grammar><ref i'
>>> ssock.read(15)
"d='bit'><p>0</p"
>>> ssock.read()
'><p>1</p></ref></grammar>'
>>> ssock.close() 
```

|  |  |
| --- | --- |
| [1] | `StringIO` 模块只包含了一个类，也叫 `StringIO`，它允许你将一个字符串转换为一个类文件对象。 `StringIO` 类在创建实例时接收字符串作为参数。 |
| [2] | 现在你有了一个类文件对象，你可用它做类文件的所有事情。比如 `read` 可以返回原始字符串。 |
| [3] | 再次调用 `read` 返回空字符串。真实文件对象的工作方式也是这样的；一旦你读取了整个文件，如果不显式定位到文件的开始位置，就不可能读取到任何其他数据。`StringIO` 对象以相同的方式进行工作。 |
| [4] | 使用 `StringIO` 对象的 `seek` 方法，你可以显式地定位到字符串的开始位置，就像在文件中定位一样。 |
| [5] | 将一个 `size` 参数传递给 `read` 方法，你还可以以块的形式读取字符串。 |
| [6] | 任何时候，`read` 都将返回字符串的未读部分。所有这些严格地按文件对象的方式工作；这就是术语*类文件对象* 的来历。 |

## 例 10.5\. 解析字符串 XML (类文件对象方式)

```py
>>> contents = "<grammar><ref id='bit'><p>0</p><p>1</p></ref></grammar>"
>>> ssock = StringIO.StringIO(contents)
>>> xmldoc = minidom.parse(ssock) 
>>> ssock.close()
>>> print xmldoc.toxml()
<?xml version="1.0" ?>
<grammar><ref id="bit"><p>0</p><p>1</p></ref></grammar> 
```

|  |  |
| --- | --- |
| [1] | 现在你可以把类文件对象 (实际是一个 `StringIO`) 传递给 `minidom.parse`，它将调用对象的 `read` 方法并高兴地开始解析，绝不会知道它的输入源自一个硬编码的字符串。 |

那么现在你知道了如何使用同一个函数，`minidom.parse`，来解析一个保存在 web 页面上、本地文件中或硬编码字符串中的 XML 文档。对于一个 web 页面，使用 `urlopen` 得到类文件对象；对于本地文件，使用 `open`；对于字符串，使用 `StringIO`。现在让我们进一步并归纳一下*这些* 不同。

## 例 10.6\. `openAnything`

```py
 def openAnything(source):                  
    # try to open with urllib (if source is http, ftp, or file URL)
    import urllib                         
    try:                                  
        return urllib.urlopen(source)      
    except (IOError, OSError):            
        pass                              
    # try to open with native open function (if source is pathname)
    try:                                  
        return open(source)                
    except (IOError, OSError):            
        pass                              
    # treat source as string
    import StringIO                       
    return StringIO.StringIO(str(source)) 
```

|  |  |
| --- | --- |
| [1] | `openAnything` 函数接受单个参数，`source`，并返回类文件对象。`source` 是某种类型的字符串；它可能是一个 URL (例如 `'http://slashdot.org/slashdot.rdf'`)，一个本地文件的完整或者部分路径名 (例如 `'binary.xml'`)，或者是一个包含了待解析 XML 数据的字符串。 |
| [2] | 首先，检查 `source` 是否是一个 URL。这里通过强制方式进行：尝试把它当作一个 URL 打开并静静地忽略打开非 URL 引起的错误。这样做非常好，因为如果 `urllib` 将来支持更多的 URL 类型，不用重新编码就可以支持它们。如果 `urllib` 能够打开 `source`，那么 `return` 可以立刻把你踢出函数，下面的 `try` 语句将不会执行。 |
| [3] | 另一方面，如果 `urllib` 向你呼喊并告诉你 `source` 不是一个有效的 URL，你假设它是一个磁盘文件的路径并尝试打开它。再一次，你不用做任何特别的事来检查 `source` 是否是一个有效的文件名 (在不同的平台上，判断文件名有效性的规则变化很大，因此不管怎样做都可能会判断错)。反而，只要盲目地打开文件并静静地捕获任何错误就可以了。 |
| [4] | 到这里，你需要假设 `source` 是一个其中有硬编码数据的字符串 (因为没有别的可以判断的了)，所以你可以使用 `StringIO` 从中创建一个类文件对象并将它返回。(实际上，由于使用了 `str` 函数，所以 `source` 没有必要一定是字符串；它可以是任何对象，你可以使用它的字符串表示形式，只要定义了它的 `__str__` 专用方法。) |

现在你可以使用这个 `openAnything` 函数联合 `minidom.parse` 构造一个函数，接收一个指向 XML 文档的 `source`，而且无需知道这个 `source` 的含义 (可以是一个 URL 或是一个本地文件名，或是一个硬编码 XML 文档的字符串形式)，然后解析它。

## 例 10.7\. 在 `kgp.py` 中使用 `openAnything`

```py
 class KantGenerator:
    def _load(self, source):
        sock = toolbox.openAnything(source)
        xmldoc = minidom.parse(sock).documentElement
        sock.close()
        return xmldoc 
```

## Footnotes

[10] 这是一部著名的电影。――译注

# 10.2\. 标准输入、输出和错误

# 10.2\. 标准输入、输出和错误

UNIX 用户已经对标准输入、标准输出和标准错误的概念非常熟悉了。这一节是为其他不熟悉的人准备的。

标准输入和标准错误 (通常缩写为 `stdout` 和 `stderr`) 是内建在每一个 UNIX 系统中的管道。当你 `print` 某些东西时，结果前往 `stdout` 管道；当你的程序崩溃并打印出调试信息 (例如 Python 中的 traceback (错误跟踪)) 的时候，信息前往 `stderr` 管道。通常这两个管道只与你正在工作的终端窗口相联，所以当一个程序打印时，你可以看到输出，而当一个程序崩溃时，你可以看到调试信息。(如果你正在一个基于窗口的 Python IDE 上工作，`stdout` 和 `stderr` 缺省为你的“交互窗口”。)

## 例 10.8\. `stdout` 和 `stderr` 介绍

```py
>>> for i in range(3):
... print 'Dive in'             
Dive in
Dive in
Dive in
>>> import sys
>>> for i in range(3):
... sys.stdout.write('Dive in') 
Dive inDive inDive in
>>> for i in range(3):
... sys.stderr.write('Dive in') 
Dive inDive inDive in 
```

|  |  |
| --- | --- |
| [1] | 正如在例 6.9 “简单计数”中看到的，你可以使用 Python 内置的 `range` 函数来构造简单的计数循环，即重复某物一定的次数。 |
| [2] | `stdout` 是一个类文件对象；调用它的 `write` 函数可以打印出你给定的任何字符串。实际上，这就是 `print` 函数真正做的事情；它在你打印的字符串后面加上一个硬回车，然后调用 `sys.stdout.write` 函数。 |
| [3] | 在最简单的例子中，`stdout` 和 `stderr` 把它们的输出发送到相同的地方：Python IDE (如果你在一个 IDE 中的话)，或者终端 (如果你从命令行运行 Python 的话)。和 `stdout` 一样，`stderr` 并不为你添加硬回车；如果需要，要自己加上。 |

`stdout` 和 `stderr` 都是类文件对象，就像在第 10.1 节 “抽象输入源”中讨论的一样，但是它们都是只写的。它们都没有 `read` 方法，只有 `write` 方法。然而，它们仍然是类文件对象，因此你可以将其它任何 (类) 文件对象赋值给它们来重定向其输出。

## 例 10.9\. 重定向输出

```py
[you@localhost kgp]$ python stdout.py
Dive in
[you@localhost kgp]$ cat out.log
This message will be logged instead of displayed 
```

(在 Windows 上，你要使用 `type` 来代替 `cat` 显示文件的内容。)

如果您还没有下载本书附带的样例程序, 可以 [下载本程序和其他样例程序](http://www.woodpecker.org.cn/diveintopython/download/diveintopython-exampleszh-cn-5.4b.zip "Download example scripts")。

```py
#stdout.py
import sys
print 'Dive in'                                          
saveout = sys.stdout                                     
fsock = open('out.log', 'w')                             
sys.stdout = fsock                                        print 'This message will be logged instead of displayed' 
sys.stdout = saveout                                     
fsock.close() 
```

|  |  |
| --- | --- |
| [1] | 打印输出到 IDE “交互窗口” (或终端，如果从命令行运行脚本的话)。 |
| [2] | 始终在重定向前保存 `stdout`，这样的话之后你还可以将其设回正常。 |
| [3] | 打开一个新文件用于写入。如果文件不存在，将会被创建。如果文件存在，将被覆盖。 |
| [4] | 所有后续的输出都会被重定向到刚才打开的新文件上。 |
| [5] | 这样只会将输出结果“打印”到日志文件中；在 IDE 窗口中或在屏幕上不会看到输出结果。 |
| [6] | 在我们将 `stdout` 搞乱之前，让我们把它设回原来的方式。 |
| [7] | 关闭日志文件。 |

重定向 `stderr` 以完全相同的方式进行，只要把 `sys.stdout` 改为 `sys.stderr`。

## 例 10.10\. 重定向错误信息

```py
[you@localhost kgp]$ python stderr.py
[you@localhost kgp]$ cat error.log
Traceback (most recent line last):
  File "stderr.py", line 5, in ?
    raise Exception, 'this error will be logged'
Exception: this error will be logged 
```

如果您还没有下载本书附带的样例程序, 可以 [下载本程序和其他样例程序](http://www.woodpecker.org.cn/diveintopython/download/diveintopython-exampleszh-cn-5.4b.zip "Download example scripts")。

```py
#stderr.py
import sys
fsock = open('error.log', 'w')               
sys.stderr = fsock                            raise Exception, 'this error will be logged' 
```

|  |  |
| --- | --- |
| [1] | 打开你要存储调试信息的日志文件。 |
| [2] | 将新打开的日志文件的文件对象赋值给 `stderr` 以重定向标准错误。 |
| [3] | 引发一个异常。从屏幕输出上可以注意到这个行为*没有* 在屏幕上打印出任何东西。所有正常的跟踪信息已经写进 `error.log`。 |
| [4] | 还要注意你既没有显式关闭日志文件，也没有将 `stderr` 设回最初的值。这样挺好，因为一旦程序崩溃 (由于引发的异常)，Python 将替我们清理并关闭文件，因此永远不恢复 `stderr` 不会造成什么影响。然而对于 `stdout`，恢复初始值相对更为重要――你可能会在后面再次操作标准输出。 |

向标准错误写入错误信息是很常见的，所以有一种较快的语法可以立刻导出信息。

## 例 10.11\. 打印到 `stderr`

```py
>>> print 'entering function'
entering function
>>> import sys
>>> print >> sys.stderr, 'entering function' 
entering function 
```

|  |  |
| --- | --- |
| [1] | `print` 语句的快捷语法可以用于写入任何打开的文件 (或者是类文件对象)。在这里，你可以将单个 `print` 语句重定向到 `stderr` 而且不用影响后面的 `print` 语句。 |

另一方面，标准输入是一个只读文件对象，它表示从前一个程序到这个程序的数据流。这个对于老的 Mac OS 用户和 Windows 用户可能不太容易理解，除非你受到过 MS-DOS 命令行的影响。在 MS-DOS 命令行中，你可以使用一行指令构造一个命令的链，使得一个程序的输出就可以成为下一个程序的输入。第一个程序只是简单地输出到标准输出上 (程序本身没有做任何特别的重定向，只是执行了普通的 `print` 语句等)，然后，下一个程序从标准输入中读取，操作系统就把一个程序的输出连接到一个程序的输入。

## 例 10.12\. 链接命令

```py
[you@localhost kgp]$ python kgp.py -g binary.xml         
01100111
[you@localhost kgp]$ cat binary.xml                      
<?xml version="1.0"?>
<!DOCTYPE grammar PUBLIC "-//diveintopython.org//DTD Kant Generator Pro v1.0//EN" "kgp.dtd">
<grammar>
<ref id="bit">
  <p>0</p>
  <p>1</p>
</ref>
<ref id="byte">
  <p><xref id="bit"/><xref id="bit"/><xref id="bit"/><xref id="bit"/>\
<xref id="bit"/><xref id="bit"/><xref id="bit"/><xref id="bit"/></p>
</ref>
</grammar>
[you@localhost kgp]$ cat binary.xml | python kgp.py -g -  
10110001 
```

|  |  |
| --- | --- |
| [1] | 正如你在第 9.1 节 “概览”中看到的，该命令将只打印一个随机的八位字符串，其中只有 `0` 或者 `1`。 |
| [2] | 该处只是简单地打印出整个 `binary.xml` 文档的内容。(Windows 用户应该用 `type` 代替 `cat`。) |
| [3] | 该处打印 `binary.xml` 的内容，但是“`&#124;`”字符，称为“管道”符，说明内容不会打印到屏幕上；它们会成为下一个命令的标准输入，在这个例子中是你调用的 Python 脚本。 |
| [4] | 为了不用指定一个文件 (例如 `binary.xml`)，你需要指定“`-`”，它会使得你的脚本从标准输入载入脚本，而不是从磁盘上的特定文件。 (下一个例子更多地说明了这是如何实现的)。所以效果和第一种语法是一样的，在那里你要直接指定语法文件，但是想想这里的扩展性。让我们把 `cat binary.xml` 换成别的什么东西――例如运行一个脚本动态生成语法――然后通过管道将它导入你的脚本。它可以来源于任何地方：数据库，或者是生成语法的元脚本，或者其他。你根本不需要修改你的 `kgp.py` 脚本就可以并入这个功能。你要做的仅仅是从标准输入取得一个语法文件，然后你就可以将其他的逻辑分离出来，放到另一程序中去了。 |

那么脚本是如何“知道”在语法文件是“`-`”时从标准输入读取? 其实不神奇；它只是代码。

## 例 10.13\. 在 `kgp.py` 中从标准输入读取

```py
 def openAnything(source):
    if source == "-":    
        import sys
        return sys.stdin
    # try to open with urllib (if source is http, ftp, or file URL)
    import urllib
    try:
[... snip ...] 
```

|  |  |
| --- | --- |
| [1] | 这是 `toolbox.py` 中的 `openAnything` 函数，以前在第 10.1 节 “抽象输入源”中你已经检视过了。所有你要做的就是在函数的开始加入 3 行代码来检测源是否是“`-`”；如果是，返回 `sys.stdin`。就这么简单！记住，`stdin` 是一个拥有 `read` 方法的类文件对象，所以其它的代码 (在 `kgp.py` 中，在那里你调用了 `openAnything`) 一点都不需要改动。 |

# 10.3\. 查询缓冲节点

# 10.3\. 查询缓冲节点

`kgp.py` 使用了多种技巧，在你进行 XML 处理时，它们或许能派上用场。第一个就是，利用输入文档的结构稳定特征来构建节点缓冲。

一个语法文件定义了一系列的 `ref` 元素。每个 `ref` 包含了一个或多个 `p` 元素，`p` 元素则可以包含很多不同的东西，包括 `xref`。对于每个 `xref`，你都能找到相对应的 `ref` 元素 (它们具有相同的 `id` 属性)，然后选择 `ref` 元素的子元素之一进行解析。(在下一部分中你将看到是如何进行这种随机选择的。)

语法的构建方式如下：先为最小的片段定义 `ref` 元素，然后使用 `xref` 定义“包含”第一个 `ref` 元素的 `ref` 元素，等等。然后，解析“最大的”引用并跟着 `xref` 跳来跳去，最后输出真实的文本。输出的文本依赖于你每次填充 `xref` 时所做的 (随机) 决策，所以每次的输出都是不同的。

这种方式非常灵活，但是有一个不好的地方：性能。当你找到一个 `xref` 并需要找到相应的 `ref` 元素时，会遇到一个问题。`xref` 有 `id` 属性，而你要找拥有相同 `id` 属性的 `ref` 元素，但是没有简单的方式做到这件事。较慢的方式是每次获取所有 `ref` 元素的完整列表，然后手动遍历并检视每一个 `id` 属性。较快的方式是只做一次，然后以字典形式构建一个缓冲。

## 例 10.14\. `loadGrammar`

```py
 def loadGrammar(self, grammar):                         
        self.grammar = self._load(grammar)                  
        self.refs = {}                                       
        for ref in self.grammar.getElementsByTagName("ref"): 
            self.refs[ref.attributes["id"].value] = ref 
```

|  |  |
| --- | --- |
| [1] | 从创建一个空字典 `self.refs` 开始。 |
| [2] | 正如你在第 9.5 节 “搜索元素”中看到的，`getElementsByTagName` 返回所有特定名称元素的一个列表。你可以很容易地得到所有 `ref` 元素的一个列表，然后遍历这个列表。 |
| [3] | 正如你在第 9.6 节 “访问元素属性”中看到的，使用标准的字典语法，你可以通过名称来访问个别元素。所以，`self.refs` 字典的键将是每个 `ref` 元素的 `id` 属性值。 |
| [4] | `self.refs` 字典的值将是 `ref` 元素本身。如你在第 9.3 节 “XML 解析”中看到的，已解析 XML 文档中的每个元素、节点、注释和文本片段都是一个对象。 |

只要构建了这个缓冲，无论何时你遇到一个 `xref` 并且需要找到具有相同 `id` 属性的 `ref` 元素，都只需在 `self.refs` 中查找它。

## 例 10.15\. 使用 `ref` 元素缓冲

```py
 def do_xref(self, node):
        id = node.attributes["id"].value
        self.parse(self.randomChildElement(self.refs[id])) 
```

你将在下一部分探究 `randomChildElement` 函数。

# 10.4\. 查找节点的直接子节点

# 10.4\. 查找节点的直接子节点

解析 XML 文档时，另一个有用的己技巧是查找某个特定元素的所有直接子元素。例如，在语法文件中，一个 `ref` 元素可以有数个 `p` 元素，其中每一个都可以包含很多东西，包括其他的 `p` 元素。你只要查找作为 `ref` 孩子的 `p` 元素，不用查找其他 `p` 元素的孩子 `p` 元素。

你可能认为你只要简单地使用 `getElementsByTagName` 来实现这点就可以了，但是你不可以这么做。`getElementsByTagName` 递归搜索并返回所有找到的元素的单个列表。由于 `p` 元素可以包含其他的 `p` 元素，你不能使用 `getElementsByTagName`，因为它会返回你不要的嵌套 `p` 元素。为了只找到直接子元素，你要自己进行处理。

## 例 10.16\. 查找直接子元素

```py
 def randomChildElement(self, node):
        choices = [e for e in node.childNodes
                   if e.nodeType == e.ELEMENT_NODE]   
        chosen = random.choice(choices)             
        return chosen 
```

|  |  |
| --- | --- |
| [1] | 正如你在例 9.9 “获取子节点”中看到的，`childNodes` 属性返回元素所有子节点的一个列表。 |
| [2] | 然而，正如你在例 9.11 “子节点可以是文本”中看到的，`childNodes` 返回的列表包含了所有不同类型的节点，包括文本节点。这并不是你在这里要查找的。你只要元素形式的孩子。 |
| [3] | 每个节点都有一个 `nodeType` 属性，它可以是`ELEMENT_NODE`, `TEXT_NODE`, `COMMENT_NODE`，或者其它值。可能值的完整列表在 `xml.dom` 包的 `__init__.py` 文件中。(关于包的介绍，参见第 9.2 节 “包”。) 但你只是对元素节点有兴趣，所以你可以过滤出一个列表，其中只包含 `nodeType` 是`ELEMENT_NODE`的节点。 |
| [4] | 只要拥有了一个真实元素的列表，选择任意一个都很容易。Python 有一个叫 `random` 的模块，它包含了好几个有用的函数。`random.choice` 函数接收一个任意数量条目的列表并随机返回其中的一个条目。比如，如果 `ref` 元素包含了多个 `p` 元素，那么 `choices` 将会是 `p` 元素的一个列表，而 `chosen` 将被赋予其中的某一个值，而这个值是随机选择的。 |

# 10.5\. 根据节点类型创建不同的处理器

# 10.5\. 根据节点类型创建不同的处理器

第三个有用的 XML 处理技巧是将你的代码基于节点类型和元素名称分散到逻辑函数中。解析后的 XML 文档是由各种类型的节点组成的，每一个都是通过 Python 对象表示的。文档本身的根层次通过一个 `Document` 对象表示。`Document` 还包含了一个或多个 `Element` 对象 (表示 XML 标记)，其中的每一个可以包含其它的 `Element` 对象、`Text` 对象 (表示文本)，或者 `Comment` 对象 (表示内嵌注释)。使用 Python 编写分离各个节点类型逻辑的分发器非常容易。

## 例 10.17\. 已解析 XML 对象的类名

```py
>>> from xml.dom import minidom
>>> xmldoc = minidom.parse('kant.xml') 
>>> xmldoc
<xml.dom.minidom.Document instance at 0x01359DE8>
>>> xmldoc.__class__                   
<class xml.dom.minidom.Document at 0x01105D40>
>>> xmldoc.__class__.__name__          
'Document' 
```

|  |  |
| --- | --- |
| [1] | 暂时假设 `kant.xml` 在当前目录中。 |
| [2] | 正如你在第 9.2 节 “包”中看到的，解析 XML 文档返回的对象是一个 `Document` 对象，就像在 `xml.dom` 包的 `minidom.py` 中定义的一样。又如你在第 5.4 节 “类的实例化”中看到的，`__class__` 是每个 Python 对象的一个内置属性。 |
| [3] | 此外，`__name__` 是每个 Python 类的内置属性，是一个字符串。这个字符串并不神秘；它和你在定义类时输入的类名相同。(参见第 5.3 节 “类的定义”。) |

好，现在你能够得到任何给定 XML 节点的类名了 (因为每个 XML 节点都是以一个 Python 对象表示的)。你怎样才能利用这点来分离解析每个节点类型的逻辑呢？答案就是 `getattr`，你第一次见它是在第 4.4 节 “通过 getattr 获取对象引用”中。

## 例 10.18\. `parse`，通用 XML 节点分发器

```py
 def parse(self, node):          
        parseMethod = getattr(self, "parse_%s" % node.__class__.__name__)  
        parseMethod(node) 
```

|  |  |
| --- | --- |
| [1] | 首先，注意你正在基于传入节点 (`node` 参数) 的类名构造一个较大的字符串。所以如果你传入一个 `Document` 节点，你就构造了字符串 `'parse_Document'`，其它类同于此。 |
| [2] | 现在你可以把这个字符串当作一个函数名称，然后通过 `getattr` 得到函数自身的引用。 |
| [3] | 最后，你可以调用函数并将节点自身作为参数传入。下一个例子将展示每个函数的定义。 |

## 例 10.19\. `parse` 分发器调用的函数

```py
 def parse_Document(self, node): 
        self.parse(node.documentElement)
    def parse_Text(self, node):    
        text = node.data
        if self.capitalizeNextWord:
            self.pieces.append(text[0].upper())
            self.pieces.append(text[1:])
            self.capitalizeNextWord = 0
        else:
            self.pieces.append(text)
    def parse_Comment(self, node): 
        pass
    def parse_Element(self, node): 
        handlerMethod = getattr(self, "do_%s" % node.tagName)
        handlerMethod(node) 
```

|  |  |
| --- | --- |
| [1] | `parse_Document` 只会被调用一次，因为在一个 XML 文档中只有一个 `Document` 节点，并且在已解析 XML 的表示中只有一个 `Document` 对象。在此它只是起到中转作用，转而解析语法文件的根元素。 |
| [2] | `parse_Text` 在节点表示文本时被调用。这个函数本身做某种特殊处理，自动将句子的第一个单词进行大写处理，而不是简单地将表示的文本追加到一个列表中。 |
| [3] | `parse_Comment` 只有一个 `pass`，因为你并不关心语法文件中嵌入的注释。但是注意，你还是要定义这个函数并显式地让它不做任何事情。如果这个函数不存在，通用 `parse` 函数在遇到一个注释的时候会执行失败，因为它试图找到并不存在的 `parse_Comment` 函数。为每个节点类型定义独立的函数――甚至你不要使用的――将会使通用 `parse` 函数保持简单和沉默。 |
| [4] | `parse_Element` 方法其实本身就是一个分发器，一个基于元素的标记名称的分发器。这个基本概念是相同的：使用元素的区别 (它们的标记名称) 然后针对每一个分发到一个独立的函数。你构建了一个类似于 `'do_xref'` 的字符串 (对 `&lt;xref&gt;` 标记而言)，找到这个名称的函数，并调用它。对其它的标记名称 (像`&lt;p&gt;` 和 `&lt;choice&gt;`) 在解析语法文件的时候都可以找到类似的函数。 |

在这个例子中，分发函数 `parse` 和 `parse_Element` 只是找到相同类中的其它方法。如果你进行的处理过程很复杂 (或者你有很多不同的标记名称)，你可以将代码分散到独立的模块中，然后使用动态导入的方式导入每个模块并调用你需要的任何函数。动态导入将在第十六章 *函数编程*中介绍。

# 10.6\. 处理命令行参数

# 10.6\. 处理命令行参数

Python 完全支持创建在命令行运行的程序，也支持通过命令行参数和短长样式来指定各种选项。这些并非是 XML 特定的，但是这样的脚本可以充分使用命令行处理，看来是时候提一下它了。

如果不理解命令行参数如何暴露给你的 Python 程序，讨论命令行处理是很困难的，所以让我们先写个简单点的程序来看一下。

## 例 10.20\. `sys.argv` 介绍

如果您还没有下载本书附带的样例程序, 可以 [下载本程序和其他样例程序](http://www.woodpecker.org.cn/diveintopython/download/diveintopython-exampleszh-cn-5.4b.zip "Download example scripts")。

```py
#argecho.py
import sys
for arg in sys.argv: 
    print arg 
```

|  |  |
| --- | --- |
| [1] | 每个传递给程序的命令行参数都在 `sys.argv` 中，而它仅仅是一个列表。这里我们在独立行中打印出每个参数。 |

## 例 10.21\. `sys.argv` 的内容

```py
[you@localhost py]$ python argecho.py             
argecho.py
[you@localhost py]$ python argecho.py abc def     
argecho.py
abc
def
[you@localhost py]$ python argecho.py --help      
argecho.py
--help
[you@localhost py]$ python argecho.py -m kant.xml 
argecho.py
-m
kant.xml 
```

|  |  |
| --- | --- |
| [1] | 关于 `sys.argv` 需要了解的第一件事情就是：它包含了你正在调用的脚本的名称。你后面会实际使用这个知识，在第十六章 *函数编程*中。现在不用担心。 |
| [2] | 命令行参数通过空格进行分隔。在 `sys.argv` 列表中，每个参数都是一个独立的元素。 |
| [3] | 命令行标志，像 `--help`，在 `sys.argv` 列表中还保存了它们自己的元素。 |
| [4] | 为了让事情更有趣，有些命令行标志本身就接收参数。比如，这里有一个标记 (`-m`) 接收一个参数 (`kant.xml`)。标记自身和标记参数只是 `sys.argv` 列表中的一串元素。并没有试图将元素与其它元素进行关联；所有你得到的是一个列表。 |

所以正如你所看到的，你确实拥有了命令行传入的所有信息，但是接下来要实际使用它似乎不那么容易。对于只是接收单个参数或者没有标记的简单程序，你可以简单地使用 `sys.argv[1]` 来访问参数。这没有什么羞耻的；我一直都是这样做的。对更复杂的程序，你需要 `getopt` 模块。

## 例 10.22\. `getopt` 介绍

```py
 def main(argv):                         
    grammar = "kant.xml"                 
    try:                                
        opts, args = getopt.getopt(argv, "hg:d", ["help", "grammar="]) 
    except getopt.GetoptError:           
        usage()                          
        sys.exit(2)                     
...
if __name__ == "__main__":
    main(sys.argv[1:]) 
```

|  |  |
| --- | --- |
| [1] | 首先，看一下例子的最后并注意你正在调用 `main` 函数，参数是 `sys.argv[1:]`。记住，`sys.argv[0]` 是你正在运行脚本的名称；在处理命令行时，你不用关心它，所以你可以砍掉它并传入列表的剩余部分。 |
| [2] | 这里就是所有有趣处理发生的地方。`getopt` 模块的 `getopt` 函数接受三个参数：参数列表 (你从 `sys.argv[1:]` 得到的)、一个包含了程序所有可能接收到的单字符命令行标志，和一个等价于单字符的长命令行标志的列表。第一次看的时候，这有点混乱，下面有更多的细节解释。 |
| [3] | 在解析这些命令行标志时，如果有任何事情错了，`getopt` 会抛出异常，你可以捕获它。你可以告诉 `getopt` 你明白的所有标志，那么这也意味着终端用户可以传入一些你不理解的命令行标志。 |
| [4] | 和 UNIX 世界中的标准实践一样，如果脚本被传入了不能理解的标志，你要打印出正确用法的一个概要并友好地退出。注意，在这里我没有写出 `usage` 函数。你还是要在某个地方写一个，使它打印出合适的概要；它不是自动的。 |

那么你传给 `getopt` 函数的参数是什么呢？好的，第一个只不过是一个命令行标志和参数的原始列表 (不包括第一个元素――脚本名称，你在调用 `main` 函数之前就已经将它砍掉了)。第二个是脚本接收的短命令行标志的一个列表。

## `"hg:d"`

`-h`

打印用法概要

`-g ...`

使用给定的语法文件或 URL

`-d`

在解析时显示调试信息

第一个标志和第三个标志是简单的独立标志；你选择是否指定它们，它们做某些事情 (打印帮助) 或者改变状态 (打开调试)。但是，第二个标志 (`-g`) *必须* 跟随一个参数――进行读取的语法文件的名称。实际上，它可以是一个文件名或者一个 web 地址，这时还不知道 (后面会确定)，但是你要知道必须要*有些东西*。所以，你可以通过在 `getopt` 函数的第二个参数的 `g` 后面放一个冒号，来向 `getopt` 说明这一点。

更复杂的是，这个脚本既接收短标志 (像 `-h`)，也接受长标志 (像 `--help`)，并且你要它们做相同的事。这就是 `getopt` 第三个参数存在的原因：它是指定长标志的一个列表，其中的长标志是和第二个参数中指定的短标志相对应的。

## `["help", "grammar="]`

`--help`

打印用法概要

`--grammar ...`

使用给定的语法文件或 URL

这里有三点要注意：

1.  所有命令行中的长标志以两个短划线开始，但是在调用 `getopt` 时，你不用包含这两个短划线。它们是能够被理解的。
2.  `--grammar` 标志的后面必须跟着另一个参数，就像 `-g` 标志一样。通过等于号标识出来：`"grammar="`。
3.  长标志列表比短标志列表更短一些，因为 `-d` 标志没有相应的长标志。这很好；只有 `-d` 才会打开调试。但是短标志和长标志的顺序必须是相同的，你应该先指定有长标志的短标志，然后才是剩下的短标志。

被搞昏没？让我们看一下真实的代码，看看它在上下文中是否起作用。

## 例 10.23\. 在 `kgp.py` 中处理命令行参数

```py
 def main(argv):                          
    grammar = "kant.xml"                
    try:                                
        opts, args = getopt.getopt(argv, "hg:d", ["help", "grammar="])
    except getopt.GetoptError:          
        usage()                         
        sys.exit(2)                     
    for opt, arg in opts:                
        if opt in ("-h", "--help"):      
            usage()                     
            sys.exit()                  
        elif opt == '-d':                
            global _debug               
            _debug = 1                  
        elif opt in ("-g", "--grammar"): 
            grammar = arg               
    source = "".join(args)               
    k = KantGenerator(grammar, source)
    print k.output() 
```

|  |  |
| --- | --- |
| [1] | `grammar` 变量会跟踪你正在使用的语法文件。如果你没有在命令行指定它 (使用 `-g` 或者 `--grammar` 标志定义它)，在这里你将初始化它。 |
| [2] | 你从 `getopt` 取回的 `opts` 变量是一个由元组 (`flag` 和 `argument`) 组成的列表。如果标志没有带任何参数，那么 `arg` 只是 `None`。这使得遍历标志更容易了。 |
| [3] | `getopt` 验证命令行标志是否可接受，但是它不会在短标志和长标志之间做任何转换。如果你指定 `-h` 标志，`opt` 将会包含 `"-h"`；如果你指定 `--help` 标志，`opt` 将会包含`"--help"` 标志。所以你需要检查它们两个。 |
| [4] | 别忘了，`-d` 标志没有相应的长标志，所以你只需要检查短形式。如果你找到了它，你就可以设置一个全局变量来指示后面要打印出调试信息。(我习惯在脚本的开发过程中使用它。什么，你以为所有这些程序都是一次成功的？) |
| [5] | 如果你找到了一个语法文件，跟在 `-g` 或者 `--grammar` 标志后面，那你就要把跟在后面的参数 (`arg`) 保存到变量`grammar` 中，覆盖掉在 `main` 函数你初始化的默认值。 |
| [6] | 就是这样。你已经遍历并处理了所有的命令行标志。这意味着所有剩下的东西都必须是命令行参数。它们由 `getopt` 函数的 `args` 变量返回。在这个例子中，你把它们当作了解析器源材料。如果没有指定命令行参数，`args` 将是一个空列表，而 `source` 将是空字符串。 |

# 10.7\. 全部放在一起

# 10.7\. 全部放在一起

你已经了解很多基础的东西。让我们回来看看所有片段是如何整合到一起的。

作为开始，这里是一个接收命令行参数的脚本，它使用 `getopt` 模块。

```py
 def main(argv):                         
...
    try:                                
        opts, args = getopt.getopt(argv, "hg:d", ["help", "grammar="])
    except getopt.GetoptError:          
...
    for opt, arg in opts:               
... 
```

创建 `KantGenerator` 类的一个实例，然后将语法文件和源文件传给它，可能在命令行没有指定。

```py
 k = KantGenerator(grammar, source) 
```

`KantGenerator` 实例自动加载语法，它是一个 XML 文件。你使用自定义的 `openAnything` 函数打开这个文件 (可能保存在一个本地文件中或者一个远程服务器上)，然后使用内置的 `minidom` 解析函数将 XML 解析为一棵 Python 对象树。

```py
 def _load(self, source):
        sock = toolbox.openAnything(source)
        xmldoc = minidom.parse(sock).documentElement
        sock.close() 
```

哦，根据这种方式，你将使用到 XML 文档结构的知识建立一个引用的小缓冲，这些引用都只是 XML 文档中的元素。

```py
 def loadGrammar(self, grammar):                         
        for ref in self.grammar.getElementsByTagName("ref"):
            self.refs[ref.attributes["id"].value] = ref 
```

如果你在命令行中指定了某些源材料，你可以使用它；否则你将打开语法文件查找“顶层”引用 (没有被其它的东西引用) 并把它作为开始点。

```py
 def getDefaultSource(self):
        xrefs = {}
        for xref in self.grammar.getElementsByTagName("xref"):
            xrefs[xref.attributes["id"].value] = 1
        xrefs = xrefs.keys()
        standaloneXrefs = [e for e in self.refs.keys() if e not in xrefs]
        return '<xref id="%s"/>' % random.choice(standaloneXrefs) 
```

现在你打开了了源材料。它是一个 XML，你每次解析一个节点。为了让代码分离并具备更高的可维护性，你可以使用针对每个节点类型的独立处理方法。

```py
 def parse_Element(self, node): 
        handlerMethod = getattr(self, "do_%s" % node.tagName)
        handlerMethod(node) 
```

你在语法里面跳来跳去，解析每一个 `p` 元素的所有孩子，

```py
 def do_p(self, node):
...
        if doit:
            for child in node.childNodes: self.parse(child) 
```

用任意一个孩子替换 `choice` 元素，

```py
 def do_choice(self, node):
        self.parse(self.randomChildElement(node)) 
```

并用对应 `ref` 元素的任意孩子替换 `xref`，前面你已经进行了缓冲。

```py
 def do_xref(self, node):
        id = node.attributes["id"].value
        self.parse(self.randomChildElement(self.refs[id])) 
```

就这样一直解析，最后得到普通文本。

```py
 def parse_Text(self, node):    
        text = node.data
...
            self.pieces.append(text) 
```

把结果打印出来。

```py
 def main(argv):                         
...
    k = KantGenerator(grammar, source)
    print k.output() 
```

# 10.8\. 小结

# 10.8\. 小结

Python 带有解析和操作 XML 文档非常强大的库。`minidom` 接收一个 XML 文件并将其解析为 Python 对象，并提供了对任意元素的随机访问。进一步，本章展示了如何利用 Python 创建一个“真实”独立的命令行脚本，连同命令行标志、命令行参数、错误处理，甚至从前一个程序的管道接收输入的能力。

在继续下一章前，你应该无困难地完成所有这些事情：

*   通过标准输入输出链接程序
*   使用 `getattr` 定义动态分发器
*   通过 `getopt` 使用命令行标志并进行验证