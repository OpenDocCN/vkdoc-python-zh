# 第三章 内置数据类型

# 第三章 内置数据类型

*   3.1\. Dictionary 介绍
    *   3.1.1\. Dictionary 的定义
    *   3.1.2\. Dictionary 的修改
    *   3.1.3\. 从 dictionary 中删除元素
*   3.2\. List 介绍
    *   3.2.1\. List 的定义
    *   3.2.2\. 向 list 中增加元素
    *   3.2.3\. 在 list 中搜索
    *   3.2.4\. 从 list 中删除元素
    *   3.2.5\. 使用 list 的运算符
*   3.3\. Tuple 介绍
*   3.4\. 变量声明
    *   3.4.1\. 变量引用
    *   3.4.2\. 一次赋多值
*   3.5\. 格式化字符串
*   3.6\. 映射 list
*   3.7\. 连接 list 与分割字符串
    *   3.7.1\. 字符串方法的历史注解
*   3.8\. 小结

让我们用点儿时间来回顾一下您的第一个 Python 程序。但首先，先说些其他的内容，因为您需要了解一下 dictionary (字典)、tuple (元组) 和 list (列表)(哦，我的老天！)。如果您是一个 Perl hacker，当然可以撇开 dictionary 和 list，但是仍然需要注意 tuple。

# 3.1\. Dictionary 介绍

# 3.1\. Dictionary 介绍

*   3.1.1\. Dictionary 的定义
*   3.1.2\. Dictionary 的修改
*   3.1.3\. 从 dictionary 中删除元素

Dictionary 是 Python 的内置数据类型之一，它定义了键和值之间一对一的关系。

> 注意
> Python 中的 dictionary 就像 Perl 中的 hash (哈希数组)。在 Perl 中，存储哈希值的变量总是以 `%` 字符开始；在 Python 中，变量可以任意取名，并且 Python 在内部会记录下其数据类型。
> 
> 注意
> Python 中的 dictionary 像 Java 中的 `Hashtable` 类的实例。
> 
> 注意
> Python 中的 dictionary 像 Visual Basic 中的 `Scripting.Dictionary` 对象的实例。

## 3.1.1\. Dictionary 的定义

## 例 3.1\. 定义 Dictionary

```py
>>> d = {"server":"mpilgrim", "database":"master"} 
>>> d
{'server': 'mpilgrim', 'database': 'master'}
>>> d["server"]                                    
'mpilgrim'
>>> d["database"]                                  
'master'
>>> d["mpilgrim"]                                  
Traceback (innermost last):
  File "<interactive input>", line 1, in ?
KeyError: mpilgrim 
```

|  |  |
| --- | --- |
| [1] | 首先我们创建了新 dictionary，它有两个元素，将其赋给变量 `d` 。每一个元素都是一个 key-value 对；整个元素集合用大括号括起来。 |
| [2] | `'server'` 是一个 key，它所关联的值是通过 `d["server"]` 来引用的，为 `'mpilgrim'`。 |
| [3] | `'database'` 是一个 key，它所关联的值是通过 `d["database"]` 来引用的，为 `'master'`。 |
| [4] | 您可以通过 key 来引用其值，但是不能通过值获取 key。所以 `d["server"]` 的值为 `'mpilgrim'`，而使用 `d["mpilgrim"]` 会引发一个异常，因为 `'mpilgrim'` 不是一个 key。 |

## 3.1.2\. Dictionary 的修改

## 例 3.2\. 修改 Dictionary

```py
>>> d
{'server': 'mpilgrim', 'database': 'master'}
>>> d["database"] = "pubs" 
>>> d
{'server': 'mpilgrim', 'database': 'pubs'}
>>> d["uid"] = "sa"        
>>> d
{'server': 'mpilgrim', 'uid': 'sa', 'database': 'pubs'} 
```

|  |  |
| --- | --- |
| [1] | 在一个 dictionary 中不能有重复的 key。给一个存在的 key 赋值会覆盖原有的值。 |
| [2] | 在任何时候都可以加入新的 key-value 对。这种语法同修改存在的值是一样的。(是的，它可能某天会给您带来麻烦。假设你一次次地修改一个 dictionary，但其间您使用的 key 并未按照您的想法进行改变。您可能以为加入了新值，但实际上只是一次又一次地修改了同一个值。) |

请注意新的元素 (key 为 `'uid'`，value 为 `'sa'`) 出现在中间。实际上，在第一个例子中的元素看上去是的有序不过是一种巧合。现在它们看上去的无序同样是一种巧合。

> 注意
> Dictionary 没有元素顺序的概念。说元素 “顺序乱了” 是不正确的，它们只是序偶的简单排列。这是一个重要的特性，它会在您想要以一种特定的，可重现的顺序 (像以 key 的字母表顺序) 存取 dictionary 元素的时候骚扰您。有一些实现这些要求的方法，它们只是没有加到 dictionary 中去。

当使用 dictionary 时，您需要知道：dictionary 的 key 是大小写敏感的。

## 例 3.3\. Dictionary 的 key 是大小写敏感的

```py
>>> d = {}
>>> d["key"] = "value"
>>> d["key"] = "other value" 
>>> d
{'key': 'other value'}
>>> d["Key"] = "third value" 
>>> d
{'Key': 'third value', 'key': 'other value'} 
```

|  |  |
| --- | --- |
| [1] | 为一个已经存在的 dictionary key 赋值，将简单覆盖原有的值。 |
| [2] | 这不会为一个已经存在的 dictionary key 赋值，因为在 Python 中是区分大小写的，也就是说 `'key'` 与 `'Key'` 是不同的。所以这种情况将在 dictionary 中创建一个新的 key-value 对。虽然看上去很相近，但是在 Python 眼里是完全不同的。 |

## 例 3.4\. 在 dictionary 中混用数据类型

```py
>>> d
{'server': 'mpilgrim', 'uid': 'sa', 'database': 'pubs'}
>>> d["retrycount"] = 3 
>>> d
{'server': 'mpilgrim', 'uid': 'sa', 'database': 'master', 'retrycount': 3}
>>> d[42] = "douglas"   
>>> d
{'server': 'mpilgrim', 'uid': 'sa', 'database': 'master',
42: 'douglas', 'retrycount': 3} 
```

|  |  |
| --- | --- |
| [1] | Dictionary 不只是用于存储字符串。Dictionary 的值可以是任意数据类型，包括字符串、整数、对象，甚至其它的 dictionary。在单个 dictionary 里，dictionary 的值并不需要全都是同一数据类型，可以根据需要混用和匹配。 |
| [2] | Dictionary 的 key 要严格多了，但是它们可以是字符串、整数或几种其它的类型 (后面还会谈到这一点)。也可以在一个 dictionary 中混用和匹配 key 的数据类型。 |

## 3.1.3\. 从 dictionary 中删除元素

## 例 3.5\. 从 dictionary 中删除元素

```py
>>> d
{'server': 'mpilgrim', 'uid': 'sa', 'database': 'master',
42: 'douglas', 'retrycount': 3}
>>> del d[42] 
>>> d
{'server': 'mpilgrim', 'uid': 'sa', 'database': 'master', 'retrycount': 3}
>>> d.clear() 
>>> d
{} 
```

|  |  |
| --- | --- |
| [1] | `del` 允许您使用 key 从一个 dictionary 中删除独立的元素。 |
| [2] | `clear` 从一个 dictionary 中清除所有元素。注意空的大括号集合表示一个没有元素的 dictionary。 |

## 进一步阅读

*   *How to Think Like a Computer Scientist* 讲授了 dictionary 和如何[使用 dictionary 模拟稀疏矩阵](http://www.ibiblio.org/obp/thinkCSpy/chap10.htm)。
*   Python Knowledge Base 有许多[使用 dictionary 的示例代码](http://www.faqts.com/knowledge-base/index.phtml/fid/541)。
*   Python Cookbook 讨论了[如何通过 key 对 dictionary 的值进行排序](http://www.activestate.com/ASPN/Python/Cookbook/Recipe/52306)。
*   *Python Library Reference* 总结了[所有的 dictionary 方法](http://www.python.org/doc/current/lib/typesmapping.html)。

# 3.2\. List 介绍

# 3.2\. List 介绍

*   3.2.1\. List 的定义
*   3.2.2\. 向 list 中增加元素
*   3.2.3\. 在 list 中搜索
*   3.2.4\. 从 list 中删除元素
*   3.2.5\. 使用 list 的运算符

List 是 Python 中使用最频繁的数据类型。如果您对 list 仅有的经验就是在 Visual Basic 中的数组或 Powerbuilder 中的数据存储，那么就打起精神学习 Python 的 list 吧。

> 注意
> Python 的 list 如同 Perl 中的数组。在 Perl 中，用来保存数组的变量总是以 `@` 字符开始；在 Python 中，变量可以任意取名，并且 Python 在内部会记录下其数据类型。
> 
> 注意
> Python 中的 list 更像 Java 中的数组 (您可以简单地这样理解，但 Python 中的 list 远比 Java 中的数组强大)。一个更好的类比是 `ArrayList` 类，它可以保存任意对象，并且可以在增加新元素时动态扩展。

## 3.2.1\. List 的定义

## 例 3.6\. 定义 List

```py
>>> li = ["a", "b", "mpilgrim", "z", "example"] 
>>> li
['a', 'b', 'mpilgrim', 'z', 'example']
>>> li[0]                                       
'a'
>>> li[4]                                       
'example' 
```

|  |  |
| --- | --- |
| [1] | 首先我们定义了一个有 5 个元素的 list。注意它们保持着初始的顺序。这不是偶然。List 是一个用方括号包括起来的有序元素的集合。 |
| [2] | List 可以作为以 0 下标开始的数组。任何一个非空 list 的第一个元素总是 `li[0]`。 |
| [3] | 这个包含 5 个元素 list 的最后一个元素是 `li[4]`，因为列表总是从 0 开始。 |

## 例 3.7\. 负的 list 索引

```py
>>> li
['a', 'b', 'mpilgrim', 'z', 'example']
>>> li[-1] 
'example'
>>> li[-3] 
'mpilgrim' 
```

|  |  |
| --- | --- |
| [1] | 负数索引从 list 的尾部开始向前计数来存取元素。任何一个非空的 list 最后一个元素总是 `li[-1]`。 |
| [2] | 如果负数索引使您感到糊涂，可以这样理解：`li[-n] == li[len(li) - n]`。所以在这个 list 里，`li[-3] == li[5 - 3] == li[2]`。 |

## 例 3.8\. list 的分片 (slice)

```py
>>> li
['a', 'b', 'mpilgrim', 'z', 'example']
>>> li[1:3]  
['b', 'mpilgrim']
>>> li[1:-1] 
['b', 'mpilgrim', 'z']
>>> li[0:3]  
['a', 'b', 'mpilgrim'] 
```

|  |  |
| --- | --- |
| [1] | 您可以通过指定 2 个索引得到 list 的子集，叫做一个 “slice” 。返回值是一个新的 list，它包含了 list 中按顺序从第一个 slice 索引 (这里为 `li[1]`) 开始，直到但是不包括第二个 slice 索引 (这里为 `li[3]`) 的所有元素。 |
| [2] | 如果一个或两个 slice 索引是负数，slice 也可以工作。如果对您有帮助，您可以这样理解：从左向右阅读 list，第一个 slice 索引指定了您想要的第一个元素，第二个 slice 索引指定了第一个您不想要的元素。返回的值为在其间的每个元素。 |
| [3] | List 从 0 开始，所以 `li[0:3]` 返回 list 的前 3 个元素，从 `li[0]` 开始，直到但不包括 `li[3]`。 |

## 例 3.9\. Slice 简写

```py
>>> li
['a', 'b', 'mpilgrim', 'z', 'example']
>>> li[:3] 
['a', 'b', 'mpilgrim']
>>> li[3:]  
['z', 'example']
>>> li[:]  
['a', 'b', 'mpilgrim', 'z', 'example'] 
```

|  |  |
| --- | --- |
| [1] | 如果左侧分片索引为 0，您可以将其省略，默认为 0。所以 `li[:3]` 同 例 3.8 “list 的分片 (slice)”") 的 `li[0:3]` 是一样的。 |
| [2] | 同样的，如果右侧分片索引是 list 的长度，可以将其省略。所以 `li[3:]` 同 `li[3:5]` 是一样的，因为这个 list 有 5 个元素。 |
| [3] | 请注意这里的对称性。在这个包含 5 个元素的 list 中，`li[:3]` 返回前 3 个元素，而 `li[3:]` 返回后 2 个元素。实际上，`li[:n]` 总是返回前 `n` 个元素，而 `li[n:]` 将返回剩下的元素，不管 list 有多长。 |
| [4] | 如果将两个分片索引全部省略，这将包括 list 的所有元素。但是与原始的名为 `li` 的 list 不同，它是一个新 list，恰好拥有与 `li` 一样的全部元素。`li[:]` 是生成一个 list 完全拷贝的一个简写。 |

## 3.2.2\. 向 list 中增加元素

## 例 3.10\. 向 list 中增加元素

```py
>>> li
['a', 'b', 'mpilgrim', 'z', 'example']
>>> li.append("new")               
>>> li
['a', 'b', 'mpilgrim', 'z', 'example', 'new']
>>> li.insert(2, "new")            
>>> li
['a', 'b', 'new', 'mpilgrim', 'z', 'example', 'new']
>>> li.extend(["two", "elements"]) 
>>> li
['a', 'b', 'new', 'mpilgrim', 'z', 'example', 'new', 'two', 'elements'] 
```

|  |  |
| --- | --- |
| [1] | `append` 向 list 的末尾追加单个元素。 |
| [2] | `insert` 将单个元素插入到 list 中。数值参数是插入点的索引。请注意，list 中的元素不必唯一，现在有两个独立的元素具有 `'new'` 这个值，`li[2]` 和 `li[6]`。 |
| [3] | `extend` 用来连接 list。请注意不要使用多个参数来调用 `extend`，要使用一个 list 参数进行调用。在本例中，这个 list 有两个元素。 |

## 例 3.11\. `extend` (扩展) 与 `append` (追加) 的差别

```py
>>> li = ['a', 'b', 'c']
>>> li.extend(['d', 'e', 'f']) 
>>> li
['a', 'b', 'c', 'd', 'e', 'f']
>>> len(li)                    
6
>>> li[-1]
'f'
>>> li = ['a', 'b', 'c']
>>> li.append(['d', 'e', 'f']) 
>>> li
['a', 'b', 'c', ['d', 'e', 'f']]
>>> len(li)                    
4
>>> li[-1]
['d', 'e', 'f'] 
```

|  |  |
| --- | --- |
| [1] | Lists 的两个方法 `extend` 和 `append` 看起来类似，但实际上完全不同。`extend` 接受一个参数，这个参数总是一个 list，并且把这个 list 中的每个元素添加到原 list 中。 |
| [2] | 在这里 list 中有 3 个元素 (`'a'`、`'b'` 和 `'c'`)，并且使用另一个有 3 个元素 (`'d'`、`'e'` 和 `'f'`) 的 list 扩展之，因此新的 list 中有 6 个元素。 |
| [3] | 另一方面，`append` 接受一个参数，这个参数可以是任何数据类型，并且简单地追加到 list 的尾部。在这里使用一个含有 3 个元素的 list 参数调用 `append` 方法。 |
| [4] | 原来包含 3 个元素的 list 现在包含 4 个元素。为什么是 4 个元素呢？因为刚刚追加的最后一个元素*本身是个 list*。List 可以包含任何类型的数据，也包括其他的 list。这或许是您所要的结果，或许不是。如果您的意图是 `extend`，请不要使用 `append`。 |

## 3.2.3\. 在 list 中搜索

## 例 3.12\. 搜索 list

```py
>>> li
['a', 'b', 'new', 'mpilgrim', 'z', 'example', 'new', 'two', 'elements']
>>> li.index("example") 
5
>>> li.index("new")     
2
>>> li.index("c")       
Traceback (innermost last):
  File "<interactive input>", line 1, in ?
ValueError: list.index(x): x not in list
>>> "c" in li           
False 
```

|  |  |
| --- | --- |
| [1] | `index` 在 list 中查找一个值的首次出现并返回索引值。 |
| [2] | `index` 在 list 中查找一个值的*首次* 出现。这里 `'new'` 在 list 中出现了两次，在 `li[2]` 和 `li[6]`，但 `index` 只返回第一个索引，`2`。 |
| [3] | 如果在 list 中没有找到值，Python 会引发一个异常。这一点与大部分的语言截然不同，大部分语言会返回某个无效索引。尽管这种处理可能令人讨厌，但它仍然是件好事，因为它说明您的程序会由于源代码的问题而崩溃，好于在后面当您使用无效索引而引起崩溃。 |
| [4] | 要测试一个值是否在 list 内，使用 `in`。如果值存在，它返回 `True`，否则返为 `False` 。 |

> 注意
> 在 2.2.1 版本之前，Python 没有单独的布尔数据类型。为了弥补这个缺陷，Python 在布尔环境 (如 `if` 语句) 中几乎接受所有东西，遵循下面的规则：
> 
> *   `0` 为 false; 其它所有数值皆为 true。
> *   空串 (`""`) 为 false; 其它所有字符串皆为 true。
> *   空 list (`[]`) 为 false; 其它所有 list 皆为 true。
> *   空 tuple (`()`) 为 false; 其它所有 tuple 皆为 true。
> *   空 dictionary (`{}`) 为 false; 其它所有 dictionary 皆为 true。
> 
> 这些规则仍然适用于 Python 2.2.1 及其后续版本，但现在您也可以使用真正的布尔值，它的值或者为 `True` 或者为 `False`。请注意第一个字母是大写的；这些值如同在 Python 中的其它东西一样都是大小写敏感的。

## 3.2.4\. 从 list 中删除元素

## 例 3.13\. 从 list 中删除元素

```py
>>> li
['a', 'b', 'new', 'mpilgrim', 'z', 'example', 'new', 'two', 'elements']
>>> li.remove("z")   
>>> li
['a', 'b', 'new', 'mpilgrim', 'example', 'new', 'two', 'elements']
>>> li.remove("new") 
>>> li
['a', 'b', 'mpilgrim', 'example', 'new', 'two', 'elements']
>>> li.remove("c")   
Traceback (innermost last):
  File "<interactive input>", line 1, in ?
ValueError: list.remove(x): x not in list
>>> li.pop()         
'elements'
>>> li
['a', 'b', 'mpilgrim', 'example', 'new', 'two'] 
```

|  |  |
| --- | --- |
| [1] | `remove` 从 list 中删除一个值的首次出现。 |
| [2] | `remove` *仅仅* 删除一个值的首次出现。在这里，`'new'` 在 list 中出现了两次，但 `li.remove("new")` 只删除了 `'new'` 的首次出现。 |
| [3] | 如果在 list 中没有找到值，Python 会引发一个异常来响应 `index` 方法。 |
| [4] | `pop` 是一个有趣的东西。它会做两件事：删除 list 的最后一个元素，然后返回删除元素的值。请注意，这与 `li[-1]` 不同，后者返回一个值但不改变 list 本身。也不同于 `li.remove(_value_)`，后者改变 list 但并不返回值。 |

## 3.2.5\. 使用 list 的运算符

## 例 3.14\. List 运算符

```py
>>> li = ['a', 'b', 'mpilgrim']
>>> li = li + ['example', 'new'] 
>>> li
['a', 'b', 'mpilgrim', 'example', 'new']
>>> li += ['two']                
>>> li
['a', 'b', 'mpilgrim', 'example', 'new', 'two']
>>> li = [1, 2] * 3              
>>> li
[1, 2, 1, 2, 1, 2] 
```

|  |  |
| --- | --- |
| [1] | Lists 也可以用 `+` 运算符连接起来。`_list_ = _list_ + _otherlist_` 相当于 `_list_.extend(_otherlist_)`。但 `+` 运算符把一个新 (连接后) 的 list 作为值返回，而 `extend` 只修改存在的 list。也就是说，对于大型 list 来说，`extend` 的执行速度要快一些。 |
| [2] | Python 支持 `+=` 运算符。`li += ['two']` 等同于 `li.extend(['two'])`。`+=` 运算符可用于 list、字符串和整数，并且它也可以被重载用于用户自定义的类中 (更多关于类的内容参见 第五章)。 |
| [3] | `*` 运算符可以作为一个重复器作用于 list。`li = [1, 2] * 3` 等同于 `li = [1, 2] + [1, 2] + [1, 2]`，即将三个 list 连接成一个。 |

## 进一步阅读

*   *How to Think Like a Computer Scientist* 讲述了 list，并且重点讲述了如何[把 list 作为函数参数传递](http://www.ibiblio.org/obp/thinkCSpy/chap08.htm)。
*   *Python Tutorial* 展示了如何[把 list 作为堆栈和队列使用](http://www.python.org/doc/current/tut/node7.html#SECTION007110000000000000000)。
*   Python Knowledge Base 回答了[有关 list 的常见问题](http://www.faqts.com/knowledge-base/index.phtml/fid/534)并且有许多[使用 list 的示例代码](http://www.faqts.com/knowledge-base/index.phtml/fid/540)。
*   *Python Library Reference* 总结了[所有的 list 方法](http://www.python.org/doc/current/lib/typesseq-mutable.html)。

# 3.3\. Tuple 介绍

# 3.3\. Tuple 介绍

Tuple 是不可变的 list。一旦创建了一个 tuple，就不能以任何方式改变它。

## 例 3.15\. 定义 tuple

```py
>>> t = ("a", "b", "mpilgrim", "z", "example") 
>>> t
('a', 'b', 'mpilgrim', 'z', 'example')
>>> t[0]                                       
'a'
>>> t[-1]                                      
'example'
>>> t[1:3]                                     
('b', 'mpilgrim') 
```

|  |  |
| --- | --- |
| [1] | 定义 tuple 与定义 list 的方式相同，但整个元素集是用小括号包围的，而不是方括号。 |
| [2] | Tuple 的元素与 list 一样按定义的次序进行排序。Tuples 的索引与 list 一样从 0 开始，所以一个非空 tuple 的第一个元素总是 `t[0]`。 |
| [3] | 负数索引与 list 一样从 tuple 的尾部开始计数。 |
| [4] | 与 list 一样分片 (slice) 也可以使用。注意当分割一个 list 时，会得到一个新的 list ；当分割一个 tuple 时，会得到一个新的 tuple。 |

## 例 3.16\. Tuple 没有方法

```py
>>> t
('a', 'b', 'mpilgrim', 'z', 'example')
>>> t.append("new")    
Traceback (innermost last):
  File "<interactive input>", line 1, in ?
AttributeError: 'tuple' object has no attribute 'append'
>>> t.remove("z")      
Traceback (innermost last):
  File "<interactive input>", line 1, in ?
AttributeError: 'tuple' object has no attribute 'remove'
>>> t.index("example") 
Traceback (innermost last):
  File "<interactive input>", line 1, in ?
AttributeError: 'tuple' object has no attribute 'index'
>>> "z" in t           
True 
```

|  |  |
| --- | --- |
| [1] | 您不能向 tuple 增加元素。Tuple 没有 `append` 或 `extend` 方法。 |
| [2] | 您不能从 tuple 删除元素。Tuple 没有 `remove` 或 `pop` 方法。 |
| [3] | 您不能在 tuple 中查找元素。Tuple 没有 `index` 方法。 |
| [4] | 然而，您可以使用 `in` 来查看一个元素是否存在于 tuple 中。 |

那么使用 tuple 有什么好处呢？

*   Tuple 比 list 操作速度快。如果您定义了一个值的常量集，并且唯一要用它做的是不断地遍历它，请使用 tuple 代替 list。
*   如果对不需要修改的数据进行 “写保护”，可以使代码更安全。使用 tuple 而不是 list 如同拥有一个隐含的 `assert` 语句，说明这一数据是常量。如果必须要改变这些值，则需要执行 tuple 到 list 的转换 (需要使用一个特殊的函数)。
*   还记得我说过 dictionary keys 可以是字符串，整数和 “其它几种类型”吗？Tuples 就是这些类型之一。Tuples 可以在 dictionary 中被用做 key，但是 list 不行。实际上，事情要比这更复杂。Dictionary key 必须是不可变的。Tuple 本身是不可改变的，但是如果您有一个 list 的 tuple，那就认为是可变的了，用做 dictionary key 就是不安全的。只有字符串、整数或其它对 dictionary 安全的 tuple 才可以用作 dictionary key。
*   Tuples 可以用在字符串格式化中，我们会很快看到。

> 注意
> Tuple 可以转换成 list，反之亦然。内置的 `tuple` 函数接收一个 list，并返回一个有着相同元素的 tuple。而 `list` 函数接收一个 tuple 返回一个 list。从效果上看，`tuple` 冻结一个 list，而 `list` 解冻一个 tuple。

## 进一步阅读

*   *How to Think Like a Computer Scientist* 讲解了 tuple 并且展示了如何[连接 tuple](http://www.ibiblio.org/obp/thinkCSpy/chap10.htm)。
*   Python Knowledge Base 展示了如何对[一个 tuple 排序](http://www.faqts.com/knowledge-base/view.phtml/aid/4553/fid/587)。
*   *Python Tutorial* 展示了如何[定义一个只包含一个元素的 tuple](http://www.python.org/doc/current/tut/node7.html#SECTION007300000000000000000)。

# 3.4\. 变量声明

# 3.4\. 变量声明

*   3.4.1\. 变量引用
*   3.4.2\. 一次赋多值

现在您已经了解了有关 dictionary、tuple、和 list 的相关知识 (哦，我的老天！)，让我们回到 第二章 的例子程序 `odbchelper.py`。

Python 与大多数其它语言一样有局部变量和全局变量之分，但是它没有明显的变量声明。变量通过首次赋值产生，当超出作用范围时自动消亡。

## 例 3.17\. 定义 `myParams` 变量

```py
 if __name__ == "__main__":
    myParams = {"server":"mpilgrim", \
                "database":"master", \
                "uid":"sa", \
                "pwd":"secret" \
                } 
```

首先注意缩进。`if` 语句是代码块，需要像函数一样缩进。

其次，变量的赋值是一条被分成了多行的命令，用反斜线 (“`\`”) 作为续行符。

> 注意
> 当一条命令用续行符 (“`\`”) 分割成多行时，后续的行可以以任何方式缩进，此时 Python 通常的严格的缩进规则无需遵守。如果您的 Python IDE 自由对后续行进行了缩进，您应该把它当成是缺省处理，除非您有特别的原因不这么做。

严格地讲，在小括号，方括号或大括号中的表达式 (如定义一个 dictionary) 可以用或者不用续行符 (“`\`”) 分割成多行。甚至在不是必需的时候，我也喜欢使用续行符，因为我认为这样会让代码读起来更容易，但那只是风格问题。

第三，您从未声明过变量 `myParams`，您只是给它赋了一个值。这点就像是 VBScript 没有设置 `option explicit` 选项一样。幸运的是，与 VBScript 不同，Python 不允许您引用一个未被赋值的变量，试图这样做会引发一个异常。

## 3.4.1\. 变量引用

## 例 3.18\. 引用未赋值的变量

```py
>>> x
Traceback (innermost last):
  File "<interactive input>", line 1, in ?
NameError: There is no variable named 'x'
>>> x = 1
>>> x
1 
```

迟早有一天您会为此而感谢 Python 。

## 3.4.2\. 一次赋多值

Python 中比较 “酷” 的一种编程简写是使用序列来一次给多个变量赋值。

## 例 3.19\. 一次赋多值

```py
>>> v = ('a', 'b', 'e')
>>> (x, y, z) = v     
>>> x
'a'
>>> y
'b'
>>> z
'e' 
```

|  |  |
| --- | --- |
| [1] | `v` 是一个三元素的 tuple，并且 `(x, y, z)` 是一个三变量的 tuple。将一个 tuple 赋值给另一个 tuple，会按顺序将 `v` 的每个值赋值给每个变量。 |

这种用法有许多种用途。我经常想要将一定范围的值赋给多个变量。在 C 语言中，可以使用 `enum` 类型，手工列出每个常量和其所对应的值，当值是连续的时候这一过程让人感到特别繁琐。而在 Python 中，您可以使用内置的 `range` 函数和多变量赋值的方法来快速进行赋值。

## 例 3.20\. 连续值赋值

```py
>>> range(7)                                                                    
[0, 1, 2, 3, 4, 5, 6]
>>> (MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY) = range(7) 
>>> MONDAY                                                                      
0
>>> TUESDAY
1
>>> SUNDAY
6 
```

|  |  |
| --- | --- |
| [1] | 内置的 `range` 函数返回一个元素为整数的 list。这个函数的简化调用形式是接收一个上限值，然后返回一个初始值从 0 开始的 list，它依次递增，直到但不包含上限值。(如果您愿意，您可以传入其它的参数来指定一个非 `0` 的初始值和非 `1` 的步长。也可以使用 `print range.__doc__` 来了解更多的细节。) |
| [2] | `MONDAY`、`TUESDAY`、`WEDNESDAY`、`THURSDAY`、`FRIDAY`、`SATURDAY` 和 `SUNDAY` 是我们定义的变量。(这个例子来自 `calendar` 模块。它是一个很有趣的打印日历的小模块，像 UNIX 的 `cal` 命令。这个 `calendar` 模块定义了一星期中每天的整数常量表示。) |
| [3] | 现在每个变量都拥有了自己的值：`MONDAY` 的值为 `0`，`TUESDAY` 的值为 `1`，等等。 |

您也可以使用多变量赋值来创建返回多个值的函数，只要返回一个包含所有值的 tuple 即可。调用者可以将其视为一个 tuple，或将值赋给独立的变量。许多标准的 Python 库都是这样做的，包括 `os` 模块，我们将在 第六章 中讨论。

## 进一步阅读

*   *Python Reference Manual* 展示了[什么时候可以忽略续行符](http://www.python.org/doc/current/ref/implicit-joining.html)和[什么时候您需要使用续行符](http://www.python.org/doc/current/ref/explicit-joining.html)的例子。
*   *How to Think Like a Computer Scientist* 演示了如何使用多变量赋值来[交换两个变量的值](http://www.ibiblio.org/obp/thinkCSpy/chap09.htm)。

# 3.5\. 格式化字符串

# 3.5\. 格式化字符串

Python 支持格式化字符串的输出 。尽管这样可能会用到非常复杂的表达式，但最基本的用法是将一个值插入到一个有字符串格式符 `%s` 的字符串中。

> 注意
> 在 Python 中，字符串格式化使用与 C 中 `sprintf` 函数一样的语法。

## 例 3.21\. 字符串的格式化

```py
>>> k = "uid"
>>> v = "sa"
>>> "%s=%s" % (k, v) 
'uid=sa' 
```

|  |  |
| --- | --- |
| [1] | 整个表达式的值为一个字符串。第一个 `%s` 被变量 `k` 的值替换；第二个 `%s` 被 `v` 的值替换。字符串中的所有其它字符 (在这个例子中，是等号) 按原样打印输出。 |

注意 `(k, v)` 是一个 tuple。我说过它们对某些东西有用。

您可能一直在想，做了这么多工作只不过是为了做简单的字符串连接。您想的不错，只不过字符串格式化不只是连接。它甚至不仅仅是格式化。它也是强制类型转换。

## 例 3.22\. 字符串格式化与字符串连接的比较

```py
>>> uid = "sa"
>>> pwd = "secret"
>>> print pwd + " is not a good password for " + uid      
secret is not a good password for sa
>>> print "%s is not a good password for %s" % (pwd, uid) 
secret is not a good password for sa
>>> userCount = 6
>>> print "Users connected: %d" % (userCount, )            
Users connected: 6
>>> print "Users connected: " + userCount                 
Traceback (innermost last):
  File "<interactive input>", line 1, in ?
TypeError: cannot concatenate 'str' and 'int' objects 
```

|  |  |
| --- | --- |
| [1] | `+` 是字符串连接操作符。 |
| [2] | 在这个简单例子中，字符串格式化实现与连接一样的结果。 |
| [3] | `(userCount, )` 是一个只包含一个元素的 tuple。是的，语法有一点奇怪，但是使用它的理由就是：显示地指出它是一个 tuple，而不是其他。实际上，当定义一个 list、tuple 或 dictionary 时，您可以总是在最后一个元素后面跟上一个逗号，但是当定义一个只包含一个元素的 tuple 时逗号是必须的。如果省略逗号，Python 不会知道 `(userCount)` 究竟是一个只包含一个元素的 tuple 还是变量 `userCount` 的值。 |
| [4] | 字符串格式化通过将 `%s` 替换成 `%d` 即可处理整数。 |
| [5] | 试图将一个字符串同一个非字符串连接会引发一个异常。与字符串格式化不同，字符串连接只能在被连接的每一个都是字符串时起作用。 |

如同 `printf` 在 C 中的作用，Python 中的字符串格式化是一把瑞士军刀。它有丰富的选项，不同的格式化格式符和可选的修正符可用于不同的数据类型。

## 例 3.23\. 数值的格式化

```py
>>> print "Today's stock price: %f" % 50.4625   
50.462500
>>> print "Today's stock price: %.2f" % 50.4625 
50.46
>>> print "Change since yesterday: %+.2f" % 1.5 
+1.50 
```

|  |  |
| --- | --- |
| [1] | `%f` 格式符选项对应一个十进制浮点数，不指定精度时打印 6 位小数。 |
| [2] | 使用包含“.2”精度修正符的 `%f` 格式符选项将只打印 2 位小数。 |
| [3] | 您甚至可以混合使用各种修正符。添加 `+` 修正符用于在数值之前显示一个正号或负号。注意“.2”精度修正符仍旧在它原来的位置，用于只打印 2 位小数。 |

## 进一步阅读

*   *Python Library Reference* 总结了[所有字符串格式化所使用的格式符](http://www.python.org/doc/current/lib/typesseq-strings.html)。
*   _Effective AWK Programming_Top) 讨论了[所有的格式符](http://www-gnats.gnu.org:8080/cgi-bin/info2www?(gawk)Control+Letters)和高级字符串格式化技术，如[指定宽度，精度和 0 填充](http://www-gnats.gnu.org:8080/cgi-bin/info2www?(gawk)Format+Modifiers)。

# 3.6\. 映射 list

# 3.6\. 映射 list

Python 的强大特性之一是其对 list 的解析，它提供一种紧凑的方法，可以通过对 list 中的每个元素应用一个函数，从而将一个 list 映射为另一个 list。

## 例 3.24\. List 解析介绍

```py
>>> li = [1, 9, 8, 4]
>>> [elem*2 for elem in li]      
[2, 18, 16, 8]
>>> li                           
[1, 9, 8, 4]
>>> li = [elem*2 for elem in li] 
>>> li
[2, 18, 16, 8] 
```

|  |  |
| --- | --- |
| [1] | 为了便于理解它，让我们从右向左看。`li` 是一个将要映射的 list。Python 循环遍历 `li` 中的每个元素。对每个元素均执行如下操作：首先临时将其值赋给变量 `elem`，然后 Python 应用函数 ``elem`*2` 进行计算，最后将计算结果追加到要返回的 list 中。 |
| [2] | 需要注意是，对 list 的解析并不改变原始的 list。 |
| [3] | 将一个 list 的解析结果赋值给对其映射的变量是安全的。不用担心存在竞争情况或任何古怪事情的发生。Python 会在内存中创建新的 list，当对 list 的解析完成时，Python 将结果赋给变量。 |

让我们回过头来看看位于 第二章 的函数 `buildConnectionString` 对 list 的解析：

```py
["%s=%s" % (k, v) for k, v in params.items()] 
```

首先，注意到你调用了 dictionary `params` 的 `items` 函数。这个函数返回一个 dictionary 中所有数据的 tuple 的 list。

## 例 3.25\. `keys`, `values` 和 `items` 函数

```py
>>> params = {"server":"mpilgrim", "database":"master", "uid":"sa", "pwd":"secret"}
>>> params.keys()   
['server', 'uid', 'database', 'pwd']
>>> params.values() 
['mpilgrim', 'sa', 'master', 'secret']
>>> params.items()  
[('server', 'mpilgrim'), ('uid', 'sa'), ('database', 'master'), ('pwd', 'secret')] 
```

|  |  |
| --- | --- |
| [1] | Dictionary 的 `keys` 方法返回一个包含所有键的 list。这个 list 没按 dictionary 定义的顺序输出 (记住，元素在 dictionary 中是无序的)，但它是一个 list。 |
| [2] | `values` 方法返回一个包含所有值的 list。它同 `keys` 方法返回的 list 输出顺序相同，所以对于所有的 `n`，`params.values()[n] == params[params.keys()[n]]` 。 |
| [3] | `items` 方法返回一个由形如 `(_key_，_value_)` 组成的 tuple 的 list。这个 list 包括 dictionary 中所有的数据。 |

现在让我们看一看 `buildConnectionString` 做了些什么。它接收一个 list，`params`.`items`()`，通过对每个元素应用字符串格式化将其映射为一个新 list。这个新 list 将与`params`.`items`()` 一一对应：新 list 中的每个元素都是 dictionary `params` 中的一个键-值对构成的的字符串。

## 例 3.26\. `buildConnectionString` 中的 list 解析

```py
>>> params = {"server":"mpilgrim", "database":"master", "uid":"sa", "pwd":"secret"}
>>> params.items()
[('server', 'mpilgrim'), ('uid', 'sa'), ('database', 'master'), ('pwd', 'secret')]
>>> [k for k, v in params.items()]                
['server', 'uid', 'database', 'pwd']
>>> [v for k, v in params.items()]                
['mpilgrim', 'sa', 'master', 'secret']
>>> ["%s=%s" % (k, v) for k, v in params.items()] 
['server=mpilgrim', 'uid=sa', 'database=master', 'pwd=secret'] 
```

|  |  |
| --- | --- |
| [1] | 请注意我们正在使用两个变量对 list `params.items()` 进行遍历。这是多变量赋值的另一种用法。`params.items()` 的第一个元素是 `('server', 'mpilgrim')`，所以在 list 解析的第一次遍历中，`k` 将为 `'server'`，`v` 将为 `'mpilgrim'`。在本例中，我们忽略了返回 list 中 `v` 的值，而只包含了 `k` 的值，所以这个 list 解析最后等于 ``params`.`keys`()`。 |
| [2] | 这里我们做着相同的事情，但是忽略了 `k` 的值，所以这个 list 解析最后等于 ``params`.`values`()`。 |
| [3] | 用一些简单的 字符串格式化将前面两个例子合并起来 ，我们就得到一个包括了 dictionary 中每个元素的 key-value 对的 list。这个看上去有点像程序的输出结果，剩下的就只是将这个 list 中的元素接起来形成一个字符串了。 |

## 进一步阅读

*   *Python Tutorial* 讨论了另一种方法来映射 list：[使用内置的 `map` 函数](http://www.python.org/doc/current/tut/node7.html#SECTION007130000000000000000)。
*   *Python Tutorial* 展示了如何[对嵌套 list 的 list 进行解析](http://www.python.org/doc/current/tut/node7.html#SECTION007140000000000000000)。

# 3.7\. 连接 list 与分割字符串

# 3.7\. 连接 list 与分割字符串

*   3.7.1\. 字符串方法的历史注解

您有了一个形如 `_key_=_value_` 的 key-value 对 list，并且想将它们合成为单个字符串。为了将任意包含字符串的 list 连接成单个字符串，可以使用字符串对象的 `join` 方法。

下面是一个在 `buildConnectionString` 函数中连接 list 的例子：

```py
 return ";".join(["%s=%s" % (k, v) for k, v in params.items()]) 
```

在我们继续之前有一个有趣的地方。我一直在重复函数是对象，字符串是对象，每个东西都是对象的概念。您也许认为我的意思是说字符串*值* 是对象。但是不对，仔细地看一下这个例子，您将会看到字符串 `";"` 本身就是一个对象，您在调用它的 `join` 方法。

总之，这里的 `join` 方法将 list 中的元素连接成单个字符串，每个元素用一个分号隔开。分隔符不必是一个分号；它甚至不必是单个字符。它可以是任何字符串。

> 小心
> `join` 只能用于元素是字符串的 list；它不进行任何的强制类型转换。连接一个存在一个或多个非字符串元素的 list 将引发一个异常。

## 例 3.27\. `odbchelper.py` 的输出结果

```py
>>> params = {"server":"mpilgrim", "database":"master", "uid":"sa", "pwd":"secret"}
>>> ["%s=%s" % (k, v) for k, v in params.items()]
['server=mpilgrim', 'uid=sa', 'database=master', 'pwd=secret']
>>> ";".join(["%s=%s" % (k, v) for k, v in params.items()])
'server=mpilgrim;uid=sa;database=master;pwd=secret' 
```

上面的字符串是从 `odbchelper` 函数返回的，被调用块打印出来，这样就给出了您开始阅读本章时令人感到吃惊的输出结果。

您可能在想是否存在一个适当的方法来将字符串分割成一个 list。当然有，它叫做 `split`。

## 例 3.28\. 分割字符串

```py
>>> li = ['server=mpilgrim', 'uid=sa', 'database=master', 'pwd=secret']
>>> s = ";".join(li)
>>> s
'server=mpilgrim;uid=sa;database=master;pwd=secret'
>>> s.split(";")    
['server=mpilgrim', 'uid=sa', 'database=master', 'pwd=secret']
>>> s.split(";", 1) 
['server=mpilgrim', 'uid=sa;database=master;pwd=secret'] 
```

|  |  |
| --- | --- |
| [1] | `split` 与 `join` 正好相反，它将一个字符串分割成多元素 list。注意，分隔符 (“`;`”) 被完全去掉了，它没有在返回的 list 中的任意元素中出现。 |
| [2] | `split` 接受一个可选的第二个参数，它是要分割的次数。(“哦，可选参数……”，您将会在下一章中学会如何在您自己的函数中使用它。) |

> 提示
> `_anystring_.`split`(_delimiter_, 1)` 是一个有用的技术，在您想要搜索一个子串，然后分别处理字符前半部分 (即 list 中第一个元素) 和后半部分 (即 list 中第二个元素) 时，使用这个技术。

## 进一步阅读

*   Python Knowledge Base 回答了[关于字符串的常见问题](http://www.faqts.com/knowledge-base/index.phtml/fid/480)，并且有许多[使用字符串的例子代码](http://www.faqts.com/knowledge-base/index.phtml/fid/539)。
*   *Python Library Reference* 总结了[所有字符串方法](http://www.python.org/doc/current/lib/string-methods.html)。
*   *Python Library Reference* 提供了 [`string` 模块](http://www.python.org/doc/current/lib/module-string.html)的文档。
*   *The Whole Python FAQ* 解释了[为什么 `join` 是字符串方法](http://www.python.org/cgi-bin/faqw.py?query=4.96&querytype=simple&casefold=yes&req=search)而不是 list 方法。

## 3.7.1\. 字符串方法的历史注解

当我开始学 Python 时，我以为 `join` 是 list 的方法，它会使用分隔符作为一个参数。很多人都有同样的感觉：在 `join` 方法的背后有一段故事。在 Python 1.6 之前，字符串完全没有这些有用的方法。有一个独立的 `string` 模块包含所有的字符串函数，每个函数使用一个字符串作为它的第一个参数。这些函数被认为足够重要，所以它们移到字符串中去了，这就使得诸如 `lower`、`upper` 和 `split` 之类的函数是有意义的。但许多核心的 Python 程序员反对新的 `join` 方法，争论说应该换成是 list 的一个方法，或不应该移动而仅仅保留为旧的 `string` 模块 (现仍然还有许多有用的东西在里面) 的一部分。我只使用新的 `join` 方法，但是您还是会看到其它写法。如果它真的使您感到麻烦，您可以使用旧的 `string.join` 函数来替代。

# 3.8\. 小结

# 3.8\. 小结

现在 `odbchelper.py` 程序和它的输出结果都应该非常清楚了。

```py
 def buildConnectionString(params):
    """Build a connection string from a dictionary of parameters.
    Returns string."""
    return ";".join(["%s=%s" % (k, v) for k, v in params.items()])
if __name__ == "__main__":
    myParams = {"server":"mpilgrim", \
                "database":"master", \
                "uid":"sa", \
                "pwd":"secret" \
                }
    print buildConnectionString(myParams) 
```

下面是 `odbchelper.py` 的输出结果：

```py
server=mpilgrim;uid=sa;database=master;pwd=secret 
```

在深入下一章学习之前，确保您可以无阻碍地完成下面的事情：

*   使用 Python IDE 来交互式地测试表达式
*   编写 Python 程序并且从 IDE 运行，或者从命令行运行
*   导入模块及调用它们的函数
*   声明函数以及 `doc string`、局部变量和适当的缩进的使用
*   定义 dictionary、tuple 和 list
*   任意一个对象的访问方法，包括：字符串、list、dictionary、函数和模块
*   通过字符串格式化连接值
*   使用 list 解析映射 list 为其他的 list
*   把字符串分割为 list 和把 list 连接为字符串