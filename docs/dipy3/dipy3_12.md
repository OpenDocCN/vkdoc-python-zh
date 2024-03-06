# Chapter 9 单元测试

# Chapter 9 单元测试

> " Certitude is not the test of certainty. We have been cocksure of many things that were not so. " — [Oliver Wendell Holmes, Jr.](http://en.wikiquote.org/wiki/Oliver_Wendell_Holmes,_Jr.)

## （不要）深入

在此章节中，你将要编写及调试一系列用于阿拉伯数字与罗马数字相互转换的方法。你阅读了在“案例学习：罗马数字”中关于构建及校验罗马数字的机制。那么，现在考虑扩展该机制为一个双向的方法。

罗马数字的规则引出很多有意思的结果：

1.  只有一种正确的途径用阿拉伯数字表示罗马数字。
2.  反过来一样，一个字符串类型的有效的罗马数字也仅可以表示一个阿拉伯数字（即，这种转换方式也是只有一种）。
3.  只有有限范围的阿拉伯数字可以以罗马数字表示，那就是 1-3999。而罗马数字表示大数字却有几种方式。例如，为了表示一个数字连续出现时正确的值则需要乘以`1000`。为了达到本节的目的，限定罗马数字在 1 到 3999 之间。
4.  无法用罗马数字来表示 0 。
5.  无法用罗马数字来表示负数 。
6.  无法用罗马数字来表示分数或非整数 。

现在，开始设计 `roman.py` 模块。它有两个主要的方法：`to_roman()` 及 `from_roman()`。`to_roman()` 方法接收一个从 `1` 到 `3999` 之间的整型数字，然后返回一个字符串类型的罗马数字。

在这里停下来。现在让我们进行一些意想不到的操作：编写一个测试用例来检测 `to_roman` 函数是否实现了你想要的功能。你想得没错：你正在编写测试尚未编写代码的代码。

这就是所谓的*测试驱动开发* 或 TDD。那两个转换方法（ `to_roman()` 及之后的 `from_roman()`）可以独立于任何使用它们的大程序而作为一个单元来被编写及测试。Python 自带一个单元测试框架，被恰当地命名为 `unittest` 模块。

单元测试是整个以测试为中心的开发策略中的一个重要部分。编写单元测试应该安排在项目的早期，同时要让它随同代码及需求变更一起更新。很多人都坚持测试代码应该先于被测试代码的，而这种风格也是我在本节中所主张的。但是，不管你何时编写，单元测试都是有好处的。

*   在编写代码之前，通过编写单元测试来强迫你使用有用的方式细化你的需求。
*   在编写代码时，单元测试可以使你避免过度编码。当所有测试用例通过时，实现的方法就完成了。
*   重构代码时，单元测试用例有助于证明新版本的代码跟老版本功能是一致的。
*   在维护代码期间，如果有人对你大喊：你最新的代码修改破坏了原有代码的状态，那么此时单元测试可以帮助你反驳（“*先生*，所有单元测试用例通过了我才提交代码的...”）。
*   在团队编码中，缜密的测试套件可以降低你的代码影响别人代码的机会，这是因为你需要优先执行别人的单元测试用例。（我曾经在代码冲刺见过这种实践。一个团队把任务分解，每个人领取其中一小部分任务，同时为其编写单元测试;然后，团队相互分享他们的单元测试用例。这样，所有人都可以在编码过程中提前发现谁的代码与其他人的不可以良好工作。）

## 一个简单的问题

每个测试都是一个孤岛。

一个测试用例仅回答一个关于它正在测试的代码问题。一个测试用例应该可以：

*   ……完全自动运行，而不需要人工干预。单元测试几乎是全自动的。
*   ……自主判断被测试的方法是通过还是失败，而不需要人工解释结果。
*   ……独立运行，而不依赖其它测试用例（即使测试的是同样的方法）。即，每一个测试用例都是一个孤岛。

让我们据此为第一个需求建立一个测试用例：

1.  `to_roman()` 方法应该返回代表`1`-`3999`的罗马数字。

这些代码功效如何并不那么显而易见。它定义了一个没有`__init__` 方法的类。而该类*当然*有其它方法，但是这些方法都不会被调用。在整个脚本中，有一个**main** 块，但它并不引用该类及它的方法。但我承诺，它做别的事情了。

```py
import roman1
import unittest

    known_values = ( (1, 'I'),
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

        '''to_roman should give known result with known input'''
        for integer, numeral in self.known_values:

if __name__ == '__main__':
    unittest.main() 
```

1.  为了编写测试用例，首先使该测试用例类成为`unittest` 模块的`TestCase` 类的子类。TestCase 提供了很多你可以用于测试特定条件的测试用例的有用的方法。
2.  这是一张我手工核实过的整型数字-罗马数字对的列表。它包括最小的十个数字、最大数字、每一个有唯一一个字符串格式的罗马数字的数字以及一个有其它有效数字产生的随机数。你没有必要测试每一个可能的输入，而需要测试所有明显的边界用例。
3.  每一个独立的测试都有它自己的不含参数及没有返回值的方法。如果方法不抛出异常而正常退出则认为测试通过;否则，测试失败。
4.  这里调用了真实的 `to_roman()` 方法. （当然，该方法还没编写;但一旦该方法被实现，这就是调用它的行号）。注意，现在你已经为 `to_roman()` 方法定义了 接口：它必须包含一个整型（被转换的数字）及返回一个字符串（罗马数字的表示形式）。如果 接口 实现与这些定义不一致，那么测试就会被视为失败。同样，当你调用 `to_roman()` 时，不要捕获任何异常。这些都是 unittest 故意设计的。当你以有效的输入调用 `to_roman()` 时它不会抛出异常。如果 `to_roman()` 抛出了异常，则测试被视为失败。
5.  假设 `to_roman()` 方法已经被正确定义，正确调用，成功实现以及返回了一个值，那么最后一步就是去检查它的返回值是否 *right* 。这是测试中一个普遍的问题。`TestCase` 类提供了一个方法 `assertEqual` 来检查两个值是否相等。如果 `to_roman()` (`result`) 的返回值跟已知的期望值 g (`numeral`)不一致，则抛出异常，并且测试失败。如果两值相等， `assertEqual` 不会做任何事情。如果 `to_roman()` 的所有返回值均与已知的期望值一致，则 `assertEqual` 不会抛出任何异常，于是，`test_to_roman_known_values` 最终会会正常退出，这就意味着 `to_roman()` 通过此次测试。

编写一个失败的测试，然后进行编码直到该测试通过。

一旦你有了测试用例，你就可以开始编写 `to_roman()` 方法。首先，你应该用一个空方法作为存根，同时确认该测试失败。因为如果在编写任何代码之前测试已经通过，那么你的测试对你的代码是完全不会有效果的！单元测试就像跳舞：测试先行，编码跟随。编写一个失败的测试，然后进行编码直到该测试通过。

```py
# roman1.py

def to_roman(n):
    '''convert integer to Roman numeral''' 
```

1.  在此阶段，你想定义 to_roman()方法的 API ，但是你还不想编写（首先，你的测试需要失败）。为了存根，需要使用 Python 保留关键字`pass`，它恰恰什么都没做。

在命令行上运行 `romantest1.py` 来执行该测试。如果使用-v 命令行参数的话，会有更详细的输出来帮助你精确地查看每一条用例的执行过程。幸运的话，你的输出应该如下：

```py
you@localhost:~/diveintopython3/examples$ python3 romantest1.py -v

 ======================================================================
FAIL: to_roman should give known result with known input
----------------------------------------------------------------------
Traceback (most recent call last):
  File "romantest1.py", line 73, in test_to_roman_known_values
    self.assertEqual(numeral, result)

---------------------------------------------------------------------- 
```

1.  运行脚本就会执行 `unittest.main()` , 该方法执行了每一条测试用例。而每一条测试用例都是 `romantest.py` 中的类方法。这些测试类没有必要的组织要求;它们每一个都包括一个独立的测试方法，或者你也可以编写一个含有多个测试方法的类。唯一的要求就是每一个测试类都必须继承 `unittest.TestCase`。
2.  对于每一个测试用例， `unittest` 模块会打印出测试方法的 `docstring` ，并且说明该测试失败还是成功。正如预期那样，该测试用例失败了。
3.  对于每一个失败的测试用例， `unittest` 模块会打印出详细的跟踪信息。在该用例中， `assertEqual()` 的调用抛出了一个 `AssertionError` 的异常，这是因为 `to_roman(1)` 本应该返回 `'I'` 的，但是它没有。（因为没有显示的返回值，故方法返回了 Python 的空值 `None`）
4.  在说明每个用例的详细执行结果之后， `unittest` 打印出一个简述来说明“多少用例被执行了”和“测试执行了多长时间”。
5.  从整体上说，该测试执行失败，因为至少有一条用例没有成功。如果测试用例没有通过的话， `unittest` 可以区别用例执行失败跟程序错误的。像 `assertXYZ` 、`assertRaises` 这样的 `assertEqual` 方法的失败是因为被声明的条件不是为真，或者预期的异常没有抛出。错误，则是另一种异常，它是因为被测试的代码或者单元测试用例本身的代码问题而引起的。

*至此*，你可以实现 `to_roman()` 方法了。

```py
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

def to_roman(n):
    '''convert integer to Roman numeral'''
    result = ''
    for numeral, integer in roman_numeral_map:

            result += numeral
            n -= integer
    return result 
```

1.  `roman_numeral_map` 是一个由元组组成的元组，它定义了三样东西：代表最基本的罗马数字的字符、罗马数字的顺序（逆序，从 `M` 到 `I`）、每一个罗马数字的阿拉伯数值。每一个内部的元组都是一个`(`数`，`值`)`对。它不但定义了单字符罗马数字，也定义了双字符罗马数字，如`CM`（“比一千小一百”）。该元组使得 `to_roman()` 方法实现起来更简单。
2.  这里得益于 `roman_numeral_map` 的数据结构，因为你不需要任何特别得逻辑去处理减法。为了转化成罗马数字，通过查找等于或者小于输入值的最大值来简化对 `roman_numeral_map` 的迭代。一旦找到，就把罗马数字的字符串追加至输出值（result）末段，同时输入值要减去相应的数值，如此重复。

如果你仍然不清楚 `to_roman()` 如何工作，可以在 `while` 循环末段添加 `print()` 调用：

```py
 while n >= integer:
    result += numeral
    n -= integer
    print('subtracting {0} from input, adding {1} to output'.format(integer, numeral)) 
```

因为用于调试的 `print()` 声明，输出会如下：

```py
>>> import roman1
>>> roman1.to_roman(1424)
subtracting 1000 from input, adding M to output
subtracting 400 from input, adding CD to output
subtracting 10 from input, adding X to output
subtracting 10 from input, adding X to output
subtracting 4 from input, adding IV to output
'MCDXXIV' 
```

这样， `to_roman()` 至少在手工检查下是工作正常的。但它会通过你编写的测试用例么？

```py
you@localhost:~/diveintopython3/examples$ python3 romantest1.py -v
test_to_roman_known_values (__main__.KnownValues)

----------------------------------------------------------------------
Ran 1 test in 0.016s

OK 
```

1.  万岁！`to_roman()` 函数通过了“known values” 测试用例。该测试用例并不复杂，但是它的确使该方法按着输入值的变化而执行，其中的输入值包括：每一个单字符罗马数字、最大值数字（`3999`）、最长字符串数字（`3888`）。通过这些，你就可以有理由对“该方法接收任何正常的输入值都工作正常”充满信心了。

“正常”输入？”嗯。那“非法”输入呢？

## “停止然后着火”

Python 方式的停止并点火实际是引发一个例外。

仅仅在“正常”值时证明方法通过的测试是不够的;你同样需要测试当输入“非法”值时方法失败。但并不是说要枚举所有的失败类型，而是说必要在你预期的范围内失败。

```py
>>> import roman1
>>> roman1.to_roman(4000)
'MMMM'
>>> roman1.to_roman(5000)
'MMMMM'

'MMMMMMMMM' 
```

1.  这明显不是你所期望的──那也不是一个合法的罗马数字！事实上，这些输入值都超过了允许的范围，但该函数却返回了假值。悄悄返回的错误值是 *很糟糕* 的，因为如果一个程序要挂掉的话，迅速且引人注目地挂掉会好很多。正如谚语“停止然后着火”。Python 方式的停止并点火实际是引发一个例外。

那问题是：我该如何表达这些内容为可测试需求呢？下面就是一个开始：

> 当输入值大于 `3999` 时， `to_roman()` 函数应该抛出一个 `OutOfRangeError` 异常。

具体测试代码如下：

```py
 '''to_roman should fail with large input''' 
```

1.  如前一个测试用例，创建一个继承于 `unittest.TestCase` 的类。你可以在每个类中实现多个测试（正如你在本节中将会看到的一样），但是我却选择了创建一个新类，因为该测试与上一个有点不同。这样，我们可以把正常输入的测试跟非法输入的测试分别放入不同的两个类中。
2.  如前一个测试用例，测试本身是类一个方法，并且该方法以 `test` 开头命名。
3.  `unittest.TestCase` 类提供 e `assertRaises` 方法，该方法需要以下参数：你期望的异常、你要测试的方法及传入给方法的参数。（如果被测试的方法需要多个参数的话，则把所有参数依次传入 `assertRaises`， assertRaises 会正确地把参数传递给被测方法的。）

请关注代码的最后一行。这里并不需要直接调用 `to_roman()` ，同时也不需要手动检查它抛出的异常类型（通过 一个 `try...except` 块来包装），而这些 `assertRaises` 方法都给我们完成了。你要做的所有事情就是告诉 assertRaises 你期望的异常类型（ `roman2.OutOfRangeError`）、被测方法（`to_roman()`）以及方法的参数（`4000`）。`assertRaises` 方法负责调用 `to_roman()` 和检查方法抛出 `roman2.OutOfRangeError` 的异常。

另外，注意你是把 `to_roman()` 方法作为参数传递;你没有调用被测方法，也不是把被测方法作为一个字符串名字传递进去。我是否在之前提到过 Python 中万物皆对象有多么轻便？

那么，当你执行该含有新测试的测试套件时，结果如下：

```py
you@localhost:~/diveintopython3/examples$ python3 romantest2.py -v
test_to_roman_known_values (__main__.KnownValues)
to_roman should give known result with known input ... ok
test_too_large (__main__.ToRomanBadInput)

======================================================================
ERROR: to_roman should fail with large input                          
----------------------------------------------------------------------
Traceback (most recent call last):
  File "romantest2.py", line 78, in test_too_large
    self.assertRaises(roman2.OutOfRangeError, roman2.to_roman, 4000)

----------------------------------------------------------------------
Ran 2 tests in 0.000s

FAILED (errors=1) 
```

1.  测试本应该是失败的（因为并没有任何代码使它通过），但是它没有真正的“失败”，而是出现了“错误”。这里有些微妙但是重要的区别。单元测试事实上有 *三种* 返回值：通过、失败以及错误。“通过”，但当然就是说测试成功了──被测代码符合你的预期。“失败”就是就如之前的测试用例一样（直到你编写代码令它通过）──执行了被测试的代码但返回值并不是所期望的。“错误”就是被测试的代码甚至没有正确执行。
2.  为什么代码没有正确执行呢？回溯说明了一切。你正在测试的模块没有叫 `OutOfRangeError` 的异常。回忆一下，该异常是你传递给 `assertRaises()` 方法的，因为你期望当传递给被测试方法一个超大值时可以抛出该异常。但是，该异常并不存在，因此 `assertRaises()` 的调用会失败。事实上测试代码并没有机会测试 `to_roman()` 方法，因为它还没有到达那一步。

为了解决该问题，你需要在 `roman2.py` 中定义 `OutOfRangeError` 。

1.  异常也是类。“越界”错误是值错误的一类──参数值超出了可接受的范围。所以，该异常继承了内建的 `ValueError` 异常类。这并不是严格的要求（它同样也可以继承于基类 `Exception`），只要它正确就行了。
2.  事实上，异常类可以不做任何事情，但是至少添加一行代码使其成为一个类。 `pass` 的真正意思是什么都不做，但是它是一行 Python 代码，所以可以使其成为类。

再次执行该测试套件。

```py
you@localhost:~/diveintopython3/examples$ python3 romantest2.py -v
test_to_roman_known_values (__main__.KnownValues)
to_roman should give known result with known input ... ok
test_too_large (__main__.ToRomanBadInput)

======================================================================
FAIL: to_roman should fail with large input
----------------------------------------------------------------------
Traceback (most recent call last):
  File "romantest2.py", line 78, in test_too_large
    self.assertRaises(roman2.OutOfRangeError, roman2.to_roman, 4000)

----------------------------------------------------------------------
Ran 2 tests in 0.016s

FAILED (failures=1) 
```

1.  新的测试仍然没有通过，但是它并没有返回错误而是失败。相反，测试失败了。这就是进步！它意味着这回 `assertRaises()` 方法的调用是成功的，同时，单元测试框架事实上也测试了 `to_roman()` 函数。
2.  当然 `to_roman()` 方法没有引发你所定义的 `OutOfRangeError` 异常，因为你并没有让它这么做。这真是个好消息！因为它意味着这是个合格的测试案例——在编写代码使之通过之前它将会以失败为结果。

现在可以编写代码使其通过了。

```py
def to_roman(n):
    '''convert integer to Roman numeral'''
    if n > 3999:

    result = ''
    for numeral, integer in roman_numeral_map:
        while n >= integer:
            result += numeral
            n -= integer
    return result 
```

1.  非常直观：如果给定的输入 (`n`) 大于`3999`，引发一个 `OutOfRangeError` 例外。本单元测试并不检测那些与例外相伴的人类可读的字符串，但你可以编写另一个测试来检查它（但请注意用户的语言或环境导致的不同国际化问题）。

这样能让测试通过吗？让我们来寻找答案。

```py
you@localhost:~/diveintopython3/examples$ python3 romantest2.py -v
test_to_roman_known_values (__main__.KnownValues)
to_roman should give known result with known input ... ok
test_too_large (__main__.ToRomanBadInput)

----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK 
```

1.  万岁！两个测试都通过了。因为你是在测试与编码之间来回反复开发的，所以你可以肯定使得其中一个测试从“失败”转变为“通过”的原因就是你刚才新添的两行代码。虽然这种信心来得并不简单，但是这种代价会在你代码的生命周期中得到回报。

## More Halting, More Fire

与测试超大值一样，也必须测试超小值。正如我们在功能需求中提到的那样，罗马数字无法表达 0 或负数。

```py
>>> import roman2
>>> roman2.to_roman(0)
''
>>> roman2.to_roman(-1)
'' 
```

显然，*这不是*好的结果。让我们为这些条件逐条添加测试。

```py
class ToRomanBadInput(unittest.TestCase):
    def test_too_large(self):
        '''to_roman should fail with large input'''

    def test_zero(self):
        '''to_roman should fail with 0 input'''

    def test_negative(self):
        '''to_roman should fail with negative input''' 
```

1.  `test_too_large()` 方法跟之前的步骤一样。我把它包含进来是为了说明新代码的位置。
2.  这里是新的测试方法： `test_zero()` 。如 `test_too_large()` 一样，它调用了在 n `unittest.TestCase` 中定义的 `assertRaises()` 方法，并且以参数值 0 传入给 `to_roman()`，最后检查它抛出相应的异常：`OutOfRangeError`。
3.  `test_negative()` 也几乎类似，除了它给 `to_roman()` 函数传入 `-1` 。如果新的测试中 *没有* 任何一个抛出了异常 `OutOfRangeError` （或者由于该函数返回了实际的值，或者由于它抛出了其他类型的异常），那么测试就被视为失败。

检查测试是否失败：

```py
you@localhost:~/diveintopython3/examples$ python3 romantest3.py -v
test_to_roman_known_values (__main__.KnownValues)
to_roman should give known result with known input ... ok
test_negative (__main__.ToRomanBadInput)
to_roman should fail with negative input ... FAIL
test_too_large (__main__.ToRomanBadInput)
to_roman should fail with large input ... ok
test_zero (__main__.ToRomanBadInput)
to_roman should fail with 0 input ... FAIL

======================================================================
FAIL: to_roman should fail with negative input
----------------------------------------------------------------------
Traceback (most recent call last):
  File "romantest3.py", line 86, in test_negative
    self.assertRaises(roman3.OutOfRangeError, roman3.to_roman, -1)
AssertionError: OutOfRangeError not raised by to_roman

======================================================================
FAIL: to_roman should fail with 0 input
----------------------------------------------------------------------
Traceback (most recent call last):
  File "romantest3.py", line 82, in test_zero
    self.assertRaises(roman3.OutOfRangeError, roman3.to_roman, 0)
AssertionError: OutOfRangeError not raised by to_roman

----------------------------------------------------------------------
Ran 4 tests in 0.000s

FAILED (failures=2) 
```

太棒了！两个测试都如期地失败了。接着转入被测试的代码并且思考如何才能使得测试通过。

```py
def to_roman(n):
    '''convert integer to Roman numeral'''

    result = ''
    for numeral, integer in roman_numeral_map:
        while n >= integer:
            result += numeral
            n -= integer
    return result 
```

1.  这是 Python 优雅的快捷方法：一次性的多比较。它等价于 `if not ((0 &lt; n) and (n &lt; 4000))`，但前者更适合阅读。这一行代码应该捕获那些超大的、负值的或者为 0 的输入。
2.  当你改变条件的时候，要确保同步更新那些提示错误信息的可读字符串。`unittest` 框架并不关心这些，但是如果你的代码抛出描述不正确的异常信息的话会使得手工调试代码变得困难。

我本应该给你展示完整的一系列与本章节不相关的例子来说明一次性多比较的快捷方式是有效的，但是我将仅仅运行本测试用例来证明它的有效性。

```py
you@localhost:~/diveintopython3/examples$ python3 romantest3.py -v
test_to_roman_known_values (__main__.KnownValues)
to_roman should give known result with known input ... ok
test_negative (__main__.ToRomanBadInput)
to_roman should fail with negative input ... ok
test_too_large (__main__.ToRomanBadInput)
to_roman should fail with large input ... ok
test_zero (__main__.ToRomanBadInput)
to_roman should fail with 0 input ... ok

----------------------------------------------------------------------
Ran 4 tests in 0.016s

OK 
```

## 还有一件事情……

还有一个把阿拉伯数字转换成罗马数字的 功能性需求 ：处理非整型数字。

```py
>>> import roman3

''

'I' 
```

1.  喔，糟糕了。
2.  喔，更糟糕了。两个用例都本该抛出异常的。但却返回了假的结果。

测试非整数并不困难。首先，定义一个 `NotIntegerError` 例外。

```py
# roman4.py
class OutOfRangeError(ValueError): pass
<mark>class NotIntegerError(ValueError): pass</mark> 
```

然后，编写一个检查 `NotIntegerError` 例外的案例。

```py
class ToRomanBadInput(unittest.TestCase):
    .
    .
    .
    def test_non_integer(self):
        '''to_roman should fail with non-integer input'''
 <mark>self.assertRaises(roman4.NotIntegerError, roman4.to_roman, 0.5)</mark> 
```

然后，检查该测试是否可以正确地失败。

```py
you@localhost:~/diveintopython3/examples$ python3 romantest4.py -v
test_to_roman_known_values (__main__.KnownValues)
to_roman should give known result with known input ... ok
test_negative (__main__.ToRomanBadInput)
to_roman should fail with negative input ... ok
test_non_integer (__main__.ToRomanBadInput)
to_roman should fail with non-integer input ... FAIL
test_too_large (__main__.ToRomanBadInput)
to_roman should fail with large input ... ok
test_zero (__main__.ToRomanBadInput)
to_roman should fail with 0 input ... ok

======================================================================
FAIL: to_roman should fail with non-integer input
----------------------------------------------------------------------
Traceback (most recent call last):
  File "romantest4.py", line 90, in test_non_integer
    self.assertRaises(roman4.NotIntegerError, roman4.to_roman, 0.5)
<mark>AssertionError: NotIntegerError not raised by to_roman</mark>

----------------------------------------------------------------------
Ran 5 tests in 0.000s

FAILED (failures=1) 
```

编修代码，使得该测试可以通过。

```py
def to_roman(n):
    '''convert integer to Roman numeral'''
    if not (0 < n < 4000):
        raise OutOfRangeError('number out of range (must be 1..3999)')

    result = ''
    for numeral, integer in roman_numeral_map:
        while n >= integer:
            result += numeral
            n -= integer
    return result 
```

1.  内建的 `isinstance()` 方法可以检查一个变量是否属于某一类型（或者，技术上的任何派生类型）。
2.  如果参数 `n` 不是 `int`，则抛出新定义的 `NotIntegerError` 异常。

最后，验证修改后的代码的确通过测试。

```py
you@localhost:~/diveintopython3/examples$ python3 romantest4.py -v
test_to_roman_known_values (__main__.KnownValues)
to_roman should give known result with known input ... ok
test_negative (__main__.ToRomanBadInput)
to_roman should fail with negative input ... ok
test_non_integer (__main__.ToRomanBadInput)
to_roman should fail with non-integer input ... ok
test_too_large (__main__.ToRomanBadInput)
to_roman should fail with large input ... ok
test_zero (__main__.ToRomanBadInput)
to_roman should fail with 0 input ... ok

----------------------------------------------------------------------
Ran 5 tests in 0.000s

OK 
```

`to_roman()` 方法通过了所有的测试，而且我也想不出别的测试了，因此，下面着手 `from_roman()`吧！

## 可喜的对称性

转换罗马数字为阿拉伯数字的实现难度听起来比反向转换要困难。当然，这种想法不无道理。例如，检查数值是否比 0 大容易，而检查一个字符串是否为有效的罗马数字则要困难些。但是，我们已经构造了一个用于检查罗马数字的规则表，因此规则表的工作可以免了。

现在剩余的工作就是转换字符串了。正如我们将要看到的一样，多亏我们定义的用于单个罗马数字映射至阿拉伯数字的良好的数据结构，`from_roman()` 的实现本质上与 `to_roman()` 一样简单。

不过，测试先行！为了证明其准确性，我们将需要一个对“已知取值”进行的测试。我们的测试套件已经包含了一个已知取值的映射表，那么，我们就重用它。

```py
 def test_from_roman_known_values(self):
        '''from_roman should give known result with known input'''
        for integer, numeral in self.known_values:
            result = roman5.from_roman(numeral)
            self.assertEqual(integer, result) 
```

这里看到了令人高兴的对称性。`to_roman()` 与 `from_roman()` 函数是互逆的。前者把整型数字转换为特殊格式化的字符串，而后者则把特殊格式化的字符串转换为整型数字。理论上，我们应该可以使一个数字“绕一圈”，即把数字传递给 `to_roman()` 方法，得到一个字符串;然后把该字符串传入 `from_roman()` 方法，得到一个整型数字，并且跟传给 to_roman()方法的数字是一样的。

```py
n = from_roman(to_roman(n)) for all values of n 
```

在本用例中，“全有取值”是说 `从 1 到 3999` 的所有数值，因为这是 `to_roman()` 方法的有效输入范围。为了表达这两个方法之间的对称性，我们可以设计这样的测试用例，它的测试数据集是从`1 到 3999 之间`（包括 1 和 3999）的所有数值，首先调用 `to_roman()` ，然后调用 `from_roman()`，最后检查输出是否与原始输入一致。

```py
class RoundtripCheck(unittest.TestCase):
    def test_roundtrip(self):
        '''from_roman(to_roman(n))==n for all n'''
        for integer in range(1, 4000):
            numeral = roman5.to_roman(integer)
            result = roman5.from_roman(numeral)
            self.assertEqual(integer, result) 
```

这些测试连失败的机会都没有。因为我们根本还没定义 `from_roman()` 函数，所以它们仅仅会抛出错误的结果。

```py
you@localhost:~/diveintopython3/examples$ python3 romantest5.py
E.E....
======================================================================
ERROR: test_from_roman_known_values (__main__.KnownValues)
from_roman should give known result with known input
----------------------------------------------------------------------
Traceback (most recent call last):
  File "romantest5.py", line 78, in test_from_roman_known_values
    result = roman5.from_roman(numeral)
AttributeError: 'module' object has no attribute 'from_roman'

======================================================================
ERROR: test_roundtrip (__main__.RoundtripCheck)
from_roman(to_roman(n))==n for all n
----------------------------------------------------------------------
Traceback (most recent call last):
  File "romantest5.py", line 103, in test_roundtrip
    result = roman5.from_roman(numeral)
AttributeError: 'module' object has no attribute 'from_roman'

----------------------------------------------------------------------
Ran 7 tests in 0.019s

FAILED (errors=2) 
```

一个简易的留空函数可以解决此问题。

```py
# roman5.py
def from_roman(s):
    '''convert Roman numeral to integer''' 
```

（嘿，你注意到了么？我定义了一个除了 docstring 之外没有任何东西的方法。这是合法的 Python 代码。事实上，一些程序员喜欢这样做。“不要留空；写点文档！”）

现在测试用力将会失败。

```py
you@localhost:~/diveintopython3/examples$ python3 romantest5.py
F.F....
======================================================================
FAIL: test_from_roman_known_values (__main__.KnownValues)
from_roman should give known result with known input
----------------------------------------------------------------------
Traceback (most recent call last):
  File "romantest5.py", line 79, in test_from_roman_known_values
    self.assertEqual(integer, result)
AssertionError: 1 != None

======================================================================
FAIL: test_roundtrip (__main__.RoundtripCheck)
from_roman(to_roman(n))==n for all n
----------------------------------------------------------------------
Traceback (most recent call last):
  File "romantest5.py", line 104, in test_roundtrip
    self.assertEqual(integer, result)
AssertionError: 1 != None

----------------------------------------------------------------------
Ran 7 tests in 0.002s

FAILED (failures=2) 
```

现在是时候编写 `from_roman()` 函数了。

```py
def from_roman(s):
    """convert Roman numeral to integer"""
    result = 0
    index = 0
    for numeral, integer in roman_numeral_map:

            result += integer
            index += len(numeral)
    return result 
```

1.  此处的匹配模式与 `to_roman()` 完全相同。遍历整个罗马数字数据结构 (一个元组的元组)，与前面不同的是不去一个个地搜索最大的整数，而是搜寻 “最大的”罗马数字字符串。

如果不清楚 `from_roman()` 如何工作，在 `while` 结尾处添加一个 `print` 语句：

```py
def from_roman(s):
    """convert Roman numeral to integer"""
    result = 0
    index = 0
    for numeral, integer in roman_numeral_map:
        while s[index:index+len(numeral)] == numeral:
            result += integer
            index += len(numeral)
 <mark>print('found', numeral, 'of length', len(numeral), ', adding', integer)</mark> 
```

```py
>>> import roman5
>>> roman5.from_roman('MCMLXXII')
found M of length 1, adding 1000
found CM of length 2, adding 900
found L of length 1, adding 50
found X of length 1, adding 10
found X of length 1, adding 10
found I of length 1, adding 1
found I of length 1, adding 1
1972 
```

重新执行一遍测试。

```py
you@localhost:~/diveintopython3/examples$ python3 romantest5.py
.......
----------------------------------------------------------------------
Ran 7 tests in 0.060s

OK 
```

这儿有两个令人激动的消息。一个是 `from_roman()` 对于所有有效输入运转正常，至少对于你测试的已知值是这样。第二个好消息是，完备性测试也通过了。与已知值测试的通过一起来看，你有理由相信 `to_roman()` 和 `from_roman()` 对于所有有效输入值工作正常。(尚不能完全相信，理论上存在这种可能性： `to_roman()` 存在错误而导致一些特定输入会产生错误的罗马数字表示，*and* `from_roman()` 也存在相应的错误，把 `to_roman()` 错误产生的这些罗马数字错误地转换为最初的整数。取决于你的应用程序和你的要求，你或许需要考虑这个可能性；如果是这样，编写更全面的测试用例直到解决这个问题。)

## 更多错误输入

现在 `from_roman()` 对于有效输入能够正常工作了，是揭开最后一个谜底的时候了：使它正常工作于无效输入的情况下。这意味着要找出一个方法检查一个字符串是不是有效的罗马数字。这比中验证有效的数字输入困难，但是你可以使用一个强大的工具：正则表达式。(如果你不熟悉正则表达式，现在是该好好读读正则表达式那一章节的时候了。)

如你在 个案研究：罗马字母 s 中所见到的，构建罗马数字有几个简单的规则：使用的字母`M` ， `D` ， `C` ， `L` ， `X` ， `V`和`I` 。让我们回顾一下：

*   有时字符是叠加组合的。`I` 是 `1`, `II` 是 `2`,而`III` 是 `3`. `VI` 是 `6` (从字面上理解, “`5` 和 `1`”), `VII` 是 `7`, 而 `VIII` 是 `8`。
*   十位的字符 (`I`、 `X`、 `C` 和 `M`) 可以被重复最多三次。对于 `4`，你则需要利用下一个能够被 5 整除的字符进行减操作得到。你不能把 `4` 表示为`IIII`，而应该表示为`IV` (“比 `5` 小 `1` ”)。`40` 则被写作 `XL` (“比 `50` 小 `10`”)，`41` 表示为 `XLI`，`42` 表示为 `XLII`，`43` 表示为 `XLIII`， `44` 表示为 `XLIV` (“比 `50` 小 `10`，加上 `5` 小 `1`”)。
*   有时，字符串是……加法的对立面。通过将某些字符串放的其他一些之前，可以从最终值中相减。例如，对于 `9`，你需要从下一个最高十位字符串中减去一个值：`8` 是 `VIII`，但 `9` 是 `IX`（“ 比 `10` 小 `1`”)，而不是`VIIII` （由于 `I` 字符不能重复四次）。`90` 是 `XC`， `900` 是 `CM`。
*   表示 5 的字符不能重复。`10` 总是表示为 `X`，而决不能是 `VV`。 `100` 总是 `C`，决不能是 `LL`。
*   罗马数字从左向右读，因此字符的顺序非常重要。`DC` 是 `600`; `CD` 则是完全不同的数字 (`400`， “比 `500` 小 `100` ”)。 `CI` 是 `101`; `IC` 甚至不是合法的罗马数字（因为你不能直接从 `100` 减 `1`；你将不得不将它表示为 `XCIX`，“比 `100` 小`10` ，然后比 `10`” 小 `1`）。

因此，有用的测试将会确保 `from_roman()` 函数应当在传入太多重复数字时失败。“太多”是多少取决于数字。

```py
class FromRomanBadInput(unittest.TestCase):
    def test_too_many_repeated_numerals(self):
        '''from_roman should fail with too many repeated numerals'''
        for s in ('MMMM', 'DD', 'CCCC', 'LL', 'XXXX', 'VV', 'IIII'):
            self.assertRaises(roman6.InvalidRomanNumeralError, roman6.from_roman, s) 
```

另一有效测试是检查某些未被重复的模式。例如，`IX` 代表 `9`，但 `IXIX` 绝不会合法。

```py
 def test_repeated_pairs(self):
        '''from_roman should fail with repeated pairs of numerals'''
        for s in ('CMCM', 'CDCD', 'XCXC', 'XLXL', 'IXIX', 'IVIV'):
            self.assertRaises(roman6.InvalidRomanNumeralError, roman6.from_roman, s) 
```

第三个测试应当检测数字是否以正确顺序出现，从最高到最低位。例如，`CL` 是 `150`，而 `LC` 永远是非法的，因为代表 `50` 的数字永远不能在 `100` 数字之前出现。 该测试包括一个随机的可选项：`I` 在 `M` 之前， `V` 在 `X` 之前，等等。

```py
 def test_malformed_antecedents(self):
        '''from_roman should fail with malformed antecedents'''
        for s in ('IIMXCC', 'VX', 'DCM', 'CMM', 'IXIV',
                  'MCMC', 'XCX', 'IVI', 'LM', 'LD', 'LC'):
            self.assertRaises(roman6.InvalidRomanNumeralError, roman6.from_roman, s) 
```

这些测试中的每个都依赖于 `from_roman()` 引发一个新的例外 `InvalidRomanNumeralError`，而该例外尚未定义。

```py
# roman6.py
class InvalidRomanNumeralError(ValueError): pass 
```

所有的测试都应该是失败的，因为 `from_roman()` 方法还没有任何有效性检查。 （如果没有失败，它们在测什么呢？）

```py
you@localhost:~/diveintopython3/examples$ python3 romantest6.py
FFF.......
======================================================================
FAIL: test_malformed_antecedents (__main__.FromRomanBadInput)
from_roman should fail with malformed antecedents
----------------------------------------------------------------------
Traceback (most recent call last):
  File "romantest6.py", line 113, in test_malformed_antecedents
    self.assertRaises(roman6.InvalidRomanNumeralError, roman6.from_roman, s)
AssertionError: InvalidRomanNumeralError not raised by from_roman

======================================================================
FAIL: test_repeated_pairs (__main__.FromRomanBadInput)
from_roman should fail with repeated pairs of numerals
----------------------------------------------------------------------
Traceback (most recent call last):
  File "romantest6.py", line 107, in test_repeated_pairs
    self.assertRaises(roman6.InvalidRomanNumeralError, roman6.from_roman, s)
AssertionError: InvalidRomanNumeralError not raised by from_roman

======================================================================
FAIL: test_too_many_repeated_numerals (__main__.FromRomanBadInput)
from_roman should fail with too many repeated numerals
----------------------------------------------------------------------
Traceback (most recent call last):
  File "romantest6.py", line 102, in test_too_many_repeated_numerals
    self.assertRaises(roman6.InvalidRomanNumeralError, roman6.from_roman, s)
AssertionError: InvalidRomanNumeralError not raised by from_roman

----------------------------------------------------------------------
Ran 10 tests in 0.058s

FAILED (failures=3) 
```

好！现在，我们要做的所有事情就是添加正则表达式到 `from_roman()` 中以测试有效的罗马数字。

```py
roman_numeral_pattern = re.compile('''
    ^                   # beginning of string
    M{0,3}              # thousands - 0 to 3 Ms
    (CM|CD|D?C{0,3})    # hundreds - 900 (CM), 400 (CD), 0-300 (0 to 3 Cs),
                        #            or 500-800 (D, followed by 0 to 3 Cs)
    (XC|XL|L?X{0,3})    # tens - 90 (XC), 40 (XL), 0-30 (0 to 3 Xs),
                        #        or 50-80 (L, followed by 0 to 3 Xs)
    (IX|IV|V?I{0,3})    # ones - 9 (IX), 4 (IV), 0-3 (0 to 3 Is),
                        #        or 5-8 (V, followed by 0 to 3 Is)
    $                   # end of string
    ''', re.VERBOSE)

def from_roman(s):
    '''convert Roman numeral to integer'''
 <mark>if not roman_numeral_pattern.search(s):
        raise InvalidRomanNumeralError('Invalid Roman numeral: {0}'.format(s))</mark>

    result = 0
    index = 0
    for numeral, integer in roman_numeral_map:
        while s[index : index + len(numeral)] == numeral:
            result += integer
            index += len(numeral)
    return result 
```

再运行一遍测试……

```py
you@localhost:~/diveintopython3/examples$ python3 romantest7.py
..........
----------------------------------------------------------------------
Ran 10 tests in 0.066s

OK 
```

本年度的虎头蛇尾奖颁发给……单词“OK”，在所有测试通过时，它由 `unittest` 模块输出。