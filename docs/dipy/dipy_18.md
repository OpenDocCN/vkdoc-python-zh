# 第十七章 动态函数

# 第十七章 动态函数

*   17.1\. 概览
*   17.2\. plural.py, 第 1 阶段
*   17.3\. plural.py, 第 2 阶段
*   17.4\. plural.py, 第 3 阶段
*   17.5\. plural.py, 第 4 阶段
*   17.6\. plural.py, 第 5 阶段
*   17.7\. plural.py, 第 6 阶段
*   17.8\. 小结

# 17.1\. 概览

# 17.1\. 概览

我想谈谈名词复数。还有，返回其它函数的函数，高级的正则表达式和生成器 (Generator)。生成器是 Python 2.3 新引入的。但首先还是让我们先来谈谈如何生成名词复数。

如果你还没有看过 第七章 *正则表达式*，现在是个绝佳的机会。这章中假定你已理解了正则表达式的基础内容并迅速深入更高级的应用。

英语是一个吸收很多外来语而令人疯掉的语言，把单数名词变成复数的规则则是复杂而又多变的。有规则，有例外，更有例外的例外。

如果你在英语国家长大或是在正规学校学习了英语，你可能对下面的基本规则很熟悉：

1.  如果一个词以 S, X 或 Z 结尾，加 ES。如 “Bass” 变成 “basses”，“fax” 变成 “faxes”，还有 “waltz” 变成 “waltzes”。
2.  如果一个词以发音的 H 结尾，加 ES；若以不发音的 H 结尾，加 S。什么是发音的 H？和其他字母混合在一起发出一个你可以听到的声音。那么，“coach” 变成 “coaches” ，“rash” 变成 “rashes”，因为在读出来时，你可以听到 CH 和 SH 的声音。但是，“cheetah” 变成 “cheetahs”，因为 H 不发音。
3.  如果一个词以发 I 音的 Y 结尾，把 Y 变成 IES；如果 Y 与元音搭配在一起发出其他声音则只添加 S。因此，“vacancy” 变成 “vacancies”，但 “day” 变成 “days”。
4.  如果一切规则都不适用，就只添加 S 并祈祷不会错。

(我知道有很多例外情况，比如：“Man” 变成 “men”，“woman” 变成 “women”，但是，“human” 却变成 “humans”。“Mouse” 变成 “mice”，“louse” 变成 “lice”，但是，“house” 却变成 “houses”。“Knife” 变成 “knives”，“wife” 变成 “wives”，但是 “lowlife” 却变成 “lowlifes”。更不要说那些复数根本就不需要变化的词了，比如 “sheep”, “deer” 和 “haiku”。)

其他的语言当然完全不同。

让我们来设计一个复数化名词的模块吧！从英语名词开始，仅考虑上面的四种规则，但是记得你将来需要不断添加规则，更可能最后添加进更多的语言。

# 17.2\. `plural.py`, 第 1 阶段

# 17.2\. `plural.py`, 第 1 阶段

你所针对的单词 (至少在英语中) 是字符串和字符。你还需要规则来找出不同的字符 (字母) 组合，并对它们进行不同的操作。这听起来像是正则表达式的工作。

## 例 17.1\. `plural1.py`

```py
 import re
def plural(noun):                            
    if re.search('[sxz]$', noun):             
        return re.sub('$', 'es', noun)        
    elif re.search('[^aeioudgkprt]h$', noun):
        return re.sub('$', 'es', noun)       
    elif re.search('[^aeiou]y$', noun):      
        return re.sub('y$', 'ies', noun)     
    else:                                    
        return noun + 's' 
```

|  |  |
| --- | --- |
| [1] | 好啦，这是一个正则表达式，但是它使用了你在 第七章 *正则表达式* 中未曾见过的语法。方括号的意思是 “完全匹配这些字符中的一个”。也就是说，`[sxz]` 意味着 “`s`，或者 `x`，再或者 `z`”，但只是其中的一个。`$` 应该不陌生，它意味着匹配字符串的结尾。也就是说，检查 `noun` 是否以 `s`，`x`，或者 `z` 结尾。 |
| [2] | `re.sub` 函数进行以正则表达式为基础的替换工作。让我们更具体地看看它。 |

## 例 17.2\. `re.sub` 介绍

```py
>>> import re
>>> re.search('[abc]', 'Mark')   
<_sre.SRE_Match object at 0x001C1FA8>
>>> re.sub('[abc]', 'o', 'Mark') 
'Mork'
>>> re.sub('[abc]', 'o', 'rock') 
'rook'
>>> re.sub('[abc]', 'o', 'caps') 
'oops' 
```

|  |  |
| --- | --- |
| [1] | `Mark` 包含 `a`，`b`，或者 `c`吗？是的，含有 `a`。 |
| [2] | 好的，现在找出 `a`，`b`，或者 `c` 并以 `o` 取代之。`Mark` 就变成 `Mork` 了。 |
| [3] | 同一方法可以将 `rock` 变成 `rook`。 |
| [4] | 你可能认为它可以将 `caps` 变成 `oaps`，但事实并非如此。`re.sub` 替换*所有* 的匹配项，并不只是第一个匹配项。因此正则表达式将会把 `caps` 变成 `oops`，因为 `c` 和 `a` 都被转换为 `o`了。 |

## 例 17.3\. 回到 `plural1.py`

```py
 import re
def plural(noun):                            
    if re.search('[sxz]$', noun):            
        return re.sub('$', 'es', noun)        
    elif re.search('[^aeioudgkprt]h$', noun): 
        return re.sub('$', 'es', noun)        
    elif re.search('[^aeiou]y$', noun):      
        return re.sub('y$', 'ies', noun)     
    else:                                    
        return noun + 's' 
```

|  |  |
| --- | --- |
| [1] | 回到 `plural` 函数。你在做什么？你在以 `es` 取代字符串的结尾。换句话说，追加 `es` 到字符串。你可以通过字符串拼合做到相同的事，例如 `noun + 'es'`，但是我使用正则表达式做这一切，既是为了保持一致，也是为了本章稍后你会明白的其它原因。 |
| [2] | 仔细看看，这是另一个新的内容。`^` 是方括号里面的第一个字符，这有特别的含义：否定。`[^abc]` 意味着 “ 除 `a`、 `b`、 和 `c` *以外的* 任意单字符”。所以，`[^aeioudgkprt]` 意味着除 `a`、 `e`、 `i`、 `o`、 `u`、 `d`、 `g`、 `k`、 `p`、 `r` 和 `t` 以外的任意字符。这个字符之后应该跟着一个 `h`，然后是字符串的结尾。你在寻找的是以发音的 H 结尾的单词。 |
| [3] | 这是一个相似的表达：匹配 Y 前面*不是* `a`、 `e`、 `i`、 `o` 和 `u`，并以这个 Y 结尾的单词。你在查找的是以发 I 音的 Y 结尾的单词。 |

## 例 17.4\. 正则表达式中否定的更多应用

```py
>>> import re
>>> re.search('[^aeiou]y$', 'vacancy') 
<_sre.SRE_Match object at 0x001C1FA8>
>>> re.search('[^aeiou]y$', 'boy')     
>>> 
>>> re.search('[^aeiou]y$', 'day')
>>> 
>>> re.search('[^aeiou]y$', 'pita')    
>>> 
```

|  |  |
| --- | --- |
| [1] | `vacancy` 匹配这个正则表达式，因为它以 `cy` 结尾，并且 `c` 不在 `a`、 `e`、 `i`、 `o` 和 `u` 之列。 |
| [2] | `boy` 不能匹配，因为它以 `oy` 结尾，并且你特别指出 `y` 之前的字符不可以是 `o`。`day` 不能匹配是因为以 `ay` 结尾。 |
| [3] | `pita` 不匹配是因为不以 `y` 结尾。 |

## 例 17.5\. 更多的 `re.sub`

```py
>>> re.sub('y$', 'ies', 'vacancy')              
'vacancies'
>>> re.sub('y$', 'ies', 'agency')
'agencies'
>>> re.sub('([^aeiou])y$', r'\1ies', 'vacancy') 
'vacancies' 
```

|  |  |
| --- | --- |
| [1] | 正则表达式把 `vacancy` 变为 `vacancies`，把 `agency` 变为 `agencies`，这正是你想要的。注意，将 `boy` 变成 `boies` 是可行的，但是永远不会发生，因为 `re.search` 首先确定是否应该应用 `re.sub`。 |
| [2] | 顺便提一下，可以将两个正则表达式 (一个确定规则适用与否，一个应用规则) 合并在一起成为一个正则表达式。这便是合并后的样子。它的大部分已经很熟悉：你应用的是在 第 7.6 节 “个案研究：解析电话号码” 学过的记忆组 (remembered group) 记住 `y` 之前的字符。然后再替换字符串，你使用一个新的语法 `\1`，这意味着：“嘿！记得前面的第一个组吗？把它放这儿”。就此而言，记住了 `y` 之前的 `c` ，然后你做替换工作，你将 `c` 替换到 `c` 的位置，并将 `ies` 替换到 `y` 的位置。(如果你有不止一个组则可以使用 `\2` 或者 `\3` 等等。) |

正则表达式替换非常强大，并且 `\1` 语法使之更加强大。但是将整个操作放在一个正则表达式中仍然晦涩难懂，也不能与前面描述的复数规则直接呼应。你原来列出的规则，比如 “如果单词以 S，X 或者 Z 结尾，结尾追加 ES”。如果你在函数中看到两行代码描述 “如果单词以 S，X 或者 Z 结尾，结尾追加 ES”，更加直观些。

# 17.3\. `plural.py`, 第 2 阶段

# 17.3\. `plural.py`, 第 2 阶段

现在你将增加一个抽象过程。你从定义一个规则列表开始：如果这样，就做那个，否则判断下一规则。让我们暂时将程序一部分复杂化以便使另一部分简单化。

## 例 17.6\. `plural2.py`

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
def match_y(noun):                            
    return re.search('[^aeiou]y$', noun)      
def apply_y(noun):                            
    return re.sub('y$', 'ies', noun)          
def match_default(noun):                      
    return 1                                  
def apply_default(noun):                      
    return noun + 's'                         
rules = ((match_sxz, apply_sxz),
         (match_h, apply_h),
         (match_y, apply_y),
         (match_default, apply_default)
         )                                     
def plural(noun):                             
    for matchesRule, applyRule in rules:       
        if matchesRule(noun):                  
            return applyRule(noun) 
```

|  |  |
| --- | --- |
| [1] | 这个版本看起来更加复杂 (至少是长了)，但做的工作没有变化：试图顺序匹配四种不同规则，并在匹配时应用恰当的正则表达式。不同之处在于，每个独立的匹配和应用规则都在自己的函数中定义，并且这些函数列于 `rules` 变量这个元组的元组之中。 |
| [2] | 使用一个 `for` 循环，你可以根据 `rules` 元组一次性进行匹配和应用规则两项工作 (一个匹配和一个应用)。`for` 循环第一轮中，`matchesRule` 将使用 `match_sxz`，`applyRule` 将使用 `apply_sxz`；在第二轮中 (假设真走到了这么远)，`matchesRule` 将被赋予 `match_h`，`applyRule` 将被赋予 `apply_h`。 |
| [3] | 记住 Python 中的一切都是对象，包括函数。`rules` 包含函数；不是指函数名，而是指函数本身。当 `matchesRule` 和 `applyRule` 在 `for` 循环中被赋值后，它们就成了你可以调用的真正函数。因此，在 `for` 循环第一轮中，这就相当于调用 `matches_sxz(noun)`。 |
| [4] | 在 `for` 循环第一轮中，这就相当于调用 `apply_sxz(noun)`，等等。 |

这个抽象过程有些令人迷惑，试着剖析函数看看实际的等价内容。这个 `for` 循环相当于：

## 例 17.7\. 剖析 `plural` 函数

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

这里的好处在于 `plural` 函数现在被简化了。它以普通的方法反复使用其它地方定义的规则。获得一个匹配规则，匹配吗？调用并应用规则。规则可以在任意地方以任意方法定义，`plural` 函数对此并不关心。

现在，添加这个抽象过程值得吗？嗯……还不值。让我们看看如何向函数添加一个新的规则。啊哈，在先前的范例中，需要向 `plural` 函数添加一个 `if` 语句；在这个例子中，需要增加两个函数：`match_foo` 和 `apply_foo`，然后更新 `rules` 列表指定在什么相对位置调用这个新匹配和新规则应用。

这其实不过是步入下一节的一个基石。让我们继续。

# 17.4\. `plural.py`, 第 3 阶段

# 17.4\. `plural.py`, 第 3 阶段

将每个匹配和规则应用分别制作成函数没有必要。你从来不会直接调用它们：你把它们定义于 `rules` 列表之中并从那里调用它们。让我们隐去它们的函数名而抓住规则定义的主线。

## 例 17.8\. `plural3.py`

```py
 import re
rules = \
  (
    (
     lambda word: re.search('[sxz]$', word),
     lambda word: re.sub('$', 'es', word)
    ),
    (
     lambda word: re.search('[^aeioudgkprt]h$', word),
     lambda word: re.sub('$', 'es', word)
    ),
    (
     lambda word: re.search('[^aeiou]y$', word),
     lambda word: re.sub('y$', 'ies', word)
    ),
    (
     lambda word: re.search('$', word),
     lambda word: re.sub('$', 's', word)
    )
   )                                           
def plural(noun):                             
    for matchesRule, applyRule in rules:       
        if matchesRule(noun):                 
            return applyRule(noun) 
```

|  |  |
| --- | --- |
| [1] | 这与第 2 阶段定义的规则是一样的。惟一的区别是不再定义 `match_sxz` 和 `apply_sxz` 之类的函数，而是以 lambda 函数 法将这些函数的内容直接 “嵌入” `rules` 列表本身。 |
| [2] | 注意 `plural` 函数完全没有变化，还是反复于一系列的规则函数，检查第一个匹配规则，如果返回真则调用第二个应用规则并返回值。和前面一样，给定单词返回单词。唯一的区别是规则函数被内嵌定义，化名作 lambda 函数。但是 `plural` 函数并不在乎它们是如何定义的，只是拿到规则列表，闭着眼睛干活。 |

现在添加一条新的规则，所有你要做的就是直接在 `rules` 列表之中定义函数：一个匹配规则，一个应用规则。这样内嵌的规则函数定义方法使得没必要的重复很容易被发现。你有四对函数，它们采用相同的模式。匹配函数就是调用 `re.search`，应用函数就是调用 `re.sub`。让我们提炼出这些共同点。

# 17.5\. `plural.py`, 第 4 阶段

# 17.5\. `plural.py`, 第 4 阶段

让我们精炼出代码中的重复之处，以便更容易地定义新规则。

## 例 17.9\. `plural4.py`

```py
 import re
def buildMatchAndApplyFunctions((pattern, search, replace)):  
    matchFunction = lambda word: re.search(pattern, word)      
    applyFunction = lambda word: re.sub(search, replace, word) 
    return (matchFunction, applyFunction) 
```

|  |  |
| --- | --- |
| [1] | `buildMatchAndApplyFunctions` 是一个动态生成其它函数的函数。它将 `pattern`，`search` 和 `replace` (实际上是一个元组，我们很快就会提到这一点)，通过使用 `lambda` 语法构建一个接受单参数 (`word`) 并以传递给 `buildMatchAndApplyFunctions` 的 `pattern` 和传递给新函数的 `word` 调用 `re.search` 的匹配函数！哇塞！ |
| [2] | 构建应用规则函数的方法相同。应用规则函数是一个接受单参数并以传递给 `buildMatchAndApplyFunctions` 的 `search` 和 `replace` 以及传递给这个应用规则函数的 `word` 调用 `re.sub` 的函数。在一个动态函数中应用外部参数值的技术被称作*闭合 (closures)*。你实际上是在应用规则函数中定义常量：它只接受一个参数 (`word`)，但用到了定义时设置的两个值 (`search` 和 `replace`)。 |
| [3] | 最终，`buildMatchAndApplyFunctions` 函数返回一个包含两个值的元组：你刚刚创建的两个函数。你在这些函数中定义的常量 (`matchFunction` 中的 `pattern` 以及 `applyFunction` 中的 `search` 和 `replace`) 保留在这些函数中，由 `buildMatchAndApplyFunctions` 一同返回。这简直太酷了。 |

如果这太费解 (它应该是这样，这是个怪异的东西)，可能需要通过了解它的使用来搞明白。

## 例 17.10\. `plural4.py` 继续

```py
patterns = \
  (
    ('[sxz]$', '$', 'es'),
    ('[^aeioudgkprt]h$', '$', 'es'),
    ('(qu|[^aeiou])y$', 'y$', 'ies'),
    ('$', '$', 's')
  )                                                 
rules = map(buildMatchAndApplyFunctions, patterns) 
```

|  |  |
| --- | --- |
| [1] | 我们的复数化规则现在被定义成一组字符串 (不是函数)。第一个字符串是你在调用 `re.search` 时使用的正则表达式；第二个和第三个字符串是你在通过调用 `re.sub` 来应用规则将名词变为复数时使用的搜索和替换表达式。 |
| [2] | 这很神奇。把传进去的 `patterns` 字符串转换为传回来的函数。如何做到的呢？将这些字符串映射给 `buildMatchAndApplyFunctions` 函数之后，三个字符串参数转换成了两个函数组成的元组。这意味着 `rules` 被转换成了前面范例中相同的内容：由许多调用 `re.search` 函数的匹配函数和调用 `re.sub` 的规则应用函数构成的函数组组成的一个元组。 |

我发誓这不是我信口雌黄：`rules` 被转换成了前面范例中相同的内容。剖析 `rules` 的定义，你看到的是：

## 例 17.11\. 剖析规则定义

```py
rules = \
  (
    (
     lambda word: re.search('[sxz]$', word),
     lambda word: re.sub('$', 'es', word)
    ),
    (
     lambda word: re.search('[^aeioudgkprt]h$', word),
     lambda word: re.sub('$', 'es', word)
    ),
    (
     lambda word: re.search('[^aeiou]y$', word),
     lambda word: re.sub('y$', 'ies', word)
    ),
    (
     lambda word: re.search('$', word),
     lambda word: re.sub('$', 's', word)
    )
   ) 
```

## 例 17.12\. `plural4.py` 的完成

```py
 def plural(noun):                                  
    for matchesRule, applyRule in rules:            
        if matchesRule(noun):                      
            return applyRule(noun) 
```

|  |  |
| --- | --- |
| [1] | 由于 `rules` 列表和前面的范例是相同的，`plural` 函数没有变化也就不令人诧异了。记住，这没什么特别的，按照顺序调用一系列函数。不必在意规则是如何定义的。在第 2 阶段，它们被定义为各具名称的函数。在第 3 阶段，他们被定义为匿名的 `lambda` 函数。现在第 4 阶段，它们通过 `buildMatchAndApplyFunctions` 映射原始的字符串列表被动态创建。无所谓，`plural` 函数的工作方法没有变。 |

还不够兴奋吧！我必须承认，在定义 `buildMatchAndApplyFunctions` 时我跳过了一个微妙之处。让我们回过头再看一下。

## 例 17.13\. 回头看 `buildMatchAndApplyFunctions`

```py
 def buildMatchAndApplyFunctions((pattern, search, replace)): 
```

|  |  |
| --- | --- |
| [1] | 注意到双括号了吗？这个函数并不是真的接受三个参数，实际上只接受一个参数：一个三元素元组。但是在函数被调用时元组被展开了，元组的三个元素也被赋予了不同的变量：`pattern`, `search` 和 `replace`。乱吗？让我们在使用中理解。 |

## 例 17.14\. 调用函数时展开元组

```py
>>> def foo((a, b, c)):
... print c
... print b
... print a
>>> parameters = ('apple', 'bear', 'catnap')
>>> foo(parameters) 
catnap
bear
apple 
```

|  |  |
| --- | --- |
| [1] | 调用 `foo` 的正确方法是使用一个三元素元组。函数被调用时，元素被分别赋予 `foo` 中的多个局部变量。 |

现在，让我们回过头看一看这个元组自动展开技巧的必要性。`patterns` 是一个元组列表，并且每个元组都有三个元素。调用 `map(buildMatchAndApplyFunctions, patterns)`，这并*不* 意味着是以三个参数调用 `buildMatchAndApplyFunctions`。使用 `map` 映射一个列表到函数时，通常使用单参数：列表中的每个元素。就 `patterns` 而言，列表的每个元素都是一个元组，所以 `buildMatchAndApplyFunctions` 总是是以元组来调用，在 `buildMatchAndApplyFunctions` 中使用元组自动展开技巧将元素赋值给可以被使用的变量。

# 17.6\. `plural.py`, 第 5 阶段

# 17.6\. `plural.py`, 第 5 阶段

你已经精炼了所有重复代码，也尽可能地把复数规则提炼到定义一个字符串列表。接下来的步骤是把这些字符串提出来放在另外的文件中，从而可以和使用它们的代码分开来维护。

首先，让我们建立一个包含你需要的所有规则的文本文件。没有什么特别的结构，不过是以空格 (或者制表符) 把字符串列成三列。你把它命名为 `rules.en`，“en” 是英语的意思。这些是英语名词复数的规则，你以后可以为其它语言添加规则文件。

## 例 17.15\. `rules.en`

```py
[sxz]$                  $               es
[^aeioudgkprt]h$        $               es
[^aeiou]y$              y$              ies
$                       $               s 
```

现在来看看如何使用规则文件。

## 例 17.16\. `plural5.py`

```py
 import re
import string                                                                     
def buildRule((pattern, search, replace)):                                        
    return lambda word: re.search(pattern, word) and re.sub(search, replace, word) 
def plural(noun, language='en'):                             
    lines = file('rules.%s' % language).readlines()          
    patterns = map(string.split, lines)                      
    rules = map(buildRule, patterns)                         
    for rule in rules:                                      
        result = rule(noun)                                  
        if result: return result 
```

|  |  |
| --- | --- |
| [1] | 在这里你还将使用闭合技术 (动态构建函数时使用函数外部定义的变量)，但是现在你把原来分开的匹配函数和规则应用函数合二为一 (你将在下一节中明了其原因)。你很快会看到，这与分别调用两个函数效果相同，只是调用的方法稍有不同。 |
| [2] | 咱们的 `plural` 函数现在接受的第二个参数是默认值为 `en` 的可选参数 `language`。 |
| [3] | 你使用 `language` 参数命名一个文件，打开这个文件并读取其中的内容到一个列表。如果 `language` 是 `en`，那么你将打开 `rules.en` 文件，读取全部内容，以其中的回车符作为分隔构建一个列表。文件的每一行将成为列表的一个元素。 |
| [4] | 如你所见，文件的每一行都有三个值，但是它们是以空白字符 (制表符或者空格符，这没什么区别) 分割。用 `string.split` 函数映射列表来创建一个每个元素都是三元素元组的新列表。因此，像 `[sxz]$ $ es` 这样的一行将被打碎并放入 `('[sxz]$', '$', 'es')` 这样的元组。这意味着 `patterns` 将最终变成元组列表的形式，就像第 4 阶段实打实编写的那样。 |
| [5] | 如果 `patterns` 是一个元组列表，那么 `rules` 就可以通过一个个调用 `buildRule` 动态地生成函数列表。调用 `buildRule(('[sxz]$', '$', 'es'))` 返回一个接受单参数 `word` 的函数。当返回的函数被调用，则将执行 `re.search('[sxz]$', word) and re.sub('$', 'es', word)`。 |
| [6] | 因为你现在构建的是一个匹配和规则应用合一的函数，你需要分别调用它们。仅仅是调用函数，如果返回了内容，那么返回的便是复数；如果没有返回 (也就是返回了`None`)，那么该规则未能匹配，就应该尝试其他规则。 |

这里的进步是你把复数规则完全分离到另外的文件中。不但这个文件可以独立于代码单独维护，而且你建立了一个命名规划使 `plural` 函数可以根据 `language` 参数使用不同的规则文件。

这里的缺陷是每次调用 `plural` 函数都需要去读取一次文件。我想我可以在整本书中都不使用 “留给读者去练习”，但是这里：为特定的语言规则文件建立一个缓存机制，并在调用期间规则文件改变时自动刷新*留给读者作为练习*。祝你顺利。

# 17.7\. `plural.py`, 第 6 阶段

# 17.7\. `plural.py`, 第 6 阶段

现在你已准备好探讨生成器 (Generator) 了。

## 例 17.17\. `plural6.py`

```py
 import re
def rules(language):                                                                 
    for line in file('rules.%s' % language):                                         
        pattern, search, replace = line.split()                                      
        yield lambda word: re.search(pattern, word) and re.sub(search, replace, word)
def plural(noun, language='en'):      
    for applyRule in rules(language): 
        result = applyRule(noun)      
        if result: return result 
```

这里使用了被称作生成器的技术，我不打算在你看过一个简单例子之前试图解释它。

## 例 17.18\. 介绍生成器

```py
>>> def make_counter(x):
... print 'entering make_counter'
... while 1:
...         yield x               
...         print 'incrementing x'
...         x = x + 1
... 
>>> counter = make_counter(2) 
>>> counter                   
<generator object at 0x001C9C10>
>>> counter.next()            
entering make_counter
2
>>> counter.next()            
incrementing x
3
>>> counter.next()            
incrementing x
4 
```

|  |  |
| --- | --- |
| [1] | `make_counter` 中出现关键字 `yield` 意味着这不是一个普通的函数。它是一种每次生成一个值的特殊函数。你可以把它看成是一个可恢复函数。调用它会返回一个生成器，它可以返回 `x` 的连续值。 |
| [2] | 想要创建一个 `make_counter` 生成器的实例，只要像其它函数一样调用。注意这并没有真正执行函数代码。你可以分辨出这一点，因为 `make_counter` 的第一行是 `print` 语句，然而没有任何内容输出。 |
| [3] | `make_counter` 函数返回一个生成器对象。 |
| [4] | 你第一次调用生成器对象的 `next()` 方法，将执行 `make_counter` 中的代码执行到第一个 `yield` 语句，然后返回生产 (yield) 出来的值。在本例中，这个值是 `2`，因为你是通过 `make_counter(2)` 来创建最初的生成器的。 |
| [5] | 不断调用生成器对象的 `next()` *将从你上次离开的位置重新开始* 并继续下去直到你又一次遇到 `yield` 语句。接下来执行 `print` 语句来打印 `incrementing x`，然后执行 `x = x + 1` 语句来真正地增加。然后你进入 `while` 的又一次循环，你所做的第一件事是 `yield x`，返回目前的 `x` 值 (现在是 3)。 |
| [6] | 第二次你调用 `counter.next()` 时，你又做一遍相同的事情，但是这次 `x` 是 `4`。如此继续。因为 `make_counter` 设置的是一个无限循环，理论上你可以永远这样继续下去，不断地递增并弹出 `x` 值。现在让我们看看生成器更具意义的应用。 |

## 例 17.19\. 使用生成器替代递归

```py
 def fibonacci(max):
    a, b = 0, 1       
    while a < max:
        yield a       
        a, b = b, a+b 
```

|  |  |
| --- | --- |
| [1] | 斐波纳契数列 (Fibonacci sequence) 是每个数都是前面两个数值和的一个数列。它从 `0` 和 `1` 开始，开始增长得很慢，但越来越快。开始这个数列你需要两个变量：`a` 从 `0`开始，`b` 从 `1` 开始。 |
| [2] | `a` 是数列的当前值，弹出它。 |
| [3] | `b` 是数列的下一个数，把它赋值给 `a`，同时计算出 (`a+b`) 并赋值给 `b` 放在一边稍后使用。注意这是并行发生的，如果 `a` 是 `3`，`b` 是 `5`，那么 `a, b = b, a+b` 将会设置 `a` 为 `5` (`b` 的原值)，`b` 为 `8` (`a` 和 `b` 之和)。 |

这样你就有了生成连续的 Fibonacci 数的函数了。当然你也可以通过递归做到，但是这里的方法更加易读。并且也与 `for` 工作得很好。

## 例 17.20\. `for` 循环中的生成器

```py
>>> for n in fibonacci(1000): 
... print n,              
0 1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 
```

|  |  |
| --- | --- |
| [1] | 你可以在 `for` 循环中直接使用 `fibonacci` 这样的生成器。`for` 循环将会创建一个生成器对象并连续调用其 `next()` 方法获得值并赋予 `for` 循环变量 (`n`)。 |
| [2] | 每轮 `for` 循环 `n` 都从 `fibonacci` 的 `yield` 语句获得一个新的值。当 `fibonacci` 超出数字限定 (`a` 超过 `max`，你在这里限定的是 `1000`) 很自然地退出 `for` 循环。 |

好了，让我们回到 `plural` 函数看看如何可以把它用起来。

## 例 17.21\. 生成器生成动态函数

```py
 def rules(language):                                                                 
    for line in file('rules.%s' % language):                                          
        pattern, search, replace = line.split()                                       
        yield lambda word: re.search(pattern, word) and re.sub(search, replace, word) 
def plural(noun, language='en'):      
    for applyRule in rules(language):  
        result = applyRule(noun)      
        if result: return result 
```

|  |  |
| --- | --- |
| [1] | `for line in file(...)` 是从文件中一行行读取的通用方法，每次一行。它能正常工作是因为 *`file` 实际上返回一个生成器*，它的 `next()` 方法返回文件中的下一行。简直太酷了，光是想想就让我满头大汗。 |
| [2] | 这没有什么神奇之处。还记得规则文件的每一行都用空白分开三个值吗？所以 `line.split()` 返回一个三元素元组，你把这些值赋给了 3 个局部变量。 |
| [3] | *然后你不断地弹出。* 你弹出什么呢？一个使用 `lambda` 动态生成的函数，而这个函数实际上是一个闭合 (把本地变量 `pattern`，`search` 和 `replace` 作为常量)。换句话说，`rules` 是一个弹出规则函数的生成器。 |
| [4] | 既然 `rules` 是一个生成器，你就可以在 `for` 循环中直接使用它。`for` 循环的第一轮你调用 `rules` 函数，打开规则文件，读取第一行，动态构建一个根据规则文件第一行匹配并应用规则的函数。`for` 循环的第二轮将会从上一轮 `rules` 中停下的位置 (`for line in file(...)` 循环内部) 读取规则文件的第二行，动态构建根据规则文件第二行匹配并应用规则的另一个函数。如此继续下去。 |

你在第 5 阶段得到的是什么？第 5 阶段中，你读取整个规则文件并在使用第一条规则之前构建一个所有规则组成的列表。现在有了生成器，你可以更舒适地做到这一切：你打开并读取第一条规则，根据它创建函数并使用之，如果它适用则根本不去读取规则文件剩下的内容，也不去建立另外的函数。

## 进一步阅读

*   PEP 255 定义生成器。
*   Python Cookbook 有[生成器的例子](http://www.google.com/search?q=generators+cookbook+site:aspn.activestate.com)。

# 17.8\. 小结

# 17.8\. 小结

这一章中我们探讨了几个不同的高级技术。它们并不都适用于任何情况。

你现在应该能自如应用如下技术：

*   应用正则表达式进行字符串替换。
*   将函数当作对象，把它们存于列表中，把它们赋值给变量，并通过变量来调用它们。
*   构建应用 `lambda` 的动态函数。
*   构建闭合，将外部变量作为常量构建动态函数。
*   构建生成器，进行逻辑递增操作并在每次调用时返回不同值的恢复执行函数。

抽象化，动态构建函数，构建闭合以及应用生成器能够使你的代码更加简单化、可读化、灵活化。你需要在简洁和功能实现两方面进行平衡。