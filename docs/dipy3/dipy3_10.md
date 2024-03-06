# Chapter 7 类 & 迭代器

# Chapter 7 类 & 迭代器

> " 东是东，西是西，东西不相及 " — [拉迪亚德·吉卜林](http://en.wikiquote.org/wiki/Rudyard_Kipling)

## 深入

生成器是一类特殊 *迭代器*。 一个产生值的函数 `yield` 是一种产生一个迭代器却不需要构建迭代器的精密小巧的方法。 我会告诉你我是什么意思。

记得 菲波拉稀生成器吗？ 这里是一个从无到有的迭代器：

[下载 `fibonacci2.py`]

```py
class Fib:
    '''生成菲波拉稀数列的迭代器'''

    def __init__(self, max):
        self.max = max

    def __iter__(self):
        self.a = 0
        self.b = 1
        return self

    def __next__(self):
        fib = self.a
        if fib > self.max:
            raise StopIteration
        self.a, self.b = self.b, self.a + self.b
        return fib 
```

让我们一行一行来分析。

```py
class Fib: 
```

类(class)？什么是类？

## 类的定义

Python 是完全面向对象的：你可以定义自己的类，从你自己或系统自带的类继承，并生成实例。

在 Python 里定义一个类非常简单。就像函数一样， 没有分开的接口定义。 只需定义类就开始编码。 Python 类以保留字 `class` 开始， 后面跟类名。 技术上来说，只需要这么多就够了，因为一个类不是必须继承其他类。

1.  类名是 `PapayaWhip`， 没有从其他类继承。 类名通常是大写字母分隔， 如`EachWordLikeThis`， 但这只是个习惯，并非必须。
2.  你可能猜到，类内部的内容都需缩进，就像函数中的代码一样， `if` 语句， `for` 循环， 或其他代码块。第一行非缩进代码表示到了类外。

`PapayaWhip` 类没有定义任何方法和属性， 但依据句法，应该在定义中有东西，这就是 `pass` 语句。 这是 Python 保留字，意思是“继续，这里看不到任何东西”。 这是一个什么都不做的语句，是一个很好的占位符，如果你的函数和类什么都不想做（删空函数或类）。

> ☞Python 中的`pass` 就像 Java 或 C 中的空大括号对 (`{}`) 。

很多类继承自其他类， 但这个类没有。 很多类有方法，这个类也没有。 Python 类不是必须有东西，除了一个名字。 特别是 C++ 程序员发现 Python 类没有显式的构造和析构函数会觉得很古怪。 尽管不是必须， Python 类 *可以* 具有类似构造函数的东西： `__init__()` 方法。

### `__init__()` 方法

本示例展示 `Fib` 类使用 `__init__` 方法。

```py
class Fib: 
```

1.  类同样可以 (而且应该) 具有`docstring`， 与模块和方法一样。
2.  类实例创建后，`__init__()` 方法被立即调用。很容易将其——但技术上来说不正确——称为该类的“构造函数” 。 很容易，因为它看起来很像 C++ 的构造函数（按约定，`__init__()` 是类中第一个被定义的方法），行为一致（是类的新实例中第一片被执行的代码）， 看起来完全一样。 错了， 因为`__init__()` 方法调用时，对象已经创建了，你已经有了一个合法类对象的引用。

每个方法的第一个参数，包括 `__init__()` 方法，永远指向当前的类对象。 习惯上，该参数叫 `self`。 该参数和 C++或 Java 中 `this` 角色一样， 但 `self` 不是 Python 的保留字， 仅仅是个命名习惯。 虽然如此，请不要取别的名字，只用 `self`； 这是一个很强的命名习惯。

在 `__init__()` 方法中， `self` 指向新创建的对象； 在其他类对象中， 它指向方法所属的实例。尽管需在定义方法时显式指定`self` ，调用方法时并 *不* 必须明确指定。 Python 会自动添加。

## 实例化类

Python 中实例化类很直接。 实例化类时就像调用函数一样简单，将 `__init__()` 方法需要的参数传入。 返回值就是新创建的对象。

```py
>>> import fibonacci2

<fibonacci2.Fib object at 0x00DB8810>

<class 'fibonacci2.Fib'>

'' 
```

1.  你正创建一个 `Fib` 类的实例（在`fibonacci2` 模块中定义） 将新创建的实例赋给变量`fib`。 你传入一个参数 `100`， 这是`Fib`的`__init__()`方法作为`max`参数传入的结束值。
2.  `fib` 是 `Fib` 的实例。
3.  每个类实例具有一个内建属性， `__class__`， 它是该对象的类。 Java 程序员可能熟悉 `Class` 类， 包含方法如 `getName()` 和 `getSuperclass()` 获取对象相关元数据。 Python 里面， 这类元数据由属性提供，但思想一致。
4.  你可访问对象的 `docstring` ，就像函数或模块中的一样。 类的所有实例共享一份 `docstring`。

> ☞Python 里面， 和调用函数一样简单的调用一个类来创建该类的新实例。 与 C++ 或 Java 不一样，没有显式的 `new` 操作符。

## 实例变量

继续下一行：

```py
class Fib:
    def __init__(self, max): 
```

1.  `self.max`是什么？ 它就是实例变量。 与作为参数传入 `__init__()` 方法的 `max`完全是两回事。 `self.max` 是实例内 “全局” 的。 这意味着可以在其他方法中访问它。

```py
class Fib:
    def __init__(self, max):

    .
    .
    .
    def __next__(self):
        fib = self.a 
```

1.  `self.max` 在 `__init__()` 方法中定义……
2.  ……在 `__next__()` 方法中引用。

实例变量特定于某个类的实例。 例如， 如果你创建 `Fib` 的两个具有不同最大值的实例， 每个实例会记住自己的值。

```py
>>> import fibonacci2
>>> fib1 = fibonacci2.Fib(100)
>>> fib2 = fibonacci2.Fib(200)
>>> fib1.max
100
>>> fib2.max
200 
```

## 菲波拉稀迭代器

*现在* 你已经准备学习如何创建一个迭代器了。 迭代器就是一个定义了 `__iter__()` 方法的类。

[下载 `fibonacci2.py`]

```py
 self.max = max

        self.a = 0
        self.b = 1
        return self

        fib = self.a
        if fib > self.max:

        self.a, self.b = self.b, self.a + self.b 
```

1.  从无到有创建一个迭代器， `fib` 应是一个类，而不是一个函数。
2.  “调用” `Fib(max)` 会创建该类一个真实的实例，并以`max`做为参数调用`__init__()` 方法。 `__init__()` 方法以实例变量保存最大值，以便随后的其他方法可以引用。
3.  当有人调用`iter(fib)`的时候，`__iter__()`就会被调用。（正如你等下会看到的， `for` 循环会自动调用它， 你也可以自己手动调用。） 在完成迭代器初始化后，（在本例中， 重置我们两个计数器 `self.a` 和 `self.b`）， `__iter__()` 方法能返回任何实现了 `__next__()` 方法的对象。 在本例（甚至大多数例子）中， `__iter__()` 仅简单返回 `self`， 因为该类实现了自己的 `__next__()` 方法。
4.  当有人在迭代器的实例中调用`next()`方法时，`__next__()` 会自动调用。 随后会有更多理解。
5.  当 `__next__()` 方法抛出 `StopIteration` 异常， 这是给调用者表示迭代用完了的信号。 和大多数异常不同， 这不是错误；它是正常情况，仅表示迭代器没有值可产生了。 如果调用者是 `for` 循环， 它会注意到该 `StopIteration` 异常并优雅的退出。 （换句话说，它会吞掉该异常。） 这点神奇之处就是使用 `for` 的关键。
6.  为了分离出下一个值， 迭代器的 `__next__()` 方法简单 `return`该值。 不要使用 `yield` ； 该语法上的小甜头仅用于你使用生成器的时候。 这里你从无到有创建迭代器，使用 `return` 代替。

完全晕了？ 太好了。 让我们看如何调用该迭代器：

```py
>>> from fibonacci2 import Fib
>>> for n in Fib(1000):
...     print(n, end=' ')
0 1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 
```

为什么？完全一模一样！ 一字节一字节的与你调用 Fibonacci-as-a-generator （模块第一个字母大写）相同。但怎么做到的？

`for` 循环内有魔力。下面是究竟发生了什么：

*   如你所见，`for` 循环调用 `Fib(1000)`。 这返回`Fib` 类的实例。 叫它 `fib_inst`。
*   背地里，且十分聪明的， `for` 循环调用 `iter(fib_inst)`， 它返回迭代器。 叫它 `fib_iter`。 本例中， `fib_iter` == `fib_inst`， 因为 `__iter__()` 方法返回 `self`，但`for` 循环不知道（也不关心）那些。
*   为“循环通过”迭代器， `for` 循环调用 `next(fib_iter)`， 它又调用 `fib_iter`对象的 `__next__()` 方法，产生下一个菲波拉稀计算并返回值。 `for` 拿到该值并赋给 `n`， 然后执行`n`值的 `for` 循环体。
*   `for`循环如何知道什么时候结束？很高兴你问到。 当`next(fib_iter)` 抛出 `StopIteration` 异常时， `for`循环将吞下该异常并优雅退出。 （其他异常将传过并如常抛出。） 在哪里你见过 `StopIteration` 异常？ 当然在 `__next__()` 方法。

## 复数规则迭代器

iter(f) 调用 f.**iter** next(f) 调用 f.**next**

现在到曲终的时候了。我们重写 复数规则生成器 为迭代器。

[下载`plural6.py`]

```py
class LazyRules:
    rules_filename = 'plural6-rules.txt'

    def __init__(self):
        self.pattern_file = open(self.rules_filename, encoding='utf-8')
        self.cache = []

    def __iter__(self):
        self.cache_index = 0
        return self

    def __next__(self):
        self.cache_index += 1
        if len(self.cache) >= self.cache_index:
            return self.cache[self.cache_index - 1]

        if self.pattern_file.closed:
            raise StopIteration

        line = self.pattern_file.readline()
        if not line:
            self.pattern_file.close()
            raise StopIteration

        pattern, search, replace = line.split(None, 3)
        funcs = build_match_and_apply_functions(
            pattern, search, replace)
        self.cache.append(funcs)
        return funcs

rules = LazyRules() 
```

因此这是一个实现了 `__iter__()` 和 `__next__()`的类。所以它可以 被用作迭代器。然后，你实例化它并将其赋给 `rules` 。这只发生一次，在 import 的时候。

让我们一口一口来吃：

```py
class LazyRules:
    rules_filename = 'plural6-rules.txt'

    def __init__(self): 
```

1.  当我们实例化 `LazyRules` 类时， 打开模式文件，但不读取任何东西。 （随后再进行）
2.  打开模式文件之后，初始化缓存。 随后读取模式文件行的时候会用到它（在 `__next__()` 方法中） 。

我们继续之前，让我们近观 `rules_filename`。它没在 `__iter__()` 方法中定义。事实上，它没在任何方法中定义。它定义于类级别。它是 *类变量*， 尽管访问时和实例变量一样 （`self.rules_filename`）， `LazyRules` 类的所有实例共享该变量。

```py
>>> import plural6
>>> r1 = plural6.LazyRules()
>>> r2 = plural6.LazyRules()

'plural6-rules.txt'
>>> r2.rules_filename
'plural6-rules.txt'

>>> r2.rules_filename
'r2-override.txt'
>>> r1.rules_filename
'plural6-rules.txt'

'plural6-rules.txt'

>>> r1.rules_filename
'papayawhip.txt'

'r2-overridetxt' 
```

1.  类的每个实例继承了 `rules_filename` 属性及它在类中定义的值。
2.  修改一个实例属性的值不影响其他实例……
3.  ……也不会修改类的属性。可以使用特殊的 `__class__` 属性来访问类属性（于此相对的是单独实例的属性）。
4.  如果修改类属性， 所有仍然继承该实例的值的实例 （如这里的`r1` ） 会受影响。
5.  已经覆盖（overridden）了该属性（如这里的 `r2` ）的所有实例 将不受影响。

现在回到我们的演示：

```py
self.cache_index = 0 
```

1.  无论何时有人——如 `for` 循环——调用 `iter(rules)`的时候，`__iter__()` 方法都会被调用。
2.  每个`__iter__()` 方法都需要做的就是必须返回一个迭代器。 在本例中，返回 `self`，意味着该类定义了`__next__()` 方法，由它来关注整个迭代过程中的返回值。

```py
 .
        .
        .
        pattern, search, replace = line.split(None, 3)

            pattern, search, replace)

        return funcs 
```

1.  无论何时有人——如 `for` 循环——调用 `__next__()` 方法， `next(rules)`都跟着被调用。 该方法仅在我们从后往前移动时比较好体会。所以我们就这么做。
2.  函数的最后一部分至少应该眼熟。 `build_match_and_apply_functions()` 函数还没修改；与它从前一样。
3.  唯一的不同是，在返回匹配和应用功能之前（保存在元组 `funcs`中），我们将其保存到 `self.cache`。

从后往前移动……

```py
 def __next__(self):
        .
        .
        .

            self.pattern_file.close()

        .
        .
        . 
```

1.  这里有点高级文件操作的技巧。 `readline()` 方法 （注意：是单数，不是复数 `readlines()`） 从一个打开的文件中精确读取一行，即下一行。（*文件对象同样也是迭代器！ 它自始至终是迭代器……*）
2.  如果有一行 `readline()` 可以读， `line` 就不会是空字符串。 甚至文件包含一个空行， `line` 将会是一个字符的字符串 `'\n'` （回车换行符）。 如果 `line` 是真的空字符串， 就意味着文件已经没有行可读了。
3.  当我们到达文件尾时， 我们应关闭文件并抛出神奇的 `StopIteration` 异常。 记住，开门见山的说是因为我们需要为下一条规则找到一个匹配和应用功能。下一条规则从文件的下一行获取…… 但已经没有下一行了！ 所以，我们没有规则返回。 迭代器结束。 （♫ 派对结束 ♫）

由后往前直到 `__next__()`方法的开始……

```py
 def __next__(self):
        self.cache_index += 1
        if len(self.cache) >= self.cache_index:

        if self.pattern_file.closed:

        .
        .
        . 
```

1.  `self.cache` 将是一个我们匹配并应用单独规则的功能列表。 （至少*那个*应该看起来熟悉！） `self.cache_index` 记录我们下一步返回的缓存条目。 如果我们还没有耗尽缓存 （*举例* 如果 `self.cache` 的长度大于 `self.cache_index`），那么我们就会命中一条缓存！ 哇！ 我们可以从缓存中返回匹配和应用功能而不是从无到有创建。
2.  另一方面，如果我们没有从缓存中命中条目， *并且* 文件对象也已关闭（这会发生， 在本方法下面一点， 正如你从预览的代码片段中所看到的），那么我们什么都不能做。 如果文件被关闭，意味着我们已经用完了它——我们已经从头至尾读取了模式文件的每一行，而且已经对每个模式创建并缓存了匹配和应用功能。文件已经读完；缓存已经用完；我也快完了。等等，什么？坚持一下，我们几乎完成了。

放到一起，发生了什么事？ 当：

*   当模块引入时，创建了`LazyRules` 类的一个单一实例， 叫 `rules`， 它打开模式文件但并没有读取。
*   当要求第一个匹配和应用功能时，检查缓存并发现缓存为空。 于是，从模式文件读取一行， 从模式中创建匹配和应用功能，并缓存之。
*   假如，因为参数的缘故，正好是第一行匹配了。如果那样，不会有更多的匹配和应用会创建，也不会有更多的行会从模式文件中读取。
*   更进一步， 因为参数的缘故，假设调用者*再次*调用 `plural()` 函数来让一个不同的单词变复数。 `plural()` 函数中的`for` 循环会调用`iter(rules)`，这会重置缓存索引但不会重置打开的文件对象。
*   第一次遍历， `for`循环会从`rules`中索要一个值，该值会调用其`__next__()`方法。然而这一次， 缓存已经被装入了一个匹配和应用功能对， 与模式文件中第一行模式一致。 由于对前一个单词做复数变换时已经被创建和缓存，它们被从缓存中返回。 缓存索引递增，打开的文件无需访问。
*   假如，因为参数的缘故，这一轮第一个规则 *不* 匹配。 所以 `for` 循环再次运转并从 `rules`请求一个值。 这会再次调用 `__next__()` 方法。 这一次， 缓存被用完了——它仅有一个条目，而我们被请求第二个——于是 `__next__()` 方法继续。 从打开的文件中读取下一行，从模式中创建匹配和应用功能，并缓存之。
*   该“读取创建并缓存”过程一直持续直到我们从模式文件中读取的规则与我们想变复数的单词不匹配。 如果我们确实在文件结束前找到了一个匹配规则，我们仅需使用它并停止，文件还一直打开。文件指针会留在我们停止读取，等待下一个 `readline()` 命令的地方。现在，缓存已经有更多条目了，并且再次从头开始来将一个新单词变复数，在读取模式文件下一行之前，缓存中的每一个条目都将被尝试。

我们已经到达复数变换的极乐世界。

1.  **最小化初始代价。** 在 `import` 时发生的唯一的事就是实例化一个单一的类并打开一个文件（但并不读取）。
2.  **最大化性能** 前述示例会在每次你想让一个单词变复数时，读遍文件并动态创建功能。本版本将在创建的同时缓存功能，在最坏情况下，仅需要读完一遍文件，无论你要让多少单词变复数。
3.  **将代码和数据分离。** 所有模式被存在一个分开的文件。代码是代码，数据是数据，二者永远不会交织。

> ☞这真的是极乐世界？ 嗯，是或不是。 这里有一些`LazyRules` 示例需要细想的地方： 模式文件被打开（在 `__init__()`中），并持续打开直到读取最后一个规则。 当 Python 退出或最后一个`LazyRules` 类的实例销毁，Python 会最终关闭文件，但是那仍然可能会是一个*很长*的时间。如果该类是一个“长时间运行”的 Python 进程的一部分，Python 可能从不退出， `LazyRules` 对象就可能一直不会释放。
> 
> 这种情况有解决办法。 不要在 `__init__()` 中打开文件并让其在一行一行读取规则时一直打开，你可以打开文件，读取所有规则，并立即关闭文件。或你可以打开文件，读取一条规则，用`tell()` 方法保存文件位置，关闭文件，后面再次打开它，使用`seek()` 方法 继续从你离开的地方读取。 或者你不需担心这些就让文件打开，如同本示例所做。 编程即是设计， 而设计牵扯到所有的权衡和限制。让一个文件一直打开太长时间可能是问题；让你代码太复杂也可能是问题。哪一个是更大的问题，依赖于你的开发团队，你的应用，和你的运行环境。

## 深入阅读

*   [迭代器类型](http://docs.python.org/3.1/library/stdtypes.html#iterator-types)
*   [PEP 234: 迭代器（ Iterators ）](http://www.python.org/dev/peps/pep-0234/)
*   [PEP 255:简单生成器（ Simple Generators ）](http://www.python.org/dev/peps/pep-0255/)
*   系统程序员的生成器诀窍（ Generator Tricks for Systems Programmers ）