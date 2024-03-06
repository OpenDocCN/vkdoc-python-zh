# 第九章 XML 处理

# 第九章 XML 处理

*   9.1\. 概览
*   9.2\. 包
*   9.3\. XML 解析
*   9.4\. Unicode
*   9.5\. 搜索元素
*   9.6\. 访问元素属性
*   9.7\. Segue

# 9.1\. 概览

# 9.1\. 概览

下面两章是关于 Python 中 XML 处理的。如果你已经对 XML 文档有了一个大概的了解，比如它是由结构化标记构成的，这些标记形成了层次模型的元素，等等这些知识都是有帮助的。如果你不明白这些，这里有[很多 XML 教程](http://directory.google.com/Top/Computers/Data_Formats/Markup_Languages/XML/Resources/FAQs,_Help,_and_Tutorials/)能够解释这些基础知识。

如果你对 XML 不是很感兴趣，你还是应该读一下这些章节，它们涵盖了不少重要的主题，比如 Python 包、Unicode、命令行参数以及如何使用 `getattr` 进行方法分发。

如果你在大学里主修哲学 (而不是像计算机科学这样的实用专业)，并且曾不幸地被伊曼努尔·康德的著作折磨地够呛，那么你会非常欣赏本章的样例程序。(这当然不意味着你必须修过哲学。)

处理 XML 有两种基本的方式。一种叫做 SAX (“Simple API for XML”)，它的工作方式是，一次读出一点 XML 内容，然后对发现的每一个元素调用一个方法。(如果你读了 第八章 *HTML 处理*，这应该听起来很熟悉，因为这是 `sgmllib` 工作的方式。) 另一种方式叫做 DOM (“Document Object Model”)，它的工作方式是，一次性读入整个 XML 文档，然后使用 Python 类创建一个内部表示形式 (以树结构进行连接)。Python 拥有这两种解析方式的标准模块，但是本章只涉及 DOM。

下面是一个完整的 Python 程序，它根据 XML 格式定义的上下文无关语法生成伪随机输出。如果你不明白是什么意思，不用担心，下面两章中将会深入检视这个程序的输入和输出。

## 例 9.1\. `kgp.py`

如果您还没有下载本书附带的样例程序, 可以 [下载本程序和其他样例程序](http://www.woodpecker.org.cn/diveintopython/download/diveintopython-exampleszh-cn-5.4b.zip "Download example scripts")。

```py
"""Kant Generator for Python
Generates mock philosophy based on a context-free grammar
Usage: python kgp.py [options] [source]
Options:
  -g ..., --grammar=...   use specified grammar file or URL
  -h, --help              show this help
  -d                      show debugging information while parsing
Examples:
  kgp.py                  generates several paragraphs of Kantian philosophy
  kgp.py -g husserl.xml   generates several paragraphs of Husserl
  kpg.py "<xref id='paragraph'/>"  generates a paragraph of Kant
  kgp.py template.xml     reads from template.xml to decide what to generate
"""
from xml.dom import minidom
import random
import toolbox
import sys
import getopt
_debug = 0
class NoSourceError(Exception): pass
class KantGenerator:
    """generates mock philosophy based on a context-free grammar"""
    def __init__(self, grammar, source=None):
        self.loadGrammar(grammar)
        self.loadSource(source and source or self.getDefaultSource())
        self.refresh()
    def _load(self, source):
        """load XML input source, return parsed XML document
        - a URL of a remote XML file ("http://diveintopython.org/kant.xml")
        - a filename of a local XML file ("~/diveintopython/common/py/kant.xml")
        - standard input ("-")
        - the actual XML document, as a string
        """
        sock = toolbox.openAnything(source)
        xmldoc = minidom.parse(sock).documentElement
        sock.close()
        return xmldoc
    def loadGrammar(self, grammar):                         
        """load context-free grammar"""                     
        self.grammar = self._load(grammar)                  
        self.refs = {}                                      
        for ref in self.grammar.getElementsByTagName("ref"):
            self.refs[ref.attributes["id"].value] = ref     
    def loadSource(self, source):
        """load source"""
        self.source = self._load(source)
    def getDefaultSource(self):
        """guess default source of the current grammar
        The default source will be one of the <ref>s that is not
        cross-referenced.  This sounds complicated but it's not.
        Example: The default source for kant.xml is
        "<xref id='section'/>", because 'section' is the one <ref>
        that is not <xref>'d anywhere in the grammar.
        In most grammars, the default source will produce the
        longest (and most interesting) output.
        """
        xrefs = {}
        for xref in self.grammar.getElementsByTagName("xref"):
            xrefs[xref.attributes["id"].value] = 1
        xrefs = xrefs.keys()
        standaloneXrefs = [e for e in self.refs.keys() if e not in xrefs]
        if not standaloneXrefs:
            raise NoSourceError, "can't guess source, and no source specified"
        return '<xref id="%s"/>' % random.choice(standaloneXrefs)
    def reset(self):
        """reset parser"""
        self.pieces = []
        self.capitalizeNextWord = 0
    def refresh(self):
        """reset output buffer, re-parse entire source file, and return output
        Since parsing involves a good deal of randomness, this is an
        easy way to get new output without having to reload a grammar file
        each time.
        """
        self.reset()
        self.parse(self.source)
        return self.output()
    def output(self):
        """output generated text"""
        return "".join(self.pieces)
    def randomChildElement(self, node):
        """choose a random child element of a node
        This is a utility method used by do_xref and do_choice.
        """
        choices = [e for e in node.childNodes
                   if e.nodeType == e.ELEMENT_NODE]
        chosen = random.choice(choices)            
        if _debug:                                 
            sys.stderr.write('%s available choices: %s\n' % \
                (len(choices), [e.toxml() for e in choices]))
            sys.stderr.write('Chosen: %s\n' % chosen.toxml())
        return chosen                              
    def parse(self, node):         
        """parse a single XML node
        A parsed XML document (from minidom.parse) is a tree of nodes
        of various types.  Each node is represented by an instance of the
        corresponding Python class (Element for a tag, Text for
        text data, Document for the top-level document).  The following
        statement constructs the name of a class method based on the type
        of node we're parsing ("parse_Element" for an Element node,
        "parse_Text" for a Text node, etc.) and then calls the method.
        """
        parseMethod = getattr(self, "parse_%s" % node.__class__.__name__)
        parseMethod(node)
    def parse_Document(self, node):
        """parse the document node
        The document node by itself isn't interesting (to us), but
        its only child, node.documentElement, is: it's the root node
        of the grammar.
        """
        self.parse(node.documentElement)
    def parse_Text(self, node):    
        """parse a text node
        The text of a text node is usually added to the output buffer
        verbatim.  The one exception is that <p class='sentence'> sets
        a flag to capitalize the first letter of the next word.  If
        that flag is set, we capitalize the text and reset the flag.
        """
        text = node.data
        if self.capitalizeNextWord:
            self.pieces.append(text[0].upper())
            self.pieces.append(text[1:])
            self.capitalizeNextWord = 0
        else:
            self.pieces.append(text)
    def parse_Element(self, node): 
        """parse an element
        An XML element corresponds to an actual tag in the source:
        <xref id='...'>, <p chance='...'>, <choice>, etc.
        Each element type is handled in its own method.  Like we did in
        parse(), we construct a method name based on the name of the
        element ("do_xref" for an <xref> tag, etc.) and
        call the method.
        """
        handlerMethod = getattr(self, "do_%s" % node.tagName)
        handlerMethod(node)
    def parse_Comment(self, node):
        """parse a comment
        The grammar can contain XML comments, but we ignore them
        """
        pass
    def do_xref(self, node):
        """handle <xref id='...'> tag
        An <xref id='...'> tag is a cross-reference to a <ref id='...'>
        tag.  <xref id='sentence'/> evaluates to a randomly chosen child of
        <ref id='sentence'>.
        """
        id = node.attributes["id"].value
        self.parse(self.randomChildElement(self.refs[id]))
    def do_p(self, node):
        """handle <p> tag
        The <p> tag is the core of the grammar.  It can contain almost
        anything: freeform text, <choice> tags, <xref> tags, even other
        <p> tags.  If a "class='sentence'" attribute is found, a flag
        is set and the next word will be capitalized.  If a "chance='X'"
        attribute is found, there is an X% chance that the tag will be
        evaluated (and therefore a (100-X)% chance that it will be
        completely ignored)
        """
        keys = node.attributes.keys()
        if "class" in keys:
            if node.attributes["class"].value == "sentence":
                self.capitalizeNextWord = 1
        if "chance" in keys:
            chance = int(node.attributes["chance"].value)
            doit = (chance > random.randrange(100))
        else:
            doit = 1
        if doit:
            for child in node.childNodes: self.parse(child)
    def do_choice(self, node):
        """handle <choice> tag
        A <choice> tag contains one or more <p> tags.  One <p> tag
        is chosen at random and evaluated; the rest are ignored.
        """
        self.parse(self.randomChildElement(node))
def usage():
    print __doc__
def main(argv):                         
    grammar = "kant.xml"                
    try:                                
        opts, args = getopt.getopt(argv, "hg:d", ["help", "grammar="])
    except getopt.GetoptError:          
        usage()                         
        sys.exit(2)                     
    for opt, arg in opts:               
        if opt in ("-h", "--help"):     
            usage()                     
            sys.exit()                  
        elif opt == '-d':               
            global _debug               
            _debug = 1                  
        elif opt in ("-g", "--grammar"):
            grammar = arg               
    source = "".join(args)              
    k = KantGenerator(grammar, source)
    print k.output()
if __name__ == "__main__":
    main(sys.argv[1:]) 
```

## 例 9.2\. `toolbox.py`

```py
"""Miscellaneous utility functions"""
def openAnything(source):            
    """URI, filename, or string --> stream
    This function lets you define parsers that take any input source
    (URL, pathname to local or network file, or actual data as a string)
    and deal with it in a uniform manner.  Returned object is guaranteed
    to have all the basic stdio read methods (read, readline, readlines).
    Just .close() the object when you're done with it.
    Examples:
    >>> from xml.dom import minidom
    >>> sock = openAnything("http://localhost/kant.xml")
    >>> doc = minidom.parse(sock)
    >>> sock.close()
    >>> sock = openAnything("c:\\inetpub\\wwwroot\\kant.xml")
    >>> doc = minidom.parse(sock)
    >>> sock.close()
    >>> sock = openAnything("<ref id='conjunction'><text>and</text><text>or</text></ref>")
    >>> doc = minidom.parse(sock)
    >>> sock.close()
    """
    if hasattr(source, "read"):
        return source
    if source == '-':
        import sys
        return sys.stdin
    # try to open with urllib (if source is http, ftp, or file URL)
    import urllib                         
    try:                                  
        return urllib.urlopen(source)     
    except (IOError, OSError):            
        pass                              
    # try to open with native open function (if source is pathname)
    try:                                  
        return open(source)               
    except (IOError, OSError):            
        pass                              
    # treat source as string
    import StringIO                       
    return StringIO.StringIO(str(source)) 
```

独立运行程序 `kgp.py`，它会解析 `kant.xml` 中默认的基于 XML 的语法，并以康德的风格打印出几段有哲学价值的段落来。

## 例 9.3\. `kgp.py` 的样例输出

```py
[you@localhost kgp]$ python kgp.py
 As is shown in the writings of Hume, our a priori concepts, in
reference to ends, abstract from all content of knowledge; in the study
of space, the discipline of human reason, in accordance with the
principles of philosophy, is the clue to the discovery of the
Transcendental Deduction.  The transcendental aesthetic, in all
theoretical sciences, occupies part of the sphere of human reason
concerning the existence of our ideas in general; still, the
never-ending regress in the series of empirical conditions constitutes
the whole content for the transcendental unity of apperception.  What
we have alone been able to show is that, even as this relates to the
architectonic of human reason, the Ideal may not contradict itself, but
it is still possible that it may be in contradictions with the
employment of the pure employment of our hypothetical judgements, but
natural causes (and I assert that this is the case) prove the validity
of the discipline of pure reason.  As we have already seen, time (and
it is obvious that this is true) proves the validity of time, and the
architectonic of human reason, in the full sense of these terms,
abstracts from all content of knowledge.  I assert, in the case of the
discipline of practical reason, that the Antinomies are just as
necessary as natural causes, since knowledge of the phenomena is a
posteriori.
    The discipline of human reason, as I have elsewhere shown, is by
its very nature contradictory, but our ideas exclude the possibility of
the Antinomies.  We can deduce that, on the contrary, the pure
employment of philosophy, on the contrary, is by its very nature
contradictory, but our sense perceptions are a representation of, in
the case of space, metaphysics.  The thing in itself is a
representation of philosophy.  Applied logic is the clue to the
discovery of natural causes.  However, what we have alone been able to
show is that our ideas, in other words, should only be used as a canon
for the Ideal, because of our necessary ignorance of the conditions.
[...snip...] 
```

这当然是胡言乱语。噢，不完全是胡言乱语。它在句法和语法上都是正确的 (尽管非常罗嗦――康德可不是你们所说的踩得到点上的那种人)。其中一些实际上是正确的 (或者至少康德可能会认同的事情)，其中一些则明显是错误的，大部分只是语无伦次。但所有内容都符合康德的风格。

让我重复一遍，如果你现在或曾经主修哲学专业，这会非常、非常有趣。

有趣之处在于，这个程序中没有一点内容是属于康德的。所有的内容都来自于上下文无关语法文件 `kant.xml`。如果你要程序使用不同的语法文件 (可以在命令行中指定)，输出信息将完全不同。

## 例 9.4\. `kgp.py` 的简单输出

```py
[you@localhost kgp]$ python kgp.py -g binary.xml
00101001
[you@localhost kgp]$ python kgp.py -g binary.xml
10110100 
```

在本章后面的内容中，你将近距离地观察语法文件的结构。现在，你只要知道语法文件定义了输出信息的结构，而 `kgp.py` 程序读取语法规则并随机确定哪些单词插入哪里。

# 9.2\. 包

# 9.2\. 包

实际上解析一个 XML 文档是很简单的：只要一行代码。但是，在你接触那行代码前，需要暂时岔开一下，讨论一下包。

## 例 9.5\. 载入一个 XML 文档 (偷瞥一下)

```py
>>> from xml.dom import minidom 
>>> xmldoc = minidom.parse('~/diveintopython/common/py/kgp/binary.xml') 
```

|  |  |
| --- | --- |
| [1] | 这个语法你之前没有见过。它看上去很像我们熟知的 `from _module_ import`，但是`"."` 使得它好像不只是普通的 import 那么简单。事实上，`xml` 称为包，`dom` 是 `xml` 中嵌套的包，而 `minidom` 是 `xml.dom` 中的模块。 |

听起来挺复杂的，其实不是。看一下确切的实现可能会有帮助。包不过是模块的目录；嵌套包是子目录。一个包 (或一个嵌套包) 中的模块也只是 `.py` 文件罢了，永远都是，只是它们是在一个子目录中，而不是在你的 Python 安装环境的主 `lib/` 目录下。

## 例 9.6\. 包的文件布局

```py
Python21/           Python 安装根目录 (可执行文件的所在地)
|
+--lib/             库目录 (标准库模块的所在地)
   |
   +-- xml/         xml 包 (实际上目录中还有其它东西)
       |
       +--sax/      xml.sax 包 (也只是一个目录)
       |
       +--dom/      xml.dom 包 (包含 minidom.py)
       |
       +--parsers/  xml.parsers 包 (内部使用) 
```

所以你说 `from xml.dom import minidom`，Python 认为它的意思是“在 `xml` 目录中查找 `dom` 目录，然后在*这个目录* 中查找 `minidom` 模块，接着导入它并以 `minidom` 命名 ”。但是 Python 更聪明；你不仅可以导入包含在一个包中的所有模块，还可以从包的模块中有选择地导入指定的类或者函数。语法都是一样的； Python 会根据包的布局理解你的意思，然后自动进行正确的导入。

## 例 9.7\. 包也是模块

```py
>>> from xml.dom import minidom         
>>> minidom
<module 'xml.dom.minidom' from 'C:\Python21\lib\xml\dom\minidom.pyc'>
>>> minidom.Element
<class xml.dom.minidom.Element at 01095744>
>>> from xml.dom.minidom import Element 
>>> Element
<class xml.dom.minidom.Element at 01095744>
>>> minidom.Element
<class xml.dom.minidom.Element at 01095744>
>>> from xml import dom                 
>>> dom
<module 'xml.dom' from 'C:\Python21\lib\xml\dom\__init__.pyc'>
>>> import xml                          
>>> xml
<module 'xml' from 'C:\Python21\lib\xml\__init__.pyc'> 
```

|  |  |
| --- | --- |
| [1] | 这里你正从一个嵌套包 (`xml.dom`)中导入一个模块 (`minidom`)。结果就是 `minidom` 被导入到了你 (程序) 的命名空间中了。要引用 `minidom` 模块中的类 (比如 `Element`)，你必须在它们的类名前面加上模块名。 |
| [2] | 这里你正从一个来自嵌套包 (`xml.dom`) 的模块 (`minidom`) 中导入一个类 (`Element`)。结果就是 `Element` 直接导入到了你 (程序) 的命名空间中。注意，这样做并不会干扰以前的导入；现在 `Element` 类可以用两种方式引用了 (但其实是同一个类)。 |
| [3] | 这里你正在导入 `dom` 包 (`xml` 的一个嵌套包)，并将其作为一个模块。一个包的任何层次都可以视为一个模块，一会儿就会看到。它甚至可以拥有自己的属性和方法，就像你在前面看到过的模块。 |
| [4] | 这里你正在将根层次的 `xml` 包作为一个模块导入。 |

那么如何才能导入一个包 (它不过是磁盘上的一个目录) 并使其成为一个模块 (它总是在磁盘上的一个文件) 呢？答案就是神奇的 `__init__.py` 文件。你明白了吧，包不只是目录，它们是包含一个特殊文件 `__init__.py` 的目录。这个文件定义了包的属性和方法。例如，`xml.dom` 包含了 `Node` 类，它在`xml/dom/__init__.py`中有所定义。当你将一个包作为模块导入 (比如从 `xml` 导入 `dom`) 的时候，实际上导入了它的 `__init__.py` 文件。

> 注意
> 一个包是一个其中带有特殊文件 `__init__.py` 的目录。`__init__.py` 文件定义了包的属性和方法。其实它可以什么也不定义；可以只是一个空文件，但是必须要存在。如果 `__init__.py` 不存在，这个目录就仅仅是一个目录，而不是一个包，它就不能被导入或者包含其它的模块和嵌套包。

那为什么非得用包呢？嗯，它们提供了在逻辑上将相关模块归为一组的方法。不使用其中带有 `sax` 和 `dom` 的 `xml` 包，作者也可以选择将所有的 `sax` 功能放入 `xmlsax.py`中，并将所有的 `dom` 功能放入 `xmldom.py`中，或者干脆将所有东西放入单个模块中。但是这样可能不实用 (写到这儿时，XML 包已经超过了 3000 行代码) 并且很难管理 (独立的源文件意味着多个人可以同时在不同的地方进行开发)。

如果你发现自己正在用 Python 编写一个大型的子系统 (或者，很有可能，当你意识到你的小型子系统已经成长为一个大型子系统时)，你应该花费些时间设计一个好的包架构。它是 Python 所擅长的事情之一，所以应该好好利用它。

# 9.3\. XML 解析

# 9.3\. XML 解析

正如我说的，实际解析一个 XML 文档是非常简单的：只要一行代码。从这里出发到哪儿去就是你自己的事了。

## 例 9.8\. 载入一个 XML 文档 (这次是真的)

```py
>>> from xml.dom import minidom                                          
>>> xmldoc = minidom.parse('~/diveintopython/common/py/kgp/binary.xml')  
>>> xmldoc                                                               
<xml.dom.minidom.Document instance at 010BE87C>
>>> print xmldoc.toxml()                                                 
<?xml version="1.0" ?>
<grammar>
<ref id="bit">
  <p>0</p>
  <p>1</p>
</ref>
<ref id="byte">
  <p><xref id="bit"/><xref id="bit"/><xref id="bit"/><xref id="bit"/>\
<xref id="bit"/><xref id="bit"/><xref id="bit"/><xref id="bit"/></p>
</ref>
</grammar> 
```

|  |  |
| --- | --- |
| [1] | 正如在上一节看到的，该语句从 `xml.dom` 包中导入 `minidom` 模块。 |
| [2] | 这就是进行所有工作的一行代码：`minidom.parse` 接收一个参数并返回 XML 文档解析后的表示形式。这个参数可以是很多东西；在本例中，它只是我本地磁盘上一个 XML 文档的文件名。(你需要将路径改为指向下载的例子所在的目录。) 但是你也可以传入一个文件对象，或甚至是一个类文件对象。这样你就可以在本章后面好好利用这一灵活性了。 |
| [3] | 从 `minidom.parse` 返回的对象是一个 `Document` 对象，它是 `Node` 类的一个子对象。这个 `Document` 对象是联锁的 Python 对象的一个复杂树状结构的根层次，这些 Python 对象完整表示了传给 `minidom.parse` 的 XML 文档。 |
| [4] | `toxml` 是 `Node` 类的一个方法 (因此可以在从 `minidom.parse` 中得到的 `Document` 对象上使用)。`toxml` 打印出了 `Node` 表示的 XML。对于 `Document` 节点，这样就会打印出整个 XML 文档。 |

现在内存中已经有了一个 XML 文档了，你可以开始遍历它了。

## 例 9.9\. 获取子节点

```py
>>> xmldoc.childNodes    
[<DOM Element: grammar at 17538908>]
>>> xmldoc.childNodes[0] 
<DOM Element: grammar at 17538908>
>>> xmldoc.firstChild    
<DOM Element: grammar at 17538908> 
```

|  |  |
| --- | --- |
| [1] | 每个 `Node` 都有一个 `childNodes` 属性，它是一个 `Node` 对象的列表。一个 `Document` 只有一个子节点，即 XML 文档的根元素 (在本例中，是 `grammar` 元素)。 |
| [2] | 为了得到第一个 (在本例中，只有一个) 子节点，只要使用正规的列表语法。回想一下，其实这里没有发生什么特别的；这只是一个由正规 Python 对象构成的正规 Python 列表。 |
| [3] | 鉴于获取某个节点的第一个子节点是有用而且常见的行为，所以 `Node` 类有一个 `firstChild` 属性，它和`childNodes[0]`具有相同的语义。(还有一个 `lastChild` 属性，它和`childNodes[-1]`具有相同的语义。) |

## 例 9.10\. `toxml` 用于任何节点

```py
>>> grammarNode = xmldoc.firstChild
>>> print grammarNode.toxml() 
<grammar>
<ref id="bit">
  <p>0</p>
  <p>1</p>
</ref>
<ref id="byte">
  <p><xref id="bit"/><xref id="bit"/><xref id="bit"/><xref id="bit"/>\
<xref id="bit"/><xref id="bit"/><xref id="bit"/><xref id="bit"/></p>
</ref>
</grammar> 
```

|  |  |
| --- | --- |
| [1] | 由于 `toxml` 方法是定义在 `Node` 类中的，所以对任何 XML 节点都是可用的，不仅仅是 `Document` 元素。 |

## 例 9.11\. 子节点可以是文本

```py
>>> grammarNode.childNodes                  
[<DOM Text node "\n">, <DOM Element: ref at 17533332>, \
<DOM Text node "\n">, <DOM Element: ref at 17549660>, <DOM Text node "\n">]
>>> print grammarNode.firstChild.toxml()    

>>> print grammarNode.childNodes[1].toxml() 
<ref id="bit">
  <p>0</p>
  <p>1</p>
</ref>
>>> print grammarNode.childNodes[3].toxml() 
<ref id="byte">
  <p><xref id="bit"/><xref id="bit"/><xref id="bit"/><xref id="bit"/>\
<xref id="bit"/><xref id="bit"/><xref id="bit"/><xref id="bit"/></p>
</ref>
>>> print grammarNode.lastChild.toxml() 
```

|  |  |
| --- | --- |
| [1] | 查看 `binary.xml` 中的 XML ，你可能会认为 `grammar` 只有两个子节点，即两个 `ref` 元素。但是你忘记了一些东西：硬回车！在`'&lt;grammar&gt;'`之后，第一个`'&lt;ref&gt;'`之前是一个硬回车，并且这个文本算作 `grammar` 元素的一个子节点。类似地，在每个`'&lt;/ref&gt;'`之后都有一个硬回车；它们都被当作子节点。所以`grammar.childNodes`实际上是一个有 5 个对象的列表：3 个 `Text` 对象和两个 `Element` 对象。 |
| [2] | 第一个子节点是一个 `Text` 对象，它表示在`'&lt;grammar&gt;'`标记之后、第一个`'&lt;ref&gt;'`标记之后的硬回车。 |
| [3] | 第二个子节点是一个 `Element` 对象，表示了第一个 `ref` 元素。 |
| [4] | 第四个子节点是一个 `Element` 对象，表示了第二个 `ref` 元素。 |
| [5] | 最后一个子节点是一个 `Text` 对象，表示了在`'&lt;/ref&gt;'`结束标记之后、`'&lt;/grammar&gt;'` 结束标记之前的硬回车。 |

## 例 9.12\. 把文本挖出来

```py
>>> grammarNode
<DOM Element: grammar at 19167148>
>>> refNode = grammarNode.childNodes[1] 
>>> refNode
<DOM Element: ref at 17987740>
>>> refNode.childNodes                  
[<DOM Text node "\n">, <DOM Text node "  ">, <DOM Element: p at 19315844>, \
<DOM Text node "\n">, <DOM Text node "  ">, \
<DOM Element: p at 19462036>, <DOM Text node "\n">]
>>> pNode = refNode.childNodes[2]
>>> pNode
<DOM Element: p at 19315844>
>>> print pNode.toxml()                 
<p>0</p>
>>> pNode.firstChild                    
<DOM Text node "0">
>>> pNode.firstChild.data               
u'0' 
```

|  |  |
| --- | --- |
| [1] | 正如你在前面的例子中看到的，第一个 `ref` 元素是 `grammarNode.childNodes[1]`，因为 childNodes[0] 是一个代表硬回车的 `Text` 节点。 |
| [2] | `ref` 元素有它自己的子节点集合，一个表示硬回车，另一个表示空格，一个表示 `p` 元素，诸如此类。 |
| [3] | 你甚至可以在这里使用 `toxml` 方法，尽管它深深嵌套在文档中。 |
| [4] | `p` 元素只有一个子节点 (在这个例子中无法看出，但是如果你不信，可以看看`pNode.childNodes`)，而且它是表示单字符`'0'`的一个 `Text` 节点。 |
| [5] | `Text` 节点的 `.data` 属性可以向你提供文本节点真正代表的字符串。但是字符串前面的`'u'`是什么意思呢？答案将自己专门有一部分来论述。 |

# 9.4\. Unicode

# 9.4\. Unicode

Unicode 是一个系统，用来表示世界上所有不同语言的字符。当 Python 解析一个 XML 文档时，所有的数据都是以 unicode 的形式保存在内存中的。

一会儿你就会了解，但首先，先看一些背景知识。

**历史注解.** 在 unicode 之前，对于每一种语言都存在独立的字符编码系统，每个系统都使用相同的数字(0-255)来表示这种语言的字符。一些语言 (像俄语) 对于如何表示相同的字符还有几种有冲突的标准；另一些语言 (像日语) 拥有太多的字符，需要多个字符集。在系统之间进行文档交流是困难的，因为对于一台计算机来说，没有方法可以识别出文档的作者使用了哪种编码模式；计算机看到的只是数字，并且这些数字可以表示不同的东西。接着考虑到试图将这些 (采用不同编码的) 文档存放到同一个地方 (比如在同一个数据库表中)；你需要在每段文本的旁边保存字符的编码，并且确保在传递文本的同时将编码也进行传递。接着考虑多语言文档，即在同一文档中使用了不同语言的字符。(比较有代表性的是使用转义符来进行模式切换；噗，我们处于俄语 koi8-r 模式，所以字符 241 表示这个；噗，现在我们处于 Mac 希腊语模式，所以字符 241 表示其它什么。等等。) 这些就是 unicode 被设计出来要解决的问题。

为了解决这些问题，unicode 用一个 2 字节数字表示每个字符，从 0 到 65535。[8] 每个 2 字节数字表示至少在一种世界语言中使用的一个唯一字符。(在多种语言中都使用的字符具有相同的数字码。) 这样就确保每个字符一个数字，并且每个数字一个字符。Unicode 数据永远不会模棱两可。

当然，仍然还存在着所有那些遗留的编码系统的情况。例如，7 位 ASCII，它可以将英文字符存诸为从 0 到 127 的数值。(65 是大写字母 “`A`”，97 是小写字母 “`a`”，等等。) 英语有着非常简单的字母表，所以它可以完全用 7 位 ASCII 来表示。像法语、西班牙语和德语之类的西欧语言都使用叫做 ISO-8859-1 的编码系统 (也叫做“latin-1”)，它使用 7 位 ASCII 字符表示从 0 到 127 的数字，但接着扩展到了 128-255 的范围来表示像 n 上带有一个波浪线(241)，和 u 上带有两个点(252)的字符。Unicode 在 0 到 127 上使用了同 7 位 ASCII 码一样的字符表，在 128 到 255 上同 ISO-8859-1 一样，接着使用剩余的数字，256 到 65535，扩展到表示其它语言的字符。

在处理 unicode 数据时，在某些地方你可能需要将数据转换回这些遗留编码系统之一。例如，为了同其它一些计算机系统集成，这些系统期望它的数据使用一种特定的单字节编码模式，或将数据打印输出到一个不识别 unicode 的终端或打印机。或将数据保存到一个明确指定编码模式的 XML 文档中。

在了解这个注解之后，让我们回到 Python 上来。

从 2.0 版开始，Python 整个语言都已经支持 unicode。XML 包使用 unicode 来保存所有解析了的 XML 数据，而且你可以在任何地方使用 unicode。

## 例 9.13\. unicode 介绍

```py
>>> s = u'Dive in'            
>>> s
u'Dive in'
>>> print s                   
Dive in 
```

|  |  |
| --- | --- |
| [1] | 为了创建一个 unicode 字符串而不是通常的 ASCII 字符串，要在字符串前面加上字母 “`u`”。注意这个特殊的字符串没有任何非 ASCII 的字符。这样很好；unicode 是 ASCII 的一个超集 (一个非常大的超集)，所以任何正常的 ASCII 都可以以 unicode 形式保存起来。 |
| [2] | 在打印字符串时，Python 试图将字符串转换为你的默认编码，通常是 ASCII 。(过会儿有更详细的说明。) 因为组成这个 unicode 字符串的字符都是 ASCII 字符，打印结果与打印正常的 ASCII 字符串是一样的；转换是无缝的，而且如果你没有注意到 `s` 是一个 unicode 字符串的话，你永远也不会注意到两者之间的差别。 |

## 例 9.14\. 存储非 ASCII 字符

```py
>>> s = u'La Pe\xf1a'         
>>> print s                   
Traceback (innermost last):
  File "<interactive input>", line 1, in ?
UnicodeError: ASCII encoding error: ordinal not in range(128)
>>> print s.encode('latin-1') 
La Pe?a 
```

|  |  |
| --- | --- |
| [1] | unicode 真正的优势，理所当然的是它保存非 ASCII 字符的能力，例如西班牙语的 “`?`”(`n` 上带有一个波浪线)。用来表示波浪线 n 的 unicode 字符编码是十六进制的 `0xf1` (十进制的 241)，你可以像这样输入：`\xf1`。 |
| [2] | 还记得我说过 `print` 函数会尝试将 unicode 字符串转换为 ASCII 从而打印它吗？嗯，在这里将不会起作用，因为你的 unicode 字符串包含非 ASCII 字符，所以 Python 会引发 `UnicodeError` 异常。 |
| [3] | 这儿就是将 unicode 转换为其它编码模式起作用的地方。`s` 是一个 unicode 字符串，但 `print` 只能打印正常的字符串。为了解决这个问题，我们调用 `encode` 方法 (它可以用于每个 unicode 字符串) 将 unicode 字符串转换为指定编码模式的正常字符串。我们向此函数传入一个参数。在本例中，我们使用 `latin-1` (也叫 `iso-8859-1`)，它包括带波浪线的 n (然而缺省的 ASCII 编码模式不包括，因为它只包含数值从 0 到 127 的字符)。 |

还记得我说过：需要从一个 unicode 得到一个正常字符串时，Python 通常默认将 unicode 转换成 ASCII 吗？嗯，这个默认编码模式是一个可以定制的选项。

## 例 9.15\. `sitecustomize.py`

```py
# sitecustomize.py 
# this file can be anywhere in your Python path,
# but it usually goes in ${pythondir}/lib/site-packages/
import sys
sys.setdefaultencoding('iso-8859-1') 
```

|  |  |
| --- | --- |
| [1] | `sitecustomize.py` 是一个特殊的脚本；Python 会在启动的时候导入它，所以在其中的任何代码都将自动运行。就像注解中提到的那样，它可以放在任何地方 (只要 `import` 能够找到它)，但是通常它位于 Python 的`lib` 目录的 `site-packages` 目录中。 |
| [2] | 嗯，`setdefaultencoding` 函数设置默认编码。Python 会在任何需要将 unicode 字符串自动转换为正规字符串的地方，使用这个编码模式。 |

## 例 9.16\. 设置默认编码的效果

```py
>>> import sys
>>> sys.getdefaultencoding() 
'iso-8859-1'
>>> s = u'La Pe\xf1a'
>>> print s                  
La Pe?a 
```

|  |  |
| --- | --- |
| [1] | 这个例子假设你已经按前一个例子中的改动对 `sitecustomize.py` 文件做了修改，并且已经重启了 Python。如果你的默认编码还是 `'ascii'`，可能你就没有正确设置 `sitecustomize.py` 文件，或者是没有重新启动 Python。默认的编码只能在 Python 启动的时候改变；之后就不能改变了。(由于一些我们现在不会仔细研究的古怪的编程技巧，你甚至不能在 Python 启动之后调用 `sys.setdefaultencoding` 函数。仔细研究 `site.py`，并搜索 “`setdefaultencoding`” 去发现为什么吧。) |
| [2] | 现在默认的编码模式已经包含了你在字符串中使用的所有字符，Python 对字符串的自动强制转换和打印就不存在问题了。 |

## 例 9.17\. 指定`.py`文件的编码

如果你打算在你的 Python 代码中保存非 ASCII 字符串，你需要在每个文件的顶端加入编码声明来指定每个 `.py` 文件的编码。这个声明定义了 `.py` 文件的编码为 UTF-8：

```py
#!/usr/bin/env python
# -*- coding: UTF-8 -*- 
```

现在，想想 XML 中的编码应该是怎样的呢？不错，每一个 XML 文档都有指定的编码。重复一下，ISO-8859-1 是西欧语言存放数据的流行编码方式。KOI8-R 是俄语流行的编码方式。编码――如果指定了的话――都在 XML 文档的首部。

## 例 9.18\. `russiansample.xml`

```py
 <?xml version="1.0" encoding="koi8-r"?>  <preface>
<title>Предисловие</title>  </preface> 
```

|  |  |
| --- | --- |
| [1] | 这是从一个真实的俄语 XML 文档中提取出来的示例；它就是这本书俄语翻译版的一部分。注意，编码 `koi8-r` 是在首部指定的。 |
| [2] | 这些是古代斯拉夫语的字符，就我所知，它们用来拼写俄语单词“Preface”。如果你在一个正常文本编辑器中打开这个文件，这些字符非常像乱码，因为它们使用了 `koi8-r` 编码模式进行编码，但是却以 `iso-8859-1` 编码模式进行显示。 |

## 例 9.19\. 解析 `russiansample.xml`

```py
>>> from xml.dom import minidom
>>> xmldoc = minidom.parse('russiansample.xml') 
>>> title = xmldoc.getElementsByTagName('title')[0].firstChild.data
>>> title                                       
u'\u041f\u0440\u0435\u0434\u0438\u0441\u043b\u043e\u0432\u0438\u0435'
>>> print title                                 
Traceback (innermost last):
  File "<interactive input>", line 1, in ?
UnicodeError: ASCII encoding error: ordinal not in range(128)
>>> convertedtitle = title.encode('koi8-r')     
>>> convertedtitle
'\xf0\xd2\xc5\xc4\xc9\xd3\xcc\xcf\xd7\xc9\xc5'
>>> print convertedtitle                        
Предисловие 
```

|  |  |
| --- | --- |
| [1] | 我假设在这里你将前一个例子以 `russiansample.xml` 为名保存在当前目录中。也出于完整性的考虑，我假设你已经删除了 `sitecustomize.py` 文件，将缺省编码改回到 `'ascii'`，或至少将 `setdefaultencoding` 一行注释起来了。 |
| [2] | 注意 `title` 标记 (现在在 `title` 变量中，上面那一长串 Python 函数我们暂且跳过，下一节再解释)――在 XML 文档的 `title` 元素中的文本数据是以 unicode 保存的。 |
| [3] | 直接打印 title 是不可能的，因为这个 unicode 字符串包含了非 ASCII 字符，所以 Python 不能把它转换为 ASCII，因为它无法理解。 |
| [4] | 但是，你能够显式地将它转换为 `koi8-r`，在本例中，我们得到一个 (正常，非 unicode) 单字节字符的字符串 (`f0`, `d2`, `c5`，等等)，它是初始 unicode 字符串中字符 `koi8-r` 编码的版本。 |
| [5] | 打印 `koi8-r` 编码的字符串有可能会在你的屏幕上显示为乱码，因为你的 Python IDE 将这些字符作为 `iso-8859-1` 的编码进行解析，而不是 `koi8-r` 编码。但是，至少它们能打印。 (并且，如果你仔细看，当在一个不支持 unicode 的文本编辑器中打开最初的 XML 文档时，会看到相同的乱码。Python 在解析 XML 文档时，将它从 `koi8-r` 转换到了 unicode，你只不过是将它转换回来。) |

总结一下，如果你以前从没有看到过 unicode，倒是有些唬人，但是在 Python 处理 unicode 数据真是非常容易。如果你的 XML 文档都是 7 位的 ASCII (像本章中的例子)，你差不多永远都不用考虑 unicode。Python 在进行解析时会将 XML 文档中的 ASCII 数据转换为 unicode，在任何需要的时候强制转换回为 ASCII，你甚至永远都不用注意。但是如果你要处理其它语言的数据，Python 已经准备好了。

## 进一步阅读

*   Unicode.org 是 unicode 标准的主页，包含了一个简要的[技术简介](http://www.unicode.org/standard/principles.html)。
*   Unicode 教程有更多关于如何使用 Python unicode 函数的例子，包括甚至在并不真的需要时如何将 unicode 强制转换为 ASCII。
*   PEP 263 涉及了何时、如何在你的`.py`文件中定义字符编码的更多细节。

## Footnotes

[8] 这一点，很不幸*仍然* 过分简单了。现在 unicode 已经扩展用来处理古老的汉字、韩文和日文文本，它们有太多不同的字符，以至于 2 字节的 unicode 系统不能全部表示。但当前 Python 不支持超出范围的编码，并且我不知道是否有正在计划进行解决的项目。对不起，你已经到了我经验的极限了。

# 9.5\. 搜索元素

# 9.5\. 搜索元素

通过一步步访问每一个节点的方式遍历 XML 文档可能很乏味。如果你正在寻找些特别的东西，又恰恰它们深深埋入了你的 XML 文档，有个捷径让你可以快速找到它：`getElementsByTagName` 。

在这部分，将使用 `binary.xml` 语法文件，它的内容如下：

## 例 9.20\. `binary.xml`

```py
<?xml version="1.0"?>
<!DOCTYPE grammar PUBLIC "-//diveintopython.org//DTD Kant Generator Pro v1.0//EN" "kgp.dtd">
<grammar>
<ref id="bit">
  <p>0</p>
  <p>1</p>
</ref>
<ref id="byte">
  <p><xref id="bit"/><xref id="bit"/><xref id="bit"/><xref id="bit"/>\
<xref id="bit"/><xref id="bit"/><xref id="bit"/><xref id="bit"/></p>
</ref>
</grammar> 
```

它有两个 `ref`，`'bit'` (位) 和 `'byte'` (字节)。一个 `bit` 是 `'0'` 或者 `'1'`，而一个 `byte` 是 8 个 `bit`。

## 例 9.21\. `getElementsByTagName` 介绍

```py
>>> from xml.dom import minidom
>>> xmldoc = minidom.parse('binary.xml')
>>> reflist = xmldoc.getElementsByTagName('ref') 
>>> reflist
[<DOM Element: ref at 136138108>, <DOM Element: ref at 136144292>]
>>> print reflist[0].toxml()
<ref id="bit">
  <p>0</p>
  <p>1</p>
</ref>
>>> print reflist[1].toxml()
<ref id="byte">
  <p><xref id="bit"/><xref id="bit"/><xref id="bit"/><xref id="bit"/>\
<xref id="bit"/><xref id="bit"/><xref id="bit"/><xref id="bit"/></p>
</ref> 
```

|  |  |
| --- | --- |
| [1] | `getElementsByTagName` 接收一个参数，即要找的元素的名称。它返回一个 `Element` 对象的列表，列表中的对象都是有指定名称的 XML 元素。在本例中，你能找到两个 `ref` 元素。 |

## 例 9.22\. 每个元素都是可搜索的

```py
>>> firstref = reflist[0]                      
>>> print firstref.toxml()
<ref id="bit">
  <p>0</p>
  <p>1</p>
</ref>
>>> plist = firstref.getElementsByTagName("p") 
>>> plist
[<DOM Element: p at 136140116>, <DOM Element: p at 136142172>]
>>> print plist[0].toxml()                     
<p>0</p>
>>> print plist[1].toxml()
<p>1</p> 
```

|  |  |
| --- | --- |
| [1] | 继续前面的例子，在 `reflist` 中的第一个对象是 `'bit'` `ref`元素。 |
| [2] | 你可以在这个 `Element` 上使用相同的 `getElementsByTagName` 方法来寻找所有在`'bit'` `ref` 元素中的`&lt;p&gt;`元素。 |
| [3] | 和前面一样，`getElementsByTagName` 方法返回一个找到元素的列表。在本例中，你有两个元素，每“位”各占一个。 |

## 例 9.23\. 搜索实际上是递归的

```py
>>> plist = xmldoc.getElementsByTagName("p") 
>>> plist
[<DOM Element: p at 136140116>, <DOM Element: p at 136142172>, <DOM Element: p at 136146124>]
>>> plist[0].toxml()                         
'<p>0</p>'
>>> plist[1].toxml()
'<p>1</p>'
>>> plist[2].toxml()                         
'<p><xref id="bit"/><xref id="bit"/><xref id="bit"/><xref id="bit"/>\
<xref id="bit"/><xref id="bit"/><xref id="bit"/><xref id="bit"/></p>' 
```

|  |  |
| --- | --- |
| [1] | 仔细注意这个例子和前面例子之间的不同。前面，你是在 `firstref` 中搜索 `p` 元素，但是这里你是在 `xmldoc` 中搜索 `p` 元素，`xmldoc` 是代表了整个 XML 文档的根层对象。这样*就会* 找到嵌套在 `ref` 元素 (它嵌套在根 `grammar` 元素中) 中的 `p` 元素。 |
| [2] | 前两个 `p` 元素在第一个 `ref` 内 (`'bit'` `ref`)。 |
| [3] | 后一个 `p` 元素在第二个 `ref` 中 (`'byte'` `ref`)。 |

# 9.6\. 访问元素属性

# 9.6\. 访问元素属性

XML 元素可以有一个或者多个属性，只要已经解析了一个 XML 文档，访问它们就太简单了。

在这部分中，将使用 `binary.xml` 语法文件，你在上一节中已经看到过了。

> 注意
> 这部分由于某个含义重叠的术语可能让人有点糊涂。在一个 XML 文档中，元素可以有属性，而 Python 对象也有属性。当你解析一个 XML 文档时，你得到了一组 Python 对象，它们代表 XML 文档中的所有片段，同时有些 Python 对象代表 XML 元素的属性。但是表示 (XML) 属性的 (Python) 对象也有 (Python) 属性，它们用于访问对象表示的 (XML) 属性。我告诉过你它让人糊涂。我会公开提出关于如何更明显地区分这些不同的建议。

## 例 9.24\. 访问元素属性

```py
>>> xmldoc = minidom.parse('binary.xml')
>>> reflist = xmldoc.getElementsByTagName('ref')
>>> bitref = reflist[0]
>>> print bitref.toxml()
<ref id="bit">
  <p>0</p>
  <p>1</p>
</ref>
>>> bitref.attributes          
<xml.dom.minidom.NamedNodeMap instance at 0x81e0c9c>
>>> bitref.attributes.keys()    
[u'id']
>>> bitref.attributes.values() 
[<xml.dom.minidom.Attr instance at 0x81d5044>]
>>> bitref.attributes["id"]    
<xml.dom.minidom.Attr instance at 0x81d5044> 
```

|  |  |
| --- | --- |
| [1] | 每个 `Element` 对象都有一个 `attributes` 属性，它是一个 `NamedNodeMap` 对象。听上去挺吓人的，其实不然，因为 `NamedNodeMap` 是一个行为像字典的对象，所以你已经知道怎么使用它了。 |
| [2] | 将 `NamedNodeMap` 视为一个字典，你可以通过 `attributes.keys()` 获得属性名称的一个列表。这个元素只有一个属性，`'id'`。 |
| [3] | 属性名称，像其它 XML 文档中的文本一样，都是以 unicode 保存的。 |
| [4] | 再次将 `NamedNodeMap` 视为一个字典，你可以通过 `attributes.values()` 获取属性值的一个列表。这些值本身是 `Attr` 类型的对象。你将在下一个例子中看到如何获取对象的有用信息。 |
| [5] | 仍然把 `NamedNodeMap` 视为一个字典，你可以通过常用的字典语法和名称访问单个的属性。(那些非常认真的读者将已经知道 `NamedNodeMap` 类是如何实现这一技巧的：通过定义一个 `__getitem__` 专用方法。其它的读者可能乐意接受这一事实：他们不需要理解它是如何工作的就可以有效地使用它。) |

## 例 9.25\. 访问单个属性

```py
>>> a = bitref.attributes["id"]
>>> a
<xml.dom.minidom.Attr instance at 0x81d5044>
>>> a.name  
u'id'
>>> a.value 
u'bit' 
```

|  |  |
| --- | --- |
| [1] | `Attr` 对象完整代表了单个 XML 元素的单个 XML 属性。属性的名称 (与你在 `bitref.attributes` `NamedNodeMap` 伪字典中寻找的对象同名) 保存在`a.name`中。 |
| [2] | 这个 XML 属性的真实文本值保存在 `a.value` 中。 |

> 注意
> 类似于字典，一个 XML 元素的属性没有顺序。属性可以以某种顺序*偶然* 列在最初的 XML 文档中，而在 XML 文档解析为 Python 对象时，`Attr` 对象以某种顺序*偶然* 列出，这些顺序都是任意的，没有任何特别的含义。你应该总是使用名称来访问单个属性，就像字典的键一样。

# 9.7\. Segue \[9\]

# 9.7\. Segue [9]

以上就是 XML 的核心内容。下一章将使用相同的示例程序，但是焦点在于能使程序更加灵活的其它方面：使用输入流处理，使用 `getattr` 进行方法分发，并使用命令行标识允许用户重新配置程序而无需修改代码。

在进入下一章前，你应该没有困难的完成这些事情：

*   使用 `minidom` 解析 XML 文档，搜索已解析文档，并以任意顺序访问元素属性和元素子元素
*   将复杂的库组织为包
*   将 unicode 字符串转换为不同的字符编码

## Footnotes

[9] “Segue”是音乐术语，意为“继续演奏”。