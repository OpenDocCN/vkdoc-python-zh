# Chapter 11 文件

> " A nine mile walk is no joke, especially in the rain. " — Harry Kemelman, The Nine Mile Walk

## 概要

在没有安装任何一个应用程序之前，我的笔记本上 Windows 系统有 38,493 个文件。安装 Python 3 后，大约增加了 3,000 个文件。文件是每一个主流操作系统的主要存储模型；这种观念如此根深蒂固以至于难以想出一种[替代物](http://en.wikipedia.org/wiki/Computer_file#History)。打个比方，你的电脑实际上就是泡在文件里了。

## 读取文本文件

在读取文件之前，你需要先打开它。在 Python 里打开一个文件很简单：

```py
a_file = open('examples/chinese.txt', encoding='utf-8') 
```

Python 有一个内置函数 `open()`，它使用一个文件名作为其参数。在以上代码中，文件名是 `'examples/chinese.txt'`。关于这个文件名，有五件值得一讲的事情：

1.  它不仅是一个文件的名字；实际上，它是文件路径和文件名的组合；一般来说，文件打开函数应该有两个参数 — 路径和文件名 — 但是函数`open()`只使用一个参数。在 Python 里，当你使用“filename,”作为参数的时候，你可以将部分或者全部的路径也包括进去。
2.  在这个例子中，目录路径中使用的是斜杠(forward slash)，但是我并没有说明我正在使用的操作系统。Windows 使用反斜杠来表示子目录，但是 Mac OS X 和 Linux 使用斜杠。但是，在 Python 中，斜杠永远都是正确的，即使是在 Windows 环境下。
3.  不使用斜杠或者反斜杠的路径被称作*相对路径(relative path)*。你也许会问，相对于什么呢？耐心一些，伙计。
4.  “filename,”参数是一个字符串。所有现代的操作系统（甚至 Windows！）使用 Unicode 编码方式来存储文件名和目录名。Python 3 全面支持非 ASCII 编码的路径。
5.  文件不一定需要在本地磁盘上。也许你挂载了一个网络驱动器。它也可以是一个完全虚拟的文件系统([an entirely virtual filesystem](http://en.wikipedia.org/wiki/Filesystem_in_Userspace))上的文件。只要你的操作系统认为它是一个文件，并且能够以文件的方式访问，那么，Python 就能打开它。

但是对`open()`函数的调用不局限于`filename`。还有另外一个叫做`encoding`参数。天哪，似乎非常耳熟的样子！

### 字符编码抬起了它腌臜的头…

字节即字节；字符是一种抽象。字符串由使用 Unicode 编码的字符序列构成。但是磁盘上的文件不是 Unicode 编码的字符序列。文件是字节序列。所以你可能会想，如果从磁盘上读取一个“文本文件”，Python 是怎样把那个字节序列转化为字符序列的呢？实际上，它是根据特定的字符解码算法来解释这些字节序列，然后返回一串使用 Unicode 编码的字符（或者也称为字符串）。

```py
# This example was created on Windows. Other platforms may
# behave differently, for reasons outlined below.
# 这个样例在 Windows 平台上创建。其他平台可能会有不同的表现，理由描述在下边
>>> file = open('examples/chinese.txt')
>>> a_string = file.read()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "C:\Python31\lib\encodings\cp1252.py", line 23, in decode
    return codecs.charmap_decode(input,self.errors,decoding_table)[0]
UnicodeDecodeError: 'charmap' codec can't decode byte 0x8f in position 28: character maps to <undefined>
>>> 
```

默认的编码方式是平台相关的。

刚才发生了什么？由于你没有指定字符编码的方式，所以 Python 被迫使用默认的编码。那么默认的编码方式是什么呢？如果你仔细看了跟踪信息(traceback)，错误出现在`cp1252.py`，这意味着 Python 此时正在使用 CP-1252 作为默认的编码方式。（在运行微软视窗操作系统的机器上，CP-1252 是一种常用的编码方式。）CP-1252 的字符集不支持这个文件上的字符编码，所以它以这个可恶的`UnicodeDecodeError`错误读取失败。

但是，还有更糟糕的！因为默认的编码方式是*平台相关的(platform-dependent)*，所以，当前的代码*也许*能够在你的电脑上运行（如果你的机器的默认编码方式是 UTF-8），但是当你把这份代码分发给其他人的时候可能就会失败（因为他们的默认编码方式可能跟你的不一样，比如说 CP-1252）。

> ☞如果你需要获得默认编码的信息，则导入`locale`模块，然后调用`locale.getpreferredencoding()`。在我安装了 Windows 的笔记本上，它的返回值是`'cp1252'`，但是在我楼上安装了 Linux 的台式机上边，它返回`'UTF8'`。你看，即使在我自己家里我都不能保证一致性(consistency)！你的运行结果也许不一样（即使在 Windows 平台上），这依赖于操作系统的版本和区域/语言选项的设置。这就是为什么每次打开一个文件的时候指定编码方式是如此重要了。

### 流对象

到目前为止，我们都知道 Python 有一个内置的函数叫做`open()`。`open()`函数返回一个流对象（stream object），它拥有一些用来获取信息和操作字符流的方法和属性。

```py
>>> a_file = open('examples/chinese.txt', encoding='utf-8')

'examples/chinese.txt'

'utf-8'

'r' 
```

1.  `name`属性反映的是当你打开文件时传递给`open()`函数的文件名。它没有被标准化(normalize)成绝对路径。
2.  同样的，`encoding`属性反映的是在你调用`open()`函数时指定的编码方式。如果你在打开文件的时候没有指定编码方式（不好的开发人员！），那么`encoding`属性反映的是`locale.getpreferredencoding()`的返回值。
3.  `mode`属性会告诉你被打开文件的访问模式。你可以传递一个可选的`mode`参数给`open()`函数。如果在打开文件的时候没有指定访问模式，Python 默认设置模式为`'r'`，意思是“在文本模式下以只读的方式打开。”在这章的后面你会看到，文件的访问模式有各种用途；不同模式能够使你写入一个文件，追加到一个文件，或者以二进制模式打开一个文件（在这种情况下，你处理的是字节，不再是字符）。

> ☞[`open()`函数的文档](http://docs.python.org/3.1/library/io.html#module-interface)列出了所有可用的文件访问模式。

### 从文本文件读取数据

在打开文件以后，你可能想要从某处开始读取它。

```py
>>> a_file = open('examples/chinese.txt', encoding='utf-8')

'Dive Into Python 是为有经验的程序员编写的一本 Python 书。\n'

'' 
```

1.  只要成功打开了一个文件（并且指定了正确的编码方式），你只需要调用流对象的`read()`方法即可以读取它。返回的结果是文件的一个字符串表示。
2.  也许你会感到意外，再次读取文件不会产生一个异常。Python 不认为到达了文件末尾(end-of-file)还继续执行读取操作是一个错误；这种情况下，它只是简单地返回一个空字符串。

无论何时，打开文件时指定`encoding`参数。

如果想要重新读取文件呢？

```py
# continued from the previous example
# 接着前一个例子

''

0

'Dive Into Python'

' '
>>> a_file.read(1)
'是'

20 
```

1.  由于你依旧在文件的末尾，继续调用`read()`方法只会返回一个空字符串。
2.  `seek()`方法使定位到文件中的特定字节。
3.  `read()`方法可以使用一个可选的参数，即所要读取的字符个数。
4.  只要愿意，你甚至可以一次读取一个字符。
5.  16 + 1 + 1 = … 20?

我们再来做一遍。

```py
# continued from the previous example
# 继续上一示例

17

'是'

20 
```

1.  移动到第 17th 个字节位置。
2.  读取一个字符。
3.  当前在第 20 个字节位置处。

你是否已经注意到了？`seek()`和`tell()`方法总是以字节的方式计数，但是，由于你是以文本文件的方式打开的，`read()`方法以*字符*的个数计数。中文字符的 UTF-8 编码需要多个字节。而文件里的英文字符每一个只需要一个字节来存储，所以你可能会产生这样的误解：`seek()`和`read()`方法对相同的目标计数。而实际上，只有对部分字符的情况是这样的。

但是，还有更糟的！

```py
 18

Traceback (most recent call last):
  File "<pyshell#12>", line 1, in <module>
    a_file.read(1)
  File "C:\Python31\lib\codecs.py", line 300, in decode
    (result, consumed) = self._buffer_decode(data, self.errors, final)
UnicodeDecodeError: 'utf8' codec can't decode byte 0x98 in position 0: unexpected code byte 
```

1.  定位到第 18th 个字节，然后试图读取一个字符。
2.  为什么这里会失败？因为在第 18 个字节处不存在字符。距离此处最近的字符从第 17 个字节开始（长度为三个字节）。试图从一个字符的中间位置读取会导致程序以`UnicodeDecodeError`错误失败。

### 关闭文件

打开文件会占用系统资源，根据文件的打开模式不同，其他的程序也许不能够访问它们。当已经完成了对文件的操作后就立即关闭它们，这很重要。

```py
# continued from the previous example
# 继续前面的例子
>>> a_file.close() 
```

然而，这还不够(anticlimactic)。

流对象`a_file`仍然存在；调用`close()`方法并没有把对象本身销毁。所以这并不是非常有效。

```py
# continued from the previous example
# 接着上一示例

Traceback (most recent call last):
  File "<pyshell#24>", line 1, in <module>
    a_file.read()
ValueError: I/O operation on closed file.

Traceback (most recent call last):
  File "<pyshell#25>", line 1, in <module>
    a_file.seek(0)
ValueError: I/O operation on closed file.

Traceback (most recent call last):
  File "<pyshell#26>", line 1, in <module>
    a_file.tell()
ValueError: I/O operation on closed file.

True 
```

1.  不能读取已经关闭了的文件；那样会引发一个`IOError`异常。
2.  也不能对一个已经关闭了的文件执行定位操作。
3.  由于文件已经关闭了，所以也就不存在所谓当前的位置了，所以`tell()`也会失败。
4.  也许你会有些意外，文件已经关闭，调用原来流对象的`close()`方法并*没有*引发异常。其实那只是一个空操作(no-op)而已。
5.  已经关闭了的流对象确实还有一个有用的属性：`closed`用来确认文件是否已经被关闭了。

### 自动关闭文件

`try..finally`也行。但是`with`更好

流对象有一个显式的`close()`方法，但是如果代码有缺陷，在调用`close()`方法以前就崩溃了呢？理论上，那个文件会在相当长的一段时间内一直打开着，这是没有必要地。当你在自己的机器上调试的时候，这不算什么大问题。但是当这种代码被移植到服务器上运行，也许就得三思了。

对于这种情况，Python 2 有一种解决办法：`try..finally`块。这种方法在 Python 3 里仍然有效，也许你可以在其他人的代码，或者从比较老的被移植到 Python 3 的代码中看到它。但是 Python 2.5 引入了一种更加简洁的解决方案，并且 Python 3 将它作为首选方案：`with`语句。

```py
with open('examples/chinese.txt', encoding='utf-8') as a_file:
    a_file.seek(17)
    a_character = a_file.read(1)
    print(a_character) 
```

这段代码调用了`open()`函数，但是它却一直没有调用`a_file.close()`。`with`语句引出一个代码块，就像`if`语句或者`for`循环一样。在这个代码块里，你可以使用变量`a_file`作为`open()`函数返回的流对象的引用。所以流对象的常规方法都是可用的 — `seek()`，`read()`，无论你想要调用什么。当`with`块结束时，*Python 自动调用`a_file.close()`*。

这就是它与众不同的地方：无论你以何种方式跳出`with`块，Python 会自动关闭那个文件…即使是因为未处理的异常而“exit”。是的，即使代码中引发了一个异常，整个程序突然中止了，Python 也能够保证那个文件能被关闭掉。

> ☞从技术上说，`with`语句创建了一个运行时环境(runtime context)。在这几个样例中，流对象的行为就像一个上下文管理器(context manager)。Python 创建了`a_file`，并且告诉它正进入一个运行时环境。当`with`块结束的时候，Python 告诉流对象它正在退出这个运行时环境，然后流对象就会调用它的`close()`方法。请阅读 附录 B，“能够在`with`块中使用的类”以获取更多细节。

`with`语句不只是针对文件而言的；它是一个用来创建运行时环境的通用框架(generic framework)，告诉对象它们正在进入和离开一个运行时环境。如果该对象是流对象，那么它就会做一些类似文件对象一样有用的动作（就像自动关闭文件！）。但是那个行为是被流对象自身定义的，而不是在`with`语句中。还有许多跟文件无关的使用上下文管理器(context manager)的方法。在这章的后面可以看到，你甚至可以自己创建它们。

### 一次读取一行数据

正如你所想的，一行数据就是这样 — 输入一些单词，按`ENTER`键，然后就在新的一行了。一行文本就是一串被某种东西分隔的字符，到底是被什么分隔的呢？好吧，这有些复杂，因为文本文件可以使用几个不同的字符来标记行末(end of a line)。每种操作系统都有自己的规矩。有一些使用回车符(carriage return)，另外一些使用换行符(line feed)，还有一些在行末同时使用这两个字符来标记。

其实你可以舒口气了，因为*Python 默认会自动处理行的结束符*。如果你告诉它，“我想从这个文本文件一次读取一行，”Python 自己会弄明白这个文本文件到底使用哪种方式标记新行，然后正确工作。

> ☞如果想要细粒度地控制(fine-grained control)使用哪种新行标记符，你可以传递一个可选的参数`newline`给`open()`函数。请阅读[`open()`函数的文档](http://docs.python.org/3.1/library/io.html#module-interface)以获取更多细节。

那么，实际中你会怎样做呢？我是指一次读取文件的一行。它如此简单优美…

```py
line_number = 0

        line_number += 1 
```

1.  使用`with`语句，安全地打开这个文件，然后让 Python 为你关闭它。
2.  为了一次读取文件的一行，使用`for`循环。是的，除了像`read()`这样显式的方法，*流对象也是一个迭代器(iterator)*，它能在你每次请求一个值时分离出单独的一行。
3.  使用字符串的`format()`方法，你可以打印出行号和行自身。格式说明符`{:&gt;4}`的意思是“使用最多四个空格使之右对齐，然后打印此参数。”变量`a_line`是包括回车符等在内的完整的一行。字符串方法`rstrip()`可以去掉尾随的空白符，包括回车符。

```py
you@localhost:~/diveintopython3$ python3 examples/oneline.py
 1 Dora
   2 Ethan
   3 Wesley
   4 John
   5 Anne
   6 Mike
   7 Chris
   8 Sarah
   9 Alex
  10 Lizzie 
```

> 是否遇到了这个错误？
> 
> ```py
> you@localhost:~/diveintopython3$ python3 examples/oneline.py
> Traceback (most recent call last):
>   File "examples/oneline.py", line 4, in &lt;module&gt;
>     print('{:&gt;4} {}'.format(line_number, a_line.rstrip()))
> ValueError: zero length field name in format 
> ```
> 
> 如果结果是这样，也许你正在使用 Python 3.0。你真的应该升级到 Python 3.1。
> 
> Python 3.0 支持字符串格式化，但是只支持显式编号了的格式说明符。Python 3.1 允许你在格式说明符里省略参数索引号。作为比照，下面是一个 Python 3.0 兼容的版本。
> 
> ```py
> print('{&lt;mark&gt;0&lt;/mark&gt;:&gt;4} {&lt;mark&gt;1&lt;/mark&gt;}'.format(line_number, a_line.rstrip())) 
> ```

## 写入文本文件

打开文件然后开始写入即可。

写入文件的方式和从它们那儿读取很相似。首先打开一个文件，获取流对象，然后你调用一些方法作用在流对象上来写入数据到文件，最后关闭文件。

为了写入而打开一个文件，可以使用`open()`函数，并且指定写入模式。有两种文件模式用于写入：

*   “写”模式会重写文件。传递`mode='w'`参数给`open()`函数。
*   “追加”模式会在文件末尾添加数据。传递`mode='a'`参数给`open()`函数。

如果文件不存在，两种模式下都会自动创建新文件，所以就不需要“如果文件还不存在，创建一个新的空白文件以能够打开它”这种琐碎的过程了。所以，只需要打开一个文件，然后开始写入即可。

在完成写入后你应该马上关闭文件，释放文件句柄(file handle)，并且保证数据被完整地写入到了磁盘。跟读取文件一样，可以调用流对象的`close()`方法，或者你也可以使用`with`语句让 Python 为你关闭文件。我敢打赌，你肯定能猜到我推荐哪种方案。

```py
 >>> with open('test.log', encoding='utf-8') as a_file:
...     print(a_file.read())                              
test succeeded

...     a_file.write('and again')
>>> with open('test.log', encoding='utf-8') as a_file:
...     print(a_file.read()) 
```

1.  大胆地创建新文件`test.log`（或者重写已经存在的文件），然后以写入方式打开文件。参数`mode='w'`的意思是文件以写入的模式打开。是的，这听起来似乎比较危险。我希望你确定不再关心那个文件以前的内容（如果有的话），因为那份数据已经没了。
2.  你可以通过`open()`函数返回的流对象的`write()`方法来给新打开的文件添加数据。当`with`块结束的时候，Python 自动关闭文件。
3.  多么有趣，我们再试一次。这一次，使用`with='a'`参数来添加数据到文件末尾，而不是重写它。追加模式绝不会破坏现有文件的内容。
4.  原来写入的行，还有追加上去的第二行现在都在文件`test.log`里了。同时请注意，回车符没有被包括进去。你可以通过`'\n'`写入一个回车符。由于一开始没有这样做，所有写入到文件的数据现在都在同一行。

### 再次讨论字符编码

你是否注意到当你在打开文件用于写入数据的时候传递给`open()`函数的`encoding`参数。它“非常重要”，不要忽略了！就如你在这章开头看到的，文件中并不存在*字符串*，它们由*字节*组成。只有当你告诉 Python 使用何种编码方式把字节流转换为字符串，从文件读取“字符串”才成为可能。相反地，写入文本到文件面临同样的问题。实际上你不能直接把字符写入到文件；字符只是一种抽象。为了写入字符到文件，Python 需要知道如何将字符串转换为字节序列。唯一能保证正确地执行转换的方法就是当你为写入而打开一个文件的时候，指定`encoding`参数。

## 二进制文件

![my dog Beauregard](img/beauregard.jpg)

不是所有的文件都包含文本内容。有一些还包含了我可爱的狗的照片。

```py
 'rb'

'examples/beauregard.jpg'

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: '_io.BufferedReader' object has no attribute 'encoding' 
```

1.  用二进制模式打开文件很简单，但是很精细。与文本模式唯一不同的是`mode`参数包含一个字符`'b'`。
2.  以二进制模式打开文件得到的流对象与之前的有很多相同的属性，包括`mode`属性，它记录了你调用`open()`函数时指定的`mode`参数的值。
3.  二进制文件的流对象也有`name`属性，就如文本文件的流对象一样。
4.  然而，确实有不同之处：二进制的流对象没有`encoding`属性。你能明白其中的道理的，对吧？现在你读写的是字节，而不是字符串，所以 Python 不需要做转换工作。从二进制文件里读出的跟你所写入的是完全一样的，所以没有执行转换的必要。

我是否提到当前正在读取字节？噢，的确如此。

```py
# continued from the previous example
# 继续前一样例
>>> an_image.tell()
0

>>> data
b'\xff\xd8\xff'

<class 'bytes'>

3
>>> an_image.seek(0)
0
>>> data = an_image.read()
>>> len(data)
3150 
```

1.  跟读取文本文件一样，你也可以从二进制文件一次读一点儿。但是它们之间有一个重大的不同之处处&#hellip;
2.  >&#hellip;你正在读取字节，而不是字符串。由于你以二进制模式打开文件，`read()`方法每次*读取指定的字节数*，而非字符数。
3.  这就意味着，你传递给`read()`方法的数目和你从`tell()`方法得到的位置序号不会出现意料之外的不匹配(unexpected mismatch)

## 非文件来源的流对象

使用`read()`方法即可从虚拟文件读取数据。

想象一下你正在编写一个库(library)，其中有一库函数用来从文件读取数据。它使用文件名作为参数，以只读的方式打开文件，读取数据，关闭文件，返回。但是你不应该只做到这个程度。你的 API 应该能够接纳*任意的类型的流对象*。

最简单的情况，只要对象包含`read()`方法，这个方法使用一个可选参数`size`并且返回值为一个串，它就是是流对象。不使用`size`参数调用`read()`的时候，这个方法应该从输入源读取所有可读的信息然后以单独的一个值返回所有数据。当使用`size`参数调用`read()`时，它从输入源读取并返回指定量的数据。当再一次被调用时，它从上一次离开的地方开始读取并返回下一个数据块。

这听起来跟你从打开一个真实文件得到的流对象一样。不同之处在于*你不再受限于真实的文件*。能够“读取”的输入源可以是任何东西：网页，内存中的字符串，甚至是另外一个程序的输出。只要你的函数使用的是流对象，调用对象的`read()`方法，你可以处理任何行为与文件类似的输入源，而不需要为每种类型的输入指定特别的代码。

```py
>>> a_string = 'PapayaWhip is the new black.'

'PapayaWhip is the new black.'

''

0

'PapayaWhip'
>>> a_file.tell()                       
10
>>> a_file.seek(18)
18
>>> a_file.read()
'new black.' 
```

1.  `io`模块定义了`StringIO`类，你可以使用它来把内存中的字符串当作文件来处理。
2.  为了从字符串创建一个流对象，可以把想要作为“文件”使用的字符串传递给`io.StringIO()`来创建一个`StringIO`的实例。
3.  调用`read()`方法“读取”整个“文件”，以`StringIO`对象为例即返回原字符串。
4.  就像一个真实的文件一样，再次调用`read()`方法返回一个空串。
5.  通过使用`StringIO`对象的`seek()`方法，你可以显式地定位到字符串的开头，就像在一个真实的文件中定位一样。
6.  通过传递`size`参数给`read()`方法，你也可以以数据块的形式读取字符串。

> ☞`io.StringIO`让你能够将一个字符串作为文本文件来看待。另外还有一个`io.ByteIO`类，它允许你将字节数组当做二进制文件来处理。

### 处理压缩文件

Python 标准库包含支持读写压缩文件的模块。有许多种不同的压缩方案；其中，[gzip](http://docs.python.org/3.1/library/gzip.html)和[bzip2](http://docs.python.org/3.1/library/bz2.html)是非 Windows 操作系统下最流行的两种压缩方式。

`gzip`模块允许你创建用来读写 gzip 压缩文件的流对象。该流对象支持`read()`方法（如果你以读取模式打开）或者`write()`方法（如果你以写入模式打开）。这就意味着，你可以使用从普通文件那儿学到的技术来*直接读写 gzip 压缩文件*，而不需要创建临时文件来保存解压缩了的数据。

作为额外的功能，它也支持`with`语句，所以当你完成了对 gzip 压缩文件的操作，Python 可以为你自动关闭它。

```py
you@localhost:~$ python3

>>> import gzip

...   z_file.write('A nine mile walk is no joke, especially in the rain.'.encode('utf-8'))
... 
>>> exit()

-rw-r--r--  1 mark mark    79 2009-07-19 14:29 out.log.gz

A nine mile walk is no joke, especially in the rain. 
```

1.  你应该问题以二进制模式打开 gzip 压缩文件。（注意`mode`参数里的`'b'`字符。）
2.  我在 Linux 系统上完成的这个例子。如果你对命令行不熟悉，这条命令用来显示刚才你在 Python shell 创建的 gzip 压缩文件的“长清单(long listings)”，你可以看到，它有 79 个字节长。而实际上这个值比一开始的字符串还要长！由于 gzip 文件包括了一个固定长度的文件头来存放一些关于文件的元数据(metadata)，所以它对于极小的文件来说效率不高。
3.  `gunzip`命令（发音：“gee-unzip”）解压缩文件然后保存其内容到一个与原来压缩文件同名的新文件中，并去掉其`.gz`扩展名。
4.  `cat`命令显示文件的内容。当前文件包含了原来你从 Python shell 直接写入到压缩文件`out.log.gz`的那个字符串。

## 标准输入、输出和错误

`sys.stdin`, `sys.stdout`, `sys.stderr`.

命令行高手已经对标准输入，标准输出和标准错误的概念相当熟悉了。这部分内容是对另一部分还不熟悉的人员准备的。

标准输出和标准错误（通常缩写为`stdout`和`stderr`）是被集成到每一个类 UNIX 操作系统中的两个管道(pipe)，包括 Mac OS X 和 Linux。当你调用`print()`的时候，需要打印的内容即被发送到`stdout`管道。当你的程序出错并且需要打印跟踪信息(traceback)时，它们被发送到`stderr`管道。默认地，这两个管道都被连接到你正在工作的终端窗口上(terminal window)；当你的程序打印某些东西，你可以在终端上看到这些输出，当程序出错，你也可以从终端上看到这些错误信息。在图形化的 Python shell 里，`stdout`和`stderr`管道默认连接到“交互式窗口(Interactive Window)”

```py
>>> for i in range(3):

PapayaWhip
PapayaWhip
PapayaWhip
>>> import sys
>>> for i in range(3):

is theis theis the
>>> for i in range(3):

new blacknew blacknew black 
```

1.  循环调用`print()`函数。没有什么特别的。
2.  `stdout`被定义在`sys`模块里，它是一个流对象(stream object)。使用任意字符串调用其`write()`函数会按原样输出。事实上，这就是`print()`函数实际在做的事情；它在串的结尾添加一个回车符，然后调用`sys.stdout.write`。
3.  最简单的情况下，`sys.stdout`和`sys.stderr`把他们的输出发送到同一个位置：Python IDE（如果你在那里执行操作），或者终端（如果你从命令行执行 Python 指令）。跟标准输出一样，标准错误也不会自动为你添加回车符。如果你需要回车符，你需要手工写入回车符到标准错误。

`sys.stdout`和`sys.stderr`都是流对象，但是他们都只支持写入。试图调用他们的`read()`方法会引发`IOError`异常。

```py
>>> import sys
>>> sys.stdout.read()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IOError: not readable 
```

### 标准输出重定向

`sys.stdout`和`sys.stderr`都是流对象，尽管他们只支持写入。但是他们是变量而不是常量。这就意味着你可以给它们赋上新值 — 任意其他流对象 — 来重定向他们的输出。

```py
import sys

class RedirectStdoutTo:
    def __init__(self, out_new):
        self.out_new = out_new

    def __enter__(self):
        self.out_old = sys.stdout
        sys.stdout = self.out_new

    def __exit__(self, *args):
        sys.stdout = self.out_old

print('A')
with open('out.log', mode='w', encoding='utf-8') as a_file, RedirectStdoutTo(a_file):
    print('B')
print('C') 
```

验证一下：

```py
you@localhost:~/diveintopython3/examples$ python3 stdout.py
A
C
you@localhost:~/diveintopython3/examples$ cat out.log
B 
```

> 你是否遇到了以下错误？
> 
> ```py
> you@localhost:~/diveintopython3/examples$ python3 stdout.py
>  File "stdout.py", line 15
>     with open('out.log', mode='w', encoding='utf-8') as a_file, RedirectStdoutTo(a_file):
>                                                               ^
> SyntaxError: invalid syntax 
> ```
> 
> 如果是这样，你可能正在使用 Python 3.0。应该升级到 Python 3.1。
> 
> Python 3.0 支持`with`语句，但是每个语句只能使用一个上下文管理器。Python 3.1 允许你在一条`with`语句中链接多个上下文件管理器。

我们先来处理最后那一部分。

```py
print('A')
with open('out.log', mode='w', encoding='utf-8') as a_file, RedirectStdoutTo(a_file):
    print('B')
print('C') 
```

这是一个复杂的`with`语句。让我改写它使之更有可读性。

```py
with open('out.log', mode='w', encoding='utf-8') as a_file:
    with RedirectStdoutTo(a_file):
        print('B') 
```

正如改动后的代码所展示的，实际上你使用了*两个*`with`语句，其中一个嵌套在另外一个的作用域(scope)里。“外层的”`with`语句你应该已经熟悉了：它打开一个使用 UTF-8 编码的叫做`out.log`的文本文件用来写入，然后把返回的流对象赋给一个叫做`a_file`的变量。但是，在此处，它并不是唯一显得古怪的事情。

```py
with RedirectStdoutTo(a_file): 
```

`as`子句(clause)到哪里去了？其实`with`语句并不一定需要`as`子句。就像你调用一个函数然后忽略其返回值一样，你也可以不把`with`语句的上下文环境赋给一个变量。在这种情况下，我们只关心`RedirectStdoutTo`上下文环境的边际效应(side effect)。``

``那么，这些边际效应都是些什么呢？我们来看一看`RedirectStdoutTo`类的内部结构。这是一个用户自定义的上下文管理器(context manager)。任何类只要定义了两个特殊方法：code&gt;__enter__()`和`__exit__()`就可以变成上下文管理器。`

```py
`class RedirectStdoutTo:

        self.out_new = out_new

        self.out_old = sys.stdout
        sys.stdout = self.out_new

        sys.stdout = self.out_old 
```

1.  在实例被创建后`__init__()`方法马上被调用。它使用一个参数，即在上下文环境的生命周期内你想用做标准输出的流对象。这个方法只是把该流对象保存在一个实例变量里(instance variable)以使其他方法在后边能够使用到它。
2.  `__enter__()`方法是一个特殊的类方法(special class method)；在进入一个上下文环境时 Python 会调用它（*即*，在`with`语句的开始处）。该方法把当前`sys.stdout`的值保存在`self.out_old`内，然后通过把`self.out_new`赋给`sys.stdout`来重定向标准输出。
3.  `__exit__()`是另外一个特殊类方法；当离开一个上下文环境时（*即*，在`with`语句的末尾）Python 会调用它。这个方法通过把保存的`self.out_old`的值赋给`sys.stdout`来恢复标准输出到原来的状态。

放到一起：

1.  这条代码会输出到 IDE 的“交互式窗口(Interactive Window)”（或者终端，如果你从命令行运行这段脚本）。
2.  这条`with`语句使用*逗号分隔的上下文环境列表*。这个列表就像一系列相互嵌套的`with`块。先列出的是“外层”的块；后列出的是“内层”的块。第一个上下文环境打开一个文件；第二个重定向`sys.stdout`到由第一个上下环境创建的流对象。
3.  由于这个`print()`函数在`with`语句创建的上下文环境里执行，所以它不会输出到屏幕；它会写入到文件`out.log`。
4.  `with`语句块结束了。Python 告诉每一个上下文管理器完成他们应该在离开上下文环境时应该做的事。这些上下文环境形成一个后进先出的栈。当离开一个上下文环境的时候，第二个上下文环境将`sys.stdout`的值恢复到它的原来状态，然后第一个上下文环境关闭那个叫做`out.log`的文件。由于标准输出已经被恢复到原来的状态，再次调用`print()`函数会马上输出到屏幕上。

重定向标准错误的原理跟这个完全一样，将`sys.stdout`替换为`sys.stderr`即可。

## 进一步阅读

*   [读写文件](http://docs.python.org/tutorial/inputoutput.html#reading-and-writing-files) Python.org 上的教程
*   [`io` 模块](http://docs.python.org/3.1/library/io.html)
*   [流对象](http://docs.python.org/3.1/library/stdtypes.html#file-objects)
*   [上下文管理器类型](http://docs.python.org/3.1/library/stdtypes.html#context-manager-types)
*   [`sys.stdout` and `sys.stderr`](http://docs.python.org/3.1/library/sys.html#sys.stdout)
*   [FUSE 来自维基百科](http://en.wikipedia.org/wiki/Filesystem_in_Userspace)