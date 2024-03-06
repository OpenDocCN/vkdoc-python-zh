# 第十四章 测试优先编程

# 第十四章 测试优先编程

*   14.1\. roman.py, 第 1 阶段
*   14.2\. roman.py, 第 2 阶段
*   14.3\. roman.py, 第 3 阶段
*   14.4\. roman.py, 第 4 阶段
*   14.5\. roman.py, 第 5 阶段

# 14.1\. `roman.py`, 第 1 阶段

# 14.1\. `roman.py`, 第 1 阶段

到目前为止，单元测试已经完成，是时候开始编写被单元测试测试的代码了。你将分阶段地完成这个工作，因此开始时所有的单元测试都是失败的，但在逐步完成 `roman.py` 的同时你会看到它们一个个地通过测试。

## 例 14.1\. `roman1.py`

这个程序可以在例子目录下的 `py/roman/stage1/` 目录中找到。

如果您还没有下载本书附带的样例程序, 可以 [下载本程序和其他样例程序](http://www.woodpecker.org.cn/diveintopython/download/diveintopython-exampleszh-cn-5.4b.zip "Download example scripts")。

```py
"""Convert to and from Roman numerals"""
#Define exceptions
class RomanError(Exception): pass                 class OutOfRangeError(RomanError): pass           class NotIntegerError(RomanError): pass
class InvalidRomanNumeralError(RomanError): pass 
def toRoman(n):
    """convert integer to Roman numeral"""
    pass                                         
def fromRoman(s):
    """convert Roman numeral to integer"""
    pass 
```

|  |  |
| --- | --- |
| [1] | 这就是如何定义你自己的 Python 异常。异常 (Exception) 也是类，通过继承已有的异常，你可以创建自定义的异常。强烈建议 (但不是必须) 你继承 `Exception` 来定义自己的异常，因为它是所有内建异常的基类。这里我定义了 `RomanError` (从 `Exception` 继承而来) 作为我所有自定义异常的基类。这是一个风格问题，我也可以直接从 `Exception` 继承建立每一个自定义异常。 |
| [2] | `OutOfRangeError` 和 `NotIntegerError` 异常将会最终被用于 `toRoman` 以标示不同类型的无效输入，更具体而言就是 `ToRomanBadInput` 测试的那些。 |
| [3] | `InvalidRomanNumeralError` 将被最终用于 `fromRoman` 以标示无效输入，具体而言就是 `FromRomanBadInput`测试的那些。 |
| [4] | 在这一步中你只是想定义每个函数的 API ，而不想具体实现它们，因此你以 Python 关键字 `pass` 姑且带过。 |

重要的时刻到了 (请打起鼓来)：你终于要对这个简陋的小模块开始运行单元测试了。目前而言，每一个测试用例都应该失败。事实上，任何测试用例在此时通过，你都应该回头看看 `romantest.py` ，仔细想想为什么你写的测试代码如此没用，以至于连什么都不作的函数都能通过测试。

用命令行选项 `-v` 运行 `romantest1.py` 可以得到更详细的输出信息，这样你就可以看到每一个测试用例的具体运行情况。如果幸运，你的结果应该是这样的：

## 例 14.2\. 以 `romantest1.py` 测试 `roman1.py` 的输出

```py
fromRoman should only accept uppercase input ... ERROR
toRoman should always return uppercase ... ERROR
fromRoman should fail with malformed antecedents ... FAIL
fromRoman should fail with repeated pairs of numerals ... FAIL
fromRoman should fail with too many repeated numerals ... FAIL
fromRoman should give known result with known input ... FAIL
toRoman should give known result with known input ... FAIL
fromRoman(toRoman(n))==n for all n ... FAIL
toRoman should fail with non-integer input ... FAIL
toRoman should fail with negative input ... FAIL
toRoman should fail with large input ... FAIL
toRoman should fail with 0 input ... FAIL
======================================================================
ERROR: fromRoman should only accept uppercase input
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage1\romantest1.py", line 154, in testFromRomanCase
    roman1.fromRoman(numeral.upper())
AttributeError: 'None' object has no attribute 'upper' ======================================================================
ERROR: toRoman should always return uppercase
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage1\romantest1.py", line 148, in testToRomanCase
    self.assertEqual(numeral, numeral.upper())
AttributeError: 'None' object has no attribute 'upper' ======================================================================
FAIL: fromRoman should fail with malformed antecedents
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage1\romantest1.py", line 133, in testMalformedAntecedent
    self.assertRaises(roman1.InvalidRomanNumeralError, roman1.fromRoman, s)
  File "c:\python21\lib\unittest.py", line 266, in failUnlessRaises
    raise self.failureException, excName
AssertionError: InvalidRomanNumeralError ======================================================================
FAIL: fromRoman should fail with repeated pairs of numerals
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage1\romantest1.py", line 127, in testRepeatedPairs
    self.assertRaises(roman1.InvalidRomanNumeralError, roman1.fromRoman, s)
  File "c:\python21\lib\unittest.py", line 266, in failUnlessRaises
    raise self.failureException, excName
AssertionError: InvalidRomanNumeralError ======================================================================
FAIL: fromRoman should fail with too many repeated numerals
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage1\romantest1.py", line 122, in testTooManyRepeatedNumerals
    self.assertRaises(roman1.InvalidRomanNumeralError, roman1.fromRoman, s)
  File "c:\python21\lib\unittest.py", line 266, in failUnlessRaises
    raise self.failureException, excName
AssertionError: InvalidRomanNumeralError ======================================================================
FAIL: fromRoman should give known result with known input
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage1\romantest1.py", line 99, in testFromRomanKnownValues
    self.assertEqual(integer, result)
  File "c:\python21\lib\unittest.py", line 273, in failUnlessEqual
    raise self.failureException, (msg or '%s != %s' % (first, second))
AssertionError: 1 != None ======================================================================
FAIL: toRoman should give known result with known input
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage1\romantest1.py", line 93, in testToRomanKnownValues
    self.assertEqual(numeral, result)
  File "c:\python21\lib\unittest.py", line 273, in failUnlessEqual
    raise self.failureException, (msg or '%s != %s' % (first, second))
AssertionError: I != None ======================================================================
FAIL: fromRoman(toRoman(n))==n for all n
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage1\romantest1.py", line 141, in testSanity
    self.assertEqual(integer, result)
  File "c:\python21\lib\unittest.py", line 273, in failUnlessEqual
    raise self.failureException, (msg or '%s != %s' % (first, second))
AssertionError: 1 != None ======================================================================
FAIL: toRoman should fail with non-integer input
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage1\romantest1.py", line 116, in testNonInteger
    self.assertRaises(roman1.NotIntegerError, roman1.toRoman, 0.5)
  File "c:\python21\lib\unittest.py", line 266, in failUnlessRaises
    raise self.failureException, excName
AssertionError: NotIntegerError ======================================================================
FAIL: toRoman should fail with negative input
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage1\romantest1.py", line 112, in testNegative
    self.assertRaises(roman1.OutOfRangeError, roman1.toRoman, -1)
  File "c:\python21\lib\unittest.py", line 266, in failUnlessRaises
    raise self.failureException, excName
AssertionError: OutOfRangeError ======================================================================
FAIL: toRoman should fail with large input
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage1\romantest1.py", line 104, in testTooLarge
    self.assertRaises(roman1.OutOfRangeError, roman1.toRoman, 4000)
  File "c:\python21\lib\unittest.py", line 266, in failUnlessRaises
    raise self.failureException, excName
AssertionError: OutOfRangeError ======================================================================
FAIL: toRoman should fail with 0 input  ---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage1\romantest1.py", line 108, in testZero
    self.assertRaises(roman1.OutOfRangeError, roman1.toRoman, 0)
  File "c:\python21\lib\unittest.py", line 266, in failUnlessRaises
    raise self.failureException, excName
AssertionError: OutOfRangeError  ----------------------------------------------------------------------
Ran 12 tests in 0.040s  FAILED (failures=10, errors=2) 
```

|  |  |
| --- | --- |
| [1] | 运行脚本将会执行 `unittest.main()`，由它来执行每个测试用例，也就是每个在 `romantest.py` 中定义的方法。对于每个测试用例，无论测试通过与否，都会输出这个方法的 `doc string`。意料之中，没有通过一个测试用例。 |
| [2] | 对于每个失败的测试用例，`unittest` 显示的跟踪信息告诉我们都发生了什么。就此处而言，调用 `assertRaises` (也称作 `failUnlessRaises`) 引发了一个 `AssertionError` 异常，因为期待 `toRoman` 所引发的 `OutOfRangeError` 异常没有出现。 |
| [3] | 在这些细节后面，`unittest` 给出了一个关于被执行测试的个数和花费时间的总结。 |
| [4] | 总而言之，由于至少一个测试用例没有通过，单元测试失败了。当某个测试用例没能通过时，`unittest` 会区分是失败 (failures) 还是错误 (errors)。失败是指调用 `assertXYZ` 方法，比如 `assertEqual` 或者 `assertRaises` 时，断言的情况没有发生或预期的异常没有被引发。而错误是指你测试的代码或单元测试本身发生了某种异常。例如：`testFromRomanCase` 方法 (“`fromRoman` 只接受大写输入”) 就是一个错误，因为调用 `numeral.upper()` 引发了一个 `AttributeError` 异常，因为 `toRoman` 的返回值不是期望的字符串类型。但是，`testZero` (“`toRoman` 应该在输入 0 时失败”) 是一个失败，因为调用 `fromRoman` 没有引发一个 `assertRaises` 期待的异常：`InvalidRomanNumeral`。 |

# 14.2\. `roman.py`, 第 2 阶段

# 14.2\. `roman.py`, 第 2 阶段

现在你有了 `roman` 模块的大概框架，到了开始写代码以通过测试的时候了。

## 例 14.3\. `roman2.py`

这个文件可以从 `py/roman/stage2/` 目录中找到。

如果您还没有下载本书附带的样例程序, 可以 [下载本程序和其他样例程序](http://www.woodpecker.org.cn/diveintopython/download/diveintopython-exampleszh-cn-5.4b.zip "Download example scripts")。

```py
"""Convert to and from Roman numerals"""
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
    result = ""
    for numeral, integer in romanNumeralMap:
        while n >= integer:      
            result += numeral
            n -= integer
    return result
def fromRoman(s):
    """convert Roman numeral to integer"""
    pass 
```

|  |  |
| --- | --- |
| [1] | `romanNumeralMap` 是一个用来定义三个内容的元组的元组：1\. 代表大部分罗马数字的字符。注意不只是单字符的罗马数字，你同样在这里定义诸如 `CM` (“比一千少一百，即 900”) 的双字符，这可以让稍后编写的 `toRoman` 简单一些。2\. 罗马数字的顺序。它们是以降序排列的，从`M` 一路到 `I`。3\. 每个罗马数字所对应的数值。每个内部的元组都是一个 `(_numeral_，_value_)` 数值对。 |
| [2] | 这里便显示出你丰富的数据结构带来的优势，你不需要什么特定的逻辑处理减法规则。你只需要通过搜寻 `romanNumeralMap` 寻找不大于输入数值的最大对应整数即可。只要找到，就在结果的结尾把这个整数对应的罗马字符添加到输出结果的末尾，从输入值中减去这个整数，一遍遍这样继续下去。 |

## 例 14.4\. `toRoman` 如何工作

如果你不明了 `toRoman` 如何工作，在 `while` 循环的结尾添加一个 `print` 语句：

```py
 while n >= integer:
            result += numeral
            n -= integer
            print 'subtracting', integer, 'from input, adding', numeral, 'to output' 
```

```py
>>> import roman2
>>> roman2.toRoman(1424)
subtracting 1000 from input, adding M to output
subtracting 400 from input, adding CD to output
subtracting 10 from input, adding X to output
subtracting 10 from input, adding X to output
subtracting 4 from input, adding IV to output
'MCDXXIV' 
```

看来 `toRoman` 可以运转了，至少手工测试可以。但能通过单元测试吗？啊哈，不，不完全可以。

## 例 14.5\. 以 `romantest2.py` 测试 `roman2.py` 的输出

要记得用 `-v` 命令行选项运行 `romantest2.py` 开启详细信息模式。

```py
fromRoman should only accept uppercase input ... FAIL
toRoman should always return uppercase ... ok  fromRoman should fail with malformed antecedents ... FAIL
fromRoman should fail with repeated pairs of numerals ... FAIL
fromRoman should fail with too many repeated numerals ... FAIL
fromRoman should give known result with known input ... FAIL
toRoman should give known result with known input ... ok  fromRoman(toRoman(n))==n for all n ... FAIL
toRoman should fail with non-integer input ... FAIL  toRoman should fail with negative input ... FAIL
toRoman should fail with large input ... FAIL
toRoman should fail with 0 input ... FAIL 
```

|  |  |
| --- | --- |
| [1] | 事实上，`toRoman` 的返回值总是大写的，因为 `romanNumeralMap` 定义的罗马字符都是以大写字母表示的。因此这个测试已经通过了。 |
| [2] | 好消息来了：这个版本的 `toRoman` 函数能够通过已知值测试。记住，这并不能证明完全没问题，但至少通过测试多种有效输入考验了这个函数：包括每个单一字符的罗马数字，可能的最大输入 (`3999`)，以及可能的最长的罗马数字 (对应于 `3888`)。从这点来看，你有理由相信这个函数对于任何有效输入都不会出问题。 |
| [3] | 但是，函数还没办法处理无效输入，每个无效输入测试都失败了。这很好理解，因为你还没有对无效输入进行检查，测试用例希望捕捉到特定的异常 (通过 `assertRaises`)，而你根本没有让这些异常引发。这是你下一阶段的工作。 |

下面是单元测试结果的剩余部分，列出了所有失败的详细信息，你已经让它降到了 10 个。

```py
 ======================================================================
FAIL: fromRoman should only accept uppercase input
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage2\romantest2.py", line 156, in testFromRomanCase
    roman2.fromRoman, numeral.lower())
  File "c:\python21\lib\unittest.py", line 266, in failUnlessRaises
    raise self.failureException, excName
AssertionError: InvalidRomanNumeralError ======================================================================
FAIL: fromRoman should fail with malformed antecedents
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage2\romantest2.py", line 133, in testMalformedAntecedent
    self.assertRaises(roman2.InvalidRomanNumeralError, roman2.fromRoman, s)
  File "c:\python21\lib\unittest.py", line 266, in failUnlessRaises
    raise self.failureException, excName
AssertionError: InvalidRomanNumeralError ======================================================================
FAIL: fromRoman should fail with repeated pairs of numerals
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage2\romantest2.py", line 127, in testRepeatedPairs
    self.assertRaises(roman2.InvalidRomanNumeralError, roman2.fromRoman, s)
  File "c:\python21\lib\unittest.py", line 266, in failUnlessRaises
    raise self.failureException, excName
AssertionError: InvalidRomanNumeralError ======================================================================
FAIL: fromRoman should fail with too many repeated numerals
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage2\romantest2.py", line 122, in testTooManyRepeatedNumerals
    self.assertRaises(roman2.InvalidRomanNumeralError, roman2.fromRoman, s)
  File "c:\python21\lib\unittest.py", line 266, in failUnlessRaises
    raise self.failureException, excName
AssertionError: InvalidRomanNumeralError ======================================================================
FAIL: fromRoman should give known result with known input
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage2\romantest2.py", line 99, in testFromRomanKnownValues
    self.assertEqual(integer, result)
  File "c:\python21\lib\unittest.py", line 273, in failUnlessEqual
    raise self.failureException, (msg or '%s != %s' % (first, second))
AssertionError: 1 != None ======================================================================
FAIL: fromRoman(toRoman(n))==n for all n
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage2\romantest2.py", line 141, in testSanity
    self.assertEqual(integer, result)
  File "c:\python21\lib\unittest.py", line 273, in failUnlessEqual
    raise self.failureException, (msg or '%s != %s' % (first, second))
AssertionError: 1 != None ======================================================================
FAIL: toRoman should fail with non-integer input
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage2\romantest2.py", line 116, in testNonInteger
    self.assertRaises(roman2.NotIntegerError, roman2.toRoman, 0.5)
  File "c:\python21\lib\unittest.py", line 266, in failUnlessRaises
    raise self.failureException, excName
AssertionError: NotIntegerError ======================================================================
FAIL: toRoman should fail with negative input
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage2\romantest2.py", line 112, in testNegative
    self.assertRaises(roman2.OutOfRangeError, roman2.toRoman, -1)
  File "c:\python21\lib\unittest.py", line 266, in failUnlessRaises
    raise self.failureException, excName
AssertionError: OutOfRangeError ======================================================================
FAIL: toRoman should fail with large input
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage2\romantest2.py", line 104, in testTooLarge
    self.assertRaises(roman2.OutOfRangeError, roman2.toRoman, 4000)
  File "c:\python21\lib\unittest.py", line 266, in failUnlessRaises
    raise self.failureException, excName
AssertionError: OutOfRangeError ======================================================================
FAIL: toRoman should fail with 0 input
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage2\romantest2.py", line 108, in testZero
    self.assertRaises(roman2.OutOfRangeError, roman2.toRoman, 0)
  File "c:\python21\lib\unittest.py", line 266, in failUnlessRaises
    raise self.failureException, excName
AssertionError: OutOfRangeError ----------------------------------------------------------------------
Ran 12 tests in 0.320s
FAILED (failures=10) 
```

# 14.3\. `roman.py`, 第 3 阶段

# 14.3\. `roman.py`, 第 3 阶段

现在 `toRoman` 对于有效的输入 (`1` 到 `3999` 整数) 已能正确工作，是正确处理那些无效输入 (任何其他输入) 的时候了。

## 例 14.6\. `roman3.py`

这个文件可以在例子目录下的 `py/roman/stage3/` 目录中找到。

如果您还没有下载本书附带的样例程序, 可以 [下载本程序和其他样例程序](http://www.woodpecker.org.cn/diveintopython/download/diveintopython-exampleszh-cn-5.4b.zip "Download example scripts")。

```py
"""Convert to and from Roman numerals"""
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
    if not (0 < n < 4000):                                             
        raise OutOfRangeError, "number out of range (must be 1..3999)" 
    if int(n) <> n:                                                    
        raise NotIntegerError, "non-integers can not be converted"
    result = ""                                                        
    for numeral, integer in romanNumeralMap:
        while n >= integer:
            result += numeral
            n -= integer
    return result
def fromRoman(s):
    """convert Roman numeral to integer"""
    pass 
```

|  |  |
| --- | --- |
| [1] | 这个写法很 Pythonic：一次进行多个比较。这等价于`if not ((0 &lt; n) and (n &lt; 4000))`，但是更容易让人理解。这是在进行范围检查，可以将过大的数、负数和零查出来。 |
| [2] | 你使用 `raise` 语句引发自己的异常。你可以引发任何内建异常或者已定义的自定义异常。第二个参数是可选的，如果给定，则会在异常未被处理时显示于追踪信息 (trackback) 之中。 |
| [3] | 这是一个非整数检查。非整数无法转化为罗马数字表示。 |
| [4] | 函数的其他部分未被更改。 |

## 例 14.7\. 观察 `toRoman` 如何处理无效输入

```py
>>> import roman3
>>> roman3.toRoman(4000)
Traceback (most recent call last):
  File "<interactive input>", line 1, in ?
  File "roman3.py", line 27, in toRoman
    raise OutOfRangeError, "number out of range (must be 1..3999)"
OutOfRangeError: number out of range (must be 1..3999)
>>> roman3.toRoman(1.5)
Traceback (most recent call last):
  File "<interactive input>", line 1, in ?
  File "roman3.py", line 29, in toRoman
    raise NotIntegerError, "non-integers can not be converted"
NotIntegerError: non-integers can not be converted 
```

## 例 14.8\. 用 `romantest3.py` 测试 `roman3.py` 的结果

```py
fromRoman should only accept uppercase input ... FAIL
toRoman should always return uppercase ... ok
fromRoman should fail with malformed antecedents ... FAIL
fromRoman should fail with repeated pairs of numerals ... FAIL
fromRoman should fail with too many repeated numerals ... FAIL
fromRoman should give known result with known input ... FAIL
toRoman should give known result with known input ... ok  fromRoman(toRoman(n))==n for all n ... FAIL
toRoman should fail with non-integer input ... ok  toRoman should fail with negative input ... ok  toRoman should fail with large input ... ok
toRoman should fail with 0 input ... ok 
```

|  |  |
| --- | --- |
| [1] | `toRoman` 仍然能通过已知值测试，这很令人鼓舞。所有第 2 阶段通过的测试仍然能通过，这说明新的代码没有对原有代码构成任何负面影响。 |
| [2] | 更令人振奋的是所有的无效输入测试现在都通过了。`testNonInteger` 这个测试能够通过是因为有了 `int(n) &lt;&gt; n` 检查。当一个非整数传递给 `toRoman` 时，`int(n) &lt;&gt; n` 检查出问题并引发 `NotIntegerError` 异常，这正是 `testNonInteger` 所期待的。 |
| [3] | `testNegative` 这个测试能够通过是因为 `not (0 &lt; n &lt; 4000)` 检查引发了 `testNegative` 期待的 `OutOfRangeError` 异常。 |

```py
 ======================================================================
FAIL: fromRoman should only accept uppercase input
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage3\romantest3.py", line 156, in testFromRomanCase
    roman3.fromRoman, numeral.lower())
  File "c:\python21\lib\unittest.py", line 266, in failUnlessRaises
    raise self.failureException, excName
AssertionError: InvalidRomanNumeralError ======================================================================
FAIL: fromRoman should fail with malformed antecedents
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage3\romantest3.py", line 133, in testMalformedAntecedent
    self.assertRaises(roman3.InvalidRomanNumeralError, roman3.fromRoman, s)
  File "c:\python21\lib\unittest.py", line 266, in failUnlessRaises
    raise self.failureException, excName
AssertionError: InvalidRomanNumeralError ======================================================================
FAIL: fromRoman should fail with repeated pairs of numerals
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage3\romantest3.py", line 127, in testRepeatedPairs
    self.assertRaises(roman3.InvalidRomanNumeralError, roman3.fromRoman, s)
  File "c:\python21\lib\unittest.py", line 266, in failUnlessRaises
    raise self.failureException, excName
AssertionError: InvalidRomanNumeralError ======================================================================
FAIL: fromRoman should fail with too many repeated numerals
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage3\romantest3.py", line 122, in testTooManyRepeatedNumerals
    self.assertRaises(roman3.InvalidRomanNumeralError, roman3.fromRoman, s)
  File "c:\python21\lib\unittest.py", line 266, in failUnlessRaises
    raise self.failureException, excName
AssertionError: InvalidRomanNumeralError ======================================================================
FAIL: fromRoman should give known result with known input
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage3\romantest3.py", line 99, in testFromRomanKnownValues
    self.assertEqual(integer, result)
  File "c:\python21\lib\unittest.py", line 273, in failUnlessEqual
    raise self.failureException, (msg or '%s != %s' % (first, second))
AssertionError: 1 != None ======================================================================
FAIL: fromRoman(toRoman(n))==n for all n
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage3\romantest3.py", line 141, in testSanity
    self.assertEqual(integer, result)
  File "c:\python21\lib\unittest.py", line 273, in failUnlessEqual
    raise self.failureException, (msg or '%s != %s' % (first, second))
AssertionError: 1 != None ----------------------------------------------------------------------
Ran 12 tests in 0.401s
FAILED (failures=6) 
```

|  |  |
| --- | --- |
| [1] | 你已将失败降至 6 个，而且它们都是关于 `fromRoman` 的：已知值测试、三个独立的无效输入测试，大小写检查和完备性检查。这意味着 `toRoman` 通过了所有可以独立通过的测试 (完备性测试也测试它，但需要 `fromRoman` 编写后一起测试)。这就是说，你应该停止对 `toRoman` 的代码编写。不必再推敲，不必再做额外的检查 “恰到好处”。停下来吧！现在，别再敲键盘了。 |

> 注意
> 全面的单元测试能够告诉你的最重要的事情是什么时候停止编写代码。当一个函数的所有单元测试都通过了，停止编写这个函数。一旦整个模块的单元测试通过了，停止编写这个模块。

# 14.4\. `roman.py`, 第 4 阶段

# 14.4\. `roman.py`, 第 4 阶段

现在 `toRoman` 完成了，是开始编写 `fromRoman` 的时候了。感谢那个将每个罗马数字和对应整数关连的完美数据结构，这个工作不比 `toRoman` 函数复杂。

## 例 14.9\. `roman4.py`

这个文件可以在例子目录下的 `py/roman/stage4/` 目录中找到。

如果您还没有下载本书附带的样例程序, 可以 [下载本程序和其他样例程序](http://www.woodpecker.org.cn/diveintopython/download/diveintopython-exampleszh-cn-5.4b.zip "Download example scripts")。

```py
"""Convert to and from Roman numerals"""
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
# toRoman function omitted for clarity (it hasn't changed)
def fromRoman(s):
    """convert Roman numeral to integer"""
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
| [1] | 这和 `toRoman` 的工作模式很相似。你遍历整个罗马数字数据结构 (一个元组的元组)，与前面不同的是不去一个个搜寻最大的整数，而是搜寻 “最大的”罗马数字字符串。 |

## 例 14.10\. `fromRoman` 如何工作

如果你不清楚 `fromRoman` 如何工作，在 `while` 结尾处添加一个 `print` 语句：

```py
 while s[index:index+len(numeral)] == numeral:
            result += integer
            index += len(numeral)
            print 'found', numeral, 'of length', len(numeral), ', adding', integer 
```

```py
>>> import roman4
>>> roman4.fromRoman('MCMLXXII')
found M , of length 1, adding 1000
found CM , of length 2, adding 900
found L , of length 1, adding 50
found X , of length 1, adding 10
found X , of length 1, adding 10
found I , of length 1, adding 1
found I , of length 1, adding 1
1972 
```

## 例 14.11\. 用 `romantest4.py` 测试 `roman4.py` 的结果

```py
fromRoman should only accept uppercase input ... FAIL
toRoman should always return uppercase ... ok
fromRoman should fail with malformed antecedents ... FAIL
fromRoman should fail with repeated pairs of numerals ... FAIL
fromRoman should fail with too many repeated numerals ... FAIL
fromRoman should give known result with known input ... ok  toRoman should give known result with known input ... ok
fromRoman(toRoman(n))==n for all n ... ok  toRoman should fail with non-integer input ... ok
toRoman should fail with negative input ... ok
toRoman should fail with large input ... ok
toRoman should fail with 0 input ... ok 
```

|  |  |
| --- | --- |
| [1] | 这儿有两个令人激动的消息。一个是 `fromRoman` 对于所有有效输入运转正常，至少对于你测试的已知值是这样。 |
| [2] | 第二个好消息是，完备性测试也通过了。与已知值测试的通过一起来看，你有理由相信 `toRoman` 和 `fromRoman` 对于所有有效输入值工作正常。(尚不能完全相信，理论上存在这种可能性：`toRoman` 存在错误而导致一些特定输入会产生错误的罗马数字表示，*并且* `fromRoman` 也存在相应的错误，把 `toRoman` 错误产生的这些罗马数字错误地转换为最初的整数。取决于你的应用程序和你的要求，你或许需要考虑这个可能性。如果是这样，编写更全面的测试用例直到解决这个问题。) |

```py
 ======================================================================
FAIL: fromRoman should only accept uppercase input
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage4\romantest4.py", line 156, in testFromRomanCase
    roman4.fromRoman, numeral.lower())
  File "c:\python21\lib\unittest.py", line 266, in failUnlessRaises
    raise self.failureException, excName
AssertionError: InvalidRomanNumeralError ======================================================================
FAIL: fromRoman should fail with malformed antecedents
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage4\romantest4.py", line 133, in testMalformedAntecedent
    self.assertRaises(roman4.InvalidRomanNumeralError, roman4.fromRoman, s)
  File "c:\python21\lib\unittest.py", line 266, in failUnlessRaises
    raise self.failureException, excName
AssertionError: InvalidRomanNumeralError ======================================================================
FAIL: fromRoman should fail with repeated pairs of numerals
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage4\romantest4.py", line 127, in testRepeatedPairs
    self.assertRaises(roman4.InvalidRomanNumeralError, roman4.fromRoman, s)
  File "c:\python21\lib\unittest.py", line 266, in failUnlessRaises
    raise self.failureException, excName
AssertionError: InvalidRomanNumeralError ======================================================================
FAIL: fromRoman should fail with too many repeated numerals
---------------------------------------------------------------------- Traceback (most recent call last):
  File "C:\docbook\dip\py\roman\stage4\romantest4.py", line 122, in testTooManyRepeatedNumerals
    self.assertRaises(roman4.InvalidRomanNumeralError, roman4.fromRoman, s)
  File "c:\python21\lib\unittest.py", line 266, in failUnlessRaises
    raise self.failureException, excName
AssertionError: InvalidRomanNumeralError ----------------------------------------------------------------------
Ran 12 tests in 1.222s
FAILED (failures=4) 
```

# 14.5\. `roman.py`, 第 5 阶段

# 14.5\. `roman.py`, 第 5 阶段

现在 `fromRoman` 对于有效输入能够正常工作了，是揭开最后一个谜底的时候了：使它正常工作于无效输入的情况下。这意味着要找出一个方法检查一个字符串是不是有效的罗马数字。这比 `toRoman` 中验证有效的数字输入困难，但是你可以使用一个强大的工具：正则表达式。

如果你不熟悉正则表达式，并且没有读过 第七章 *正则表达式*，现在是该好好读读的时候了。

如你在 第 7.3 节 “个案研究：罗马字母”中所见到的，构建罗马数字有几个简单的规则：使用字母 `M`, `D`, `C`, `L`, `X`, `V` 和 `I`。让我们回顾一下：

1.  字符是被“加”在一起的：`I` 是 `1`，`II` 是 `2`，`III` 是 `3`。`VI` 是 `6` (看上去就是 “`5` 加 `1`”)，`VII` 是 `7`，`VIII` 是 `8`。
2.  这些字符 (`I`, `X`, `C` 和 `M`) 最多可以重复三次。对于 `4`，你则需要利用下一个能够被 5 整除的字符进行减操作得到。你不能把 `4` 表示为 `IIII` 而应该表示为 `IV` (“比 `5` 小 `1` ”)。`40` 则被写作 `XL` (“比 `50` 小 `10`”)，`41` 表示为 `XLI`，`42` 表示为 `XLII`，`43` 表示为 `XLIII`，`44` 表示为 `XLIV` (“比`50`小`10`，加上 `5` 小 `1`”)。
3.  类似地，对于数字 `9`，你必须利用下一个能够被 10 整除的字符进行减操作得到：`8` 是 `VIII`，而 `9` 是 `IX` (“比 `10` 小 `1`”)，而不是 `VIIII` (由于 `I` 不能重复四次)。`90` 表示为 `XC`，`900` 表示为 `CM`。
4.  含五的字符不能被重复：`10` 应该表示为 `X`，而不会是 `VV`。`100` 应该表示为 `C`，而不是 `LL`。
5.  罗马数字一般从高位到低位书写，从左到右阅读，因此不同顺序的字符意义大不相同。`DC` 是 `600`，`CD` 是完全另外一个数 (`400`，“比 `500` 少 `100`”)。`CI` 是 `101`，而 `IC` 根本就不是一个有效的罗马数字 (因为你无法从`100`直接减`1`，应该写成 `XCIX`，意思是 “比 `100` 少 `10`，然后加上数字 `9`，也就是比 `10` 少 `1`”)。

## 例 14.12\. `roman5.py`

这个程序可以在例子目录下的`py/roman/stage5/` 目录中找到。

如果您还没有下载本书附带的样例程序, 可以 [下载本程序和其他样例程序](http://www.woodpecker.org.cn/diveintopython/download/diveintopython-exampleszh-cn-5.4b.zip "Download example scripts")。

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
    if not (0 < n < 4000):
        raise OutOfRangeError, "number out of range (must be 1..3999)"
    if int(n) <> n:
        raise NotIntegerError, "non-integers can not be converted"
    result = ""
    for numeral, integer in romanNumeralMap:
        while n >= integer:
            result += numeral
            n -= integer
    return result
#Define pattern to detect valid Roman numerals
romanNumeralPattern = '^M?M?M?(CM|CD|D?C?C?C?)(XC|XL|L?X?X?X?)(IX|IV|V?I?I?I?)$' 
def fromRoman(s):
    """convert Roman numeral to integer"""
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
| [1] | 这只是 第 7.3 节 “个案研究：罗马字母” 中讨论的匹配模版的继续。十位上可能是`XC` (`90`)，`XL` (`40`)，或者可能是 `L` 后面跟着 0 到 3 个 `X` 字符。个位则可能是 `IX` (`9`)，`IV` (`4`)，或者是一个可能是 `V` 后面跟着 0 到 3 个 `I` 字符。 |
| [2] | 把所有的逻辑编码成正则表达式，检查无效罗马字符的代码就很简单了。如果 `re.search` 返回一个对象则表示匹配了正则表达式，输入是有效的，否则输入无效。 |

这里你可能会怀疑，这个面目可憎的正则表达式是否真能查出错误的罗马字符表示。没关系，不必完全听我的，不妨看看下面的结果：

## 例 14.13\. 用 `romantest5.py` 测试 `roman5.py` 的结果

```py
 fromRoman should only accept uppercase input ... ok  toRoman should always return uppercase ... ok
fromRoman should fail with malformed antecedents ... ok  fromRoman should fail with repeated pairs of numerals ... ok  fromRoman should fail with too many repeated numerals ... ok
fromRoman should give known result with known input ... ok
toRoman should give known result with known input ... ok
fromRoman(toRoman(n))==n for all n ... ok
toRoman should fail with non-integer input ... ok
toRoman should fail with negative input ... ok
toRoman should fail with large input ... ok
toRoman should fail with 0 input ... ok
----------------------------------------------------------------------
Ran 12 tests in 2.864s
OK 
```

|  |  |
| --- | --- |
| [1] | 有件事我未曾讲过，那就是默认情况下正则表达式大小写敏感。由于正则表达式 `romanNumeralPattern` 是以大写字母构造的，`re.search` 将拒绝不全部是大写字母构成的输入。因此大写输入的检查就通过了。 |
| [2] | 更重要的是，无效输入测试也通过了。例如，上面这个用例测试了 `MCMC` 之类的情形。正如你所见，这不匹配正则表达式，因此 `fromRoman` 引发一个测试用例正在等待的 `InvalidRomanNumeralError` 异常，所以测试通过了。 |
| [3] | 事实上，所有的无效输入测试都通过了。正则表达式捕捉了你在编写测试用例时所能预见的所有情况。 |
| [4] | 最终迎来了 “`OK`”这个平淡的“年度大奖”，所有测试都通过后 `unittest` 模块就会输出它。 |

> 注意
> 当所有测试都通过了，停止编程。