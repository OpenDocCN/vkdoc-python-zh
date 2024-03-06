# 第十五章 重构

# 第十五章 重构

*   15.1\. 处理 bugs
*   15.2\. 应对需求变化
*   15.3\. 重构
*   15.4\. 后记
*   15.5\. 小结

# 15.1\. 处理 bugs

# 15.1\. 处理 bugs

尽管你很努力地编写全面的单元测试，但是 bug 还是会出现。我所说的 “bug” 是什么呢？Bug 是你还没有编写的测试用例。

## 例 15.1\. 关于 Bug

```py
>>> import roman5
>>> roman5.fromRoman("") 
0 
```

|  |  |
| --- | --- |
| [1] | 在前面的章节中你注意到一个空字符串会匹配上那个检查罗马数字有效性的正则表达式了吗？对于最终版本中的正则表达式这一点仍然没有改变。这就是一个 Bug ，你希望空字符串能够像其他无效的罗马数字表示一样引发 `InvalidRomanNumeralError` 异常。 |

在重现这个 Bug 并修改它之前你应该编写一个会失败的测试用例来说明它。

## 例 15.2\. 测试 bug (`romantest61.py`)

```py
 class FromRomanBadInput(unittest.TestCase):                                      
    # previous test cases omitted for clarity (they haven't changed)
    def testBlank(self):
        """fromRoman should fail with blank string"""
        self.assertRaises(roman.InvalidRomanNumeralError, roman.fromRoman, "") 
```

|  |  |
| --- | --- |
| [1] | 这里很简单。以空字符串调用 `fromRoman` 并确保它会引发一个 `InvalidRomanNumeralError` 异常。难点在于找出 Bug，既然你已经知道它了，测试就简单了。 |

因为你的代码存在一个 Bug，并且你编写了测试这个 Bug 的测试用例，所以测试用例将会失败：

## 例 15.3\. 用 `romantest61.py` 测试 `roman61.py` 的结果

```py
fromRoman should only accept uppercase input ... ok
toRoman should always return uppercase ... ok
fromRoman should fail with blank string ... FAIL
fromRoman should fail with malformed antecedents ... ok
fromRoman should fail with repeated pairs of numerals ... ok
fromRoman should fail with too many repeated numerals ... ok
fromRoman should give known result with known input ... ok
toRoman should give known result with known input ... ok
fromRoman(toRoman(n))==n for all n ... ok
toRoman should fail with non-integer input ... ok
toRoman should fail with negative input ... ok
toRoman should fail with large input ... ok
toRoman should fail with 0 input ... ok
======================================================================
FAIL: fromRoman should fail with blank string
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage6\romantest61.py", line 137, in testBlank
    self.assertRaises(roman61.InvalidRomanNumeralError, roman61.fromRoman, "")
  File "c:\python21\lib\unittest.py", line 266, in failUnlessRaises
    raise self.failureException, excName
AssertionError: InvalidRomanNumeralError ----------------------------------------------------------------------
Ran 13 tests in 2.864s
FAILED (failures=1) 
```

*现在* 你可以修改这个 Bug 了。

## 例 15.4\. 修改 Bug (`roman62.py`)

这个文件可以在例子目录下的 `py/roman/stage6/` 目录中找到。

```py
 def fromRoman(s):
    """convert Roman numeral to integer"""
    if not s: 
        raise InvalidRomanNumeralError, 'Input can not be blank'
    if not re.search(romanNumeralPattern, s):
        raise InvalidRomanNumeralError, 'Invalid Roman numeral: %s' % s
    result = 0
    index = 0
    for numeral, integer in romanNumeralMap:
        while s[index:index+len(numeral)] == numeral:
            result += integer
            index += len(numeral)
    return result 
```

|  |  |
| --- | --- |
| [1] | 只需要两行代码：一行直接检查空字符串和一行 `raise` 语句。 |

## 例 15.5\. 用 `romantest62.py` 测试 `roman62.py` 的结果

```py
fromRoman should only accept uppercase input ... ok
toRoman should always return uppercase ... ok
fromRoman should fail with blank string ... ok  fromRoman should fail with malformed antecedents ... ok
fromRoman should fail with repeated pairs of numerals ... ok
fromRoman should fail with too many repeated numerals ... ok
fromRoman should give known result with known input ... ok
toRoman should give known result with known input ... ok
fromRoman(toRoman(n))==n for all n ... ok
toRoman should fail with non-integer input ... ok
toRoman should fail with negative input ... ok
toRoman should fail with large input ... ok
toRoman should fail with 0 input ... ok
----------------------------------------------------------------------
Ran 13 tests in 2.834s
OK 
```

|  |  |
| --- | --- |
| [1] | 空字符串测试用例现在通过了，说明 Bug 被修正了。 |
| [2] | 所有其他测试用例依然通过，证明这个 Bug 修正没有影响到其他部分。不需要再编程了。 |

这样编程，并没有令 Bug 修正变得简单。简单的 Bug (就像这一个) 需要简单的测试用例，复杂 Bug 则需要复杂的测试用例。以测试为核心的氛围*好像* 延长了修正 Bug 的时间，因为你需要先贴切地描述出 Bug (编写测试用例) 然后才去修正它。如果测试用例没能正确通过，你需要思量这个修改错了还是测试用例本身出现了 Bug。无论如何，从长远上讲，这样在测试代码和代码之间的反复是值得的，因为这样会使 Bug 在第一时间就被修正的可能性大大提高。而且不论如何更改，你都可以轻易地重新运行*所有* 测试用例，新代码破坏老代码的机会也变得微乎其微。今天的单元测试就是明天的回归测试 (regression test)。

# 15.2\. 应对需求变化

# 15.2\. 应对需求变化

尽管你竭尽努力地分析你的客户，并点灯熬油地提炼出精确的需求，但需求还是会是不断变化。大部分客户在看到产品前不知道他们想要什么。即便知道，也不擅于精确表述出他们的有效需求。即便能表述出来，他们在下一个版本一定会要求更多的功能。因此你需要做好更新测试用例的准备以应对需求的改变。

假设你想要扩展罗马数字转换函数的范围。还记得没有哪个字符可以重复三遍以上这条规则吗？呃，现在罗马人希望给这条规则来个例外，用连续出现 4 个 `M` 字符来表示 `4000`。如果这样改了，你就可以把转换范围从 `1..3999` 扩展到 `1..4999`。但你先要对测试用例进行修改。

## 例 15.6\. 修改测试用例以适应新需求 (`romantest71.py`)

这个文件可以在例子目录下的 `py/roman/stage7/` 目录中找到。

如果您还没有下载本书附带的样例程序, 可以 [下载本程序和其他样例程序](http://www.woodpecker.org.cn/diveintopython/download/diveintopython-exampleszh-cn-5.4b.zip "Download example scripts")。

```py
 import roman71
import unittest
class KnownValues(unittest.TestCase):
    knownValues = ( (1, 'I'),
                    (2, 'II'),
                    (3, 'III'),
                    (4, 'IV'),
                    (5, 'V'),
                    (6, 'VI'),
                    (7, 'VII'),
                    (8, 'VIII'),
                    (9, 'IX'),
                    (10, 'X'),
                    (50, 'L'),
                    (100, 'C'),
                    (500, 'D'),
                    (1000, 'M'),
                    (31, 'XXXI'),
                    (148, 'CXLVIII'),
                    (294, 'CCXCIV'),
                    (312, 'CCCXII'),
                    (421, 'CDXXI'),
                    (528, 'DXXVIII'),
                    (621, 'DCXXI'),
                    (782, 'DCCLXXXII'),
                    (870, 'DCCCLXX'),
                    (941, 'CMXLI'),
                    (1043, 'MXLIII'),
                    (1110, 'MCX'),
                    (1226, 'MCCXXVI'),
                    (1301, 'MCCCI'),
                    (1485, 'MCDLXXXV'),
                    (1509, 'MDIX'),
                    (1607, 'MDCVII'),
                    (1754, 'MDCCLIV'),
                    (1832, 'MDCCCXXXII'),
                    (1993, 'MCMXCIII'),
                    (2074, 'MMLXXIV'),
                    (2152, 'MMCLII'),
                    (2212, 'MMCCXII'),
                    (2343, 'MMCCCXLIII'),
                    (2499, 'MMCDXCIX'),
                    (2574, 'MMDLXXIV'),
                    (2646, 'MMDCXLVI'),
                    (2723, 'MMDCCXXIII'),
                    (2892, 'MMDCCCXCII'),
                    (2975, 'MMCMLXXV'),
                    (3051, 'MMMLI'),
                    (3185, 'MMMCLXXXV'),
                    (3250, 'MMMCCL'),
                    (3313, 'MMMCCCXIII'),
                    (3408, 'MMMCDVIII'),
                    (3501, 'MMMDI'),
                    (3610, 'MMMDCX'),
                    (3743, 'MMMDCCXLIII'),
                    (3844, 'MMMDCCCXLIV'),
                    (3888, 'MMMDCCCLXXXVIII'),
                    (3940, 'MMMCMXL'),
                    (3999, 'MMMCMXCIX'),
                    (4000, 'MMMM'),                                       
                    (4500, 'MMMMD'),
                    (4888, 'MMMMDCCCLXXXVIII'),
                    (4999, 'MMMMCMXCIX'))
    def testToRomanKnownValues(self):
        """toRoman should give known result with known input"""
        for integer, numeral in self.knownValues:
            result = roman71.toRoman(integer)
            self.assertEqual(numeral, result)
    def testFromRomanKnownValues(self):
        """fromRoman should give known result with known input"""
        for integer, numeral in self.knownValues:
            result = roman71.fromRoman(numeral)
            self.assertEqual(integer, result)
class ToRomanBadInput(unittest.TestCase):
    def testTooLarge(self):
        """toRoman should fail with large input"""
        self.assertRaises(roman71.OutOfRangeError, roman71.toRoman, 5000) 
    def testZero(self):
        """toRoman should fail with 0 input"""
        self.assertRaises(roman71.OutOfRangeError, roman71.toRoman, 0)
    def testNegative(self):
        """toRoman should fail with negative input"""
        self.assertRaises(roman71.OutOfRangeError, roman71.toRoman, -1)
    def testNonInteger(self):
        """toRoman should fail with non-integer input"""
        self.assertRaises(roman71.NotIntegerError, roman71.toRoman, 0.5)
class FromRomanBadInput(unittest.TestCase):
    def testTooManyRepeatedNumerals(self):
        """fromRoman should fail with too many repeated numerals"""
        for s in ('MMMMM', 'DD', 'CCCC', 'LL', 'XXXX', 'VV', 'IIII'):     
            self.assertRaises(roman71.InvalidRomanNumeralError, roman71.fromRoman, s)
    def testRepeatedPairs(self):
        """fromRoman should fail with repeated pairs of numerals"""
        for s in ('CMCM', 'CDCD', 'XCXC', 'XLXL', 'IXIX', 'IVIV'):
            self.assertRaises(roman71.InvalidRomanNumeralError, roman71.fromRoman, s)
    def testMalformedAntecedent(self):
        """fromRoman should fail with malformed antecedents"""
        for s in ('IIMXCC', 'VX', 'DCM', 'CMM', 'IXIV',
                  'MCMC', 'XCX', 'IVI', 'LM', 'LD', 'LC'):
            self.assertRaises(roman71.InvalidRomanNumeralError, roman71.fromRoman, s)
    def testBlank(self):
        """fromRoman should fail with blank string"""
        self.assertRaises(roman71.InvalidRomanNumeralError, roman71.fromRoman, "")
class SanityCheck(unittest.TestCase):
    def testSanity(self):
        """fromRoman(toRoman(n))==n for all n"""
        for integer in range(1, 5000):                                    
            numeral = roman71.toRoman(integer)
            result = roman71.fromRoman(numeral)
            self.assertEqual(integer, result)
class CaseCheck(unittest.TestCase):
    def testToRomanCase(self):
        """toRoman should always return uppercase"""
        for integer in range(1, 5000):
            numeral = roman71.toRoman(integer)
            self.assertEqual(numeral, numeral.upper())
    def testFromRomanCase(self):
        """fromRoman should only accept uppercase input"""
        for integer in range(1, 5000):
            numeral = roman71.toRoman(integer)
            roman71.fromRoman(numeral.upper())
            self.assertRaises(roman71.InvalidRomanNumeralError,
                              roman71.fromRoman, numeral.lower())
if __name__ == "__main__":
    unittest.main() 
```

|  |  |
| --- | --- |
| [1] | 原来的已知值没有改变 (它们仍然是合理的测试值) 但你需要添加几个大于 `4000` 的值。这里我添加了 `4000` (最短的一个)，`4500` (次短的一个)，`4888` (最长的一个) 和 `4999` (值最大的一个)。 |
| [2] | “最大输入”的定义改变了。以前是以 `4000` 调用 `toRoman` 并期待一个错误；而现在 `4000-4999` 成为了有效输入，需要将这个最大输入提升至 `5000`。 |
| [3] | “过多字符重复” 的定义也改变了。这个测试以前是以 `'MMMM'` 调用 `fromRoman` 并期待一个错误；而现在 `MMMM` 被认为是一个有效的罗马数字表示，需要将这个“过多字符重复”改为 `'MMMMM'`。 |
| [4] | 完备测试和大小写测试原来在 `1` 到 `3999` 范围内循环。现在范围扩展了，这个 `for` 循环需要将范围也提升至 `4999`。 |

现在你的测试用例和新需求保持一致了，但是你的程序代码还没有，因此几个测试用例的失败是意料之中的事。

## 例 15.7\. 用 `romantest71.py` 测试 `roman71.py` 的结果

```py
 fromRoman should only accept uppercase input ... ERROR  toRoman should always return uppercase ... ERROR
fromRoman should fail with blank string ... ok
fromRoman should fail with malformed antecedents ... ok
fromRoman should fail with repeated pairs of numerals ... ok
fromRoman should fail with too many repeated numerals ... ok
fromRoman should give known result with known input ... ERROR  toRoman should give known result with known input ... ERROR  fromRoman(toRoman(n))==n for all n ... ERROR  toRoman should fail with non-integer input ... ok
toRoman should fail with negative input ... ok
toRoman should fail with large input ... ok
toRoman should fail with 0 input ... ok 
```

|  |  |
| --- | --- |
| [1] | 我们的大小写检查是因为循环范围是 `1` 到 `4999`，而 `toRoman` 只接受 `1` 到 `3999` 之间的数，因此测试循环到 `4000` 就会失败。 |
| [2] | `fromRoman` 的已知值测试在遇到 `'MMMM'` 就会失败，因为 `fromRoman` 还认为这是一个无效的罗马数字表示。 |
| [3] | `toRoman` 的已知值测试在遇到 `4000` 就会失败，因为 `toRoman` 仍旧认为这超出了有效值范围。 |
| [4] | 完备测试在遇到 `4000` 也会失败，因为 `toRoman` 也会认为这超出了有效值范围。 |

```py
 ======================================================================
ERROR: fromRoman should only accept uppercase input
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage7\romantest71.py", line 161, in testFromRomanCase
    numeral = roman71.toRoman(integer)
  File "roman71.py", line 28, in toRoman
    raise OutOfRangeError, "number out of range (must be 1..3999)"
OutOfRangeError: number out of range (must be 1..3999) ======================================================================
ERROR: toRoman should always return uppercase
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage7\romantest71.py", line 155, in testToRomanCase
    numeral = roman71.toRoman(integer)
  File "roman71.py", line 28, in toRoman
    raise OutOfRangeError, "number out of range (must be 1..3999)"
OutOfRangeError: number out of range (must be 1..3999) ======================================================================
ERROR: fromRoman should give known result with known input
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage7\romantest71.py", line 102, in testFromRomanKnownValues
    result = roman71.fromRoman(numeral)
  File "roman71.py", line 47, in fromRoman
    raise InvalidRomanNumeralError, 'Invalid Roman numeral: %s' % s
InvalidRomanNumeralError: Invalid Roman numeral: MMMM ======================================================================
ERROR: toRoman should give known result with known input
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage7\romantest71.py", line 96, in testToRomanKnownValues
    result = roman71.toRoman(integer)
  File "roman71.py", line 28, in toRoman
    raise OutOfRangeError, "number out of range (must be 1..3999)"
OutOfRangeError: number out of range (must be 1..3999) ======================================================================
ERROR: fromRoman(toRoman(n))==n for all n
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage7\romantest71.py", line 147, in testSanity
    numeral = roman71.toRoman(integer)
  File "roman71.py", line 28, in toRoman
    raise OutOfRangeError, "number out of range (must be 1..3999)"
OutOfRangeError: number out of range (must be 1..3999) ----------------------------------------------------------------------
Ran 13 tests in 2.213s
FAILED (errors=5) 
```

既然新的需求导致了测试用例的失败，你该考虑修改代码以便它能再次通过测试用例。(在你开始编写单元测试时要习惯一件事：被测试代码永远不会在编写测试用例“之前”编写。正因为如此，你还有一些工作要做，一旦可以通过所有的测试用例，停止编码。)

## 例 15.8\. 为新的需求编写代码 (`roman72.py`)

这个文件可以在例子目录下的 `py/roman/stage7/` 目录中找到。

```py
"""Convert to and from Roman numerals"""
import re
#Define exceptions
class RomanError(Exception): pass
class OutOfRangeError(RomanError): pass
class NotIntegerError(RomanError): pass
class InvalidRomanNumeralError(RomanError): pass
#Define digit mapping
romanNumeralMap = (('M',  1000),
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
def toRoman(n):
    """convert integer to Roman numeral"""
    if not (0 < n < 5000):                                                         
        raise OutOfRangeError, "number out of range (must be 1..4999)"
    if int(n) <> n:
        raise NotIntegerError, "non-integers can not be converted"
    result = ""
    for numeral, integer in romanNumeralMap:
        while n >= integer:
            result += numeral
            n -= integer
    return result
#Define pattern to detect valid Roman numerals
romanNumeralPattern = '^M?M?M?M?(CM|CD|D?C?C?C?)(XC|XL|L?X?X?X?)(IX|IV|V?I?I?I?)$' 
def fromRoman(s):
    """convert Roman numeral to integer"""
    if not s:
        raise InvalidRomanNumeralError, 'Input can not be blank'
    if not re.search(romanNumeralPattern, s):
        raise InvalidRomanNumeralError, 'Invalid Roman numeral: %s' % s
    result = 0
    index = 0
    for numeral, integer in romanNumeralMap:
        while s[index:index+len(numeral)] == numeral:
            result += integer
            index += len(numeral)
    return result 
```

|  |  |
| --- | --- |
| [1] | `toRoman` 只需要在取值范围检查一处做个小改动。将原来的 `0 &lt; n &lt; 4000`，更改为现在的检查 `0 &lt; n &lt; 5000`。你还要更改你 `raise` 的错误信息以反映接受新取值范围 (`1..4999` 而不再是 `1..3999`)。你不需要改变函数的其他部分，它们已经适用于新的情况。(它们会欣然地为新的 1000 添加 `'M'`，以 `4000` 为例，函数会返回 `'MMMM'` 。之前没能这样做是因为到范围检查时就被停了下来。) |
| [2] | 你对 `fromRoman` 也不需要做过多的修改。唯一的修改就在 `romanNumeralPattern`：如果你注意的话，你会发现你只需在正则表达式的第一部分增加一个可选的 `M` 。这就允许最多 4 个 `M` 字符而不再是 3 个，意味着你允许代表 `4999` 而不只是 `3999` 的罗马数字。`fromRoman` 函数本身是普遍适用的，它并不在意字符被多少次的重复，只是根据重复的罗马字符对应的数值进行累加。以前没能处理 `'MMMM'` 是因为你通过正则表达式的检查强行停止了。 |

你可能会怀疑只需这两处小改动。嘿，不相信我的话，你自己看看吧：

## 例 15.9\. 用 `romantest72.py` 测试 `roman72.py` 的结果

```py
fromRoman should only accept uppercase input ... ok
toRoman should always return uppercase ... ok
fromRoman should fail with blank string ... ok
fromRoman should fail with malformed antecedents ... ok
fromRoman should fail with repeated pairs of numerals ... ok
fromRoman should fail with too many repeated numerals ... ok
fromRoman should give known result with known input ... ok
toRoman should give known result with known input ... ok
fromRoman(toRoman(n))==n for all n ... ok
toRoman should fail with non-integer input ... ok
toRoman should fail with negative input ... ok
toRoman should fail with large input ... ok
toRoman should fail with 0 input ... ok
----------------------------------------------------------------------
Ran 13 tests in 3.685s
OK 
```

|  |  |
| --- | --- |
| [1] | 所有的测试用例都通过了，停止编写代码。 |

全面的单元测试意味着不必依赖于程序员的一面之词：“相信我！”

# 15.3\. 重构

# 15.3\. 重构

全面的单元测试带来的最大好处不是你的全部测试用例最终通过时的成就感；也不是被责怪破坏了别人的代码时能够*证明* 自己的自信。最大的好处是单元测试给了你自由去无情地重构。

重构是在可运行代码的基础上使之工作得更好的过程。通常，“更好”意味着“更快”，也可能意味着 “使用更少的内存”，或者 “使用更少的磁盘空间”，或者仅仅是“更优雅的代码”。不管对你，对你的项目意味什么，在你的环境中，重构对任何程序的长期良性运转都是重要的。

这里，“更好” 意味着 “更快”。更具体地说，`fromRoman` 函数可以更快，关键在于那个丑陋的、用于验证罗马数字有效性的正则表达式。尝试不用正则表达式去解决是不值得的 (这样做很难，而且可能也快不了多少)，但可以通过预编译正则表达式使函数提速。

## 例 15.10\. 编译正则表达式

```py
>>> import re
>>> pattern = '^M?M?M?$'
>>> re.search(pattern, 'M')               
<SRE_Match object at 01090490>
>>> compiledPattern = re.compile(pattern) 
>>> compiledPattern
<SRE_Pattern object at 00F06E28>
>>> dir(compiledPattern)                  
['findall', 'match', 'scanner', 'search', 'split', 'sub', 'subn']
>>> compiledPattern.search('M')           
<SRE_Match object at 01104928> 
```

|  |  |
| --- | --- |
| [1] | 这是你看到过的 `re.search` 语法。把一个正则表达式作为字符串 (`pattern`) 并用这个字符串来匹配 (`'M'`)。如果能够匹配，函数返回 一个 match 对象，可以用来确定匹配的部分和如何匹配的。 |
| [2] | 这里是一个新的语法：`re.compile` 把一个正则表达式作为字符串参数接受并返回一个 pattern 对象。注意这里没去匹配字符串。编译正则表达式和以特定字符串 (`'M'`) 进行匹配不是一回事，所牵扯的只是正则表达式本身。 |
| [3] | `re.compile` 返回的已编译的 pattern 对象有几个值得关注的功能：包括了几个 `re` 模块直接提供的功能 (比如：`search` 和 `sub`)。 |
| [4] | 用 `'M'` 作参数来调用已编译的 pattern 对象的 `search` 函数与用正则表达式和字符串 `'M'` 调用 `re.search` 可以得到相同的结果，只是快了很多。 (事实上，`re.search` 函数仅仅将正则表达式编译，然后为你调用编译后的 pattern 对象的 `search` 方法。) |

> 注意
> 在需要多次使用同一个正则表达式的情况下，应该将它进行编译以获得一个 pattern 对象，然后直接调用这个 pattern 对象的方法。

## 例 15.11\. `roman81.py` 中已编译的正则表达式

这个文件可以在例子目录下的 `py/roman/stage8/` 目录中找到。

如果您还没有下载本书附带的样例程序, 可以 [下载本程序和其他样例程序](http://www.woodpecker.org.cn/diveintopython/download/diveintopython-exampleszh-cn-5.4b.zip "Download example scripts")。

```py
# toRoman and rest of module omitted for clarity
romanNumeralPattern = \
    re.compile('^M?M?M?M?(CM|CD|D?C?C?C?)(XC|XL|L?X?X?X?)(IX|IV|V?I?I?I?)$') 
def fromRoman(s):
    """convert Roman numeral to integer"""
    if not s:
        raise InvalidRomanNumeralError, 'Input can not be blank'
    if not romanNumeralPattern.search(s):                                    
        raise InvalidRomanNumeralError, 'Invalid Roman numeral: %s' % s
    result = 0
    index = 0
    for numeral, integer in romanNumeralMap:
        while s[index:index+len(numeral)] == numeral:
            result += integer
            index += len(numeral)
    return result 
```

|  |  |
| --- | --- |
| [1] | 看起来很相似，但实质却有很大改变。`romanNumeralPattern` 不再是一个字符串了，而是一个由 `re.compile` 返回的 pattern 对象。 |
| [2] | 这意味着你可以直接调用 `romanNumeralPattern` 的方法。这比每次调用 `re.search` 要快很多。模块被首次导入 (import) 之时，正则表达式被一次编译并存储于 `romanNumeralPattern`。之后每次调用 `fromRoman` 时，你可以立刻以正则表达式匹配输入的字符串，而不需要在重复背后的这些编译的工作。 |

那么编译正则表达式可以提速多少呢？你自己来看吧：

## 例 15.12\. 用 `romantest81.py` 测试 `roman81.py` 的结果

```py
.............  ----------------------------------------------------------------------
Ran 13 tests in 3.385s  OK 
```

|  |  |
| --- | --- |
| [1] | 有一点说明一下：这里，我在运行单元测试时*没有* 使用 `-v` 选项，因此输出的也不再是每个测试完整的 `doc string`，而是用一个圆点来表示每个通过的测试。(失败的测试标用 `F` 表示，发生错误则用 `E` 表示，你仍旧可以获得失败和错误的完整追踪信息以便查找问题所在。) |
| [2] | 运行 `13` 个测试耗时 `3.385` 秒，与之相比是没有预编译正则表达式时的 `3.685 秒`。这是一个 `8%` 的整体提速，记住单元测试的大量时间实际上花在做其他工作上。(我单独测试了正则表达式部分的耗时，不考虑单元测试的其他环节，正则表达式编译可以让匹配 `search` 平均提速 `54%`。)小小修改还真是值得。 |
| [3] | 对了，不必顾虑什么，预先编译正则表达式并没有破坏什么，你刚刚证实这一点。 |

我还想做另外一个性能优化工作。就正则表达式语法的复杂性而言，通常有不止一种方法来构造相同的表达式是不会令人惊讶的。在 [comp.lang.python](http://groups.google.com/groups?group=comp.lang.python) 上对该模块进行一些讨论后，有人建议我使用 `{_m_,_n_}` 语法来查找可选重复字符。

## 例 15.13\. `roman82.py`

这个文件可以在例子目录下的 `py/roman/stage8/` 目录中找到。

如果您还没有下载本书附带的样例程序, 可以 [下载本程序和其他样例程序](http://www.woodpecker.org.cn/diveintopython/download/diveintopython-exampleszh-cn-5.4b.zip "Download example scripts")。

```py
# rest of program omitted for clarity
#old version
#romanNumeralPattern = \
#   re.compile('^M?M?M?M?(CM|CD|D?C?C?C?)(XC|XL|L?X?X?X?)(IX|IV|V?I?I?I?)$')
#new version
romanNumeralPattern = \
    re.compile('^M{0,4}(CM|CD|D?C{0,3})(XC|XL|L?X{0,3})(IX|IV|V?I{0,3})$') 
```

|  |  |
| --- | --- |
| [1] | 你已经将 `M?M?M?M?` 替换为 `M{0,4}`。它们的含义相同：“匹配 0 到 4 个 `M` 字符”。类似地，`C?C?C?` 改成了 `C{0,3}` (“匹配 0 到 3 个 `C` 字符”) 接下来的 `X` 和 `I` 也一样。 |

这样的正则表达简短一些 (虽然可读性不太好)。核心问题是，是否能加快速度？

## 例 15.14\. 以 `romantest82.py` 测试 `roman82.py` 的结果

```py
.............
----------------------------------------------------------------------
Ran 13 tests in 3.315s  OK 
```

|  |  |
| --- | --- |
| [1] | 总体而言，这种正则表达使单元测试提速 2%。这不太令人振奋，但记住 `search` 函数只是整体单元测试的一个小部分，很多时间花在了其他方面。(我另外的测试表明这个应用了新语法的正则表达式使 `search` 函数提速 `11%` 。) 通过预先编译和使用新语法重写可以使正则表达式的性能提升超过 `60%`，令单元测试的整体性能提升超过 `10%`。 |
| [2] | 比任何的性能提升更重要的是模块仍然运转完好。这便是我早先提到的自由：自由地调整、修改或者重写任何部分并且保证在此过程中没有把事情搞得一团糟。这并不是给无休止地为了调整代码而调整代码以许可；你有很切实的目标 (“让 `fromRoman` 更快”)，而且你可以实现这个目标，不会因为考虑在改动过程中是否会引入新的 Bug 而有所迟疑。 |

还有另外一个我想做的调整，我保证这是最后一个，之后我会停下来，让这个模块歇歇。就像你多次看到的，正则表达式越晦涩难懂越快，我可不想在六个月内再回头试图维护它。是呀！测试用例通过了，我便知道它工作正常，但如果我搞不懂它是*如何* 工作的，添加新功能、修正新 Bug，或者维护它都将变得很困难。正如你在 第 7.5 节 “松散正则表达式” 看到的，Python 提供了逐行注释你的逻辑的方法。

## 例 15.15\. `roman83.py`

该文件可以在例子目录下的 `py/roman/stage8/` 目录中找到。

如果您还没有下载本书附带的样例程序, 可以 [下载本程序和其他样例程序](http://www.woodpecker.org.cn/diveintopython/download/diveintopython-exampleszh-cn-5.4b.zip "Download example scripts")。

```py
# rest of program omitted for clarity
#old version
#romanNumeralPattern = \
#   re.compile('^M{0,4}(CM|CD|D?C{0,3})(XC|XL|L?X{0,3})(IX|IV|V?I{0,3})$')
#new version
romanNumeralPattern = re.compile('''
    ^                   # beginning of string
    M{0,4}              # thousands - 0 to 4 M's
    (CM|CD|D?C{0,3})    # hundreds - 900 (CM), 400 (CD), 0-300 (0 to 3 C's),
                        #            or 500-800 (D, followed by 0 to 3 C's)
    (XC|XL|L?X{0,3})    # tens - 90 (XC), 40 (XL), 0-30 (0 to 3 X's),
                        #        or 50-80 (L, followed by 0 to 3 X's)
    (IX|IV|V?I{0,3})    # ones - 9 (IX), 4 (IV), 0-3 (0 to 3 I's),
                        #        or 5-8 (V, followed by 0 to 3 I's)
    $                   # end of string
    ''', re.VERBOSE) 
```

|  |  |
| --- | --- |
| [1] | `re.compile` 函数的第二个参数是可选的，这个参数通过一个或一组标志 (flag) 来控制预编译正则表达式的选项。这里你指定了 `re.VERBOSE` 选项，告诉 Python 正则表达式里有内联注释。注释和它们周围的空白*不* 会被认做正则表达式的一部分，在编译正则表达式时 `re.compile` 函数会忽略它们。这个新 “verbose” 版本与老版本完全一样，只是更具可读性。 |

## 例 15.16\. 用 `romantest83.py` 测试 `roman83.py` 的结果

```py
.............
----------------------------------------------------------------------
Ran 13 tests in 3.315s  OK 
```

|  |  |
| --- | --- |
| [1] | 新 “verbose” 版本和老版本的运行速度一样。事实上，编译的 pattern 对象也一样，因为 `re.compile` 函数会剔除掉所有你添加的内容。 |
| [2] | 新 “verbose” 版本可以通过所有老版本通过的测试。什么都没有改变，但在六个月后重读该模块的程序员却有了理解功能如何实现的机会。 |

# 15.4\. 后记

# 15.4\. 后记

聪明的读者在学习前一节时想得会更深入一层。现在写的这个程序中最令人头痛的性能负担是正则表达式，但它是必需的，因为没有其它方法来识别罗马数字。但是，它们只有 5000 个，为什么不一次性地构建一个查询表来读取？不必用正则表达式凸现了这个主意的好处。你建立了整数到罗马数字查询表的时候，罗马数字到整数的逆向查询表也构建了。

更大的好处在于，你已经拥有一整套完全的单元测试。你修改了多半的代码，但单元测试还是一样的，因此你可以确定你的新代码与来的代码一样可以正常工作。

## 例 15.17\. `roman9.py`

这个文件可以在例子目录下的 `py/roman/stage9/` 目录中找到。

如果您还没有下载本书附带的样例程序, 可以 [下载本程序和其他样例程序](http://www.woodpecker.org.cn/diveintopython/download/diveintopython-exampleszh-cn-5.4b.zip "Download example scripts")。

```py
#Define exceptions
class RomanError(Exception): pass
class OutOfRangeError(RomanError): pass
class NotIntegerError(RomanError): pass
class InvalidRomanNumeralError(RomanError): pass
#Roman numerals must be less than 5000
MAX_ROMAN_NUMERAL = 4999
#Define digit mapping
romanNumeralMap = (('M',  1000),
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
#Create tables for fast conversion of roman numerals.
#See fillLookupTables() below.
toRomanTable = [ None ]  # Skip an index since Roman numerals have no zero
fromRomanTable = {}
def toRoman(n):
    """convert integer to Roman numeral"""
    if not (0 < n <= MAX_ROMAN_NUMERAL):
        raise OutOfRangeError, "number out of range (must be 1..%s)" % MAX_ROMAN_NUMERAL
    if int(n) <> n:
        raise NotIntegerError, "non-integers can not be converted"
    return toRomanTable[n]
def fromRoman(s):
    """convert Roman numeral to integer"""
    if not s:
        raise InvalidRomanNumeralError, "Input can not be blank"
    if not fromRomanTable.has_key(s):
        raise InvalidRomanNumeralError, "Invalid Roman numeral: %s" % s
    return fromRomanTable[s]
def toRomanDynamic(n):
    """convert integer to Roman numeral using dynamic programming"""
    result = ""
    for numeral, integer in romanNumeralMap:
        if n >= integer:
            result = numeral
            n -= integer
            break
    if n > 0:
        result += toRomanTable[n]
    return result
def fillLookupTables():
    """compute all the possible roman numerals"""
    #Save the values in two global tables to convert to and from integers.
    for integer in range(1, MAX_ROMAN_NUMERAL + 1):
        romanNumber = toRomanDynamic(integer)
        toRomanTable.append(romanNumber)
        fromRomanTable[romanNumber] = integer
fillLookupTables() 
```

这样有多快呢？

## 例 15.18\. 用 `romantest9.py` 测试 `roman9.py` 的结果

```py
 .............
----------------------------------------------------------------------
Ran 13 tests in 0.791s
OK 
```

还记得吗？你原有版本的最快速度是 13 个测试耗时 3.315 秒。当然，这样的比较不完全公平，因为这个新版本需要更长的时间来导入 (当它填充查询表时)。但是导入只需一次，在运行过程中可以忽略。

这个重构的故事的寓意是什么？

*   简洁是美德。
*   特别是使用正则表达式时。
*   并且单元测试给了你大规模重构的信心……即使原有的代码不是你写的。

# 15.5\. 小结

# 15.5\. 小结

单元测试是一个强大的概念，使用得当的话既可以减少维护成本又可以增加长期项目的灵活性。同样重要的是要意识到单元测试并不是“灵丹妙药”，也不是“银弹”。编写好的测试用例很困难，保持其更新更需要磨练 (特别是当顾客对修复严重的 Bug 大呼小叫之时)。单元测试不是其它形式测试的替代品，比如说功能性测试、集成测试以及可用性测试。但它切实可行且功效明显，一旦相识，你会反问为什么以往没有应用它。

这一章涵盖了很多内容，有很多都不是 Python 所特有的。很多语言都有单元测试框架，都要求你理解相同的基本概念：

*   测试用例的设计方针是目的单一、可以自动运行、互不干扰。
*   在被测试代码编写*之前* 编写测试用例。
*   编写测试有效输入的测试用例")并检查正确的结果。
*   编写测试无效输入的测试用例")并检查正确的失败。
*   为描述 Bug 或反映新需求而编写和升级测试用例。
*   为改进性能、可伸缩性、可读性、可维护性和任何缺少的特性而无情地重构。

另外，你应该能够自如地做到如下 Python 的特有工作：

*   继承 `unittest.TestCase` 生成子类并为每个单独的测试用例编写方法。
*   使用 `assertEqual` 检查已知结果的返回。
*   使用 `assertRaises` 检查函数是否引发已知异常。
*   在 `if __name__` 子句中调用 `unittest.main()` 来一次性运行所有测试用例。
*   以详细 (verbose) 或者普通 (regular) 模式运行单元测试

## 进一步阅读

*   XProgramming.com 有多种语言的 [单元测试框架](http://www.xprogramming.com/software.htm) 的下载链接。