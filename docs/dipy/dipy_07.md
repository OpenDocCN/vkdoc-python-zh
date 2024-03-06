# 第六章 异常和文件处理

# 第六章 异常和文件处理

*   6.1\. 异常处理
    *   6.1.1\. 为其他用途使用异常
*   6.2\. 与文件对象共事
    *   6.2.1\. 读取文件
    *   6.2.2\. 关闭文件
    *   6.2.3\. 处理 I/O 错误
    *   6.2.4\. 写入文件
*   6.3\. for 循环
*   6.4\. 使用 sys.modules
*   6.5\. 与目录共事
*   6.6\. 全部放在一起
*   6.7\. 小结

在本章中，将研究异常、文件对象、`for` 循环、`os` 和 `sys` 模块等内容。如果你已经在其它编程语言中使用过异常，你可以简单看看第一部分来了解 Python 的语法。但是本章其它的内容仍需仔细研读。

# 6.1\. 异常处理

# 6.1\. 异常处理

*   6.1.1\. 为其他用途使用异常

与许多面向对象语言一样，Python 具有异常处理，通过使用 `try...except` 块来实现。

> 注意
> Python 使用 `try...except` 来处理异常，使用 `raise` 来引发异常。Java 和 C++ 使用 `try...catch` 来处理异常，使用 `throw` 来引发异常。

异常在 Python 中无处不在；实际上在标准 Python 库中的每个模块都使用了它们，并且 Python 自已会在许多不同的情况下引发它们。在整本书中你已经再三看到它们了。

*   使用不存在的字典关键字将引发 `KeyError` 异常。
*   搜索列表中不存在的值将引发 `ValueError` 异常。
*   调用不存在的方法将引发 `AttributeError` 异常。
*   引用不存在的变量将引发 `NameError` 异常。
*   未强制转换就混用数据类型将引发 `TypeError` 异常。

在这些情况下，我们都在简单地使用 Python IDE：一个错误发生了，异常被打印出来 (取决于你的 IDE，可能会有意地以一种刺眼的红色形式表示)，这便是。这叫做*未处理* 异常；当异常被引发时，没有代码来明确地关注和处理它，所以异常被传给置在 Python 中的缺省的处理，它会输出一些调试信息并且终止运行。在 IDE 中，这不是什么大事，但是如果发生在你真正的 Python 程序运行的时候，整个程序将会终止。

然而，一个异常不一定会引起程序的完全崩溃。当异常引发时，可以被*处理* 掉。有时候一个异常实际是因为代码中的 bug (比如使用一个不存在的变量)，但是许多时候，一个异常是可以预见的。如果你打开一个文件，它可能不存在。如果你连接一个数据库，它可能不可连接或没有访问所需的正确的安全证书。如果知道一行代码可能会引发异常，你应该使用一个 `try...except` 块来处理异常。

## 例 6.1\. 打开一个不存在的文件

```py
>>> fsock = open("/notthere", "r")      
Traceback (innermost last):
  File "<interactive input>", line 1, in ?
IOError: [Errno 2] No such file or directory: '/notthere'
>>> try:
... fsock = open("/notthere")       
... except IOError:                     
... print "The file does not exist, exiting gracefully"
... print "This line will always print" 
The file does not exist, exiting gracefully
This line will always print 
```

|  |  |
| --- | --- |
| [1] | 使用内置 `open` 函数，我们可以试着打开一个文件来读取 (在下一节有关于 `open` 的更多内容)。但是那个文件不存在，所以这样就引发 `IOError` 异常。因为我们没有提供任何显式的对 `IOError` 异常的检查，Python 仅仅打印出某个关于发生了什么的调试信息，然后终止。 |
| [2] | 我们试图打开同样不存在的文件，但是这次我们在一个 `try...except` 内来执行它。 |
| [3] | 当 `open` 方法引发 `IOError` 异常时，我们已经准备好处理它了。`except IOError:` 行捕捉异常，接着执行我们自已的代码块，这个代码块在本例中只是打印出更令人愉快的错误信息。 |
| [4] | 一旦异常被处理了，处理通常在 `try...except` 块之后的第一行继续进行。注意这一行将总是打印出来，无论异常是否发生。如果在你的根目录下确实有一个叫 `notthere` 的文件，对 `open` 的调用将成功，`except` 子句将忽略，并且最后一行仍将执行。 |

异常可能看上去不友好 (毕竟，如果你不捕捉异常，整个程序将崩溃)，但是考虑一下别的方法。你该不会希望获得一个指向不存在的文件的对象吧？不管怎么样你都得检查它的有效性，而且如果你忘记了，你的程序将会在下面某个地方给出奇怪的错误，这样你将不得不追溯到源程序。我确信你做过这种事；这可并不有趣。使用异常，一发生错误，你就可以在问题的源头通过标准的方法来处理它们。

## 6.1.1\. 为其他用途使用异常

除了处理实际的错误条件之外，对于异常还有许多其它的用处。在标准 Python 库中一个普通的用法就是试着导入一个模块，然后检查是否它能使用。导入一个并不存在的模块将引发一个 `ImportError` 异常。你可以使用这种方法来定义多级别的功能――依靠在运行时哪个模块是有效的，或支持多种平台 (即平台特定代码被分离到不同的模块中)。

你也能通过创建一个从内置的 `Exception` 类继承的类定义你自己的异常，然后使用 `raise` 命令引发你的异常。如果你对此感兴趣，请看进一步阅读的部分。

下面的例子演示了如何使用异常支持特定平台功能。代码来自 `getpass` 模块，一个从用户获得口令的封装模块。获得口令在 UNIX、Windows 和 Mac OS 平台上的实现是不同的，但是这个代码封装了所有的不同之处。

## 例 6.2\. 支持特定平台功能

```py
 # Bind the name getpass to the appropriate function
  try:
      import termios, TERMIOS                     
  except ImportError:
      try:
          import msvcrt                           
      except ImportError:
          try:
              from EasyDialogs import AskPassword 
          except ImportError:
              getpass = default_getpass           
          else:                                   
              getpass = AskPassword
      else:
          getpass = win_getpass
  else:
      getpass = unix_getpass 
```

|  |  |
| --- | --- |
| [1] | `termios` 是 UNIX 独有的一个模块，它提供了对于输入终端的底层控制。如果这个模块无效 (因为它不在你的系统上，或你的系统不支持它)，则导入失败，Python 引发我们捕捉的 `ImportError` 异常。 |
| [2] | OK，我们没有 `termios`，所以让我们试试 `msvcrt`，它是 Windows 独有的一个模块，可以提供在 Microsoft Visual C++ 运行服务中的许多有用的函数的一个 API。如果导入失败，Python 会引发我们捕捉的 `ImportError` 异常。 |
| [3] | 如果前两个不能工作，我们试着从 `EasyDialogs` 导入一个函数，它是 Mac OS 独有的一个模块，提供了各种各样类型的弹出对话框。再一次，如果导入失败，Python 会引发一个我们捕捉的 `ImportError` 异常。 |
| [4] | 这些平台特定的模块没有一个有效 (有可能，因为 Python 已经移植到了许多不同的平台上了)，所以我们需要回头使用一个缺省口令输入函数 (这个函数定义在 `getpass` 模块中的别的地方)。注意我们在这里所做的：我们将函数 `default_getpass` 赋给变量 `getpass`。如果你读了官方 `getpass` 文档，它会告诉你 `getpass` 模块定义了一个 `getpass` 函数。它是这样做的：通过绑定 `getpass` 到正确的函数来适应你的平台。然后当你调用 `getpass` 函数时，你实际上调用了平台特定的函数，是这段代码已经为你设置好的。你不需要知道或关心你的代码正运行在何种平台上；只要调用 `getpass`，则它总能正确处理。 |
| [5] | 一个 `try...except` 块可以有一条 `else` 子句，就像 `if` 语句。如果在 `try` 块中没有异常引发，然后 `else` 子句被执行。在本例中，那就意味着如果 `from EasyDialogs import AskPassword` 导入可工作，所以我们应该绑定 `getpass` 到 `AskPassword` 函数。其它每个 `try...except` 块有着相似的 `else` 子句，当我们发现一个 `import` 可用时，就绑定 `getpass` 到适合的函数。 |

## 进一步阅读

*   *Python Tutorial* 讨论了异常，包括[定义和引发你自已的异常，以及一次处理多个异常](http://www.python.org/doc/current/tut/node10.html#SECTION0010400000000000000000)。
*   *Python Library Reference* 总结了[所有内置异常](http://www.python.org/doc/current/lib/module-exceptions.html)。
*   *Python Library Reference* 提供了 [getpass](http://www.python.org/doc/current/lib/module-getpass.html) 模块的文档。
*   *Python Library Reference* 提供了 [`traceback` 模块](http://www.python.org/doc/current/lib/module-traceback.html) 的文档，这个模块在异常引发之后，提供了底层的对异常属性的处理。
*   *Python Reference Manual* 讨论了 [`try...except` 块](http://www.python.org/doc/current/ref/try.html) 的内部工作方式。

# 6.2\. 与文件对象共事

# 6.2\. 与文件对象共事

*   6.2.1\. 读取文件
*   6.2.2\. 关闭文件
*   6.2.3\. 处理 I/O 错误
*   6.2.4\. 写入文件

Python 有一个内置函数，`open`，用来打开在磁盘上的文件。`open` 返回一个文件对象，它拥有一些方法和属性，可以得到被打开文件的信息，以及对被打开文件进行操作。

## 例 6.3\. 打开文件

```py
>>> f = open("/music/_singles/kairo.mp3", "rb") 
>>> f                                           
<open file '/music/_singles/kairo.mp3', mode 'rb' at 010E3988>
>>> f.mode                                      
'rb'
>>> f.name                                      
'/music/_singles/kairo.mp3' 
```

|  |  |
| --- | --- |
| [1] | `open` 方法可以接收三个参数：文件名、模式和缓冲区参数。只有第一个参数 (文件名) 是必须的；其它两个是可选的。如果没有指定，文件以文本方式打开。这里我们以二进制方式打开文件进行读取。(`print open.__doc__` 会给出所有可能模式的很好的解释。) |
| [2] | `open` 函数返回一个对象 (到现在为止，这一点应该不会使你感到吃惊)。一个文件对象有几个有用的属性。 |
| [3] | 文件对象的 `mode` 属性告诉你文件以何种模式被打开。 |
| [4] | 文件对象的 `name` 属性告诉你文件对象所打开的文件名。 |

## 6.2.1\. 读取文件

你打开文件之后，你要做的第一件事是从中读取，正如下一个例子所展示的。

## 例 6.4\. 读取文件

```py
>>> f
<open file '/music/_singles/kairo.mp3', mode 'rb' at 010E3988>
>>> f.tell()              
0
>>> f.seek(-128, 2)       
>>> f.tell()              
7542909
>>> tagData = f.read(128) 
>>> tagData
'TAGKAIRO****THE BEST GOA         ***DJ MARY-JANE***            
Rave Mix                      2000http://mp3.com/DJMARYJANE     \037'
>>> f.tell()              
7543037 
```

|  |  |
| --- | --- |
| [1] | 一个文件对象维护它所打开文件的状态。文件对象的 `tell` 方法告诉你在被打开文件中的当前位置。因为我们还没有对这个文件做任何事，当前位置为 `0`，它是文件的起始处。 |
| [2] | 文件对象的 `seek` 方法在被打开文件中移动到另一个位置。第二个参数指出第一个参数是什么意思：`0` 表示移动到一个绝对位置 (从文件起始处算起)，`1` 表示移到一个相对位置 (从当前位置算起)，还有 `2` 表示相对于文件尾的位置。因为我们搜索的 MP3 标记保存在文件的末尾，我们使用 `2` 并且告诉文件对象从文件尾移动到 `128` 字节的位置。 |
| [3] | `tell` 方法确认了当前位置已经移动了。 |
| [4] | `read` 方法从被打开文件中读取指定个数的字节，并且返回含有读取数据的字符串。可选参数指定了读取的最大字节数。如果没有指定参数，`read` 将读到文件末尾。(我们本可以在这里简单地说 `read()` ，因为我们确切地知道在文件的何处，事实上，我们读的是最后 128 个字节。) 读出的数据赋给变量 `tagData`，并且当前的位置根据所读的字节数作了修改。 |
| [5] | `tell` 方法确认了当前位置已经移动了。如果做一下算术，你会看到在读了 128 个字节之后，位置数已经增加了 128。 |

## 6.2.2\. 关闭文件

打开文件消耗系统资源，并且其间其它程序可能无法访问它们 (取决于文件模式)。这就是一旦操作完毕就该关闭文件的重要所在。

## 例 6.5\. 关闭文件

```py
>>> f
<open file '/music/_singles/kairo.mp3', mode 'rb' at 010E3988>
>>> f.closed       
False
>>> f.close()      
>>> f
<closed file '/music/_singles/kairo.mp3', mode 'rb' at 010E3988>
>>> f.closed       
True
>>> f.seek(0)      
Traceback (innermost last):
  File "<interactive input>", line 1, in ?
ValueError: I/O operation on closed file
>>> f.tell()
Traceback (innermost last):
  File "<interactive input>", line 1, in ?
ValueError: I/O operation on closed file
>>> f.read()
Traceback (innermost last):
  File "<interactive input>", line 1, in ?
ValueError: I/O operation on closed file
>>> f.close() 
```

|  |  |
| --- | --- |
| [1] | 文件对象的 `closed` 属性表示对象是打开还是关闭了文件。在本例中，文件仍然打开着 (`closed` 是 `False`)。 |
| [2] | 为了关闭文件，调用文件对象的 `close` 方法。这样就释放掉你加在文件上的锁 (如果有的话)，刷新被缓冲的系统还未写入的输出 (如果有的话)，并且释放系统资源。 |
| [3] | `closed` 属性证实了文件被关闭了。 |
| [4] | 文件被关闭了，但这并不意味着文件对象不再存在。变量 `f` 将继续存在，直到它超出作用域或被手工删除。然而，一旦文件被关闭，操作它的方法就没有一个能使用；它们都会引发异常。 |
| [5] | 对一个文件已经关闭的文件对象调用 `close` *不会* 引发异常，它静静地失败。 |

## 6.2.3\. 处理 I/O 错误

现在你已经足能理解前一章的例子程序 `fileinfo.py` 的文件处理代码了。下面这个例子展示了如何安全地打开文件和读取文件，以及优美地处理错误。

## 例 6.6\. `MP3FileInfo` 中的文件对象

```py
 try:                                
            fsock = open(filename, "rb", 0) 
            try:                           
                fsock.seek(-128, 2)         
                tagdata = fsock.read(128)   
            finally:                        
                fsock.close()              
            .
            .
            .
        except IOError:                     
            pass 
```

|  |  |
| --- | --- |
| [1] | 因为打开和读取文件有风险，并且可能引发异常，所有这些代码都用一个 `try...except` 块封装。(嘿，标准化的缩近不好吗？这就是你开始欣赏它的地方。) |
| [2] | `open` 函数可能引发 `IOError` 异常。(可能是文件不存在。) |
| [3] | `seek` 方法可能引发 `IOError` 异常。(可能是文件长度小于 128 字节。) |
| [4] | `read` 方法可能引发 `IOError` 异常。(可能磁盘有坏扇区，或它在一个网络驱动器上，而网络刚好断了。) |
| [5] | 这是新的：一个 `try...finally` 块。一旦文件通过 `open` 函数被成功地打开，我们应该绝对保证把它关闭，即使是在 `seek` 或 `read` 方法引发了一个异常时。`try...finally` 块可以用来：在 `finally` 块中的代码将*总是* 被执行，甚至某些东西在 `try` 块中引发一个异常也会执行。可以这样考虑，不管在路上发生什么，代码都会被 “即将灭亡” 地执行。 |
| [6] | 最后，处理我们的 `IOError` 异常。它可能是由调用 `open`、`seek` 或 `read` 引发的 `IOError` 异常。这里，我们其实不用关心，因为将要做的事就是静静地忽略它然后继续。(记住，`pass` 是一条不做任何事的 Python 语句。) 这样完全合法，“处理” 一个异常可以明确表示不做任何事。它仍然被认为处理过了，并且处理将正常继续，从 `try...except` 块的下一行代码开始。 |

## 6.2.4\. 写入文件

正如你所期待的，你也能用与读取文件同样的方式写入文件。有两种基本的文件模式：

*   追加 (Append) 模式将数据追加到文件尾。
*   写入 (write) 模式将覆盖文件的原有内容。

如果文件还不存在，任意一种模式都将自动创建文件，因此从来不需要任何复杂的逻辑：“如果 log 文件还不存在，将创建一个新的空文件，正因为如此，你可以第一次就打开它”。打开文件并开始写就可以了。

## 例 6.7\. 写入文件

```py
>>> logfile = open('test.log', 'w') 
>>> logfile.write('test succeeded') 
>>> logfile.close()
>>> print file('test.log').read()   
test succeeded
>>> logfile = open('test.log', 'a') 
>>> logfile.write('line 2')
>>> logfile.close()
>>> print file('test.log').read()   
test succeededline 2 
```

|  |  |
| --- | --- |
| [1] | 你可以大胆地开始创建新文件 `test.log` 或覆盖现有文件，并为写入目的而打开它。(第二个参数 `"w"` 的意思是为文件写入而打开。) 是的，它和想象中的一样危险。我希望你不要关心文件以前的内容，因为它现在已经不存在了。 |
| [2] | 你可以使用 `open` 返回的文件对象的 `write` 方法向一个新打开的文件添加数据。 |
| [3] | `file` 是 `open` 的同义语。这一行语句打开文件，读取内容，并打印它们。 |
| [4] | 碰巧你知道 `test.log` 存在 (因为你刚向它写完了数据)，所以你可以打开它并向其追加数据。(`"a"` 参数的意思是为追加目的打开文件。) 实际上即使文件不存在你也可以这样做，因为以追加方式打开一文件时，如果需要的话会创建文件。但是追加操作*从不* 损坏文件的现有内容。 |
| [5] | 正如你所看到的，原来的行和你以追加方式写入的第二行现在都在 `test.log` 中了。同时注意两行之间并没包含回车符。因为两次写入文件时都没有明确地写入回车符，所以文件中没有包含回车符。你可以用 `"\n"` 写入回车符。因为你没做这项工作，所以你写到文件的所有内容都将显示在同一行上。 |

## 进一步阅读

*   *Python Tutorial* 讨论了文件的读取和写入，包括如何[将一个文件一次一行地读到 list 中](http://www.python.org/doc/current/tut/node9.html#SECTION009210000000000000000)。
*   eff-bot 讨论了[各种各样读取文件方法](http://www.effbot.org/guides/readline-performance.htm) 的效率和性能。
*   Python Knowledge Base 回答了[关于文件的常见问题](http://www.faqts.com/knowledge-base/index.phtml/fid/552)。
*   *Python Library Reference* 总结了[所有文件对象模块](http://www.python.org/doc/current/lib/bltin-file-objects.html)。

# 6.3\. `for` 循环

# 6.3\. `for` 循环

与其它大多数语言一样，Python 也拥有 `for` 循环。你到现在还未曾看到它们的唯一原因就是，Python 在其它太多的方面表现出色，通常你不需要它们。

其它大多数语言没有像 Python 一样的强大的 list 数据类型，所以你需要亲自做很多事情，指定开始，结束和步长，来定义一定范围的整数或字符或其它可重复的实体。但是在 Python 中，`for` 循环简单地在一个列表上循环，与 list 解析的工作方式相同。

## 例 6.8\. `for` 循环介绍

```py
>>> li = ['a', 'b', 'e']
>>> for s in li:         
... print s          
a
b
e
>>> print "\n".join(li)  
a
b
e 
```

|  |  |
| --- | --- |
| [1] | `for` 循环的语法同 list 解析相似。`li` 是一个 list，而 `s` 将从第一个元素开始依次接收每个元素的值。 |
| [2] | 像 `if` 语句或其它任意缩进块，`for` 循环可以包含任意数目的代码行。 |
| [3] | 这就是你以前没看到过 `for` 循环的原因：至今我们都不需要它。太令人吃惊了，当你想要的只是一个 `join` 或是 list 解析时，在其它语言中常常需要使用 `for` 循环。 |

要做一个 “通常的” (Visual Basic 标准的) 计数 `for` 循环也非常简单。

## 例 6.9\. 简单计数

```py
>>> for i in range(5):             
... print i
0
1
2
3
4
>>> li = ['a', 'b', 'c', 'd', 'e']
>>> for i in range(len(li)):       
... print li[i]
a
b
c
d
e 
```

|  |  |
| --- | --- |
| [1] | 正如你在 例 3.20 “连续值赋值” 所看到的，`range` 生成一个整数的 list，通过它来控制循环。我知道它看上去有些奇怪，但是它对计数循环偶尔 (我只是说*偶尔*) 会有用 。 |
| [2] | 我们从来没这么用过。这是 Visual Basic 的思维风格。摆脱它吧。正确遍历 list 的方法是前面的例子所展示的。 |

`for` 循环不仅仅用于简单计数。它们可以遍历任何类型的东西。下面的例子是一个用 `for` 循环遍历 dictionary 的例子。

## 例 6.10\. 遍历 dictionary

```py
>>> import os
>>> for k, v in os.environ.items():       
... print "%s=%s" % (k, v)
USERPROFILE=C:\Documents and Settings\mpilgrim
OS=Windows_NT
COMPUTERNAME=MPILGRIM
USERNAME=mpilgrim
[...略...]
>>> print "\n".join(["%s=%s" % (k, v)
... for k, v in os.environ.items()]) 
USERPROFILE=C:\Documents and Settings\mpilgrim
OS=Windows_NT
COMPUTERNAME=MPILGRIM
USERNAME=mpilgrim
[...略...] 
```

|  |  |
| --- | --- |
| [1] | `os.environ` 是在你的系统上所定义的环境变量的 dictionary。在 Windows 下，这些变量是可以从 MS-DOS 访问的用户和系统变量。在 UNIX 下，它们是在你的 shell 启动脚本中所 export (输出) 的变量。在 Mac OS 中，没有环境变量的概念，所以这个 dictionary 为空。 |
| [2] | `os.environ.items()` 返回一个 tuple 的 list：`[(_key1_, _value1_), (_key2_, _value2_), ...]`。`for` 循环对这个 list 进行遍历。第一轮，它将 `_key1_` 赋给 `k` ，`_value1_` 赋给 `v`，所以 `k` = `USERPROFILE`，`v` = `C:\Documents and Settings\mpilgrim`。第二轮，`k` 得到第二个键字 `OS`，`v` 得到相应的值 `Windows_NT`。 |
| [3] | 使用多变量赋值和 list 解析，你可以使用单行语句来替换整个 `for` 循环。在实际的编码中是否这样做只是个人风格问题；我喜欢它是因为，将一个 dictionary 映射到一个 list，然后将 list 合并成一个字符串，这一过程显得很清晰。其它的程序员宁愿将其写成一个 `for` 循环。请注意在两种情况下输出是一样的，然而这一版本稍微快一些，因为它只有一条 `print` 语句而不是许多。 |

现在我们来看看在 第五章 介绍的样例程序 `fileinfo.py` 中 `MP3FileInfo` 的 `for` 循环 。

## 例 6.11\. `MP3FileInfo` 中的 `for` 循环

```py
 tagDataMap = {"title"   : (  3,  33, stripnulls),
                  "artist"  : ( 33,  63, stripnulls),
                  "album"   : ( 63,  93, stripnulls),
                  "year"    : ( 93,  97, stripnulls),
                  "comment" : ( 97, 126, stripnulls),
                  "genre"   : (127, 128, ord)}                               
    .
    .
    .
            if tagdata[:3] == "TAG":
                for tag, (start, end, parseFunc) in self.tagDataMap.items(): 
                    self[tag] = parseFunc(tagdata[start:end]) 
```

|  |  |
| --- | --- |
| [1] | `tagDataMap` 是一个类属性，它定义了我们正在一个 MP3 文件中搜索的标记。标记存储为定长字段，只要我们读出文件最后 128 个字节，那么第 3 到 32 字节总是歌曲的名字，33-62 总是歌手的名字，63-92 为专辑的名字，等等。请注意 `tagDataMap` 是一个 tuple 的 dictionary，每个 tuple 包含两个整数和一个函数引用。 |
| [2] | 这个看上去复杂一些，但其实并非如此。这里的 `for` 变量结构与 `items` 所返回的 list 的元素的结构相匹配。记住，`items` 返回一个形如 `(_key_, _value_)` 的 tuple 的 list。list 第一个元素是 `("title", (3, 33, &lt;function stripnulls&gt;))`，所以循环的第一轮，`tag` 为 `"title"`，`start` 为 `3`，`end` 为 `33`，`parseFunc` 为函数 `stripnulls`。 |
| [3] | 现在我们已经从一个单个的 MP3 标记中提取出了所有的参数，将标记数据保存起来挺容易。我们从 `start` 到 `end` 对 `tagdata` 进行分片")，从而得到这个标记的实际数据，调用 `parseFunc` 对数据进行后续的处理，接着将 `parseFunc` 的返回值作为值赋值给伪字典 `self` 中的键字 `tag`。在遍历完 `tagDataMap` 中所有元素之后，`self` 拥有了所有标记的值，你知道看上去是什么样。 |

# 6.4\. 使用 ``sys`.modules`

# 6.4\. 使用 ``sys`.modules`

与其它任何 Python 的东西一样，模块也是对象。只要导入了，总可以用全局 dictionary ``sys`.modules` 来得到一个模块的引用。

## 例 6.12\. ``sys`.modules` 介绍

```py
>>> import sys                          
>>> print '\n'.join(sys.modules.keys()) 
win32api
os.path
os
exceptions
__main__
ntpath
nt
sys
__builtin__
site
signal
UserDict
stat 
```

|  |  |
| --- | --- |
| [1] | `sys` 模块包含了系统级的信息，像正在运行的 Python 的版本 (`sys`.version` 或`sys`.version_info`)，和系统级选项，像最大允许递归的深度 (`sys`.getrecursionlimit()` 和`sys`.setrecursionlimit()`)。 |
| [2] | `sys`.modules` 是一个字典，它包含了从 Python 开始运行起，被导入的所有模块。键字就是模块名，键值就是模块对象。请注意除了你的程序导入的模块外还有其它模块。Python 在启动时预先装入了一些模块，如果你在一个 Python IDE 环境下，`sys`.modules` 包含了你在 IDE 中运行的所有程序所导入的所有模块。 |

下面的例子展示了如何使用 ``sys`.modules`。

## 例 6.13\. 使用 ``sys`.modules`

```py
>>> import fileinfo         
>>> print '\n'.join(sys.modules.keys())
win32api
os.path
os
fileinfo
exceptions
__main__
ntpath
nt
sys
__builtin__
site
signal
UserDict
stat
>>> fileinfo
<module 'fileinfo' from 'fileinfo.pyc'>
>>> sys.modules["fileinfo"] 
<module 'fileinfo' from 'fileinfo.pyc'> 
```

|  |  |
| --- | --- |
| [1] | 当导入新的模块，它们加入到 `sys`.modules` 中。这就解释了为什么第二次导入相同的模块时非常的快：Python 已经在`sys`.modules` 中装入和缓冲了，所以第二次导入仅仅对字典做了一个查询。 |
| [2] | 一旦给出任何以前导入过的模块名 (以字符串方式)，通过 ``sys`.modules` 字典，你可以得到对模块本身的一个引用。 |

下面的例子将展示通过结合使用 `__module__` 类属性和 ``sys`.modules` dictionary 来获取已知类所在的模块。

## 例 6.14\. `__module__` 类属性

```py
>>> from fileinfo import MP3FileInfo
>>> MP3FileInfo.__module__              
'fileinfo'
>>> sys.modules[MP3FileInfo.__module__] 
<module 'fileinfo' from 'fileinfo.pyc'> 
```

|  |  |
| --- | --- |
| [1] | 每个 Python 类都拥有一个内置的类属性 `__module__`，它定义了这个类的模块的名字。 |
| [2] | 将它与 ``sys`.modules` 字典复合使用，你可以得到定义了某个类的模块的引用。 |

现在准备好了，看看在样例程序 第五章 ``sys`.modules`介绍的`fileinfo.py` 中是如何使用的。这个例子显示它的一部分代码。

## 例 6.15\. `fileinfo.py` 中的 ``sys`.modules`

```py
 def getFileInfoClass(filename, module=sys.modules[FileInfo.__module__]):       
        "get file info class from filename extension"                             
        subclass = "%sFileInfo" % os.path.splitext(filename)[1].upper()[1:]        
        return hasattr(module, subclass) and getattr(module, subclass) or FileInfo 
```

|  |  |
| --- | --- |
| [1] | 这是一个有两个参数的函数；`filename` 是必须的，但 `module` 是可选的并且 module 的缺省值包含了 `FileInfo` 类。这样看上去效率低，因为你可能认为 Python 会在每次函数调用时计算这个 ``sys`.modules`表达式。实际上，Python 仅会对缺省表达式计算一次，是在模块导入的第一次。正如后面我们会看到的，我们永远不会用一个`module`参数来调用这个函数，所以`module` 的功能是作为一个函数级别的常量。 |
| [2] | 我们会在后面再仔细研究这一行，在我们了解了 `os` 模块之后。那么现在，只要相信 `subclass` 最终为一个类的名字就行了，像 `MP3FileInfo`。 |
| [3] | 你已经了解了 `getattr`，它可以通过名字得到一个对象的引用。`hasattr` 是一个补充性的函数，用来检查一个对象是否具有一个特定的属性；在本例中，用来检查一个模块是否有一个特别的类 (然而它可以用于任何类和任何属性，就像 `getattr`)。用英语来说，这行代码是说，“If this module has the class named by `subclass` then return it, otherwise return the base class `FileInfo` (如果这个模块有一个名为 `subclass` 的类，那么返回它，否则返回基类 `FileInfo`)”。 |

## 进一步阅读

*   *Python Tutorial* 讨论了[缺省参数到底在什么时候和是如何计算的](http://www.python.org/doc/current/tut/node6.html#SECTION006710000000000000000)。
*   *Python Library Reference* 提供了 [`sys`](http://www.python.org/doc/current/lib/module-sys.html) 模块的文档。

# 6.5\. 与目录共事

# 6.5\. 与目录共事

`os.path` 模块有几个操作文件和目录的函数。这里，我们看看如何操作路径名和列出一个目录的内容。

## 例 6.16\. 构造路径名

```py
>>> import os
>>> os.path.join("c:\\music\\ap\\", "mahadeva.mp3")  
'c:\\music\\ap\\mahadeva.mp3'
>>> os.path.join("c:\\music\\ap", "mahadeva.mp3")   
'c:\\music\\ap\\mahadeva.mp3'
>>> os.path.expanduser("~")                         
'c:\\Documents and Settings\\mpilgrim\\My Documents'
>>> os.path.join(os.path.expanduser("~"), "Python") 
'c:\\Documents and Settings\\mpilgrim\\My Documents\\Python' 
```

|  |  |
| --- | --- |
| [1] | `os.path` 是一个模块的引用；使用哪一个模块要看你正运行在哪种平台上。就像 `getpass` 通过将 `getpass` 设置为一个与平台相关的函数从而封装了平台之间的不同。`os` 通过设置 `path` 封装不同的相关平台模块。 |
| [2] | `os.path` 的 `join` 函数把一个或多个部分路径名连接成一个路径名。在这个简单的例子中，它只是将字符串进行连接。(请注意在 Windows 下处理路径名是一个麻烦的事，因为反斜线字符必须被转义。) |
| [3] | 在这个几乎没有价值的例子中，在将路径名加到文件名上之前，`join` 将在路径名后添加额外的反斜线。当发现这一点时我高兴极了，因为当用一种新的语言创建我自已的工具包时，`addSlashIfNecessary` 总是我必须要写的那些愚蠢的小函数之一。在 Python 中*不要* 写这样的愚蠢的小函数，聪明的人已经为你考虑到了。 |
| [4] | `expanduser` 将对使用 `~` 来表示当前用户根目录的路径名进行扩展。在任何平台上，只要用户拥有一个根目录，它就会有效，像 Windows、UNIX 和 Mac OS X，但在 Mac OS 上无效。 |
| [5] | 将这些技术组合在一起，你可以容易地为在用户根目录下的目录和文件构造出路径名。 |

## 例 6.17\. 分割路径名

```py
>>> os.path.split("c:\\music\\ap\\mahadeva.mp3")                        
('c:\\music\\ap', 'mahadeva.mp3')
>>> (filepath, filename) = os.path.split("c:\\music\\ap\\mahadeva.mp3") 
>>> filepath                                                            
'c:\\music\\ap'
>>> filename                                                            
'mahadeva.mp3'
>>> (shortname, extension) = os.path.splitext(filename)                 
>>> shortname
'mahadeva'
>>> extension
'.mp3' 
```

|  |  |
| --- | --- |
| [1] | `split` 函数对一个全路径名进行分割，返回一个包含路径和文件名的 tuple。还记得我说过你可以使用多变量赋值从一个函数返回多个值吗？对，`split` 就是这样一个函数。 |
| [2] | 我们将 `split` 函数的返回值赋值给一个两个变量的 tuple。每个变量接收到返回 tuple 相对应的元素值。 |
| [3] | 第一个变量，`filepath`，接收到从 `split` 返回 tuple 的第一个元素的值，文件路径。 |
| [4] | 第二个变量，`filename`，接收到从 `split` 返回 tuple 的第二个元素的值，文件名。 |
| [5] | `os.path` 也包含了一个 `splitext` 函数，可以用来对文件名进行分割，并且返回一个包含了文件名和文件扩展名的 tuple。我们使用相同的技术来将它们赋值给独立的变量。 |

## 例 6.18\. 列出目录

```py
>>> os.listdir("c:\\music\\_singles\\")              
['a_time_long_forgotten_con.mp3', 'hellraiser.mp3',
'kairo.mp3', 'long_way_home1.mp3', 'sidewinder.mp3', 
'spinning.mp3']
>>> dirname = "c:\\"
>>> os.listdir(dirname)                              
['AUTOEXEC.BAT', 'boot.ini', 'CONFIG.SYS', 'cygwin',
'docbook', 'Documents and Settings', 'Incoming', 'Inetpub', 'IO.SYS',
'MSDOS.SYS', 'Music', 'NTDETECT.COM', 'ntldr', 'pagefile.sys',
'Program Files', 'Python20', 'RECYCLER',
'System Volume Information', 'TEMP', 'WINNT']
>>> [f for f in os.listdir(dirname)
... if os.path.isfile(os.path.join(dirname, f))] 
['AUTOEXEC.BAT', 'boot.ini', 'CONFIG.SYS', 'IO.SYS', 'MSDOS.SYS',
'NTDETECT.COM', 'ntldr', 'pagefile.sys']
>>> [f for f in os.listdir(dirname)
... if os.path.isdir(os.path.join(dirname, f))]  
['cygwin', 'docbook', 'Documents and Settings', 'Incoming',
'Inetpub', 'Music', 'Program Files', 'Python20', 'RECYCLER',
'System Volume Information', 'TEMP', 'WINNT'] 
```

|  |  |
| --- | --- |
| [1] | `listdir` 函数接收一个路径名，并返回那个目录的内容的 list。 |
| [2] | `listdir` 同时返回文件和文件夹，并不指出哪个是文件，哪个是文件夹。 |
| [3] | 你可以使用过滤列表和 `os.path` 模块的 `isfile` 函数，从文件夹中将文件分离出来。`isfile` 接收一个路径名，如果路径表示一个文件，则返回 1，否则为 0。在这里，我们使用 `os.path`.`join` 来确保得到一个全路径名，但 `isfile` 对部分路径 (相对于当前目录) 也是有效的。你可以使用 `os.getcwd()` 来得到当前目录。 |
| [4] | `os.path` 还有一个 `isdir` 函数，当路径表示一个目录，则返回 1，否则为 0。你可以使用它来得到一个目录下的子目录列表。 |

## 例 6.19\. 在 `fileinfo.py` 中列出目录

```py
 def listDirectory(directory, fileExtList):                                        
    "get list of file info objects for files of particular extensions" 
    fileList = [os.path.normcase(f)
                for f in os.listdir(directory)]             
    fileList = [os.path.join(directory, f) 
               for f in fileList
                if os.path.splitext(f)[1] in fileExtList] 
```

|  |  |
| --- | --- |
| [1] | `os.listdir(directory)` 返回在 `directory` 中所有文件和文件夹的一个 list。 |
| [2] | 使用 `f` 对 list 进行遍历，我们使用 `os.path.normcase(f)` 根据操作系统的缺省值对大小写进行标准化处理。`normcase` 是一个有用的函数，用于对大小写不敏感操作系统的一个补充。这种操作系统认为 `mahadeva.mp3` 和 `mahadeva.MP3` 是同一个文件名。例如，在 Windows 和 Mac OS 下，`normcase` 将把整个文件名转换为小写字母；而在 UNIX 兼容的系统下，它将返回未作修改的文件名。 |
| [3] | 再次用 `f` 对标准化后的 list 进行遍历，我们使用 `os.path.splitext(f)` 将每个文件名分割为名字和扩展名。 |
| [4] | 对每个文件，我们查看扩展名是否在我们关心的文件扩展名 list 中 (`fileExtList`，被传递给 `listDirectory` 函数)。 |
| [5] | 对每个我们所关心的文件，我们使用 `os.path.join(directory, f)` 来构造这个文件的全路径名，接着返回这个全路径名的 list。 |

> 注意
> 只要有可能，你就应该使用在 `os` 和 `os.path` 中的函数进行文件、目录和路径的操作。这些模块是对平台相关模块的封装模块，所以像 `os.path.split` 这样的函数可以工作在 UNIX、Windows、Mac OS 和 Python 所支持的任一种平台上。

还有一种获得目录内容的方法。它非常强大，并使用了一些你在命令行上工作时可能已经熟悉的通配符。

## 例 6.20\. 使用 `glob` 列出目录

```py
>>> os.listdir("c:\\music\\_singles\\")               
['a_time_long_forgotten_con.mp3', 'hellraiser.mp3',
'kairo.mp3', 'long_way_home1.mp3', 'sidewinder.mp3',
'spinning.mp3']
>>> import glob
>>> glob.glob('c:\\music\\_singles\\*.mp3')           
['c:\\music\\_singles\\a_time_long_forgotten_con.mp3',
'c:\\music\\_singles\\hellraiser.mp3',
'c:\\music\\_singles\\kairo.mp3',
'c:\\music\\_singles\\long_way_home1.mp3',
'c:\\music\\_singles\\sidewinder.mp3',
'c:\\music\\_singles\\spinning.mp3']
>>> glob.glob('c:\\music\\_singles\\s*.mp3')          
['c:\\music\\_singles\\sidewinder.mp3',
'c:\\music\\_singles\\spinning.mp3']
>>> glob.glob('c:\\music\\*\\*.mp3') 
```

|  |  |
| --- | --- |
| [1] | 正如你前面看到的，`os.listdir` 简单地取一个目录路径，返回目录中的所有文件和子目录。 |
| [2] | `glob` 模块，另一方面，接受一个通配符并且返回文件的或目录的完整路径与之匹配。这个通配符是一个目录路径加上“*.mp3”，它将匹配所有的 `.mp3` 文件。注意返回列表的每一个元素已经包含了文件的完整路径。 |
| [3] | 如果你要查找指定目录中所有以“s”开头并以“.mp3”结尾的文件，也可以这么做。 |
| [4] | 现在考查这种情况：你有一个 `music` 目录，它包含几个子目录，子目录中包含一些 `.mp3` 文件。使用两个通配符，仅仅调用 `glob` 一次就可以立刻获得所有这些文件的一个 list。一个通配符是 `"*.mp3"` (用于匹配 `.mp3` 文件)，另一个通配符是*子目录名本身*，用于匹配 `c:\music` 中的所有子目录。这看上去很简单，但它蕴含了强大的功能。 |

## 进一步阅读

*   Python Knowledge Base 回答了[关于 `os` 模块的问题](http://www.faqts.com/knowledge-base/index.phtml/fid/240)。
*   *Python Library Reference* 提供了 [`os`](http://www.python.org/doc/current/lib/module-os.html) 模块和 [`os.path`](http://www.python.org/doc/current/lib/module-os.path.html) 模块的文档。

# 6.6\. 全部放在一起

# 6.6\. 全部放在一起

再一次，所有的多米诺骨牌都放好了。我们已经看过每行代码是如何工作的了。现在往回走一步，看一下放在一起是怎么样的。

## 例 6.21\. `listDirectory`

```py
 def listDirectory(directory, fileExtList):                                         
    "get list of file info objects for files of particular extensions"
    fileList = [os.path.normcase(f)
                for f in os.listdir(directory)]           
    fileList = [os.path.join(directory, f) 
               for f in fileList
                if os.path.splitext(f)[1] in fileExtList]                          
    def getFileInfoClass(filename, module=sys.modules[FileInfo.__module__]):       
        "get file info class from filename extension"                             
        subclass = "%sFileInfo" % os.path.splitext(filename)[1].upper()[1:]        
        return hasattr(module, subclass) and getattr(module, subclass) or FileInfo 
    return [getFileInfoClass(f)(f) for f in fileList] 
```

|  |  |
| --- | --- |
| [1] | `listDirectory` 是整个模块主要的有趣之处。它接收一个 dictionary (在我的例子中如 `c:\music\_singles\`) 和一个感兴趣的文件扩展名列表 (如 `['.mp3']`)，接着它返回一个类实例的 list ，这些类实例的行为像 dictionary，包含了在目录中每个感兴趣文件的元数据。并且实现起来只用了几行直观的代码。 |
| [2] | 正如在前一节我们所看到的，这行代码得到一个全路径名的列表，它的元素是在 `directory` 中有着我们感兴趣的文件后缀 (由 `fileExtList` 所指定的) 的所有文件的路径名。 |
| [3] | 老学校出身的 Pascal 程序员可能对嵌套函数感到熟悉，但大部分人，当我告诉他们 Python 支持嵌套函数时，都茫然地看着我。*嵌套函数*，从字面理解，是定义在函数内的函数。嵌套函数 `getFileInfoClass` 只能在定义它的函数 `listDirectory` 内进行调用。正如任何其它的函数一样，不需要一个接口声明或奇怪的什么东西，只要定义函数，开始编码就行了。 |
| [4] | 既然你已经看过 `os` 模块了，这一行应该能理解了。它得到文件的扩展名 (`os.path.splitext(filename)[1]`)，将其转换为大写字母 (`.upper()`)，从圆点处进行分片 (`[1:]`)，使用字符串格式化从其中生成一个类名。所以 `c:\music\ap\mahadeva.mp3` 变成 `.mp3` 再变成 `MP3` 再变成 `MP3FileInfo`。 |
| [5] | 在生成完处理这个文件的处理类的名字之后，我们查阅在这个模块中是否存在这个处理类。如果存在，我们返回这个类，否则我们返回基类 `FileInfo`。这一点很重要：*这个函数返回一个类*。不是类的实例，而是类本身。 |
| [6] | 对每个属于我们 “感兴趣文件” 列表 (`fileList`)中的文件，我们用文件名 (`f`) 来调用 `getFileInfoClass`。调用 `getFileInfoClass(f)` 返回一个类；我们并不知道确切是哪一个类，但是我们并不关心。接着我们创建这个类 (不管它是什么) 的一个实例，传入文件名 (又是 `f`) 给 `__init__` 方法。正如我们在本章的前面所看到的，`FileInfo` 的 `__init__` 方法设置了 `self["name"]`，它将引发 `__setitem__` 的调用，而 `__setitem__` 在子类 (`MP3FileInfo`) 中被覆盖掉了，用来适当地对文件进行分析，取出文件的元数据。我们对所有感兴趣的文件进行处理，返回结果实例的一个 list。 |

请注意 `listDirectory` 完全是通用的。它事先不知道将得到哪种类型的文件，也不知道哪些定义好的类能够处理这些文件。它检查目录中要进行处理的文件，然后反观本身模块，了解定义了什么特别的处理类 (像 `MP3FileInfo`)。你可以对这个程序进行扩充，对其它类型的文件进行处理，只要用适合的名字定义类：`HTMLFileInfo` 用于 HTML 文件，`DOCFileInfo` 用于 Word `.doc` 文件，等等。不需要改动函数本身， `listDirectory` 将会对它们都进行处理，将工作交给适当的类，接着收集结果。

# 6.7\. 小结

# 6.7\. 小结

在 第五章 介绍的 `fileinfo.py` 程序现在应该完全理解了。

```py
"""Framework for getting filetype-specific metadata.
Instantiate appropriate class with filename.  Returned object acts like a
dictionary, with key-value pairs for each piece of metadata.
    import fileinfo
    info = fileinfo.MP3FileInfo("/music/ap/mahadeva.mp3")
    print "\\n".join(["%s=%s" % (k, v) for k, v in info.items()])
Or use listDirectory function to get info on all files in a directory.
    for info in fileinfo.listDirectory("/music/ap/", [".mp3"]):
        ...
Framework can be extended by adding classes for particular file types, e.g.
HTMLFileInfo, MPGFileInfo, DOCFileInfo.  Each class is completely responsible for
parsing its files appropriately; see MP3FileInfo for example.
"""
import os
import sys
from UserDict import UserDict
def stripnulls(data):
    "strip whitespace and nulls"
    return data.replace("\00", "").strip()
class FileInfo(UserDict):
    "store file metadata"
    def __init__(self, filename=None):
        UserDict.__init__(self)
        self["name"] = filename
class MP3FileInfo(FileInfo):
    "store ID3v1.0 MP3 tags"
    tagDataMap = {"title"   : (  3,  33, stripnulls),
                  "artist"  : ( 33,  63, stripnulls),
                  "album"   : ( 63,  93, stripnulls),
                  "year"    : ( 93,  97, stripnulls),
                  "comment" : ( 97, 126, stripnulls),
                  "genre"   : (127, 128, ord)}
    def __parse(self, filename):
        "parse ID3v1.0 tags from MP3 file"
        self.clear()
        try:                               
            fsock = open(filename, "rb", 0)
            try:                           
                fsock.seek(-128, 2)        
                tagdata = fsock.read(128)  
            finally:                       
                fsock.close()              
            if tagdata[:3] == "TAG":
                for tag, (start, end, parseFunc) in self.tagDataMap.items():
                    self[tag] = parseFunc(tagdata[start:end])               
        except IOError:                    
            pass                           
    def __setitem__(self, key, item):
        if key == "name" and item:
            self.__parse(item)
        FileInfo.__setitem__(self, key, item)
def listDirectory(directory, fileExtList):                                        
    "get list of file info objects for files of particular extensions"
    fileList = [os.path.normcase(f)
                for f in os.listdir(directory)]           
    fileList = [os.path.join(directory, f) 
               for f in fileList
                if os.path.splitext(f)[1] in fileExtList] 
    def getFileInfoClass(filename, module=sys.modules[FileInfo.__module__]):      
        "get file info class from filename extension"                             
        subclass = "%sFileInfo" % os.path.splitext(filename)[1].upper()[1:]       
        return hasattr(module, subclass) and getattr(module, subclass) or FileInfo
    return [getFileInfoClass(f)(f) for f in fileList]                             
if __name__ == "__main__":
    for info in listDirectory("/music/_singles/", [".mp3"]):
        print "\n".join(["%s=%s" % (k, v) for k, v in info.items()])
        print 
```

在研究下一章之前，确保你可以无困难地完成下面的事情：

*   使用 `try...except` 来捕捉异常
*   使用 `try...finally` 来保护额外的资源
*   读取文件
*   在一个 `for` 循环中一次赋多个值
*   使用 `os` 模块来满足你的跨平台文件操作的需要
*   通过将类看成对象并传入参数，动态地实例化未知类型的类