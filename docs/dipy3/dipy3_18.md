# Chapter 15 案例研究:将`chardet`移植到 Python 3

> " Words, words. They’re all we have to go on. " — [Rosencrantz and Guildenstern are Dead](http://www.imdb.com/title/tt0100519/quotes)

## 概述

未知的或者不正确的字符编码是因特网上无效数据(gibberish text)的头号起因。在第三章，我们讨论过字符编码的历史，还有 Unicode 的产生，“一个能处理所有情况的大块头。”如果在网络上不再存在乱码这回事，我会爱上她的…因为所有的编辑系统(authoring system)保存有精确的编码信息，所有的传输协议都支持 Unicode，所有处理文本的系统在执行编码间转换的时候都可以保持高度精确。

我也会喜欢 pony。

Unicode　pony。

Unipony 也行。

这一章我会处理编码的自动检测。

## 什么是字符编码自动检测？

它是指当面对一串不知道编码信息的字节流的时候，尝试着确定一种编码方式以使我们能够读懂其中的文本内容。它就像我们没有解密钥匙的时候，尝试破解出编码。

### 那不是不可能的吗？

通常来说，是的，不可能。但是，有一些编码方式为特定的语言做了优化，而语言并非随机存在的。有一些字符序列在某种语言中总是会出现，而其他一些序列对该语言来说则毫无意义。一个熟练掌握英语的人翻开报纸，然后发现“txzqJv 2!dasd0a QqdKjvz”这样一些序列，他会马上意识到这不是英语（即使它完全由英语中的字母组成）。通过研究许多具有“代表性(typical)”的文本，计算机算法可以模拟人的这种对语言的感知，并且对一段文本的语言做出启发性的猜测。

换句话说就是，检测编码信息就是检测语言的类型，并辅之一些额外信息，比如每种语言通常会使用哪些编码方式。

### 这样的算法存在吗？

结果证明，是的，它存在。所有主流的浏览器都有字符编码自动检测的功能，因为因特网上总是充斥着大量缺乏编码信息的页面。[Mozilla Firefox 包含有一个自动检测字符编码的库](http://lxr.mozilla.org/seamonkey/source/extensions/universalchardet/src/base/)，它是开源的。[我将它导入到了 Python 2](http://chardet.feedparser.org/)，并且取绰号为`chardet`模块。这一章中，我会带领你一步一步地将`chardet`模块从 Python 2 移植到 Python 3。

## 介绍`chardet`模块

在开始代码移植之前，如果我们能理解代码是如何工作的这将非常有帮助！以下是一个简明地关于`chardet`模块代码结构的手册。`chardet`库太大，不可能都放在这儿，但是你可以[从`chardet.feedparser.org`下载它](http://chardet.feedparser.org/download/)。

编码检测就是语言检测。

`universaldetector.py`是检测算法的主入口点，它包含一个类，即`UniversalDetector`。（可能你会认为入口点是`chardet/__init__.py`中的`detect`函数，但是它只是一个便捷的包装方法，它会创建`UniversalDetector`对象，调用对象的方法，然后返回其结果。）

`UniversalDetector`共处理５类编码方式：

1.  包含字节顺序标记(BOM)的 UTF-n。它包括 UTF-8，大尾端和小尾端的 UTF-16，还有所有４字节顺序的 UTF-32 的变体。
2.  转义编码，它们与７字节的 ASCII 编码兼容，非 ASCII 编码的字符会以一个转义序列打头。比如：ISO-2022-JP(日文)和 HZ-GB-2312(中文).
3.  多字节编码，在这种编码方式中，每个字符使用可变长度的字节表示。比如：Big5(中文)，SHIFT_JIS(日文)，EUC-KR(韩文)和缺少 BOM 标记的 UTF-8。
4.  单字节编码，这种编码方式中，每个字符使用一个字节编码。例如：KOI8-R(俄语)，windows-1255(希伯来语)和 TIS-620(泰国语)。
5.  windows-1252，它主要被根本不知道字符编码的中层管理人员(middle manager)在 Microsoft Windows 上使用。

### 有 BOM 标记的 UTF-n

如果文本以 BOM 标记打头，我们可以合理地假设它使用了 UTF-8，UTF-16 或者 UTF-32 编码。（BOM 会告诉我们是其中哪一种，这就是它的功能。）这个过程在`UniversalDetector`中完成，并且不需要深入处理，会非常快地返回其结果。

### 转义编码

如果文本包含有可识别的能指示出某种转义编码的转义序列，`UniversalDetector`会创建一个`EscCharSetProber`对象（在`escprober.py`中定义），然后以该文本调用它。

`EscCharSetProber`会根据 HZ-GB-2312，ISO-2022-CN，ISO-2022-JP，和 ISO-2022-KR(在`escsm.py`中定义)来创建一系列的状态机(state machine)。`EscCharSetProber`将文本一次一个字节地输入到这些状态机中。如果某一个状态机最终唯一地确定了字符编码，`EscCharSetProber`迅速地将该有效结果返回给`UniversalDetector`，然后`UniversalDetector`将其返回给调用者。如果某一状态机进入了非法序列，它会被放弃，然后使用其他的状态机继续处理。

### 多字节编码

假设没有 BOM 标记，`UniversalDetector`会检测该文本是否包含任何高位字符(high-bit character)。如果有的话，它会创建一系列的“探测器(probers)”，检测这段广西是否使用多字节编码，单字节编码，或者作为最后的手段，是否为`windows-1252`编码。

这里的多字节编码探测器，即`MBCSGroupProber`（在`mbcsgroupprober.py`中定义），实际上是一个管理一组其他探测器的 shell，它用来处理每种多字节编码：Big5，GB2312，EUC-TW，EUC-KR，EUC-JP，SHIFT_JIS 和 UTF-8。`MBCSGroupProber`将文本作为每一个特定编码探测器的输入，并且检测其结果。如果某个探测器报告说它发现了一个非法的字节序列，那么该探测器则会被放弃，不再进一步处理（因此，换句话说就是，任何对`UniversalDetector`.`feed()`的子调用都会忽略那个探测器）。如果某一探测器报告说它有足够理由确信找到了正确的字符编码，那么`MBCSGroupProber`会将这个好消息传递给`UniversalDetector`，然后`UniversalDetector`将结果返回给调用者。

大多数的多字节编码探测器从类`MultiByteCharSetProber`(定义在`mbcharsetprober.py`中)继承而来，简单地挂上合适的状态机和分布分析器(distribution analyzer)，然后让`MultiByteCharSetProber`做剩余的工作。`MultiByteCharSetProber`将文本作为特定编码状态机的输入，每次一个字节，寻找能够指示出一个确定的正面或者负面结果的字节序列。同时，`MultiByteCharSetProber`会将文本作为特定编码分布分析机的输入。

分布分析机（在`chardistribution.py`中定义）使用特定语言的模型，此模型中的字符在该语言被使用得最频繁。一旦`MultiByteCharSetProber`把足够的文本给了分布分析机，它会根据其中频繁使用字符的数目，字符的总数和特定语言的分配比(distribution ratio)，来计算置信度(confidence rating)。如果置信度足够高，`MultiByteCharSetProber`会将结果返回给`MBCSGroupProber`，然后由`MBCSGroupProber`返回给`UniversalDetector`，最后`UniversalDetector`将其返回给调用者。

对于日语来说检测会更加困难。单字符的分布分析并不总能区别出`EUC-JP`和`SHIFT_JIS`，所以`SJISProber`（在`sjisprober.py`中定义）也使用双字符的分布分析。`SJISContextAnalysis`和`EUCJPContextAnalysis`(都定义在`jpcntx.py`中，并且都从类`JapaneseContextAnalysis`中继承）检测文本中的平假名音节字符(Hiragana syllabary characher)的出现次数。一旦处理了足够量的文本，它会返回一个置信度给`SJISProber`，`SJISProber`检查两个分析器的结果，然后将置信度高的那个返回给`MBCSGroupProber`。

### 单字节编码

说正经的，我的 Unicode pony 哪儿去了？

单字节编码的探测器，即`SBCSGroupProber`（定义在`sbcsgroupprober.py`中），也是一个管理一组其他探测器的 shell，它会尝试单字节编码和语言的每种组合：`windows-1251`，`KOI8-R`，`ISO-8859-5`，`MacCyrillic`，`IBM855`，and `IBM866`(俄语)；`ISO-8859-7`和`windows-1253`(希腊语)；`ISO-8859-5`和`windows-1251`(保加利亚语)；`ISO-8859-2`和`windows-1250`(匈牙利语)；`TIS-620`(泰国语)；`windows-1255`和`ISO-8859-8`(希伯来语)。

`SBCSGroupProber`将文本输入给这些特定编码+语言的探测器，然后检测它们的返回值。这些探测器的实现为某一个类，即`SingleByteCharSetProber`(在`sbcharsetprober.py`中定义)，它使用语言模型(language model)作为其参数。语言模型定义了典型文本中不同双字符序列出现的频度。`SingleByteCharSetProber`处理文本，统计出使用得最频繁的双字符序列。一旦处理了足够多的文本，它会根据频繁使用的序列的数目，字符总数和特定语言的分布系数来计算其置信度。

希伯来语被作为一种特殊的情况处理。如果在双字符分布分析中，文本被认定为是希伯来语，`HebrewProber`(在`hebrewprober.py`中定义)会尝试将其从 Visual Hebrew（源文本一行一行地被“反向”存储，然后一字不差地显示出来，这样就能从右到左的阅读）和 Logical Hebrew（源文本以阅读的顺序保存，在客户端从右到左进行渲染）区别开来。因为有一些字符在两种希伯来语中会以不同的方式编码，这依赖于它们是出现在单词的中间或者末尾，这样我们可以合理的猜测源文本的存储方向，然后返回合适的编码方式(`windows-1255`对应 Logical Hebrew，或者`ISO-8859-8`对应 Visual Hebrew)。

### `windows-1252`

如果`UniversalDetector`在文本中检测到一个高位字符，但是其他的多字节编码探测器或者单字节编码探测器都没有返回一个足够可靠的结果，它就会创建一个`Latin1Prober`对象(在`latin1prober.py`中定义)，尝试从中检测以`windows-1252`方式编码的英文文本。这种检测存在其固有的不可靠性，因为在不同的编码中，英文字符通常使用了相同的编码方式。唯一一种区别能出`windows-1252`的方法是通过检测常用的符号，比如[弯引号(smart quotes)](http://en.wikipedia.org/wiki/Smart_quotes)，撇号(curly apostrophes)，版权符号(copyright symbol)等这一类的符号。如果可能`Latin1Prober`会自动降低其置信度以使其他更精确的探测器检出结果。

## 运行`2to3`

我们将要开始移植`chardet`模块到 Python 3 了。Python 3 自带了一个叫做`2to3`的实用脚本，它使用 Python 2 的源代码作为输入，然后尽其可能地将其转换到 Python 3 的规范。某些情况下这很简单 — 一个被重命名或者被移动到其他模块中的函数 — 但是有些情况下，这个过程会变得非常复杂。想要了解所有它*能*做的事情，请参考附录，使用`2to3`将代码移植到 Python 3。接下来，我们会首先运行一次`2to3`，将它作用在`chardet`模块上，但是就如你即将看到的，在该自动化工具完成它的魔法表演后，仍然存在许多工作需要我们来收拾。

`chardet`包被分割为一些不同的文件，它们都放在同一个目录下。`2to3`能够立即处理多个文件：只需要将目录名作为命令行参数传递给`2to3`，然后它会轮流处理每个文件。

```py
C:\home\chardet> python c:\Python30\Tools\Scripts\2to3.py -w chardet\
RefactoringTool: Skipping implicit fixer: buffer
RefactoringTool: Skipping implicit fixer: idioms
RefactoringTool: Skipping implicit fixer: set_literal
RefactoringTool: Skipping implicit fixer: ws_comma
--- chardet\__init__.py (original)
+++ chardet\__init__.py (refactored)
@@ -18,7 +18,7 @@
 __version__ = "1.0.1"

 def detect(aBuf):
~~-    import universaldetector~~
<ins>+    from . import universaldetector</ins>
     u = universaldetector.UniversalDetector()
     u.reset()
     u.feed(aBuf)
--- chardet\big5prober.py (original)
+++ chardet\big5prober.py (refactored)
@@ -25,10 +25,10 @@
 # 02110-1301  USA
 ######################### END LICENSE BLOCK #########################

~~-from mbcharsetprober import MultiByteCharSetProber~~
~~-from codingstatemachine import CodingStateMachine~~
~~-from chardistribution import Big5DistributionAnalysis~~
~~-from mbcssm import Big5SMModel~~
<ins>+from .mbcharsetprober import MultiByteCharSetProber</ins>
<ins>+from .codingstatemachine import CodingStateMachine</ins>
<ins>+from .chardistribution import Big5DistributionAnalysis</ins>
<ins>+from .mbcssm import Big5SMModel</ins>

 class Big5Prober(MultiByteCharSetProber):
     def __init__(self):
--- chardet\chardistribution.py (original)
+++ chardet\chardistribution.py (refactored)
@@ -25,12 +25,12 @@
 # 02110-1301  USA
 ######################### END LICENSE BLOCK #########################

~~-import constants~~
~~-from euctwfreq import EUCTWCharToFreqOrder, EUCTW_TABLE_SIZE, EUCTW_TYPICAL_DISTRIBUTION_RATIO~~
~~-from euckrfreq import EUCKRCharToFreqOrder, EUCKR_TABLE_SIZE, EUCKR_TYPICAL_DISTRIBUTION_RATIO~~
~~-from gb2312freq import GB2312CharToFreqOrder, GB2312_TABLE_SIZE, GB2312_TYPICAL_DISTRIBUTION_RATIO~~
~~-from big5freq import Big5CharToFreqOrder, BIG5_TABLE_SIZE, BIG5_TYPICAL_DISTRIBUTION_RATIO~~
~~-from jisfreq import JISCharToFreqOrder, JIS_TABLE_SIZE, JIS_TYPICAL_DISTRIBUTION_RATIO~~
<ins>+from . import constants</ins>
<ins>+from .euctwfreq import EUCTWCharToFreqOrder, EUCTW_TABLE_SIZE, EUCTW_TYPICAL_DISTRIBUTION_RATIO</ins>
<ins>+from .euckrfreq import EUCKRCharToFreqOrder, EUCKR_TABLE_SIZE, EUCKR_TYPICAL_DISTRIBUTION_RATIO</ins>
<ins>+from .gb2312freq import GB2312CharToFreqOrder, GB2312_TABLE_SIZE, GB2312_TYPICAL_DISTRIBUTION_RATIO</ins>
<ins>+from .big5freq import Big5CharToFreqOrder, BIG5_TABLE_SIZE, BIG5_TYPICAL_DISTRIBUTION_RATIO</ins>
<ins>+from .jisfreq import JISCharToFreqOrder, JIS_TABLE_SIZE, JIS_TYPICAL_DISTRIBUTION_RATIO</ins>

 ENOUGH_DATA_THRESHOLD = 1024
 SURE_YES = 0.99
.
.
<mark>. (it goes on like this for a while)</mark>
.
.
RefactoringTool: Files that were modified:
RefactoringTool: chardet\__init__.py
RefactoringTool: chardet\big5prober.py
RefactoringTool: chardet\chardistribution.py
RefactoringTool: chardet\charsetgroupprober.py
RefactoringTool: chardet\codingstatemachine.py
RefactoringTool: chardet\constants.py
RefactoringTool: chardet\escprober.py
RefactoringTool: chardet\escsm.py
RefactoringTool: chardet\eucjpprober.py
RefactoringTool: chardet\euckrprober.py
RefactoringTool: chardet\euctwprober.py
RefactoringTool: chardet\gb2312prober.py
RefactoringTool: chardet\hebrewprober.py
RefactoringTool: chardet\jpcntx.py
RefactoringTool: chardet\langbulgarianmodel.py
RefactoringTool: chardet\langcyrillicmodel.py
RefactoringTool: chardet\langgreekmodel.py
RefactoringTool: chardet\langhebrewmodel.py
RefactoringTool: chardet\langhungarianmodel.py
RefactoringTool: chardet\langthaimodel.py
RefactoringTool: chardet\latin1prober.py
RefactoringTool: chardet\mbcharsetprober.py
RefactoringTool: chardet\mbcsgroupprober.py
RefactoringTool: chardet\mbcssm.py
RefactoringTool: chardet\sbcharsetprober.py
RefactoringTool: chardet\sbcsgroupprober.py
RefactoringTool: chardet\sjisprober.py
RefactoringTool: chardet\universaldetector.py
RefactoringTool: chardet\utf8prober.py 
```

现在我们对测试工具 — `test.py` — 应用`2to3`脚本。

```py
C:\home\chardet> python c:\Python30\Tools\Scripts\2to3.py -w test.py
RefactoringTool: Skipping implicit fixer: buffer
RefactoringTool: Skipping implicit fixer: idioms
RefactoringTool: Skipping implicit fixer: set_literal
RefactoringTool: Skipping implicit fixer: ws_comma
--- test.py (original)
+++ test.py (refactored)
@@ -4,7 +4,7 @@
 count = 0
 u = UniversalDetector()
 for f in glob.glob(sys.argv[1]):
~~-    print f.ljust(60),~~
<ins>+    print(f.ljust(60), end=' ')</ins>
     u.reset()
     for line in file(f, 'rb'):
         u.feed(line)
@@ -12,8 +12,8 @@
     u.close()
     result = u.result
     if result['encoding']:
~~-        print result['encoding'], 'with confidence', result['confidence']~~
<ins>+        print(result['encoding'], 'with confidence', result['confidence'])</ins>
     else:
~~-        print '******** no result'~~
<ins>+        print('******** no result')</ins>
     count += 1
~~-print count, 'tests'~~
<ins>+print(count, 'tests')</ins>
RefactoringTool: Files that were modified:
RefactoringTool: test.py 
```

看吧，还不算太难。只是转换了一些 impor 和 print 语句。说到这儿，那些 import 语句*原来*到底存在什么问题呢？为了回答这个问题，你需要知道`chardet`是如果被分割到多个文件的。

## 题外话，关于多文件模块

`chardet`是一个*多文件模块*。我也可以将所有的代码都放在一个文件里(并命名为`chardet.py`)，但是我没有。我创建了一个目录(叫做`chardet`)，然后我在那个目录里创建了一个`__init__.py`文件。*如果 Python 看到目录里有一个`__init__.py`文件，它会假设该目录里的所有文件都是同一个模块的某部分。*模块名为目录的名字。目录中的文件可以引用目录中的其他文件，甚至子目录中的也行。（再讲一分钟这个。）但是整个文件集合被作为一个单独的模块呈现给其他的 Python 代码 — 就好像所有的函数和类都在一个`.py`文件里。

在`__init__.py`中到底有些什么？什么也没有。一切。界于两者之间。`__init__.py`文件不需要定义任何东西；它确实可以是一个空文件。或者也可以使用它来定义我们的主入口函数。或者把我们所有的函数都放进去。或者其他函数都放，单单不放某一个函数…

> ☞包含有`__init__.py`文件的目录总是被看作一个多文件的模块。没有`__init__.py`文件的目录中，那些`.py`文件是不相关的。

我们来看看它实际上是怎样工作的。

```py
>>> import chardet

['__builtins__', '__doc__', '__file__', '__name__',
 '__package__', '__path__', '__version__', 'detect']

<module 'chardet' from 'C:\Python31\lib\site-packages\chardet\__init__.py'> 
```

1.  除了常见的类属性，在`chardet`模块中只多了一个`detect()`函数。
2.  这是我们发觉`chardet`模块不只是一个文件的第一个线索：“module”被当作文件`chardet/`目录中的`__init__.py`文件列出来。

我们再来瞟一眼`__init__.py`文件。

```py
 u = universaldetector.UniversalDetector()
    u.reset()
    u.feed(aBuf)
    u.close()
    return u.result 
```

1.  `__init__.py`文件定义了`detect()`函数，它是`chardet`库的主入口点。
2.  但是`detect()`函数没有任何实际的代码！事实上，它所做的事情只是导入了`universaldetector`模块然后开始调用它。但是`universaldetector`定义在哪儿？

答案就在那行古怪的`import`语句中：

```py
from . import universaldetector 
```

翻译成中文就是，“导入`universaldetector`模块；它跟我在同一目录，”这里的我即指文件`chardet/__init__.py`。这是一种提供给多文件模块中文件之间互相引用的方法，不需要担心它会与已经安装的搜索路径中的模块发生命名冲突。该条`import`语句*只会*在`chardet/`目录中查找`universaldetector`模块。

这两条概念 — `__init__.py`和相对导入 — 意味着我们可以将模块分割为任意多个块。`chardet`模块由 36 个`.py`文件组成 — 36！但我们所需要做的只是使用`chardet/__init__.py`文件中定义的某个函数。还有一件事情没有告诉你，`detect()`使用了相对导入来引用了`chardet/universaldetector.py`中定义的一个类，然后这个类又使用了相对导入引用了其他 5 个文件的内容，它们都在`chardet/`目录中。

> ☞如果你发现自己正在用 Python 写一个大型的库（或者更可能的情况是，当你意识到你的小模块已经变得很大的时候），最好花一些时间将它重构为一个多文件模块。这是 Python 所擅长的许多事情之一，那就利用一下这个优势吧。

## 修复`2to3`脚本所不能做的

### `False` is invalid syntax

你确实有测试样例，对吧？

现在开始真正的测试：使用测试集运行测试工具。由于测试集被设计成可以覆盖所有可能的代码路径，它是用来测试移植后的代码，保证 bug 不会埋伏在某个地方的一种不错的办法。

```py
C:\home\chardet> python test.py tests\*\*
Traceback (most recent call last):
  File "test.py", line 1, in <module>
    from chardet.universaldetector import UniversalDetector
  File "C:\home\chardet\chardet\universaldetector.py", line 51
    self.done = constants.False
                              ^
SyntaxError: invalid syntax 
```

唔，一个小麻烦。在 Python 3 中，`False`是一个保留字，所以不能把它用作变量名。我们来看一看`constants.py`来确定这是在哪儿定义的。以下是`constants.py`在执行`2to3`脚本之前原来的版本。

```py
import __builtin__
if not hasattr(__builtin__, 'False'):
    False = 0
    True = 1
else:
    False = __builtin__.False
    True = __builtin__.True 
```

这一段代码用来允许库在低版本的 Python 2 中运行，在 Python 2.3 以前，Python 没有内置的`bool`类型。这段代码检测内置的`True`和`False`常量是否缺失，如果必要的话则定义它们。

但是，Python 3 总是有`bool`类型的，所以整个这片代码都没有必要。最简单的方法是将所有的`constants.True`和`constants.False`都分别替换成`True`和`False`，然后将这段死代码从`constants.py`中移除。

所以`universaldetector.py`中的以下行：

```py
self.done = constants.False 
```

变成了

```py
self.done = False 
```

啊哈，是不是很有满足感？代码不仅更短了，而且更具可读性。

### No module named `constants`

是时候再运行一次`test.py`了，看看它能走多远。

```py
C:\home\chardet> python test.py tests\*\*
Traceback (most recent call last):
  File "test.py", line 1, in <module>
    from chardet.universaldetector import UniversalDetector
  File "C:\home\chardet\chardet\universaldetector.py", line 29, in <module>
    import constants, sys
ImportError: No module named constants 
```

说什么了？不存在叫做`constants`的模块？可是当然有`constants`这个模块了。它就在`chardet/constants.py`中。

还记得什么时候`2to3`脚本会修复所有那些导入语句吗？这个包内有许多的相对导入 — 即，在同一个库中，导入其他模块的模块 — 但是在*Python 3 中相对导入的逻辑已经变了*。在 Python 2 中，我们只需要`import constants`，然后它就会首先在`chardet/`目录中查找。在 Python 3 中，[所有的导入语句默认使用绝对路径](http://www.python.org/dev/peps/pep-0328/)。如果想要在 Python 3 中使用相对导入，你需要显式地说明：

```py
from . import constants 
```

但是。`2to3`脚本难道不是要自动修复这些的吗？好吧，它确实这样做了，但是该条导入语句在同一行组合了两种不同的导入类型：库内部对`constants`的相对导入，还有就是对`sys`模块的绝对导入，`sys`模块已经预装在了 Python 的标准库里。在 Python 2 里，我们可以将其组合到一条导入语句中。在 Python 3 中，我们不能这样做，并且`2to3`脚本也不是那样聪明，它不能把这条导入语句分成两条。

解决的办法是把这条导入语句手动的分成两条。所以这条二合一的导入语句：

```py
import constants, sys 
```

需要变成两条分享的导入语句：

```py
from . import constants
import sys 
```

在`chardet`库中还分散着许多这类问题的变体。某些地方它是“`import constants, sys`”；其他一些地方则是“`import constants, re`”。修改的方法是一样的：手工地将其分割为两条语句，一条为相对导入准备，另一条用于绝对导入。

前进！

### Name `'file'` is not defined

open()代替了原来的 file()。PapayaWhip 则替代了原来的 black

再来一次，运行`test.py`来执行我们的测试样例…

```py
C:\home\chardet> python test.py tests\*\*
tests\ascii\howto.diveintomark.org.xml
Traceback (most recent call last):
  File "test.py", line 9, in <module>
    for line in file(f, 'rb'):
NameError: name 'file' is not defined 
```

这一条也出乎我的意外，因为在记忆中我一直都在使用这种风格的代码。在 Python 2 里，全局的`file()`函数是`open()`函数的一个别名，`open()`函数是打开文件用于读取的标准方法。在 Python 3 中，全局的`file()`函数不再存在了，但是`open()`还保留着。

这样的话，最简单的解决办法就是将`file()`调用替换为对`open()`的调用：

```py
for line in open(f, 'rb'): 
```

这即是我关于这个问题想要说的。

### Can’t use a string pattern on a bytes-like object

现在事情开始变得有趣了。对于“有趣，”我的意思是“跟地狱一样让人迷茫。”

```py
C:\home\chardet> python test.py tests\*\*
tests\ascii\howto.diveintomark.org.xml
Traceback (most recent call last):
  File "test.py", line 10, in <module>
    u.feed(line)
  File "C:\home\chardet\chardet\universaldetector.py", line 98, in feed
    if self._highBitDetector.search(aBuf):
TypeError: can't use a string pattern on a bytes-like object 
```

我们先来看看`self._highBitDetector`是什么，然后再来调试这个错误。它被定义在`UniversalDetector`类的`__init__`方法中。

```py
class UniversalDetector:
    def __init__(self):
        self._highBitDetector = re.compile(r'[\x80-\xFF]') 
```

这段代码预编译一条正则表达式，它用来查找在 128–255 (0x80–0xFF)范围内的非 ASCII 字符。等一下，这似乎不太准确；我需要对更精确的术语来描述它。这个模式用来在 128-255 范围内查找非 ASCII 的*bytes*。

问题就出在这儿了。

在 Python 2 中，字符串是一个字节数组，它的字符编码信息被分开记录着。如果想要 Python 2 跟踪字符编码，你得使用 Unicode 编码的字符串(`u''`)。但是在 Python 3 中，字符串永远都是 Python 2 中所谓的 Unicode 编码的字符串 — 即，Unicode 字符数组（可能存在可变长字节）。由于这条正则表达式是使用字符串模式定义的，所以它只能用来搜索字符串 — 再强调一次，字符数组。但是我们所搜索的并非字符串，它是一个字节数组。看一看 traceback，该错误发生在`universaldetector.py`：

```py
def feed(self, aBuf):
    .
    .
    .
    if self._mInputState == ePureAscii:
        if self._highBitDetector.search(aBuf): 
```

`aBuf`是什么？让我们原路回到调用`UniversalDetector.feed()`的地方。有一处地方调用了它，是测试工具，`test.py`。

```py
u = UniversalDetector()
.
.
.
for line in open(f, 'rb'):
    u.feed(line) 
```

非字符数组，而是一个字节数组。

在此处我们找到了答案：`UniversalDetector.feed()`方法中，`aBuf`是从磁盘文件中读到的一行。仔细看一看用来打开文件的参数：`'rb'`。`'r'`是用来读取的；OK，没什么了不起的，我们在读取文件。啊，但是`'b'`是用以读取“二进制”数据的。如果没有标记`'b'`，`for`循环会一行一行地读取文件，然后将其转换为一个字符串 — Unicode 编码的字符数组 — 根据系统默认的编码方式。但是使用`'b'`标记后，`for`循环一行一行地读取文件，然后将其按原样存储为字节数组。该字节数组被传递给了 `UniversalDetector.feed()`方法，最后给了预编译好的正则表达式，`self._highBitDetector`，用来搜索高位…字符。但是没有字符；有的只是字节。苍天哪。

我们需要该正则表达式搜索的并不是字符数组，而是一个字节数组。

只要我们认识到了这一点，解决办法就有了。使用字符串定义的正则表达式可以搜索字符串。使用字节数组定义的正则表达式可以搜索字节数组。我们只需要改变用来定义正则表达式的参数的类型为字节数组，就可以定义一个字节数组模式。（还有另外一个该问题的实例，在下一行。）

```py
 class UniversalDetector:
      def __init__(self):
~~-         self._highBitDetector = re.compile(r'[\x80-\xFF]')~~
~~-         self._escDetector = re.compile(r'(\033|~{)')~~
<ins>+         self._highBitDetector = re.compile(b'[\x80-\xFF]')</ins>
<ins>+         self._escDetector = re.compile(b'(\033|~{)')</ins>
          self._mEscCharSetProber = None
          self._mCharSetProbers = []
          self.reset() 
```

在整个代码库内搜索对`re`模块的使用发现了另外两个该类型问题的实例，出现在`charsetprober.py`文件中。再次，以上代码将正则表达式定义为字符串，但是却将它们作用在`aBuf`上，而`aBuf`是一个字节数组。解决方案还是一样的：将正则表达式模式定义为字节数组。

```py
 class CharSetProber:
      .
      .
      .
      def filter_high_bit_only(self, aBuf):
~~-         aBuf = re.sub(r'([\x00-\x7F])+', ' ', aBuf)~~
<ins>+         aBuf = re.sub(b'([\x00-\x7F])+', b' ', aBuf)</ins>
          return aBuf

      def filter_without_english_letters(self, aBuf):
~~-         aBuf = re.sub(r'([A-Za-z])+', ' ', aBuf)~~
<ins>+         aBuf = re.sub(b'([A-Za-z])+', b' ', aBuf)</ins>
          return aBuf 
```

### Can't convert `'bytes'` object to `str` implicitly

奇怪，越来越不寻常了…

```py
C:\home\chardet> python test.py tests\*\*
tests\ascii\howto.diveintomark.org.xml
Traceback (most recent call last):
  File "test.py", line 10, in <module>
    u.feed(line)
  File "C:\home\chardet\chardet\universaldetector.py", line 100, in feed
    elif (self._mInputState == ePureAscii) and self._escDetector.search(self._mLastChar + aBuf):
TypeError: Can't convert 'bytes' object to str implicitly 
```

在此存在一个 Python 解释器与代码风格之间的不协调。`TypeError`可以出现在那一行的任意地方，但是 traceback 不能明确定地指出错误的位置。可能是第一个或者第二个条件语句(conditional)，对 traceback 来说，它们是一样的。为了缩小调试的范围，我们需要把这条代码分割成两行，像这样：

```py
elif (self._mInputState == ePureAscii) and \
    self._escDetector.search(self._mLastChar + aBuf): 
```

然后再运行测试工具：

```py
C:\home\chardet> python test.py tests\*\*
tests\ascii\howto.diveintomark.org.xml
Traceback (most recent call last):
  File "test.py", line 10, in <module>
    u.feed(line)
  File "C:\home\chardet\chardet\universaldetector.py", line 101, in feed
    self._escDetector.search(self._mLastChar + aBuf):
TypeError: Can't convert 'bytes' object to str implicitly 
```

啊哈！错误不在第一个条件语句上(`self._mInputState == ePureAscii`)，是第二个的问题。但是，是什么引发了`TypeError`错误呢？也许你会想`search()`方法需要另外一种类型的参数，但是那样的话，就不会产生当前这种 traceback 了。Python 函数可以使用任何类型参数；只要传递了正确数目的参数，函数就可以执行。如果我们给函数传递了类型不匹配的参数，代码可能就会*崩溃*，但是这样一来，traceback 就会指向函数内部的某一代码块了。但是当前得到的 traceback 告诉我们，错误就出现在开始调用`search()`函数那儿。所以错误肯定就出在`+`操作符上，该操作用于构建最终会传递给`search()`方法的参数。

从前一次调试的过程中，我们已经知道`aBuf`是一个字节数组。那么`self._mLastChar`又是什么呢？它是一个在`reset()`中定义的实例变量，而`reset()`方法刚好就是被`__init__()`调用的。

```py
class UniversalDetector:
    def __init__(self):
        self._highBitDetector = re.compile(b'[\x80-\xFF]')
        self._escDetector = re.compile(b'(\033|~{)')
        self._mEscCharSetProber = None
        self._mCharSetProbers = []
 <mark>self.reset()</mark>

    def reset(self):
        self.result = {'encoding': None, 'confidence': 0.0}
        self.done = False
        self._mStart = True
        self._mGotData = False
        self._mInputState = ePureAscii
 <mark>self._mLastChar = ''</mark> 
```

现在我们找到问题的症结所在了。你发现了吗？`self._mLastChar`是一个字符串，而`aBuf`是一个字节数组。而我们不允许对字符串和字节数组做连接操作 — 即使是空串也不行。

那么，`self._mLastChar`到底是什么呢？在`feed()`方法中，在 traceback 报告的位置以下几行就是了。

```py
if self._mInputState == ePureAscii:
    if self._highBitDetector.search(aBuf):
        self._mInputState = eHighbyte
    elif (self._mInputState == ePureAscii) and \
            self._escDetector.search(self._mLastChar + aBuf):
        self._mInputState = eEscAscii

<mark>self._mLastChar = aBuf[-1]</mark> 
```

`feed()`方法被一次一次地调用，每次都传递给它几个字节。该方法处理好它收到的字节（以`aBuf`传递进去的），然后将最后一个字节保存在`self._mLastChar`中，以便下次调用时还会用到。（在多字节编码中，`feed()`在调用的时候可能只收到了某个字符的一半，然后下次调用时另一半才被传到。）但是因为`aBuf`已经变成了一个字节数组，所以`self._mLastChar`也需要与其匹配。可以这样做：

```py
 def reset(self):
      .
      .
      .
~~-     self._mLastChar = ''~~
<ins>+     self._mLastChar = b''</ins> 
```

在代码库中搜索“`mLastChar`”，`mbcharsetprober.py`中也发现一个相似的问题，与之前不同的是，它记录的是最后*2*个字符。`MultiByteCharSetProber`类使用一个单字符列表来记录末尾的两个字符。在 Python 3 中，这需要使用一个整数列表，因为实际上它记录的并不是是字符，而是字节对象。（字节对象即范围在`0-255`内的整数。）

```py
 class MultiByteCharSetProber(CharSetProber):
      def __init__(self):
          CharSetProber.__init__(self)
          self._mDistributionAnalyzer = None
          self._mCodingSM = None
~~-         self._mLastChar = ['\x00', '\x00']~~
<ins>+         self._mLastChar = [0, 0]</ins>

      def reset(self):
          CharSetProber.reset(self)
          if self._mCodingSM:
              self._mCodingSM.reset()
          if self._mDistributionAnalyzer:
              self._mDistributionAnalyzer.reset()
~~-         self._mLastChar = ['\x00', '\x00']~~
<ins>+         self._mLastChar = [0, 0]</ins> 
```

### Unsupported operand type(s) for +: `'int'` and `'bytes'`

有好消息，也有坏消息。好消息是我们一直在前进着…

```py
C:\home\chardet> python test.py tests\*\*
tests\ascii\howto.diveintomark.org.xml
Traceback (most recent call last):
  File "test.py", line 10, in <module>
    u.feed(line)
  File "C:\home\chardet\chardet\universaldetector.py", line 101, in feed
    self._escDetector.search(self._mLastChar + aBuf):
TypeError: unsupported operand type(s) for +: 'int' and 'bytes' 
```

…坏消息是，我们好像一直都在原地踏步。

但我们确实一直在取得进展！真的！即使 traceback 在相同的地方再次出现，这一次的错误毕竟与上次不同。前进！那么，这次又是什么错误呢？上一次我们确认过了，这一行代码不应该会再做连接`int`型和字节数组(`bytes`)的操作。事实上，我们刚刚花了相当长一段时间来保证`self._mLastChar`是一个字节数组。它怎么会变成`int`呢？

答案不在上几行代码中，而在以下几行。

```py
if self._mInputState == ePureAscii:
    if self._highBitDetector.search(aBuf):
        self._mInputState = eHighbyte
    elif (self._mInputState == ePureAscii) and \
            self._escDetector.search(self._mLastChar + aBuf):
        self._mInputState = eEscAscii

<mark>self._mLastChar = aBuf[-1]</mark> 
```

字符串中的元素仍然是字符串，字节数组中的元素则为整数。

该错误没有发生在`feed()`方法第一次被调用的时候；而是在*第二次*调用的过程中，在`self._mLastChar`被赋值为`aBuf`末尾的那个字节之后。好吧，这又会有什么问题呢？因为获取字节数组中的单个元素会产生一个整数，而不是字节数组。它们之间的区别，请看以下在交互式 shell 中的操作：

```py
 >>> len(aBuf)
3
>>> mLastChar = aBuf[-1]

191

<class 'int'>

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unsupported operand type(s) for +: 'int' and 'bytes'

>>> mLastChar
b'\xbf'

b'\xbf\xef\xbb\xbf' 
```

1.  定义一个长度为 3 的字节数组。
2.  字节数组的最后一个元素为 191。
3.  它是一个整数。
4.  连接整数和字节数组的操作是不允许的。我们重复了在`universaldetector.py`中发现的那个错误。
5.  啊，这就是解决办法了。使用列表分片从数组的最后一个元素中创建一个新的字节数组，而不是直接获取这个元素。即，从最后一个元素开始切割，直到到达数组的末尾。当前`mLastChar`是一个长度为 1 的字节数组。
6.  连接长度分别为 1 和 3 的字节数组，则会返回一个新的长度为 4 的字节数组。

所以，为了保证`universaldetector.py`中的`feed()`方法不管被调用多少次都能够正常运行，我们需要将`self._mLastChar`实例化为一个长度为 0 的字节数组，并且保证它一直是一个字节数组。

```py
 self._escDetector.search(self._mLastChar + aBuf):
          self._mInputState = eEscAscii

~~- self._mLastChar = aBuf[-1]~~
<ins>+ self._mLastChar = aBuf[-1:]</ins> 
```

### `ord()` expected string of length 1, but `int` found

困了吗？就要完成了…

```py
C:\home\chardet> python test.py tests\*\*
tests\ascii\howto.diveintomark.org.xml                       ascii with confidence 1.0
tests\Big5\0804.blogspot.com.xml
Traceback (most recent call last):
  File "test.py", line 10, in <module>
    u.feed(line)
  File "C:\home\chardet\chardet\universaldetector.py", line 116, in feed
    if prober.feed(aBuf) == constants.eFoundIt:
  File "C:\home\chardet\chardet\charsetgroupprober.py", line 60, in feed
    st = prober.feed(aBuf)
  File "C:\home\chardet\chardet\utf8prober.py", line 53, in feed
    codingState = self._mCodingSM.next_state(c)
  File "C:\home\chardet\chardet\codingstatemachine.py", line 43, in next_state
    byteCls = self._mModel['classTable'][ord(c)]
TypeError: ord() expected string of length 1, but int found 
```

OK，因为`c`是`int`类型的，但是`ord()`需要一个长度为 1 的字符串。就是这样了。`c`在哪儿定义的？

```py
# codingstatemachine.py
def next_state(self, c):
    # for each byte we get its class
    # if it is first byte, we also get byte length
    byteCls = self._mModel['classTable'][ord(c)] 
```

不是这儿； 此处`c`只是被传递给了`next_state()`函数。我们再上一级看看。

```py
# utf8prober.py
def feed(self, aBuf):
    for c in aBuf:
        codingState = self._mCodingSM.next_state(c) 
```

看到了吗？在 Python 2 中，`aBuf`是一个字符串，所以`c`就是一个长度为 1 的字符串。（那就是我们通过遍历字符串所得到的 — 所有的字符，一次一个。）因为现在`aBuf`是一个字节数组，所以`c`变成了`int`类型的，而不再是长度为 1 的字符串。也就是说，没有必要再调用`ord()`函数了，因为`c`已经是`int`了！

这样修改：

```py
 def next_state(self, c):
      # for each byte we get its class
      # if it is first byte, we also get byte length
~~-     byteCls = self._mModel['classTable'][ord(c)]~~
<ins>+     byteCls = self._mModel['classTable'][c]</ins> 
```

在代码库中搜索“`ord(c)`”后，发现`sbcharsetprober.py`中也有相似的问题…

```py
# sbcharsetprober.py
def feed(self, aBuf):
    if not self._mModel['keepEnglishLetter']:
        aBuf = self.filter_without_english_letters(aBuf)
    aLen = len(aBuf)
    if not aLen:
        return self.get_state()
    for c in aBuf:
 <mark>order = self._mModel['charToOrderMap'][ord(c)]</mark> 
```

…还有`latin1prober.py`…

```py
# latin1prober.py
def feed(self, aBuf):
    aBuf = self.filter_with_english_letters(aBuf)
    for c in aBuf:
 <mark>charClass = Latin1_CharToClass[ord(c)]</mark> 
```

`c`在`aBuf`中遍历，这就意味着它是一个整数，而非字符串。解决方案是相同的：把`ord(c)`就替换成`c`。

```py
 # sbcharsetprober.py
  def feed(self, aBuf):
      if not self._mModel['keepEnglishLetter']:
          aBuf = self.filter_without_english_letters(aBuf)
      aLen = len(aBuf)
      if not aLen:
          return self.get_state()
      for c in aBuf:
~~-         order = self._mModel['charToOrderMap'][ord(c)]~~
<ins>+         order = self._mModel['charToOrderMap'][c]</ins>

  # latin1prober.py
  def feed(self, aBuf):
      aBuf = self.filter_with_english_letters(aBuf)
      for c in aBuf:
~~-         charClass = Latin1_CharToClass[ord(c)]~~
<ins>+         charClass = Latin1_CharToClass[c]</ins> 
```

### Unorderable types: `int()` >= `str()`

继续我们的路吧。

```py
C:\home\chardet> python test.py tests\*\*
tests\ascii\howto.diveintomark.org.xml                       ascii with confidence 1.0
tests\Big5\0804.blogspot.com.xml
Traceback (most recent call last):
  File "test.py", line 10, in <module>
    u.feed(line)
  File "C:\home\chardet\chardet\universaldetector.py", line 116, in feed
    if prober.feed(aBuf) == constants.eFoundIt:
  File "C:\home\chardet\chardet\charsetgroupprober.py", line 60, in feed
    st = prober.feed(aBuf)
  File "C:\home\chardet\chardet\sjisprober.py", line 68, in feed
    self._mContextAnalyzer.feed(self._mLastChar[2 - charLen :], charLen)
  File "C:\home\chardet\chardet\jpcntx.py", line 145, in feed
    order, charLen = self.get_order(aBuf[i:i+2])
  File "C:\home\chardet\chardet\jpcntx.py", line 176, in get_order
    if ((aStr[0] >= '\x81') and (aStr[0] <= '\x9F')) or \
TypeError: unorderable types: int() >= str() 
```

这都是些什么？“Unorderable types”？字节数组与字符串之间的差异引起的问题再一次出现了。看一看以下代码：

```py
class SJISContextAnalysis(JapaneseContextAnalysis):
    def get_order(self, aStr):
        if not aStr: return -1, 1
        # find out current char's byte length
 <mark>if ((aStr[0] >= '\x81') and (aStr[0] <= '\x9F')) or \</mark>
           ((aStr[0] >= '\xE0') and (aStr[0] <= '\xFC')):
            charLen = 2
        else:
            charLen = 1 
```

`aStr`从何而来？再深入栈内看一看：

```py
def feed(self, aBuf, aLen):
    .
    .
    .
    i = self._mNeedToSkipCharNum
    while i < aLen:
 <mark>order, charLen = self.get_order(aBuf[i:i+2])</mark> 
```

看，是`aBuf`，我们的老战友。从我们在这一章中所遇到的问题你也可以猜到了问题的关键了，因为`aBuf`是一个字节数组。此处`feed()`方法并不是整个地将它传递出去；而是先对它执行分片操作。就如你在这章前面看到的，对字节数组执行分片操作的返回值仍然为字节数组，所以传递给`get_order()`方法的`aStr`仍然是字节数组。

那么以下代码是怎样处理`aStr`的呢？它将该字节第一个元素与长度为 1 的字符串进行比较操作。在 Python 2，这是可以的，因为`aStr`和`aBuf`都是字符串，所以`aStr[0]`也是字符串，并且我们允许比较两个字符串的是否相等。但是在 Python 3 中，`aStr`和`aBuf`都是字节数组，而`aStr[0]`就成了一个整数，没有执行显式地强制转换的话，是不能对整数和字符串执行相等性比较的。

在当前情况下，没有必要添加强制转换，这会让代码变得更加复杂。`aStr[0]`产生一个整数；而我们所比较的对象都是常量(constant)。那就把长度为 1 的字符串换成整数吧。我们也顺便把`aStr`换成`aBuf`吧，因为`aStr`本来也不是一个字符串。

```py
 class SJISContextAnalysis(JapaneseContextAnalysis):
~~-     def get_order(self, aStr):~~
~~-      if not aStr: return -1, 1~~
<ins>+     def get_order(self, aBuf):</ins>
<ins>+      if not aBuf: return -1, 1</ins>
          # find out current char's byte length
~~-         if ((aStr[0] >= '\x81') and (aStr[0] <= '\x9F')) or \~~
~~-            ((aBuf[0] >= '\xE0') and (aBuf[0] <= '\xFC')):~~
<ins>+         if ((aBuf[0] >= 0x81) and (aBuf[0] <= 0x9F)) or \</ins>
<ins>+            ((aBuf[0] >= 0xE0) and (aBuf[0] <= 0xFC)):</ins>
              charLen = 2
          else:
              charLen = 1

          # return its order if it is hiragana
~~-      if len(aStr) > 1:~~
~~-             if (aStr[0] == '\202') and \~~
~~-                (aStr[1] >= '\x9F') and \~~
~~-                (aStr[1] <= '\xF1'):~~
~~-                return ord(aStr[1]) - 0x9F, charLen~~
<ins>+      if len(aBuf) > 1:</ins>
<ins>+             if (aBuf[0] == 0x202) and \</ins>
<ins>+                (aBuf[1] >= 0x9F) and \</ins>
<ins>+                (aBuf[1] <= 0xF1):</ins>
<ins>+                return aBuf[1] - 0x9F, charLen</ins>

          return -1, charLen

  class EUCJPContextAnalysis(JapaneseContextAnalysis):
~~-     def get_order(self, aStr):~~
~~-      if not aStr: return -1, 1~~
<ins>+     def get_order(self, aBuf):</ins>
<ins>+      if not aBuf: return -1, 1</ins>
          # find out current char's byte length
~~-         if (aStr[0] == '\x8E') or \~~
~~-           ((aStr[0] >= '\xA1') and (aStr[0] <= '\xFE')):~~
<ins>+         if (aBuf[0] == 0x8E) or \</ins>
<ins>+           ((aBuf[0] >= 0xA1) and (aBuf[0] <= 0xFE)):</ins>
              charLen = 2
~~-         elif aStr[0] == '\x8F':~~
<ins>+         elif aBuf[0] == 0x8F:</ins>
              charLen = 3
          else:
              charLen = 1

        # return its order if it is hiragana
~~-    if len(aStr) > 1:~~
~~-           if (aStr[0] == '\xA4') and \~~
~~-              (aStr[1] >= '\xA1') and \~~
~~-              (aStr[1] <= '\xF3'):~~
~~-                 return ord(aStr[1]) - 0xA1, charLen~~
<ins>+    if len(aBuf) > 1:</ins>
<ins>+           if (aBuf[0] == 0xA4) and \</ins>
<ins>+              (aBuf[1] >= 0xA1) and \</ins>
<ins>+              (aBuf[1] <= 0xF3):</ins>
<ins>+               return aBuf[1] - 0xA1, charLen</ins>

        return -1, charLen 
```

在代码库中查找`ord()`函数，我们在`chardistribution.py`中也发现了同样的问题（更确切地说，在以下这些类中，`EUCTWDistributionAnalysis`，`EUCKRDistributionAnalysis`，`GB2312DistributionAnalysis`，`Big5DistributionAnalysis`，`SJISDistributionAnalysis`和`EUCJPDistributionAnalysis`）。对于它们存在的问题，解决办法与我们对`jpcntx.py`中的类`EUCJPContextAnalysis`和`SJISContextAnalysis`的做法相似。

### Global name `'reduce'` is not defined

再次陷入中断…

```py
C:\home\chardet> python test.py tests\*\*
tests\ascii\howto.diveintomark.org.xml                       ascii with confidence 1.0
tests\Big5\0804.blogspot.com.xml
Traceback (most recent call last):
  File "test.py", line 12, in <module>
    u.close()
  File "C:\home\chardet\chardet\universaldetector.py", line 141, in close
    proberConfidence = prober.get_confidence()
  File "C:\home\chardet\chardet\latin1prober.py", line 126, in get_confidence
    total = reduce(operator.add, self._mFreqCounter)
NameError: global name 'reduce' is not defined 
```

根据官方手册：[What’s New In Python 3.0](http://docs.python.org/3.0/whatsnew/3.0.html#builtins)，函数`reduce()`已经从全局名字空间中移出，放到了`functools`模块中。引用手册中的内容：“如果需要，请使用`functools.reduce()`，99％的情况下，显式的`for`循环使代码更有可读性。”你可以从 Guido van Rossum 的一篇日志中看到关于这项决策的更多细节：[The fate of reduce() in Python 3000](http://www.artima.com/weblogs/viewpost.jsp?thread=98196)。

```py
def get_confidence(self):
    if self.get_state() == constants.eNotMe:
        return 0.01

 <mark>total = reduce(operator.add, self._mFreqCounter)</mark> 
```

`reduce()`函数使用两个参数 — 一个函数，一个列表（更严格地说，可迭代的对象就行了） — 然后将函数增量式地作用在列表的每个元素上。换句话说，这是一种良好而高效的用于综合(add up)列表所有元素并返回其结果的方法。

这种强大的技术使用如此频繁，所以 Python 就添加了一个全局的`sum()`函数。

```py
 def get_confidence(self):
      if self.get_state() == constants.eNotMe:
          return 0.01

~~-     total = reduce(operator.add, self._mFreqCounter)~~
<ins>+     total = sum(self._mFreqCounter)</ins> 
```

由于我们不再使用`operator`模块，所以可以在文件最上方移除那条`import`语句。

```py
 from .charsetprober import CharSetProber
  from . import constants
~~- import operator~~ 
```

可以开始测试了吧？（快要吐血的样子…）

```py
C:\home\chardet> python test.py tests\*\*
tests\ascii\howto.diveintomark.org.xml                       ascii with confidence 1.0
tests\Big5\0804.blogspot.com.xml                             Big5 with confidence 0.99
tests\Big5\blog.worren.net.xml                               Big5 with confidence 0.99
tests\Big5\carbonxiv.blogspot.com.xml                        Big5 with confidence 0.99
tests\Big5\catshadow.blogspot.com.xml                        Big5 with confidence 0.99
tests\Big5\coolloud.org.tw.xml                               Big5 with confidence 0.99
tests\Big5\digitalwall.com.xml                               Big5 with confidence 0.99
tests\Big5\ebao.us.xml                                       Big5 with confidence 0.99
tests\Big5\fudesign.blogspot.com.xml                         Big5 with confidence 0.99
tests\Big5\kafkatseng.blogspot.com.xml                       Big5 with confidence 0.99
tests\Big5\ke207.blogspot.com.xml                            Big5 with confidence 0.99
tests\Big5\leavesth.blogspot.com.xml                         Big5 with confidence 0.99
tests\Big5\letterlego.blogspot.com.xml                       Big5 with confidence 0.99
tests\Big5\linyijen.blogspot.com.xml                         Big5 with confidence 0.99
tests\Big5\marilynwu.blogspot.com.xml                        Big5 with confidence 0.99
tests\Big5\myblog.pchome.com.tw.xml                          Big5 with confidence 0.99
tests\Big5\oui-design.com.xml                                Big5 with confidence 0.99
tests\Big5\sanwenji.blogspot.com.xml                         Big5 with confidence 0.99
tests\Big5\sinica.edu.tw.xml                                 Big5 with confidence 0.99
tests\Big5\sylvia1976.blogspot.com.xml                       Big5 with confidence 0.99
tests\Big5\tlkkuo.blogspot.com.xml                           Big5 with confidence 0.99
tests\Big5\tw.blog.xubg.com.xml                              Big5 with confidence 0.99
tests\Big5\unoriginalblog.com.xml                            Big5 with confidence 0.99
tests\Big5\upsaid.com.xml                                    Big5 with confidence 0.99
tests\Big5\willythecop.blogspot.com.xml                      Big5 with confidence 0.99
tests\Big5\ytc.blogspot.com.xml                              Big5 with confidence 0.99
tests\EUC-JP\aivy.co.jp.xml                                  EUC-JP with confidence 0.99
tests\EUC-JP\akaname.main.jp.xml                             EUC-JP with confidence 0.99
tests\EUC-JP\arclamp.jp.xml                                  EUC-JP with confidence 0.99
.
.
.
316 tests 
```

天哪，伙计，她真的欢快地跑起来了！*[/me does a little dance](http://www.hampsterdance.com/)*

## 总结

我们学到了什么？

1.  尝试大批量地把代码从 Python 2 移植到 Python 3 上是一件让人头疼的工作。没有捷径。它确实很困难。
2.  自动化的`2to3`脚本确实有用，但是它只能做一些简单的辅助工作 — 函数重命名，模块重命名，语法修改等。之前，它被认为是一项会让人印象深刻的大工程，但是最后，实际上它只是一个能智能地执行查找替换机器人。
3.  在移植`chardet`库的时候遇到的头号问题就是：字符串和字节对象之间的差异。在我们这个情况中，这种问题比较明显，因为整个`chardet`库就是一直在执行从字节流到字符串的转换。但是“字节流”出现的方式会远超出你的想象。以“二进制”模式读取文件？我们会获得字节流。获取一份 web 页面？调用 web API？这也会返回字节流。
4.  *你*需要彻底地了解所面对的程序。如果那段程序是自己写自然非常好，但是至少，我们需要够理解所有晦涩难懂的细节。因为 bug 可能埋伏在任何地方。
5.  测试样例是必要的。没有它们的话不要尝试着移植代码。我自信移植后的`chardet`模块能在 Python 3 中工作的*唯一*理由是，我一开始就使用了测试集合来检验所有主要的代码路径。如果你还没有任何测试集，在移植代码之前自己写一些吧。如果你的测试集合太小，那么请写全。如果测试集够了，那么，我们就又可以开始历险了。