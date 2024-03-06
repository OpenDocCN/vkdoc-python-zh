# 第十三章 单元测试

# 第十三章 单元测试

*   13.1\. 罗马数字程序介绍 II
*   13.2\. 深入
*   13.3\. romantest.py 介绍
*   13.4\. 正面测试 (Testing for success)
*   13.5\. 负面测试 (Testing for failure)
*   13.6\. 完备性检测 (Testing for sanity)

# 13.1\. 罗马数字程序介绍 II

# 13.1\. 罗马数字程序介绍 II

在前面的章节中，通过阅读代码，你迅速“深入”，以最快的速度理解了各个程序。既然你已对 Python 有了一定的了解，那么接下来让我们看看程序开发*之前* 的工作。

在接下来的几章中，你将会编写、调试和优化一系列工具函数来进行罗马数字和阿拉伯数字之间的转换。你已从第 7.3 节 “个案研究：罗马字母”中获知构造和验证罗马数字的机制，现在我们要做的事是退后一步去思考如何将这些机制扩展到一个双向转换的工具。

罗马数字的规则有如下一些有趣的特点：

1.  一个特定数字以罗马数字表示时只有单一方式。
2.  反之亦然：一个有效的罗马数字表示的数也只对应一个阿拉伯数字表示。(也就是说转换成阿拉伯数字表示只有一种方法。)
3.  我们研究的是 `1` 和 `3999` 之间的数字的罗马数字表示。(罗马数字有很多方法用以记录更大的数，例如在数字上加线表示`1000`倍的数，但你不必去理会这些。就本章而言，我们姑且把罗马数字限定在 `1` 到 `3999` 之间)。
4.  罗马数字无法表示 `0`。(令人诧异，古罗马竟然没有 `0` 这个数字的概念。数字是为数数服务的，没有怎么数呢？)
5.  罗马数字不能表示负数。
6.  罗马数字无法表示分数和非整数。

基于如上所述，你将如何构造罗马数字转换函数呢？

## `roman.py` 功能需求

1.  `toRoman` 应该能返回 `1` 到 `3999` 中任意数的罗马数字表示。
2.  `toRoman` 在遇到 `1` 到 `3999` 之外的数字时应该失败。
3.  `toRoman` 在遇到非整数时应该失败。
4.  `fromRoman` 应该能将给定的有效罗马数字表示转换为阿拉伯数字表示。
5.  `fromRoman` 在遇到无效罗马数字表示时应该失败。
6.  将一个数转换为罗马数字表示，再转换回阿拉伯数字表示后应该和最初的数相同。因此，`fromRoman(toRoman(n)) == n` 对于 `1..3999` 之间所有 `n` 都适用。
7.  `toRoman` 返回的罗马数字应该使用大写字母。
8.  `fromRoman` 应该只接受大写罗马数字 (也就是说给定小写字母进行转换时应该失败)。

## 进一步阅读

*   这个站点 有关于罗马数字更多的内容，包括罗马人如何使用罗马数字的迷人 [历史](http://www.wilkiecollins.demon.co.uk/roman/intro.htm) (简言之：充满偶然性和反复无常)。

# 13.2\. 深入

# 13.2\. 深入

现在你已经定义了你的转换程序所应有的功能，下面一步会有点儿出乎你的意料：你将要开发一个测试组件 (test suite) 来测试你未来的函数以确保它们工作正常。没错：你将为还未开发的程序开发测试代码。

这就是所谓的单元测试，因为这两个转换函数可以被当作一个单元来开发和测试，不用考虑它们可能今后成为一个大程序的一部分。Python 有一个单元测试框架，被恰如其分地称作 `unittest` 模块。

> 注意
> Python 2.1 和之后的版本已经包含了 `unittest`。Python 2.0 用户则可以从 [`pyunit.sourceforge.net`](http://pyunit.sourceforge.net/)下载。

单元测试是以测试为核心开发策略的重要组成部分。如果你要写单元测试代码，尽早 (最好是在被测试代码开发之前) 开发并根据代码开发和需求的变化不断更新是很重要的。单元测试不能取代更高层面的功能和系统测试，但在开发的每个阶段都很重要：

*   代码开发之前，强迫你以有效的方式考虑需求的细节。
*   代码开发中，防止过度开发。通过了所有测试用例，程序的开发就完成了。
*   重构代码时，确保新版和旧版功能一致。
*   维护代码时，当你的代码更改导致别人代码出问题时帮你留住面子。(“但是*先生*，我检入 (check in) 代码时所有的单元测试都通过了……”)
*   在团队开发时，可以使你有信心，保证自己提交的代码不会破坏其他人的代码，因为你可以 先运行其他人的单元测试代码。(我在“代码风暴”中见过这种事情。一个团队将任务拆分，每个人都根据自己那部分的需求开发单元测试，然后与其他成员共享。没有人会出太大的偏差而导致代码无法集成。)

# 13.3\. `romantest.py` 介绍

# 13.3\. `romantest.py` 介绍

这是将被开发并保存为 `roman.py` 的罗马数字转换程序的完整测试组件 (test suite)。很难立刻看出它们是如何协同工作的，似乎所有类或者方法之间都没有关系。这是有原因的，而且你很快就会明了。

## 例 13.1\. `romantest.py`

如果您还没有下载本书附带的样例程序, 可以 [下载本程序和其他样例程序](http://www.woodpecker.org.cn/diveintopython/download/diveintopython-exampleszh-cn-5.4b.zip "Download example scripts")。

```py
"""Unit test for roman.py"""
import roman
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
                    (3999, 'MMMCMXCIX'))                       
    def testToRomanKnownValues(self):                          
        """toRoman should give known result with known input"""
        for integer, numeral in self.knownValues:              
            result = roman.toRoman(integer)                    
            self.assertEqual(numeral, result)                  
    def testFromRomanKnownValues(self):                          
        """fromRoman should give known result with known input"""
        for integer, numeral in self.knownValues:                
            result = roman.fromRoman(numeral)                    
            self.assertEqual(integer, result)                    
class ToRomanBadInput(unittest.TestCase):                            
    def testTooLarge(self):                                          
        """toRoman should fail with large input"""                   
        self.assertRaises(roman.OutOfRangeError, roman.toRoman, 4000)
    def testZero(self):                                              
        """toRoman should fail with 0 input"""                       
        self.assertRaises(roman.OutOfRangeError, roman.toRoman, 0)   
    def testNegative(self):                                          
        """toRoman should fail with negative input"""                
        self.assertRaises(roman.OutOfRangeError, roman.toRoman, -1)  
    def testNonInteger(self):                                        
        """toRoman should fail with non-integer input"""             
        self.assertRaises(roman.NotIntegerError, roman.toRoman, 0.5) 
class FromRomanBadInput(unittest.TestCase):                                      
    def testTooManyRepeatedNumerals(self):                                       
        """fromRoman should fail with too many repeated numerals"""              
        for s in ('MMMM', 'DD', 'CCCC', 'LL', 'XXXX', 'VV', 'IIII'):             
            self.assertRaises(roman.InvalidRomanNumeralError, roman.fromRoman, s)
    def testRepeatedPairs(self):                                                 
        """fromRoman should fail with repeated pairs of numerals"""              
        for s in ('CMCM', 'CDCD', 'XCXC', 'XLXL', 'IXIX', 'IVIV'):               
            self.assertRaises(roman.InvalidRomanNumeralError, roman.fromRoman, s)
    def testMalformedAntecedent(self):                                           
        """fromRoman should fail with malformed antecedents"""                   
        for s in ('IIMXCC', 'VX', 'DCM', 'CMM', 'IXIV',
                  'MCMC', 'XCX', 'IVI', 'LM', 'LD', 'LC'):                       
            self.assertRaises(roman.InvalidRomanNumeralError, roman.fromRoman, s)
class SanityCheck(unittest.TestCase):        
    def testSanity(self):                    
        """fromRoman(toRoman(n))==n for all n"""
        for integer in range(1, 4000):       
            numeral = roman.toRoman(integer) 
            result = roman.fromRoman(numeral)
            self.assertEqual(integer, result)
class CaseCheck(unittest.TestCase):                   
    def testToRomanCase(self):                        
        """toRoman should always return uppercase"""  
        for integer in range(1, 4000):                
            numeral = roman.toRoman(integer)          
            self.assertEqual(numeral, numeral.upper())
    def testFromRomanCase(self):                      
        """fromRoman should only accept uppercase input"""
        for integer in range(1, 4000):                
            numeral = roman.toRoman(integer)          
            roman.fromRoman(numeral.upper())          
            self.assertRaises(roman.InvalidRomanNumeralError,
                              roman.fromRoman, numeral.lower())
if __name__ == "__main__":
    unittest.main() 
```

## 进一步阅读

*   PyUnit 主页 对于使用 [`unittest` 框架](http://pyunit.sourceforge.net/pyunit.html) 以及本章没能涵盖的高级特性有深入的讨论。
*   PyUnit FAQ 解释了 [为什么测试用例要和被测试代码分开存放](http://pyunit.sourceforge.net/pyunit.html#WHERE) 。
*   *Python Library Reference* 总结了 [`unittest`](http://www.python.org/doc/current/lib/module-unittest.html) 模块。
*   ExtremeProgramming.org 讨论 [你为什么需要编写单元测试](http://www.extremeprogramming.org/rules/unittests.html)。
*   The Portland Pattern Repository 有一个持续的 [单元测试](http://www.c2.com/cgi/wiki?UnitTests) 讨论，包括了一个 [标准的定义](http://www.c2.com/cgi/wiki?StandardDefinitionOfUnitTest)，为什么你需要 [首先开发单元测试代码](http://www.c2.com/cgi/wiki?CodeUnitTestFirst) 以及另外一些深层次 [案例](http://www.c2.com/cgi/wiki?UnitTestTrial)。

# 13.4\. 正面测试 (Testing for success)

# 13.4\. 正面测试 (Testing for success)

单元测试的基础是构建独立的测试用例 (test case)。一个测试用例只回答一个关于被测试代码的问题。

一个测试用例应该做到：

*   完全独立运行，不需要人工输入。单元测试应该是自动的。
*   可以自己判断被测试函数是通过还是失败，不需要人工干预结果。
*   独立运行，可以与其他测试用例隔离 (尽管它们可能测试着同一个函数)。每个测试用例是一个孤岛。

基于如上原则，让我们构建第一个测试用例。应符合如下要求：

1.  `toRoman` 应该为所有 `1` 到 `3999` 的整数返回罗马数字表示。

## 例 13.2\. `testToRomanKnownValues`

```py
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
                    (3999, 'MMMCMXCIX'))                        
    def testToRomanKnownValues(self):                           
        """toRoman should give known result with known input"""
        for integer, numeral in self.knownValues:              
            result = roman.toRoman(integer)                      
            self.assertEqual(numeral, result) 
```

|  |  |
| --- | --- |
| [1] | 编写测试用例的第一步就是继承 `unittest` 模块中的 `TestCase` 类，它提供了很多可以用在你的测试用例中来测试特定情况的有用方法。 |
| [2] | 这是我手工转换的一个 integer/numeral 对列表。它包含了最小的十个数、最大的数、每个单字符罗马数字对应的数，以及其他随机挑选的有效数样本。单元测试的关键不在于所有可能的输入，而是一个有代表性的样本。 |
| [3] | 每个独立测试本身都是一个方法，既不需要参数也不返回任何值。如果该方法正常退出没有引发异常，测试被认为通过；如果测试引发异常，测试被认为失败。 |
| [4] | 这里你真正调用了 `toRoman` 函数。(当然，函数还没有编写，但一旦被编写，这里便是调用之处。) 注意你在这里为 `toRoman` 函数定义了 API ：它必须接受整数 (待转换的数) 并返回一个字符串 (对应的罗马数字表示)，如果 API 不是这样，测试将失败。 |
| [5] | 同样值得注意，你在调用 `toRoman` 时没有试图捕捉任何可能发生的异常。这正是我们所希望的。以有效输入调用 `toRoman` 不会引发任何异常，而你看到的这些输入都是有效的。如果 `toRoman` 引发了异常，则测试失败。 |
| [6] | 假设 `toRoman` 函数被正确编写，正确调用，运行成功并返回一个值，最后一步便是检查这个返回值*正确* 与否。这是一个常见的问题，`TestCase` 类提供了一个方法：`assertEqual`，来测试两个值是否相等。如果 `toRoman` 返回的结果 (`value`) 不等于我们预期的值 (`numeral`)，`assertEqual` 将会引发一个异常，测试也就此失败。如果两个值相等，`assertEqual` 什么也不做。如果每个从 `toRoman` 返回的值都等于预期值，`assertEqual` 便不会引发异常，于是 `testToRomanKnownValues` 最终正常退出，这意味着 `toRoman` 通过了该测试。 |

# 13.5\. 负面测试 (Testing for failure)

# 13.5\. 负面测试 (Testing for failure)

使用有效输入确保函数成功通过测试还不够，你还需要测试无效输入导致函数失败的情形。但并不是任何失败都可以，必须如你预期地失败。

还记得 `toRoman` 的其他要求吧：

1.  `toRoman` 在输入值为 `1` 到 `3999` 之外时失败。
2.  `toRoman` 在输入值为非整数时失败。

在 Python 中，函数以引发异常的方式表示失败。`unittest` 模块提供了用于测试函数是否在给定无效输入时引发特定异常的方法。

## 例 13.3\. 测试 `toRoman` 的无效输入

```py
 class ToRomanBadInput(unittest.TestCase):                            
    def testTooLarge(self):                                          
        """toRoman should fail with large input"""                   
        self.assertRaises(roman.OutOfRangeError, roman.toRoman, 4000) 
    def testZero(self):                                              
        """toRoman should fail with 0 input"""                       
        self.assertRaises(roman.OutOfRangeError, roman.toRoman, 0)    
    def testNegative(self):                                          
        """toRoman should fail with negative input"""                
        self.assertRaises(roman.OutOfRangeError, roman.toRoman, -1)  
    def testNonInteger(self):                                        
        """toRoman should fail with non-integer input"""             
        self.assertRaises(roman.NotIntegerError, roman.toRoman, 0.5) 
```

|  |  |
| --- | --- |
| [1] | `unittest` 模块中的 `TestCase` 类提供了 `assertRaises` 方法，它接受这几个参数：预期的异常、测试的函数，以及传递给函数的参数。(如果被测试函数有不止一个参数，把它们按顺序全部传递给 `assertRaises` ，它会把这些参数传给被测的函数。) 特别注意这里的操作：不是直接调用 `toRoman` 再手工查看是否引发特定异常 (使用 `try...except` 块捕捉异常)，`assertRaises` 为我们封装了这些。所有你要做的就是把异常 (`roman.OutOfRangeError`)、函数 (`toRoman`) 以及 `toRoman` 的参数 (`4000`) 传递给 `assertRaises` ，它会调用 `toRoman` 查看是否引发 `roman.OutOfRangeError` 异常。(还应注意到你是把 `toRoman` 函数本身当作一个参数，而不是调用它，传递它的时候也不是把它的名字作为一个字符串。我提到过吗？无论是函数还是异常， Python 中万物皆对象)。 |
| [2] | 与测试过大的数相伴的便是测试过小的数。记住，罗马数字不能表示 `0` 和负数，所以你要分别编写测试用例 ( `testZero` 和 `testNegative`)。在 `testZero` 中，你测试 `toRoman` 调用 `0` 引发的 `roman.OutOfRangeError` 异常，如果*没能* 引发 `roman.OutOfRangeError` (不论是返回了一个值还是引发了其他异常)，则测试失败。 |
| [3] | 要求 #3：`toRoman` 不能接受非整数输入，所以这里你测试 `toRoman` 在输入 `0.5` 时引发 `roman.NotIntegerError` 异常。如果 `toRoman` 没有引发 `roman.NotIntegerError` 异常，则测试失败。 |

接下来的两个要求与前三个类似，不同点是他们所针对的是 `fromRoman` 而不是 `toRoman`：

1.  `fromRoman` 应该能将输入的有效罗马数字转换为相应的阿拉伯数字表示。
2.  `fromRoman` 在输入无效罗马数字时应该失败。

要求 #4 与要求 #1 的处理方法相同，即测试一个已知样本中的一个个数字对。要求 #5 与 #2 和 #3 的处理方法相同，即通过无效输入确认 `fromRoman` 引发恰当的异常。

## 例 13.4\. 测试 `fromRoman` 的无效输入

```py
 class FromRomanBadInput(unittest.TestCase):                                      
    def testTooManyRepeatedNumerals(self):                                       
        """fromRoman should fail with too many repeated numerals"""              
        for s in ('MMMM', 'DD', 'CCCC', 'LL', 'XXXX', 'VV', 'IIII'):             
            self.assertRaises(roman.InvalidRomanNumeralError, roman.fromRoman, s) 
    def testRepeatedPairs(self):                                                 
        """fromRoman should fail with repeated pairs of numerals"""              
        for s in ('CMCM', 'CDCD', 'XCXC', 'XLXL', 'IXIX', 'IVIV'):               
            self.assertRaises(roman.InvalidRomanNumeralError, roman.fromRoman, s)
    def testMalformedAntecedent(self):                                           
        """fromRoman should fail with malformed antecedents"""                   
        for s in ('IIMXCC', 'VX', 'DCM', 'CMM', 'IXIV',
                  'MCMC', 'XCX', 'IVI', 'LM', 'LD', 'LC'):                       
            self.assertRaises(roman.InvalidRomanNumeralError, roman.fromRoman, s) 
```

|  |  |
| --- | --- |
| [1] | 没什么新鲜的，与测试 `toRoman` 无效输入时相同的模式，只是你有了一个新的异常：`roman.InvalidRomanNumeralError`。`roman.py` 中一共要定义三个异常 (另外的两个是 `roman.OutOfRangeError` 和 `roman.NotIntegerError`)。稍后你在开始编写 `roman.py` 时将会知道如何定义这些异常。 |

# 13.6\. 完备性检测 (Testing for sanity)

# 13.6\. 完备性检测 (Testing for sanity)

你经常会发现一组代码中包含互逆的转换函数，一个把 A 转换为 B ，另一个把 B 转换为 A。在这种情况下，创建“完备性检测”可以使你在由 A 转 B 再转 A 的过程中不会出现丢失精度或取整等错误。

考虑这个要求：

1.  如果你给定一个数，把它转化为罗马数字表示，然后再转换回阿拉伯数字表示，你所得到的应该是最初给定的那个数。因此，对于 `1..3999` 中的`n`，`fromRoman(toRoman(n)) == n` 总成立。

## 例 13.5\. 以 `toRoman` 测试 `fromRoman` 的输出

```py
 class SanityCheck(unittest.TestCase):        
    def testSanity(self):                    
        """fromRoman(toRoman(n))==n for all n"""
        for integer in range(1, 4000):         
            numeral = roman.toRoman(integer) 
            result = roman.fromRoman(numeral)
            self.assertEqual(integer, result) 
```

|  |  |
| --- | --- |
| [1] | 你已经见到过 `range` 函数，但这里它以两个参数被调用，返回了从第一个参数 (`1`) 开始到*但不包括* 第二个参数 (`4000`) 的整数列表。因此，`1..3999` 就是准备转换为罗马数字表示的有效值列表。 |
| [2] | 我想提一下，这里的 `integer` 并不是一个 Python 关键字，而只是没有什么特别的变量名。 |
| [3] | 这里的测试逻辑显而易见：把一个数 (`integer`) 转换为罗马数字表示的数 (`numeral`)，然后再转换回来 (`result`) 并确保最后的结果和最初的数是同一个数。如果不是，`assertEqual` 便会引发异常，测试也便立刻失败。如果所有的结果都和初始数一致，`assertEqual` 将会保持沉默，整个 `testSanity` 方法将会最终也保持沉默，测试则将会被认定为通过。 |

最后两个要求和其他的要求不同，似乎既武断而又微不足道：

1.  `toRoman` 返回的罗马数字应该使用大写字母。
2.  `fromRoman` 应该只接受大写罗马数字 (也就是说给定小写字母进行转换时应该失败)。

事实上，它们确实有点武断，譬如你完全可以让 `fromRoman` 接受小写和大小写混合的输入；但他们也不是完全武断；如果 `toRoman` 总是返回大写的输出，那么 `fromRoman` 至少应该接受大写字母输入，不然 “完备性检测” (要求 #6) 就会失败。不管怎么说，*只* 接受大写输入还是武断的，但就像每个系统都会告诉你的那样，大小写总会出问题，因此事先规定这一点还是有必要的。既然有必要规定，那么也就有必要测试。

## 例 13.6\. 大小写测试

```py
 class CaseCheck(unittest.TestCase):                   
    def testToRomanCase(self):                        
        """toRoman should always return uppercase"""  
        for integer in range(1, 4000):                
            numeral = roman.toRoman(integer)          
            self.assertEqual(numeral, numeral.upper())         
    def testFromRomanCase(self):                      
        """fromRoman should only accept uppercase input"""
        for integer in range(1, 4000):                
            numeral = roman.toRoman(integer)          
            roman.fromRoman(numeral.upper())                    
            self.assertRaises(roman.InvalidRomanNumeralError,
                              roman.fromRoman, numeral.lower()) 
```

|  |  |
| --- | --- |
| [1] | 关于这个测试用例最有趣的一点不在于它测试了什么，而是它不测试什么。它不会测试 `toRoman` 的返回值是否正确或者一致；这些问题由其他测试用例来回答。整个测试用例仅仅测试大写问题。你也许觉得应该将它并入到完备性测试，毕竟都要遍历整个输入值范围并调用 `toRoman`。[11]但是这样将会违背一条基本规则")：每个测试用例只回答一个的问题。试想一下，你将这个测试并入到完备性测试中，然后遇到了测试失败。你还需要进一步分析以便判定测试用例的哪部分出了问题。如果你需要分析方能找出问题所在，无疑你的测试用例在设计上出了问题。 |
| [2] | 这有一个和前面相似的情况：尽管 “你知道” `toRoman` 总是返回大写字母，你还是需要把返回值显式地转换成大写字母后再传递给只接受大写的 `fromRoman` 进行测试。为什么？因为 `toRoman` 只返回大写字母是一个独立的需求。如果你改变了这个需求，例如改成总是返回小写字母，那么 `testToRomanCase` 测试用例也应作出调整，但这个测试用例应该仍能通过。这是另外一个基本规则")：每个测试用例必须可以与其他测试用例隔离工作，每个测试用例是一个“孤岛”。 |
| [3] | 注意你并没有使用 `fromRoman` 的返回值。这是一个有效的 Python 语法：如果一个函数返回一个值，但没有被使用，Python 会直接把这个返回值扔掉。这正是你所希望的，这个测试用例并不对返回值进行测试，只是测试 `fromRoman` 接受大写字母而不引发异常。 |
| [4] | 这行有点复杂，但是它与 `ToRomanBadInput` 和 `FromRomanBadInput` 测试很相似。 你在测试以特定值 (`numeral.lower()`，循环中目前罗马数字的小写版) 调用特定函数 (`roman.fromRoman`) 会确实引发特定的异常 (`roman.InvalidRomanNumeralError`)。如果 (在循环中的每一次) 确实如此，测试通过；如果有一次不是这样 (比如引发另外的异常或者不引发异常)，测试失败。 |

在下一章中，你将看到如何编写可以通过这些测试的代码。

## Footnotes

[11] “除了诱惑什么我都能抗拒。 (I can resist everything except temptation.)”――Oscar Wilde