# Chapter 2 内置数据类型

# Chapter 2 内置数据类型

> " Wonder is the foundation of all philosophy, inquiry its progress, ignorance its end. " — Michel de Montaigne

## 深入

让我们暂时将 第一份 Python 程序 抛在脑后，来聊一聊数据类型。在 Python 中， 每个值都有一种数据类型，但您并不需要声明变量的数据类型。那该方式是如何运作的呢？Python 根据每个变量的初始赋值情况分析其类型，并在内部对其进行跟踪。

Python 有多种内置数据类型。以下是比较重要的一些：

1.  **Booleans［布尔型］** 或为 `True［真］` 或为 `False［假］`。
2.  **Numbers［数值型］** 可以是 Integers［整数］（`1` 和 `2`）、Floats［浮点数］（`1.1` 和 `1.2`）、Fractions［分数］（`1/2` 和 `2/3`）；甚至是 [Complex Number［复数］](http://en.wikipedia.org/wiki/Complex_number)。
3.  **Strings［字符串型］** 是 Unicode 字符序列，*例如：* 一份 HTML 文档。
4.  **Bytes［字节］** 和 **Byte Arrays［字节数组］**， *例如:* 一份 JPEG 图像文件。
5.  **Lists［列表］** 是值的有序序列。
6.  **Tuples［元组］** 是有序而不可变的值序列。
7.  **Sets［集合］** 是装满无序值的包裹。
8.  **Dictionaries［字典］** 是键值对的无序包裹。

当然，还有更多的类型。在 Python 中一切均为对象，因此存在像 *module［模块］*、 *function［函数］*、 *class［类］*、 *method［方法］*、 *file［文件］* 甚至 *compiled code［已编译代码］* 这样的类型。您已经见过这样一些例子：模块的 name、 函数的 `docstrings` *等等*。将学到的包括 《类 *与* 迭代器》 中的 Classes［类］，以及 《文件》 中的 Files［文件］。

Strings［字符串］和 Bytes［字节串］比较重要，也相对复杂，足以开辟独立章节予以讲述。让我们先看看其它类型。

## 布尔类型

在布尔类型上下文中，您几乎可以使用任何表达式。

布尔类型或为真或为假。Python 有两个被巧妙地命名为 `True` 和 `False` 的常量，可用于对布尔类型的直接赋值。表达式也可以计算为布尔类型的值。在某些地方（如 `if` 语句），Python 所预期的就是一个可计算出布尔类型值的表达式。这些地方称为 *布尔类型上下文环境*。事实上，可在布尔类型上下文环境中使用任何表达式，而 Python 将试图判断其真值。在布尔类型上下文环境中，不同的数据类型对于何值为真、何值为假有着不同的规则。（看过本章稍后的实例后，这一点将更好理解。）

例如，看看 `humansize.py` 中的这个片段：

```py
if size < 0:
    raise ValueError('number must be non-negative') 
```

`size` 是整数， 0 是整数，而 `&lt;` 是数字运算符。`size &lt; 0` 表达式的结果始终是布尔值。可在 Python 交互式 shell 中自行测试下结果：

```py
>>> size = 1
>>> size < 0
False
>>> size = 0
>>> size < 0
False
>>> size = -1
>>> size < 0
True 
```

由于 Python 2 的一些遗留问题，布尔值可以当做数值对待。`True` 为 `1`；`False` 为 0 。

```py
>>> True + True
2
>>> True - False
1
>>> True * False
0
>>> True / False
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ZeroDivisionError: int division or modulo by zero 
```

喔，喔，喔！别那么干。忘掉我刚才说的。

## 数值类型

数值类型是可畏的。有太多类型可选了。Python 同时支持 Integer［整型］ 和 Floating Point［浮点型］ 数值。无任何类型声明可用于区分；Python 通过是否有 小数 点来分辨它们。

```py
 <class 'int'>

True

2

2.0
>>> type(2.0)
<class 'float'> 
```

1.  可以使用 `type()` 函数来检测任何值或变量的类型。正如所料，`1` 为 `int` 类型。
2.  同样，还可使用 `isinstance()` 函数判断某个值或变量是否为给定某个类型。
3.  将一个 `int` 与一个 `int` 相加将得到一个 `int` 。
4.  将一个 `int` 与一个 `float` 相加将得到一个 `float` 。Python 把 `int` 强制转换为 `float` 以进行加法运算；然后返回一个 `float` 类型的结果。

### 将整数强制转换为浮点数及反向转换

正如刚才所看到的，一些运算符（如：加法）会根据需把整数强制转换为浮点数。也可自行对其进行强制转换。

```py
 2.0

2

2

-2

1.1234567890123457

<class 'int'> 
```

1.  通过调用`float()` 函数，可以显示地将 `int` 强制转换为 `float`。
2.  毫不出奇，也可以通过调用 `int()` 将 `float` 强制转换为 `int` 。
3.  `int()` 将进行取整，而不是四舍五入。
4.  对于负数，`int()` 函数朝着 0 的方法进行取整。它是个真正的取整（截断）函数，而不是 floor［地板］函数。
5.  浮点数精确到小数点后 15 位。
6.  整数可以任意大。

> ☞Python 2 对于`int［整型］` 和 `long［长整型］` 采用不同的数据类型。`int` 数据类型受到 `sys.maxint` 的限制，因平台该限制也会有所不同，但通常是 `2**32-1` 。Python 3 只有一种整数类型，其行为方式很有点像 Python 2 的旧 `long［长整数］` 类型。参阅 [PEP 237](http://www.python.org/dev/peps/pep-0237) 了解更多细节。

### 常见数值运算

对数值可进行各种类型的运算。

```py
 5.5

5

−6

5.0

121

1 
```

1.  `/` 运算符执行浮点除法。即便分子和分母都是 `int`，它也返回一个 `float` 浮点数。
2.  `//` 运算符执行古怪的整数除法。如果结果为正数，可将其视为朝向小数位取整（不是四舍五入），但是要小心这一点。
3.  当整数除以负数， `//` 运算符将结果朝着最近的整数“向上”四舍五入。从数学角度来说，由于 `−6` 比 `−5` 要小，它是“向下”四舍五入，如果期望将结果取整为 `−5`，它将会误导你。
4.  `//` 运算符并非总是返回整数结果。如果分子或者分母是 `float`，它仍将朝着最近的整数进行四舍五入，但实际返回的值将会是 `float` 类型。
5.  `**` 运算符的意思是“计算幂”，`11**2` 结果为 `121` 。
6.  `%` 运算符给出了进行整除之后的余数。`11` 除以 `2` 结果为 `5` 以及余数 `1`，因此此处的结果为 `1`。

> ☞在 Python 2 中，运算符 `/` 通常表示整数除法，但是可以通过在代码中加入特殊指令，使其看起来像浮点除法。在 Python 3 中，`/` 运算符总是表示浮点除法。参阅 [PEP 238](http://www.python.org/dev/peps/pep-0238/) 了解更多细节。

### 分数

Python 并不仅仅局限于整数和浮点数类型。它可以完成你在高中阶段学过、但几乎已经全部忘光的所有古怪数学运算。

```py
 >>> x
Fraction(1, 3)

Fraction(2, 3)

Fraction(3, 2)

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "fractions.py", line 96, in __new__
    raise ZeroDivisionError('Fraction(%s, 0)' % numerator)
ZeroDivisionError: Fraction(0, 0) 
```

1.  为启用 fractions 模块，必先引入 `fractions` 模块。
2.  为定义一个分数，创建一个 `Fraction` 对象并传入分子和分母。
3.  可对分数进行所有的常规数学计算。运算返回一个新的 `Fraction` 对象。`2 * (1/3) = (2/3)`
4.  `Fraction` 对象将会自动进行约分。`(6/4) = (3/2)`
5.  在杜绝创建以零为分母的分数方面，Python 有着良好的敏感性。

### 三角函数

还可在 Python 中进行基本的三角函数运算。

```py
>>> import math

3.1415926535897931

1.0

0.99999999999999989 
```

1.  `math` 模块中有一个代表 π 的常量，表示圆的周长与直径之比率（圆周率）。
2.  `math` 模块包括了所有的基本三角函数，包括：`sin()`、 `cos()`、`tan()` 及像 `asin()` 这样的变体函数。
3.  然而要注意的是 Python 并不支持无限精度。`tan(π / 4)` 将返回 `1.0`，而不是 `0.99999999999999989`。

### 布尔上下文环境中的数值

零值是 false［假］，非零值是 true［真］。

可以在 `if` 这样的 布尔类型上下文环境中 使用数值。零值是 false［假］，非零值是 true［真］。

```py
 ...   if anything:
...     print("yes, it's true")
...   else:
...     print("no, it's false")
...

yes, it's true
>>> is_it_true(-1)
yes, it's true
>>> is_it_true(0)
no, it's false

yes, it's true
>>> is_it_true(0.0)
no, it's false
>>> import fractions

yes, it's true
>>> is_it_true(fractions.Fraction(0, 1))
no, it's false 
```

1.  您知道可以在 Python 交互式 Shell 中定义自己的函数吗？只需在每行的结尾按 `回车键` ，然后在某一空行按 `回车键` 结束。
2.  在布尔类型上下文环境中，非零整数为真；零为假。
3.  非零浮点数为真； `0.0` 为假。请千万小心这一点！如果有轻微的四舍五入偏差（正如在前面小节中看到的那样，这并非不可能的事情），那么 Python 将测试 `0.0000000000001` 而不是 0 ，并将返回一个 `True` 值。
4.  分数也可在布尔类型上下文环境中使用。无论 `n` 为何值，`Fraction(0, n)` 为假。所有其它分数为真。

## 列表

列表是 Python 的主力数据类型。当提到 “列表 ”时，您脑海中可能会闪现“必须进一步声明大小的数组，只能包含同一类对象“ 等想法。千万别这么想。列表比那要酷得多。

> ☞ Python 中的列表类似 Perl 5 中的数组。在 Perl 5 中，存储数组的变量总是以字符 `@` 开头；在 Python 中，变量可随意命名，Python 仅在内部对数据类型进行跟踪。
> 
> ☞ Python 中的列表更像 Java 中的数组（尽管可以把列表当做生命中所需要的一切来使用)。一个更好的比喻可能是 `ArrayList` 类，该类可以容纳任何对象，并可在添加新元素时进行动态拓展。

### 创建列表

列表创建非常轻松：使用中括号包裹一系列以逗号分割的值即可。

```py
 >>> a_list
['a', 'b', 'mpilgrim', 'z', 'example']

'a'

'example'

'example'

'mpilgrim' 
```

1.  首先，创建一个包含 5 个元素的列表。要注意的是它们保持了最初的顺序。这并不是偶然的。列表是元素的有序集合。
2.  列表可当做以零为基点的数组使用。非空列表的首个元素始终是 `a_list[0]` 。
3.  该 5 元素列表的最后一个元素是 `a_list[4]`，因为列表（索引）总是以零为基点的。
4.  使用负索引值可从列表的尾部向前计数访问元素。任何非空列表的最后一个元素总是 `a_list[-1]` 。
5.  如果负数令你混淆，可将其视为如下方式： `a_list[-`n`] == a_list[len(a_list) -`n`]` 。因此在此列表中， `a_list[-3] == a_list[5 - 3] == a_list[2]`。

### 列表切片

a_list[0] 是列表的第一个元素。

定义列表后，可从其中获取任何部分作为新列表。该技术称为对列表进行 *切片* 。

```py
>>> a_list
['a', 'b', 'mpilgrim', 'z', 'example']

['b', 'mpilgrim']

['b', 'mpilgrim', 'z']

['a', 'b', 'mpilgrim']

['a', 'b', 'mpilgrim']

['z', 'example']

['a', 'b', 'mpilgrim', 'z', 'example'] 
```

1.  通过指定两个索引值，可以从列表中获取称作“切片”的某个部分。返回值是一个新列表，它包含列表(??切片)中所有元素，按顺序从第一个切片索引开始（本例中为 `a_list[1]`），截止但不包含第二个切片索引（本例中的 `a_list[3]`）。
2.  如果切片索引之一或两者均为负数，切片操作仍可进行。如果有帮助的话，您可以这么思考：自左向右读取列表，第一个切片索引指明了想要的第一个元素，第二个切片索引指明了第一个不想要的元素。返回值是两者之间的任何值。 between.
3.  列表是以零为起点的，因此 `a_list[0:3]` 返回列表的头三个元素，从 `a_list[0]` 开始，截止到但不包括 `a_list[3]` 。
4.  如果左切片索引为零，可以将其留空而将零隐去。因此 `a_list[:3]` 与 `a_list[0:3]` 是完全相同的，因为起点 0 被隐去了。
5.  同样，如果右切片索引为列表的长度，也可以将其留空。因此 `a_list[3:]` 与 `a_list[3:5]` 是完全相同的，因为该列表有五个元素。此处有个好玩的对称现象。在这个五元素列表中， `a_list[:3]` 返回头三个元素，而 `a_list[3:]` 返回最后两个元素。事实上，无论列表的长度是多少, `a_list[:`n`]` 将返回头 `n` 个元素，而 `a_list[`n`:]` 返回其余部分。
6.  如果两个切片索引都留空，那么将包括列表所有的元素。但该返回值与最初的 `a_list` 变量并不一样。它是一个新列表，只不过恰好拥有完全相同的元素而已。`a_list[:]` 是对列表进行复制的一条捷径。

### 向列表中新增项

有四种方法可用于向列表中增加元素。

```py
>>> a_list = ['a']

['a', 2.0, 3]

>>> a_list
['a', 2.0, 3, True]

>>> a_list
['a', 2.0, 3, True, 'four', 'Ω']

>>> a_list
['Ω', 'a', 2.0, 3, True, 'four', 'Ω'] 
```

1.  `+` 运算符连接列表以创建一个新列表。列表可包含任何数量的元素；没有大小限制（除了可用内存的限制）。然而，如果内存是个问题，那就必须知道在进行连接操作时，将在内存中创建第二个列表。在该情况下，新列表将会立即被赋值给已有变量 `a_list` 。因此，实际上该行代码包含两个步骤 — 连接然后赋值 — 当处理大型列表时，该操作可能（暂时）消耗大量内存。
2.  列表可包含任何数据类型的元素，单个列表中的元素无须全为同一类型。下面的列表中包含一个字符串、一个浮点数和一个整数。
3.  `append()` 方法向列表的尾部添加一个新的元素。（现在列表中有 *四种* 不同数据类型！）
4.  列表是以类的形式实现的。“创建”列表实际上是将一个类实例化。因此，列表有多种方法可以操作。`extend()` 方法只接受一个列表作为参数，并将该参数的每个元素都添加到原有的列表中。
5.  `insert()` 方法将单个元素插入到列表中。第一个参数是列表中将被顶离原位的第一个元素的位置索引。列表中的元素并不一定要是唯一的；比如说：现有两个各自独立的元素，其值均为 `'Ω'`:，第一个元素 `a_list[0]` 以及最后一个元素 `a_list[6]` 。

> ☞``a_list`.insert(0, `value`)`就像是 Perl 中的`unshift()` 函数。它将一个元素添加到列表的头部，所有其它的元素都被顶理原先的位置以腾出空间。

让我们进一步看看 `append()` 和 `extend()` 的区别。

```py
>>> a_list = ['a', 'b', 'c']

>>> a_list
['a', 'b', 'c', 'd', 'e', 'f']

6
>>> a_list[-1]
'f'

>>> a_list
['a', 'b', 'c', 'd', 'e', 'f', ['g', 'h', 'i']]

7
>>> a_list[-1]
['g', 'h', 'i'] 
```

1.  `extend()` 方法只接受一个参数，而该参数总是一个列表，并将列表 `a_list` 中所有的元素都添加到该列表中。
2.  如果开始有个 3 元素列表，然后将它与另一个 3 元素列表进行 extend 操作，结果是将获得一个 6 元素列表。
3.  另一方面， `append()` 方法只接受一个参数，但可以是任何数据类型。在此，对一个 3 元素列表调用 `append()` 方法。
4.  如果开始的时候有个 6 元素列表，然后将一个列表 append［添加］上去，结果就会……得到一个 7 元素列表。为什么是 7 个？因为最后一个元素（刚刚 append［添加］ 的元素） *本身是个列表* 。列表可包含任何类型的数据，包括其它列表。这可能是你所需要的结果，也许不是。但如果这就是你想要的，那这就是你所得到的。

### 在列表中检索值

```py
>>> a_list = ['a', 'b', 'new', 'mpilgrim', 'new']

2

True
>>> 'c' in a_list
False

3

2

Traceback (innermost last):
  File "<interactive input>", line 1, in ?ValueError: list.index(x): x not in list 
```

1.  如你所期望， `count()` 方法返回了列表中某个特定值出现的次数。
2.  如果你想知道的是某个值是否出现在列表中， `in` 运算符将会比使用 `count()` 方法要略快一些。`in` 运算符总是返回 `True` 或 `False`；它不会告诉你该值出现在什么位置。
3.  如果想知道某个值在列表中的精确位置，可调用 `index()` 方法。尽管可以通过第二个参数（以 0 为基点的）索引值来指定起点，通过第三个参数（以 0 基点的）索引来指定搜索终点，但缺省情况下它将搜索整个列表，
4.  `index()` 方法将查找某值在列表中的*第一次*出现。在该情况下，`'new'` 在列表中出现了两次，分别为 `a_list[2]` 和 `a_list[4]`，但 `index()` 方法将只返回第一次出现的位置索引值。
5.  可能 *出乎* 您的预期，如果在列表中没有找到该值，`index()` 方法将会引发一个例外。

等等，什么？是这样的：如果没有在列表中找到该值， `index()` 方法将会引发一个例外。这是 Python 语言最显著不同之处，其它多数语言将会返回一些无效的索引值（像是 `-1`）。当然，一开始这一点看起来比较讨厌，但我想您会逐渐欣赏它。这意味着您的程序将会在问题的源头处崩溃，而不是之后奇怪地、默默地崩溃。请记住， `-1` 是合法的列表索引值。如果 `index()` 方法返回 `-1`，可能会导致调整过程变得不那么有趣！

### 从列表中删除元素

列表永远不会有缝隙。

列表可以自动拓展或者收缩。您已经看到了拓展部分。也有几种方法可从列表中删除元素。

```py
>>> a_list = ['a', 'b', 'new', 'mpilgrim', 'new']
>>> a_list[1]
'b'

>>> a_list
['a', 'new', 'mpilgrim', 'new']

'new' 
```

1.  可使用 `del` 语句从列表中删除某个特定元素。
2.  删除索引 `1` 之后再访问索引 `1` 将 *不会* 导致错误。被删除元素之后的所有元素将移动它们的位置以“填补”被删除元素所产生的“缝隙”。

不知道位置索引？这不成问题，您可以通过值而不是索引删除元素。

```py
 >>> a_list
['a', 'mpilgrim', 'new']

>>> a_list
['a', 'mpilgrim']
>>> a_list.remove('new')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: list.remove(x): x not in list 
```

1.  还可以通过 `remove()` 方法从列表中删除某个元素。`remove()` 方法接受一个 *value* 参数，并删除列表中该值的第一次出现。同样，被删除元素之后的所有元素将会将索引位置下移，以“填补缝隙”。列表永远不会有“缝隙”。
2.  您可以尽情地调用 `remove()` 方法，但如果试图删除列表中不存在的元素，它将引发一个例外。

### Removing Items From A List: Bonus Round

另一有趣的列表方法是 `pop()` 。`pop()` 方法是从列表删除元素的另一方法，但有点变化。

```py
>>> a_list = ['a', 'b', 'new', 'mpilgrim']

'mpilgrim'
>>> a_list
['a', 'b', 'new']

'b'
>>> a_list
['a', 'new']
>>> a_list.pop()
'new'
>>> a_list.pop()
'a'

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: pop from empty list 
```

1.  如果不带参数调用， `pop()` 列表方法将删除列表中最后的元素，*并返回所删除的值*。
2.  可以从列表中 pop［弹出］任何元素。只需传给 `pop()` 方法一个位置索引值。它将删除该元素，将其后所有元素移位以“填补缝隙”,然后返回它删除的值。
3.  对空列表调用 `pop()` 将会引发一个例外。

> ☞不带参数调用的 `pop()` 列表方法就像 Perl 中的 `pop()` 函数。它从列表中删除最后一个元素并返回所删除元素的值。Perl 还有另一个函数 `shift()`，可用于删除第一个元素并返回其值；在 Python 中，该函数相当于 ``a_list`.pop(0)` 。

### 布尔上下文环境中的列表

空列表为假；其它所有列表为真。

可以在 `if` 这样的 布尔类型上下文环境中 使用列表。

```py
>>> def is_it_true(anything):
...   if anything:
...     print("yes, it's true")
...   else:
...     print("no, it's false")
...

no, it's false

yes, it's true

yes, it's true 
```

1.  在布尔类型上下文环境中，空列表为假值。
2.  任何至少包含一个上元素的列表为真值。
3.  任何至少包含一个上元素的列表为真值。元素的值无关紧要。

## 元组

元素 是不可变的列表。一旦创建之后，用任何方法都不可以修改元素。

```py
 >>> a_tuple
('a', 'b', 'mpilgrim', 'z', 'example')

'a'

'example'

('b', 'mpilgrim') 
```

1.  元组的定义方式和列表相同，除了整个元素的集合都用圆括号，而不是方括号闭合。
2.  和列表一样，元组的元素都有确定的顺序。元组的索引也是以零为基点的，和列表一样，因此非空元组的第一个元素总是 `a_tuple[0]` 。
3.  负的索引从元组的尾部开始计数，这和列表也是一样的。
4.  和列表一样，元组也可以进行切片操作。对列表切片可以得到新的列表；对元组切片可以得到新的元组。

元组和列表的主要区别是元组不能进行修改。用技术术语来说，元组是 不可变更 的。从实践的角度来说，没有可用于修改元组的方法。列表有像 `append()`、 `extend()`、 `insert()`、`remove()` 和 `pop()` 这样的方法。这些方法，元组都没有。可以对元组进行切片操作（因为该方法创建一个新的元组），可以检查元组是否包含了特定的值（因为该操作不修改元组），还可以……就那么多了。

```py
# continued from the previous example
>>> a_tuple
('a', 'b', 'mpilgrim', 'z', 'example')

Traceback (innermost last):
  File "<interactive input>", line 1, in ?AttributeError: 'tuple' object has no attribute 'append'

Traceback (innermost last):
  File "<interactive input>", line 1, in ?AttributeError: 'tuple' object has no attribute 'remove'

4

True 
```

1.  无法向元组添加元素。元组没有 `append()` 或 `extend()` 方法。
2.  不能从元组中删除元素。元组没有 `remove()` 或 `pop()` 方法。
3.  *可以* 在元组中查找元素，由于该操作不改变元组。
4.  还可以使用 `in` 运算符检查某元素是否存在于元组中。

那么元组有什么好处呢？

*   元组的速度比列表更快。如果定义了一系列常量值，而所需做的仅是对它进行遍历，那么请使用元组替代列表。
*   对不需要改变的数据进行“写保护”将使得代码更加安全。使用元组替代列表就像是有一条隐含的 `assert` 语句显示该数据是常量，特别的想法（及特别的功能）必须重写。（？？）
*   一些元组可用作字典键（特别是包含字符串、数值和其它元组这样的*不可变*数据的元组）。列表永远不能当做字典键使用，因为列表不是不可变的。

> ☞元组可转换成列表，反之亦然。内建的 `tuple()` 函数接受一个列表参数，并返回一个包含同样元素的元组，而 `list()` 函数接受一个元组参数并返回一个列表。从效果上看， `tuple()` 冻结列表，而 `list()` 融化元组。

### 布尔上下文环境中的元组

可以在 `if` 这样的 布尔类型上下文环境中 使用元组。

```py
>>> def is_it_true(anything):
...   if anything:
...     print("yes, it's true")
...   else:
...     print("no, it's false")
...

no, it's false

yes, it's true

yes, it's true

<class 'bool'>
>>> type((False,))
<class 'tuple'> 
```

1.  在布尔类型上下文环境中，空元组为假值。
2.  任何至少包含一个上元素的元组为真值。
3.  任何至少包含一个上元素的元组为真值。元素的值无关紧要。不过此处的逗号起什么作用呢？
4.  为创建单元素元组，需要在值之后加上一个逗号。没有逗号，Python 会假定这只是一对额外的圆括号，虽然没有害处，但并不创建元组。

### 同时赋多个值

以下是一种很酷的编程捷径：在 Python 中，可使用元组来一次赋多值。

```py
>>> v = ('a', 2, True)

>>> x
'a'
>>> y
2
>>> z
True 
```

1.  `v` 是一个三元素的元组，而 `(x, y, z)` 是包含三个变量的元组。将其中一个赋值给另一个将会把 `v` 中的每个值按顺序赋值给每一个变量。

该特性有多种用途。假设需要将某个名称指定某个特定范围的值。可以使用内建的 `range()` 函数进行多变量赋值以快速地进行连续变量赋值。

```py
 0
>>> TUESDAY
1
>>> SUNDAY
6 
```

1.  内建的 `range()` 函数构造了一个整数序列。（从技术上来说， `range()` 函数返回的既不是列表也不是元组，而是一个 迭代器，但稍后您将学到它们的区别。） `MONDAY`、 `TUESDAY`、 `WEDNESDAY`、 `THURSDAY`、 `FRIDAY`、 `SATURDAY` 和 `SUNDAY` 是您所定义的变量。（本例来自于 `calendar` 模块，该短小而有趣的模块打印日历，有点像 UNIX 程序 `cal` 。该 `calendar` 模块为星期数定义了整数常量。
2.  现在，每个变量都有其值了： `MONDAY` 为 0， `TUESDAY` 为 `1`，如此类推。

还可以使用多变量赋值创建返回多值的函数，只需返回一个包含所有值的元组。调用者可将返回值视为一个简单的元组，或将其赋值给不同的变量。许多标准 Python 类库这么干，包括在下一章将学到的 `os` 模块。

## 集合

集合 set 是装有独特值的无序“袋子”。一个简单的集合可以包含任何数据类型的值。如果有两个集合，则可以执行像联合、交集以及集合求差等标准集合运算。

### 创建集合

重中之重。创建集合非常简单。

```py
 >>> a_set
{1}

<class 'set'>

>>> a_set
{1, 2} 
```

1.  要创建只包含一个值的集合，仅需将该值放置于花括号之间。（`{}`）。
2.  实际上，集合以 类 的形式实现，但目前还无须考虑这一点。
3.  要创建多值集合，请将值用逗号分开，并用花括号将所有值包裹起来。

还可以 列表 为基础创建集合。

```py
>>> a_list = ['a', 'b', 'mpilgrim', True, False, 42]

{'a', False, 'b', True, 'mpilgrim', 42}

['a', 'b', 'mpilgrim', True, False, 42] 
```

1.  要从列表创建集合，可使用 `set()` 函数。（懂得如何实现集合的学究可能指出这实际上并不是调用某个函数，而是对某个类进行实例化。我*保证*在本书稍后的地方将会学到其中的区别。目前而言，仅需知道 `set()` 行为与函数类似，以及它返回一个集合。）
2.  正如我之前提到的，简单的集合可以包括任何数据类型的值。而且，如我之前所提到的，集合是 *无序的*。该集合并不记得用于创建它的列表中元素的最初顺序。如果向集合中添加元素，它也不会记得添加的顺序。
3.  初始的列表并不会发生变化。

还没有任何值？没有问题。可以创建一个空的集合。

```py
 set()

<class 'set'>

0

>>> type(not_sure)
<class 'dict'> 
```

1.  要创建空集合，可不带参数调用 `set()` 。
2.  打印出来的空集合表现形式看起来有点儿怪。也许，您期望看到一个 `{}` 吧 ？该符号表示一个空的字典，而不是一个空的集合。本章稍后您将学到关于字典的内容。
3.  尽管打印出的形式奇怪，这 *确实是* 一个集合……
4.  …… 同时该集合没有任何成员。
5.  由于从 Python 2 沿袭而来历史的古怪规定，不能使用两个花括号来创建空集合。该操作实际创建一个空字典，而不是一个空集合。

### 修改集合

有两种方法可向现有集合中添加值： `add()` 方法和 `update()` 方法。

```py
>>> a_set = {1, 2}

>>> a_set
{1, 2, 4}

3

>>> a_set
{1, 2, 4}

3 
```

1.  `add()` 方法接受单个可以是任何数据类型的参数，并将该值添加到集合之中。
2.  该集合现在有三个成员了。
3.  集合是装 *唯一值* 的袋子。如果试图添加一个集合中已有的值，将不会发生任何事情。将不会引发一个错误；只是一条空操作。
4.  该集合 *仍然* 只有三个成员。

```py
>>> a_set = {1, 2, 3}
>>> a_set
{1, 2, 3}

{1, 2, 3, 4, 6}

>>> a_set
{1, 2, 3, 4, 5, 6, 8, 9, 13}

>>> a_set
{1, 2, 3, 4, 5, 6, 8, 9, 10, 13, 20, 30} 
```

1.  `update()` 方法仅接受一个集合作为参数，并将其所有成员添加到初始列表中。其行为方式就像是对参数集合中的每个成员调用 `add()` 方法。
2.  由于集合不能包含重复的值，因此重复的值将会被忽略。
3.  实际上，可以带任何数量的参数调用 `update()` 方法。如果调用时传递了两个集合， `update()` 将会被每个集合中的每个成员添加到初始的集合当中（丢弃重复值）。
4.  `update()` 方法还可接受一些其它数据类型的对象作为参数，包括列表。如果调用时传入列表，`update()` 将会把列表中所有的元素添加到初始集合中。

### 从集合中删除元素

有三种方法可以用来从集合中删除某个值。前两种，`discard()` 和 `remove()` 有细微的差异。

```py
>>> a_set = {1, 3, 6, 10, 15, 21, 28, 36, 45}
>>> a_set
{1, 3, 36, 6, 10, 45, 15, 21, 28}

>>> a_set
{1, 3, 36, 6, 45, 15, 21, 28}

>>> a_set
{1, 3, 36, 6, 45, 15, 21, 28}

>>> a_set
{1, 3, 36, 6, 45, 15, 28}

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 21 
```

1.  `discard()` 接受一个单值作为参数，并从集合中删除该值。
2.  如果针对一个集合中不存在的值调用 `discard()` 方法，它不进行任何操作。不产生错误；只是一条空指令。
3.  `remove()` 方法也接受一个单值作为参数，也从集合中将其删除。
4.  区别在这里：如果该值不在集合中，`remove()` 方法引发一个 `KeyError` 例外。

就像列表，集合也有个 `pop()` 方法。

```py
>>> a_set = {1, 3, 6, 10, 15, 21, 28, 36, 45}

1
>>> a_set.pop()
3
>>> a_set.pop()
36
>>> a_set
{6, 10, 45, 15, 21, 28}

>>> a_set
set()

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'pop from an empty set' 
```

1.  `pop()` 方法从集合中删除某个值，并返回该值。然而，由于集合是无序的，并没有“最后一个”值的概念，因此无法控制删除的是哪一个值。它基本上是随机的。
2.  `clear()` 方法删除集合中 *所有* 的值，留下一个空集合。它等价于 `a_set = set()`，该语句创建一个新的空集合，并用之覆盖 `a_set` 变量的之前的值。
3.  试图从空集合中弹出某值将会引发 `KeyError` 例外。

### 常见集合操作

Python 的 `集合` 类型支持几种常见的运算。

```py
>>> a_set = {2, 4, 5, 9, 12, 21, 30, 51, 76, 127, 195}

True
>>> 31 in a_set
False
>>> b_set = {1, 2, 3, 5, 6, 8, 9, 12, 15, 17, 18, 21}

{1, 2, 195, 4, 5, 6, 8, 12, 76, 15, 17, 18, 3, 21, 30, 51, 9, 127}

{9, 2, 12, 5, 21}

{195, 4, 76, 51, 30, 127}

{1, 3, 4, 6, 8, 76, 15, 17, 18, 195, 127, 30, 51} 
```

1.  要检测某值是否是集合的成员，可使用 `in` 运算符。其工作原理和列表的一样。
2.  `union()` 方法返回一个新集合，其中装着 *在两个* 集合中出现的元素。
3.  `intersection()` 方法返回一个新集合，其中装着 *同时* 在两个集合中出现的所有元素。
4.  `difference()` 方法返回的新集合中，装着所有在 `a_set` 出现但未在 `b_set` 中的元素。
5.  `symmetric_difference()` 方法返回一个新集合，其中装着所有 *只在其中一个* 集合中出现的元素。

这三种方法是对称的。

```py
# continued from the previous example

{3, 1, 195, 4, 6, 8, 76, 15, 17, 18, 51, 30, 127}

True

True

True

False 
```

1.  `a_set` 与 `b_set` 的对称差分 *看起来* 和`b_set` 与 `a_set` 的对称差分不同，但请记住：集合是无序的。任何两个包含所有同样值（无一遗漏）的集合可认为是相等的。
2.  而这正是这里发生的事情。不要被 Python Shell 对这些集合的输出形式所愚弄了。它们包含相同的值，因此是相等的。
3.  对两个集合的 Union［并集］操作也是对称的。
4.  对两个集合的 Intersection［交集］操作也是对称的。
5.  对两个集合的 Difference［求差］操作不是对称的。这是有意义的；它类似于从一个数中减去另一个数。操作数的顺序会导致结果不同。

最后，有几个您可能会问到的问题。

```py
>>> a_set = {1, 2, 3}
>>> b_set = {1, 2, 3, 4}

True

True

>>> a_set.issubset(b_set)
False
>>> b_set.issuperset(a_set)
False 
```

1.  `a_set` 是 `b_set` 的 子集 — 所有 `a_set` 的成员均为 `b_set` 的成员。
2.  同样的问题反过来说， `b_set` 是 `a_set` 的 超集，因为 `a_set` 的所有成员均为 `b_set` 的成员。
3.  一旦向 `a_set` 添加一个未在 `b_set` 中出现的值，两项测试均返回 `False` 。

### 布尔上下文环境中的集合

可在 `if` 这样的 布尔类型上下文环境中 使用集合。

```py
>>> def is_it_true(anything):
...   if anything:
...     print("yes, it's true")
...   else:
...     print("no, it's false")
...

no, it's false

yes, it's true

yes, it's true 
```

1.  在布尔类型上下文环境中，空集合为假值。
2.  任何至少包含一个上元素的集合为真值。
3.  任何至少包含一个上元素的集合为真值。元素的值无关紧要。

## 字典

字典 是键值对的无序集合。向字典添加一个键的同时，必须为该键增添一个值。（之后可随时修改该值。） Python 的字典为通过键获取值进行了优化，而不是反过来。

> ☞Python 中的字典与 Perl 5 中的 hash [散列]类似。在 Perl 5 中，散列存储的变量总是以一个 `%` 符开头。在 Python 中，变量可以随意命名，而 Python 内部跟踪其数据类型。

### 创建字典

创建字典非常简单。其语法与 集合 的类似，但应当指定键值对而不是值。有了字典后，可以通过键来查找值。

```py
 >>> a_dict
{'server': 'db.diveintopython3.org', 'database': 'mysql'}

'db.diveintopython3.org'

'mysql'

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'db.diveintopython3.org' 
```

1.  首先，通过将两个字典项指定给 `a_dict` 变量创建了一个新字典。每个字典项都是一组键值对，整个字典项集合都被大括号包裹在内。
2.  `'server'` 为键，通过 `a_dict['server']` 引用的关联值为 `'db.diveintopython3.org'` 。
3.  `'database'` 为键，通过 `a_dict['database']` 引用的关联值为 `'mysql'` 。
4.  可以通过键获取值，但不能通过值获取键。因此 `a_dict['server']` 为 `'db.diveintopython3.org'`，而 `a_dict['db.diveintopython3.org']` 会引发例外，因为 `'db.diveintopython3.org'` 并不是键。

### 修改字典

字典没有预定义的大小限制。可以随时向字典中添加新的键值对，或者修改现有键所关联的值。继续前面的例子：

```py
>>> a_dict
{'server': 'db.diveintopython3.org', 'database': 'mysql'}

>>> a_dict
{'server': 'db.diveintopython3.org', 'database': 'blog'}

{'server': 'db.diveintopython3.org', 'user': 'mark', 'database': 'blog'}

>>> a_dict
{'server': 'db.diveintopython3.org', 'user': 'dora', 'database': 'blog'}

>>> a_dict
{'User': 'mark', 'server': 'db.diveintopython3.org', 'user': 'dora', 'database': 'blog'} 
```

1.  在字典中不允许有重复的键。对现有的键赋值将会覆盖旧值。
2.  可随时添加新的键值对。该语法与修改现有值相同。
3.  新字典项（键为 `'user'`，值为 `'mark'`）出现在中间。事实上，在第一个例子中字典项按顺序出现是个巧合；现在它们不按顺序出现同样也是个巧合。
4.  对既有字典键进行赋值只会用新值替代旧值。
5.  该操作会将 `user` 键的值改回 "mark" 吗？不会！仔细看看该键——有个大写的 `U` 出现在 `"User"` 中。字典键是区分大小写的，因此该语句创建了一组新的键值对，而不是覆盖既有的字典项。对你来说它们可能是一样的，但对于 Python 而言它们是完全不同的。

### 混合值字典

字典并非只能用于字符串。字典的值可以是任何数据类型，包括整数、布尔值、任何对象，甚至是其它的字典。而且就算在同一字典中，所有的值也无须是同一类型，您可根据需要混合匹配。字典的键要严格得多，可以是字符串、整数和其它一些类型。在同一字典中也可混合、匹配使用不同数据类型的键。

实际上，您已经在 your first Python program 见过一个将非字符串用作键的字典了。

```py
SUFFIXES = {1000: ['KB', 'MB', 'GB', 'TB', 'PB', 'EB', 'ZB', 'YB'],
            1024: ['KiB', 'MiB', 'GiB', 'TiB', 'PiB', 'EiB', 'ZiB', 'YiB']} 
```

让我们在交互式 shell 中剖析一下：

```py
>>> SUFFIXES = {1000: ['KB', 'MB', 'GB', 'TB', 'PB', 'EB', 'ZB', 'YB'],
...             1024: ['KiB', 'MiB', 'GiB', 'TiB', 'PiB', 'EiB', 'ZiB', 'YiB']}

2

True

['KB', 'MB', 'GB', 'TB', 'PB', 'EB', 'ZB', 'YB']

['KiB', 'MiB', 'GiB', 'TiB', 'PiB', 'EiB', 'ZiB', 'YiB']

'TB' 
```

1.  类似 列表 和 集合 ，`len()` 函数将返回字典中键的数量。
2.  而且像列表和集合一样，可使用 `in` 运算符以测试某个特定的键是否在字典中。
3.  `1000` *是* 字典 `SUFFIXES` 的一个键；其值为一个 8 元素列表（确切地说，是 8 个字符串）。
4.  同样， `1024` 是字典 `SUFFIXES` 的键；其值也是一个 8 元素列表。
5.  由于 `SUFFIXES[1000]` 是列表，可以通过它们的 0 基点索引来获取列表中的单个元素。

### 布尔上下文环境中的字典

空字典为假值；所有其它字典为真值。

可以在 `if` 这样的 布尔类型上下文环境中 使用字典。

```py
>>> def is_it_true(anything):
...   if anything:
...     print("yes, it's true")
...   else:
...     print("no, it's false")
...

no, it's false

yes, it's true 
```

1.  在布尔类型上下文环境中，空字典为假值。
2.  至少包含一个键值对的字典为真值。

## `None`

`None` 是 Python 的一个特殊常量。它是一个 空 值。`None` 与 `False` 不同。`None` 不是 0 。`None` 不是空字符串。将 `None` 与任何非 `None` 的东西进行比较将总是返回 `False` 。

`None` 是唯一的空值。它有着自己的数据类型（`NoneType`）。可将 `None` 赋值给任何变量，但不能创建其它 `NoneType` 对象。所有值为 `None` 变量是相等的。

```py
>>> type(None)
<class 'NoneType'>
>>> None == False
False
>>> None == 0
False
>>> None == ''
False
>>> None == None
True
>>> x = None
>>> x == None
True
>>> y = None
>>> x == y
True 
```

### 布尔上下文环境中的 `None`

在 布尔类型上下文环境中， `None` 为假值，而 `not None` 为真值。

```py
>>> def is_it_true(anything):
...   if anything:
...     print("yes, it's true")
...   else:
...     print("no, it's false")
...
>>> is_it_true(None)
no, it's false
>>> is_it_true(not None)
yes, it's true 
```

## 深入阅读

*   [布尔运算](http://docs.python.org/3.1/library/stdtypes.html#boolean-operations-and-or-not)
*   [数值类型](http://docs.python.org/3.1/library/stdtypes.html#numeric-types-int-float-long-complex)
*   [序列类型](http://docs.python.org/3.1/library/stdtypes.html#sequence-types-str-unicode-list-tuple-buffer-xrange)
*   [集合类型](http://docs.python.org/3.1/library/stdtypes.html#set-types-set-frozenset)
*   [映射类型](http://docs.python.org/3.1/library/stdtypes.html#mapping-types-dict)
*   [`fractions［分数］` 模块](http://docs.python.org/3.1/library/fractions.html)
*   [`math［数学］` 模块](http://docs.python.org/3.1/library/math.html)
*   [PEP 237: 统一长整数和整数](http://www.python.org/dev/peps/pep-0237/)
*   [PEP 238: 修改除法运算符](http://www.python.org/dev/peps/pep-0238/)

您在这里: 主页 ‣ 深入 Python 3 ‣