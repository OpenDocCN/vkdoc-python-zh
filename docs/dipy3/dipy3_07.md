# Chapter 4 字符串

# Chapter 4 字符串

> " I’m telling you this ’cause you’re one of my friends. My alphabet starts where your alphabet ends! " — Dr. Seuss, On Beyond Zebra!

## 在开始之前需要掌握的一些知识

你是否知道 [Bougainville](http://en.wikipedia.org/wiki/Bougainville_Province) 人有世界上最小的字母表？他们的 [Rotokas 字母表](http://en.wikipedia.org/wiki/Rotokas_alphabet)只包含了 12 个字母： A, E, G, I, K, O, P, R, S, T, U, 和 V。另一方面，像汉语，日语和韩语这些语言，它们则有成千上万个字符。当然啦，英语共有 26 个字母 — 如果把大写和小写分别计算的话，52 个 — 外加少量的标点符号，比如*!@#$%&*

当人们说起“文本”，他们通常指显示在屏幕上的字符或者其他的记号；但是计算机不能直接处理这些字符和标记；它们只认识位(bit)和字节(byte)。实际上，从屏幕上的每一块文本都是以某种*字符编码(character encoding)*的方式保存的。粗略地说就是，字符编码提供一种映射，使屏幕上显示的内容和内存、磁盘内存储的内容对应起来。有许多种不同的字符编码，有一些是为特定的语言，比如俄语、中文或者英语，设计、优化的，另外一些则可以用于多种语言的编码。

在实际操作中则会比上边描述的更复杂一些。许多字符在几种编码里是共用的，但是在实际的内存或者磁盘上，不同的编码方式可能会使用不同的字节序列来存储他们。所以，你可以把字符编码当做一种解码密钥。当有人给你一个字节序列 — 文件，网页，或者别的什么 — 并且告诉你它们是“文本”时，就需要知道他们使用了何种编码方式，然后才能将这些字节序列解码成字符。如果他们给的是错误的“密钥”或者根本没有给你“密钥”，那就得自己来破解这段编码，这可是一个艰难的任务。有可能你使用了错误的解码方式，然后出现一些莫名其妙的结果。

你所了解的关于字符串的知识都是错的。

你肯定见过这样的网页，在撇号(`'`)该出现的地方被奇怪的像问号的字符替代了。这种情况通常意味着页面的作者没有正确的声明其使用的编码方式，浏览器只能自己来猜测，结果就是一些正确的和意料之外的字符的混合体。如果原文是英语，那只是不方便阅读而已；在其他的语言环境下，结果可能是完全不可读的。

现有的字符编码各类给世界上每种主要的语言都提供了编码方案。由于每种语言的各不相同，而且在以前内存和硬盘都很昂贵，所以每种字符编码都为特定的语言做了优化。上边这句话的意思是，每种编码都使用数字(0–255)来代表这种语言的字符。比如，你也许熟悉 ASCII 编码，它将英语中的字符都当做从 0–127 的数字来存储。（65 表示大写的“A”，97 表示小写的“a”，_&_c。）英语的字母表很简单，所以它能用不到 128 个数字表达出来。如果你懂得 2 进制计数的话，它只使用了一个字节内的 7 位。

西欧的一些语言，比如法语，西班牙语和德语等，比英语有更多的字母。或者，更准确的说，这些语言含有与变音符号(diacritical marks)组合起来的字母，像西班牙语里的`ñ`。这些语言最常用的编码方式是 CP-1252，又叫做“windows-1252”，因为它在微软的视窗操作系统上被广泛使用。CP-1252 和 ASCII 在 0–127 这个范围内的字符是一样的，但是 CP-1252 为`ñ`(n-with-a-tilde-over-it, 241)，`Ü`(u-with-two-dots-over-it, 252)这类字符而扩展到了 128–255 这个范围。然而，它仍然是一种单字节的编码方式；可能的最大数字为 255，这仍然可以用一个字节来表示。

然而，像中文，日语和韩语等语言，他们的字符如此之多而不得不需要多字节编码的字符集。即，使用两个字节的数字(0–255)代表每个“字符”。但是就跟不同的单字节编码方式一样，多字节编码方式之间也有同样的问题，即他们使用的数字是相同的，但是表达的内容却不同。相对于单字节编码方式它们只是使用的数字范围更广一些，因为有更多的字符需要表示。

在没有网络的时代，“文本”由自己输入，偶尔才会打印出来，大多数情况下使用以上的编码方案是可行的。那时没有太多的“纯文本”。源代码使用 ASCII 编码，其他人也都使用字处理器，这些字处理器定义了他们自己的格式（非文本的），这些格式会连同字符编码信息和风格样式一起记录其中，_&_c。人们使用与原作者相同的字处理软件读取这些文档，所以或多或少地能够使用。

现在，我们考虑一下像 email 和 web 这样的全球网络的出现。大量的“纯文本”文件在全球范围内流转，它们在一台电脑上被撰写出来，通过第二台电脑进行传输，最后在另外一台电脑上显示。计算机只能识别数字，但是这些数字可能表达的是其他的东西。Oh no! 怎么办呢。。好吧，那么系统必须被设计成在每一段“纯文本”上都搭载编码信息。记住，编码方式是将计算机可读的数字映射成人类可读的字符的解码密钥。失去解码密钥则意味着混乱不清的，莫名其妙的信息，或者更糟。

现在我们考虑尝试把多段文本存储在同一个地方，比如放置所有收到邮件的数据库。这仍然需要对每段文本存储其相关的字符编码信息，只有这样才能正确地显示它们。这很困难吗？试试搜索你的 email 数据库，这意味着需要在运行时进行编码之间的转换。很有趣是吧…

现在我们来分析另外一种可能性，即多语言文档，同一篇文档里来自几种不同语言的字符混在一起。（提示：处理这样文档的程序通常使用转义符在不同的“模式(modes)”之间切换。噗！现在是俄语 koi8-r 模式，所以 241 代表 Я；噗噗！现在到了 Mac Greek 模式，所以 241 代表 ώ。）当然，你也会想要搜索*这些*文档。

现在，你就哭吧，因为以前所了解的关于字符串的知识都是错的，根本就没有所谓的“纯文本”。

## Unicode

*Unicode 入门。*

Unicode 编码系统为表达*任意*语言的*任意*字符而设计。它使用 4 字节的数字来表达每个字母、符号，或者表意文字(ideograph)。每个数字代表唯一的至少在某种语言中使用的符号。（并不是所有的数字都用上了，但是总数已经超过了 65535，所以 2 个字节的数字是不够用的。）被几种语言共用的字符通常使用相同的数字来编码，除非存在一个在理的语源学(etymological)理由使不这样做。不考虑这种情况的话，每个字符对应一个数字，每个数字对应一个字符。即不存在二义性。不再需要记录“模式”了。`U+0041`总是代表`'A'`，即使这种语言没有`'A'`这个字符。

初次面对这个创想，它看起来似乎很伟大。一种编码方式即可解决所有问题。文档可包含多种语言。不再需要在各种编码方式之间进行“模式转换“。但是很快，一个明显的问题跳到我们面前。4 个字节？只为了单独一个字符‽ 这似乎太浪费了，特别是对像英语和西语这样的语言，他们只需要不到 1 个字节即可以表达所需的字符。事实上，对于以象形为基础的语言（比如中文）这种方法也有浪费，因为这些语言的字符也从来不需要超过 2 个字节即可表达。

有一种 Unicode 编码方式每 1 个字符使用 4 个字节。它叫做 UTF-82，因为 32 位 = 4 字节。UTF-32 是一种直观的编码方式；它收录每一个 Unicode 字符（4 字节数字）然后就以那个数字代表该字符。这种方法有其优点，最重要的一点就是可以在常数时间内定位字符串里的第`N`个字符，因为第`N`个字符从第`4×Nth`个字节开始。另外，它也有其缺点，最明显的就是它使用 4 个“诡异”的字节来存储每个“诡异”的字符…

尽管有 Unicode 字符非常多，但是实际上大多数人不会用到超过前 65535 个以外的字符。因此，就有了另外一种 Unicode 编码方式，叫做 UTF-16(因为 16 位 = 2 字节)。UTF-16 将 0–65535 范围内的字符编码成 2 个字节，如果真的需要表达那些很少使用的[“星芒层(astral plane)](http://en.wikipedia.org/wiki/Astral_character)”内超过这 65535 范围的 Unicode 字符，则需要使用一些诡异的技巧来实现。UTF-16 编码最明显的优点是它在空间效率上比 UTF-32 高两倍，因为每个字符只需要 2 个字节来存储（除去 65535 范围以外的），而不是 UTF-32 中的 4 个字节。并且，如果我们假设某个字符串不包含任何星芒层中的字符，那么我们依然可以在常数时间内找到其中的第`N`个字符，直到它不成立为止这总是一个不错的推断…

但是对于 UTF-32 和 UTF-16 编码方式还有一些其他不明显的缺点。不同的计算机系统会以不同的顺序保存字节。这意味着字符`U+4E2D`在 UTF-16 编码方式下可能被保存为`4E 2D`或者`2D 4E`，这取决于该系统使用的是大尾端(big-endian)还是小尾端(little-endian)。（对于 UTF-32 编码方式，则有更多种可能的字节排列。）只要文档没有离开你的计算机，它还是安全的 — 同一台电脑上的不同程序使用相同的字节顺序(byte order)。但是当我们需要在系统之间传输这个文档的时候，也许在万维网中，我们就需要一种方法来指示当前我们的字节是怎样存储的。不然的话，接收文档的计算机就无法知道这两个字节`4E 2D`表达的到底是`U+4E2D`还是`U+2D4E`。

为了解决*这个*问题，多字节的 Unicode 编码方式定义了一个“字节顺序标记(Byte Order Mark)”，它是一个特殊的非打印字符，你可以把它包含在文档的开头来指示你所使用的字节顺序。对于 UTF-16，字节顺序标记是`U+FEFF`。如果收到一个以字节`FF FE`开头的 UTF-16 编码的文档，你就能确定它的字节顺序是单向的(one way)的了；如果它以`FE FF`开头，则可以确定字节顺序反向了。

不过，UTF-16 还不够完美，特别是要处理许多 ASCII 字符时。如果仔细想想的话，甚至一个中文网页也会包含许多的 ASCII 字符 — 所有包围在可打印中文字符周围的元素(element)和属性(attribute)。能够在常数时间内找到第`Nth`个字符当然非常好，但是依然存在着纠缠不休的星芒层字符的问题，这意味着你不能*保证*每个字符都是 2 个字节长，所以，除非你维护着另外一个索引，不然就不能*真正意义上的*在常数时间内定位第`N`个字符。另外，朋友，世界上肯定还存在很多的 ASCII 文本…

另外一些人琢磨着这些问题，他们找到了一种解决方法：

UTF-8 The range of integers used to code the abstract characters is called the codespace. A particular integer in this set is called a code point. When an abstract character is mapped or assigned to a particular code point in the codespace, it is then referred to as an encoded character. <-->

UTF-8 是一种为 Unicode 设计的*变长(variable-length)*编码系统。即，不同的字符可使用不同数量的字节编码。对于 ASCII 字符(A-Z, _&_c.)UTF-8 仅使用 1 个字节来编码。事实上，UTF-8 中前 128 个字符(0–127)使用的是跟 ASCII 一样的编码方式。像ñ和ö这样的“扩展拉丁字符(Extended Latin)”则使用 2 个字节来编码。（这里的字节并不是像 UTF-16 中那样简单的 Unicode 编码点(unicode code point)；它使用了一些位变换(bit-twiddling)。）中文字符比如“中”则占用了 3 个字节。很少使用的“星芒层字符”则占用 4 个字节。

缺点：因为每个字符使用不同数量的字节编码，所以寻找串中第`N`个字符是一个 O(N)复杂度的操作 — 即，串越长，则需要更多的时间来定位特定的字符。同时，还需要位变换来把字符编码成字节，把字节解码成字符。

优点：在处理经常会用到的 ASCII 字符方面非常有效。在处理扩展的拉丁字符集方面也不比 UTF-16 差。对于中文字符来说，比 UTF-32 要好。同时，（在这一条上你得相信我，因为我不打算给你展示它的数学原理。）由位操作的天性使然，使用 UTF-8 不再存在字节顺序的问题了。一份以 UTF-8 编码的文档在不同的计算机之间是一样的比特流。

## 概述

在 Python 3，所有的字符串都是使用 Unicode 编码的字符序列。不再存在以 UTF-8 或者 CP-1252 编码的情况。也就是说，“这个字符串是以 UTF-8 编码的吗？不再是一个有效问题。”UTF-8 是一种将字符编码成字节序列的方式。如果需要将字符串转换成特定编码的字节序列，Python 3 可以为你做到。如果需要将一个字节序列转换成字符串，Python 3 也能为你做到。字节即字节，并非字符。字符在计算机内只是一种抽象。字符串则是一种抽象的序列。

```py
 9

'深'

'深入 Python 3' 
```

1.  为了创建一个字符串，将其用引号包围。Python 字符串可以通过单引号(`'`)或者双引号(`"`)来定义。
2.  内置函数`len()`可返回字符串的长度，*即*字符的个数。这与获得列表，元组，集合或者字典的长度的函数是同一个。Python 中，字符串可以想像成由字符组成的元组。
3.  Just like getting individual items out of a list, you can get individual characters out of a string using index notation. 与取得列表中的元素一样，也可以通过下标记号取得字符串中的某个字符。
4.  类似列表，可以使用`+`操作符来连接(concatenate)字符串。

## 格式化字符串

字符串可以使用单引号或者双引号来定义。

我们再来看一看`humansize.py`：

```py
 1024: ['KiB', 'MiB', 'GiB', 'TiB', 'PiB', 'EiB', 'ZiB', 'YiB']}

def approximate_size(size, a_kilobyte_is_1024_bytes=True):

    Keyword arguments:
    size -- file size in bytes
    a_kilobyte_is_1024_bytes -- if True (default), use multiples of 1024
                                if False, use multiples of 1000

    Returns: string

    if size < 0:

    multiple = 1024 if a_kilobyte_is_1024_bytes else 1000
    for suffix in SUFFIXES[multiple]:
        size /= multiple
        if size < multiple:

    raise ValueError('number too large') 
```

1.  `'KB'`, `'MB'`, `'GB'`… 这些是字符串。
2.  函数的文档字符串(docstring)也是字符串。当前的文档字符串占用了多行，所以它使用了相邻的 3 个引号来标记字符串的起始和终止。
3.  这 3 个引号代表该文档字符串的终止。
4.  这是另外一个字符串，作为一个可读的提示信息传递给异常。
5.  瓦哦…那是什么？

Python 3 支持把值格式化(format)成字符串。可以有非常复杂的表达式，最基本的用法是使用单个占位符(placeholder)将一个值插入字符串。

```py
>>> username = 'mark'

"mark's password is PapayaWhip" 
```

1.  不，`PapayaWhip`真的不是我的密码。
2.  这里包含了很多知识。首先，这里使用了一个字符串字面值的方法调用。*字符串也是对象*，对象则有其方法。其次，整个表达式返回一个字符串。最后，`{0}`和`{1}` 叫做*替换字段(replacement field)*，他们会被传递给`format()`方法的参数替换。

### 复合字段名

在前一个例子中，替换字段只是简单的整数，这是最简单的用法。整型替换字段被当做传给`format()`方法的参数列表的位置索引。即，`{0}`会被第一个参数替换（在此例中即`username`），`{1}`被第二个参数替换（`password），_&_c。可以有跟参数一样多的替换字段，同时你也可以使用任意多个参数来调用`format()`。但是替换字段远比这个强大。`

```py
>>> import humansize

>>> si_suffixes
['KB', 'MB', 'GB', 'TB', 'PB', 'EB', 'ZB', 'YB']

'1000KB = 1MB' 
```

1.  不需要调用`humansize`模块定义的任何函数我们就可以抓取到其所定义的数据结构：国际单位制(SI, 来自法语 Système International)的后缀列表（以 1000 为进制）。
2.  这一句看上去有些复杂，其实不是这样的。`{0}`代表传递给`format()`方法的第一个参数，即`si_suffixes`。注意`si_suffixes`是一个列表。所以`{0[0]}`指代`si_suffixes`的第一个元素，即`'KB'`。同时，`{0[1]}`指代该列表的第二个元素，即：`'MB'`。大括号以外的内容 — 包括`1000`，等号，还有空格等 — 则按原样输出。语句最后返回字符串为`'1000KB = 1MB'`。

{0}会被 format()的第 1 个参数替换，{1}则被其第 2 个参数替换。

这个例子说明*格式说明符可以通过利用（类似）Python 的语法访问到对象的元素或属性*。这就叫做*复合字段名(compound field names)*。以下复合字段名都是“有效的”。

*   使用列表作为参数，并且通过下标索引来访问其元素（跟上一例类似）
*   使用字典作为参数，并且通过键来访问其值
*   使用模块作为参数，并且通过名字来访问其变量及函数
*   使用类的实例作为参数，并且通过名字来访问其方法和属性
*   *以上方法的任意组合*

为了使你确信的确如此，下面这个样例就组合使用了上面所有方法：

```py
>>> import humansize
>>> import sys
>>> '1MB = 1000{0.modules[humansize].SUFFIXES[1000][0]}'.format(sys)
'1MB = 1000KB' 
```

下面是描述它如何工作的：

*   `sys`模块保存了当前正在运行的 Python 实例的信息。由于已经导入了这个模块，因此可以将其作为`format()`方法的参数。所以替换域`{0}`指代`sys`模块。
*   `sys.modules` is a dictionary of all the modules that have been imported in this Python instance. The keys are the module names as strings; the values are the module objects themselves. So the replacement field `{0.modules}` refers to the dictionary of imported modules. `sys.modules`是一个保存当前 Python 实例中所有已经导入模块的字典。模块的名字作为字典的键；模块自身则是键所对应的值。所以`{0.modules}`指代保存当前己被导入模块的字典。
*   `sys.modules['humansize']`即刚才导入的`humansize`模块。所以替换域`{0.modules[humansize]}`指代`humansize`模块。请注意以上两句在语法上轻微的不同。在实际的 Python 代码中，字典`sys.modules`的键是字符串类型的；为了引用它们，我们需要在模块名周围放上引号（*比如* `'humansize'`）。但是在使用替换域的时候，我们在省略了字典的键名周围的引号（*比如* `humansize`）。在此，我们引用[PEP 3101：字符串格式化高级用法](http://www.python.org/dev/peps/pep-3101/)，“解析键名的规则非常简单。如果名字以数字开头，则它被当作数字使用，其他情况则被认为是字符串。”
*   `sys.modules['humansize'].SUFFIXES`是在`humansize`模块的开头定义的一个字典对象。 `{0.modules[humansize].SUFFIXES}`即指向该字典。
*   `sys.modules['humansize'].SUFFIXES[1000]`是一个 SI（国际单位制）后缀列表：`['KB', 'MB', 'GB', 'TB', 'PB', 'EB', 'ZB', 'YB']`。所以替换域`{0.modules[humansize].SUFFIXES[1000]}`指向该列表。
*   `sys.modules['humansize'].SUFFIXES[1000][0]`即 SI 后缀列表的第一个元素：`'KB'`。因此，整个替换域`{0.modules[humansize].SUFFIXES[1000][0]}`最后都被两个字符`KB`替换。

### 格式说明符

但是，还有一些问题我们没有讲到！再来看一看`humansize.py`中那一行奇怪的代码：

```py
if size < multiple:
    return '{0:.1f} {1}'.format(size, suffix) 
```

`{1}`会被传递给`format()`方法的第二个参数替换，即`suffix`。但是`{0:.1f}`是什么意思呢？它其实包含了两方面的内容：`{0}`你已经能理解，`:.1f`则不一定了。第二部分（包括冒号及其后边的部分）即*格式说明符(format specifier)*，它进一步定义了被替换的变量应该如何被格式化。

> ☞格式说明符的允许你使用各种各种实用的方法来修饰被替换的文本，就像 C 语言中的`printf()`函数一样。我们可以添加使用零填充(zero-padding)，衬距(space-padding)，对齐字符串(align strings)，控制 10 进制数输出精度，甚至将数字转换成 16 进制数输出。

在替换域中，冒号(`:`)标示格式说明符的开始。“`.1`”的意思是四舍五入到保留一们小数点。“`f`”的意思是定点数（与指数标记法或者其他 10 进制数表示方法相对应）。因此，如果给定`size 为`698.24`，`suffix`为`'GB'`，那么格式化后的字符串将是`'698.2 GB'`，因为`698.24`被四舍五入到一位小数表示，然后后缀`'GB'`再被追加到这个串最后。`

```py
>>> '{0:.1f} {1}'.format(698.24, 'GB')
'698.2 GB' 
```

想了解格式说明符的复杂细节，请参阅 Python 官方文档[关于格式化规范的迷你语言](http://docs.python.org/3.1/library/string.html#format-specification-mini-language)

## 其他常用字符串方法

除了格式化，关于字符串还有许多其他实用的使用技巧。

```py
 ... sult of years of scientif-
... ic study combined with the
... experience of years.'''

['Finished files are the re-',
 'sult of years of scientif-',
 'ic study combined with the',
 'experience of years.']

finished files are the re-
sult of years of scientif-
ic study combined with the
experience of years.

6 
```

1.  我们可以在 Python 的交互式 shell 里输入多行(multiline)字符串。一旦我们以三个引号标记多行字符串的开始，按`ENTER`键，Python shell 会提示你继续这个字符串的输入。连续输入三个结束引号以终止该字符串的输入，再敲`ENTER`键则会执行该条命令（在当前例子中，把这个字符串赋给变量`s`）。
2.  `splitlines()`方法以多行字符串作为输入，返回一个由字符串组成的列表，列表的元素即原来的单行字符串。请注意，每行行末的回车符没有被包括进去。
3.  `lower()`方法把整个字符串转换成小写的。（类似地，`upper()`方法执行大写化转换操作。）
4.  `count()`方法对串中的指定的子串进行计数。是的，在那一句中确实出现了 6 个字母“f”。

还有一种经常会遇到的情况。比如有如下形式的键-值对列表 `key1`=`value1`&`key2`=`value2`，我们需要将其分离然后产生一个这样形式的字典`{key1: value1, key2: value2}`。

```py
>>> query = 'user=pilgrim&database=master&password=PapayaWhip'

>>> a_list
['user=pilgrim', 'database=master', 'password=PapayaWhip']

>>> a_list_of_lists
[['user', 'pilgrim'], ['database', 'master'], ['password', 'PapayaWhip']]

>>> a_dict
{'password': 'PapayaWhip', 'user': 'pilgrim', 'database': 'master'} 
```

1.  `split()`方法使用一个参数，即指定的分隔符，然后根据这个分隔符将串分离成一个字符串列表。此处，分隔符即字符“`&`”，它还可以是其他的内容。
2.  现在我们有了一个字符串列表，其中的每个串由三部分组成：键，等号和值。我们可以使用列表解析来遍历整个列表，然后利用第一个等号标记将每个字符串再分离成两个子串。（理论上，值也可以包含等号标记，如果执行`'key=value=foo'.split('=')`，那么我们会得到一个三元素列表`['key', 'value', 'foo']`。）
3.  最后，通过调用`dict()`函数 Python 会把那个包含列表的列表(list-of-lists)转换成字典对象。

> ☞上一个例子跟解析 URL 的请求参数(query parameters)很相似，但是真实的 URL 解析实际上比这个复杂得多。如果需要处理 URL 请求参数，我们最好使用[`urllib.parse.parse_qs()`](http://docs.python.org/3.1/library/urllib.parse.html#urllib.parse.parse_qs)函数，它可以处理一些不常见的边缘情况。

### 字符串的分片

定义一个字符串以后，我们可以截取其中的任意部分形成新串。这种操作被称作字符串的*分片(slice)*。字符串分片跟列表的分片(slicing lists)原理是一样的，从直观上也说得通，因为字符串本身就是一些字符序列。

```py
>>> a_string = 'My alphabet starts where your alphabet ends.'

'alphabet'

'alphabet starts where your alphabet en'

'My'

'My alphabet starts'

' where your alphabet ends.' 
```

1.  我们可以通过指定两个索引值来获得原字符串的一个“slice”。该操作的返回值是一个新串，依次包含了从原串中第一个索引位置开始，直到但是不包含第二个索引位置之间的所有字符。
2.  就像给列表做分片一样，我们也可以使用负的索引值来分片字符串。
3.  字符串的下标索引是从 0 开始的，所以`a_string[0:2]`会返回原字符串的前两个元素，从`a_string[0]`开始，直到但不包括`a_string[2]`。
4.  如果省略了第一个索引值，Python 会默认它的值为 0。所以`a_string[:18]`跟`a_string[0:18]`的效果是一样的，因为从 0 开始是被 Python 默认的。
5.  同样地，如果第 2 个索引值是原字符串的长度，那么我们也可以省略它。所以，在此处`a_string[18:]`跟`a_string[18:44]`的结果是一样的，因为这个串的刚好有 44 个字符。这种规则存在某种有趣的对称性。在这个由 44 个字符组成的串中，`a_string[:18]`会返回前 18 个字符，而`a_string[18:]`则会返回除了前 18 个字符以外字符串的剩余部分。事实上`a_string[:`n`]`总是会返回串的前`n`个字符，而`a_string[`n`:]`则会返回其余的部分，这与串的长度无关。

## String vs. Bytes

字节即字节；字符是一种抽象。一个不可变(immutable)的 Unicode 编码的字符序列叫做*string*。一串由 0 到 255 之间的数字组成的序列叫做*bytes*对象。

```py
 >>> by
b'abcde'

<class 'bytes'>

5

>>> by
b'abcde\xff'

6

97

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'bytes' object does not support item assignment 
```

1.  使用“byte 字面值”语法`b''`来定义`bytes`对象。byte 字面值里的每个字节可以是 ASCII 字符或者是从`\x00`到`\xff`编码了的 16 进制数。
2.  `bytes`对象的类型是`bytes`。
3.  跟列表和字符串一样，我们可以通过内置函数`len()`来获得`bytes`对象的长度。
4.  使用`+`操作符可以连接`bytes`对象。操作的结果是一个新的`bytes`对象。
5.  连接 5 个字节的和 1 个字节的`bytes`对象会返回一个 6 字节的`bytes`对象。
6.  一如列表和字符串，可以使用下标记号来获取`bytes`对象中的单个字节。对字符串做这种操作获得的元素仍为字符串，而对`bytes`对象做这种操作的返回值则为整数。确切地说，是 0–255 之间的整数。
7.  `bytes`对象是不可变的；我们不可以给单个字节赋上新值。如果需要改变某个字节，可以组合使用字符串的切片和连接操作(效果跟字符串是一样的)，或者我们也可以将`bytes`对象转换为`bytearray`对象。

```py
>>> by = b'abcd\x65'

>>> barr
bytearray(b'abcde')

5

>>> barr
bytearray(b'fbcde') 
```

1.  使用内置函数`bytearray()`来完成从`bytes`对象到可变的`bytearray`对象的转换。
2.  所有对`bytes`对象的操作也可以用在`bytearray`对象上。
3.  有一点不同的就是，我们可以使用下标标记给`bytearray`对象的某个字节赋值。并且，这个值必须是 0–255 之间的一个整数。

我们*决不应该*这样混用 bytes 和 strings。

```py
>>> by = b'd'
>>> s = 'abcde'

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: can't concat bytes to str

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: Can't convert 'bytes' object to str implicitly

1 
```

1.  不能连接`bytes`对象和字符串。他们两种不同的数据类型。
2.  也不允许针对字符串中`bytes`对象的出现次数进行计数，因为串里面根本没有`bytes`。字符串是一系列的字符序列。也许你是想要先把这些字节序列通过某种编码方式进行解码获得字符串，然后对该字符串进行计数？可以，但是需要显式地指明它。Python 3 不会隐含地将 bytes 转换成字符串，或者进行相反的操作。
3.  好巧啊…这一行代码刚好给我们演示了使用特定编码方式将`bytes`对象转换成字符串后该串的出现次数。

所以，这就是字符串与字节数组之间的联系了：`bytes`对象有一个`decode()`方法，它使用某种字符编码作为参数，然后依照这种编码方式将`bytes`对象转换为字符串，对应地，字符串有一个`encode()`方法，它也使用某种字符编码作为参数，然后依照它将串转换为`bytes`对象。在上一个例子中，解码的过程相对直观一些 — 使用 ASCII 编码将一个字节序列转换为字符串。同样的过程对其他的编码方式依然有效 — 传统的（非 Unicode）编码方式也可以，只要它们能够编码串中的所有字符。

```py
 >>> len(a_string)
9

>>> by
b'\xe6\xb7\xb1\xe5\x85\xa5 Python'
>>> len(by)
13

>>> by
b'\xc9\xee\xc8\xeb Python'
>>> len(by)
11

>>> by
b'\xb2`\xa4J Python'
>>> len(by)
11

>>> roundtrip
'深入 Python'
>>> a_string == roundtrip
True 
```

1.  `a_string`是一个字符串。它有 9 个字符。
2.  `by`是一个`bytes`对象。它有 13 个字节。它是通过`a_string`使用 UTF-8 编码而得到的一串字节序列。
3.  `by`还是一个`bytes`对象。它有 11 个字节。它是通过`a_string`使用[GB18030](http://en.wikipedia.org/wiki/GB_18030)编码而得到的一串字节序列。
4.  此时的`by`仍旧是一个`bytes`对象，由 11 个字节组成。它又是一种*完全不同的字节序列*，我们通过对`a_string`使用[Big5](http://en.wikipedia.org/wiki/Big5)编码得到。
5.  `roundtrip`是一个字符串，共有 9 个字符。它是通过对`by`使用 Big5 解码算法得到的一个字符序列。并且，从执行结果可以看出，`roundtrip`与`a_string`是完全一样的。

## 补充内容：Python 源码的编码方式

Python 3 会假定我们的源码 — *即*`.py`文件 — 使用的是 UTF-8 编码方式。

> ☞Python 2 里，`.py`文件默认的编码方式为 ASCII。Python 3 的[源码的默认编码方式为 UTF-8](http://www.python.org/dev/peps/pep-3120/)

如果想使用一种不同的编码方式来保存 Python 代码，我们可以在每个文件的第一行放置编码声明(encoding declaration)。以下声明定义`.py`文件使用 windows-1252 编码方式：

```py
# -*- coding: windows-1252 -*- 
```

从技术上说，字符编码的重载声明也可以放在第二行，如果第一行被类 UNIX 系统中的[hash-bang](http://en.wikipedia.org/wiki/Sha-bang)命令占用了。

```py
#!/usr/bin/python3
# -*- coding: windows-1252 -*- 
```

了解更多信息，请参阅[PEP 263: 指定 Python 源码的编码方式](http://www.python.org/dev/peps/pep-0263/)。

## 进一步阅读

关于 Python 中的 Unicode：

*   [Python Unicode HOWTO](http://docs.python.org/3.1/howto/unicode.html)
*   [Python 3 中的新鲜事: 文本 vs. 数据，而非 Unicode vs. 8-bit](http://docs.python.org/3.0/whatsnew/3.0.html#text-vs-data-instead-of-unicode-vs-8-bit)

关于 Unicode 本身：

*   [每个软件开发人员应该无条件、至少掌握的关于 Unicode 和字符集的知识](http://www.joelonsoftware.com/articles/Unicode.html)
*   [关于 Unicode 的优势](http://www.tbray.org/ongoing/When/200x/2003/04/06/Unicode)
*   [关于字元字串(character string)](http://www.tbray.org/ongoing/When/200x/2003/04/13/Strings)
*   [字符 vs. 字节](http://www.tbray.org/ongoing/When/200x/2003/04/26/UTF)

关于其他的编码方式：

*   [XML 文档的编码方式](http://feedparser.org/docs/character-encoding.html)
*   [HTML 文档的编码方式](http://blog.whatwg.org/the-road-to-html-5-character-encoding)

关于字符串及其格式化：

*   [`string` — 常用字符串操作](http://docs.python.org/3.1/library/string.html)
*   [格式化字符串的语法](http://docs.python.org/3.1/library/string.html#formatstrings)
*   [关于格式化规范的迷你语言](http://docs.python.org/3.1/library/string.html#format-specification-mini-language)
*   [PEP 3101: 字符串格式化高级应用](http://www.python.org/dev/peps/pep-3101/)

Updated October 7, 2009 • Difficulty level: ♦♦♦♢♢