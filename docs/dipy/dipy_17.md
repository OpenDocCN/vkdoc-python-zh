# 第十六章 函数编程

# 第十六章 函数编程

*   16.1\. 概览
*   16.2\. 找到路径
*   16.3\. 重识列表过滤
*   16.4\. 重识列表映射
*   16.5\. 数据中心思想编程
*   16.6\. 动态导入模块
*   16.7\. 全部放在一起
*   16.8\. 小结

# 16.1\. 概览

# 16.1\. 概览

在 第十三章 *单元测试* 中，你学会了单元测试的哲学。在 第十四章 *测试优先编程* 中你步入了 Python 基本的单元测试操作，在 第十五章 *重构* 部分，你看到单元测试如何令大规模重构变得容易。本章将在这些程序样例的基础上，集中关注于超越单元测试本身的更高级的 Python 特有技术。

下面是一个作为简单回归测试 (regression test) 框架运行的完整 Python 程序。它将你前面编写的单独单元测试模块组织在一起成为一个测试套件并一次性运行。实际上这是本书的构建代码的一部分；我为几个样例程序都编写了单元测试 (不是只有 第十三章 *单元测试* 中的 `roman.py` 模块)，我的自动构建代码的第一个工作便是确保我所有的例子可以正常工作。如果回归测试程序失败，构建过程当即终止。我可不想因为发布了不能工作的样例程序而让你在下载他们后坐在显示器前抓耳挠腮地为程序不能运转而烦恼。

## 例 16.1\. `regression.py`

如果您还没有下载本书附带的样例程序, 可以 [下载本程序和其他样例程序](http://www.woodpecker.org.cn/diveintopython/download/diveintopython-exampleszh-cn-5.4b.zip "Download example scripts")。

```py
"""Regression testing framework
This module will search for scripts in the same directory named
XYZtest.py.  Each such script should be a test suite that tests a
module through PyUnit.  (As of Python 2.1, PyUnit is included in
the standard library as "unittest".)  This script will aggregate all
found test suites into one big test suite and run them all at once.
"""
import sys, os, re, unittest
def regressionTest():
    path = os.path.abspath(os.path.dirname(sys.argv[0]))   
    files = os.listdir(path)                               
    test = re.compile("test\.py$", re.IGNORECASE)          
    files = filter(test.search, files)                     
    filenameToModuleName = lambda f: os.path.splitext(f)[0]
    moduleNames = map(filenameToModuleName, files)         
    modules = map(__import__, moduleNames)                 
    load = unittest.defaultTestLoader.loadTestsFromModule  
    return unittest.TestSuite(map(load, modules))          
if __name__ == "__main__":                   
    unittest.main(defaultTest="regressionTest") 
```

把这段代码放在本书其他样例代码相同的目录下运行之，`_`module`_test.py` 中的所有单元测试将被找到并一起被运行。

## 例 16.2\. `regression.py` 的样例输出

```py
[you@localhost py]$ python regression.py -v
help should fail with no object ... ok  help should return known result for apihelper ... ok
help should honor collapse argument ... ok
help should honor spacing argument ... ok
buildConnectionString should fail with list input ... ok  buildConnectionString should fail with string input ... ok
buildConnectionString should fail with tuple input ... ok
buildConnectionString handles empty dictionary ... ok
buildConnectionString returns known result with known input ... ok
fromRoman should only accept uppercase input ... ok  toRoman should always return uppercase ... ok
fromRoman should fail with blank string ... ok
fromRoman should fail with malformed antecedents ... ok
fromRoman should fail with repeated pairs of numerals ... ok
fromRoman should fail with too many repeated numerals ... ok
fromRoman should give known result with known input ... ok
toRoman should give known result with known input ... ok
fromRoman(toRoman(n))==n for all n ... ok
toRoman should fail with non-integer input ... ok
toRoman should fail with negative input ... ok
toRoman should fail with large input ... ok
toRoman should fail with 0 input ... ok
kgp a ref test ... ok
kgp b ref test ... ok
kgp c ref test ... ok
kgp d ref test ... ok
kgp e ref test ... ok
kgp f ref test ... ok
kgp g ref test ... ok
----------------------------------------------------------------------
Ran 29 tests in 2.799s
OK 
```

|  |  |
| --- | --- |
| [1] | 前五个测试来自于 `apihelpertest.py`，用以测试 第四章 *自省的威力* 中的样例代码。 |
| [2] | 接下来的五个测试来自于 `odbchelpertest.py`，用以测试 第二章 *第一个 Python 程序* 中的样例代码。 |
| [3] | 其他的测试来自于 `romantest.py`，你在 第十三章 *单元测试* 中深入学习过。 |

# 16.2\. 找到路径

# 16.2\. 找到路径

从命令行运行 Python 代码时，知道所运行代码在磁盘上的存储位置有时候是有必要的。

这是一个不那么容易想起，但一想起就很容易解决的小麻烦。答案是 `sys.argv`。正如你在 第九章 *XML 处理* 中看到的，它包含了很多命令行参数。它也同样记录了运行脚本的名字，和你调用它时使用的命令一摸一样。这些信息足以令我们确定文件的位置。

## 例 16.3\. `fullpath.py`

如果您还没有下载本书附带的样例程序, 可以 [下载本程序和其他样例程序](http://www.woodpecker.org.cn/diveintopython/download/diveintopython-exampleszh-cn-5.4b.zip "Download example scripts")。

```py
 import sys, os
print 'sys.argv[0] =', sys.argv[0]             
pathname = os.path.dirname(sys.argv[0])         print 'path =', pathname
print 'full path =', os.path.abspath(pathname) 
```

|  |  |
| --- | --- |
| [1] | 无论如何运行一段脚本，`sys.argv[0]` 总是包含脚本的名字，和调用时使用的命令一摸一样。你很快会发现，它不一定包含任何路径信息。 |
| [2] | `os.path.dirname` 接受作为字符串传来的文件名并返回路径部分。如果给定的文件名不包含任何路径信息，`os.path.dirname` 返回空字符串。 |
| [3] | `os.path.abspath` 是这里的关键。它接受的路径名可以是部分的甚至是完全空白，但总能返回完整有效的路径名。 |

进一步地解释 `os.path.abspath` 是有必要的。它非常灵活，可以接受任何类型的路径名。

## 例 16.4\. `os.path.abspath` 的进一步解释

```py
>>> import os
>>> os.getcwd()                        
/home/you
>>> os.path.abspath('')                
/home/you
>>> os.path.abspath('.ssh')            
/home/you/.ssh
>>> os.path.abspath('/home/you/.ssh') 
/home/you/.ssh
>>> os.path.abspath('.ssh/../foo/')    
/home/you/foo 
```

|  |  |
| --- | --- |
| [1] | `os.getcwd()` 返回当前的工作路径。 |
| [2] | 用空字符串调用 `os.path.abspath` 将返回当前的工作路径，与 `os.getcwd()`的效果相同。 |
| [3] | 以不完整的路径名调用 `os.path.abspath` 可以构建一个基于当前工作路径且完整有效的路径名。 |
| [4] | 以完整的路径名调用 `os.path.abspath` 则简单地将其直接返回。 |
| [5] | `os.path.abspath` 还*格式化* 返回的路径名。注意这个例子在我根本没有‘foo’目录时同样奏效。`os.path.abspath` 从不检查你的磁盘，而仅仅是字符串操作。 |

> 注意
> 传递给 `os.path.abspath` 的路径名和文件名可以不存在。
> 
> 注意
> `os.path.abspath` 不仅构建完整路径名，还能格式化路径名。这意味着如果你正工作于 `/usr/` 目录，`os.path.abspath('bin/../local/bin')` 将会返回 `/usr/local/bin`。它把路径名格式化为尽可能简单的形式。如果你只是希望简单地返回这样的格式化路径名而不需要完整路径名，可以使用 `os.path.normpath`。

## 例 16.5\. `fullpath.py` 的样例输出

```py
[you@localhost py]$ python /home/you/diveintopython/common/py/fullpath.py 
sys.argv[0] = /home/you/diveintopython/common/py/fullpath.py
path = /home/you/diveintopython/common/py
full path = /home/you/diveintopython/common/py
[you@localhost diveintopython]$ python common/py/fullpath.py               
sys.argv[0] = common/py/fullpath.py
path = common/py
full path = /home/you/diveintopython/common/py
[you@localhost diveintopython]$ cd common/py
[you@localhost py]$ python fullpath.py                                     
sys.argv[0] = fullpath.py
path = 
full path = /home/you/diveintopython/common/py 
```

|  |  |
| --- | --- |
| [1] | 在第一种情况下，`sys.argv[0]` 包含代码的完整路径。你可以通过 `os.path.dirname` 函数将文件名从其中剥离出来并返回完整的路径，`os.path.abspath` 则是简单地把你传递给它的值返回。 |
| [2] | 如果脚本是以不完整路名被运行的，`sys.argv[0]` 还是会包含命令行中出现的一切。`os.path.dirname` 将会给你一个 (相对于当前工作路径的) 不完整的路径名，`os.path.abspath` 将会以不完整路径名为基础构建一个完整的路径名。 |
| [3] | 如果没有给定任何路径，而是从当前目录运行脚本，`os.path.dirname` 将简单地返回一个空字符串。由于是从当前目录运行脚本，`os.path.abspath` 将针对给定的空字符串给出你所希望获知的当前目录。 |

> 注意
> 就像 `os` 和 `os.path` 模块的其他函数，`os.path.abspath` 是跨平台的。如果你是在 Windows (使用反斜杠作为路径符号) 或 Mac OS (使用冒号) 上运行，它们同样工作，只是将获得与我稍有不同的结果。`os` 的所有函数都是这样的。

**补充.** 一位读者对这个结果并不满意，他希望能够从当前路径运行所有单元测试，而不是从 `regression.py` 所在目录运行。他建议以下面的代码加以取代：

## 例 16.6\. 在当前目录运行脚本

```py
import sys, os, re, unittest
def regressionTest():
    path = os.getcwd()       
    sys.path.append(path)    
    files = os.listdir(path) 
```

|  |  |
| --- | --- |
| [1] | 不是将 `path` 设置为运行代码所在的路径，而是将它设置为当前目录。可以是你在运行脚本之前所在的任何路径，而不需要是运行脚本所在的路径。(多次体味这句话，直到你真正理解了它。) |
| [2] | 将这个目录添加到 Python 库搜索路径中，你稍后动态导入单元测试模块时，Python 就能找到它们了。如果 `path` 就是正在运行代码的存储目录，你就不需要这样做了，因为 Python 总会查找这个目录。 |
| [3] | 函数的其他部分不变。 |

这个技术允许你在多个项目中重用 `regression.py` 代码。只需要将这个代码放在一个普通目录中，在运行项目前将路径更改为项目的目录。项目中所有的单元测试被找到并运行，而不仅仅局限于 `regression.py` 所在目录的单元测试。

# 16.3\. 重识列表过滤

# 16.3\. 重识列表过滤

你已经熟识了应用列表解析来过滤列表。这里介绍的是达到相同效果的另一种令很多人感觉清晰的实现方法。

Python 有一个内建 `filter` 函数，它接受两个参数：一个函数和一个列表，返回一个列表。[12] 作为第一个参数传递给 `filter` 的函数本身应接受一个参数，`filter` 返回的列表将会包含被传入列表参数传递给 `filter` 所有可以令函数返回真 (true) 的元素。

都明白了吗？并没有听起来那么难。

## 例 16.7\. `filter` 介绍

```py
>>> def odd(n):                 
... return n % 2
... 
>>> li = [1, 2, 3, 5, 9, 10, 256, -3]
>>> filter(odd, li)             
[1, 3, 5, 9, -3]
>>> [e for e in li if odd(e)]   
>>> filteredList = []
>>> for n in li:                
... if odd(n):
...         filteredList.append(n)
... 
>>> filteredList
[1, 3, 5, 9, -3] 
```

|  |  |
| --- | --- |
| [1] | `odd` 使用内建的取模 (mod) 函数 “`%`” 对于为奇数的 `n` 返回 `1`；为偶数的返回 `0`。 |
| [2] | `filter` 接受两个参数：一个函数 (`odd`) 和一个列表 (`li`)。它依列表循环为每个元素调用 `odd` 函数。如果 `odd` 返回的是真 (记住，Python 认为所有非零值为真)，则该元素被放在返回列表中，如若不然则被过滤掉。结果是一个只包含原列表中奇数的列表，出现顺序则和原列表相同。 |
| [3] | 你可以通过遍历的方式完成相同的工作，正如在 第 4.5 节 “过滤列表” 中看到的。 |
| [4] | 你可以通过 `for` 循环的方式完成相同的工作。取决于你的编程背景，这样也许更“直接”，但是像 `filter` 函数这样的实现方法更清晰。不但编写简单，而且易于读懂。`for` 循环就好比近距离的绘画：你可以看到所有的细节，但是或许你应该花几秒时间退后几步看一看图画的全景：“啊，你仅仅是要过滤列表！” |

## 例 16.8\. `regression.py` 中的 `filter`

```py
 files = os.listdir(path)                                
    test = re.compile("test\.py$", re.IGNORECASE)           
    files = filter(test.search, files) 
```

|  |  |
| --- | --- |
| [1] | 正如你在 第 16.2 节 “找到路径” 中看到的，`path` 可能包括正在运行脚本的完全或者部分路径名，或者当脚本运行自当前目录时包含一个空的字符串。任何一种情况下，`files` 都会获得正运行脚本所在目录的文件名。 |
| [2] | 这是一个预编译的正则表达式。正如你在 第 15.3 节 “重构”中看到的，如果你需要反复使用同一个正则表达式，你应该编译它已获得更快的性能。编译后的对象将含有接受一个待寻找字符串作为参数的 `search` 方法。如果这个正则表达式匹配字符串，`search` 方法返回一个包含正则表达式匹配信息的 `Match` 对象；否则返回 `None`，这是 Python 空 (null) 值。 |
| [3] | 对于 `files` 列表中的每个元素，你将会调用正则表达式编译对象 `test` 的 `search` 方法。如果正则表达匹配，方法将会返回一个被 Python 认定为真 (true) 的 `Match` 对象；如果正则表达不匹配，`search` 方法将会返回被认定为假 (false) 的 `None`，元素将被排除。 |

**历史注释.** Python 2.0 早期的版本不包含 列表解析，因此不能 以列表解析方式过滤，`filter` 函数是当时唯一的方法。即便是在引入列表解析的 2.0 版，有些人仍然钟情于老派的 `filter` (和这章稍后将见到的它的伴侣函数 `map` )。两种方法并存于世，使用哪种方法只是风格问题，`map` 和 `filter` 将在未来的 Python 版本中被废止的讨论尚无定论。

## 例 16.9\. 以列表解析法过滤

```py
 files = os.listdir(path)                               
    test = re.compile("test\.py$", re.IGNORECASE)          
    files = [f for f in files if test.search(f)] 
```

|  |  |
| --- | --- |
| [1] | 这种方法将完成和 `filter` 函数完全相同的工作。哪种方法更清晰完全取决于你自己。 |

## Footnotes

[12] 从技术层面上讲，`filter` 的第二个参数可以是任意的序列，包括列表、元组以及定义了 `__getitem__` 特殊方法而能像列表一样工作的自定义类。在可能情况下，`filter` 会返回与输入相同的数据类型，也就是过滤一个列表返回一个列表，过滤一个元组返回一个元组。

# 16.4\. 重识列表映射

# 16.4\. 重识列表映射

你对使用列表解析映射列表的做法已经熟知。另一种方法可以完成同样的工作：使用内建 `map` 函数。它的工作机理和 `filter` 函数类似。

## 例 16.10\. `map` 介绍

```py
>>> def double(n):
... return n*2
... 
>>> li = [1, 2, 3, 5, 9, 10, 256, -3]
>>> map(double, li)                       
[2, 4, 6, 10, 18, 20, 512, -6]
>>> [double(n) for n in li]               
[2, 4, 6, 10, 18, 20, 512, -6]
>>> newlist = []
>>> for n in li:                          
... newlist.append(double(n))
... 
>>> newlist
[2, 4, 6, 10, 18, 20, 512, -6] 
```

|  |  |
| --- | --- |
| [1] | `map` 接受一个函数和一个列表作为参数，[13] 并对列表中的每个元素依次调用函数返回一个新的列表。在这个例子中，函数仅仅是将每个元素乘以 2。 |
| [2] | 使用列表解析的方法你可以做到相同的事情。列表解析是在 Python 2.0 版时被引入的；而 `map` 则古老得多。 |
| [3] | 你如果坚持以 Visual Basic 程序员自居，通过 `for` 循环的方法完成相同的任务也完全可以。 |

## 例 16.11\. `map` 与混合数据类型的列表

```py
>>> li = [5, 'a', (2, 'b')]
>>> map(double, li)                       
[10, 'aa', (2, 'b', 2, 'b')] 
```

|  |  |
| --- | --- |
| [1] | 作为一个旁注，我想指出只要提供的那个函数能够正确处理各种数据类型，`map` 对于混合数据类型列表的处理同样出色。在这里，`double` 函数仅仅是将给定参数乘以 2，Python 则会根据参数的数据类型决定正确操作的方法。对整数而言，这意味着乘 2；对字符串而言，意味着把自身和自身连接；对于元组，意味着构建一个包括原始元组全部元素和原始元组组合在一起的新元组。 |

好了，玩够了。让我们来看一些真实代码。

## 例 16.12\. `regression.py` 中的 `map`

```py
 filenameToModuleName = lambda f: os.path.splitext(f)[0] 
    moduleNames = map(filenameToModuleName, files) 
```

|  |  |
| --- | --- |
| [1] | 正如你在 第 4.7 节 “使用 lambda 函数” 中所见，`lambda` 定义一个内联函数。也正如你在 例 6.17 “分割路径名” 中所见，`os.path.splitext` 接受一个文件名并返回一个元组 `(_name_, _extension_)`。因此 `filenameToModuleName` 是一个接受文件名，剥离出其扩展名，然后只返回文件名称的函数。 |
| [2] | 调用 `map` 将把 `files` 列出的所有文件名传递给 `filenameToModuleName` 函数，并且返回每个函数调用结果所组成的列表。换句话说，你剔除掉文件名的扩展名，并将剔除后的文件名存于 `moduleNames` 之中。 |

如你在本章剩余部分将看到的，你可以将这种数据中心思想扩展到定义和执行一个容纳来自很多单个测试套件的测试的一个测试套件的最终目标。

## Footnotes

[13] 同前，我需要指出 `map` 可以接受一个列表、元组，或者一个像序列一样的对象。参见前面的关于 `filter` 的脚注。

# 16.5\. 数据中心思想编程

# 16.5\. 数据中心思想编程

现在的你，可能正抓耳挠腮地狠想，为什么这样比使用 `for` 循环和直接调用函数好。这是一个非常好的问题。通常这是一个程序观问题。使用 `map` 和 `filter` 强迫你围绕数据进行思考。

就此而言，你从没有数据开始，你所做的第一件事是获得当前脚本的目录路径，并获得该目录中的文件列表。这就是关键的一步，使你有了待处理的真实数据：文件名列表。

当然，你知道你并不关心所有的文件，而只关心测试套件。你有*太多数据*，因此你需要过滤(`filter`)数据。你如何知道哪些数据应该保留？你需要一个测试来确定，因此你定义一个测试并把它传给 `filter` 函数。这里你应用了一个正则表达式来确定，但无论如何构建测试，原则是一样的。

现在你有了每个测试套件的文件名 (且局限于测试套件，因为所有其他内容都被过滤掉了)，但是你还需要以模块名来替代之。你有正确数量的数据，只是*格式不正确*。因此，你定义了一个函数来将文件名转换为模块名，并使用这个函数映射整个列表。从一个文件名，你可以获得一个模块名，从一个文件名列表，你可以获得一个模块名列表。

如果不应用 `filter`，你也可以使用 `for` 循环结合一个 `if` 语句的方法。`map` 的使用则可以由一个 `for` 循环和一个函数调用来取代。但是 `for` 循环看起来像是个繁重的工作。至少，简单讲是在浪费时间，糟糕的话还会隐埋 Bug。例如，你需要弄清楚如何测试这样一个条件：“这个文件是测试套件吗？”这是应用特定的逻辑，没有哪个语言能自动为我们写出其代码。但是一旦你搞清楚了，你还需要费尽周折地定义一个新的空列表，写一个 `for` 循环以及一个 `if` 语句并手工地调用 `append` 将符合条件的元素一个个添加到新列表中，然后一路上注意区分哪个变量里放着过滤后的数据，哪个变量里放着未过滤的老数据。为什么不直接定义测试条件，然后由 Python 为你完成接下来的工作呢？

当然啦，你可以尝试眩一点的做法，去删除列表中的元素而不新建一个列表。但是你以前吃过这样的亏。试图在循环中改变数据结构是很容易出问题的。Python 是一个这样工作的语言吗？用多长时间你才能搞清这一点？你能确定记得你第二次这样尝试的安全性？程序员在和这类纯技术课题较劲的过程中，花费了太多的时间，犯了太多的错误，却并没有什么意义。这样并不可能令你的程序有所进步，只不过是费力不讨好。

我在第一次学习 Python 时是抵触列表解析的，而且我抗拒 `filter` 和 `map` 的时间更长。我坚持着我更艰难的生活，固守着类似于 `for` 循环和 `if` 语句以及一步步地以代码为中心的编程方式。而且我的 Python 程序看起来很像是 Visual Basic 程序，细化每一个函数中的每一个操作步骤。它们却有着同样的小错误和隐蔽的 Bug。这一切其实都没有意义。

让这一切都远去吧。费力不讨好的编程不重要，数据重要。并且数据并不麻烦，它们不过就是数据。如果多了，就过滤。如果不是我们要的，就映射。聚焦在数据上，摒弃费力的劳作。

# 16.6\. 动态导入模块

# 16.6\. 动态导入模块

好了，大道理谈够了。让我们谈谈动态导入模块吧。

首先，让我们看一看正常的模块导入。`import _module_` 语法查看搜索路径，根据给定的名字寻找模块并导入它们。你甚至可以这样做：以逗号分割同时导入多个模块，本章代码前几行就是这样做的。

## 例 16.13\. 同时导入多个模块

```py
 import sys, os, re, unittest 
```

|  |  |
| --- | --- |
| [1] | 这里同时导入四个模块：`sys` (为系统函数和得到命令行参数)、`os` (为目录列表之类的操作系统函数)、`re` (为正则表达式)，以及 `unittest` (为单元测试)。 |

现在让我们用动态导入做同样的事。

## 例 16.14\. 动态导入模块

```py
>>> sys = __import__('sys')           
>>> os = __import__('os')
>>> re = __import__('re')
>>> unittest = __import__('unittest')
>>> sys                               
>>> <module 'sys' (built-in)>
>>> os
>>> <module 'os' from '/usr/local/lib/python2.2/os.pyc'> 
```

|  |  |
| --- | --- |
| [1] | 内建 `__import__` 函数与 `import` 语句的既定目标相同，但它是一个真正的函数，并接受一个字符串参数。 |
| [2] | 变量 `sys` 现在是 `sys` 模块，和 `import sys` 的结果完全相同。变量 `os` 现在是一个 `os` 模块，等等。 |

因此 `__import__` 导入一个模块，但是是通过一个字符串参数来做到的。依此处讲，你用以导入的仅仅是一个硬编码性的字符串，但它可以是一个变量，或者一个函数调用的结果。并且你指向模块的变量也不必与模块名匹配。你可以导入一系列模块并把它们指派给一个列表。

## 例 16.15\. 动态导入模块列表

```py
>>> moduleNames = ['sys', 'os', 're', 'unittest'] 
>>> moduleNames
['sys', 'os', 're', 'unittest']
>>> modules = map(__import__, moduleNames)        
>>> modules                                       
[<module 'sys' (built-in)>,
<module 'os' from 'c:\Python22\lib\os.pyc'>,
<module 're' from 'c:\Python22\lib\re.pyc'>,
<module 'unittest' from 'c:\Python22\lib\unittest.pyc'>]
>>> modules[0].version                            
'2.2.2 (#37, Nov 26 2002, 10:24:37) [MSC 32 bit (Intel)]'
>>> import sys
>>> sys.version
'2.2.2 (#37, Nov 26 2002, 10:24:37) [MSC 32 bit (Intel)]' 
```

|  |  |
| --- | --- |
| [1] | `moduleNames` 只是一个字符串列表。没什么特别的，只是这些名字刚好是你可应需而用的可导入模块名。 |
| [2] | 简单得令人惊奇，通过映射 `__import__` 就实现了导入。记住，列表 (`moduleNames`) 的每个元素将被用来一次次调用函数 (`__import__`) 并以一个返回值构成的列表作为返回结果。 |
| [3] | 所以现在你已经由一个字符串列表构建起了一个实际模块的列表。(你的路径可能不同，这取决于你的操作系统、你安装 Python 的位置、月亮残缺的程度等等。) |
| [4] | 从这些是真实模块这一点出发，让我们来看一些模块属性。记住，`modules[0]` *是* `sys` 模块，因此，`modules[0].version` *是* `sys.version`。所有模块的其他属性和方法也都可用。`import` 语句没什么神奇的，模块也没什么神奇的。模块就是对象，一切都是对象。 |

现在，你应该能够把这一切放在一起，并搞清楚本章大部分样例代码是做什么的。

# 16.7\. 全部放在一起

# 16.7\. 全部放在一起

你已经学习了足够的知识，现在来分析本章样例代码的前七行：读取一个目录并从中导入选定的模块。

## 例 16.16\. `regressionTest` 函数

```py
 def regressionTest():
    path = os.path.abspath(os.path.dirname(sys.argv[0]))   
    files = os.listdir(path)                               
    test = re.compile("test\.py$", re.IGNORECASE)          
    files = filter(test.search, files)                     
    filenameToModuleName = lambda f: os.path.splitext(f)[0]
    moduleNames = map(filenameToModuleName, files)         
    modules = map(__import__, moduleNames)                 
load = unittest.defaultTestLoader.loadTestsFromModule  
return unittest.TestSuite(map(load, modules)) 
```

让我们一行行交互地看。假定当前目录是 `c:\diveintopython\py`，其中有包含本章脚本在内的本书众多样例。正如在 第 16.2 节 “找到路径” 中所见，脚本目录将存于 `path` 变量，因此让我们从这里开始以实打实的代码起步。

## 例 16.17\. 步骤 1：获得所有文件

```py
>>> import sys, os, re, unittest
>>> path = r'c:\diveintopython\py'
>>> files = os.listdir(path) 
>>> files 
['BaseHTMLProcessor.py', 'LICENSE.txt', 'apihelper.py', 'apihelpertest.py',
'argecho.py', 'autosize.py', 'builddialectexamples.py', 'dialect.py',
'fileinfo.py', 'fullpath.py', 'kgptest.py', 'makerealworddoc.py',
'odbchelper.py', 'odbchelpertest.py', 'parsephone.py', 'piglatin.py',
'plural.py', 'pluraltest.py', 'pyfontify.py', 'regression.py', 'roman.py', 'romantest.py',
'uncurly.py', 'unicode2koi8r.py', 'urllister.py', 'kgp', 'plural', 'roman',
'colorize.py'] 
```

|  |  |
| --- | --- |
| [1] | `files` 是由脚本所在目录的所有文件和目录构成的列表。(如果你已经运行了其中的一些样例，可能还会看到一些 `.pyc` 文件。) |

## 例 16.18\. 步骤 2：找到你关注的多个文件

```py
>>> test = re.compile("test\.py$", re.IGNORECASE)           
>>> files = filter(test.search, files)                      
>>> files                                                   
['apihelpertest.py', 'kgptest.py', 'odbchelpertest.py', 'pluraltest.py', 'romantest.py'] 
```

|  |  |
| --- | --- |
| [1] | 这个正则表达式将匹配以 `test.py` 结尾的任意字符串。注意，你必须转义这个点号，因为正则表达式中的点号通常意味着 “匹配任意单字符”，但是你实际上想匹配的事一个真正的点号。 |
| [2] | 被编译的正则表达式就像一个函数，因此你可以用它来过滤文件和目录构成的大列表，找寻符合正则表达式的所有元素。 |
| [3] | 剩下的是一个单元测试脚本列表，因为只有它们是形如 `SOMETHINGtest.py` 的文件。 |

## 例 16.19\. 步骤 3：映射文件名到模块名

```py
>>> filenameToModuleName = lambda f: os.path.splitext(f)[0] 
>>> filenameToModuleName('romantest.py')                    
'romantest'
>>> filenameToModuleName('odchelpertest.py')
'odbchelpertest'
>>> moduleNames = map(filenameToModuleName, files)          
>>> moduleNames                                             
['apihelpertest', 'kgptest', 'odbchelpertest', 'pluraltest', 'romantest'] 
```

|  |  |
| --- | --- |
| [1] | 正如你在 第 4.7 节 “使用 lambda 函数” 中所见，`lambda` 快餐式地创建内联单行函数。这里应用你在 例 6.17 “分割路径名” 中已经见过的，标准库的 `os.path.splitext` 将一个带有扩展名的文件名返回成只包含文件名称的那部分。 |
| [2] | `filenameToModuleName` 是一个函数。`lambda` 函数并不比你以 `def` 语句定义的普通函数神奇。你可以如其他函数一样地调用 `filenameToModuleName`，它也将如你所愿：从参数中剔除扩展名。 |
| [3] | 现在你可以通过 `map` 把这个函数应用于单元测试文件列表中的每一个文件。 |
| [4] | 结果当然如你所愿：以指代模块的字符串构成的一个列表。 |

## 例 16.20\. 步骤 4：映射模块名到模块

```py
>>> modules = map(__import__, moduleNames)                  
>>> modules                                                 
[<module 'apihelpertest' from 'apihelpertest.py'>,
<module 'kgptest' from 'kgptest.py'>,
<module 'odbchelpertest' from 'odbchelpertest.py'>,
<module 'pluraltest' from 'pluraltest.py'>,
<module 'romantest' from 'romantest.py'>]
>>> modules[-1]                                             
<module 'romantest' from 'romantest.py'> 
```

|  |  |
| --- | --- |
| [1] | 正如你在 第 16.6 节 “动态导入模块” 中所见，你可以通过 `map` 和 `__import__` 的协同工作，将模块名 (字符串) 映射到实际的模块 (像其他模块一样可以被调用和使用)。 |
| [2] | `modules` 现在是一个模块列表，其中的模块和其他模块一样。 |
| [3] | 该列表的最后一个模块*是* `romantest` 模块，和通过 `import romantest` 导入的模块完全等价。 |

## 例 16.21\. 步骤 5：将模块载入测试套件

```py
>>> load = unittest.defaultTestLoader.loadTestsFromModule 
>>> map(load, modules)                     
[<unittest.TestSuite tests=[
  <unittest.TestSuite tests=[<apihelpertest.BadInput testMethod=testNoObject>]>,
  <unittest.TestSuite tests=[<apihelpertest.KnownValues testMethod=testApiHelper>]>,
  <unittest.TestSuite tests=[
    <apihelpertest.ParamChecks testMethod=testCollapse>, 
    <apihelpertest.ParamChecks testMethod=testSpacing>]>, 
    ...
  ]
]
>>> unittest.TestSuite(map(load, modules)) 
```

|  |  |
| --- | --- |
| [1] | 模块对象的存在，使你不但可以像其他模块一样地使用它们；通过类的实例化和函数的调用，你还可以内省模块，从而弄清楚已经有了那些类和函数。这正是 `loadTestsFromModule` 方法的工作：内省每一个模块并为每个模块返回一个 `unittest.TestSuite` 对象。每个 `TestSuite` (测试套件) 对象都包含一个 `TestCase` 对象的列表，每个对象对应着你的模块中的一个测试方法。 |
| [2] | 最后，你将`TestSuite`列表封装成一个更大的测试套件。`unittest` 模块会很自如地遍历嵌套于测试套件中的树状结构，最后深入到独立测试方法，一个个加以运行并判断通过或是失败。 |

自省过程是 `unittest` 模块经常为我们做的一项工作。还记得我们的独立测试模块仅仅调用了看似神奇的 `unittest.main()` 函数就大刀阔斧地完成了全部工作吗？`unittest.main()` 实际上创建了一个 `unittest.TestProgram` 的实例，而这个实例实际上创建了一个 `unittest.defaultTestLoader` 的实例并以调用它的模块启动它。 (如果你不给出，如何知道调用它的模块是哪一个？通过使用同样神奇的 `__import__('__main__')` 命令，动态导入正在运行的模块。我可以就 `unittest` 模块中使用的所有技巧和技术写一本书，但那样我就没法写完这本了。)

## 例 16.22\. 步骤 6：告知 `unittest` 使用你的测试套件

```py
 if __name__ == "__main__":                   
    unittest.main(defaultTest="regressionTest") 
```

|  |  |
| --- | --- |
| [1] | 在不使用 `unittest` 模块来为我们做这一切的神奇工作的情况下，你实际上已自己做到了。你已经创建了一个自己就能导入模块、调用 `unittest.defaultTestLoader` 并封装于一个测试套件的 `regressionTest` 函数。现在你所要做的不是去寻找测试并以通用的方法构建一个测试套件，而是告诉 `unittest` 前面那些，它将调用 `regressionTest` 函数，而它会返回可以直接使用的 `TestSuite`。 |

# 16.8\. 小结

# 16.8\. 小结

`regression.py` 程序及其输出到现在应该很清楚了。

你现在应该能够很自如地做到如下事情：

*   从命令行操作路径信息。
*   不使用列表解析，使用 `filter` 过滤列表。
*   不使用列表解析，使用 `map` 映射列表。
*   动态导入模块。