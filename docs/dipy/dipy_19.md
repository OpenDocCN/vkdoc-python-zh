# 第十八章 性能优化

# 第十八章 性能优化

*   18.1\. 概览
*   18.2\. 使用 timeit 模块
*   18.3\. 优化正则表达式
*   18.4\. 优化字典查找
*   18.5\. 优化列表操作
*   18.6\. 优化字符串操作
*   18.7\. 小结

性能优化 (Performance tuning) 是一件多姿多彩的事情。Python 是一种解释性语言并不表示你不应该担心代码优化。但也不必*太* 担心。

# 18.1\. 概览

# 18.1\. 概览

由于代码优化过程中存在太多的不明确因素，以至于你很难清楚该从何入手。

让我们从这里开始：*你真的确信你要这样做吗？* 你的代码真的那么差吗？值得花时间去优化它吗？在你的应用程序的生命周期中，与花费在等待一个远程数据库服务器，或是等待用户输入相比，运行这段代码将花费多少时间？

第二，*你确信已经完成代码编写了吗？* 过早的优化就像是在一块半生不熟的蛋糕上撒糖霜。你花费了几小时、几天 (或更长) 时间来优化你的代码以提高性能，却发现它不能完成你希望它做的工作。那是浪费时间。

这并不是说代码优化毫无用处，但是你需要检查一下整个系统，并且确定把时间花在这上面是值得的。在优化代码上每花费一分钟，就意味着你少了增加新功能、编写文档或者陪你的孩子玩或者编写单元测试的一分钟。

哦，是的，单元测试。不必我说，在开始性能优化之前你需要一个完全的单元测试集。你最不需要的就是在乱动你的算法时引入新的问题。

谨记着这些忠告，让我们来看一些优化 Python 代码的技术。我们要研究的代码是 Soundex 算法的实现。Soundex 是一种 20 世纪在美国人口普查中归档姓氏的方法。它把听起来相似的姓氏归在一起，使得在即便错误拼写的情况下调查者仍能查找到。Soundex 今天仍然因差不多的原因被应用着，当然现在用计算机数据库服务器了。大部分的数据库服务器都有 Soundex 函数。

Soundex 算法有几个差别不大的变化版本。这是本章使用的：

1.  名字的第一个字母不变。
2.  根据特定的对照表，将剩下的字母转换为数字：

    *   B、 F、 P 和 V 转换为 1。
    *   C、 G、 J、 K、 Q、 S、 X 和 Z 转换为 2。
    *   D 和 T 转换为 3。
    *   L 转换为 4。
    *   M 和 N 转换为 5。
    *   R 转换为 6。
    *   所有其他字母转换为 9。
3.  去除连续重复。

4.  去除所有 9。
5.  如果结果都少于四个字符 (第一个字母加上后面的三位字符)，就以零补齐。
6.  如果结果超过四个字符，丢弃掉四位之后的字符。

比如，我的名字 `Pilgrim` 被转换为 P942695。没有连续重复，所以这一步不需要做。然后是去除 9，剩下 P4265。太长了，所以你把超出的字符丢弃，剩下 P426。

另一个例子：`Woo` 被转换为 W99，变成 W9，变成 W，然后以补零成为 W000。

这是 Soundex 函数的第一次尝试：

## 例 18.1\. `soundex/stage1/soundex1a.py`

如果您还没有下载本书附带的样例程序, 可以 [下载本程序和其他样例程序](http://www.woodpecker.org.cn/diveintopython/download/diveintopython-exampleszh-cn-5.4b.zip "Download example scripts")。

```py
 import string, re
charToSoundex = {"A": "9",
                 "B": "1",
                 "C": "2",
                 "D": "3",
                 "E": "9",
                 "F": "1",
                 "G": "2",
                 "H": "9",
                 "I": "9",
                 "J": "2",
                 "K": "2",
                 "L": "4",
                 "M": "5",
                 "N": "5",
                 "O": "9",
                 "P": "1",
                 "Q": "2",
                 "R": "6",
                 "S": "2",
                 "T": "3",
                 "U": "9",
                 "V": "1",
                 "W": "9",
                 "X": "2",
                 "Y": "9",
                 "Z": "2"}
def soundex(source):
    "convert string to Soundex equivalent"
    # Soundex requirements:
    # source string must be at least 1 character
    # and must consist entirely of letters
    allChars = string.uppercase + string.lowercase
    if not re.search('^[%s]+$' % allChars, source):
        return "0000"
    # Soundex algorithm:
    # 1\. make first character uppercase
    source = source[0].upper() + source[1:]
    # 2\. translate all other characters to Soundex digits
    digits = source[0]
    for s in source[1:]:
        s = s.upper()
        digits += charToSoundex[s]
    # 3\. remove consecutive duplicates
    digits2 = digits[0]
    for d in digits[1:]:
        if digits2[-1] != d:
            digits2 += d
    # 4\. remove all "9"s
    digits3 = re.sub('9', '', digits2)
    # 5\. pad end with "0"s to 4 characters
    while len(digits3) < 4:
        digits3 += "0"
    # 6\. return first 4 characters
    return digits3[:4]
if __name__ == '__main__':
    from timeit import Timer
    names = ('Woo', 'Pilgrim', 'Flingjingwaller')
    for name in names:
        statement = "soundex('%s')" % name
        t = Timer(statement, "from __main__ import soundex")
        print name.ljust(15), soundex(name), min(t.repeat()) 
```

## 进一步阅读

*   Soundexing and Genealogy 给出了 Soundex 发展的年代表以及地域变化。

# 18.2\. 使用 `timeit` 模块

# 18.2\. 使用 `timeit` 模块

关于 Python 代码优化你需要知道的最重要问题是，决不要自己编写计时函数。

为一个很短的代码计时都很复杂。处理器有多少时间用于运行这个代码？有什么在后台运行吗？每个现代计算机都在后台运行持续或者间歇的程序。小小的疏忽可能破坏你的百年大计，后台服务偶尔被 “唤醒” 在最后千分之一秒做一些像查收信件，连接计时通信服务器，检查应用程序更新，扫描病毒，查看是否有磁盘被插入光驱之类很有意义的事。在开始计时测试之前，把一切都关掉，断开网络的连接。再次确定一切都关上后关掉那些不断查看网络是否恢复的服务等等。

接下来是计时框架本身引入的变化因素。Python 解释器是否缓存了方法名的查找？是否缓存代码块的编译结果？正则表达式呢? 你的代码重复运行时有副作用吗？不要忘记，你的工作结果将以比秒更小的单位呈现，你的计时框架中的小错误将会带来不可挽回的结果扭曲。

Python 社区有句俗语：“Python 自己带着电池。” 别自己写计时框架。Python 2.3 具备一个叫做 `timeit` 的完美计时工具。

## 例 18.2\. `timeit` 介绍

如果您还没有下载本书附带的样例程序, 可以 [下载本程序和其他样例程序](http://www.woodpecker.org.cn/diveintopython/download/diveintopython-exampleszh-cn-5.4b.zip "Download example scripts")。

```py
>>> import timeit
>>> t = timeit.Timer("soundex.soundex('Pilgrim')",
... "import soundex")   
>>> t.timeit()              
8.21683733547
>>> t.repeat(3, 2000000)    
[16.48319309109, 16.46128984923, 16.44203948912] 
```

|  |  |
| --- | --- |
| [1] | `timeit` 模块定义了接受两个参数的 `Timer` 类。两个参数都是字符串。第一个参数是你要计时的语句，这里你计时的是以`'Pilgrim'`参数调用 Soundex 函数。传递给 `Timer` 的第二个参数是为第一个参数语句构建环境的导入语句。从内部讲，`timeit` 构建起一个独立的虚拟环境，手工地执行建立语句 (导入 `soundex` 模块)，然后手工地编译和执行被计时语句 (调用 Soundex 函数)。 |
| [2] | 只要有了 `Timer` 对象，最简单的事就是调用 `timeit()`，它调用你的函数一百万次并返回所耗费的秒数。 |
| [3] | `Timer` 对象的另一个主要方法是 `repeat()`，它接受两个可选参数。第一个参数是重复整个测试的次数，第二个参数是每个测试中调用被计时语句的次数。两个参数都是可选的，它们的默认值分别是 `3` 和 `1000000`。`repeat()` 方法返回以秒记录的每个测试循环的耗时列表。 |

> 提示
> 你可以在命令行使用 `timeit` 模块来测试一个已存在的 Python 程序，而不需要修改代码。在 [`docs.python.org/lib/node396.html`](http://docs.python.org/lib/node396.html) 查看文档中关于命令行选项的内容。

注意 `repeat()` 返回一个时间列表。由于 Python 计时器使用的处理器时间的微小变化 (或者那些你没办法根除的可恶的后台进程)，这些时间中几乎不可能出现重复。你的第一想法也许是说：“让我们求平均值获得真实的数据。”

事实上，那几乎是确定错误的。你的代码或者 Python 解释器的变化可能缩短耗时，那些没办法去除的可恶后台进程或者其他 Python 解释器以外的因素也许令耗时延长。如果计时结果之间的差异超过百分之几，太多的可变因素使你没法相信结果，如果不是这样则可以取最小值而丢弃其他结果。

Python 有一个方便的 `min` 函数返回输入列表中的最小值：

```py
>>> min(t.repeat(3, 1000000))
8.22203948912 
```

> 提示
> `timeit` 模块只有在你知道哪段代码需要优化时使用。如果你有一个很大的 Python 程序并且不知道你的性能问题所在，查看 [`hotshot` 模块](http://docs.python.org/lib/module-hotshot.html)。

# 18.3\. 优化正则表达式

# 18.3\. 优化正则表达式

Soundex 函数的第一件事是检查输入是否是一个空字符串。怎样做是最好的方法？

如果你回答 “正则表达式”，坐在角落里反省你糟糕的直觉。正则表达式几乎永远不是最好的答案，而且应该被尽可能避开。这不仅仅是基于性能考虑，而是因为调试和维护都很困难，当然性能也是个原因。

这是 `soundex/stage1/soundex1a.py` 检查 `source` 是否全部由字母构成的一段代码，至少是一个字母 (而不是空字符串)：

```py
 allChars = string.uppercase + string.lowercase
    if not re.search('^[%s]+$' % allChars, source):
        return "0000" 
```

`soundex1a.py` 表现如何？为了方便，`__main__` 部分包含了一段代码：调用 `timeit` 模块，为三个不同名字分别建立测试，依次测试，并显示每个测试的最短耗时：

```py
 if __name__ == '__main__':
    from timeit import Timer
    names = ('Woo', 'Pilgrim', 'Flingjingwaller')
    for name in names:
        statement = "soundex('%s')" % name
        t = Timer(statement, "from __main__ import soundex")
        print name.ljust(15), soundex(name), min(t.repeat()) 
```

那么，应用正则表达式的 `soundex1a.py` 表现如何呢？

```py
C:\samples\soundex\stage1>python soundex1a.py
Woo             W000 19.3356647283
Pilgrim         P426 24.0772053431
Flingjingwaller F452 35.0463220884 
```

正如你预料，名字越长，算法耗时就越长。有几个工作可以令我们减小这个差距 (使函数对于长输入花费较短的相对时间) 但是算法的本质决定它不可能每次运行时间都相同。

另一点应铭记于心的是，我们测试的是有代表性的名字样本。`Woo` 是个被缩短到单字符并补零的小样本；`Pilgrim` 是个夹带着特别字符和忽略字符的平均长度的正常样本；`Flingjingwaller` 是一个包含连续重复字符并且特别长的样本。其它的测试可能同样有帮助，但它们已经很好地代表了不同的样本范围。

那么那个正则表达式如何呢？嗯，缺乏效率。因为这个表达式测试不止一个范围的字符 (`A-Z` 的大写范围和 `a-z` 的小写字母范围)，我们可以使用一个正则表达式的缩写语法。这便是 `soundex/stage1/soundex1b.py`:

```py
 if not re.search('^[A-Za-z]+$', source):
        return "0000" 
```

`timeit` 显示 `soundex1b.py` 比 `soundex1a.py` 稍微快一些，但是没什么令人激动的变化：

```py
C:\samples\soundex\stage1>python soundex1b.py
Woo             W000 17.1361133887
Pilgrim         P426 21.8201693232
Flingjingwaller F452 32.7262294509 
```

在 第 15.3 节 “重构” 中我们看到正则表达式可以被编译并在重用时以更快速度获得结果。因为这个正则表达式在函数中每次被调用时都不变化，我们可以编译它一次并使用被编译的版本。这便是 `soundex/stage1/soundex1c.py`：

```py
isOnlyChars = re.compile('^[A-Za-z]+$').search
def soundex(source):
    if not isOnlyChars(source):
        return "0000" 
```

`soundex1c.py` 中使用被编译的正则表达式产生了显著的提速：

```py
C:\samples\soundex\stage1>python soundex1c.py
Woo             W000 14.5348347346
Pilgrim         P426 19.2784703084
Flingjingwaller F452 30.0893873383 
```

但是这样的优化是正路吗？这里的逻辑很简单：输入 `source` 应该是非空，并且需要完全由字母构成。如果编写一个循环查看每个字符并且抛弃正则表达式，是否会更快些？

这便是 `soundex/stage1/soundex1d.py`：

```py
 if not source:
        return "0000"
    for c in source:
        if not ('A' <= c <= 'Z') and not ('a' <= c <= 'z'):
            return "0000" 
```

这个技术在 `soundex1d.py` 中恰好*不及* 编译后的正则表达式快 (尽管比使用未编译的正则表达式快[14])：

```py
C:\samples\soundex\stage1>python soundex1d.py
Woo             W000 15.4065058548
Pilgrim         P426 22.2753567842
Flingjingwaller F452 37.5845122774 
```

为什么 `soundex1d.py` 没能更快？答案来自 Python 的编译本质。正则表达式引擎以 C 语言编写，被编译后则能本能地在你的计算机上运行。另一方面，循环是以 Python 编写，要通过 Python 解释器。尽管循环相对简单，但没能简单到补偿花在代码解释上的时间。正则表达式永远不是正确答案……但例外还是存在的。

恰巧 Python 提供了一个晦涩的字符串方法。你有理由不了解它，因为本书未曾提到它。这个方法便是 `isalpha()`，它检查一个字符串是否只包含字母。

这便是 `soundex/stage1/soundex1e.py`：

```py
 if (not source) and (not source.isalpha()):
        return "0000" 
```

在 `soundex1e.py` 中应用这个特殊方法我们能得到多少好处? 很多。

```py
C:\samples\soundex\stage1>python soundex1e.py
Woo             W000 13.5069504644
Pilgrim         P426 18.2199394057
Flingjingwaller F452 28.9975225902 
```

## 例 18.3\. 目前为止最好的结果：`soundex/stage1/soundex1e.py`

```py
 import string, re
charToSoundex = {"A": "9",
                 "B": "1",
                 "C": "2",
                 "D": "3",
                 "E": "9",
                 "F": "1",
                 "G": "2",
                 "H": "9",
                 "I": "9",
                 "J": "2",
                 "K": "2",
                 "L": "4",
                 "M": "5",
                 "N": "5",
                 "O": "9",
                 "P": "1",
                 "Q": "2",
                 "R": "6",
                 "S": "2",
                 "T": "3",
                 "U": "9",
                 "V": "1",
                 "W": "9",
                 "X": "2",
                 "Y": "9",
                 "Z": "2"}
def soundex(source):
    if (not source) and (not source.isalpha()):
        return "0000"
    source = source[0].upper() + source[1:]
    digits = source[0]
    for s in source[1:]:
        s = s.upper()
        digits += charToSoundex[s]
    digits2 = digits[0]
    for d in digits[1:]:
        if digits2[-1] != d:
            digits2 += d
    digits3 = re.sub('9', '', digits2)
    while len(digits3) < 4:
        digits3 += "0"
    return digits3[:4]
if __name__ == '__main__':
    from timeit import Timer
    names = ('Woo', 'Pilgrim', 'Flingjingwaller')
    for name in names:
        statement = "soundex('%s')" % name
        t = Timer(statement, "from __main__ import soundex")
        print name.ljust(15), soundex(name), min(t.repeat()) 
```

## Footnotes

[14] 注意 `soundex1d.py` 在后两个测试点上都比 `soundex1b.py` 慢，这点与作者所说的矛盾。本章另还有多处出现了正文与测试结果矛盾的地方，每个地方都会用译注加以说明。这个 bug 将在下个版本中得到修正。――译注

# 18.4\. 优化字典查找

# 18.4\. 优化字典查找

Soundex 算法的第二步是依照特定规则将字符转换为数字。做到这点最好的方法是什么？

最明显的解决方案是定义一个以单字符为键并以所对应数字为值的字典，以字典查找每个字符。这便是 `soundex/stage1/soundex1e.py` 中使用的方法 (目前最好的结果)：

```py
charToSoundex = {"A": "9",
                 "B": "1",
                 "C": "2",
                 "D": "3",
                 "E": "9",
                 "F": "1",
                 "G": "2",
                 "H": "9",
                 "I": "9",
                 "J": "2",
                 "K": "2",
                 "L": "4",
                 "M": "5",
                 "N": "5",
                 "O": "9",
                 "P": "1",
                 "Q": "2",
                 "R": "6",
                 "S": "2",
                 "T": "3",
                 "U": "9",
                 "V": "1",
                 "W": "9",
                 "X": "2",
                 "Y": "9",
                 "Z": "2"}
def soundex(source):
    # ... input check omitted for brevity ...
    source = source[0].upper() + source[1:]
    digits = source[0]
    for s in source[1:]:
        s = s.upper()
        digits += charToSoundex[s] 
```

你已经为 `soundex1e.py` 计时，这便是其表现：

```py
C:\samples\soundex\stage1>python soundex1c.py
Woo             W000 13.5069504644
Pilgrim         P426 18.2199394057
Flingjingwaller F452 28.9975225902 
```

这段代码很直接，但它是最佳解决方案吗？为每个字符分别调用 `upper()` 看起来不是很有效率，为整个字符串调用 `upper()` 一次可能会好些。

然后是一砖一瓦地建立 `digits` 字符串。一砖一瓦的建造好像非常欠缺效率。在 Python 内部，解释器需要在循环的每一轮创建一个新的字符串，然后丢弃旧的。

但是，Python 擅长于列表。可以自动地将字符串作为列表来对待。而且使用 `join()` 方法可以很容易地将列表合并成字符串。

这便是 `soundex/stage2/soundex2a.py`，通过 `map` 和 `lambda` 把所有字母转换为数字：

```py
 def soundex(source):
    # ...
    source = source.upper()
    digits = source[0] + "".join(map(lambda c: charToSoundex[c], source[1:])) 
```

太震惊了，`soundex2a.py` 并不快：

```py
C:\samples\soundex\stage2>python soundex2a.py
Woo             W000 15.0097526362
Pilgrim         P426 19.254806407
Flingjingwaller F452 29.3790847719 
```

匿名 `lambda` 函数的使用耗费掉了从以字符列表替代字符串争取来的时间。

`soundex/stage2/soundex2b.py` 使用了一个列表遍历来替代 `map` 和 `lambda`：

```py
 source = source.upper()
    digits = source[0] + "".join([charToSoundex[c] for c in source[1:]]) 
```

在 `soundex2b.py` 中使用列表遍历比 `soundex2a.py` 中使用 `map` 和 `lambda` 快，但还没有最初的代码快 (`soundex1e.py` 中一砖一瓦的构建字符串[15])：

```py
C:\samples\soundex\stage2>python soundex2b.py
Woo             W000 13.4221324219
Pilgrim         P426 16.4901234654
Flingjingwaller F452 25.8186157738 
```

是时候从本质不同的方法来思考了。字典查找是一个普通目的实现工具。字典的键可以是任意长度的字符串 (或者很多其他数据类型) 但这里我们只和单字符键*和* 单字符值打交道。恰巧 Python 有处理这种情况的特别函数：`string.maketrans` 函数。

这便是 `soundex/stage2/soundex2c.py`：

```py
allChar = string.uppercase + string.lowercase
charToSoundex = string.maketrans(allChar, "91239129922455912623919292" * 2)
def soundex(source):
    # ...
    digits = source[0].upper() + source[1:].translate(charToSoundex) 
```

这儿在干什么？`string.maketrans` 创建一个两个字符串间的翻译矩阵：第一参数和第二参数。就此而言，第一个参数是字符串 `ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz`，第二个参数是字符串 `9123912992245591262391929291239129922455912623919292`。看到其模式了？恰好与我们用冗长的字典构建的模式相同。A 映射到 9，B 映射到 1，C 映射到 2 等等。但它不是一个字典。而是一个你可以通过字符串方法 `translate` 使用的特别数据结构。它根据 `string.maketrans` 定义的矩阵将每个字符翻译为对应的数字。

`timeit` 显示 `soundex2c.py` 比定义字典并对输入进行循环一砖一瓦地构建输出快很多：

```py
C:\samples\soundex\stage2>python soundex2c.py
Woo             W000 11.437645008
Pilgrim         P426 13.2825062962
Flingjingwaller F452 18.5570110168 
```

你不可能做得更多了。Python 有一个特殊函数，通过使用它做到了一个和你的工作差不多的事情。就用它并继续吧！

## 例 18.4\. 目前的最佳结果：`soundex/stage2/soundex2c.py`

```py
 import string, re
allChar = string.uppercase + string.lowercase
charToSoundex = string.maketrans(allChar, "91239129922455912623919292" * 2)
def soundex(source):
    if (not source) or (not source.isalpha()):
        return "0000"
    digits = source[0].upper() + source[1:].translate(charToSoundex)
    digits2 = digits[0]
    for d in digits[1:]:
        if digits2[-1] != d:
            digits2 += d
    digits3 = re.sub('9', '', digits2)
    while len(digits3) < 4:
        digits3 += "0"
    return digits3[:4]
if __name__ == '__main__':
    from timeit import Timer
    names = ('Woo', 'Pilgrim', 'Flingjingwaller')
    for name in names:
        statement = "soundex('%s')" % name
        t = Timer(statement, "from __main__ import soundex")
        print name.ljust(15), soundex(name), min(t.repeat()) 
```

## Footnotes

[15] 事实恰好相反，`soundex2b.py` 在每个点上都快于 `soundex1e.py`。――译注

# 18.5\. 优化列表操作

# 18.5\. 优化列表操作

Soundex 算法的第三步是去除连续重复字符。怎样做是最佳方法？

这里是我们目前在 `soundex/stage2/soundex2c.py` 中的代码：

```py
 digits2 = digits[0]
    for d in digits[1:]:
        if digits2[-1] != d:
            digits2 += d 
```

这里是 `soundex2c.py` 的性能表现：

```py
C:\samples\soundex\stage2>python soundex2c.py
Woo             W000 11.437645008
Pilgrim         P426 13.2825062962
Flingjingwaller F452 18.5570110168 
```

第一件事是考虑，考察在循环的每一轮都检查 `digits[-1]` 是否有效率。列表索引代价大吗？如果把上一个数字存在另外的变量中以便检查是否会获益？

这里的 `soundex/stage3/soundex3a.py` 将回答这个问题：

```py
 digits2 = ''
    last_digit = ''
    for d in digits:
        if d != last_digit:
            digits2 += d
            last_digit = d 
```

`soundex3a.py` 并不比 `soundex2c.py` 运行得快多少，而且甚至可能更会慢些 (差异还没有大到可以确信这一点)：

```py
C:\samples\soundex\stage3>python soundex3a.py
Woo             W000 11.5346048171
Pilgrim         P426 13.3950636184
Flingjingwaller F452 18.6108927252 
```

为什么 `soundex3a.py` 不更快呢？其实 Python 的索引功能恰恰很有效。重复使用 `digits2[-1]` 根本没什么问题。另一方面，手工保留上一个数字意味着我们每存储一个数字都要为*两个* 变量赋值，这便抹杀了我们避开索引查找所带来的微小好处。

让我们从本质上不同的方法来思考。如果可以把字符串当作字符列表来对待，那么使用列表遍历遍寻列表便成为可能。问题是代码需要使用列表中的上一个字符，而且使用列表遍历做到这一点并不容易。

但是，使用内建的 `range()` 函数创建一个索引数字构成的列表是可以的。使用这些索引数字一步步搜索列表并拿出与前面不同的字符。这样将使你得到一个字符串列表，使用字符串方法 `join()` 便可重建字符串。

这便是 `soundex/stage3/soundex3b.py`：

```py
 digits2 = "".join([digits[i] for i in range(len(digits))
                       if i == 0 or digits[i-1] != digits[i]]) 
```

这样快了吗？一个字，否。

```py
C:\samples\soundex\stage3>python soundex3b.py
Woo             W000 14.2245271396
Pilgrim         P426 17.8337165757
Flingjingwaller F452 25.9954005327 
```

有可能因为目前的这些方法都是 “字符串中心化” 的。Python 可以通过一个命令把一个字符串转化为一个字符列表：`list('abc')` 返回 `['a', 'b', 'c']`。更进一步，列表可以被很快地*就地* 改变。与其一砖一瓦地建造一个新的列表 (或者字符串)，为什么不选择操作列表的元素呢？

这便是 `soundex/stage3/soundex3c.py`，就地修改列表去除连续重复元素：

```py
 digits = list(source[0].upper() + source[1:].translate(charToSoundex))
    i=0
    for item in digits:
        if item==digits[i]: continue
        i+=1
        digits[i]=item
    del digits[i+1:]
    digits2 = "".join(digits) 
```

这比 `soundex3a.py` 或 `soundex3b.py` 快吗？不，实际上这是目前最慢的一种方法[16]：

```py
C:\samples\soundex\stage3>python soundex3c.py
Woo             W000 14.1662554878
Pilgrim         P426 16.0397885765
Flingjingwaller F452 22.1789341942 
```

我们在这儿除了试用了几种 “聪明” 的技术，根本没有什么进步。到目前为止最快的方法就是最直接的原始方法 (`soundex2c.py`)。有时候聪明未必有回报。

## 例 18.5\. 目前的最佳结果：`soundex/stage2/soundex2c.py`

```py
 import string, re
allChar = string.uppercase + string.lowercase
charToSoundex = string.maketrans(allChar, "91239129922455912623919292" * 2)
def soundex(source):
    if (not source) or (not source.isalpha()):
        return "0000"
    digits = source[0].upper() + source[1:].translate(charToSoundex)
    digits2 = digits[0]
    for d in digits[1:]:
        if digits2[-1] != d:
            digits2 += d
    digits3 = re.sub('9', '', digits2)
    while len(digits3) < 4:
        digits3 += "0"
    return digits3[:4]
if __name__ == '__main__':
    from timeit import Timer
    names = ('Woo', 'Pilgrim', 'Flingjingwaller')
    for name in names:
        statement = "soundex('%s')" % name
        t = Timer(statement, "from __main__ import soundex")
        print name.ljust(15), soundex(name), min(t.repeat()) 
```

## Footnotes

[16] `soundex3c.py` 比 `soundex3b.py` 快。――译注

# 18.6\. 优化字符串操作

# 18.6\. 优化字符串操作

Soundex 算法的最后一步是对短结果补零和截短长结果。最佳的做法是什么？

这是目前在 `soundex/stage2/soundex2c.py` 中的做法：

```py
 digits3 = re.sub('9', '', digits2)
    while len(digits3) < 4:
        digits3 += "0"
    return digits3[:4] 
```

这里是 `soundex2c.py` 的表现：

```py
C:\samples\soundex\stage2>python soundex2c.py
Woo             W000 12.6070768771
Pilgrim         P426 14.4033353401
Flingjingwaller F452 19.7774882003 
```

思考的第一件事是以循环取代正则表达式。这里的代码来自 `soundex/stage4/soundex4a.py`：

```py
 digits3 = ''
    for d in digits2:
        if d != '9':
            digits3 += d 
```

`soundex4a.py` 快了吗？是的：

```py
C:\samples\soundex\stage4>python soundex4a.py
Woo             W000 6.62865531792
Pilgrim         P426 9.02247576158
Flingjingwaller F452 13.6328416042 
```

但是，等一下。一个从字符串去除字符的循环？我们可以用一个简单的字符串方法做到。这便是 `soundex/stage4/soundex4b.py`：

```py
 digits3 = digits2.replace('9', '') 
```

`soundex4b.py` 快了吗？这是个有趣的问题，它取决输入值：

```py
C:\samples\soundex\stage4>python soundex4b.py
Woo             W000 6.75477414029
Pilgrim         P426 7.56652144337
Flingjingwaller F452 10.8727729362 
```

`soundex4b.py` 中的字符串方法对于大多数名字比循环快，但是对于短小的情况 (很短的名字) 却比 `soundex4a.py` 略微慢些。性能优化并不总是一致的，对于一个情况快些，却可能对另外一些情况慢些。就此而言，大多数情况将会从改变中获益，所以就改吧，但是别忘了原则。

最后仍很重要的是，让我们检测算法的最后两步：以零补齐短结果和截短超过四字符的长结果。你在 `soundex4b.py` 中看到的代码就是做这个工作的，但是太没效率了。看一下 `soundex/stage4/soundex4c.py` 找出原因：

```py
 digits3 += '000'
    return digits3[:4] 
```

我们为什么需要一个 `while` 循环来补齐结果？我们早就知道我们需要把结果截成四字符，并且我们知道我们已经有了至少一个字符 (直接从 `source` 中拿过来的起始字符)。这意味着我们可以仅仅在输出的结尾添加三个零，然后截断它。不要害怕重新理解问题，从不太一样的角度看问题可以获得简单的解决方案。

我们丢弃 `while` 循环后从 `soundex4c.py` 中获得怎样的速度？太明显了：

```py
C:\samples\soundex\stage4>python soundex4c.py
Woo             W000 4.89129791636
Pilgrim         P426 7.30642134685
Flingjingwaller F452 10.689832367 
```

最后，还有一件事可以令这三行运行得更快：你可以把它们合并为一行。看一眼 `soundex/stage4/soundex4d.py`：

```py
 return (digits2.replace('9', '') + '000')[:4] 
```

在 `soundex4d.py` 中把所有代码放在一行可以比 `soundex4c.py` 稍微快那么一点：

```py
C:\samples\soundex\stage4>python soundex4d.py
Woo             W000 4.93624105857
Pilgrim         P426 7.19747593619
Flingjingwaller F452 10.5490700634 
```

它非常难懂，而且优化也不明显。这值得吗？我希望你有很好的见解。性能并不是一切。你在优化方面的努力应该与程序的可读性和可维护性相平衡。

# 18.7\. 小结

# 18.7\. 小结

这一章展示了性能优化的几个重要方面，这里是就 Python 而言，但它们却普遍适用。

*   如果你要在正则表达式和编写循环间抉择，选择正则表达式。正则表达式因其是以 C 语言编译的可以本能地在你的计算机上运行，你的循环却以 Python 编写需要通过 Python 解释器运行。
*   如果你需要在正则表达式和字符串方法间抉择，选择字符串方法。它们都是以 C 编译的，所以选取简单的。
*   字典查找的通常应用很快，但是 `string.maketrans` 之类的特殊函数和 `isalpha()` 之类的字符串方法更快。如果 Python 有定制方法给你用，就使它吧！
*   别太聪明了。有时一些明显的算法是最快的。
*   不要太迷恋性能优化，性能并不是一切。

最后一点太重要了，这章中你令这个程序提速三倍并且令百万次的调用节省 20 秒。太棒了！现在思考一下：在那百万次的函数调用中，有多少秒花在周边应用程序等待数据库连接？花在磁盘输入/输出上？花在等待用户输入上？不要在过度优化算法上花时间，从而忽略了其它地方可以做的明显改进。开发你编写运行良好的 Python 代码的直觉，如果发现明显的失误则修正它们，并不对其它部分过分操作。