# Chapter 10 重构

> " After one has played a vast quantity of notes and more notes, it is simplicity that emerges as the crowning reward of art. " — [Frédéric Chopin](http://en.wikiquote.org/wiki/Fr%C3%A9d%C3%A9ric_Chopin)

## 深入

就算是竭尽了全力编写全面的单元测试，还是会遇到错误。我所说的“错误”是什么意思？错误是尚未写到的测试实例。

```py
>>> import roman7

0 
```

1.  这就是错误。和其它无效罗马数字的一系列字符一样，空字符串将引发 `InvalidRomanNumeralError` 例外。

在重现该错误后，应该在修复前写出一个导致该失败情形的测试实例，这样才能描述该错误。

```py
class FromRomanBadInput(unittest.TestCase):  
    .
    .
    .
    def testBlank(self):
        '''from_roman should fail with blank string''' 
```

1.  这段代码非常简单。通过传入一个空字符串调用 `from_roman()` ，并确保其引发一个 `InvalidRomanNumeralError` 例外。难的是发现错误；找到了该错误之后对它进行测试是件轻松的工作。

由于代码有错误，且有用于测试该错误的测试实例，该测试实例将会导致失败：

```py
you@localhost:~/diveintopython3/examples$ python3 romantest8.py -v
from_roman should fail with blank string ... FAIL
from_roman should fail with malformed antecedents ... ok
from_roman should fail with repeated pairs of numerals ... ok
from_roman should fail with too many repeated numerals ... ok
from_roman should give known result with known input ... ok
to_roman should give known result with known input ... ok
from_roman(to_roman(n))==n for all n ... ok
to_roman should fail with negative input ... ok
to_roman should fail with non-integer input ... ok
to_roman should fail with large input ... ok
to_roman should fail with 0 input ... ok

======================================================================
FAIL: from_roman should fail with blank string
----------------------------------------------------------------------
Traceback (most recent call last):
  File "romantest8.py", line 117, in test_blank
    self.assertRaises(roman8.InvalidRomanNumeralError, roman8.from_roman, '')
<mark>AssertionError: InvalidRomanNumeralError not raised by from_roman</mark>

----------------------------------------------------------------------
Ran 11 tests in 0.171s

FAILED (failures=1) 
```

*现在* 可以修复该错误了。

```py
def from_roman(s):
    '''convert Roman numeral to integer'''

        raise InvalidRomanNumeralError('Input can not be blank')
    if not re.search(romanNumeralPattern, s):

    result = 0
    index = 0
    for numeral, integer in romanNumeralMap:
        while s[index:index+len(numeral)] == numeral:
            result += integer
            index += len(numeral)
    return result 
```

1.  只需两行代码：一行明确地对空字符串进行检查，另一行为 `raise` 语句。
2.  在本书中还尚未提到该内容，因此现在让我们讲讲 字符串格式化 最后一点内容。从 Python 3.1 起，在格式化标示符中使用位置索引时可以忽略数字。也就是说，无需使用格式化标示符 `{0}` 来指向 `format()` 方法的第一个参数，只需简单地使用 `{}` 而 Python 将会填入正确的位置索引。该规则适用于任何数量的参数；第一个 `{}` 代表 `{0}`，第二个 `{}` 代表 `{1}`，以此类推。

```py
you@localhost:~/diveintopython3/examples$ python3 romantest8.py -v

from_roman should fail with malformed antecedents ... ok
from_roman should fail with repeated pairs of numerals ... ok
from_roman should fail with too many repeated numerals ... ok
from_roman should give known result with known input ... ok
to_roman should give known result with known input ... ok
from_roman(to_roman(n))==n for all n ... ok
to_roman should fail with negative input ... ok
to_roman should fail with non-integer input ... ok
to_roman should fail with large input ... ok
to_roman should fail with 0 input ... ok

----------------------------------------------------------------------
Ran 11 tests in 0.156s 
```

1.  现在空字符串测试实例通过了测试，也就是说错误被修正了。
2.  所有其它测试实例仍然可以通过，说明该错误修正没有破坏其它部分。代码编写结束。

用此方式编写代码将使得错误修正变得更困难。简单的错误（像这个）需要简单的测试实例；复杂的错误将会需要复杂的测试实例。在以测试为中心的环境中，由于必须在代码中精确地描述错误（编写测试实例），然后修正错误本身，看起来 *好像* 修正错误需要更多的时间。而如果测试实例无法正确地通过，则又需要找出到底是修正方案有错误，还数测试实例本身就有错误。然而从长远看，这种在测试代码和经测试代码之间的来回折腾是值得的，因为这样才更有可能在第一时间修正错误。同时，由于可以对新代码轻松地重新运行 *所有* 测试实例，在修正新代码时破坏旧代码的机会更低。今天的单元测试就是明天的回归测试。

## 控制需求变化

为了获取准确的需求，尽管已经竭力将客户“钉”在原地，并经历了反复剪切、粘贴的痛苦，但需求仍然会变化。大多数客户在看到产品之前不知道自己想要什么，而且就算知道，他们也不擅长清晰地表述自己的想法。而即便擅长表述，他们在下一个版本中也会提出更多要求。因此，必须随时准备好更新测试实例以应对需求变化。

举个例子来说，假定我们要扩展罗马数字转换函数的能力范围。正常情况下，罗马数字中的任何一个字符在同一行中不得重复出现三次以上。但罗马人却愿意该规则有个例外：通过一行中的 4 个 `M` 字符来代表 `4000` 。进行该修改后，将会把可转换数字的范围从 `1..3999` 拓展为 `1..4999`。但首先必须对测试实例进行一些修改。

```py
class KnownValues(unittest.TestCase):
    known_values = ( (1, 'I'),
                      .
                      .
                      .
                     (3999, 'MMMCMXCIX'),

                     (4500, 'MMMMD'),
                     (4888, 'MMMMDCCCLXXXVIII'),
                     (4999, 'MMMMCMXCIX') )

class ToRomanBadInput(unittest.TestCase):
    def test_too_large(self):
        '''to_roman should fail with large input'''

    .
    .
    .

class FromRomanBadInput(unittest.TestCase):
    def test_too_many_repeated_numerals(self):
        '''from_roman should fail with too many repeated numerals'''

            self.assertRaises(roman8.InvalidRomanNumeralError, roman8.from_roman, s)

    .
    .
    .

class RoundtripCheck(unittest.TestCase):
    def test_roundtrip(self):
        '''from_roman(to_roman(n))==n for all n'''

            numeral = roman8.to_roman(integer)
            result = roman8.from_roman(numeral)
            self.assertEqual(integer, result) 
```

1.  现有的已知数值不会变（它们依然是合理的测试数值），但必须在 `4000` 范围之内（外）增加一些。在此，我已经添加了 `4000` (最短)、 `4500` (第二短)、 `4888` (最长) 和 `4999` (最大)。
2.  “过大值输入” 的定义已经发生了变化。该测试用于通过传入 `4000` 调用 `to_roman()` 并期望引发一个错误；目前 `4000-4999` 是有效的值，必须将该值调整为 `5000` 。
3.  “太多重复数字”的定义也发生了变化。该测试通过传入 `'MMMM'` 调用 `from_roman()` 并预期发生一个错误；目前 `MMMM` 被认定为有效的罗马数字，必须将该条件修改为 `'MMMMM'` 。
4.  对范围内的每个数字进行完整循环测试，从 `1` 到 `3999`。由于范围已经进行了拓展，该 `for` 循环同样需要修改为以 `4999` 为上限。

现在，测试实例已经按照新的需求进行了更新，但代码还没有，因按照预期，某些测试实例将返回失败结果。

```py
you@localhost:~/diveintopython3/examples$ python3 romantest9.py -v
from_roman should fail with blank string ... ok
from_roman should fail with malformed antecedents ... ok
from_roman should fail with non-string input ... ok
from_roman should fail with repeated pairs of numerals ... ok
from_roman should fail with too many repeated numerals ... ok

to_roman should fail with negative input ... ok
to_roman should fail with non-integer input ... ok
to_roman should fail with large input ... ok
to_roman should fail with 0 input ... ok

======================================================================
ERROR: from_roman should give known result with known input
----------------------------------------------------------------------
Traceback (most recent call last):
  File "romantest9.py", line 82, in test_from_roman_known_values
    result = roman9.from_roman(numeral)
  File "C:\home\diveintopython3\examples\roman9.py", line 60, in from_roman
    raise InvalidRomanNumeralError('Invalid Roman numeral: {0}'.format(s))
<mark>roman9.InvalidRomanNumeralError: Invalid Roman numeral: MMMM</mark>

======================================================================
ERROR: to_roman should give known result with known input
----------------------------------------------------------------------
Traceback (most recent call last):
  File "romantest9.py", line 76, in test_to_roman_known_values
    result = roman9.to_roman(integer)
  File "C:\home\diveintopython3\examples\roman9.py", line 42, in to_roman
    raise OutOfRangeError('number out of range (must be 0..3999)')
<mark>roman9.OutOfRangeError: number out of range (must be 0..3999)</mark>

======================================================================
ERROR: from_roman(to_roman(n))==n for all n
----------------------------------------------------------------------
Traceback (most recent call last):
  File "romantest9.py", line 131, in testSanity
    numeral = roman9.to_roman(integer)
  File "C:\home\diveintopython3\examples\roman9.py", line 42, in to_roman
    raise OutOfRangeError('number out of range (must be 0..3999)')
<mark>roman9.OutOfRangeError: number out of range (must be 0..3999)</mark>

----------------------------------------------------------------------
Ran 12 tests in 0.171s

FAILED (errors=3) 
```

1.  一旦遇到 `'MMMM'`，`from_roman()` 已知值测试将会失败，因为 `from_roman()` 仍将其视为无效罗马数字。
2.  一旦遇到 `4000`，`to_roman()` 已知值测试将会失败，因为 `to_roman()` 仍将其视为超范围数字。
3.  而往返（译注：指在普通数字和罗马数字之间来回转换）检查遇到 `4000` 时也会失败，因为 `to_roman()` 仍认为其超范围。

现在，我们有了一些由新需求导致失败的测试实例，可以考虑修正代码让它与新测试实例一致起来。（刚开始编写单元测试的时候，被测试代码绝不会在测试实例“之前”出现确实让人感觉有点怪。）尽管编码工作被置后安排，但还是不少要做的事情，一旦与测试实例相符，编码工作就可以结束了。一旦习惯单元测试后，您可能会对自己曾在编程时不进行测试感到很奇怪。）

```py
roman_numeral_pattern = re.compile('''
    ^                   # beginning of string

    (CM|CD|D?C{0,3})    # hundreds - 900 (CM), 400 (CD), 0-300 (0 to 3 Cs),
                        #            or 500-800 (D, followed by 0 to 3 Cs)
    (XC|XL|L?X{0,3})    # tens - 90 (XC), 40 (XL), 0-30 (0 to 3 Xs),
                        #        or 50-80 (L, followed by 0 to 3 Xs)
    (IX|IV|V?I{0,3})    # ones - 9 (IX), 4 (IV), 0-3 (0 to 3 Is),
                        #        or 5-8 (V, followed by 0 to 3 Is)
    $                   # end of string
    ''', re.VERBOSE)

def to_roman(n):
    '''convert integer to Roman numeral'''

        raise OutOfRangeError('number out of range (must be 1..4999)')
    if not isinstance(n, int):
        raise NotIntegerError('non-integers can not be converted')

    result = ''
    for numeral, integer in roman_numeral_map:
        while n >= integer:
            result += numeral
            n -= integer
    return result

def from_roman(s):
    .
    .
    . 
```

1.  根本无需对 `from_roman()` 函数进行任何修改。唯一需要修改的是 `roman_numeral_pattern` 。仔细观察下，将会发现我已经在正则表达式的第一部分中将 `M` 字符的数量从 `3` 优化为 `4` 。该修改将允许等价于 `4999` 而不是 `3999` 的罗马数字。实际的 `from_roman()` 函数完全是通用的；它只查找重复的罗马数字字符并将它们加起来，而不关心它们重复了多少次。之前无法处理 `'MMMM'` 的唯一原因是我们通过正则表达式匹配明确地阻止了它这么做。
2.  `to_roman()` 函数只需在范围检查中进行一个小改动。将之前检查 `0 &lt; n &lt; 4000` 的地方现在修改为检查 `0 &lt; n &lt; 5000` 。同时修改 `引发` 的错误信息，以体现新的可接受范围 (`1..4999` 取代 `1..3999`) 。无需对函数剩下部分进行任何修改；它已经能够应对新的实例。（它将对找到的每个千位增加 `'M'` ；如果给定 `4000`，它将给出 `'MMMM'`。之前它不这么做的唯一原因是我们通过范围检查明确地阻止了它。）

所需做的就是这两处小修改，但你可能会有点怀疑。嗨，别光听我说，你自己看看吧。

```py
you@localhost:~/diveintopython3/examples$ python3 romantest9.py -v
from_roman should fail with blank string ... ok
from_roman should fail with malformed antecedents ... ok
from_roman should fail with non-string input ... ok
from_roman should fail with repeated pairs of numerals ... ok
from_roman should fail with too many repeated numerals ... ok
from_roman should give known result with known input ... ok
to_roman should give known result with known input ... ok
from_roman(to_roman(n))==n for all n ... ok
to_roman should fail with negative input ... ok
to_roman should fail with non-integer input ... ok
to_roman should fail with large input ... ok
to_roman should fail with 0 input ... ok

----------------------------------------------------------------------
Ran 12 tests in 0.203s 
```

1.  所有测试实例均通过了。代码编写结束。

全面单元测试的意思是：无需依赖某个程序员来说“相信我吧。”

## 重构

关于全面单元测试，最美妙的事情不是在所有的测试实例通过后的那份心情，也不是别人抱怨你破坏了代码，而你通过实践 *证明* 自己没有时的快感。单元测试最美妙之处在于它给了你大刀阔斧进行重构的自由。

重构是修改可运作代码，使其表现更佳的过程。通常，“更佳”指的是“更快”，但它也可能指的是“占用更少内存“、”占用更少磁盘空间“或者”更加简洁”。对于你的环境、你的项目来说，无论重构意味着什么，它对程序的长期健康都至关重要。

本例中，“更佳”的意思既包括“更快”也包括“更易于维护”。具体而言，因为用于验证罗马数字的正则表达式生涩冗长，该 `from_roman()` 函数比我所希望的更慢，也更加复杂。现在，你可能会想，“当然，正则表达式就又臭又长的，难道我有其它办法验证任意字符串是否为罗马数字吗？”

答案是：只针对 5000 个数进行转换；为什么不知建立一个查询表呢？意识到 *根本不需要使用正则表达式* 之后，这个主意甚至变得更加理想了。在建立将整数转换为罗马数字的查询表的同时，还可以建立将罗马数字转换为整数的逆向查询表。在需要检查任意字符串是否是有效罗马数字的时候，你将收集到所有有效的罗马数字。“验证”工作简化为一个简单的字典查询。

最棒的是，你已经有了一整套单元测试。可以修改模块中一半以上的代码，而单元测试将会保持不变。这意味着可以向你和其他人证明：新代码运作和最初的一样好。

```py
class OutOfRangeError(ValueError): pass
class NotIntegerError(ValueError): pass
class InvalidRomanNumeralError(ValueError): pass

roman_numeral_map = (('M',  1000),
                     ('CM', 900),
                     ('D',  500),
                     ('CD', 400),
                     ('C',  100),
                     ('XC', 90),
                     ('L',  50),
                     ('XL', 40),
                     ('X',  10),
                     ('IX', 9),
                     ('V',  5),
                     ('IV', 4),
                     ('I',  1))

to_roman_table = [ None ]
from_roman_table = {}

def to_roman(n):
    '''convert integer to Roman numeral'''
    if not (0 < n < 5000):
        raise OutOfRangeError('number out of range (must be 1..4999)')
    if int(n) != n:
        raise NotIntegerError('non-integers can not be converted')
    return to_roman_table[n]

def from_roman(s):
    '''convert Roman numeral to integer'''
    if not isinstance(s, str):
        raise InvalidRomanNumeralError('Input must be a string')
    if not s:
        raise InvalidRomanNumeralError('Input can not be blank')
    if s not in from_roman_table:
        raise InvalidRomanNumeralError('Invalid Roman numeral: {0}'.format(s))
    return from_roman_table[s]

def build_lookup_tables():
    def to_roman(n):
        result = ''
        for numeral, integer in roman_numeral_map:
            if n >= integer:
                result = numeral
                n -= integer
                break
        if n > 0:
            result += to_roman_table[n]
        return result

    for integer in range(1, 5000):
        roman_numeral = to_roman(integer)
        to_roman_table.append(roman_numeral)
        from_roman_table[roman_numeral] = integer

build_lookup_tables() 
```

让我们打断一下，进行一些剖析工作。可以说，最重要的是最后一行：

```py
build_lookup_tables() 
```

可以注意到这是一次函数调用，但没有 `if` 语句包裹住它。这不是 `if __name__ == '__main__'` 语块；*模块被导入时* 它将会被调用。（重要的是必须明白：模块将只被导入一次，随后被缓存了。如果导入一个已导入模块，将不会导致任何事情发生。因此这段代码将只在第一此导入时运行。）

那么，该 `build_lookup_tables()` 函数究竟进行了哪些操作呢?很高兴你问这个问题。

```py
to_roman_table = [ None ]
from_roman_table = {}
.
.
.
def build_lookup_tables():

        result = ''
        for numeral, integer in roman_numeral_map:
            if n >= integer:
                result = numeral
                n -= integer
                break
        if n > 0:
            result += to_roman_table[n]
        return result

    for integer in range(1, 5000):

        from_roman_table[roman_numeral] = integer 
```

1.  这是一段聪明的程序代码……也许过于聪明了。上面定义了 `to_roman()` 函数；它在查询表中查找值并返回结果。而 `build_lookup_tables()` 函数重定义了 `to_roman()` 函数用于实际操作（像添加查询表之前的例子一样）。在 `build_lookup_tables()` 函数内部，对 `to_roman()` 的调用将会针对该重定义的版本。一旦 `build_lookup_tables()` 函数退出，重定义的版本将会消失 — 它的定义只在 `build_lookup_tables()` 函数的作用域内生效。
2.  该行代码将调用重定义的 `to_roman()` 函数，该函数实际计算罗马数字。
3.  一旦获得结果（从重定义的 `to_roman()` 函数），可将整数及其对应的罗马数字添加到两个查询表中。

查询表建好后，剩下的代码既容易又快捷。

```py
def to_roman(n):
    '''convert integer to Roman numeral'''
    if not (0 < n < 5000):
        raise OutOfRangeError('number out of range (must be 1..4999)')
    if int(n) != n:
        raise NotIntegerError('non-integers can not be converted')

def from_roman(s):
    '''convert Roman numeral to integer'''
    if not isinstance(s, str):
        raise InvalidRomanNumeralError('Input must be a string')
    if not s:
        raise InvalidRomanNumeralError('Input can not be blank')
    if s not in from_roman_table:
        raise InvalidRomanNumeralError('Invalid Roman numeral: {0}'.format(s)) 
```

1.  像前面那样进行同样的边界检查之后，`to_roman()` 函数只需在查询表中查找并返回适当的值。
2.  同样，`from_roman()` 函数也缩水为一些边界检查和一行代码。不再有正则表达式。不再有循环。O(1) 转换为或转换到罗马数字。

但这段代码可以运作吗？为什么可以，是的它可以。而且我可以证明。

```py
you@localhost:~/diveintopython3/examples$ python3 romantest10.py -v
from_roman should fail with blank string ... ok
from_roman should fail with malformed antecedents ... ok
from_roman should fail with non-string input ... ok
from_roman should fail with repeated pairs of numerals ... ok
from_roman should fail with too many repeated numerals ... ok
from_roman should give known result with known input ... ok
to_roman should give known result with known input ... ok
from_roman(to_roman(n))==n for all n ... ok
to_roman should fail with negative input ... ok
to_roman should fail with non-integer input ... ok
to_roman should fail with large input ... ok
to_roman should fail with 0 input ... ok

----------------------------------------------------------------------

OK 
```

1.  它不仅能够回答你的问题，还运行得非常快！好象速度提升了 10 倍。当然，这种比较并不公平，因为此版本在导入时耗时更长（在建造查询表时）。但由于只进行一次导入，启动的成本可以由对 `to_roman()` 和 `from_roman()` 函数的所有调用摊薄。由于该测试进行几千次函数调用（来回单独测试上万次），节省出来的效率成本得以迅速提升！

这个故事的寓意是什么？

*   简单是一种美德。
*   特别在涉及到正则表达式的时候。
*   单元测试令你在进行大规模重构时充满自信。

## 摘要

单元测试是一个威力强大的概念，如果正确实施，不但可以降低维护成本，还可以提高长期项目的灵活性。但同时还必须明白：单元测试既不是灵丹妙药，也不是解决问题的魔术，更不是银弹。编写良好的测试实例非常艰难，确保它们时刻保持最新必须成为一项纪律（特别在客户要求关键错误修正时）。单元测试不是功能测试、集成测试或用户承受能力测试等其它测试的替代品。但它是可行的、行之有效的，见识过其功用后，你将对之前曾没有用它而感到奇怪。

这几章覆盖的内容很多，很大一部分都不是 Python 所特有的。许多语言都有单元测试框架，但所有框架都要求掌握同一基本概念：

*   设计测试实例是件具体、自动且独立的工作。
*   在编写被测试代码 *之前* 编写测试实例。
*   编写用于检查好输入并验证正确结果的测试
*   编写用于测试“坏”输入并做出正确失败响应的测试。
*   编写并更新测试实例以反映新的需求
*   毫不留情地重构以提升性能、可扩展性、可读性、可维护性及任何缺乏的特性。