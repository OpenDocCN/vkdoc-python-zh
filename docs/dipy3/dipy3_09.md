# Chapter 6 闭合 与 生成器

# Chapter 6 闭合 与 生成器

> " My spelling is Wobbly. It’s good spelling but it Wobbles, and the letters get in the wrong places. " — Winnie-the-Pooh

## 深入

出于传递所有理解的原因，我一直对语言非常着迷。我指的不是编程语言。好吧，是编程语言，但同时也是自然语言。使用英语。英语是一种七拼八凑的语言，它从德语、法语、西班牙语和拉丁语（等等）语言中借用了大量词汇。事实上，“借用”是不恰当的词汇，“掠夺”更加符合。或者也许叫“同化“——就像博格人（译注：根据维基百科资料，Borg 是《星际旅行》虚构宇宙中的一个种族，该译法未经原作者映证）。是的，我喜欢这样。

`我们就是博格人。你们的语言和词源特性将会被添加到我们自己的当中。抵抗是徒劳的。`

在本章中，将开始学习复数名词。以及返回其它函数的函数、高级正则表达式和生成器。但首先，让我们聊聊如何生成复数名词。（如果还没有阅读《正则表达式》一章，现在也许是个好时机读一读。本章将假定您理解了正则表达式的基础，并迅速进入更高级的用法。）

如果在讲英语的国家长大，或在正规的学校学习过英语，您可能对下面的基本规则很熟悉 ：

*   如果某个单词以 S 、X 或 Z 结尾，添加 ES 。*Bass* 变成 *basses*， *fax* 变成 *faxes*，而 *waltz* 变成 *waltzes*。
*   如果某个单词以发音的 H 结尾，加 ES；如果以不发音的 H 结尾，只需加上 S 。什么是发音的 H ？指的是它和其它字母组合在一起发出能够听到的声音。因此 *coach* 变成 *coaches* 而 *rash* 变成 *rashes*，因为在说这两个单词的时候，能够听到 CH 和 SH 的发音。但是 *cheetah* 变成 *cheetahs*，因为 H 不发音。
*   如果某个单词以发 I 音的字母 Y 结尾，将 Y 改成 IES；如果 Y 与某个原因字母组合发其它音的话，只需加上 S 。因此 *vacancy* 变成 *vacancies*，但 *day* 变成 *days* 。
*   如果所有这些规则都不适用，只需加上 S 并作最好的打算。

（我知道，还有许多例外情况。*Man* 变成 *men* 而 *woman* 变成 *women*，但是 *human* 变成 *humans*。*Mouse* 变成 *mice* ； *louse* 变成 *lice*，但 *house* 变成 *houses*。*Knife* 变成 *knives* ；*wife* 变成 *wives*，但是 *lowlife* 变成 *lowlifes*。而且甚至我还没有开始提到那些原型和复数形式相同的单词，就像 *sheep*、 *deer* 和 *haiku*。）

其它语言，当然是完全不同的。

让我们设计一个 Python 类库用来自动进行英语名词的复数形式转换。我们将以这四条规则为起点，但要记住的不可避免地还要增加更多规则。

## 我知道，让我们用正则表达式！

因此，您正在看着单词，至少是英语单词，也就是说您正在看着字符的字符串。规则说你必须找到不同的字符组合，然后进行不同的处理。这听起来是正则表达式的工作！

[下载 `plural1.py`]

```py
import re

def plural(noun):          

    elif re.search('[^aeioudgkprt]h$', noun):
        return re.sub('$', 'es', noun)       
    elif re.search('[^aeiou]y$', noun):      
        return re.sub('y$', 'ies', noun)     
    else:
        return noun + 's' 
```

1.  这是一条正则表达式，但它使用了在 *《正则表达式》* 一章中没有讲过的语法。中括号表示“匹配这些字符的其中之一”。因此 `[sxz]` 的意思是： “`s`、 `x` 或 `z`”，但只匹配其中之一。对 `$` 应该很熟悉了，它匹配字符串的结尾。经过组合，该正则表达式将测试 `noun` 是否以 `s`、 `x` 或 `z` 结尾。
2.  该 `re.sub()` 函数执行基于正则表达式的字符串替换。

让我们看看正则表达式替换的细节。

```py
>>> import re

<_sre.SRE_Match object at 0x001C1FA8>

'Mork'

'rook'

'oops' 
```

1.  字符串 `Mark` 包含 `a`、 `b` 或 `c` 吗？是的，它包含 `a` 。
2.  好了，现在查找 `a`、 `b` 或 `c`，并将其替换为 `o`。`Mark` 变成了 `Mork`。
3.  同一函数将 `rock` 转换为 `rook` 。
4.  您可能会认为该函数会将 `caps` 转换为 `oaps`，但实际上并是这样。`re.sub` 替换 *所有的* 匹配项，而不仅仅是第一个匹配项。因此该正则表达式将 `caps` 转换为 `oops`，因为无论是 `c` 还是 `a` 均被转换为 `o` 。

接下来，回到 `plural()` 函数……

```py
def plural(noun):          
    if re.search('[sxz]$', noun):            

        return re.sub('$', 'es', noun)

        return re.sub('y$', 'ies', noun)     
    else:
        return noun + 's' 
```

1.  此处将字符串的结尾（通过 `$` 匹配）替换为字符串 `es` 。换句话来说，向字符串尾部添加一个 `es` 。可以通过字符串链接来完成同样的变化，例如 `noun + 'es'`，但我对每条规则都选用正则表达式，其原因将在本章稍后更加清晰。
2.  仔细看看，这里出现了新的变化。作为方括号中的第一个字符， `^` 有特别的含义：非。`[^abc]` 的意思是：“ *除了* `a`、 `b` 或 `c` 之外的任何字符”。因此 `[^aeioudgkprt]` 的意思是除了 `a`、 `e`、 `i`、 `o`、 `u`、 `d`、 `g`、 `k`、 `p`、`r` 或 `t` 之外的任何字符。然后该字符必须紧随一个 `h`，其后是字符串的结尾。所匹配的是以 H 结尾且 H 发音的单词。
3.  此处有同样的模式：匹配以 Y 结尾的单词，而 Y 之前的字符 *不是* `a`、 `e`、 `i`、 `o` 或 `u`。所匹配的是以 Y 结尾，且 Y 发音听起来像 I 的单词。

让我们看看“否定”正则表达式的更多细节。

```py
>>> import re

<_sre.SRE_Match object at 0x001C1FA8>

>>> 
>>> re.search('[^aeiou]y$', 'day')
>>> 

>>> 
```

1.  `vacancy` 匹配该正则表达式，因为它以 `cy` 结尾，且 `c` 并非 `a`、 `e`、 `i`、 `o` 或 `u`。
2.  `boy` 不匹配，因为它以 `oy` 结尾，可以明确地说 `y` 之前的字符不能是 `o` 。`day` 不匹配，因为它以 `ay` 结尾。
3.  `pita` 不匹配，因为它不以 `y` 结尾。

```py
 'vacancies'
>>> re.sub('y$', 'ies', 'agency')
'agencies'

'vacancies' 
```

1.  该正则表达式将 `vacancy` 转换为 `vacancies` ，将 `agency` 转换为 `agencies`，这正是想要的结果。注意，它也会将 `boy` 转换为 `boies`，但这永远也不会在函数中发生，因为我们首先进行了 `re.search` 以找出永远不应进行该 `re.sub` 操作的单词。
2.  顺便，我还想指出可以将该两条正则表达式合并起来（一条查找是否应用该规则，另一条实际应用规则），使其成为一条正则表达式。它看起来是下面这个样子：其中多数内容看起来应该很熟悉：使用了在 案例研究：分析电话号码 中用到的记忆分组。该分组用于保存字母 `y` 之前的字符。然后在替换字符串中，用到了新的语法： `\1`，它表示“嘿，记住的第一个分组呢？把它放到这里。”在此例中， 记住了 `y` 之前的 `c` ，在进行替换时，将用 `c` 替代 `c`，用 `ies` 替代 `y` 。（如果有超过一个的记忆分组，可以使用 `\2` 和 `\3` 等等。）

正则表达式替换功能非常强大，而 `\1` 语法则使之愈加强大。但是，将整个操作组合成一条正则表达式也更难阅读，而且也没有直接映射到刚才所描述的复数规则。刚才所阐述的规则，像 “如果单词以 S 、X 或 Z 结尾，则添加 ES 。”如果查看该函数，有两行代码都在表述“如果以 S 、X 或 Z 结尾，那么添加 ES 。”它没有之前那种模式更直接。

## 函数列表

现在要增加一些抽象层次的内容。我们开始时定义了一系列规则：如果这样，那样做；否则前往下一条规则。现在让我们对部分程序进行临时的复杂化，以简化另一部分。

```py
import re

def match_sxz(noun):
    return re.search('[sxz]$', noun)

def apply_sxz(noun):
    return re.sub('$', 'es', noun)

def match_h(noun):
    return re.search('[^aeioudgkprt]h$', noun)

def apply_h(noun):
    return re.sub('$', 'es', noun)

    return re.search('[^aeiou]y$', noun)

    return re.sub('y$', 'ies', noun)

def match_default(noun):
    return True

def apply_default(noun):
    return noun + 's'

         (match_h, apply_h),
         (match_y, apply_y),
         (match_default, apply_default)
         )

def plural(noun):           

        if matches_rule(noun):
            return apply_rule(noun) 
```

1.  现在，每条匹配规则都有自己的函数，它们返回对 `re.search()` 函数调用结果。
2.  每条应用规则也都有自己的函数，它们调用 `re.sub()` 函数以应用恰当的复数变化规则。
3.  现在有了一个 `rules` 数据结构——一个函数对的序列，而不是一个函数（`plural()`）实现多个条规则。
4.  由于所有的规则被分割成单独的数据结构，新的 `plural()` 函数可以减少到几行代码。使用 `for` 循环，可以一次性从 `rules` 这个数据结构中取出匹配规则和应用规则这两样东西（一条匹配对应一条应用）。在 `for` 循环的第一次迭代过程中， `matches_rule` 将获取 `match_sxz`，而 `apply_rule` 将获取 `apply_sxz`。在第二次迭代中（假定可以进行到这一步）， `matches_rule` 将会赋值为 `match_h`，而 `apply_rule` 将会赋值为 `apply_h` 。该函数确保最终能够返回某个值，因为终极匹配规则 (`match_default`) 只返回 `True`，意思是对应的应用规则 (`apply_default`) 将总是被应用。

变量 “rules” 是一系列函数对。

该技术能够成功运作的原因是 Python 中一切都是对象，包括了函数。数据结构 `rules` 包含了函数——不是函数的名称，而是实际的函数对象。在 `for` 循环中被赋值后，`matches_rule` 和 `apply_rule` 是可实际调用的函数。在第一次 `for` 循环的迭代过程中，这相当于调用 `matches_sxz(noun)`，如果返回一个匹配值，将调用 `apply_sxz(noun)` 。

如果这种附加抽象层令你迷惑，可以试着展开函数以了解其等价形式。整个 `for` 循环等价于下列代码：

```py
 def plural(noun):
    if match_sxz(noun):
        return apply_sxz(noun)
    if match_h(noun):
        return apply_h(noun)
    if match_y(noun):
        return apply_y(noun)
    if match_default(noun):
        return apply_default(noun) 
```

这段代码的好处是 `plural()` 函数被简化了。它处理一系列其它地方定义的规则，并以通用的方式对它们进行迭代。

1.  获取某匹配规则
2.  是否匹配？然后调用应用规则，并返回结果。
3.  不匹配？返回步骤 1 。

这些规则可在任何地方以任何方式定义。`plural()` 函数并不关心。

现在，新增的抽象层是否值得呢？嗯，还没有。让我们考虑下要向函数中新增一条规则时该如何操作。在第一例中，将需要新增一条 `if` 语句到 `plural()` 函数中。在第二例中，将需要新增两个函数， `match_foo()` 和 `apply_foo()`，然后更新 `rules` 序列以指定新的匹配和应用函数按照其它规则按顺序调用。

但是对于下一节来说，这只是一个跳板而已。让我们继续……

## 匹配模式列表

其实并不是真的有必要为每个匹配和应用规则定义各自的命名函数。它们从未直接被调用，而只是被添加到 `rules` 序列并从该处被调用。此外，每个函数遵循两种模式的其中之一。所有的匹配函数调用 `re.search()`，而所有的应用函数调用 `re.sub()`。让我们将模式排除在考虑因素之外，使新规则定义更加简单。

```py
import re

def build_match_and_apply_functions(pattern, search, replace):

        return re.search(pattern, word)

        return re.sub(search, replace, word) 
```

1.  `build_match_and_apply_functions()` 函数用于动态创建其它函数。它接受 `pattern`、 `search` 和 `replace` 三个参数，并定义了 `matches_rule()` 函数，该函数通过传给 `build_match_and_apply_functions()` 函数的 `pattern` 及传递给所创建的 `matchs_rules()` 函数的 `word` 调用 `re.search()` 函数，哇。
2.  应用函数的创建工作采用了同样的方式。应用函数只接受一个参数，并使用传递给 `build_match_and_apply_functions()` 函数的 `search` 和 `replace` 参数、以及传递给要创建 `apply_rule()` 函数的 `word` 调用 `re.sub()`。在动态函数中使用外部参数值的技术称为 *闭合【closures】*。基本上，常量的创建工作都在创建应用函数过程中完成：它接受一个参数 （`word`），但实际操作还加上了另外两个值（`search` 和 `replace`），该两个值都在定义应用函数时进行设置。
3.  最后，`build_match_and_apply_functions()` 函数返回一个包含两个值的元组：即刚才所创建的两个函数。在这些函数中定义的常量（ `match_rule()` 函数中的 `pattern` 函数，`apply_rule()` 函数中的 `search` 和 `replace` ）与这些函数呆在一起，即便是在从 `build_match_and_apply_functions()` 中返回后也一样。这真是非常酷的一件事情。

但如果此方式导致了难以置信的混乱（应该是这样，它确实有点奇怪），在看看如何使用之后可能会清晰一些。

```py
 (
    ('[sxz]$',           '$',  'es'),
    ('[^aeioudgkprt]h$', '$',  'es'),
    ('(qu|[^aeiou])y$',  'y$', 'ies'),

  )

         for (pattern, search, replace) in patterns] 
```

1.  我们的复数形式“规则”现在被定义为 *字符串* 的元组的元组（而不是函数）。每个组的第一个字符串是在 `re.search()` 中用于判断该规则是否匹配的正则表达式。各组中的第二和第三个字符串是在 `re.sub()` 中将实际用于使用规则将名词转换为复数形式的搜索和替换表达式。
2.  此处的后备规则略有变化。在前例中，`match_default()` 函数仅返回 `True`，意思是如果更多的指定规则无一匹配，代码将简单地向给定词汇的尾部添加一个 `s`。本例则进行了一些功能等同的操作。最后的正则表达式询问单词是否有一个结尾（`$` 匹配字符串的结尾）。当然，每个字符串都有一个结尾，甚至是空字符串也有，因此该规则将始终被匹配。因此，它实现了 `match_default()` 函数同样的目的，始终返回 `True`：它确保了如果没有更多的指定规则用于匹配，代码将向给定单词的尾部增加一个 `s` 。
3.  本行代码非常神奇。它以 `patterns` 中的字符串序列为参数，并将其转换为一个函数序列。怎么做到的？通过将字符串“映射”到 `build_match_and_apply_functions()` 函数。也就是说，它接受每组三重字符串为参数，并将该三个字符串作为实参调用 `build_match_and_apply_functions()` 函数。 `build_match_and_apply_functions()` 函数返回一个包含两个函数的元组。也就是说该 `规则` 最后的结尾与前例在功能上是等价的：一个元组列表，每个元组都是一对函数。第一个函数是调用 `re.search()` 的匹配函数；而第二个函数调用 `re.sub()` 的应用函数。

此版本脚本的最前面是主入口点—— `plural()` 函数。

```py
def plural(noun):

        if matches_rule(noun):
            return apply_rule(noun) 
```

1.  由于 `规则` 列表与前例中的一样（实际上确实相同），因此毫不奇怪 `plural()` 函数基本没有发生变化。它是完全通用的，它以规则函数列表为参数，并按照顺序调用它们。它并不关系规则是如何定义的。在前例中，它们被定义为各自命名的函数。现在它们通过将 `build_match_and_apply_functions()` 函数的输出映射为源字符串的列表来动态创建。这没有任何关系； `plural()` 函数将以同样方式运作。

## 匹配模式文件

目前，已经排除了重复代码，增加了足够的抽象性，因此复数形式规则可以字符串列表的形式进行定义。下一个逻辑步骤是将这些字符串放入一个单独的文件中，因此可独立于使用它们的代码来进行维护。

首先，让我们创建一份包含所需规则的文本文件。没有花哨的数据结构，只有空白符分隔的三列字符串。将其命名为 `plural4-rules.txt`.

```py
[sxz]$               $    es
[^aeioudgkprt]h$     $    es
[^aeiou]y$          y$    ies
$                    $    s 
```

下面看看如何使用该规则文件。

```py
import re

    def matches_rule(word):
        return re.search(pattern, word)
    def apply_rule(word):
        return re.sub(search, replace, word)
    return (matches_rule, apply_rule)

rules = []

                pattern, search, replace)) 
```

1.  `build_match_and_apply_functions()` 函数没有发生变化。仍然使用了闭合技术：通过外部函数中定义的变量来动态创建两个函数。
2.  全局的 `open()` 函数打开文件并返回一个文件对象。此例中，将要打开的文件包含了名词复数形式的模式字符串。`with` 语句创建了叫做 *context【上下文】*的东西：当 `with` 块结束时，Python 将自动关闭文件，即便是在 `with` 块中引发了例外也会这样。在 《文件》 一章中将学到关于 `with` 块和文件对象的更多内容。
3.  `for line in &lt;fileobject&gt;` 代码从打开的文件中读取数据，并将文本赋值给 `line` 变量。在 《文件》 一章中将学到更多关于读取文件的内容。
4.  文件中每行都有三个值，单它们通过空白分隔（制表符或空白，没有区别）。要将它们分开，可使用字符串方法 `split()` 。`split()` 方法的第一个参数是 `None`，表示“对任何空白字符进行分隔（制表符或空白，没有区别）”。第二个参数是 `3`，意思是“针对空白分隔三次，丢弃该行剩下的部分。”像 `[sxz]$ $ es` 这样的行将被分割为列表 `['[sxz]$', '$', 'es']`，意思是 `pattern` 获得值 `'[sxz]$'`， `search` 获得值 `'$'`，而 `replace` 获得值 `'es'`。对于短短的一行代码来说确实威力够大的。
5.  最后，将 `pattern` 、 `search` 和 `replace` 传入 `build_match_and_apply_functions()` 函数，它将返回一个函数的元组。将该元组添加到 `rules` 列表，最终 `rules` 将储存 `plural()` 函数所预期的匹配和应用函数列表。

此处的改进是将复数形式规则独立地放到了一份外部文件中，因此可独立于使用它的代码单独对规则进行维护。代码是代码，数据是数据，生活更美好。

## 生成器

如果有个通用 `plural()` 函数解析规则文件不就更棒了吗？获取规则，检查匹配，应用相应的转换，进入下一条规则。这是 `plural()` 函数所必须完成的事，也是 `plural()` 函数必须做的事。

```py
def rules(rules_filename):
    with open(rules_filename, encoding='utf-8') as pattern_file:
        for line in pattern_file:
            pattern, search, replace = line.split(None, 3)
            yield build_match_and_apply_functions(pattern, search, replace)

def plural(noun, rules_filename='plural5-rules.txt'):
    for matches_rule, apply_rule in rules(rules_filename):
        if matches_rule(noun):
            return apply_rule(noun)
    raise ValueError('no matching rule for {0}'.format(noun)) 
```

*这段*代码到底是如何运作的？让我们先看一个交互式例子。

```py
>>> def make_counter(x):
...     print('entering make_counter')
...     while True:

...         print('incrementing x')
...         x = x + 1
... 

<generator object at 0x001C9C10>

entering make_counter
2

incrementing x
3

incrementing x
4 
```

1.  `make_counter` 中出现的 `yield` 命令的意思是这不是一个普通的函数。它是一次生成一个值的特殊类型函数。可以将其视为可恢复函数。调用该函数将返回一个可用于生成连续 `x` 值的 *生成器【Generator】*。
2.  为创建 `make_counter` 生成器的实例，仅需像调用其它函数那样对它进行调用。注意该调用并不实际执行函数代码。可以这么说，是因为 `make_counter()` 函数的第一行调用了 `print()`，但实际并未打印任何内容。
3.  该 `make_counter()` 函数返回了一个生成器对象。
4.  `next()` 函数以一个生成器对象为参数，并返回其下一个值。对 `counter` 生成器第一次调用 `next()` ，它针对第一条 `yield` 语句执行 `make_counter()` 中的代码，然后返回所产生的值。在此情况下，该代码输出将为 `2`，因其仅通过调用 `make_counter(2)` 对生成器进行初始创建。
5.  对同一生成器对象反复调用 `next()` 将确切地从上次调用的位置开始继续，直到下一条 `yield` 语句。所有的变量、局部数据等内容在 `yield` 时被保存，在 `next()` 时被恢复。下一行代码等待被执行以调用 `print()` 以打印出 `incrementing x` 。之后，执行语句 `x = x + 1`。然后它继续通过 `while` 再次循环，而它再次遇上的第一条语句是 `yield x`，该语句将保存所有一切状态，并返回当前 `x` 的值（当前为 `3`）。
6.  第二次调用 `next(counter)` 时，又进行了同样的工作，但这次 `x` 为 `4`。

由于 `make_counter` 设置了一个无限循环，理论上可以永远执行该过程，它将不断递增 `x` 并输出数值。还是让我们看一个更加实用的生成器用法。

### 斐波那奇生成器

“yield” 暂停一个函数。“next()” 从其暂停处恢复其运行。

```py
def fib(max):

    while a < max: 
```

1.  斐波那契序列是一系列的数字，每个数字都是其前两个数字之和。它从 0 和 `1` 开始，初始时上升缓慢，但越来越快。启动该序列需要两个变量：从 0 开始的 `a`，和从 `1` 开始的 `b` 。
2.  `a` 是当前序列中的数字，因此对它进行 yield 操作。
3.  `b` 是序列中下一个数字，因此将它赋值给 `a`，但同时计算下一个值 (`a + b`) 并将其赋值给 `b` 以供稍后使用。注意该步骤是并行发生的；如果 `a` 为 `3` 且 `b` 为 `5`，那么 `a, b = b, a + b` 将会把 `a` 设置 `5` （`b` 之前的值），将 `b` 设置为 `8` （ `a` 和 `b` 之前值的和）。

因此，现在有了一个连续输出斐波那契数值的函数。当然，还可以使用递归来完成该功能，但这个方式更易于阅读。同样，它也与 `for` 循环合作良好。

```py
>>> from fibonacci import fib

0 1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987

[0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610, 987] 
```

1.  可以在 `for` 循环中直接使用像 `fib()` 这样的生成器。`for` 循环将会自动调用 `next()` 函数，从 `fib()` 生成器获取数值并赋值给 `for` 循环索引变量。（`n`）
2.  每经过一次 `for` 循环， `n` 从 `fib()` 的 `yield` 语句获取一个新值，所需做的仅仅是输出它。一旦 `fib()` 的数字用尽（`a` 大于 `max`，即本例中的 `1000`）， `for` 循环将会自动退出。
3.  这是一个很有用的用法：将一个生成器传递给 `list()` 函数，它将遍历整个生成器（就像前例中的 `for` 循环）并返回所有数值的列表。

### 复数规则生成器

让我们回到 `plural5.py` 看看该版本的 `plural()` 函数是如何运作的。

```py
def rules(rules_filename):
    with open(rules_filename, encoding='utf-8') as pattern_file:
        for line in pattern_file:

def plural(noun, rules_filename='plural5-rules.txt'):

        if matches_rule(noun):
            return apply_rule(noun)
    raise ValueError('no matching rule for {0}'.format(noun)) 
```

1.  此处没有太神奇的代码。由于规则文件中每行都靠包括以空白相间的三个值，因此使用 `line.split(None, 3)` 获取三个“列”的值并将它们赋值给三个局部变量。
2.  *然后使用了 yield。* 但生产了什么呢？通过老朋友—— `build_match_and_apply_functions()` 动态创建的两个函数，这与之前的例子是一样的。换而言之， `rules()` 是*按照需求*连续生成匹配和应用函数的生成器。
3.  由于 `rules()` 是生成器，可直接在 `for` 循环中使用它。对 `for` 循环的第一次遍历，可以调用 `rules()` 函数打开模式文件，读取第一行，从该行的模式动态创建一个匹配函数和应用函数，然后生成动态创建的函数。对 `for` 循环的第二次遍历，将会精确地回到 `rules()` 中上次离开的位置（在 `for line in pattern_file` 循环的中间）。要进行的第一项工作是读取文件（仍处于打开状态）的下一行，基于该行的模式动态创建另一匹配和应用函数，然后生成两个函数。

通过第四步获得了什么呢？启动时间。在第四步中引入 `plural4` 模块时，它读取了整个模式文件，并创建了一份所有可能规则的列表，甚至在考虑调用 `plural()` 函数之前。有了生成器，可以轻松地处理所有工作：可以读取规则，创建函数并试用它们，如果该规则可用甚至可以不读取文件剩下的部分或创建更多的函数。

失去了什么？性能！每次调用 `plural()` 函数，`rules()` 生成器将从头开始——这意味着重新打开模式文件，并从头开始读取，每次一行。

要是能够两全其美多好啊：最低的启动成本（无需对 `import` 执行任何代码），*同时* 最佳的性能（无需一次次地创建同一函数）。哦，还需将规则保存在单独的文件中（因为代码和数据要泾渭分明），还有就是永远不必两次读取同一行。

要实现该目标，必须建立自己的生成器。在进行*此工作*之前，必须对 Python 的类进行学习。

## 深入阅读

*   [PEP 255: 简单生成器](http://www.python.org/dev/peps/pep-0255/)
*   [理解 Python 的 “with” 语句](http://effbot.org/zone/python-with-statement.htm)
*   [Python 中的闭合](http://ynniv.com/blog/2007/08/closures-in-python.html)
*   [斐波那契数值](http://en.wikipedia.org/wiki/Fibonacci_number)
*   [英语的不规则复数名词](http://www2.gsu.edu/~wwwesl/egw/crump.htm)