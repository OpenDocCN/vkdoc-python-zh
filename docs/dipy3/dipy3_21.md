# Chapter B 特殊方法名称

> " My specialty is being right when other people are wrong. " — [George Bernard Shaw](http://en.wikiquote.org/wiki/George_Bernard_Shaw)

## 深入

在本书其它几处，我们已经见识过一些特殊方法——即在使用某些语法时 Python 所调用的“神奇”方法。使用特殊方法，类用起来如同序列、字典、函数、迭代器，或甚至像个数字！本附录为我们已经见过特殊方法提供了参考，并对一些更加深奥的特殊方法进行了简要介绍。

## 基础知识

如果曾阅读 《类的简介》一章，你可能已经见识过了最常见的特殊方法： `__init__()` 方法。盖章结束时，我写的类多数需要进行一些初始化工作。还有一些其它的基础特殊方法对调试自定义类也特别有用。

| 序号 | 目的 | 所编写代码 | Python 实际调用 |
| --- | --- | --- | --- |
| ① | 初始化一个实例 | `x = MyClass()` | [`x.__init__()`](http://docs.python.org/3.1/reference/datamodel.html#object.__init__) |
| ② | 字符串的“官方”表现形式 | `repr(x)` | [`x.__repr__()`](http://docs.python.org/3.1/reference/datamodel.html#object.__repr__) |
| ③ | 字符串的“非正式”值 | [`str(x)`](http://docs.python.org/3.1/reference/datamodel.html#object.__str__) | `x.__str__()` |
| ④ | 字节数组的“非正式”值 | `bytes(x)` | `x.__bytes__()` |
| ⑤ | 格式化字符串的值 | `format(x,`format_spec`)` | [`x.__format__(`format_spec`)`](http://docs.python.org/3.1/reference/datamodel.html#object.__format__) |

1.  对 `__init__()` 方法的调用发生在实例被创建 *之后* 。如果要控制实际创建进程，请使用 `__new__()` 方法。
2.  按照约定， `__repr__()` 方法所返回的字符串为合法的 Python 表达式。
3.  在调用 `print(x)` 的同时也调用了 `__str__()` 方法。
4.  由于 `bytes` 类型的引入而*从 Python 3 开始出现*。
5.  按照约定，`format_spec` 应当遵循 [迷你语言格式规范【Format Specification Mini-Language】](http://www.python.org/doc/3.1/library/string.html#formatspec)。Python 标准类库中的 `decimal.py` 提供了自己的 `__format__()` 方法。

## 行为方式与迭代器类似的类

在 《迭代器》一章中，我们已经学习了如何使用 `__iter__()` 和 `__next__()` 方法从零开始创建迭代器。

| 序号 | 目的 | 所编写代码 | Python 实际调用 |
| --- | --- | --- | --- |
| ① | 遍历某个序列 | `iter(seq)` | [`seq.__iter__()`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__iter__) |
| ② | 从迭代器中获取下一个值 | `next(seq)` | [`seq.__next__()`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__next__) |
| ③ | 按逆序创建一个迭代器 | `reversed(seq)` | [`seq.__reversed__()`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__reversed__) |

1.  无论何时创建迭代器都将调用 `__iter__()` 方法。这是用初始值对迭代器进行初始化的绝佳之处。
2.  无论何时从迭代器中获取下一个值都将调用 `__next__()` 方法。
3.  `__reversed__()` 方法并不常用。它以一个现有序列为参数，并将该序列中所有元素从尾到头以逆序排列生成一个新的迭代器。

正如我们在 《迭代器》一章中看到的，`for` 循环也可用作迭代器。在下面的循环中：

```py
for x in seq:
    print(x) 
```

Python 3 将会调用 `seq.__iter__()` 以创建一个迭代器，然后对迭代器调用 `__next__()` 方法以获取 `x` 的每个值。当 `__next__()` 方法引发 `StopIteration` 例外时， `for` 循环正常结束。

## 计算属性

| 序号 | 目的 | 所编写代码 | Python 实际调用 |
| --- | --- | --- | --- |
| ① | 获取一个计算属性（无条件的） | `x.my_property` | [`x.__getattribute__(`'my_property'`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__getattribute__) |
| --- | --- | --- | --- |
| ② | 获取一个计算属性（后备） | `x.my_property` | [`x.__getattr__(`'my_property'`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__getattr__) |
| --- | --- | --- | --- |
| ③ | 设置某属性 | `x.my_property = value` | [`x.__setattr__(`'my_property'`,`value`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__setattr__) |
| --- | --- | --- | --- |
| ④ | 删除某属性 | `del x.my_property` | [`x.__delattr__(`'my_property'`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__delattr__) |
| --- | --- | --- | --- |
| ⑤ | 列出所有属性和方法 | `dir(x)` | [`x.__dir__()`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__dir__) |
| --- | --- | --- | --- |

1.  如果某个类定义了 `__getattribute__()` 方法，在 *每次引用属性或方法名称时* Python 都调用它（特殊方法名称除外，因为那样将会导致讨厌的无限循环）。
2.  如果某个类定义了 `__getattr__()` 方法，Python 将只在正常的位置查询属性时才会调用它。如果实例 `x` 定义了属性 `color`， `x.color` 将 *不会* 调用 `x.__getattr__('color')`；而只会返回 `x.color` 已定义好的值。
3.  无论何时给属性赋值，都会调用 `__setattr__()` 方法。
4.  无论何时删除一个属性，都将调用 `__delattr__()` 方法。
5.  如果定义了 `__getattr__()` 或 `__getattribute__()` 方法， `__dir__()` 方法将非常有用。通常，调用 `dir(x)` 将只显示正常的属性和方法。如果 `__getattr()__` 方法动态处理 `color` 属性， `dir(x)` 将不会将 `color` 列为可用属性。可通过覆盖 `__dir__()` 方法允许将 `color` 列为可用属性，对于想使用你的类但却不想深入其内部的人来说，该方法非常有益。

`__getattr__()` 和 `__getattribute__()` 方法的区别非常细微，但非常重要。可以用两个例子来解释一下：

```py
class Dynamo:
    def __getattr__(self, key):

            return 'PapayaWhip'
        else:

>>> dyn = Dynamo()

'PapayaWhip'
>>> dyn.color = 'LemonChiffon'

'LemonChiffon' 
```

1.  属性名称以字符串的形式传入 `__getattr()__` 方法。如果名称为 `'color'`，该方法返回一个值。（在此情况下，它只是一个硬编码的字符串，但可以正常地进行某些计算并返回结果。）
2.  如果属性名称未知， `__getattr()__` 方法必须引发一个 `AttributeError` 例外，否则在访问未定义属性时，代码将只会默默地失败。（从技术角度而言，如果方法不引发例外或显式地返回一个值，它将返回 `None` ——Python 的空值。这意味着 *所有* 未显式定义的属性将为 `None`，几乎可以肯定这不是你想看到的。）
3.  `dyn` 实例没有名为 `color` 的属性，因此在提供计算值时将调用 `__getattr__()` 。
4.  在显式地设置 `dyn.color` 之后，将不再为提供 `dyn.color` 的值而调用 `__getattr__()` 方法，因为 `dyn.color` 已在该实例中定义。

另一方面，`__getattribute__()` 方法是绝对的、无条件的。

```py
class SuperDynamo:
    def __getattribute__(self, key):
        if key == 'color':
            return 'PapayaWhip'
        else:
            raise AttributeError

>>> dyn = SuperDynamo()

'PapayaWhip'
>>> dyn.color = 'LemonChiffon'

'PapayaWhip' 
```

1.  在获取 `dyn.color` 的值时将调用 `__getattribute__()` 方法。
2.  即便已经显式地设置 `dyn.color`，在获取 `dyn.color` 的值时, *仍将调用* `__getattribute__()` 方法。如果存在 `__getattribute__()` 方法，将在每次查找属性和方法时 *无条件地调用* 它，哪怕在创建实例之后已经显式地设置了属性。

> ☞ 如果定义了类的 `__getattribute__()` 方法，你可能还想定义一个 `__setattr__()` 方法，并在两者之间进行协同，以跟踪属性的值。否则，在创建实例之后所设置的值将会消失在黑洞中。

必须特别小心 `__getattribute__()` 方法，因为 Python 在查找类的方法名称时也将对其进行调用。

```py
class Rastan:
    def __getattribute__(self, key):

    def swim(self):
        pass

>>> hero = Rastan()

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 3, in __getattribute__
AttributeError 
```

1.  该类定义了一个总是引发 `AttributeError` 例外的 `__getattribute__()` 方法。没有属性或方法的查询会成功。
2.  调用 `hero.swim()` 时，Python 将在 `Rastan` 类中查找 `swim()` 方法。该查找将执行整个 `__getattribute__()` 方法，因为所有的属性和方法查找都通过 `__getattribute__()` 方法。在此例中， `__getattribute__()` 方法引发 `AttributeError` 例外，因此该方法查找过程将会失败，而方法调用也将失败。

## 行为方式与函数类似的类

可以让类的实例变得可调用——就像函数可以调用一样——通过定义 `__call__()` 方法。

| 序号 | 目的 | 所编写代码 | Python 实际调用 |
| --- | --- | --- | --- |
|  | 像调用函数一样“调用”一个实例 | `my_instance()` | [`my_instance.__call__()`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__call__) |

[`zipfile` 模块](http://docs.python.org/3.1/library/zipfile.html) 通过该方式定义了一个可以使用给定密码解密 经加密 zip 文件的类。该 zip 解密 算法需要在解密的过程中保存状态。通过将解密器定义为类，使我们得以在 decryptor 类的单个实例中对该状态进行维护。状态在 `__init__()` 方法中进行初始化，如果文件 经加密 则进行更新。但由于该类像函数一样“可调用”，因此可以将实例作为 `map()` 函数的第一个参数传入，代码如下：

```py
# excerpt from zipfile.py
class _ZipDecrypter:
.
.
.
    def __init__(self, pwd):

        self.key1 = 591751049
        self.key2 = 878082192
        for p in pwd:
            self._UpdateKeys(p)

        assert isinstance(c, int)
        k = self.key2 | 2
        c = c ^ (((k * (k¹)) >> 8) & 255)
        self._UpdateKeys(c)
        return c
.
.
.

bytes = zef_file.read(12) 
```

1.  `_ZipDecryptor` 类维护了以三个旋转密钥形式出现的状态，该状态稍后将在 `_UpdateKeys()` 方法中更新（此处未展示）。
2.  该类定义了一个 `__call__()` 方法，使得该类可像函数一样调用。在此例中，`__call__()` 对 zip 文件的单个字节进行解密，然后基于经解密的字节对旋转密码进行更新。
3.  `zd` 是 `_ZipDecryptor` 类的一个实例。变量 `pwd` 被传入 `__init__()` 方法，并在其中被存储和用于首次旋转密码更新。
4.  给出 zip 文件的头 12 个字节，将这些字节映射给 `zd` 进行解密，实际上这将导致调用 `__call__()` 方法 12 次，也就是 更新内部状态并返回结果字节 12 次。

## 行为方式与序列类似的类

如果类作为一系列值的容器出现——也就是说如果对某个类来说，是否“包含”某值是件有意义的事情——那么它也许应该定义下面的特殊方法已，让它的行为方式与序列类似。

| 序号 | 目的 | 所编写代码 | Python 实际调用 |
| --- | --- | --- | --- |
|  | 序列的长度 | `len(seq)` | [`seq.__len__()`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__len__) |
|  | 了解某序列是否包含特定的值 | `x in seq` | [`seq.__contains__(`x`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__contains__) |

[`cgi` 模块](http://docs.python.org/3.1/library/cgi.html) 在其 `FieldStorage` 类中使用了这些方法，该类用于表示提交给动态网页的所有表单字段或查询参数。

```py
# A script which responds to http://example.com/search?q=cgi
import cgi
fs = cgi.FieldStorage()

  do_search()

# An excerpt from cgi.py that explains how that works
class FieldStorage:
.
.
.

        if self.list is None:
            raise TypeError('not indexable') 
```

1.  一旦创建了 `cgi.FieldStorage` 类的实例，就可以使用 “`in`” 运算符来检查查询字符串中是否包含了某个特定参数。
2.  而 `__contains__()` 方法是令该魔法生效的主角。
3.  如果代码为 `if 'q' in fs`，Python 将在 `fs` 对象中查找 `__contains__()` 方法，而该方法在 `cgi.py` 中已经定义。`'q'` 的值被当作 `key` 参数传入 `__contains__()` 方法。
4.  同样的 `FieldStorage` 类还支持返回其长度，因此可以编写代码 `len(`fs`)` 而其将调用 `FieldStorage` 的 `__len__()` 方法，并返回其识别的查询参数个数。
5.  `self.keys()` 方法检查 `self.list is None` 是否为真值，因此 `__len__` 方法无需重复该错误检查。

## 行为方式与字典类似的类

在前一节的基础上稍作拓展，就不仅可以对 “`in`” 运算符和 `len()` 函数进行响应，还可像全功能字典一样根据键来返回值。

| 序号 | 目的 | 所编写代码 | Python 实际调用 |
| --- | --- | --- | --- |
|  | 通过键来获取值 | `x[key]` | [`x.__getitem__(`key`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__getitem__) |
|  | 通过键来设置值 | `x[key] = value` | [`x.__setitem__(`key`,`value`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__setitem__) |
|  | 删除一个键值对 | `del x[key]` | [`x.__delitem__(`key`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__delitem__) |
|  | 为缺失键提供默认值 | `x[nonexistent_key]` | [`x.__missing__(`nonexistent_key`)`](http://docs.python.org/3.1/library/collections.html#collections.defaultdict.__missing__) |

[`cgi` 模块](http://docs.python.org/3.1/library/cgi.html) 的 `FieldStorage` 类 同样定义了这些特殊方法，也就是说可以像下面这样编码：

```py
# A script which responds to http://example.com/search?q=cgi
import cgi
fs = cgi.FieldStorage()
if 'q' in fs:

# An excerpt from cgi.py that shows how it works
class FieldStorage:
.
.
.

        if self.list is None:
            raise TypeError('not indexable')
        found = []
        for item in self.list:
            if item.name == key: found.append(item)
        if not found:
            raise KeyError(key)
        if len(found) == 1:
            return found[0]
        else:
            return found 
```

1.  `fs` 对象是 `cgi.FieldStorage` 类的一个实例，但仍然可以像 `fs['q']` 这样估算表达式。
2.  `fs['q']` 将 `key` 参数设置为 `'q'` 来调用 `__getitem__()` 方法。然后它将在其内部维护的查询参数列表 (`self.list`) 中查找一个 `.name` 与给定键相符的字典项。

## 行为方式与数值类似的类

使用适当的特殊方法，可以将类的行为方式定义为与数字相仿。也就是说，可以进行相加、相减，并进行其它数学运算。这就是 分数 的实现方式—— `Fraction` 类实现了这些特殊方法，然后就可以进行下列运算了：

```py
>>> from fractions import Fraction
>>> x = Fraction(1, 3)
>>> x / 3
Fraction(1, 9) 
```

以下是实现“类数字”类的完整特殊方法清单：

| 序号 | 目的 | 所编写代码 | Python 实际调用 |
| --- | --- | --- | --- |
|  | 加法 | `x + y` | [`x.__add__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__add__) |
|  | 减法 | `x - y` | [`x.__sub__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__sub__) |
|  | 乘法 | `x * y` | [`x.__mul__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__mul__) |
|  | 除法 | `x / y` | [`x.__truediv__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__truediv__) |
|  | 地板除 | `x // y` | [`x.__floordiv__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__floordiv__) |
|  | 取模（取余） | `x % y` | [`x.__mod__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__mod__) |
|  | 地板除 *&* 取模 | `divmod(x, y)` | [`x.__divmod__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__divmod__) |
|  | 乘幂 | `x ** y` | [`x.__pow__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__pow__) |
|  | 左位移 | `x &lt;&lt; y` | [`x.__lshift__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__lshift__) |
|  | 右位移 | `x &gt;&gt; y` | [`x.__rshift__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__rshift__) |
|  | 按位 `and` | `x & y` | [`x.__and__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__and__) |
|  | 按位 `xor` | `x ^ y` | [`x.__xor__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__xor__) |
|  | 按位 `or` | `x &#124; y` | [`x.__or__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__or__) |

如果 `x` 是某个实现了所有这些方法的类的实例，那么万事大吉。但如果未实现其中之一呢？或者更糟，如果实现了，但却无法处理某几类参数会怎么样？例如：

```py
>>> from fractions import Fraction
>>> x = Fraction(1, 3)
>>> 1 / x
Fraction(3, 1) 
```

这并 *不是* 传入一个 `分数` 并将其除以一个整数（如前例那样）的情况。前例中的情况非常直观： `x / 3` 调用 `x.__truediv__(3)`，而`Fraction` 的 `__truediv__()` 方法处理所有的数学运算。但整数并不“知道”如何对分数进行数学计算。因此本例该如何运作呢？

和 *反映操作* 相关的还有第二部分算数特殊方法。给定一个二元算术运算 （*例如：* `x / y`），有两种方法来实现它：

1.  告诉 `x` 将自己除以 `y`，或者
2.  告诉 `y` 去除 `x`

之前提到的特殊方法集合采用了第一种方式：对于给定 `x / y`，它们为 `x` 提供了一种途径来表述“我知道如何将自己除以 `y`。”下面的特殊方法集合采用了第二种方法：它们向 `y` 提供了一种途径来表述“我知道如何成为分母，并用自己去除 `x`。”

| 序号 | 目的 | 所编写代码 | Python 实际调用 |
| --- | --- | --- | --- |
|  | 加法 | `x + y` | [`y.__radd__(`x`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__radd__) |
|  | 减法 | `x - y` | [`y.__rsub__(`x`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__rsub__) |
|  | 乘法 | `x * y` | [`y.__rmul__(`x`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__rmul__) |
|  | 除法 | `x / y` | [`y.__rtruediv__(`x`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__rtruediv__) |
|  | 地板除 | `x // y` | [`y.__rfloordiv__(`x`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__rfloordiv__) |
|  | 取模（取余） | `x % y` | [`y.__rmod__(`x`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__rmod__) |
|  | 地板除 *&* 取模 | `divmod(x, y)` | [`y.__rdivmod__(`x`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__rdivmod__) |
|  | 乘幂 | `x ** y` | [`y.__rpow__(`x`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__rpow__) |
|  | 左位移 | `x &lt;&lt; y` | [`y.__rlshift__(`x`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__rlshift__) |
|  | 右位移 | `x &gt;&gt; y` | [`y.__rrshift__(`x`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__rrshift__) |
|  | 按位 `and` | `x & y` | [`y.__rand__(`x`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__rand__) |
|  | 按位 `xor` | `x ^ y` | [`y.__rxor__(`x`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__rxor__) |
|  | 按位 `or` | `x &#124; y` | [`y.__ror__(`x`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__ror__) |
|  |

但是等一下！还有更多特殊方法！如果在进行“原地”操作，如： `x /= 3`，还可定义更多的特殊方法。

| 序号 | 目的 | 所编写代码 | Python 实际调用 |
| --- | --- | --- | --- |
|  | 原地加法 | `x += y` | [`x.__iadd__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__iadd__) |
|  | 原地减法 | `x -= y` | [`x.__isub__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__isub__) |
|  | 原地乘法 | `x *= y` | [`x.__imul__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__imul__) |
|  | 原地除法 | `x /= y` | [`x.__itruediv__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__itruediv__) |
|  | 原地地板除法 | `x //= y` | [`x.__ifloordiv__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__ifloordiv__) |
|  | 原地取模 | `x %= y` | [`x.__imod__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__imod__) |
|  | 原地乘幂 | `x **= y` | [`x.__ipow__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__ipow__) |
|  | 原地左位移 | `x &lt;&lt;= y` | [`x.__ilshift__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__ilshift__) |
|  | 原地右位移 | `x &gt;&gt;= y` | [`x.__irshift__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__irshift__) |
|  | 原地按位 `and` | `x &= y` | [`x.__iand__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__iand__) |
|  | 原地按位 `xor` | `x ^= y` | [`x.__ixor__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__ixor__) |
|  | 原地按位 `or` | `x &#124;= y` | [`x.__ior__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__ior__) |

注意：多数情况下，并不需要原地操作方法。如果未对特定运算定义“就地”方法，Python 将会试着使用（普通）方法。例如，为执行表达式 `x /= y`，Python 将会：

1.  试着调用 `x.__itruediv__(`y`)`。如果该方法已经定义，并返回了 `NotImplemented` 之外的值，那已经大功告成了。
2.  试图调用 `x.__truediv__(`y`)`。如果该方法已定义并返回一个 `NotImplemented` 之外的值， `x` 的旧值将被丢弃，并将所返回的值替代它，就像是进行了 `x = x / y` 运算。
3.  试图调用 `y.__rtruediv__(`x`)`。如果该方法已定义并返回了一个 `NotImplemented` 之外的值，`x` 的旧值将被丢弃，并用所返回值进行替换。

因此如果想对原地运算进行优化，仅需像 `__itruediv__()` 方法一样定义“原地”方法。否则，基本上 Python 将会重新生成原地运算公式，以使用常规的运算及变量赋值。

还有一些“一元”数学运算，可以对“类-数字”对象自己执行。

| 序号 | 目的 | 所编写代码 | Python 实际调用 |
| --- | --- | --- | --- |
|  | 负数 | `-x` | [`x.__neg__()`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__neg__) |
|  | 正数 | `+x` | [`x.__pos__()`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__pos__) |
|  | 绝对值 | `abs(x)` | [`x.__abs__()`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__abs__) |
|  | 取反 | `~x` | [`x.__invert__()`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__invert__) |
|  | 复数 | `complex(x)` | [`x.__complex__()`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__complex__) |
|  | 整数转换 | `int(x)` | [`x.__int__()`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__int__) |
|  | 浮点数 | `float(x)` | [`x.__float__()`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__float__) |
|  | 四舍五入至最近的整数 | `round(x)` | [`x.__round__()`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__round__) |
|  | 四舍五入至最近的 `n` 位小数 | `round(x, n)` | [`x.__round__(n)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__round__) |
|  | `&gt;= x` 的最小整数 | `math.ceil(x)` | [`x.__ceil__()`](http://docs.python.org/3.1/library/math.html#math.ceil) |
|  | `&lt;= x`的最大整数 | `math.floor(x)` | [`x.__floor__()`](http://docs.python.org/3.1/library/math.html#math.floor) |
|  | 对 `x` 朝向 0 取整 | `math.trunc(x)` | [`x.__trunc__()`](http://docs.python.org/3.1/library/math.html#math.trunc) |
| [PEP 357](http://www.python.org/dev/peps/pep-0357/) | 作为列表索引的数字 | `a_list[x]` | [`a_list[x.__index__()]`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__index__) |

## 可比较的类

我将此内容从前一节中拿出来使其单独成节，是因为“比较”操作并不局限于数字。许多数据类型都可以进行比较——字符串、列表，甚至字典。如果要创建自己的类，且对象之间的比较有意义，可以使用下面的特殊方法来实现比较。

| 序号 | 目的 | 所编写代码 | Python 实际调用 |
| --- | --- | --- | --- |
|  | 相等 | `x == y` | [`x.__eq__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__eq__) |
|  | 不相等 | `x != y` | [`x.__ne__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__ne__) |
|  | 小于 | `x &lt; y` | [`x.__lt__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__lt__) |
|  | 小于或等于 | `x &lt;= y` | [`x.__le__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__le__) |
|  | 大于 | `x &gt; y` | [`x.__gt__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__gt__) |
|  | 大于或等于 | `x &gt;= y` | [`x.__ge__(`y`)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__ge__) |
|  | 布尔上上下文环境中的真值 | `if x:` | [`x.__bool__()`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__bool__) |

> ☞如果定义了 `__lt__()` 方法但没有定义 `__gt__()` 方法，Python 将通过经交换的算子调用 `__lt__()` 方法。然而，Python 并不会组合方法。例如，如果定义了 `__lt__()` 方法和 `__eq()__` 方法，并试图测试是否 `x &lt;= y`，Python 不会按顺序调用 `__lt__()` 和 `__eq()__` 。它将只调用 `__le__()` 方法。

## 可序列化的类

Python 支持 任意对象的序列化和反序列化。（多数 Python 参考资料称该过程为 “pickling” 和 “unpickling”）。该技术对与将状态保存为文件并在稍后恢复它非常有意义。所有的 内置数据类型 均已支持 pickling 。如果创建了自定义类，且希望它能够 pickle，阅读 [pickle 协议](http://docs.python.org/3.1/library/pickle.html) 了解下列特殊方法何时以及如何被调用。

| 序号 | 目的 | 所编写代码 | Python 实际调用 |
| --- | --- | --- | --- |
|  | 自定义对象的复制 | `copy.copy(x)` | [`x.__copy__()`](http://docs.python.org/3.1/library/copy.html) |
|  | 自定义对象的深度复制 | `copy.deepcopy(x)` | [`x.__deepcopy__()`](http://docs.python.org/3.1/library/copy.html) |
|  | 在 pickling 之前获取对象的状态 | `pickle.dump(x,`file`)` | [`x.__getstate__()`](http://docs.python.org/3.1/library/pickle.html#pickle-state) |
|  | 序列化某对象 | `pickle.dump(x,`file`)` | [`x.__reduce__()`](http://docs.python.org/3.1/library/pickle.html#pickling-class-instances) |
|  | 序列化某对象（新 pickling 协议） | `pickle.dump(x,`file`,`protocol_version`)` | [`x.__reduce_ex__(`protocol_version`)`](http://docs.python.org/3.1/library/pickle.html#pickling-class-instances) |
| * | 控制 unpickling 过程中对象的创建方式 | `x = pickle.load(`file`)` | [`x.__getnewargs__()`](http://docs.python.org/3.1/library/pickle.html#pickling-class-instances) |
| * | 在 unpickling 之后还原对象的状态 | `x = pickle.load(`file`)` | [`x.__setstate__()`](http://docs.python.org/3.1/library/pickle.html#pickle-state) |

*   要重建序列化对象，Python 需要创建一个和被序列化的对象看起来一样的新对象，然后设置新对象的所有属性。`__getnewargs__()` 方法控制新对象的创建过程，而 `__setstate__()` 方法控制属性值的还原方式。

## 可在 `with` 语块中使用的类

`with` 语块定义了 [运行时刻上下文环境](http://www.python.org/doc/3.1/library/stdtypes.html#typecontextmanager)；在执行 `with` 语句时将“进入”该上下文环境，而执行该语块中的最后一条语句将“退出”该上下文环境。

| 序号 | 目的 | 所编写代码 | Python 实际调用 |
| --- | --- | --- | --- |
|  | 在进入 `with` 语块时进行一些特别操作 | `with x:` | [`x.__enter__()`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__enter__) |
|  | 在退出 `with` 语块时进行一些特别操作 | `with x:` | [`x.__exit__()`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__exit__) |

以下是 `with`file`` 习惯用法 的运作方式：

```py
# excerpt from io.py:
def _checkClosed(self, msg=None):
    '''Internal: raise an ValueError if file is closed
    '''
    if self.closed:
        raise ValueError('I/O operation on closed file.'
                         if msg is None else msg)

def __enter__(self):
    '''Context management protocol.  Returns self.'''

def __exit__(self, *args):
    '''Context management protocol.  Calls close()''' 
```

1.  该文件对象同时定义了一个 `__enter__()` 和一个 `__exit__()` 方法。该 `__enter__()` 方法检查文件是否处于打开状态；如果没有， `_checkClosed()` 方法引发一个例外。
2.  `__enter__()` 方法将始终返回 `self` —— 这是 `with` 语块将用于调用属性和方法的对象
3.  在 `with` 语块结束后，文件对象将自动关闭。怎么做到的？在 `__exit__()` 方法中调用了 `self.close()` .

> ☞该 `__exit__()` 方法将总是被调用，哪怕是在 `with` 语块中引发了例外。实际上，如果引发了例外，该例外信息将会被传递给 `__exit__()` 方法。查阅 [With 状态上下文环境管理器](http://www.python.org/doc/3.1/reference/datamodel.html#with-statement-context-managers) 了解更多细节。

要了解关于上下文管理器的更多内容，请查阅 《自动关闭文件》 和 《重定向标准输出》。

## 真正神奇的东西

如果知道自己在干什么，你几乎可以完全控制类是如何比较的、属性如何定义，以及类的子类是何种类型。

| 序号 | 目的 | 所编写代码 | Python 实际调用 |
| --- | --- | --- | --- |
|  | 类构造器 | `x = MyClass()` | [`x.__new__()`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__new__) |
| * | 类析构器 | `del x` | [`x.__del__()`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__del__) |
|  | 只定义特定集合的某些属性 | [`x.__slots__()`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__slots__) |
|  | 自定义散列值 | `hash(x)` | [`x.__hash__()`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__hash__) |
|  | 获取某个属性的值 | `x.color` | [`type(x).__dict__['color'].__get__(x, type(x))`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__get__) |
|  | 设置某个属性的值 | `x.color = 'PapayaWhip'` | [`type(x).__dict__['color'].__set__(x, 'PapayaWhip')`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__set__) |
|  | 删除某个属性 | `del x.color` | [`type(x).__dict__['color'].__del__(x)`](http://www.python.org/doc/3.1/reference/datamodel.html#object.__delete__) |
|  | 控制某个对象是否是该对象的实例 your class | `isinstance(x, MyClass)` | [`MyClass.__instancecheck__(x)`](http://www.python.org/dev/peps/pep-3119/#overloading-isinstance-and-issubclass) |
|  | 控制某个类是否是该类的子类 | `issubclass(C, MyClass)` | [`MyClass.__subclasscheck__(C)`](http://www.python.org/dev/peps/pep-3119/#overloading-isinstance-and-issubclass) |
|  | 控制某个类是否是该抽象基类的子类 | `issubclass(C, MyABC)` | [`MyABC.__subclasshook__(C)`](http://docs.python.org/3.1/library/abc.html#abc.ABCMeta.__subclasshook__) |

* 确切掌握 Python 何时调用 `__del__()` 特别方法 [是件难以置信的复杂](http://www.python.org/doc/3.1/reference/datamodel.html#object.__del__)事情。要想完全理解它，必须清楚 [Python 如何在内存中跟踪对象](http://www.python.org/doc/3.1/reference/datamodel.html#objects-values-and-types)。以下有一篇好文章介绍 [Python 垃圾收集和类析构器](http://www.electricmonk.nl/log/2008/07/07/python-destructor-and-garbage-collection-notes/)。还可以阅读 [《弱引用》](http://mindtrove.info/articles/python-weak-references/)、[《`weakref` 模块》](http://docs.python.org/3.1/library/weakref.html)，还可以将 [《`gc` 模块》](http://www.python.org/doc/3.1/library/gc.html) 当作补充阅读材料。

## 深入阅读

本附录中提到的模块：

*   [`zipfile` 模块](http://docs.python.org/3.1/library/zipfile.html)
*   [`cgi` 模块](http://docs.python.org/3.1/library/cgi.html)
*   [`collections` 模块](http://www.python.org/doc/3.1/library/collections.html)
*   [`math［数学］` 模块](http://docs.python.org/3.1/library/math.html)
*   [`pickle` 模块](http://docs.python.org/3.1/library/pickle.html)
*   [`copy` 模块](http://docs.python.org/3.1/library/copy.html)
*   [`abc` (“抽象基类”) 模块](http://docs.python.org/3.1/library/abc.html)

其它启发式阅读：

*   [迷你语言格式规范](http://www.python.org/doc/3.1/library/string.html#formatspec)
*   [Python 数据模型](http://www.python.org/doc/3.1/reference/datamodel.html)
*   [内建类型](http://www.python.org/doc/3.1/library/stdtypes.html)
*   [PEP 357: 使任何对象可以使用切片](http://www.python.org/dev/peps/pep-0357/)
*   [PEP 3119: 抽象基类简介](http://www.python.org/dev/peps/pep-3119/)