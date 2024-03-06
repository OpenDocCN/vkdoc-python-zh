# Chapter 3 解析

> " Our imagination is stretched to the utmost, not, as in fiction, to imagine things which are not really there, but just to comprehend those things which are. " — [Richard Feynman](http://en.wikiquote.org/wiki/Richard_Feynman)

## 深入

这一章节将围绕一个非常强大的技术向你介绍列表解析，字典解析和集合解析这三个概念。但是，我要先打个岔介绍两个帮助你浏览本地文件系统的模块。

## 处理文件和目录

Python 3 带有一个模块叫做 `os`，代表 “操作系统(operating system)。” [`os` 模块](http://docs.python.org/3.1/library/os.html) 包含非常多的函数用于获取(和修改)本地目录、文件进程、环境变量等的信息。Python 尽最大的努力在所有支持的操作系统上提供一个统一的 API， 这样你就可以在保证程序能够在任何的计算机上运行的同时尽量少的包含平台特定的代码。

### 当前工作目录

当你刚刚开始学习 Python 的时候， 你将花大量的时间在 Python Shell 上。 在整本书中，你将一直看见类似下面的例子:

1.  在`examples` 目录导入某一个模块
2.  调用模块的某一个函数
3.  解释输出结果

总是有一个当前工作目录

如果你不知道当前工作目录， 第一步很可能会得到一个`ImportError`。 为什么? 因为 Python 将在导入搜索路径中查找示例模块， 但是由于`examples` 目录没有包含在搜索路径中，查找将失败。 你可以通过下面两个方法之一来解决这个问题:

1.  将`examples`目录加入到导入搜索路径中
2.  将当前工作目录切换到`examples`目录

Python 在任何时候都在暗地里记住了当前工作目录这个属性。无论你是在 Python Shell 中，还是在命令行运行你自己的 Python 脚本，抑或是在 Web 服务器上运行 Python CGI 脚本，当前工作目录总是存在。

`os` 模块提供了两个函数处理当前工作目录

```py
 C:\Python31

C:\Users\pilgrim\diveintopython3\examples 
```

1.  `os` 是 Python 自带的; 你可以在任何时间，任何地方导入它。
2.  使用`os.getcwd()` 函数获得当前工作目录。当你运行一个图形化的 Python Shell 时，当前工作目录默认将是 Python Shell 的可执行文件所在的目录。在 Windows 上， 这个目录取决于你将 Python 安装在哪里; 默认位置是 `c:\Python31`。如果你通过命令行运行 Python Shell，当前工作目录是你运行`python3`时所在的目录。
3.  使用`os.chdir()`函数改变当前工作目录
4.  运行`os.chdir()`函数时，即使在 Windows 上，我也总是使用 Linux 风格的路径(正斜杠，没有盘符)。这就是 Python 尝试隐藏操作系统差异的一个地方。

### 处理文件名和目录名

既然我们说到了目录，我得指出 `os.path` 模块。`os.path` 模块包含了操作文件名和目录名的函数.

```py
>>> import os

/Users/pilgrim/diveintopython3/examples/humansize.py

/Users/pilgrim/diveintopython3/examples\humansize.py

c:\Users\pilgrim

c:\Users\pilgrim\diveintopython3\examples\humansize.py 
```

1.  `os.path.join()` 函数从一个或多个路径片段中构造一个路径名。 在这个例子中， 它仅仅是简单的拼接字符串.
2.  这个例子稍微复杂一点， 在和文件名拼接前，`join`函数给路径名添加一个额外的斜杠。由于我在 Windows 上写这个例子， 这个斜杠是一个反斜杠而不是正斜杠。如果你在 Linux 或者 Mac OS X 上重现这个例子， 你将会看见正斜杠. 无论你使用哪种形式的斜杠，Python 都可以访问到文件。
3.  `os.path.expanduser()` 用来将包含`~`符号（表示当前用户 Home 目录）的路径扩展为完整的路径。在任何有 Home 目录概念的操作系统上(包括 Linux，Mac OS X 和 Windows)，这个函数都能工作。返回的路径不以斜杠结尾，但是`os.path.join()`并不介意这一点。
4.  结合这些技术，你可以很方便的构造出用户 Home 目录下的文件和目录的路径。 `os.path.join()`可以接受任何数量的参数。当我发现这一点时我大喜过望， 因为在一门新的语言中构造我的工具箱时，`addSlashIfNecessary()`总是我不得不写的愚蠢的小函数之一。*不要* 在 Python 中写这个愚蠢的小函数，聪明的人们已经帮你考虑过这个问题了。

`os.path` 也包含用于分割完整路径名，目录名和文件名的函数

```py
>>> pathname = '/Users/pilgrim/diveintopython3/examples/humansize.py'

('/Users/pilgrim/diveintopython3/examples', 'humansize.py')

'/Users/pilgrim/diveintopython3/examples'

'humansize.py'

>>> shortname
'humansize'
>>> extension
'.py' 
```

1.  `split` 函数分割一个完整路径并返回目录和文件名。
2.  还记得我说过在函数返回多个值时应该使用多变量赋值 吗 ? `os.path.split()` 函数正是这样做的。 将`split`函数的返回值赋值给一个二元组。每个变量获得了返回元组中的对应元素的值。
3.  第一个变量`dirname`，获得了`os.path.split()` 函数返回元组中的第一个元素，文件所在的目录。
4.  第二个变量`filename`，获得了`os.path.split()` 函数返回元组中的第二个元素，文件名。
5.  `os.path` 也包含`os.path.splitext()` 函数，它分割一个文件名并返回短文件名和扩展名。可以使用同样的技术将它们的值赋值给不同的变量。

### 罗列目录内容

`glob` 模块是 Python 标准库中的另一个工具，它可以通过编程的方法获得一个目录的内容，并且它使用熟悉的命令行下的通配符。

`glob` 模块使用 shell 风格的通配符。

```py
>>> os.chdir('/Users/pilgrim/diveintopython3/')
>>> import glob

['examples\\feed-broken.xml',
'examples\\feed-ns0.xml',
'examples\\feed.xml']

['alphameticstest.py',
'pluraltest1.py',
'pluraltest2.py',
'pluraltest3.py',
'pluraltest4.py',
'pluraltest5.py',
'pluraltest6.py',
'romantest1.py',
'romantest10.py',
'romantest2.py',
'romantest3.py',
'romantest4.py',
'romantest5.py',
'romantest6.py',
'romantest7.py',
'romantest8.py',
'romantest9.py'] 
```

1.  `glob` 模块接受一个通配符并返回所有匹配的文件和目录的路径。在这个例子中，通配符是一个目录名加上 “`*.xml`”， 它匹配`examples`子目录下的所有`.xml` 文件。
2.  现在我们将当前工作目录切换到`examples` 目录。 `os.chdir()` 可以接受相对路径.
3.  在 glob 模式中你可以使用多个通配符。这个例子在当前工作目录中找出所有扩展名为`.py`并且在文件名中包含单词`test` 的文件。

### 获取文件元信息

每一个现代文件系统都对文件存储了元信息: 创建时间，最后修改时间，文件大小等等。Python 单独提供了一个的 API 用于访问这些元信息。 你不需要打开文件。知道文件名就足够了。

```py
>>> import os

c:\Users\pilgrim\diveintopython3\examples

1247520344.9537716

time.struct_time(tm_year=2009, tm_mon=7, tm_mday=13, tm_hour=17,
tm_min=25, tm_sec=44, tm_wday=0, tm_yday=194, tm_isdst=1) 
```

1.  当前工作目录是`examples` 文件夹。
2.  `feed.xml`是`examples` 文件夹中的一个文件。 调用`os.stat()` 函数返回一个包含多种文件元信息的对象。
3.  `st_mtime` 是最后修改时间，它的格式不是很有用。(技术上讲，它是从纪元，也就是 1970 年 1 月 1 号的第一秒钟，到现在的秒数)
4.  `time` 模块是 Python 标准库的一部分。 它包含用于在不同时间格式中转换，将时间格式化成字符串以及处理时区的函数。
5.  `time.localtime()` 函数将从纪元到现在的秒数这个格式表示的时间(`os.stat()`函数返回值的`st_mtime` 属性)转换成更有用的包含年、月、日、小时、分钟、秒的结构体。这个文件的最后修改时间是 2009 年 7 月 13 日下午 5:25。

```py
# continued from the previous example

3070
>>> import humansize

'3.0 KiB' 
```

1.  `os.stat()` 函数也通过`st_size` 属性返回文件大小。文件`feed.xml` 的大小是 `3070` 字节。
2.  你可以将`st_size` 属性作为参数传给`approximate_size()` 函数。

### 构造绝对路径

在前一节中，`glob.glob()` 函数返回一个相对路径的列表。第一个例子的路径类似`'examples\feed.xml'`，而第二个例子的路径`'romantest1.py'`更短。只要你保持在当前工作目录中，你就可以使用这些相对路径来打开文件或者获得文件的元信息。但是当你希望构造一个从根目录开始或者是包含盘符的绝对路径时，你就需要用到`os.path.realpath()`函数了。

```py
>>> import os
>>> print(os.getcwd())
c:\Users\pilgrim\diveintopython3\examples
>>> print(os.path.realpath('feed.xml'))
c:\Users\pilgrim\diveintopython3\examples\feed.xml 
```

## 列表解析

你可以在列表解析中使用任何的 Python 表达式。

列表解析提供了一种紧凑的方式，实现了通过对列表中每一个元素应用一个函数的方法来将一个列表映射到另一个列表.

```py
>>> a_list = [1, 9, 8, 4]

[2, 18, 16, 8]

[1, 9, 8, 4]

>>> a_list
[2, 18, 16, 8] 
```

1.  为了理解这一点，请从右向左看。 `a_list`是你要映射的列表。Python 解释器逐个访问`a_list`的元素，并临时将元素赋值给变量`elem`。 然后 Python 对元素应用函数``elem` * 2`并且将结果添加到返回列表中。
2.  列表解析创造一个新的列表而不改变原列表。
3.  可以安全的将列表解析的结果赋值给被映射的变量。Python 会在内存中构造新的列表，在列表解析完成后将结果赋值给原来的变量。

你可以在列表解析中使用任何的 Python 表达式， 包括`os` 模块中用于操作文件和目录的函数。

```py
>>> import os, glob

['feed-broken.xml', 'feed-ns0.xml', 'feed.xml']

['c:\\Users\\pilgrim\\diveintopython3\\examples\\feed-broken.xml',
'c:\\Users\\pilgrim\\diveintopython3\\examples\\feed-ns0.xml',
'c:\\Users\\pilgrim\\diveintopython3\\examples\\feed.xml'] 
```

1.  这里返回当前目录下的所有`.xml` 文件。
2.  列表解析接受`.xml` 文件列表并将其转化成全路径的列表。

列表解析也可以过滤列表，生成比原列表短的结果列表。

```py
>>> import os, glob

['pluraltest6.py',
'romantest10.py',
'romantest6.py',
'romantest7.py',
'romantest8.py',
'romantest9.py'] 
```

1.  你可以在列表解析的最后加入`if`子句来过滤列表。对于列表中每一个元素`if` 关键字后面的表达式都会被计算。如果表达式的计算结果为`True`，那么这个元素将会被包含在输出中。这个列表解析在当前目录查找所有`.py` 文件，而 `if` 表达式通过测试文件大小是否大于`6000`字节对列表进行过滤。有 6 个符合条件的文件，所以这个列表解析返回包含六个文件名的列表。

到目前为止的例子中的列表解析都只是用了一些简单的表达式， 乘以一个常数、调用一个函数或者是在过滤后返回原始元素。 然而列表解析并不限制表达式的复杂程度。

```py
>>> import os, glob

[(3074, 'c:\\Users\\pilgrim\\diveintopython3\\examples\\feed-broken.xml'),
(3386, 'c:\\Users\\pilgrim\\diveintopython3\\examples\\feed-ns0.xml'),
(3070, 'c:\\Users\\pilgrim\\diveintopython3\\examples\\feed.xml')]
>>> import humansize

[('3.0 KiB', 'feed-broken.xml'),
('3.3 KiB', 'feed-ns0.xml'),
('3.0 KiB', 'feed.xml')] 
```

1.  这个列表解析找到当前工作目录下的所有`.xml`文件， 对于每一个文件构造一个包含文件大小(通过调用`os.stat()`获得)和绝对路径(通过调用`os.path.realpath()`)的元组。
2.  这个列表解析在前一个的基础上对每一个`.xml`文件的大小应用`approximate_size()`函数。

## 字典解析

字典解析和列表解析类似，只不过它生成字典而不是列表。

```py
>>> import os, glob

('alphameticstest.py', nt.stat_result(st_mode=33206, st_ino=0, st_dev=0,
st_nlink=0, st_uid=0, st_gid=0, st_size=2509, st_atime=1247520344,
st_mtime=1247520344, st_ctime=1247520344))

<class 'dict'>

['romantest8.py', 'pluraltest1.py', 'pluraltest2.py', 'pluraltest5.py',
'pluraltest6.py', 'romantest7.py', 'romantest10.py', 'romantest4.py',
'romantest9.py', 'pluraltest3.py', 'romantest1.py', 'romantest2.py',
'romantest3.py', 'romantest5.py', 'romantest6.py', 'alphameticstest.py',
'pluraltest4.py']

2509 
```

1.  这不是字典解析; 而是列表解析。它找到所有名称中包含`test`的`.py`文件，然后构造包含文件名和文件元信息(通过调用`os.stat()`函数得到)的元组。
2.  结果列表的每一个元素是元组。
3.  这是一个字典解析。 除了两点以外，它的语法同列表解析很类似。首先，它被花括号而不是方括号包围; 第二，对于每一个元素它包含由冒号分隔的两个表达式，而不是列表解析的一个。冒号前的表达式(在这个例子中是`f`)是字典的键;冒号后面的表达式(在这个例子中是`os.stat(f)`)是值。
4.  字典解析返回结果是字典。
5.  这个字典的键很简单，就是`glob.glob('*test*.py')`调用返回的文件名。
6.  每一个键对应的值是`os.stat()`函数的返回值。这意味着我们可以在字典中通过文件名查找到它的文件元信息。元信息的一个部分是文件大小`st_size`。这个文件`alphameticstest.py` 的大小是`2509`字节。

同列表解析一样，你可以在字典解析中包含`if`字句来过滤输入序列，对于每一个元素字句中的表达式都会被求值。

```py
>>> import os, glob, humansize

['romantest9', 'romantest8', 'romantest7', 'romantest6', 'romantest10', 'pluraltest6']

'6.5 KiB' 
```

1.  这个字典解析获得当前目录下所有的文件的列表(`glob.glob('*')`)，通过`os.stat(f)`获得每一个文件的元信息， 然后构造一个键是文件名，值是文件元信息的字典。
2.  这个字典解析在前一个基础上过滤掉文件小于`6000`字节的文件(`if meta.st_size &gt; 6000`)， 并用过滤出的列表构造字典， 字典的键是文件名去掉扩展名的部分(`os.path.splitext(f)[0]`) ，字典的值是每个文件的人类可读的近似大小(`humansize.approximate_size(meta.st_size)`)。
3.  正如你在前一个例子中所看见的，有 6 个这样的文件，所以字典中有 6 个元素。
4.  每一个键对应的值是`approximate_size()`函数返回的字符串。

### 其他同字典解析有关的小技巧

这里是一个可能有用的通过字典解析实现的小技巧: 交换字典的键和值。

```py
>>> a_dict = {'a': 1, 'b': 2, 'c': 3}
>>> {value:key for key, value in a_dict.items()}
{1: 'a', 2: 'b', 3: 'c'} 
```

## 集合解析

同样,集合也有自己的集合解析的语法。它和字典解析的非常相似，唯一的不同是集合只有值而没有键:值对。

```py
>>> a_set = set(range(10))
>>> a_set
{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}

{0, 1, 4, 81, 64, 9, 16, 49, 25, 36}

{0, 8, 2, 4, 6}

{32, 1, 2, 4, 8, 64, 128, 256, 16, 512} 
```

1.  集合解析可以接受一个集合作为参数。这个集合解析计算数字 0-`9`这个集合的的平方。
2.  同列表解析和字典解析一样， 集合解析也可以包含`if` 字句来在将元素放入结果集合前进行过滤。
3.  集合解析的输入并不一定要是集合; 可以是任何序列。

## 进一步阅读

*   [`os` module](http://docs.python.org/3.1/library/os.html)
*   [`os` — Portable access to operating system specific features](http://www.doughellmann.com/PyMOTW/os/)
*   [`os.path` module](http://docs.python.org/3.1/library/os.path.html)
*   [`os.path` — Platform-independent manipulation of file names](http://www.doughellmann.com/PyMOTW/ospath/)
*   [`glob` module](http://docs.python.org/3.1/library/glob.html)
*   [`glob` — Filename pattern matching](http://www.doughellmann.com/PyMOTW/glob/)
*   [`time` module](http://docs.python.org/3.1/library/time.html)
*   [`time` — Functions for manipulating clock time](http://www.doughellmann.com/PyMOTW/time/)
*   [List comprehensions](http://docs.python.org/3.1/tutorial/datastructures.html#list-comprehensions)
*   [Nested list comprehensions](http://docs.python.org/3.1/tutorial/datastructures.html#nested-list-comprehensions)
*   [Looping techniques](http://docs.python.org/3.1/tutorial/datastructures.html#looping-techniques)