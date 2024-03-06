# Chapter 8 高级迭代器

# Chapter 8 高级迭代器

> " Great fleas have little fleas upon their backs to bite ’em, And little fleas have lesser fleas, and so ad infinitum. " — Augustus De Morgan

## 深入

H`AWAII + IDAHO + IOWA + OHIO == STATES`. 或者，换个说法, `510199 + 98153 + 9301 + 3593 == 621246`. 我在说是方言吗？不，这只是一个谜题。

让我来给你解释一下。

```py
HAWAII + IDAHO + IOWA + OHIO == STATES
510199 + 98153 + 9301 + 3593 == 621246

H = 5
A = 1
W = 0
I = 9
D = 8
O = 3
S = 6
T = 2
E = 4 
```

像这样的谜题被称为*cryptarithms* 或者 *字母算术(alphametics)*。字母可以拼出实际的单词，而如果你把每一个字母都用`0–9`中的某一个数字代替后, 也同样可以#8220;拼出” 一个算术等式。关键的地方是找出每个字母都映射到了哪个数字。每个字母所有出现的地方都必须映射到同一个数字，数字不能重复, 并且“单词”不能以 0 开始。

最著名的字母算术谜题是`SEND + MORE = MONEY`。

在这一章中，我们将深入一个最初由 Raymond Hettinger 编写的难以置信的 Python 程序。这个程序*只用 14 行代码*来解决字母算术谜题。

[下载 `alphametics.py`]

```py
import re
import itertools

def solve(puzzle):
    words = re.findall('[A-Z]+', puzzle.upper())
    unique_characters = set(''.join(words))
    assert len(unique_characters) <= 10, 'Too many letters'
    first_letters = {word[0] for word in words}
    n = len(first_letters)
    sorted_characters = ''.join(first_letters) + \
        ''.join(unique_characters - first_letters)
    characters = tuple(ord(c) for c in sorted_characters)
    digits = tuple(ord(c) for c in '0123456789')
    zero = digits[0]
    for guess in itertools.permutations(digits, len(characters)):
        if zero not in guess[:n]:
            equation = puzzle.translate(dict(zip(characters, guess)))
            if eval(equation):
                return equation

if __name__ == '__main__':
    import sys
    for puzzle in sys.argv[1:]:
        print(puzzle)
        solution = solve(puzzle)
        if solution:
            print(solution) 
```

你可以从命令行运行这个程序。在 Linux 上, 运行情况看起来是这样的。(取决于你机器的速度，计算可能要花一些时间，而且不会有进度条。耐心等待就好了。)

```py
you@localhost:~/diveintopython3/examples$ python3 alphametics.py "HAWAII + IDAHO + IOWA + OHIO == STATES"
HAWAII + IDAHO + IOWA + OHIO = STATES
510199 + 98153 + 9301 + 3593 == 621246
you@localhost:~/diveintopython3/examples$ python3 alphametics.py "I + LOVE + YOU == DORA"
I + LOVE + YOU == DORA
1 + 2784 + 975 == 3760
you@localhost:~/diveintopython3/examples$ python3 alphametics.py "SEND + MORE == MONEY"
SEND + MORE == MONEY
9567 + 1085 == 10652 
```

## 找到一个模式所有出现的地方

字母算术谜题解决者做的第一件事是找到谜题中所有的字母(A–Z)。

```py
>>> import re

['16', '2', '4', '8']

['SEND', 'MORE', 'MONEY'] 
```

1.  `re` 模块是正则表达式的 Python 实现。它有一个漂亮的函数`findall()`，接受一个正则表达式和一个字符串作为参数，然后找出字符串中出现该模式的所有地方。在这个例子里，模式匹配的是数字序列。`findall()`函数返回所有匹配该模式的子字符串的列表。
2.  这里正则表达式匹配的是字母序列。再一次，返回值是一个列表，其中的每一个元素是匹配该正则表达式的字符串。

这是另外一个稍微复杂一点的例子。

```py
>>> re.findall(' s.*? s', "The sixth sick sheikh's sixth sheep's sick.")
[' sixth s', " sheikh's s", " sheep's s"] 
```

这是英语中[最难的绕口令](http://en.wikipedia.org/wiki/Tongue-twister)。

很惊奇？这个正则表达式寻找一个空格，一个 `s`, 然后是最短的任何字符构成的序列(`.*?`), 然后是一个空格, 然后是另一个`s`。 在输入字符串中，我看见了五个匹配:

1.  `The &lt;mark&gt;sixth s&lt;/mark&gt;ick sheikh's sixth sheep's sick.`
2.  `The sixth &lt;mark&gt;sick s&lt;/mark&gt;heikh's sixth sheep's sick.`
3.  `The sixth sick &lt;mark&gt;sheikh's s&lt;/mark&gt;ixth sheep's sick.`
4.  `The sixth sick sheikh's &lt;mark&gt;sixth s&lt;/mark&gt;heep's sick.`
5.  `The sixth sick sheikh's sixth &lt;mark&gt;sheep's s&lt;/mark&gt;ick.`

但是`re.findall()`函数值只返回了 3 个匹配。准确的说，它返回了第一，第三和第五个。为什么呢？因为*它不会返回重叠的匹配*。第一个匹配和第二个匹配是重叠的，所以第一个被返回了，第二个被跳过了。然后第三个和第四个重叠，所以第三个被返回了，第四个被跳过了。最后，第五个被返回了。三个匹配，不是五个。

这和字母算术解决者没有任何关系；我只是觉得这很有趣。

## 在序列中寻找不同的元素

Sets 使得在序列中查找不同的元素变得很简单。

```py
>>> a_list = ['The', 'sixth', 'sick', "sheik's", 'sixth', "sheep's", 'sick']

{'sixth', 'The', "sheep's", 'sick', "sheik's"}
>>> a_string = 'EAST IS EAST'

{'A', ' ', 'E', 'I', 'S', 'T'}
>>> words = ['SEND', 'MORE', 'MONEY']

'SENDMOREMONEY'

{'E', 'D', 'M', 'O', 'N', 'S', 'R', 'Y'} 
```

1.  给出一个有若干字符串组成的列表，`set()`函数返回列表中不同的字符串组成的集合。把它想象成一个`for`循环可以帮助理解。从列表出拿出第一个元素，放到集合。第二个，第三个，第四个。第五个，等等, 它已经在集合里面了，因为 Python 集合不允许重复，所以它只被列出了一次。第六个。第七个又是一个重复的，所以它只被列出了一次。原来的列表甚至不需要事先排好序。
2.  同样的技术也适用于字符串，因为一个字符串就是一个字符序列。
3.  给出一个字符串列表, `''.join(`a_list`)`将所有的字符串拼接成一个。
4.  所以，给出一个字符串列表，这行代码返回这些字符串中出现过的不重复的字符。

字母算术解决者通过这个技术来建立谜题中出现的不同字符的集合。

```py
unique_characters = set(''.join(words)) 
```

这个列表在接下来迭代可能的解法的时候将被用来将数字分配给字符。

## 作出断言

和很多编程语言一样，Python 有一个`assert`语句。这是它的用法。

```py
 Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AssertionError

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AssertionError: Only for very large values of 2 
```

1.  `assert` 语句后面跟任何合法的 Python 表达式。在这个例子里, 表达式 `1 + 1 == 2` 的求值结果为 `True`, 所以 `assert` 语句没有做任何事情。
2.  然而, 如果 Python 表达式求值结果为 `False`, `assert` 语句会抛出一个 `AssertionError`.
3.  你可以提供一个人类可读的消息，`AssertionError`异常被抛出的时候它可以被用于打印输出。

因此, 这行代码:

```py
assert len(unique_characters) <= 10, 'Too many letters' 
```

…等价于:

```py
if len(unique_characters) > 10:
    raise AssertionError('Too many letters') 
```

字母算术谜题使用这个`assert` 语句来排除谜题包含多于 10 个的不同的字母的情况。因为每个不同的字母对应一个不同的数字，而数子只有 10 个,含有多于 10 个的不同的字母的谜题是不可能有解的。

## 生成器表达式

生成表达式类似生成器函数，只不过它不是函数。

```py
>>> unique_characters = {'E', 'D', 'M', 'O', 'N', 'S', 'R', 'Y'}

<generator object <genexpr> at 0x00BADC10>

69
>>> next(gen)
68

(69, 68, 77, 79, 78, 83, 82, 89) 
```

1.  生成器表达式类似一个 yield 值的匿名函数。表达式本身看起来像列表解析, 但不是用方括号而是用圆括号包围起来。
2.  生成器表达式返回迭代器。
3.  调用 `next(`gen`)` 返回迭代器的下一个值。
4.  如果你愿意，你可以将生成器表达式传给`tuple()`, `list()`, 或者 `set()`来迭代所有的值并且返回元组，列表或者集合。在这种情况下，你不需要一对额外的括号 — 将生成器表达式`ord(c) for c in unique_characters` 传给 `tuple()` 函数就可以了, Python 会推断出它是一个生成器表达式。

> ☞使用生成器表达式取代列表解析可以同时节省 CPU 和 内存(RAM)。如果你构造一个列表的目的仅仅是传递给别的函数，(*比如* 传递给`tuple()` 或者 `set()`), 用生成器表达式替代吧!

这里是到达同样目的的另一个方法, 使用生成器函数:

```py
def ord_map(a_string):
    for c in a_string:
        yield ord(c)

gen = ord_map(unique_characters) 
```

生成器表达式功能相同但更紧凑。

## 计算排列… 懒惰的方法!

首先, 排列到底是个什么东西? 排列是一个数学概念。(取决于你在处理哪种数学，排列有好几个定义。在这里我们说的是组合数学, 如果你完全不知道组合数学是什么也不用担心。同往常一样, [维基百科是你的朋友](http://en.wikipedia.org/wiki/Permutation)。)

想法是这样的，你有某物件(可以是数字，可以是字母，也可以是跳舞的熊)的一个列表，接着找出将它们拆开然后组合成小一点的列表的所有可能。所有的小列表的大小必须一致。最小是 1，最大是元素的总数目。哦，也不能有重复。数学家说“让我们找出 3 个元素取 2 个的排列,” 意思是你有一个 3 个元素的序列，然后你找出所有可能的有序对。

```py
 (1, 2)
>>> next(perms)
(1, 3)
>>> next(perms)

>>> next(perms)
(2, 3)
>>> next(perms)
(3, 1)
>>> next(perms)
(3, 2)

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration 
```

1.  `itertools`模块里有各种各样的有趣的东西，包括`permutations()`函数，它把查找排列的所有辛苦的工作的做了。
2.  `permutations()` 函数接受一个序列(这里是 3 个数字组成的列表) 和一个表示你要的排列的元素的数目的数字。函数返回迭代器，你可以在`for` 循环或其他老地方使用它。这里我遍历迭代器来显示所有的值。
3.  `[1, 2, 3]`取 2 个的第一个排列是`(1, 2)`。
4.  记住排列是有序的: `(2, 1)` 和 `(1, 2)`是不同的。
5.  这就是了。这些就是`[1, 2, 3]`取两个的所有排列。像`(1, 1)` 或者 `(2, 2)`这样的元素对没有出现，因为它们包含重复导致它们不是合法的排列。当没有更多排列的时候，迭代器抛出一个`StopIteration`异常。

`itertools`模块有各种各样的有趣的东西。

`permutations()`函数并不一定要接受列表。它接受任何序列 — 甚至是字符串。

```py
>>> import itertools

>>> next(perms)

>>> next(perms)
('A', 'C', 'B')
>>> next(perms)
('B', 'A', 'C')
>>> next(perms)
('B', 'C', 'A')
>>> next(perms)
('C', 'A', 'B')
>>> next(perms)
('C', 'B', 'A')
>>> next(perms)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration

[('A', 'B', 'C'), ('A', 'C', 'B'),
 ('B', 'A', 'C'), ('B', 'C', 'A'),
 ('C', 'A', 'B'), ('C', 'B', 'A')] 
```

1.  字符串就是一个字符序列。对于查找排列来说，字符串`'ABC'`和列表 `['A', 'B', 'C']`是等价的。
2.  `['A', 'B', 'C']`取 3 个的第一个排列是`('A', 'B', 'C')`。还有 5 个其他的排列 — 同样的 3 个字符，不同的顺序。
3.  由于`permutations()`函数总是返回迭代器，一个简单的调试排列的方法是将这个迭代器传给内建的`list()`函数来立刻看见所有的排列。

## `itertools`模块中的其它有趣的东西

```py
>>> import itertools

[('A', '1'), ('A', '2'), ('A', '3'), 
 ('B', '1'), ('B', '2'), ('B', '3'), 
 ('C', '1'), ('C', '2'), ('C', '3')]

[('A', 'B'), ('A', 'C'), ('B', 'C')] 
```

1.  `itertools.product()`函数返回包含两个序列的笛卡尔乘积的迭代器。
2.  `itertools.combinations()`函数返回包含给定序列的给定长度的所有组合的迭代器。这和`itertools.permutations()`函数很类似，除了不包含因为只有顺序不同而重复的情况。所以`itertools.permutations('ABC', 2)`同时返回`('A', 'B')` and `('B', 'A')` (同其它的排列一起), `itertools.combinations('ABC', 2)` 不会返回`('B', 'A')` ，因为它和`('A', 'B')`是重复的，只是顺序不同而已。

[下载 `favorite-people.txt`]

```py
 >>> names
['Dora\n', 'Ethan\n', 'Wesley\n', 'John\n', 'Anne\n',
'Mike\n', 'Chris\n', 'Sarah\n', 'Alex\n', 'Lizzie\n']

>>> names
['Dora', 'Ethan', 'Wesley', 'John', 'Anne',
'Mike', 'Chris', 'Sarah', 'Alex', 'Lizzie']

>>> names
['Alex', 'Anne', 'Chris', 'Dora', 'Ethan',
'John', 'Lizzie', 'Mike', 'Sarah', 'Wesley']

>>> names
['Alex', 'Anne', 'Dora', 'John', 'Mike',
'Chris', 'Ethan', 'Sarah', 'Lizzie', 'Wesley'] 
```

1.  这个表达式将文本内容以一行一行组成的列表的形式返回。
2.  不幸的是，(对于这个例子来说), `list(open(`filename`))` 表达式返回的每一行的末尾都包含回车。这个列表解析使用`rstrip()` 字符串方法移除每一行尾部的空白。(字符串也有一个`lstrip()`方法移除头部的空白，以及`strip()`方法头尾都移除。)
3.  `sorted()` 函数接受一个列表并将它排序后返回。默认情况下，它按字母序排序。
4.  然而，`sorted()`函数也接受一个函数作为`key` 参数, 并且使用 key 来排序。在这个例子里，排序函数是`len()`,所以它按`len(`each item`)`来排序。短的名字排在前面，然后是稍长，接着是更长的。

这和`itertools`模块有什么关系? 很高兴你问了这个问题。

```py
…continuing from the previous interactive shell…
>>> import itertools

>>> groups
<itertools.groupby object at 0x00BB20C0>
>>> list(groups)
[(4, <itertools._grouper object at 0x00BA8BF0>),
 (5, <itertools._grouper object at 0x00BB4050>),
 (6, <itertools._grouper object at 0x00BB4030>)]

...     print('Names with {0:d} letters:'.format(name_length))
...     for name in name_iter:
...         print(name)
... 
Names with 4 letters:
Alex
Anne
Dora
John
Mike
Names with 5 letters:
Chris
Ethan
Sarah
Names with 6 letters:
Lizzie
Wesley 
```

1.  `itertools.groupby()`函数接受一个序列和一个 key 函数, 并且返回一个生成二元组的迭代器。每一个二元组包含`key_function(`each item`)`的结果和另一个包含着所有共享这个 key 结果的元素的迭代器。
2.  调用`list()` 函数会“耗尽”这个迭代器, *也就是说* 你生成了迭代器中所有元素才创造了这个列表。迭代器没有“重置”按钮。你一旦耗尽了它，你没法重新开始。如果你想要再循环一次(例如, 在接下去的`for`循环里面), 你得调用`itertools.groupby()`来创建一个新的迭代器。
3.  在这个例子里，给出一个*已经按长度排序*的名字列表, `itertools.groupby(names, len)`将会将所有的 4 个字母的名字放在一个迭代器里面，所有的 5 个字母的名字放在另一个迭代器里，以此类推。`groupby()`函数是完全通用的; 它可以将字符串按首字母，将数字按因子数目, 或者任何你能想到的 key 函数进行分组。

> ☞`itertools.groupby()`只有当输入序列已经按分组函数排过序才能正常工作。在上面的例子里面，你用`len()` 函数分组了名字列表。这能工作是因为输入列表已经按长度排过序了。

Are you watching closely?

```py
>>> list(range(0, 3))
[0, 1, 2]
>>> list(range(10, 13))
[10, 11, 12]

[0, 1, 2, 10, 11, 12]

[(0, 10), (1, 11), (2, 12)]

[(0, 10), (1, 11), (2, 12)]

[(0, 10), (1, 11), (2, 12), (None, 13)] 
```

1.  `itertools.chain()`函数接受两个迭代器，返回一个迭代器，它包含第一个迭代器的所有内容，以及跟在后面的来自第二个迭代器的所有内容。(实际上，它接受任何数目的迭代器，并把它们按传入顺序串在一起。)
2.  `zip()`函数的作用不是很常见，结果它却非常有用: 它接受任何数目的序列然后返回一个迭代器，其第一个元素是每个序列的第一个元素组成的元组，然后是每个序列的第二个元素（组成的元组），以此类推。
3.  `zip()` 在到达最短的序列结尾的时候停止。`range(10, 14)` 有四个元素(10, 11, 12, 和 13), 但是 `range(0, 3)`只有 3 个, 所以 `zip()`函数返回包含 3 个元素的迭代器。
4.  相反，`itertools.zip_longest()`函数在到达*最长的*序列的结尾的时候才停止, 对短序列结尾之后的元素填入`None`值.

好吧，这些都很有趣，但是和字母算术谜题解决者有什么联系呢? 请看下面:

```py
>>> characters = ('S', 'M', 'E', 'D', 'O', 'N', 'R', 'Y')
>>> guess = ('1', '2', '0', '3', '4', '5', '6', '7')

(('S', '1'), ('M', '2'), ('E', '0'), ('D', '3'),
 ('O', '4'), ('N', '5'), ('R', '6'), ('Y', '7'))

{'E': '0', 'D': '3', 'M': '2', 'O': '4',
 'N': '5', 'S': '1', 'R': '6', 'Y': '7'} 
```

1.  给出一个字母列表和一个数字列表(两者的元素的形式都是 1 个字符的字符串), `zip`函数按顺序创建一组组字母，数字对。
2.  为什么这很酷? 因为这个数据结构正好可以用来传递给`dict()`函数来创建以字母为键，对应数字为值的字典。(这不是实现这个目的唯一方法。你当然可以使用字典解析来直接创建字典。) 尽管字典的打印形式以另一个顺序列出了这些键值对(字典本身没有#8220;顺序” ), 但是你可以看见每一个字母都按`characters` 和 `guess`序列的原始顺序对应到了相应的数字。

算术谜题解决者使用这个技术对每一个可能的解法创建一个将谜题中的字母映射到解法中的数字的字典。

```py
characters = tuple(ord(c) for c in sorted_characters)
digits = tuple(ord(c) for c in '0123456789')
...
for guess in itertools.permutations(digits, len(characters)):
    ...
 <mark>equation = puzzle.translate(dict(zip(characters, guess)))</mark> 
```

但是`translate()`方法是什么呢? 啊哈, 我们现在到了*真正*有趣的部分了。

## 一种新的操作字符串的方法

Python 字符串有很多方法。我们在字符串章节中学习了其中一些: `lower()`, `count()`, 和 `format()`。现在我要给你介绍一个强大但鲜为人知的操作字符串的技术: `translate()` 方法。

```py
 {65: 79}

'MORK' 
```

1.  字符串翻译从一个转换表开始, 转换表就是一个将一个字符映射到另一个字符的字典。实际上，“字符” 是不正确的 — 转换表实际上是将一个 *字节（byte)*映射到另一个。
2.  记住，Python 3 中的字节是整形数。`ord()` 函数返回字符的 ASCII 码。在这个例子中，字符是 A–Z, 所以返回的是从 65 到 90 的字节。
3.  一个字符串的`translate()`方法接收一个转换表，并用它来转换该字符串。换句话说，它将出现在转换表的键中的字节替换为该键对应的值。在这个例子里， 将`MARK` “翻译为” `MORK`.

现在你开始进入*真正*有趣的部分了。

这和解决字母算术谜题有什么关系呢？实际上，关系大着呢。

```py
 >>> characters
(83, 77, 69, 68, 79, 78, 82, 89)

>>> guess
(57, 49, 53, 55, 48, 54, 56, 50)

>>> translation_table
{68: 55, 69: 53, 77: 49, 78: 54, 79: 48, 82: 56, 83: 57, 89: 50}

'9567 + 1085 == 10652' 
```

1.  使用生成器表达式, 我们快速的计算出字符串中每个字符的字节值。`characters`是`alphametics.solve()`函数中的`sorted_characters`的示例值 .
2.  使用另一个生成器表达式，我们快速的计算出字符串中每个数字的字节值。计算结果`guess`, 正好是`alphametics.solve()`函数中的`itertools.permutations()`函数返回值的格式。
3.  通过将`characters` 和 `guess`zipping 出来的元素对序列构造出的字典来作为转换表。这正是`alphametics.solve()` 在`for` 循环里面干的事情。
4.  最后我们将转换表传递给原始字符串的`translate()`方法。这会将字符串中的每个字母转化成相应的数字(基于`characters`中字母和`guess`中的数字)。结果是一个字符串形式的合法的 Python 表达式。

这相当令人难忘。但你能对正巧是一个合法 Python 表达式的字符串干什么呢？

## 将任何字符串作为 Python 表达式求值

这是谜题的最后一部分(或者说, 谜题解决者的最后一部分)。经过华丽的字符串操作，我们得到了类似`'9567 + 1085 == 10652'`这样的一个字符串。但那是一个字符串，字符串有什么好的？输入`eval()`, Python 通用求值工具。

```py
>>> eval('1 + 1 == 2')
True
>>> eval('1 + 1 == 3')
False
>>> eval('9567 + 1085 == 10652')
True 
```

但是等一下，不止这些! `eval()` 并不限于布尔表达式。它能处理*任何* Python 表达式并且返回*任何*数据类型。

```py
>>> eval('"A" + "B"')
'AB'
>>> eval('"MARK".translate({65: 79})')
'MORK'
>>> eval('"AAAAA".count("A")')
5
>>> eval('["*"] * 5')
['*', '*', '*', '*', '*'] 
```

等一下，还没完呢!

```py
>>> x = 5

25

25
>>> import math

2.2360679774997898 
```

1.  `eval()`接受的表达式可以引用在`eval()`之外定义的全局变量。如果(`eval()`)在函数内被调用, 它也可以引用局部变量。
2.  以及函数。
3.  以及模块。

喂，等一下…

```py
>>> import subprocess

'Desktop         Library         Pictures \
 Documents       Movies          Public   \
 Music           Sites' 
```

1.  `subprocess` 模块允许你执行任何 shell 命令并以字符串形式获得输出。
2.  执行任意的 shell 命令可能会导致永久的（不好的）后果。

更坏的是，由于存在全局函数`__import__()`，它接收字符串形式的模块名，导入模块，并返回模块的引用。和`eval()`的能力结合起来，你可以构造一个单独的表达式来删除你所有的文件:

1.  现在想象一下`'rm -rf ~'`的输出。实际上它不会有任何输出，但是你也不会有任何文件还留着。

eval() 是邪恶的

好吧, 邪恶部分是对来自非信任源的表达式进行求值。你应该只在信任的输入上使用`eval()`。当然，关键的部分是确定什么是“可信任的”。但有一点我敢肯定: 你**不**应该将这个字母算术表达式放到网上最为一个小的 web 服务。不要错误的认为，“Gosh, 这个函数在求值以前做了那么多的字符串操作。*我想不出* 谁能利用这个漏洞。” **会**有人找出穿过这些字符串操作把危险的可执行代码放进来的方法的。([更奇怪的事情都发生过。](http://www.securityfocus.com/blogs/746)), 然后你就得和你的服务器说再见了。

但是肯定有*某种*办法可以安全的求值表达式吧？将`eval()`放到一个不能访问和伤害外部世界的沙盒里面。嗯，对也不对。

```py
>>> x = 5

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<string>", line 1, in <module>
NameError: name 'x' is not defined

>>> import math

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<string>", line 1, in <module>
NameError: name 'math' is not defined 
```

1.  传给`eval()`函数的第二和第三个函数担当了求值表达式是的全局和局部名字空间的角色。在这个例子里，它们都是空的，意味着当字符串`"x * 5"`被求值的时候, 在全局和本地的名字空间都没有变量`x`, 所以 `eval()`抛出了一个异常。
2.  你可以通过一个个列出的方式选择性在全局名字空间里面包含一些值。这些 — 并且这有这些 — 变量在求值的时候可用。
3.  即使你刚刚导入了`math`模块, 你没有在传给`eval()`函数的名字空间里包含它，所以求值失败了。

哎呀，这很简单。 让我来做一个字母算术谜题的 Web 服务吧！

```py
 25

2.2360679774997898 
```

1.  即使你传入空的字典作为全局和局部名字空间，所有的 Python 内建函数在求值时还是可用的。所以`pow(5, 2)`可以工作, 因为 `5` 和 `2`是字面量，而`pow()`是内建函数。
2.  很不幸 (如果你不明白为什么不幸，继续读。), `__import__()` 也是一个内建函数，所以它也能工作。

是的，这意味着即使你在调用`eval()`的时候显式的将全局和局部名字空间设置为空字典，你仍然可以做坏事。

```py
>>> eval("__import__('subprocess').getoutput('rm /some/random/file')", {}, {}) 
```

哎呀. 幸亏我没有做那个字母算术 web 服务。存在*任何*安全的使用 `eval()`的方法吗? 嗯, 有也没有。

```py
>>> eval("__import__('math').sqrt(5)",

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<string>", line 1, in <module>
NameError: name '__import__' is not defined
>>> eval("__import__('subprocess').getoutput('rm -rf /')",

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<string>", line 1, in <module>
NameError: name '__import__' is not defined 
```

1.  为了安全的求值不受信任的表达式, 你需要定义一个将`"__builtins__"` 映射为 `None`(Python 的空值)的全局名字空间字典. 在内部, “内建” 函数包含在一个叫做`"__builtins__"`的伪模块内。这个伪模块(*即* 内建函数的集合) 在没有被你显式的覆盖的情况下对被求值的表达式是总是可用的。
2.  请确保你覆盖的是`__builtins__`。 不是`__builtin__`, `__built-ins__`, 或者其它某个变量，否则程序还是可以运行但是会有巨大的风险。

那么`eval()`现在安全了? 嗯，是也不是。

```py
>>> eval("2 ** 2147483647", 
```

1.  即使不能访问到`__builtins__`, 你还是可以开启一个拒绝服务攻击。例如, 试图求`2` 的 `2147483647`次方会导致你的服务器的 CPU 利用率到达 100% 一段时间。(如果你在交互式 shell 中试验这个, 请多按几次 `Ctrl-C`来跳出来。) 技术上讲，这个表达式 最终*将会*返回一个值, 但是在这段时间里你的服务器将啥也干不了。

最后, Python 表达式的求值*是*可能达到某种意义的“安全”的, 但结果是在现实生活中没什么用。如果你只是玩玩没有问题，如果你只给它传递安全的输入也没有问题。但是其它的情况完全是自找麻烦。

## 把所有东西放在一起

总的来说: 这个程序通过暴力解决字母算术谜题， *也就是*通过穷举所有可能的解法。为了达到目的，它

1.  通过`re.findall()`函数找到谜题中的所有字母
2.  使用集合和`set()`函数找到谜题出现的所有*不同*的字母
3.  通过`assert`语句检查是否有超过 10 个的不同的字母 (意味着谜题无解)
4.  通过一个生成器对象将字符转换成对应的 ASCII 码值
5.  使用`itertools.permutations()`函数计算所有可能的解法
6.  使用`translate()`字符串方法将所有可能的解转换成 Python 表达式
7.  使用`eval()`函数通过求值 Python 表达式来检验解法
8.  返回第一个求值结果为`True`的解法

…仅仅 14 行代码.

## 进一步阅读

*   [`itertools` 模块](http://docs.python.org/3.1/library/itertools.html)
*   [`itertools` — 用于高效循环的迭代器函数](http://www.doughellmann.com/PyMOTW/itertools/)
*   [观看 Raymond Hettinger 在 PyCon 2009 上的 “Easy AI with Python” 演讲](http://blip.tv/file/1947373/)
*   [Recipe 576615: Alphametics solver](http://code.activestate.com/recipes/576615/), Raymond Hettinger 的原始的适用于 Python 2 的算木谜题解决程序
*   [More of Raymond Hettinger’s recipes](http://code.activestate.com/recipes/users/178123/) in the ActiveState Code repository
*   [算木谜题在维基百科上的页面](http://en.wikipedia.org/wiki/Verbal_arithmetic)
*   [字母索引](http://www.tkcs-collins.com/truman/alphamet/index.shtml), 包含 [很多谜题](http://www.tkcs-collins.com/truman/alphamet/alphamet.shtml) 以及 [一个创建你自己的谜题的工具](http://www.tkcs-collins.com/truman/alphamet/alpha_gen.shtml)

非常感谢 Raymond Hettinger 同意重现授权他的代码，因此我才能将它移植到 Python 3 并作为本章的基础。