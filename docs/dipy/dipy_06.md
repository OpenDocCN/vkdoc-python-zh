# 第五章 对象和面向对象

# 第五章 对象和面向对象

*   5.1\. 概览
*   5.2\. 使用 from module import 导入模块
*   5.3\. 类的定义
    *   5.3.1\. 初始化并开始类编码
    *   5.3.2\. 了解何时去使用 self 和 **init**
*   5.4\. 类的实例化
    *   5.4.1\. 垃圾回收
*   5.5\. 探索 UserDict：一个封装类
*   5.6\. 专用类方法
    *   5.6.1\. 获得和设置数据项
*   5.7\. 高级专用类方法
*   5.8\. 类属性介绍
*   5.9\. 私有函数
*   5.10\. 小结

这一章，和此后的许多章，均讨论了面向对象的 Python 程序设计。

# 5.1\. 概览

# 5.1\. 概览

下面是一个完整的，可运行的 Python 程序。请阅读模块、类和函数的 `doc string`s，可以大概了解这个程序所做的事情和工作情况。像平时一样，不用担心你不理解的东西，这就是本章其它部分将告诉你的内容。

## 例 5.1\. `fileinfo.py`

如果您还没有下载本书附带的样例程序, 可以 [下载本程序和其他样例程序](http://www.woodpecker.org.cn/diveintopython/download/diveintopython-exampleszh-cn-5.4b.zip "Download example scripts")。

```py
"""Framework for getting filetype-specific metadata.
Instantiate appropriate class with filename.  Returned object acts like a
dictionary, with key-value pairs for each piece of metadata.
    import fileinfo
    info = fileinfo.MP3FileInfo("/music/ap/mahadeva.mp3")
    print "\\n".join(["%s=%s" % (k, v) for k, v in info.items()])
Or use listDirectory function to get info on all files in a directory.
    for info in fileinfo.listDirectory("/music/ap/", [".mp3"]):
        ...
Framework can be extended by adding classes for particular file types, e.g.
HTMLFileInfo, MPGFileInfo, DOCFileInfo.  Each class is completely responsible for
parsing its files appropriately; see MP3FileInfo for example.
"""
import os
import sys
from UserDict import UserDict
def stripnulls(data):
    "strip whitespace and nulls"
    return data.replace("\00", "").strip()
class FileInfo(UserDict):
    "store file metadata"
    def __init__(self, filename=None):
        UserDict.__init__(self)
        self["name"] = filename
class MP3FileInfo(FileInfo):
    "store ID3v1.0 MP3 tags"
    tagDataMap = {"title"   : (  3,  33, stripnulls),
                  "artist"  : ( 33,  63, stripnulls),
                  "album"   : ( 63,  93, stripnulls),
                  "year"    : ( 93,  97, stripnulls),
                  "comment" : ( 97, 126, stripnulls),
                  "genre"   : (127, 128, ord)}
    def __parse(self, filename):
        "parse ID3v1.0 tags from MP3 file"
        self.clear()
        try:                               
            fsock = open(filename, "rb", 0)
            try:                           
                fsock.seek(-128, 2)        
                tagdata = fsock.read(128)  
            finally:                       
                fsock.close()              
            if tagdata[:3] == "TAG":
                for tag, (start, end, parseFunc) in self.tagDataMap.items():
                    self[tag] = parseFunc(tagdata[start:end])               
        except IOError:                    
            pass                           
    def __setitem__(self, key, item):
        if key == "name" and item:
            self.__parse(item)
        FileInfo.__setitem__(self, key, item)
def listDirectory(directory, fileExtList):                                        
    "get list of file info objects for files of particular extensions"
    fileList = [os.path.normcase(f)
                for f in os.listdir(directory)]           
    fileList = [os.path.join(directory, f) 
               for f in fileList
                if os.path.splitext(f)[1] in fileExtList] 
    def getFileInfoClass(filename, module=sys.modules[FileInfo.__module__]):      
        "get file info class from filename extension"                             
        subclass = "%sFileInfo" % os.path.splitext(filename)[1].upper()[1:]       
        return hasattr(module, subclass) and getattr(module, subclass) or FileInfo
    return [getFileInfoClass(f)(f) for f in fileList]                             
if __name__ == "__main__":
    for info in listDirectory("/music/_singles/", [".mp3"]): 
        print "\n".join(["%s=%s" % (k, v) for k, v in info.items()])
        print 
```

|  |  |
| --- | --- |
| [1] | 这个程序的输入要取决于你硬盘上的文件。为了得到有意义的输出，你应该修改目录路径指向你自已机器上的一个 MP3 文件目录。 |

下面就是从我的机器上得到的输出。你的输出将不一样，除非，由于某些令人吃惊的巧合，你与我有着共同的音乐品味。

```py
album=
artist=Ghost in the Machine
title=A Time Long Forgotten (Concept
genre=31
name=/music/_singles/a_time_long_forgotten_con.mp3
year=1999
comment=http://mp3.com/ghostmachine
album=Rave Mix
artist=***DJ MARY-JANE***
title=HELLRAISER****Trance from Hell
genre=31
name=/music/_singles/hellraiser.mp3
year=2000
comment=http://mp3.com/DJMARYJANE
album=Rave Mix
artist=***DJ MARY-JANE***
title=KAIRO****THE BEST GOA
genre=31
name=/music/_singles/kairo.mp3
year=2000
comment=http://mp3.com/DJMARYJANE
album=Journeys
artist=Masters of Balance
title=Long Way Home
genre=31
name=/music/_singles/long_way_home1.mp3
year=2000
comment=http://mp3.com/MastersofBalan
album=
artist=The Cynic Project
title=Sidewinder
genre=18
name=/music/_singles/sidewinder.mp3
year=2000
comment=http://mp3.com/cynicproject
album=Digitosis@128k
artist=VXpanded
title=Spinning
genre=255
name=/music/_singles/spinning.mp3
year=2000
comment=http://mp3.com/artists/95/vxp 
```

# 5.2\. 使用 `from _module_ import` 导入模块

# 5.2\. 使用 `from _module_ import` 导入模块

Python 有两种导入模块的方法。两种都有用，你应该知道什么时候使用哪一种方法。一种方法，`import _module_`，你已经在第 2.4 节 “万物皆对象”看过了。另一种方法完成同样的事情，但是它与第一种有着细微但重要的区别。

下面是 `from _module_ import` 的基本语法：

```py
 from UserDict import UserDict 
```

它与你所熟知的 `import _module_` 语法很相似，但是有一个重要的区别：`UserDict` 被直接导入到局部名字空间去了，所以它可以直接使用，而不需要加上模块名的限定。你可以导入独立的项或使用 `from _module_ import *` 来导入所有东西。

> 注意
> Python 中的 `from _module_ import *` 像 Perl 中的 `use _module_` ；Python 中的 `import _module_` 像 Perl 中的 `require _module_` 。
> 
> 注意
> Python 中的 `from _module_ import *` 像 Java 中的 `import _module_.*` ；Python 中的 `import _module_` 像 Java 中的 `import _module_` 。

## 例 5.2\. `import _module_` *vs.* `from _module_ import`

```py
>>> import types
>>> types.FunctionType             
<type 'function'>
>>> FunctionType                   
Traceback (innermost last):
  File "<interactive input>", line 1, in ?
NameError: There is no variable named 'FunctionType'
>>> from types import FunctionType 
>>> FunctionType                   
<type 'function'> 
```

|  |  |
| --- | --- |
| [1] | `types` 模块不包含方法，只是表示每种 Python 对象类型的属性。注意这个属性必需用模块名 `types` 进行限定。 |
| [2] | `FunctionType` 本身没有被定义在当前名字空间中；它只存在于 `types` 的上下文环境中。 |
| [3] | 这个语法从 `types` 模块中直接将 `FunctionType` 属性导入到局部名字空间中。 |
| [4] | 现在 `FunctionType` 可以直接使用，与 `types` 无关了。 |

什么时候你应该使用 `from _module_ import`？

*   如果你要经常访问模块的属性和方法，且不想一遍又一遍地敲入模块名，使用 `from _module_ import`。
*   如果你想要有选择地导入某些属性和方法，而不想要其它的，使用 `from _module_ import`。
*   如果模块包含的属性和方法与你的某个模块同名，你必须使用 `import _module_` 来避免名字冲突。

除了这些情况，剩下的只是风格问题了，你会看到用两种方式编写的 Python 代码。

> 小心
> 尽量少用 `from module import *` ，因为判定一个特殊的函数或属性是从哪来的有些困难，并且会造成调试和重构都更困难。

## 进一步阅读关于模块导入技术

*   eff-bot 有更多关于 [`import _module_` *vs.* `from _module_ import`](http://www.effbot.org/guides/import-confusion.htm) 的论述。
*   *Python Tutorial* 讨论了高级的导入技术，包括 [`from _module_ import *`](http://www.python.org/doc/current/tut/node8.html#SECTION008410000000000000000)。

# 5.3\. 类的定义

# 5.3\. 类的定义

*   5.3.1\. 初始化并开始类编码
*   5.3.2\. 了解何时去使用 self 和 **init**

Python 是完全面向对象的：你可以定义自已的类，从自已的或内置的类继承，然后从你定义的类创建实例。

在 Python 中定义类很简单。就像定义函数，没有单独的接口定义。只要定义类，然后就可以开始编码。Python 类以保留字 `class` 开始，后面跟着类名。从技术上讲，有这些就够了，因为一个类并非必须从其它类继承。

## 例 5.3\. 最简单的 Python 类

```py
 class Loaf: 
    pass 
```

|  |  |
| --- | --- |
| [1] | 这个类的名字是 `Loaf`，它没有从其它类继承。类名通常是第一个字母大写，如：`EachWordLikeThis`，但这只是一个习惯，不是一个必要条件。 |
| [2] | 这个类没有定义任何方法或属性，但是从语法上，需要在定义中有些东西，所以你使用 `pass`。这是一个 Python 保留字，仅仅表示 “向前走，不要往这看”。它是一条什么都不做的语句，当你删空函数或类时，它是一个很好的占位符。 |
| [3] | 你可能猜到了，在类中的所有东西都要缩近，就像位于函数、`if` 语句，`for` 循环，诸如此类的代码。第一条不缩近的东西不属于这个类。 |

> 注意
> 在 Python 中的 `pass` 语句就像 Java 或 C 中的大括号空集 (`{}`)。

当然，实际上大多数的类都是从其它的类继承来的，并且它们会定义自已的类方法和属性。但是就像你刚才看到的，除了名字以外，类没有什么必须要具有的。特别是，C++ 程序员可能会感到奇怪，Python 的类没有显示的构造函数和析构函数。Python 类的确存在与构造函数相似的东西：`__init__` 方法。

## 例 5.4\. 定义 `FileInfo` 类

```py
 from UserDict import UserDict
class FileInfo(UserDict): 
```

|  |  |
| --- | --- |
| [1] | 在 Python 中，类的基类只是简单地列在类名后面的小括号里。所以 `FileInfo` 类是从 `UserDict` 类 (它是从 `UserDict` 模块导进来的) 继承来的。`UserDict` 是一个像字典一样工作的类，它允许你完全子类化字典数据类型，同时增加你自已的行为。{也存在相似的类 `UserList` 和 `UserString` ，它们允许你子类化列表和字符串。)[2] 在这个类的背后有一些“巫术”，我们将在本章的后面，随着更进一步地研究 `UserDict` 类，揭开这些秘密。 |

> 注意
> 在 Python 中，类的基类只是简单地列在类名后面的小括号里。不像在 Java 中有一个特殊的 `extends` 关键字。

Python 支持多重继承。在类名后面的小括号中，你可以列出许多你想要的类名，以逗号分隔。

## 5.3.1\. 初始化并开始类编码

本例演示了使用 `__init__` 方法来进行 `FileInfo` 类的初始化。

## 例 5.5\. 初始化 `FileInfo` 类

```py
 class FileInfo(UserDict):
    "store file metadata"              
    def __init__(self, filename=None): 
```

|  |  |
| --- | --- |
| [1] | 类也可以 (并且应该) 有 `doc string`s ，就像方法和函数一样。 |
| [2] | `__init__` 在类的实例创建后被立即调用。它可能会引诱你称之为类的构造函数，但这种说法并不正确。说它引诱，是因为它看上去像 (按照习惯，`__init__` 是类中第一个定义的方法)，行为也像 (在一个新创建的类实例中，它是首先被执行的代码)，并且叫起来也像 (“init”当然意味着构造的本性)。说它不正确，是因为对象在调用 `__init__` 时已经被构造出来了，你已经有了一个对类的新实例的有效引用。但 `__init__` 是在 Python 中你可以得到的最接近构造函数的东西，并且它也扮演着非常相似的角色。 |
| [3] | 每个类方法的第一个参数，包括 `__init__`，都是指向类的当前实例的引用。按照习惯这个参数总是被称为 `self`。在 `__init__` 方法中，`self` 指向新创建的对象；在其它的类方法中，它指向方法被调用的类实例。尽管当定义方法时你需要明确指定 `self`，但在调用方法时，你*不* 用指定它，Python 会替你自动加上的。 |
| [4] | `__init__` 方法可以接受任意数目的参数，就像函数一样，参数可以用缺省值定义，即可以设置成对于调用者可选。在本例中，`filename` 有一个缺省值 `None`，即 Python 的空值。 |

> 注意
> 习惯上，任何 Python 类方法的第一个参数 (对当前实例的引用) 都叫做 `self`。这个参数扮演着 C++ 或 Java 中的保留字 `this` 的角色，但 `self` 在 Python 中并不是一个保留字，它只是一个命名习惯。虽然如此，也请除了 `self` 之外不要使用其它的名字，这是一个非常坚固的习惯。

## 例 5.6\. 编写 `FileInfo` 类

```py
 class FileInfo(UserDict):
    "store file metadata"
    def __init__(self, filename=None):
        UserDict.__init__(self)        
        self["name"] = filename 
```

|  |  |
| --- | --- |
| [1] | 一些伪面向对象语言，像 Powerbuilder 有一种“扩展”构造函数和其它事件的概念，即父类的方法在子类的方法执行前被自动调用。Python 不是这样，你必须显示地调用在父类中的合适方法。 |
| [2] | 我告诉过你，这个类像字典一样工作，那么这里就是第一个印象。我们将参数 `filename` 赋值给对象 `name` 关键字，作为它的值。 |
| [3] | 注意 `__init__` 方法从不返回一个值。 |

## 5.3.2\. 了解何时去使用 `self` 和 `__init__`

当定义你自已的类方法时，你*必须* 明确将 `self` 作为每个方法的第一个参数列出，包括 `__init__`。当从你的类中调用一个父类的一个方法时，你必须包括 `self` 参数。但当你从类的外部调用你的类方法时，你不必对 `self` 参数指定任何值；你完全将其忽略，而 Python 会自动地替你增加实例的引用。我知道刚开始这有些混乱，它并不是自相矛盾的，因为它依靠于一个你还不了解的区别 (在绑定与非绑定方法之间)，故看上去是矛盾的。

噢。我知道有很多知识需要吸收，但是你要掌握它。所有的 Python 类以相同的方式工作，所以一旦你学会了一个，就是学会了全部。如果你忘了别的任何事，也要记住这件事，因为我认定它会让你出错：

> 注意
> `__init__` 方法是可选的，但是一旦你定义了，就必须记得显示调用父类的 `__init__` 方法 (如果它定义了的话)。这样更是正确的：无论何时子类想扩展父类的行为，后代方法必须在适当的时机，使用适当的参数，显式调用父类方法。

## 进一步阅读关于 Python 类

*   *Learning to Program* 有优雅的[类的介绍](http://www.freenetpages.co.uk/hp/alan.gauld/tutclass.htm)。
*   *How to Think Like a Computer Scientist* 展示了如何[使用类来实现复合数据类型模型](http://www.ibiblio.org/obp/thinkCSpy/chap12.htm)。
*   *Python Tutorial* 深入考虑了[类、名字空间和继承](http://www.python.org/doc/current/tut/node11.html)。
*   Python Knowledge Base 回答了[关于类的常见问题](http://www.faqts.com/knowledge-base/index.phtml/fid/242)。

## Footnotes

[2] 在 2.2 之后已经可以从 dict、list 来派生子类了，关于这一点作者在后文也会提到。――译注

# 5.4\. 类的实例化

# 5.4\. 类的实例化

*   5.4.1\. 垃圾回收

在 Python 中对类进行实例化很直接。要对类进行实例化，只要调用类 (就好像它是一个函数)，传入定义在 `__init__` 方法中的参数。返回值将是新创建的对象。

## 例 5.7\. 创建 `FileInfo` 实例

```py
>>> import fileinfo
>>> f = fileinfo.FileInfo("/music/_singles/kairo.mp3") 
>>> f.__class__                                        
<class fileinfo.FileInfo at 010EC204>
>>> f.__doc__                                          
'store file metadata'
>>> f                                                  
{'name': '/music/_singles/kairo.mp3'} 
```

|  |  |
| --- | --- |
| [1] | 你正在创建 `FileInfo` 类 (定义在 `fileinfo` 模块中) 的实例，并且将新创建的实例赋值给变量 `f`。你传入了一个参数，`/music/_singles/kairo.mp3`，它将最后作为在 `FileInfo` 中 `__init__` 方法中的 `filename` 参数。 |
| [2] | 每一个类的实例有一个内置属性，`__class__`，它是对象的类。(注意这个表示包括了在我机器上的实例的物理地址，你的表示不会一样。)Java 程序员可能对 `Class` 类熟悉，这个类包含了像 `getName` 和 `getSuperclass` 之类用来得到一个对象元数据信息的方法。在 Python 中，这类元数据可以直接通过对象本身的属性，像 `__class__`、`__name__` 和 `__bases__` 来得到。 |
| [3] | 你可以像对函数或模块一样来访问实例的 `doc string`。一个类的所有实例共享相同的 `doc string`。 |
| [4] | 还记得什么时候 `__init__` 方法[将它的 `filename` 参数赋给 `self["name"]`](defining_classes.html#fileinfo.class.example "例 5.4\. 定义 FileInfo 类") 吗？哦，答案在这。在创建类实例时你传入的参数被正确发送到 `__init__` 方法中 (当我们创建类实例时，我们所传递的参数被正确地发送给 `__init__` 方法 (随同一起传递的还有对象的引用，`self`，它是由 Python 自动添加的)。 |

> 注意
> 在 Python 中，创建类的实例只要调用一个类，仿佛它是一个函数就行了。不像 C++ 或 Java 有一个明确的 `new` 操作符。

## 5.4.1\. 垃圾回收

如果说创建一个新的实例是容易的，那么销毁它们甚至更容易。通常，不需要明确地释放实例，因为当指派给它们的变量超出作用域时，它们会被自动地释放。内存泄漏在 Python 中很少见。

## 例 5.8\. 尝试实现内存泄漏

```py
>>> def leakmem():
... f = fileinfo.FileInfo('/music/_singles/kairo.mp3') 
... 
>>> for i in range(100):
... leakmem() 
```

|  |  |
| --- | --- |
| [1] | 每次 `leakmem` 函数被调用，你创建了 `FileInfo` 的一个实例，将其赋给变量 `f`，这个变量是函数内的一个局部变量。然后函数结束时没有释放 `f`，所以你可能认为有内存泄漏，但是你错了。当函数结束时，局部变量 `f` 超出了作用域。在这个地方，不再有任何对 `FileInfo` 新创建实例的引用 (因为除了 `f` 我们从未将其赋值给其它变量)，所以 Python 替我们销毁掉实例。 |
| [2] | 不管我们调用 `leakmem` 函数多少次，决不会泄漏内存，因为每一次，Python 将在从 `leakmem` 返回前销毁掉新创建的 `FileInfo` 类实例。 |

对于这种垃圾收集的方式，技术上的术语叫做“引用计数”。Python 维护着对每个实例的引用列表。在上面的例子中，只有一个 `FileInfo` 的实例引用：局部变量 `f`。当函数结束时，变量 `f` 超出作用域，所以引用计数降为 `0`，则 Python 自动销毁掉实例。

在 Python 的以前版本中，存在引用计数失败的情况，这样 Python 不能在后面进行清除。如果你创建两个实例，它们相互引用 (例如，双重链表，每一个结点有都一个指向列表中前一个和后一个结点的指针)，任一个实例都不会被自动销毁，因为 Python (正确) 认为对于每个实例都存在一个引用。Python 2.0 有一种额外的垃圾回收方式，叫做“标记后清除”，它足够聪明，可以正确地清除循环引用。

作为曾经读过哲学专业的一员，让我感到困惑的是，当没有人对事物进行观察时，它们就消失了，但是这确实是在 Python 中所发生的。通常，你可以完全忘记内存管理，让 Python 在后面进行清理。

## 进一步阅读

*   *Python Library Reference* 总结了[像 `__class__` 之类的内置属性](http://www.python.org/doc/current/lib/specialattrs.html)。
*   *Python Library Reference* 提供了 [`gc` 模块的文档](http://www.python.org/doc/current/lib/module-gc.html)，此模块给予你对 Python 的垃圾回收的底层控制权。

# 5.5\. 探索 `UserDict`：一个封装类

# 5.5\. 探索 `UserDict`：一个封装类

如你所见，`FileInfo` 是一个有着像字典一样的行为方式的类。为了进一步揭示这一点，让我们看一看在 `UserDict` 模块中的 `UserDict` 类，它是我们的 `FileInfo` 类的父类。它没有什么特别的，也是用 Python 写的，并且保存在一个 `.py` 文件里，就像我们其他的代码。特别之处在于，它保存在你的 Python 安装目录的 `lib` 目录下。

> 提示
> 在 Windows 下的 ActivePython IDE 中，你可以快速打开在你的库路径中的任何模块，使用 File->Locate... (****Ctrl**-L**)。

## 例 5.9\. 定义 `UserDict` 类

```py
 class UserDict:                                
    def __init__(self, dict=None):             
        self.data = {}                         
        if dict is not None: self.update(dict) 
```

|  |  |
| --- | --- |
| [1] | 注意 `UserDict` 是一个基类，不是从任何其他类继承而来。 |
| [2] | 这就是我们在 `FileInfo` 类中进行了覆盖的 `__init__` 方法。注意这个父类的参数列表与子类不同。很好，每个子类可以拥有自已的参数集，只要使用正确的参数调用父类就可以了。这里父类有一个定义初始值的方法 (通过在 `dict` 参数中传入一个字典)，这一方法我们的 `FileInfo` 没有用上。 |
| [3] | Python 支持数据属性 (在 Java 和 Powerbuilder 中叫做 “实例变量”，在 C++ 中叫 “数据成员”)，它是由某个特定的类实例所拥有的数据。在本例中，每个 `UserDict` 实例将拥有一个 `data` 数据属性。要从类外的代码引用这个属性，需要用实例的名字限定它，`_instance_.data`，限定的方法与你用模块的名字来限定函数一样。要在类的内部引用一个数据属性，我们使用 `self` 作为限定符。习惯上，所有的数据属性都在 `__init__` 方法中初始化为有意义的值。然而，这并不是必须的，因为数据属性，像局部变量一样，当你首次赋给它值的时候突然产生。 |
| [4] | `update` 方法是一个字典复制器：它把一个字典中的键和值全部拷贝到另一个字典。这个操作*并不* 事先清空目标字典，如果一些键在目标字典中已经存在，则它们将被覆盖，那些键名在目标字典中不存在的则不改变。应该把 `update` 看作是合并函数，而不是复制函数。 |
| [5] | 这个语法你可能以前没看过 (我还没有在这本书中的例子中用过它)。这是一条 `if` 语句，但是没有在下一行有一个缩近块，而只是在冒号后面，在同一行上有单条语句。这完全是合法的，它只是当你在一个块中仅有一条语句时的一个简写。(它就像在 C++ 中没有用大括号包括的单行语句。) 你可以用这种语法，或者可以在后面的行写下缩近代码，但是不能对同一个块同时用两种方式。 |

> 注意
> Java 和 Powerbuilder 支持通过参数列表的重载，*也就是* 一个类可以有同名的多个方法，但这些方法或者是参数个数不同，或者是参数的类型不同。其它语言 (最明显如 PL/SQL) 甚至支持通过参数名的重载，*也就是* 一个类可以有同名的多个方法，这些方法有相同类型，相同个数的参数，但参数名不同。Python 两种都不支持，总之是没有任何形式的函数重载。一个 `__init__` 方法就是一个 `__init__` 方法，不管它有什么样的参数。每个类只能有一个 `__init__` 方法，并且如果一个子类拥有一个 `__init__` 方法，它*总是* 覆盖父类的 `__init__` 方法，甚至子类可以用不同的参数列表来定义它。
> 
> 注意
> Python 的原作者 Guido 是这样解释方法覆盖的：“子类可以覆盖父类中的方法。因为方法没有特殊的优先级设置，父类中的一个方法在调用同类中的另一方法时，可能其实调用到的却是一个子类中覆盖父类同名方法的方法。 (C++ 程序员可能会这样想：所有的 Python 方法都是虚函数。)”如果你不明白 (它令我颇感困惑)，不必在意。我想我要跳过它。[3]
> 
> 小心
> 应该总是在 `__init__` 方法中给一个实例的所有数据属性赋予一个初始值。这样做将会节省你在后面调试的时间，不必为捕捉因使用未初始化 (也就是不存在) 的属性而导致的 `AttributeError` 异常费时费力。

## 例 5.10\. `UserDict` 常规方法

```py
 def clear(self): self.data.clear()          
    def copy(self):                             
        if self.__class__ is UserDict:          
            return UserDict(self.data)         
        import copy                             
        return copy.copy(self)                 
    def keys(self): return self.data.keys()     
    def items(self): return self.data.items()  
    def values(self): return self.data.values() 
```

|  |  |
| --- | --- |
| [1] | `clear` 是一个普通的类方法，可以在任何时候被任何人公开调用。注意，`clear` 像所有的类方法一样 (常规的或专用的)，使用 `self` 作为它的第一个参数。(记住，当你调用方法时，不用包括 `self`；这件事是 Python 替你做的。) 还应注意这个封装类的基本技术：将一个真正的字典 (`data`) 作为数据属性保存起来，定义所有真正字典所拥有的方法，并且将每个类方法重定向到真正字典上的相应方法。(你可能已经忘了，字典的 `clear` 方法删除它的所有关键字和关键字相应的值。) |
| [2] | 真正字典的 `copy` 方法会返回一个新的字典，它是原始字典的原样的复制 (所有的键-值对都相同)。但是 `UserDict` 不能简单地重定向到 `self.data.copy`，因为那个方法返回一个真正的字典，而我们想要的是返回同一个类的一个新的实例，就像是 `self`。 |
| [3] | 我们使用 `__class__` 属性来查看 `self` 是否是一个 `UserDict`，如果是，太好了，因为我们知道如何拷贝一个 `UserDict`：只要创建一个新的 `UserDict` ，并传给它真正的字典，这个字典已经存放在 `self.data` 中了。然后你立即返回这个新的 `UserDict`，你甚至于不需要在下面一行中使用 `import copy`。 |
| [4] | 如果 `self.__class__` 不是 `UserDict`，那么 `self` 一定是 `UserDict` 的某个子类 (如可能为 `FileInfo`)，生活总是存在意外。`UserDict` 不知道如何生成它的子类的一个原样的拷贝，例如，有可能在子类中定义了其它的数据属性，所以我们只能完全复制它们，确定拷贝了它们的全部内容。幸运的是，Python 带了一个模块可以正确地完成这件事，它叫做 `copy`。在这里我不想深入细节 (然而它是一个绝对酷的模块，你是否已经想到要自已研究它了呢？)。说 `copy` 能够拷贝任何 Python 对象就够了，这就是我们在这里用它的原因。 |
| [5] | 其余的方法是直截了当的重定向到 `self.data` 的内置函数上。 |

> 注意
> 在 Python 2.2 之前的版本中，你不可以直接子类化字符串、列表以及字典之类的内建数据类型。作为补偿，Python 提供封装类来模拟内建数据类型的行为，比如：`UserString`、`UserList` 和 `UserDict`。通过混合使用普通和特殊方法，`UserDict` 类能十分出色地模仿字典。在 Python 2.2 和其后的版本中，你可以直接从 `dict` 内建数据类型继承。本书 `fileinfo_fromdict.py` 中有这方面的一个例子。

如例子中所示，在 Python 中，你可以直接继承自内建数据类型 `dict`，这样做有三点与 `UserDict` 不同。

## 例 5.11\. 直接继承自内建数据类型 `dict`

```py
 class FileInfo(dict):                  
    "store file metadata"
    def __init__(self, filename=None): 
        self["name"] = filename 
```

|  |  |
| --- | --- |
| [1] | 第一个区别是你不需要导入 `UserDict` 模块，因为 `dict` 是已经可以使用的内建数据类型。第二个区别是你不是继承自 `UserDict.UserDict` ，而是直接继承自 `dict`。 |
| [2] | 第三个区别有些晦涩，但却很重要。`UserDict` 内部的工作方式要求你手工地调用它的 `__init__` 方法去正确初始化它的内部数据结构。`dict` 并不这样工作，它不是一个封装所以不需要明确的初始化。 |

## 进一步阅读

*   *Python Library Reference* 提供了 [`UserDict` 模块](http://www.python.org/doc/current/lib/module-UserDict.html) 和 [`copy` 模块](http://www.python.org/doc/current/lib/module-copy.html) 的文档。

## Footnotes

[3] 实际上，这一点并不是那么难以理解。考虑两个类，`base` 和 `child`，`base` 中的方法 `a` 需要调用 `self.b`；而我们又在 `child` 中覆盖了方法 `b`。然后我们创建一个 `child` 的实例，`ch`。调用 `ch.a`，那么此时的方法 `a` 调用的 `b` 函数将不是 `base.b`，而是 `child.b`。――译注

# 5.6\. 专用类方法

# 5.6\. 专用类方法

*   5.6.1\. 获得和设置数据项

除了普通的类方法，Python 类还可以定义专用方法。专用方法是在特殊情况下或当使用特别语法时由 Python 替你调用的，而不是在代码中直接调用 (像普通的方法那样)。

就像你在上一节所看到的，普通的方法对在类中封装字典很有帮助。但是只有普通方法是不够的，因为除了对字典调用方法之外，还有很多事情可以做的。例如，你可以通过一种没有包括明确方法调用的语法来获得和设置数据项。这就是专用方法产生的原因：它们提供了一种方法，可以将非方法调用语法映射到方法调用上。

## 5.6.1\. 获得和设置数据项

## 例 5.12\. `__getitem__` 专用方法

```py
 def __getitem__(self, key): return self.data[key] 
```

```py
>>> f = fileinfo.FileInfo("/music/_singles/kairo.mp3")
>>> f
{'name':'/music/_singles/kairo.mp3'}
>>> f.__getitem__("name") 
'/music/_singles/kairo.mp3'
>>> f["name"]             
'/music/_singles/kairo.mp3' 
```

|  |  |
| --- | --- |
| [1] | `__getitem__` 专用方法很简单。像普通的方法 `clear`，`keys` 和 `values` 一样，它只是重定向到字典，返回字典的值。但是怎么调用它呢？哦，你可以直接调用 `__getitem__`，但是在实际中你其实不会那样做：我在这里执行它只是要告诉你它是如何工作的。正确地使用 `__getitem__` 的方法是让 Python 来替你调用。 |
| [2] | 这个看上去就像你用来得到一个字典值的语法，事实上它返回你期望的值。下面是隐藏起来的一个环节：暗地里，Python 已经将这个语法转化为 `f.__getitem__("name")` 的方法调用。这就是为什么 `__getitem__` 是一个专用类方法的原因，不仅仅是你可以自已调用它，还可以通过使用正确的语法让 Python 来替你调用。 |

当然，Python 有一个与 `__getitem__` 类似的 `__setitem__` 专用方法，参见下面的例子。

## 例 5.13\. `__setitem__` 专用方法

```py
 def __setitem__(self, key, item): self.data[key] = item 
```

```py
>>> f
{'name':'/music/_singles/kairo.mp3'}
>>> f.__setitem__("genre", 31) 
>>> f
{'name':'/music/_singles/kairo.mp3', 'genre':31}
>>> f["genre"] = 32            
>>> f
{'name':'/music/_singles/kairo.mp3', 'genre':32} 
```

|  |  |
| --- | --- |
| [1] | 与 `__getitem__` 方法一样，`__setitem__` 简单地重定向到真正的字典 `self.data` ，让它来进行工作。并且像 `__getitem__` 一样，通常你不会直接调用它，当你使用了正确的语法，Python 会替你调用 `__setitem__` 。 |
| [2] | 这个看上去像正常的字典语法，当然除了 `f` 实际上是一个类，它尽可能地打扮成一个字典，并且 `__setitem__` 是打扮的一个重点。这行代码实际上暗地里调用了 `f.__setitem__("genre", 32)`。 |

`__setitem__` 是一个专用类方法，因为它可以让 Python 来替你调用，但是它仍然是一个类方法。就像在 `UserDict` 中定义 `__setitem__` 方法一样容易，我们可以在子类中重新定义它，对父类的方法进行覆盖。这就允许我们定义出在某些方面像字典一样动作的类，但是可以定义它自已的行为，超过和超出内置的字典。

这个概念是本章中我们正在学习的整个框架的基础。每个文件类型可以拥有一个处理器类，这些类知道如何从一个特殊的文类型得到元数据。只要知道了某些属性 (像文件名和位置)，处理器类就知道如何自动地得到其它的属性。它的实现是通过覆盖 `__setitem__` 方法，检查特别的关键字，然后当找到后加入额外的处理。

例如，`MP3FileInfo` 是 `FileInfo` 的子类。在设置了一个 `MP3FileInfo` 类的 `name` 时，并不只是设置 `name` 关键字 (像父类 `FileInfo` 所做的)，它还要在文件自身内进行搜索 MP3 的标记然后填充一整套关键字。下面的例子将展示其工作方式。

## 例 5.14\. 在 `MP3FileInfo` 中覆盖 `__setitem__`

```py
 def __setitem__(self, key, item):         
        if key == "name" and item:            
            self.__parse(item)                
        FileInfo.__setitem__(self, key, item) 
```

|  |  |
| --- | --- |
| [1] | 注意我们的 `__setitem__` 方法严格按照与父类方法相同的形式进行定义。这一点很重要，因为 Python 将替你执行方法，而它希望这个函数用确定个数的参数进行定义。(从技术上说，参数的名字没有关系，只是个数。) |
| [2] | 这里就是整个 `MP3FileInfo` 类的难点：如果给 `name` 关键字赋一个值，我们还想做些额外的事情。 |
| [3] | 我们对 `name` 所做的额外处理封装在了 `__parse` 方法中。这是定义在 `MP3FileInfo` 中的另一个类方法，则当我们调用它时，我们用 `self` 对其限定。仅是调用 `__parse` 将只会看成定义在类外的普通方法，调用 `self.__parse` 将会看成定义在类中的一个类方法。这不是什么新东西，你用同样的方法来引用数据属性。 |
| [4] | 在做完我们额外的处理之后，我们需要调用父类的方法。记住，在 Python 中不会自动为你完成，需手工执行。注意，我们在调用直接父类，`FileInfo`，尽管它没有 `__setitem__` 方法。没问题，因为 Python 将会沿着父类树走，直到它找到一个拥有我们正在调用方法的类，所以这行代码最终会找到并且调用定义在 `UserDict` 中的 `__setitem__`。 |

> 注意
> 当在一个类中存取数据属性时，你需要限定属性名：`self._attribute_`。当调用类中的其它方法时，你属要限定方法名：`self._method_`。

## 例 5.15\. 设置 `MP3FileInfo` 的 `name`

```py
>>> import fileinfo
>>> mp3file = fileinfo.MP3FileInfo()                   
>>> mp3file
{'name':None}
>>> mp3file["name"] = "/music/_singles/kairo.mp3"      
>>> mp3file
{'album': 'Rave Mix', 'artist': '***DJ MARY-JANE***', 'genre': 31,
'title': 'KAIRO****THE BEST GOA', 'name': '/music/_singles/kairo.mp3',
'year': '2000', 'comment': 'http://mp3.com/DJMARYJANE'}
>>> mp3file["name"] = "/music/_singles/sidewinder.mp3" 
>>> mp3file
{'album': '', 'artist': 'The Cynic Project', 'genre': 18, 'title': 'Sidewinder', 
'name': '/music/_singles/sidewinder.mp3', 'year': '2000', 
'comment': 'http://mp3.com/cynicproject'} 
```

|  |  |
| --- | --- |
| [1] | 首先，我们创建了一个 `MP3FileInfo` 的实例，没有传递给它文件名。(我们可以不用它，因为 `__init__` 方法的 `filename` 参数是可选的。) 因为 `MP3FileInfo` 没有它自已的 `__init__` 方法，Python 沿着父类树走，发现了 `FileInfo` 的 `__init__` 方法。这个 `__init__` 方法手工调用了 `UserDict` 的 `__init__` 方法，然后设置 `name` 关键字为 `filename`，它为 `None`，因为我们还没有传入一个文件名。所以，`mp3file` 最初看上去像是有一个关键字的字典，`name` 的值为 `None`。 |
| [2] | 现在真正有趣的开始了。设置 `mp3file` 的 `name` 关键字触发了 `MP3FileInfo` 上的 `__setitem__` 方法 (而不是 `UserDict` 的)，这个方法注意到我们正在用一个真实的值来设置 `name` 关键字，接着调用 `self.__parse`。尽管我们完全还没有研究过 `__parse` 方法，从它的输出你可以看出，它设置了其它几个关键字：`album`、`artist`、`genre`、`title`、`year` 和 `comment`。 |
| [3] | 修改 `name` 关键字将再次经受同样的处理过程：Python 调用 `__setitem__`，`__setitem__`调用 `self.__parse`，`self.__parse` 设置其它所有的关键字。 |

# 5.7\. 高级专用类方法

# 5.7\. 高级专用类方法

除了 `__getitem__` 和 `__setitem__` 之外 Python 还有更多的专用函数。某些可以让你模拟出你甚至可能不知道的功能。

下面的例子将展示 `UserDict` 一些其他专用方法。

## 例 5.16\. `UserDict` 中更多的专用方法

```py
 def __repr__(self): return repr(self.data)     
    def __cmp__(self, dict):                       
        if isinstance(dict, UserDict):            
            return cmp(self.data, dict.data)      
        else:                                     
            return cmp(self.data, dict)           
    def __len__(self): return len(self.data)       
    def __delitem__(self, key): del self.data[key] 
```

|  |  |
| --- | --- |
| [1] | `__repr__` 是一个专用的方法，在当调用 `repr(_instance_)` 时被调用。`repr` 函数是一个内置函数，它返回一个对象的字符串表示。它可以用在任何对象上，不仅仅是类的实例。你已经对 `repr` 相当熟悉了，尽管你不知道它。在交互式窗口中，当你只敲入一个变量名，接着按**ENTER**，Python 使用 `repr` 来显示变量的值。自已用一些数据来创建一个字典 `d` ，然后用 `print repr(d)` 来看一看吧。 |
| [2] | `__cmp__` 在比较类实例时被调用。通常，你可以通过使用 `==` 比较任意两个 Python 对象，不只是类实例。有一些规则，定义了何时内置数据类型被认为是相等的，例如，字典在有着全部相同的关键字和值时是相等的。对于类实例，你可以定义 `__cmp__` 方法，自已编写比较逻辑，然后你可以使用 `==` 来比较你的类，Python 将会替你调用你的 `__cmp__` 专用方法。 |
| [3] | `__len__` 在调用 `len(_instance_)` 时被调用。`len` 是一个内置函数，可以返回一个对象的长度。它可以用于任何被认为理应有长度的对象。字符串的 `len` 是它的字符个数；字典的 `len` 是它的关键字的个数；列表或序列的 `len` 是元素的个数。对于类实例，定义 `__len__` 方法，接着自已编写长度的计算，然后调用 `len(_instance_)`，Python 将替你调用你的 `__len__` 专用方法。 |
| [4] | `__delitem__` 在调用 `del _instance_[_key_]` 时调用 ，你可能记得它作为从字典中删除单个元素的方法。当你在类实例中使用 `del` 时，Python 替你调用 `__delitem__` 专用方法。 |

> 注意
> 在 Java 中，通过使用 `str1 == str2` 可以确定两个字符串变量是否指向同一块物理内存位置。这叫做*对象同一性*，在 Python 中写为 `str1 is str2`。在 Java 中要比较两个字符串值，你要使用 `str1.equals(str2)`；在 Python 中，你要使用 `str1 == str2`。某些 Java 程序员，他们已经被教授得认为，正是因为在 Java 中 `==` 是通过同一性而不是值进行比较，所以世界才会更美好。这些人要接受 Python 的这个“严重缺失”可能要花些时间。

在这个地方，你可能会想，“所有这些工作只是为了在类中做一些我可以对一个内置数据类型所做的操作”。不错，如果你能够从像字典一样的内置数据类型进行继承的话，事情就容易多了 (那样整个 `UserDict` 类将完全不需要了)。尽管你可以这样做，专用方法仍然是有用的，因为它们可以用于任何的类，而不只是像 `UserDict` 这样的封装类。

专用方法意味着*任何类* 可以像字典一样保存键-值对，只要定义 `__setitem__` 方法。任何类可以表现得像一个序列，只要定义 `__getitem__` 方法。任何定义了 `__cmp__` 方法的类可以用 `==` 进行比较。并且如果你的类表现为拥有类似长度的东西，不要定义 `GetLength` 方法，而定义 `__len__` 方法，并使用 `len(_instance_)`。

> 注意
> 其它的面向对象语言仅让你定义一个对象的物理模型 (“这个对象有 `GetLength` 方法”)，而 Python 的专用类方法像 `__len__` 允许你定义一个对象的逻辑模型 (“这个对象有一个长度”)。

Python 存在许多其它的专用方法。有一整套的专用方法，可以让类表现得象数值一样，允许你在类实例上进行加、减，以及执行其它算数操作。(关于这一点典型的例子就是表示复数的类，数值带有实数和虚数部分。) `__call__` 方法让一个类表现得像一个函数，允许你直接调用一个类实例。并且存在其它的专用函数，允许类拥有只读或只写数据属性，在后面的章节中我们会更多地谈到这些。

## 进一步阅读

*   *Python Reference Manual* 提供了[所有专用类方法](http://www.python.org/doc/current/ref/specialnames.html)的文档。

# 5.8\. 类属性介绍

# 5.8\. 类属性介绍

你已经知道了数据属性，它们是被一个特定的类实例所拥有的变量。Python 也支持类属性，它们是由类本身所拥有的。

## 例 5.17\. 类属性介绍

```py
 class MP3FileInfo(FileInfo):
    "store ID3v1.0 MP3 tags"
    tagDataMap = {"title"   : (  3,  33, stripnulls),
                  "artist"  : ( 33,  63, stripnulls),
                  "album"   : ( 63,  93, stripnulls),
                  "year"    : ( 93,  97, stripnulls),
                  "comment" : ( 97, 126, stripnulls),
                  "genre"   : (127, 128, ord)} 
```

```py
>>> import fileinfo
>>> fileinfo.MP3FileInfo            
<class fileinfo.MP3FileInfo at 01257FDC>
>>> fileinfo.MP3FileInfo.tagDataMap 
{'title': (3, 33, <function stripnulls at 0260C8D4>), 
'genre': (127, 128, <built-in function ord>), 
'artist': (33, 63, <function stripnulls at 0260C8D4>), 
'year': (93, 97, <function stripnulls at 0260C8D4>), 
'comment': (97, 126, <function stripnulls at 0260C8D4>), 
'album': (63, 93, <function stripnulls at 0260C8D4>)}
>>> m = fileinfo.MP3FileInfo()      
>>> m.tagDataMap
{'title': (3, 33, <function stripnulls at 0260C8D4>), 
'genre': (127, 128, <built-in function ord>), 
'artist': (33, 63, <function stripnulls at 0260C8D4>), 
'year': (93, 97, <function stripnulls at 0260C8D4>), 
'comment': (97, 126, <function stripnulls at 0260C8D4>), 
'album': (63, 93, <function stripnulls at 0260C8D4>)} 
```

|  |  |
| --- | --- |
| [1] | `MP3FileInfo` 是类本身，不是任何类的特别实例。 |
| [2] | `tagDataMap` 是一个类属性：字面的意思，一个类的属性。它在创建任何类实例之前就有效了。 |
| [3] | 类属性既可以通过直接对类的引用，也可以通过对类的任意实例的引用来使用。 |

> 注意
> 在 Java 中，静态变量 (在 Python 中叫类属性) 和实例变量 (在 Python 中叫数据属性) 两者都是紧跟在类定义之后定义的 (一个有 `static` 关键字，一个没有)。在 Python 中，只有类属性可以定义在这里，数据属性定义在 `__init__` 方法中。

类属性可以作为类级别的常量来使用 (这就是为什么我们在 `MP3FileInfo` 中使用它们)，但是它们不是真正的常量。你也可以修改它们。

> 注意
> 在 Python 中没有常量。如果你试图努力的话什么都可以改变。这一点满足 Python 的核心原则之一：坏的行为应该被克服而不是被取缔。如果你真正想改变 `None` 的值，也可以做到，但当无法调试的时候别来找我。

## 例 5.18\. 修改类属性

```py
>>> class counter:
... count = 0                     
... def __init__(self):
...         self.__class__.count += 1 
... 
>>> counter
<class __main__.counter at 010EAECC>
>>> counter.count                     
0
>>> c = counter()
>>> c.count                           
1
>>> counter.count
1
>>> d = counter()                     
>>> d.count
2
>>> c.count
2
>>> counter.count
2 
```

|  |  |
| --- | --- |
| [1] | `count` 是 `counter` 类的一个类属性。 |
| [2] | `__class__` 是每个类实例的一个内置属性 (也是每个类的)。它是一个类的引用，而 `self` 是一个类 (在本例中，是 `counter` 类) 的实例。 |
| [3] | 因为 `count` 是一个类属性，它可以在我们创建任何类实例之前，通过直接对类引用而得到。 |
| [4] | 创建一个类实例会调用 `__init__` 方法，它会给类属性 `count` 加 `1`。这样会影响到类自身，不只是新创建的实例。 |
| [5] | 创建第二个实例将再次增加类属性 `count`。注意类属性是如何被类和所有类实例所共享的。 |

# 5.9\. 私有函数

# 5.9\. 私有函数

与大多数语言一样，Python 也有私有的概念：

*   私有函数不可以从它们的模块外面被调用
*   私有类方法不能够从它们的类外面被调用
*   私有属性不能够从它们的类外面被访问

与大多数的语言不同，一个 Python 函数，方法，或属性是私有还是公有，完全取决于它的名字。

如果一个 Python 函数，类方法，或属性的名字以两个下划线开始 (但不是结束)，它是私有的；其它所有的都是公有的。 Python 没有类方法*保护* 的概念 (只能用于它们自已的类和子类中)。类方法或者是私有 (只能在它们自已的类中使用) 或者是公有 (任何地方都可使用)。

在 `MP3FileInfo` 中，有两个方法：`__parse` 和 `__setitem__`。正如我们已经讨论过的，`__setitem__` 是一个专有方法；通常，你不直接调用它，而是通过在一个类上使用字典语法来调用，但它是公有的，并且如果有一个真正好的理由，你可以直接调用它 (甚至从 `fileinfo` 模块的外面)。然而，`__parse` 是私有的，因为在它的名字前面有两个下划线。

> 注意
> 在 Python 中，所有的专用方法 (像 `__setitem__`) 和内置属性 (像 `__doc__`) 遵守一个标准的命名习惯：开始和结束都有两个下划线。不要对你自已的方法和属性用这种方法命名；到最后，它只会把你 (或其它人) 搞乱。

## 例 5.19\. 尝试调用一个私有方法

```py
>>> import fileinfo
>>> m = fileinfo.MP3FileInfo()
>>> m.__parse("/music/_singles/kairo.mp3") 
Traceback (innermost last):
  File "<interactive input>", line 1, in ?
AttributeError: 'MP3FileInfo' instance has no attribute '__parse' 
```

|  |  |
| --- | --- |
| [1] | 如果你试图调用一个私有方法，Python 将引发一个有些误导的异常，宣称那个方法不存在。当然它确实存在，但是它是私有的，所以在类外是不可使用的。严格地说，私有方法在它们的类外是可以访问的，只是不*容易* 处理。在 Python 中没有什么是真正私有的；在内部，私有方法和属性的名字被忽然改变和恢复，以致于使得它们看上去用它们给定的名字是无法使用的。你可以通过 `_MP3FileInfo__parse` 名字来使用 `MP3FileInfo` 类的 `__parse` 方法。知道了这个方法很有趣，然后要保证决不在真正的代码中使用它。私有方法由于某种原因而私有，但是像其它很多在 Python 中的东西一样，它们的私有化基本上是习惯问题，而不是强迫的。 |

## 进一步阅读

*   *Python Tutorial* 讨论了[私有变量](http://www.python.org/doc/current/tut/node11.html#SECTION0011600000000000000000)的内部工作方式。

# 5.10\. 小结

# 5.10\. 小结

实打实的对象把戏到此为止。你将在 第十二章 中看到一个真实世界应用程序的专有类方法，它使用 `getattr` 创建一个到远程 Web 服务的代理。

下一章将继续使用本章的例程探索其他 Python 的概念，例如：异常、文件对象 和 `for` 循环。

在研究下一章之前，确保你可以无困难地完成下面的事情：

*   使用 `import _module_` 或 `from _module_ import`导入模块
*   定义和实例化类
*   定义 `__init__` 方法和其他专用类方法，并理解它们何时会调用
*   子类化 `UserDict` 来定义行为像字典的类
*   定义数据属性和类属性，并理解它们之间的不同
*   定义私有属性和方法