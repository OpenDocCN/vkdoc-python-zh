# Chapter 1 你的第一个 Python 程序

> " Don’t bury your burden in saintly silence. You have a problem? Great. Rejoice, dive in, and investigate. " — [Ven. Henepola Gunaratana](http://en.wikiquote.org/wiki/Buddhism)

## Diving In

通常程序设计的书籍都会以一堆关于基础知识的章节开始，最终逐步的构建一些有用的东西。让我们跳过所有的那些东西，来看一个完整的、可以直接运行的 Python 程序。可能刚开始你根本看不懂，但不要担心，因为你会去一行一行的仔细研究。但是首先还是要通读一遍，看看里面什么东西（如果有的话）是你可以看懂的。

```py
SUFFIXES = {1000: ['KB', 'MB', 'GB', 'TB', 'PB', 'EB', 'ZB', 'YB'],
            1024: ['KiB', 'MiB', 'GiB', 'TiB', 'PiB', 'EiB', 'ZiB', 'YiB']}

def approximate_size(size, a_kilobyte_is_1024_bytes=True):
    '''Convert a file size to human-readable form.

    Keyword arguments:
    size -- file size in bytes
    a_kilobyte_is_1024_bytes -- if True (default), use multiples of 1024
                                if False, use multiples of 1000

    Returns: string

    '''
    if size < 0:
        raise ValueError('number must be non-negative')

    multiple = 1024 if a_kilobyte_is_1024_bytes else 1000
    for suffix in SUFFIXES[multiple]:
        size /= multiple
        if size < multiple:
            return '{0:.1f} {1}'.format(size, suffix)

    raise ValueError('number too large')

if __name__ == '__main__':
    print(approximate_size(1000000000000, False))
    print(approximate_size(1000000000000)) 
```

现在让我们从命令行来运行这个程序。在 Windows 上，类似这样：

```py
c:\home\diveintopython3\examples> c:\python31\python.exe humansize.py
1.0 TB
931.3 GiB 
```

在 Mac OS X 或者 Linux 上，类似这样：

```py
you@localhost:~/diveintopython3/examples$ python3 humansize.py
1.0 TB
931.3 GiB 
```

刚刚发生了什么？你执行了你的第一个 Python 程序。你从命令行调用了 Python 解释器，并且传递了一个你想 Python 去执行的脚本的名称。这个脚本定义了一个单一的函数，这个 `approximate_size()` 函数把一个精确到字节的文件大小计算成一个有漂亮格式（大约计算的）的大小。（你可能已经在 Windows Explorer，或者 Mac OS X Finder，或者 Linux 上的 Nautilus 或 Dolphin 或 Thunar 看到过这个。如果你按照多列的列表来显示一个文件夹的文档，它就会显示一个包含文档图标、文档名称、大小、类型、最后修改日期等等信息的表格。如果这个文件夹包含一个 1093 字节大小名叫 `TODO` 的文件，你的文件管理器将不会显示成 `TODO 1093 bytes`，而用 `TODO 1 KB` 的显示格式代替。那就是 `approximate_size()` 函数所做的事情。）

看看这个脚本的底部，你会看到对 `print(approximate_size(`arguments`))` 的两次调用。这些叫做函数调用 —— 第一个调用了 `approximate_size()` 函数并传递了一些参数，接着直接把返回值传递给了 `print()` 函数。这个 `print()` 函数是内置的，你将从不会看到它的一个显式的声明。你只管在需要的任何时候任何地方使用它就行。（有很多内置函数，更多的函数独立于各个 *modules* （模块）里面。保持耐心，你会逐步熟悉它们的。）

那么为什么每次在命令行运行脚本都会给你同样的输出结果呢？我们将讲解这个。首先，让我们来看一下 `approximate_size()` 函数。

## 声明函数

像多数其他语言一样， Python 也有函数， 但是它没有像 C++ 一样的单独头文件，也没有像 Pascal 一样 `interface`/`implementation` （接口／实现）部分。当你需要一个函数的时候，就像这样声明它就行：

```py
def approximate_size(size, a_kilobyte_is_1024_bytes=True): 
```

当你需要一个函数的时候，只要声明它就行。

函数声明以关键字 `def` 开头，紧跟着函数的名称，然后是用括号括起来的参数。多个参数以逗号分割。

同时注意，函数不定义一个返回数据类型。 Python 函数不指定它们的返回值的类型，甚至不指定它们是否返回一个值。（事实上，每个 Python 函数都返回一个值，如果这个函数曾经执行了 `return` 语句，它将返回那个值，否则它将返回 Python 里面的空值 `None`。）

> ☞在某些语言里面，函数（返回一个值）以 `function` 开头，同时子程序（不返回值的）以 `sub` 开头。 Python 里面没有子程序。所有的东西都是一个函数，所有的函数都返回一个值（即使它是 `None` 值），并且所有的函数都以 `def` 开头。

`approximate_size()` 函数有两个参数 — `size` 和 `a_kilobyte_is_1024_bytes` — 但都没有指定数据类型。在 Python 里面，变量从来不会显式的指定类型。Python 会在内部算出一个变量的类型并进行跟踪。

> ☞在 Java 和其他静态类型的语言里面，你必须给函数返回值和每个函数参数指定数据类性。而在 Python 里面，你从来不需要给任何东西指定显式的数据类型。根据你赋的值，Python 会在内部对数据类型进行跟踪。

### 可选的和命名的参数

Python 允许函数函数有默认值。如果函数被调用的时候没有指定参数，那么参数将使用默认值。不仅如此，通过使用命名参数还可以按照任何顺序指定参数。

让我们再看一下 `approximate_size()` 函数的声明：

```py
def approximate_size(size, a_kilobyte_is_1024_bytes=True): 
```

第二个参数 `a_kilobyte_is_1024_bytes` 指定了一个默认值 `True`。 意思是这个参数是 *optional （可选的）*，你可以在调用的时候不指定它，Python 将看成你调用的时候使用了 `True` 作为第二个参数。

现在看一下这个脚本的底部：

```py
if __name__ == '__main__': 
```

1.  这个对 `approximate_size()` 函数的调用指定了两个参数。在 `approximate_size()` 函数里面，`a_kilobyte_is_1024_bytes` 的值将为 `False`，因为你显式的传入了 `False` 作为第二个参数。
2.  这个对 `approximate_size()` 函数的调用只指定了一个参数。但这是可以的，因为第二个参数是可选的！由于调用者没有指定，第二个参数就会使用在函数声明的时候定义的默认值 `True`。

你也可以通过名称将值传入一个函数。

```py
>>> from humansize import approximate_size

'4.0 KB'

'4.0 KB'

'4.0 KB'

 File "<stdin>", line 1
SyntaxError: non-keyword arg after keyword arg

 File "<stdin>", line 1
SyntaxError: non-keyword arg after keyword arg 
```

1.  这个对 `approximate_size()` 函数的调用给第一个参数（(`size`）指定了值 `4000`，并且给名为 `a_kilobyte_is_1024_bytes` 的参数指定了值 `False`。（那碰巧是第二个参数，但这没有关系，马上你就会了解到。）
2.  这个对 `approximate_size()` 函数的调用给名为 `size` 参数指定了值 `4000`，并为名为 `a_kilobyte_is_1024_bytes` 的参数指定了值 `False`。（这些命名参数碰巧和函数声明时列出的参数顺序一样，但同样不要紧。）
3.  这个对 `approximate_size()` 函数的调用给名为 `a_kilobyte_is_1024_bytes` 的参数指定了值 `False`，然后给名为 `size` 的参数指定了值 `4000`。（看到了没？我告诉过你顺序没有关系。）
4.  这个调用会失败，因为你在命名参数后面紧跟了一个非命名（位置的）的参数，这个一定不会工作。从左到右的读取参数列表，一旦你有一个命名的参数，剩下的参数也必须是命名的。
5.  这个调用也会失败，和前面一个调用同样的原因。 是不是很惊讶？别忘了，你给名为 `size` 的参数传入了值 `4000`，那么“显然的” `False` 这个值意味着对应了 `a_kilobyte_is_1024_bytes` 参数。但是 Python 不按照这种方式工作。只要你有一个命名参数，它右边的所有参数也都需要是命名参数。

## 编写易读的代码

我不会长期指手划脚的来烦你，解释给你的代码添加文档注释的重要性。只要知道代码被编写一次但是会被阅读很多次，而且你的代码最要的读者就是你自己，在编写它的六个月以后（*例如*，当你忘记了所有的东西但是又需要去修正一些东西的时候）。 Python 使得编写易读的代码非常容易，因此要利用好这个优势。六个月以后你将会感谢我。

### 文档字符串

你可以通过使用一个文档字符串（简称 `docstring` ）的方式给 Python 添加文档注释。在这个程序中，这个 `approximate_size()` 函数有一个 `docstring`：

```py
def approximate_size(size, a_kilobyte_is_1024_bytes=True):
    '''Convert a file size to human-readable form.

    Keyword arguments:
    size -- file size in bytes
    a_kilobyte_is_1024_bytes -- if True (default), use multiples of 1024
                                if False, use multiples of 1000

    Returns: string

    ''' 
```

每个函数都值得有一个合适的 docstring （文档字符串）。

三重引号表示一个多行的字符串。在开始引号和结束引号之间的所有东西都属于一个单独的字符串的一部分，包括回车、前导空格、和其他引号字符。你可以在任何地方使用它们，但是你会发现大部分时候它们在定义 `docstring` （文档注释）的时候使用。

> ☞三重引号也是一种容易的方法，用来定义一个同时包含单引号和双引号的字符串，就像 Perl 5 里面的 `qq/.../` 一样。

三重引号之间的所有东西都是这个函数的 `docstring` （文档字符串），用来用文档描述这个函数是做什么的。一个 `docstring` （文档字符串），如果有的话，必须是一个函数里面定义的第一个东西（也就是说，紧跟着函数声明的下一行）。 你不需要严格的给你的每个函数提供一个 `docstring` （文档字符串），但大部分时候你总是应该提供。我知道你在曾经使用过的每一种程序语言里面听说过这个，但是 Python 给你提供了额外的诱因：这个 `docstring` （文档字符串）就像这个函数的一个属性一样在运行时有效。

> ☞很多 Python 的集成开发环境（IDE）使用 `docstring` （文档字符串）来提供上下文敏感的文档，以便于当你输入一个函数名称的时候，它的 `docstring` 会以一个提示文本的方式显式出来。这可能会极其有用，但它只有在你写出好的 `docstring` （文档字符串）的时候才有用。

## `import` 的搜索路径

在进一步讲解之前，我想简要的说一下库的搜索路径。当你试图导入（import）一个模块的时候，Python 会寻找几个地方。具体来说，它会搜寻在 `sys.path` 里面定义的所有目录。这只是一个列表，你可以容易地查看它或者使用标准的列表方法去修改它。（在内置数据类型你会了解更多关于列表的信息。）

```py
 ['', 
 '/usr/lib/python31.zip', 
 '/usr/lib/python3.1',
 '/usr/lib/python3.1/plat-linux2@EXTRAMACHDEPPATH@', 
 '/usr/lib/python3.1/lib-dynload', 
 '/usr/lib/python3.1/dist-packages', 
 '/usr/local/lib/python3.1/dist-packages']

<module 'sys' (built-in)>

['/home/mark/diveintopython3/examples', 
 '', 
 '/usr/lib/python31.zip', 
 '/usr/lib/python3.1', 
 '/usr/lib/python3.1/plat-linux2@EXTRAMACHDEPPATH@', 
 '/usr/lib/python3.1/lib-dynload', 
 '/usr/lib/python3.1/dist-packages', 
 '/usr/local/lib/python3.1/dist-packages'] 
```

1.  导入 `sys` 模块，使它的所有函数和属性可以被使用。
2.  `sys.path` 是一个目录名称的列表，它构成了当前的搜索路径。（你会看到不一样的结果，这取决于你的操作系统，你正在运行的 Python 的版本，以及它原来被安装的位置。） Python 会从头到尾的浏览这些目录（按照这个顺序），寻找一个和你正要导入的模块名称匹配的 `.py` 文件。
3.  其实，我说谎了。真实情况比那个更加复杂，因为不是所有的模块都按照 `.py` 文件来存储。有些，比如 `sys` 模块，属于内置模块（*built-in modules*）， 他们事实上被置入到 Python 本身里面了。 内置模块使用起来和常规模块一样，但是无法取得它们的 Python 源代码，因为它们不是用 Python 写的！（ `sys` 模块是用 C 语言写的。）
4.  通过添加一个目录名称到 `sys.path` 里，你可以在运行时添加一个新的目录到 Python 的搜索路径中，然后无论任何时候你想导入一个模块，Python 都会同样的去查找那个目录。只要 Python 在运行，都会一直有效。
5.  通过使用 `sys.path.insert(0,`new_path`)`，你可以插入一个新的目录到 `sys.path` 列表的第一项，从而使其出现在 Python 搜索路径的开头。这几乎总是你想要的。万一出现名字冲突（例如，Python 自带了版本 2 的一个特定的库，但是你想使用版本 3），这个方法就能确保你的模块能够被发现和使用，替代 Python 自带的版本。

## 一切都是对象

假如你还不了解，我重复一下，我刚刚说过 Python 函数有属性，并且那些属性在运行时是可用的。一个函数，就像 Python 里面所有其他东西一样，是一个对象。

运行交互式的 Python Shell，按照下面的执行：

```py
 4.0 KiB

Convert a file size to human-readable form.

    Keyword arguments:
    size -- file size in bytes
    a_kilobyte_is_1024_bytes -- if True (default), use multiples of 1024
                                if False, use multiples of 1000

    Returns: string 
```

1.  第一行导入了作为一个模块的 `humansize` 程序 — 我们可以交互式的使用的一大块代码，或者来自于一个更大的 Python 程序。一旦你导入了一个模块，你就可以引用它的任何公有的函数、类、或者属性。模块可以通过这种方式访问其他模块的功能，同样的你也可以在 Python 交互式的 Shell 里面做这样的事情。这是一个重要的概念，贯穿这本书，你会看到更多的关于它的内容。
2.  当你想使用在导入的模块中定义的函数的时候，你需要包含模块的名称。因此你不能仅仅指明 `approximate_size`，它必须是 `humansize.approximate_size` 才行。如果你曾经使用过 Java 里面的类，你就会依稀的感觉到这种方式比较熟悉。
3.  除了按照你期望的方式调用这个函数，你查看了这个函数的其中一个属性： `__doc__`。

> ☞Python 里面的 `import` 就像 Perl 里面的 `require`。一旦你导入（`import`）了一个 Python 模块，你就可以通过 `module`.`function` 的方式访问它的函数；一旦你要求（`require`）了一个 Perl 模块，你就可以通过 `module`::`function` 的方式访问它的函数。

### 什么是一个对象？

Python 里面的所有东西都是对象，所有东西都可以有属性和方法。所有函数都有一个内置的属性 `__doc__`，用来返回这个函数的源代码里面定义的文档字符串（`docstring`）。 `sys` 模块是一个对象，它有（除了别的以外）一个名叫 `path` 的属性，等等。

不过，这还是没有回答这个更基础的问题：什么是一个对象？不同的程序语言用不同的方式定义了“对象”。在有些地方，它意味着*所有的*对象*必须*要有属性和方法；在另一些地方，它意味着所有的对象都是可衍生（可以创建子类）的。在 Python 里面，定义更加宽松。有些对象既没有属性也没有方法，然而它可以有。不是所有的对象都是可衍生的。但是，所有的东西都是对象，从这个意义上说，它能够被赋值到一个变量或者作为一个参数传入一个函数。

你可能从其他程序语言环境中听说过 “first-class object” 的说法。在 Python 中，函数是 *first-class objects*，你可以将一个函数作为一个参数传递给另外一个函数；模块是 *first-class objects*，你可以把整个模块作为一个参数传递给一个函数；类是 first-class objects，而且类的单独的实例也是 first-class objects。

这个很重要，因此刚开始我会重复几次以防你忘记了：*在 Python 里面所有东西都是对象*。字符串是对象，列表是对象，函数是对象，类是对象，类的实例是对象，甚至模块也是对象。

## 代码縮进

Python 函数没有明确的开始（`begin`）或者结束（`end`），也没有用大括号来标记函数从哪里开始从哪里停止。唯一的定界符就是一个冒号（`:`）和代码自身的缩进。

```py
 multiple = 1024 if a_kilobyte_is_1024_bytes else 1000

        size /= multiple
        if size < multiple:
            return '{0:.1f} {1}'.format(size, suffix)

    raise ValueError('number too large') 
```

1.  代码块是通过它们的缩进来定义的。我说的“代码块”，意思是指函数，`if` 语句、 `for` 循环、 `while` 循环，等等。 缩进表示一个代码块的开始，非缩进表示一个代码的结束。没有明确的大括号、中括号、或者关键字。这意味着空白很重要，而且必须要是一致的。在这个例子中，这个函数按照四个空格缩进。它不需要一定是四个空格，只是需要保持一致。第一个没有缩进的行标记了这个函数的结束。
2.  在 Python 中，一个 `if` 语句后面紧跟了一个代码块。如果 `if` 表达式的值为 true 则缩进的代码会被执行，否则它会跳到 `else` 代码块（如果有的话）。注意表达式的周围没有括号。
3.  这一行在 `if` 代码块里面。这个 `raise` 语句将抛出一个异常（类型是 `ValueError` ），但只有在 `size &lt; 0` 的时候才抛出。
4.  这*不是*函数的结尾。完全空白的行不算。它们使代码更加易读，但它们不算作代码块的定界符。这个函数在下一行继续。
5.  这个 `for` 循环也标记了一个代码块的开始。代码块可以包含多行，只要它们都按照同样的数额缩进。这个 `for` 循环里面有三行。对于多行的代码块，也没有其他特殊的语法，只要缩进就可以了。

在刚开始的一些反对声和一些类比到 Fortran 的嘲笑之后，你将会平和的看待这个并开始领会到它的好处。一个主要的好处是所有的 Python 程序看起来都类似，因为缩进是一个语言的要求，不是一个风格的问题。这使得阅读和理解其他人的 Python 代码更加容易。

> ☞Python 使用回车符来分割语句，使用一个冒号和缩进来分割代码块。 C++ 和 Java 使用分号来分割语句，使用大括号来分割代码块。

## 异常

异常在 Python 中无处不在。事实上在标准 Python 库里面的每个模块都使用它们，而且在很多不同情形下， Python 自身也会抛出异常。贯穿这本书，你会反复的看到它们。

什么是一个异常？通常情况下，它是一个错误，提示某个东西出问题了。（不是所有的异常都是错误，但目前来说别担心那个） 某些程序语言鼓励对错误返回代码的使用，你可以对它进行*检查*。 Python 鼓励对异常的使用，你可以对它进行*处理*。

当一个错误发生在 Python Shell 里面的时候，它会打印一些关于这个异常以及它如何发生的详细信息，就此而已。这个被称之为一个 *未被处理* 的异常。在这个异常被抛出的时候，没有代码注意到并处理它，因此它把它的路径冒出来，返回到 Python Shell 的最顶层，输出一些调试信息，然后圆满结束。在这个 Shell 中，这没什么大不了的，但是如果在你的实际 Python 程序正在运行的时候发生，并且对这个异常没有做任何处理的话，整个程序就会嘎的一声停下来。可能那正是你想要的，也可能不是。

> ☞不像 Java， Python 函数不声明它们可能会抛出哪些异常。它取决于你去判断哪些可能的异常是你需要去捕获的。

一个异常不会造成整个程序崩溃。不过，异常是可以被*处理*的。有时候一个异常是真正地由于你代码里面的一个 bug 所引起的（比如访问一个不存在的变量），但有时候一个 异常是你可以预料到的东西。如果你在打开一个文件，它有可能不存在。如果你在导入一个模块，它可能没有被安装。如果你在连接到一个数据库，它有可能是无效的，或者你可能没有访问它需要的安全认证信息。如果你知道某行代码可能抛出一个异常，你应该使用 `try...except` 块来处理这个异常。

> ☞Python 使用 `try...except` 块来处理异常，使用 `raise` 语句来抛出异常。 Java 和 C++ 使用 `try...catch` 块来处理异常，使用 `throw` 语句来抛出异常。

这个 `approximate_size()` 函数在两个不同的情况下抛出异常：如果给定的 `size` 的值大于这个函数打算处理的值，或者如果它小于零。

```py
if size < 0:
    raise ValueError('number must be non-negative') 
```

抛出一个异常的语法足够简单。使用 `raise` 语句，紧跟着异常的名称，和一个人们可以读取的字符串用来调试。这个语法让人想起调用的函数。（实际上，异常是用类来实现的，这个 `raise` 语句事实上正在创建一个 `ValueError` 类的实例并传递一个字符串 `'number must be non-negative'` 到它的初始化方法里面。但是，我们已经有些超前了！）

> ☞你不需要在抛出异常的函数里面去处理它。如果一个函数没有处理它，这个异常会被传递到它的调用函数，然后那个函数的调用函数，等等“在这个堆栈上面。” 如果这个异常从来没有被处理，你的程序将会崩溃， Python 将会打印一个 “traceback” 的标准错误信息，并以此结束。这也可能正是你想要的，它取决于你的程序具体做什么。

### 捕获导入错误

其中一个 Python 的内置异常是 `ImportError`，它会在你试图导入一个模块并且失败的时候抛出。这有可能由于多种原因引起，但是最简单的情况是当在你的 import 搜索路径里面找不到这个模块的时候会发生。你可以用这个来包含可选的特性到你的程序中。例如， 这个 `chardet` 库 提供字符编码自动检测。也许你的程序想在这个库存在的时候使用它，但是如果用户没有安装，也会优雅地继续执行。你可以使用 `try..except` 块来做这样的事情。

```py
<mark>try</mark>:
  import chardet
<mark>except</mark> ImportError:
  chardet = None 
```

然后，你可以用一个简单的 `if` 语句来检查 `chardet` 模块是否存在：

```py
if chardet:
  # do something
else:
  # continue anyway 
```

另一个对 `ImportError` 异常的通常使用是当两个模块实现了一个公共的 API，但我们更想要其中一个的时候。（可能它速度更快，或者使用了更少的内存。） 你可以试着导入其中一个模块，并且在这个模块导入失败的时候退回到另一个不同的模块。例如， XML 的章节谈论了两个模块实现一个公共的 API，叫做 `ElementTree` API。 第一个，`lxml` 是一个第三方的模块，你需要自己下载和安装。第二个， `xml.etree.ElementTree` 比较慢，但属于 Python 3 标准库的一部分。

```py
try:
    from lxml import etree
except ImportError:
    import xml.etree.ElementTree as etree 
```

在这个 `try..except` 块的结尾，你导入了*某个*模块并取名为 `etree`。由于两个模块实现了一个公共的 API，你剩下的代码不需要一直去检查哪个模块被导入了。而且由于这个一定会被导入的模块总是叫做 `etree`，你余下的代码就不会被调用不同名称模块的 `if` 语句所打乱。

## Unbound 变量

再看看 `approximate_size()` 函数里面的这行代码：

```py
multiple = 1024 if a_kilobyte_is_1024_bytes else 1000 
```

你从不声明这个 `multiple` 变量，你只是给它赋值了。这样就可以了，因为 Python 让你那样做。 Python 将不会让你做的是，引用了一个变量，但从不给它赋值。这样的尝试将会抛出一个 `NameError` 的异常。

```py
>>> x
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'x' is not defined
>>> x = 1
>>> x
1 
```

将来有一天，你会因为这个而感谢 Python 。

## 所有的东西都是区分大小写的

Python 里面所有的名称都是区分大小写的：变量名、函数名、类名、模块名称、异常名称。如果你可以获取它、设置它、调用它、构建它、导入它、或者抛出它，那么它就是区分大小写的。

```py
>>> an_integer = 1
>>> an_integer
1
>>> AN_INTEGER
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'AN_INTEGER' is not defined
>>> An_Integer
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'An_Integer' is not defined
>>> an_inteGer
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'an_inteGer' is not defined 
```

等等。

## 运行脚本

Python 里面所有东西都是对象。

Python 模块是对象，并且有几个有用的属性。在你编写它们的时候，通过包含一个特殊的仅在你从命令行运行 Python 文件的时候执行的代码块，你可以使用这些属性容易地测试你的模块。看看 `humansize.py` 的最后几行代码：

```py
 if __name__ == '__main__':
    print(approximate_size(1000000000000, False))
    print(approximate_size(1000000000000)) 
```

> ☞像 C 语言一样， Python 使用 `==` 来做比较，用 `=` 来赋值。不同于 C 语言的是， Python 不支持内嵌的赋值，所以没有机会出现你本以为在做比较而且意外的写成赋值的情况。

那么是什么使得这个 `if` 语句特别的呢？ 好吧，模块是对象，并且所有模块都有一个内置的属性 `__name__`。一个模块的 `__name__` 属性取决于你怎么来使用这个模块。如果你 `import` 这个模块，那么 `__name__` 就是这个模块的文件名，不包含目录的路径或者文件的扩展名。

```py
>>> import humansize
>>> humansize.__name__
'humansize' 
```

但是你也可以当作一个独立的程序直接运行这个模块，那样的话 `__name__` 将是一个特殊的默认值 `__main__`。 Python 将会评估这个 `if` 语句，寻找一个值为 true 的表达式，然后执行这个 `if` 代码块。在这个例子中，打印两个值。

```py
c:\home\diveintopython3> c:\python31\python.exe humansize.py
1.0 TB
931.3 GiB 
```

这就是你的第一个 Python 程序！

## 深入阅读

*   [PEP 257: Docstring 约定](http://www.python.org/dev/peps/pep-0257/)解释了用什么来从大量的 `docstring` 中分辨出一个好的 `docstring`。
*   [Python 教程：文档字符串](http://docs.python.org/3.1/tutorial/controlflow.html#documentation-strings)也略微提到了这个主题。
*   [PEP 8: Python 代码的风格指南](http://www.python.org/dev/peps/pep-0008/)讨论了好的缩进风格。
*   [Python 参考手册](http://docs.python.org/3.1/reference/)解释了为什么说 Python 里面所有东西都是对象，因为有些人是书呆子，喜欢详细地讨论一些东西。