# 第八章 HTML 处理

# 第八章 HTML 处理

*   8.1\. 概览
*   8.2\. sgmllib.py 介绍
*   8.3\. 从 HTML 文档中提取数据
*   8.4\. BaseHTMLProcessor.py 介绍
*   8.5\. locals 和 globals
*   8.6\. 基于 dictionary 的字符串格式化
*   8.7\. 给属性值加引号
*   8.8\. dialect.py 介绍
*   8.9\. 全部放在一起
*   8.10\. 小结

# 8.1\. 概览

# 8.1\. 概览

我经常在 [comp.lang.python](http://groups.google.com/groups?group=comp.lang.python) 上看到关于如下的问题： “ 怎么才能从我的 HTML 文档中列出所有的 [头|图像|链接] 呢？” “怎么才能 [分析|解释|munge] 我的 HTML 文档的文本，但是又要保留标记呢？” “怎么才能一次给我所有的 HTML 标记 [增加|删除|加引号] 属性呢？” 本章将回答所有这些问题。

下面给出一个完整的，可工作的 Python 程序，它分为两部分。第一部分，`BaseHTMLProcessor.py` 是一个通用工具，它可以通过扫描标记和文本块来帮助您处理 HTML 文件。第二部分，`dialect.py` 是一个例子，演示了如何使用 `BaseHTMLProcessor.py` 来转化 HTML 文档，保留文本但是去掉了标记。阅读文档字符串 (`doc string`) 和注释来了解将要发生事情的概况。大部分内容看上去像巫术，因为任一个这些类的方法是如何调用的不是很清楚。不要紧，所有内容都会按进度被逐步地展示出来。

## 例 8.1\. `BaseHTMLProcessor.py`

如果您还没有下载本书附带的样例程序, 可以 [下载本程序和其他样例程序](http://www.woodpecker.org.cn/diveintopython/download/diveintopython-exampleszh-cn-5.4b.zip "Download example scripts")。

```py
 from sgmllib import SGMLParser
import htmlentitydefs
class BaseHTMLProcessor(SGMLParser):
    def reset(self):                       
        # extend (called by SGMLParser.__init__)
        self.pieces = []
        SGMLParser.reset(self)
    def unknown_starttag(self, tag, attrs):
        # called for each start tag
        # attrs is a list of (attr, value) tuples
        # e.g. for <pre class="screen">, tag="pre", attrs=[("class", "screen")]
        # Ideally we would like to reconstruct original tag and attributes, but
        # we may end up quoting attribute values that weren't quoted in the source
        # document, or we may change the type of quotes around the attribute value
        # (single to double quotes).
        # Note that improperly embedded non-HTML code (like client-side Javascript)
        # may be parsed incorrectly by the ancestor, causing runtime script errors.
        # All non-HTML code must be enclosed in HTML comment tags (<!-- code -->)
        # to ensure that it will pass through this parser unaltered (in handle_comment).
        strattrs = "".join([' %s="%s"' % (key, value) for key, value in attrs])
        self.pieces.append("<%(tag)s%(strattrs)s>" % locals())
    def unknown_endtag(self, tag):         
        # called for each end tag, e.g. for </pre>, tag will be "pre"
        # Reconstruct the original end tag.
        self.pieces.append("</%(tag)s>" % locals())
    def handle_charref(self, ref):         
        # called for each character reference, e.g. for "&#160;", ref will be "160"
        # Reconstruct the original character reference.
        self.pieces.append("&#%(ref)s;" % locals())
    def handle_entityref(self, ref):       
        # called for each entity reference, e.g. for "&copy;", ref will be "copy"
        # Reconstruct the original entity reference.
        self.pieces.append("&%(ref)s" % locals())
        # standard HTML entities are closed with a semicolon; other entities are not
        if htmlentitydefs.entitydefs.has_key(ref):
            self.pieces.append(";")
    def handle_data(self, text):           
        # called for each block of plain text, i.e. outside of any tag and
        # not containing any character or entity references
        # Store the original text verbatim.
        self.pieces.append(text)
    def handle_comment(self, text):        
        # called for each HTML comment, e.g. <!-- insert Javascript code here -->
        # Reconstruct the original comment.
        # It is especially important that the source document enclose client-side
        # code (like Javascript) within comments so it can pass through this
        # processor undisturbed; see comments in unknown_starttag for details.
        self.pieces.append("<!--%(text)s-->" % locals())
    def handle_pi(self, text):             
        # called for each processing instruction, e.g. <?instruction>
        # Reconstruct original processing instruction.
        self.pieces.append("<?%(text)s>" % locals())
    def handle_decl(self, text):
        # called for the DOCTYPE, if present, e.g.
        # <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
        #     "http://www.w3.org/TR/html4/loose.dtd">
        # Reconstruct original DOCTYPE
        self.pieces.append("<!%(text)s>" % locals())
    def output(self):              
        """Return processed HTML as a single string"""
        return "".join(self.pieces) 
```

## 例 8.2\. `dialect.py`

```py
 import re
from BaseHTMLProcessor import BaseHTMLProcessor
class Dialectizer(BaseHTMLProcessor):
    subs = ()
    def reset(self):
        # extend (called from __init__ in ancestor)
        # Reset all data attributes
        self.verbatim = 0
        BaseHTMLProcessor.reset(self)
    def start_pre(self, attrs):            
        # called for every <pre> tag in HTML source
        # Increment verbatim mode count, then handle tag like normal
        self.verbatim += 1                 
        self.unknown_starttag("pre", attrs)
    def end_pre(self):                     
        # called for every </pre> tag in HTML source
        # Decrement verbatim mode count
        self.unknown_endtag("pre")         
        self.verbatim -= 1                 
    def handle_data(self, text):                                        
        # override
        # called for every block of text in HTML source
        # If in verbatim mode, save text unaltered;
        # otherwise process the text with a series of substitutions
        self.pieces.append(self.verbatim and text or self.process(text))
    def process(self, text):
        # called from handle_data
        # Process text block by performing series of regular expression
        # substitutions (actual substitions are defined in descendant)
        for fromPattern, toPattern in self.subs:
            text = re.sub(fromPattern, toPattern, text)
        return text
class ChefDialectizer(Dialectizer):
    """convert HTML to Swedish Chef-speak
    based on the classic chef.x, copyright (c) 1992, 1993 John Hagerman
    """
    subs = ((r'a([nu])', r'u\1'),
            (r'A([nu])', r'U\1'),
            (r'a\B', r'e'),
            (r'A\B', r'E'),
            (r'en\b', r'ee'),
            (r'\Bew', r'oo'),
            (r'\Be\b', r'e-a'),
            (r'\be', r'i'),
            (r'\bE', r'I'),
            (r'\Bf', r'ff'),
            (r'\Bir', r'ur'),
            (r'(\w*?)i(\w*?)$', r'\1ee\2'),
            (r'\bow', r'oo'),
            (r'\bo', r'oo'),
            (r'\bO', r'Oo'),
            (r'the', r'zee'),
            (r'The', r'Zee'),
            (r'th\b', r't'),
            (r'\Btion', r'shun'),
            (r'\Bu', r'oo'),
            (r'\BU', r'Oo'),
            (r'v', r'f'),
            (r'V', r'F'),
            (r'w', r'w'),
            (r'W', r'W'),
            (r'([a-z])[.]', r'\1\.  Bork Bork Bork!'))
class FuddDialectizer(Dialectizer):
    """convert HTML to Elmer Fudd-speak"""
    subs = ((r'[rl]', r'w'),
            (r'qu', r'qw'),
            (r'th\b', r'f'),
            (r'th', r'd'),
            (r'n[.]', r'n, uh-hah-hah-hah.'))
class OldeDialectizer(Dialectizer):
    """convert HTML to mock Middle English"""
    subs = ((r'i([bcdfghjklmnpqrstvwxyz])e\b', r'y\1'),
            (r'i([bcdfghjklmnpqrstvwxyz])e', r'y\1\1e'),
            (r'ick\b', r'yk'),
            (r'ia([bcdfghjklmnpqrstvwxyz])', r'e\1e'),
            (r'eea', r'e\1e'),
            (r'([bcdfghjklmnpqrstvwxyz])y', r'\1ee'),
            (r'([bcdfghjklmnpqrstvwxyz])er', r'\1re'),
            (r'([aeiou])re\b', r'\1r'),
            (r'ia([bcdfghjklmnpqrstvwxyz])', r'i\1e'),
            (r'tion\b', r'cioun'),
            (r'ion\b', r'ioun'),
            (r'aid', r'ayde'),
            (r'ai', r'ey'),
            (r'ay\b', r'y'),
            (r'ay', r'ey'),
            (r'ant', r'aunt'),
            (r'ea', r'ee'),
            (r'oa', r'oo'),
            (r'ue', r'e'),
            (r'oe', r'o'),
            (r'ou', r'ow'),
            (r'ow', r'ou'),
            (r'\bhe', r'hi'),
            (r've\b', r'veth'),
            (r'se\b', r'e'),
            (r"'s\b", r'es'),
            (r'ic\b', r'ick'),
            (r'ics\b', r'icc'),
            (r'ical\b', r'ick'),
            (r'tle\b', r'til'),
            (r'll\b', r'l'),
            (r'ould\b', r'olde'),
            (r'own\b', r'oune'),
            (r'un\b', r'onne'),
            (r'rry\b', r'rye'),
            (r'est\b', r'este'),
            (r'pt\b', r'pte'),
            (r'th\b', r'the'),
            (r'ch\b', r'che'),
            (r'ss\b', r'sse'),
            (r'([wybdp])\b', r'\1e'),
            (r'([rnt])\b', r'\1\1e'),
            (r'from', r'fro'),
            (r'when', r'whan'))
def translate(url, dialectName="chef"):
    """fetch URL and translate using dialect
    dialect in ("chef", "fudd", "olde")"""
    import urllib                      
    sock = urllib.urlopen(url)         
    htmlSource = sock.read()           
    sock.close()                       
    parserName = "%sDialectizer" % dialectName.capitalize()
    parserClass = globals()[parserName]                    
    parser = parserClass()                                 
    parser.feed(htmlSource)
    parser.close()         
    return parser.output() 
def test(url):
    """test all dialects against URL"""
    for dialect in ("chef", "fudd", "olde"):
        outfile = "%s.html" % dialect
        fsock = open(outfile, "wb")
        fsock.write(translate(url, dialect))
        fsock.close()
        import webbrowser
        webbrowser.open_new(outfile)
if __name__ == "__main__":
    test("http://diveintopython.org/odbchelper_list.html") 
```

## 例 8.3\. `dialect.py` 的输出结果

运行这个脚本会将 第 3.2 节 “List 介绍” 转换成模仿瑞典厨师用语 (mock Swedish Chef-speak) (来自 The Muppets)、模仿埃尔默唠叨者用语 (mock Elmer Fudd-speak) (来自 Bugs Bunny 卡通画) 和模仿中世纪英语 (mock Middle English) (零散地来源于乔叟的*《坎特伯雷故事集》*)。如果您查看输出页面的 HTML 源代码，您会发现所有的 HTML 标记和属性没有改动，但是在标记之间的文本被转换成模仿语言了。如果您观查得更仔细些，您会发现，实际上，仅有标题和段落被转换了；代码列表和屏幕例子没有改动。

```py
<div class="abstract">
<p>Lists awe <span class="application">Pydon</span>'s wowkhowse datatype.
If youw onwy expewience wif wists is awways in
<span class="application">Visuaw Basic</span> ow (God fowbid) de datastowe
in <span class="application">Powewbuiwdew</span>, bwace youwsewf fow
<span class="application">Pydon</span> wists.</p>
</div> 
```

# 8.2\. `sgmllib.py` 介绍

# 8.2\. `sgmllib.py` 介绍

HTML 处理分成三步：将 HTML 分解成它的组成片段，对片段进行加工，接着将片段再重新合成 HTML。第一步是通过 `sgmllib.py` 来完成的，它是标准 Python 库的一部分。

理解本章的关键是要知道 HTML 不只是文本，更是结构化文本。这种结构来源于开始与结束标记的或多或少分级序列。通常您并不以这种方式处理 HTML ，而是以*文本方式* 在一个文本编辑中对其进行处理，或以*可视的方式* 在一个浏览器中进行浏览或页面编辑工具中进行编辑。`sgmllib.py` 表现出了 HTML 的*结构*。

`sgmllib.py` 包含一个重要的类：`SGMLParser`。`SGMLParser` 将 HTML 分解成有用的片段，比如开始标记和结束标记。在它成功地分解出某个数据为一个有用的片段后，它会根据所发现的数据，调用一个自身内部的方法。为了使用这个分析器，您需要子类化 `SGMLParser` 类，并且覆盖这些方法。这就是当我说它表示了 HTML *结构* 的意思：HTML 的结构决定了方法调用的次序和传给每个方法的参数。

`SGMLParser` 将 HTML 分析成 8 类数据，然后对每一类调用单独的方法：

开始标记 (Start tag)

是开始一个块的 HTML 标记，像 `&lt;html&gt;`、`&lt;head&gt;`、`&lt;body&gt;` 或 `&lt;pre&gt;` 等，或是一个独一的标记，像 `&lt;br&gt;` 或 `&lt;img&gt;` 等。当它找到一个开始标记 *`tagname`*，`SGMLParser` 将查找名为 `start__`tagname`_` 或 `do__`tagname`_` 的方法。例如，当它找到一个 `&lt;pre&gt;` 标记，它将查找一个 `start_pre` 或 `do_pre` 的方法。如果找到了，`SGMLParser` 会使用这个标记的属性列表来调用这个方法；否则，它用这个标记的名字和属性列表来调用 `unknown_starttag` 方法。

结束标记 (End tag)

是结束一个块的 HTML 标记，像 `&lt;/html&gt;`、`&lt;/head&gt;`、`&lt;/body&gt;` 或 `&lt;/pre&gt;` 等。当找到一个结束标记时，`SGMLParser` 将查找名为 `end__`tagname`_` 的方法。如果找到，`SGMLParser` 调用这个方法，否则它使用标记的名字来调用 `unknown_endtag` 。

字符引用 (Character reference)

用字符的十进制或等同的十六进制来表示的转义字符，像 `&#160;`。当找到，`SGMLParser` 使用十进制或等同的十六进制字符文本来调用 `handle_charref` 。

实体引用 (Entity reference)

HTML 实体，像 `&copy;`。当找到，`SGMLParser` 使用 HTML 实体的名字来调用 `handle_entityref` 。

注释 (Comment)

HTML 注释，包括在 `&lt;!-- ... --&gt;`之间。当找到，`SGMLParser` 用注释内容来调用 `handle_comment`。

处理指令 (Processing instruction)

HTML 处理指令，包括在 `&lt;? ... &gt;` 之间。当找到，`SGMLParser` 用处理指令内容来调用 `handle_pi`。

声明 (Declaration)

HTML 声明，如 `DOCTYPE`，包括在 `&lt;! ... &gt;`之间。当找到，`SGMLParser` 用声明内容来调用 `handle_decl`。

文本数据 (Text data)

文本块。不满足其它 7 种类别的任何东西。当找到，`SGMLParser` 用文本来调用 `handle_data`。

> 重要
> Python 2.0 存在一个 bug，即 `SGMLParser` 完全不能识别声明 (`handle_decl` 永远不会调用)，这就意味着 `DOCTYPE` 被静静地忽略掉了。这个错误在 Python 2.1 中改正了。

`sgmllib.py` 所附带的一个测试套件举例说明了这一点。您可以运行 `sgmllib.py`，在命令行下传入一个 HTML 文件的名字，然后它会在分析标记和其它元素的同时将它们打印出来。它的实现是通过子类化 `SGMLParser` 类，然后定义 `unknown_starttag`，`unknown_endtag`，`handle_data` 和其它方法来实现的。这些方法简单地打印出它们的参数。

> 提示
> 在 Windows 下的 ActivePython IDE 中，您可以在 “Run script” 对话框中指定命令行参数。用空格将多个参数分开。

## 例 8.4\. `sgmllib.py` 的样例测试

下面是一个片段，来自本书的 HTML 版本的目录，toc.html。当然，您的存储路径可能与我的有所不同。 (如果您还没有下载本书的 HTML 版本，可以从 [`diveintopython.org/`](http://diveintopython.org/) 下载。

```py
c:\python23\lib> type "c:\downloads\diveintopython\html\toc\index.html"
 <!DOCTYPE html
  PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html lang="en">
   <head>
      <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
      <title>Dive Into Python</title>
      <link rel="stylesheet" href="diveintopython.css" type="text/css">
... 略 ... 
```

通过 `sgmllib.py` 的测试套件来运行它，会得到如下的输出结果:

```py
c:\python23\lib> python sgmllib.py "c:\downloads\diveintopython\html\toc\index.html"
data: '\n\n'
start tag: <html lang="en" >
data: '\n   '
start tag: <head>
data: '\n      '
start tag: <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1" >
data: '\n   \n      '
start tag: <title>
data: 'Dive Into Python'
end tag: </title>
data: '\n      '
start tag: <link rel="stylesheet" href="diveintopython.css" type="text/css" >
data: '\n      '
... 略 ... 
```

下面是本章其它部分的路标：

*   子类化 `SGMLParser` 来创建从 HTML 文档中抽取感兴趣的数据的类。
*   子类化 `SGMLParser` 来创建 `BaseHTMLProcessor`，它覆盖了所有 8 个处理方法，然后使用它们从片段中重建原始的 HTML。
*   子类化 `BaseHTMLProcessor` 来创建 `Dialectizer`，它增加了一些方法，专门用来处理指定的 HTML 标记，然后覆盖了 `handle_data` 方法，提供了用来处理 HTML 标记之间文本块的框架。
*   子类化 `Dialectizer` 来创建定义了文本处理规则的类。这些规则被 `Dialectizer.handle_data` 使用。
*   编写一个测试套件，它可以从 `http://diveintopython.org/` 处抓取一个真正的 web 页面，然后处理它。

继续阅读本章，您还可以学习到有关 `locals`、`globals` 和基于 dictionary 的字符串格式化的内容。

# 8.3\. 从 HTML 文档中提取数据

# 8.3\. 从 HTML 文档中提取数据

为了从 HTML 文档中提取数据，将 `SGMLParser` 类进行子类化，然后对想要捕捉的标记或实体定义方法。

从 HTML 文档中提取数据的第一步是得到某个 HTML 文件。如果在您的硬盘里存放着 HTML 文件，您可以使用处理文件的函数将它读出来，但是真正有意思的是从实际的网页得到 HTML。

## 例 8.5\. `urllib` 介绍

```py
>>> import urllib                                       
>>> sock = urllib.urlopen("http://diveintopython.org/") 
>>> htmlSource = sock.read()                            
>>> sock.close()                                        
>>> print htmlSource                                    
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd"><html><head>
      <meta http-equiv='Content-Type' content='text/html; charset=ISO-8859-1'>
   <title>Dive Into Python</title>
<link rel='stylesheet' href='diveintopython.css' type='text/css'>
<link rev='made' href='mailto:mark@diveintopython.org'>
<meta name='keywords' content='Python, Dive Into Python, tutorial, object-oriented, programming, documentation, book, free'>
<meta name='description' content='a free Python tutorial for experienced programmers'>
</head>
<body bgcolor='white' text='black' link='#0000FF' vlink='#840084' alink='#0000FF'>
<table cellpadding='0' cellspacing='0' border='0' width='100%'>
<tr><td class='header' width='1%' valign='top'>diveintopython.org</td>
<td width='99%' align='right'><hr size='1' noshade></td></tr>
<tr><td class='tagline' colspan='2'>Python&nbsp;for&nbsp;experienced&nbsp;programmers</td></tr>
[...略...] 
```

|  |  |
| --- | --- |
| [1] | `urllib` 模块是标准 Python 库的一部分。它包含了一些函数，可以从基于互联网的 URL (主要指网页) 来获取信息并且真正取回数据。 |
| [2] | `urllib` 模块最简单的使用是提取用 `urlopen` 函数取回的网页的整个文本。打开一个 URL 同打开一个文件相似。`urlopen` 的返回值是像文件一样的对象，它具有一个文件对象一样的方法。 |
| [3] | 使用由 `urlopen` 所返回的类文件对象所能做的最简单的事情就是 `read`，它可以将网页的整个 HTML 读到一个字符串中。这个对象也支持 `readlines` 方法，这个方法可以将文本按行放入一个列表中。 |
| [4] | 当用完这个对象，要确保将它 `close`，就如同一个普通的文件对象。 |
| [5] | 现在我们将 `http://diveintopython.org/` 主页的完整的 HTML 保存在一个字符串中了，接着我们将分析它。 |

## 例 8.6\. `urllister.py` 介绍

如果您还没有下载本书附带的样例程序, 可以 [下载本程序和其他样例程序](http://www.woodpecker.org.cn/diveintopython/download/diveintopython-exampleszh-cn-5.4b.zip "Download example scripts")。

```py
 from sgmllib import SGMLParser
class URLLister(SGMLParser):
    def reset(self):                              
        SGMLParser.reset(self)
        self.urls = []
    def start_a(self, attrs):                     
        href = [v for k, v in attrs if k=='href']  
        if href:
            self.urls.extend(href) 
```

|  |  |
| --- | --- |
| [1] | `reset` 由 `SGMLParser` 的 `__init__` 方法来调用，也可以在创建一个分析器实例时手工来调用。所以如果您需要做初始化，在 `reset` 中去做，而不要在 `__init__` 中做。这样当某人重用一个分析器实例时，可以正确地重新初始化。 |
| [2] | 只要找到一个 `&lt;a&gt;` 标记，`start_a` 就会由 `SGMLParser` 进行调用。这个标记可以包含一个 `href` 属性，或者包含其它的属性，如 `name` 或 `title`。`attrs` 参数是一个 tuple 的 list，`[(_attribute_, _value_), (_attribute_, _value_), ...]`。或者它可以只是一个有效的 HTML 标记 `&lt;a&gt;` (尽管无用)，这时 `attrs` 将是个空 list。 |
| [3] | 我们可以通过一个简单的多变量 list 映射来查找这个 `&lt;a&gt;` 标记是否拥有一个 `href` 属性。 |
| [4] | 像 `k=='href'` 的字符串比较是区分大小写的，但是这里是安全的。因为 `SGMLParser` 会在创建 `attrs` 时将属性名转化为小写。 |

## 例 8.7\. 使用 `urllister.py`

```py
>>> import urllib, urllister
>>> usock = urllib.urlopen("http://diveintopython.org/")
>>> parser = urllister.URLLister()
>>> parser.feed(usock.read())         
>>> usock.close()                     
>>> parser.close()                    
>>> for url in parser.urls: print url 
toc/index.html
#download
#languages
toc/index.html
appendix/history.html
download/diveintopython-html-5.0.zip
download/diveintopython-pdf-5.0.zip
download/diveintopython-word-5.0.zip
download/diveintopython-text-5.0.zip
download/diveintopython-html-flat-5.0.zip
download/diveintopython-xml-5.0.zip
download/diveintopython-common-5.0.zip 
...略... 
```

|  |  |
| --- | --- |
| [1] | 调用定义在 `SGMLParser` 中的 `feed` 方法，将 HTML 内容放入分析器中。 [4] 这个方法接收一个字符串，这个字符串就是 `usock.read()` 所返回的。 |
| [2] | 像处理文件一样，一旦处理完毕，您应该 `close` 您的 URL 对象。 |
| [3] | 您也应该 `close` 您的分析器对象，但出于不同的原因。`feed` 方法不保证对传给它的全部 HTML 进行处理，它可能会对其进行缓冲处理，等待接收更多的内容。只要没有更多的内容，就应调用 `close` 来刷新缓冲区，并且强制所有内容被完全处理。 |
| [4] | 一旦分析器被 `close`，分析过程也就结束了。`parser.urls` 中包含了在 HTML 文档中所有的链接 URL。(如果当您读到此处发现输出结果不一样，那是因为下载了本书的更新版本。) |

## Footnotes

[4] 像 `SGMLParser` 这样的分析器，技术术语叫做*消费者 (consumer)*。它消费 HTML，并且拆分它。也许因此就选择了 `feed` 这个名字，以便同*消费者* 这个主题相适应。就个人来说，它让我想象在动物园看展览。里面有一个黑漆漆的兽穴，没有树，没有植物，没有任何生命的迹象。但只要您非常安静地站着，尽可能靠近着瞧，您会看到在远处的角落里有两只明眸在盯着您。但是您会安慰自已那不过是心理作用。唯一知道兽穴里并不是空无一物的方法，就是在栅栏上有一个不明显的标记，上面写着 “禁止给分析器喂食”。但也许只有我这么想，不管怎么样，这种心理想象很有意思。

# 8.4\. `BaseHTMLProcessor.py` 介绍

# 8.4\. `BaseHTMLProcessor.py` 介绍

`SGMLParser` 自身不会产生任何结果。它只是分析，分析，再分析，对于它找到的有趣的东西会调用相应的一个方法，但是这些方法什么都不做。`SGMLParser` 是一个 HTML *消费者 (consumer)*：它接收 HTML，将其分解成小的、结构化的小块。正如您所看到的，在前一节中，您可以定义 `SGMLParser` 的子类，它可以捕捉特别标记和生成有用的东西，如一个网页中所有链接的一个列表。现在我们将沿着这条路更深一步。我们要定义一个可以捕捉 `SGMLParser` 所丢出来的所有东西的一个类，接着重建整个 HTML 文档。用技术术语来说，这个类将是一个 HTML *生产者 (producer)*。

`BaseHTMLProcessor` 子类化 `SGMLParser`，并且提供了全部的 8 个处理方法：`unknown_starttag`、`unknown_endtag`、`handle_charref`、`handle_entityref`、`handle_comment`、`handle_pi`、`handle_decl` 和 `handle_data`。

## 例 8.8\. `BaseHTMLProcessor` 介绍

```py
 class BaseHTMLProcessor(SGMLParser):
    def reset(self):                        
        self.pieces = []
        SGMLParser.reset(self)
    def unknown_starttag(self, tag, attrs): 
        strattrs = "".join([' %s="%s"' % (key, value) for key, value in attrs])
        self.pieces.append("<%(tag)s%(strattrs)s>" % locals())
    def unknown_endtag(self, tag):          
        self.pieces.append("</%(tag)s>" % locals())
    def handle_charref(self, ref):          
        self.pieces.append("&#%(ref)s;" % locals())
    def handle_entityref(self, ref):        
        self.pieces.append("&%(ref)s" % locals())
        if htmlentitydefs.entitydefs.has_key(ref):
            self.pieces.append(";")
    def handle_data(self, text):            
        self.pieces.append(text)
    def handle_comment(self, text):         
        self.pieces.append("<!--%(text)s-->" % locals())
    def handle_pi(self, text):              
        self.pieces.append("<?%(text)s>" % locals())
    def handle_decl(self, text):
        self.pieces.append("<!%(text)s>" % locals()) 
```

|  |  |
| --- | --- |
| [1] | `reset` 由 `SGMLParser.__init__` 来调用。在调用父类方法之前将 `self.pieces` 初始化为空列表。`self.pieces` 是一个数据属性，将用来保存将要构造的 HTML 文档的片段。每个处理器方法都将重构 `SGMLParser` 所分析出来的 HTML，并且每个方法将生成的字符串追加到 `self.pieces` 之后。注意，`self.pieces` 是一个 list。也许您想将它定义为一个字符串，然后不停地将每个片段追加到它的后面。这样做是可以的，但是 Python 在处理 list 方面效率更高一些。 [5] |
| [2] | 因为 `BaseHTMLProcessor` 没有为特别标记定义方法 (如在 `URLLister` 中的`start_a` 方法)， `SGMLParser` 将对每一个开始标记调用 `unknown_starttag` 方法。这个方法接收标记 (`tag`) 和属性的名字/值对的 list(`attrs`) 两参数，重新构造初始的 HTML，接着将结果追加到 `self.pieces` 后。 这里的字符串格式化有些陌生，我们将留到下一节再说明。 |
| [3] | 重构结束标记要简单得多，只是使用标记名字，把它包在 `&lt;/...&gt;` 括号中。 |
| [4] | 当 `SGMLParser` 找到一个字符引用时，会用原始的引用来调用 `handle_charref`。如果 HTML 文档包含 `&#160;` 这个引用，`ref` 将为 `160`。重构原始的完整的字符引用只要将 `ref` 包装在 `&#...;` 字符中间。 |
| [5] | 实体引用同字符引用相似，但是没有#号。重建原始的实体引用只要将 `ref` 包装在 `&...;` 字符串中间。(实际上，一位博学的读者曾经向我指出，除些之外还稍微有些复杂。仅有某种标准的 HTML 实体以一个分号结束；其它看上去差不多的实体并不如此。幸运的是，标准 HTML 实体集已经定义在 Python 的一个叫做 `htmlentitydefs` 的模块中了。从而引出额外的 `if` 语句。) |
| [6] | 文本块则简单地不经修改地追加到 `self.pieces` 后。 |
| [7] | HTML 注释包装在 `&lt;!--...--&gt;` 字符中。 |
| [8] | 处理指令包装在 `&lt;?...&gt;` 字符中。 |

> 重要
> HTML 规范要求所有非 HTML (像客户端的 JavaScript) 必须包括在 HTML 注释中，但不是所有的页面都是这么做的 (而且所有的最新的浏览器也都容许不这样做) 。`BaseHTMLProcessor` 不允许这样，如果脚本嵌入得不正确，它将被当作 HTML 一样进行分析。例如，如果脚本包含了小于和等于号，`SGMLParser` 可能会错误地认为找到了标记和属性。`SGMLParser` 总是把标记名和属性名转换成小写，这样可能破坏了脚本，并且 `BaseHTMLProcessor` 总是用双引号来将属性封闭起来 (尽管原始的 HTML 文档可能使用单引号或没有引号) ，这样必然会破坏脚本。应该总是将您的客户端脚本放在 HTML 注释中进行保护。

## 例 8.9\. `BaseHTMLProcessor` 输出结果

```py
 def output(self):               
        """Return processed HTML as a single string"""
        return "".join(self.pieces) 
```

|  |  |
| --- | --- |
| [1] | 这是在 `BaseHTMLProcessor` 中的一个方法，它永远不会被父类 `SGMLParser` 所调用。因为其它的处理器方法将它们重构的 HTML 保存在 `self.pieces` 中，这个函数需要将所有这些片段连接成一个字符串。正如前面提到的，Python 在处理列表方面非常出色，但对于字符串处理就逊色了。所以我们只有在某人确实需要它时才创建完整的字符串。 |
| [2] | 如果您愿意，也可以换成使用 `string` 模块的 `join` 方法：`string.join(self.pieces, "")`。 |

## 进一步阅读

*   W3C 讨论了[字符和实体引用](http://www.w3.org/TR/REC-html40/charset.html#entities)。
*   *Python Library Reference* 解答了您的怀疑，即 [`htmlentitydefs` 模块](http://www.python.org/doc/current/lib/module-htmlentitydefs.html)的确名符其实。

## Footnotes

[5] Python 处理 list 比字符串快的原因是：list 是可变的，但字符串是不可变的。这就是说向 list 进行追加只是增加元素和修改索引。因为字符串在创建之后不能被修改，像 `s = s + newpiece` 这样的代码将会从原值和新片段的连接结果中创建一个全新的字符串，然后丢弃原来的字符串。这样就需要大量昂贵的内存管理，并且随着字符串变长，所需要的开销也在增长。所以在一个循环中执行 `s = s + newpiece` 非常不好。用技术术语来说，向一个 list 追加 `n` 个项的代价为 `O(n)`，而向一个字符串追加 `n` 个项的代价是 `O(n&lt;sup&gt;2&lt;/sup&gt;)`。

# 8.5\. `locals` 和 `globals`

# 8.5\. `locals` 和 `globals`

我们先偏离一下 HTML 处理的主题，讨论一下 Python 如何处理变量。Python 有两个内置的函数，`locals` 和 `globals`，它们提供了基于 dictionary 的访问局部和全局变量的方式。

还记得 `locals` 吗？您第一次是在这里看到的：

```py
 def unknown_starttag(self, tag, attrs):
        strattrs = "".join([' %s="%s"' % (key, value) for key, value in attrs])
        self.pieces.append("<%(tag)s%(strattrs)s>" % locals()) 
```

不，等等，此时您还不能理解 `locals` 。首先，您需要学习关于命名空间的知识。这很枯燥，但是很重要，因此要要耐心些。

Python 使用叫做名字空间的东西来记录变量的轨迹。名字空间只是一个 dictionary ，它的键字就是变量名，它的值就是那些变量的值。实际上，名字空间可以像 Python 的 dictionary 一样进行访问，一会儿我们就会看到。

在一个 Python 程序中的任何一个地方，都存在几个可用的名字空间。每个函数都有着自已的名字空间，叫做局部名字空间，它记录了函数的变量，包括函数的参数和局部定义的变量。每个模块拥有它自已的名字空间，叫做全局名字空间，它记录了模块的变量，包括函数、类、其它导入的模块、模块级的变量和常量。还有就是内置名字空间，任何模块均可访问它，它存放着内置的函数和异常。

当一行代码要使用变量 `x` 的值时，Python 会到所有可用的名字空间去查找变量，按照如下顺序：

1.  局部名字空间――特指当前函数或类的方法。如果函数定义了一个局部变量 `x`，或一个参数 `x`，Python 将使用它，然后停止搜索。
2.  全局名字空间――特指当前的模块。如果模块定义了一个名为 `x` 的变量，函数或类，Python 将使用它然后停止搜索。
3.  内置名字空间――对每个模块都是全局的。作为最后的尝试，Python 将假设 `x` 是内置函数或变量。

如果 Python 在这些名字空间找不到 `x`，它将放弃查找并引发一个 `NameError` 异常，同时传递 `There is no variable named 'x'` 这样一条信息，回到 例 3.18 “引用未赋值的变量”，您会看到一路上都有这样的信息。但是您并没有体会到 Python 在给出这样的错误之前做了多少的努力。

> 重要
> Python 2.2 引入了一种略有不同但重要的改变，它会影响名字空间的搜索顺序：嵌套的作用域。 在 Python 2.2 版本之前，当您在一个嵌套函数或 `lambda` 函数中引用一个变量时，Python 会在当前 (嵌套的或 `lambda`) 函数的名字空间中搜索，然后在模块的名字空间。Python 2.2 将只在当前 (嵌套的或 `lambda`) 函数的名字空间中搜索，*然后是在父函数的名字空间* 中搜索，接着是模块的名字空间中搜索。Python 2.1 可 以两种方式工作，缺省地，按 Python 2.0 的方式工作。但是您可以把下面一行代码增加到您的模块头部，使您的模块工作起来像 Python 2.2 的方式：
> 
> ```py
> from __future__ import nested_scopes 
> ```

您是否为此而感到困惑？不要灰心！我敢说这一点非常酷。像 Python 中的许多事情一样，名字空间*在运行时直接可以访问*。怎么样？不错吧，局部名字空间可以通过内置的 `locals` 函数来访问。全局 (模块级别) 名字空间可以通过内置的 `globals` 函数来访问。

## 例 8.10\. `locals` 介绍

```py
>>> def foo(arg): 
... x = 1
... print locals()
... 
>>> foo(7)        
{'arg': 7, 'x': 1}
>>> foo('bar')    
{'arg': 'bar', 'x': 1} 
```

|  |  |
| --- | --- |
| [1] | 函数 `foo` 在它的局部名字空间中有两个变量：`arg` (它的值是被传入函数的) 和 `x` (它是在函数里定义的)。 |
| [2] | `locals` 返回一个名字/值对的 dictionary。这个 dictionary 的键字是字符串形式的变量名字，dictionary 的值是变量的实际值。所以用 `7` 来调用 `foo`，会打印出包含函数两个局部变量的 dictionary：`arg` (`7`) 和 `x` (`1`)。 |
| [3] | 回想一下，Python 有动态数据类型，所以您可以非常容易地传递给 `arg` 一个字符串，这个函数 (和对 `locals` 的调用) 将仍然很好的工作。`locals` 可以用于所有类型的变量。 |

`locals` 对局部 (函数) 名字空间做了些什么，`globals` 就对全局 (模块) 名字空间做了什么。然而 `globals` 更令人兴奋，因为一个模块的名字空间是更令人兴奋的。[6] 模块的名字空间不仅仅包含了模块级的变量和常量，还包括了所有在模块中定义的函数和类。除此以外，它还包括了任何被导入到模块中的东西。

回想一下 `from _module_ import` 和 `import _module_` 之间的不同。使用 `import _module_`，模块自身被导入，但是它保持着自已的名字空间，这就是为什么您需要使用模块名来访问它的函数或属性：`_module_._function_` 的原因。但是使用 `from _module_ import`，实际上是从另一个模块中将指定的函数和属性导入到您自己的名字空间，这就是为什么您可以直接访问它们却不需要引用它们所来源的模块。使用 `globals` 函数，您会真切地看到这一切的发生。

## 例 8.11\. `globals` 介绍

看看下面列出的在文件 `BaseHTMLProcessor.py` 尾部的代码块：

```py
 if __name__ == "__main__":
    for k, v in globals().items():             
        print k, "=", v 
```

|  |  |
| --- | --- |
| [1] | 不要被吓坏了，想想以前您已经全部都看到过了。`globals` 函数返回一个 dictionary，我们使用 `items` 方法和多变量赋值来遍历 dictionary。在这里唯一的新东西就是 `globals` 函数。 |

现在从命令行运行这个脚本，会得到下面的输出 (注意您的输出可能有略微的不同，这依赖于您的系统平台和所安装的 Python 版本)：

```py
c:\docbook\dip\py> python BaseHTMLProcessor.py 
```

```py
SGMLParser = sgmllib.SGMLParser                
htmlentitydefs = <module 'htmlentitydefs' from 'C:\Python23\lib\htmlentitydefs.py'> 
BaseHTMLProcessor = __main__.BaseHTMLProcessor 
__name__ = __main__                            
... rest of output omitted for brevity... 
```

|  |  |
| --- | --- |
| [1] | 我们使用了 `from _module_ import` 把 `SGMLParser` 从 `sgmllib` 中导入。也就是说它被直接导入到我们的模块名字空间了，就是这样。 |
| [2] | 把上面的例子和 `htmlentitydefs` 对比一下，它是用 `import` 被导入的。也就是说 `htmlentitydefs` 模块本身被导入了名字空间，但是定义在 `htmlentitydefs` 之中的 `entitydefs` 变量却没有。 |
| [3] | 这个模块只定义一个类，`BaseHTMLProcessor`，不错。注意这儿的值就是类本身，不是一个特别的类实例。 |
| [4] | 记得 `if __name__` 技巧吗？当运行一个模块时 (相对于从另外一个模块中导入而言)，内置的 `__name__` 是一个特殊值 `__main__`。因为我们是把这个模块当作脚本从命令来运行的，故 `__name__` 值为 `__main__`，这就是为什么我们这段简单地打印 `globals` 的代码可以执行的原因。 |

> 注意
> 使用 `locals` 和 `globals` 函数，通过提供变量的字符串名字您可以动态地得到任何变量的值。这种方法提供了这样的功能：`getattr` 函数允许您通过提供函数的字符串名来动态地访问任意的函数。

在 `locals` 与 `globals` 之间有另外一个重要的区别，您应该在它困扰您之前就了解它。它无论如何都会困扰您的，但至少您还会记得曾经学习过它。

## 例 8.12\. `locals` 是只读的，`globals` 不是

```py
 def foo(arg):
    x = 1
    print locals()    
    locals()["x"] = 2 
    print "x=",x      
z = 7
print "z=",z
foo(3)
globals()["z"] = 8     print "z=",z 
```

|  |  |
| --- | --- |
| [1] | 因为使用 `3` 来调用 `foo`，会打印出 `{'arg': 3, 'x': 1}`。这个应该没什么奇怪的。 |
| [2] | `locals` 是一个返回 dictionary 的函数，这里您在 dictionary 中设置了一个值。您可能认为这样会改变局部变量 `x` 的值为 `2`，但并不会。`locals` 实际上没有返回局部名字空间，它返回的是一个拷贝。所以对它进行改变对局部名字空间中的变量值并无影响。 |
| [3] | 这样会打印出 `x= 1`，而不是 `x= 2`。 |
| [4] | 在有了对 `locals` 的经验之后，您可能认为这样*不会* 改变 `z` 的值，但是可以。由于 Python 在实现过程中内部有所区别 (关于这些区别我宁可不去研究，因为我自已还没有完全理解) ，`globals` 返回实际的全局名字空间，而不是一个拷贝：与 `locals` 的行为完全相反。所以对 `globals` 所返回的 dictionary 的任何的改动都会直接影响到全局变量。 |
| [5] | 这样会打印出 `z= 8`，而不是 `z= 7`。 |

## Footnotes

[6] 我没有说得太多吧。

# 8.6\. 基于 dictionary 的字符串格式化

# 8.6\. 基于 dictionary 的字符串格式化

为什么学习 `locals` 和 `globals`？因为接下来就可以学习关于基于 dictionary 的字符串格式化。或许您还能记起，字符串格式化提供了一种将值插入字符串中的一种便捷的方法。值被列在一个 tuple 中，按照顺序插入到字符串中每个格式化标记所在的位置上。尽管这种做法效率高，但还不是最容易阅读的代码，特别是当插入多个值的时候。仅用眼看一遍字符串，您不能马上就明白结果是什么；您需要经常地在字符串和值的 tuple 之间进行反复查看。

有另外一种字符串格式化的形式，它使用 dictionary 而不是值的 tuple。

## 例 8.13\. 基于 dictionary 的字符串格式化介绍

```py
>>> params = {"server":"mpilgrim", "database":"master", "uid":"sa", "pwd":"secret"}
>>> "%(pwd)s" % params                                    
'secret'
>>> "%(pwd)s is not a good password for %(uid)s" % params 
'secret is not a good password for sa'
>>> "%(database)s of mind, %(database)s of body" % params 
'master of mind, master of body' 
```

|  |  |
| --- | --- |
| [1] | 这种字符串格式化形式不用显式的值的 tuple，而是使用一个 dictionary，`params`。并且标记也不是在字符串中的一个简单 `%s`，而是包含了一个用括号包围起来的名字。这个名字是 `params` dictionary 中的一个键字，所以 `%(pwd)s` 标记被替换成相应的值 `secret`。 |
| [2] | 基于 dictionary 的字符串格式化可用于任意数量的有名的键字。每个键字必须在一个给定的 dictionary 中存在，否则这个格式化操作将失败并引发一个 `KeyError` 的异常。 |
| [3] | 您甚至可以两次指定同一键字，每个键字出现之处将被同一个值所替换。 |

那么为什么您偏要使用基于 dictionary 的字符串格式化呢？的确，仅为了进行字符串格式化，就事先创建一个有键字和值的 dictionary 看上去的确有些小题大作。它的真正最大用处是当您碰巧已经有了像 `locals` 一样的有意义的键字和值的 dictionary 的时候。

## 例 8.14\. `BaseHTMLProcessor.py` 中的基于 dictionary 的字符串格式化

```py
 def handle_comment(self, text):        
        self.pieces.append("<!--%(text)s-->" % locals()) 
```

|  |  |
| --- | --- |
| [1] | 使用内置的 `locals` 函数是最普通的基于 dictionary 的字符串格式化的应用。这就是说您可以在您的字符串 (本例中是 `text`，它作为一个参数传递给类方法) 中使用局部变量的名字，并且每个命名的变量将会被它的值替换。如果 `text` 是 `'Begin page footer'`，字符串格式化 `"&lt;!--%(text)s--&gt;" % locals()` 将得到字符串 `'&lt;!--Begin page footer--&gt;'`。 |

## 例 8.15\. 基于 dictionary 的字符串格式化的更多内容

```py
 def unknown_starttag(self, tag, attrs):
        strattrs = "".join([' %s="%s"' % (key, value) for key, value in attrs]) 
        self.pieces.append("<%(tag)s%(strattrs)s>" % locals()) 
```

|  |  |
| --- | --- |
| [1] | 当这个模块被调用时，`attrs` 是一个键/值 tuple 的 list，就像一个 dictionary 的 `items`。这就意味着我们可以使用多变量赋值来遍历它。到现在这将是一种熟悉的模式，但是这里有很多东西，让我们分开来看：1\. 假设 `attrs` 是 `[('href', 'index.html'), ('title', 'Go to home page')]`。2\. 在这个列表解析的第一轮循环中，`key` 将为 `'href'`，`value` 将为 `'index.html'`。3\. 字符串格式化 `' %s="%s"' % (key, value)` 将生成 `' href="index.html"'`。这个字符串就作为这个列表解析返回值的第一个元素。4\. 在第二轮中，`key` 将为 `'title'`，`value` 将为 `'Go to home page'`。5\. 字符串格式化将生成 `' title="Go to home page"'`。6\. 这个 list 理解返回两个生成的字符串 list，并且 `strattrs` 将把这个 list 的两个元素连接在一起形成 `' href="index.html" title="Go to home page"'`。 |
| [2] | 现在，使用基于 dictionary 的字符串格式化，我们将 `tag` 和 `strattrs` 的值插入到一个字符串中。所以，如果 `tag` 是 `'a'`，最终的结果会是 `'&lt;a href="index.html" title="Go to home page"&gt;'`，并且这就是追加到 `self.pieces` 后面的东西。 |

> 重要
> 使用 `locals` 来应用基于 dictionary 的字符串格式化是一种方便的作法，它可以使复杂的字符串格式化表达式更易读。但它需要花费一定的代价。在调用 `locals` 方面有一点性能上的问题，这是由于 `locals` 创建了局部名字空间的一个拷贝引起的。

# 8.7\. 给属性值加引号

# 8.7\. 给属性值加引号

在 [comp.lang.python](http://groups.google.com/groups?group=comp.lang.python) 上的一个常见问题是 “我有一些 HTML 文档，属性值没有用引号括起来，并且我想将它们全部括起来，我怎么才能实现它呢？” [7] (一般这种事情的出现是由于一个项目经理加入到一个大的项目中来，而他又抱着 HTML 是一种标记语言的教条，要求所有的页面必须能够通过 HTML 校验器的验证。而属性值没有被引号括起来是一种常见的对 HTML 规范的违反。) 不管什么原因，未括起来的属性值通过将 HTML 送进 `BaseHTMLProcessor` 可以容易地修复。

`BaseHTMLProcessor` 消费 (consume) HTML (因为它是从 `SGMLParser` 派生来的) 并生成等价的 HTML。但是这个 HTML 输出与输入的并不一样。标记和属性名最终会转化为小写字母，即使它们可能以大写字母开始或是大小写的混合形式。属性值将被双引号引起来，即使它们原来可能是用单引号括起来的或根本没有括起来。这就是最后我们可以受益的边际效应。

## 例 8.16\. 给属性值加引号

```py
>>> htmlSource = """        
... <html>
... <head>
... <title>Test page</title>
... </head>
... <body>
... <ul>
... <li><a href=index.html>Home</a></li>
... <li><a href=toc.html>Table of contents</a></li>
... <li><a href=history.html>Revision history</a></li>
... </body>
... </html>
... """
>>> from BaseHTMLProcessor import BaseHTMLProcessor
>>> parser = BaseHTMLProcessor()
>>> parser.feed(htmlSource) 
>>> print parser.output()   
<html>
<head>
<title>Test page</title>
</head>
<body>
<ul>
<li><a href="index.html">Home</a></li>
<li><a href="toc.html">Table of contents</a></li>
<li><a href="history.html">Revision history</a></li>
</body>
</html> 
```

|  |  |
| --- | --- |
| [1] | 请注意，在 `&lt;a&gt;` 标记中的 `href` 属性值没有被适当地括起来 (还要注意，除了文档字符串之外，我们还将三重引号用到了 `doc string` 之外的其它地方，并且是不会少于直接在 IDE 中的使用。它们非常有用。) |
| [2] | 装填分析器。 |
| [3] | 使用定义在 `BaseHTMLProcessor` 中的 `output` 函数，我们得到单个字符串的输出，并且属性值被完全括起来了。让我们想一下这里实际上发生了多少事：`SGMLParser` 分析整个 HTML 文档，将其分解为一片片的标记、引用、数据等等。`BaseHTMLProcessor` 使用这些元素来重新构造 HTML 的片段 (如果您想查看的话它们仍然保存在 `parser.pieces` 中) 。最后，我们调用 `parser.output`，它将所有的 HTML 片段连接成一个字符串。 |

## Footnotes

[7] 好吧，其实并不是那么普通的一个问题。在那不都是问 “我应该用何种编辑器来写 Python 代码？” (回答：Emacs) 或 “Python 比 Perl 是好还是坏？” (回答：“Perl 比 Python 差，因为人们想让它差的。” ――Larry Wall，1998 年 10 月 14 日) 但是关于 HTML 处理的问题，或者这种提法或者另一种提法，大约一个月就要出现一次，在这些问题之中，这个问题是最常见的一个。

# 8.8\. `dialect.py` 介绍

# 8.8\. `dialect.py` 介绍

`Dialectizer` 是 `BaseHTMLProcessor` 的简单 (和拙劣) 的派生类。它通过一系列的替换对文本块进行了处理，但是它确保在 `&lt;pre&gt;`...`&lt;/pre&gt;` 块之间的任何东西不被修改地通过。

为了处理 `&lt;pre&gt;` 块，我们在 `Dialectizer` 中定义了两个方法：`start_pre` 和 `end_pre`。

## 例 8.17\. 处理特别标记

```py
 def start_pre(self, attrs):             
        self.verbatim += 1                  
        self.unknown_starttag("pre", attrs) 
    def end_pre(self):                      
        self.unknown_endtag("pre")          
        self.verbatim -= 1 
```

|  |  |
| --- | --- |
| [1] | 每当 `SGMLParser` 在 HTML 源代码中发现一个 `&lt;pre&gt;` 时，都会调用 `start_pre`。(马上我们就会确切地看到它是如何发生的。) 这个方法使用单个参数：`attrs`，这个参数会包含标记的属性 (如果存在的话) 。`attrs` 是一个键/值 tuple 的 list，就像 `unknown_starttag` 中所使用的。 |
| [2] | 在 `reset` 方法中，我们初始化了一个数据属性，它作为 `&lt;pre&gt;` 标记的一个计数器。每当我们找到一个 `&lt;pre&gt;` 标记，我们增加计数器的值；每当我们找到一个 `&lt;/pre&gt;` 标记，我们将减少计数器的值。(我们本可以把它实现为一个标志，即或把它设为 `1`，或重置为 `0`，但这样做只是为了方便，并且这样做可以处理古怪 (但有可能) 的 `&lt;pre&gt;` 标记嵌套的情况。) 马上我们将会看到这个计数器是多么的好用。 |
| [3] | 不错，这就是我们对 `&lt;pre&gt;` 标记所做的唯一的特殊处理。现在我们将属性列表传给 `unknown_starttag`，由它来进行缺省的处理。 |
| [4] | 每当 `SGMLParser` 找到一个 `&lt;/pre&gt;` 标记时，会调用 `end_pre`。因为结束标记不能包含属性，因此这个方法没有参数。 |
| [5] | 首先我们要进行缺省处理，就像其它结束标记做的一样。 |
| [6] | 其次我们将计数器减少，标记这个 `&lt;pre&gt;` 块已经被关闭了。 |

到了这个地方，有必要对 `SGMLParser` 更深入一层。我已经多次声明 (到目前为止您应已经把它做为信条了) ，就是 `SGMLParser` 查找每一个标记并且如果存在特定的方法就调用它们。例如：我们刚刚看到处理 `&lt;pre&gt;` 和 `&lt;/pre&gt;` 的 `start_pre` 和 `end_pre` 的定义。但这是如何发生的呢？嗯，也没什么神奇的，只不过是出色的 Python 编码。

## 例 8.18\. `SGMLParser`

```py
 def finish_starttag(self, tag, attrs):               
        try:                                            
            method = getattr(self, 'start_' + tag)       
        except AttributeError:                           
            try:                                        
                method = getattr(self, 'do_' + tag)      
            except AttributeError:                      
                self.unknown_starttag(tag, attrs)        
                return -1                               
            else:                                       
                self.handle_starttag(tag, method, attrs) 
                return 0                                
        else:                                           
            self.stack.append(tag)                      
            self.handle_starttag(tag, method, attrs)    
            return 1                                     
    def handle_starttag(self, tag, method, attrs):      
        method(attrs) 
```

|  |  |
| --- | --- |
| [1] | 此处，`SGMLParser` 已经找到了一个开始标记，并且分析出属性列表。唯一要做的事情就是检查对于这个标记是否存在一个特别的处理方法，否则我们就应该求助于缺省方法 (`unknown_starttag`) 。 |
| [2] | `SGMLParser` 的 “神奇” 之处除了我们的老朋友 `getattr` 之外就没有什么了。您以前可能没注意到，`getattr` 将查找定义在一个对象的继承者中或对象自身的方法。这里对象是 `self`，即当前实例。所以，如果 `tag` 是 `'pre'`，这里对 `getattr` 的调用将会在当前实例 (它是 `Dialectizer` 类的一个实例) 中查找一个名为 `start_pre` 的方法。 |
| [3] | 如果 `getattr` 所查找的方法在对象或它的任何继承者中不存在的话，它会引发一个 `AttributeError` 的异常。但没有关系，因为我们把对 `getattr` 的调用包装到一个 `try...except` 块中了，并且显式地捕捉 `AttributeError` 异常。 |
| [4] | 因为我们没有找到一个 `start_xxx` 方法，在放弃之前，我们将还要查找一个 `do_xxx` 方法。这个可替换的命名模式一般用于单独的标记，如 `&lt;br&gt;`，这些标记没有相应的结束标记。但是您可以使用任何一种模式，正如您看到的，`SGMLParser` 对每个标记尝试两次。(您不应该对相同的标记同时定义 `start_xxx` 和 `do_xxx` 处理方法，因为这样的话只有 `start_xxx` 方法会被调用。) |
| [5] | 另一个 `AttributeError` 异常，它是说用 `do_xxx` 来调用 `getattr` 失败了。因为对同一个标记我们既没有找到 `start_xxx` 也没有找到 `do_xxx` 处理方法，这样我们捕捉到了异常并且求助于缺省方法：`unknown_starttag`。 |
| [6] | 记得吗？`try...except` 块可以有一个 `else` 子句，当在 `try...except` 块中没有异常被引发时，它将被调用。逻辑上，意味着我们*确实* 找到了这个标记的 `do_xxx` 方法，所以我们将要调用它。 |
| [7] | 顺便说，不要为这些不同的返回值而担心；理论上他们有意义，但实际上它们没有任何用处。也不要担心 `self.stack.append(tag)` ; `SGMLParser` 内部会知晓您的开始标记是否有合适的结束标记与之匹配，但是它不会对这些信息做任何操作。理论上，您能使用这个模块校验您的标记是否完全匹配，但是这或许没有多大价值，并且这样的内容已经超出了本章所要讨论的范畴。现在有您更需要担心的问题。 |
| [8] | `start_xxx` 和 `do_xxx` 方法并不被直接调用；标记名、方法和属性被传给 `handle_starttag` 这个方法，以便继承者可以覆盖它，并改变*全部* 开始标记分发的方式。我们不需要控制这个层面，所以我们只让这个方法做它自已的事，就是用属性 list 来调用方法 (`start_xxx` 或 `do_xxx`) 。记住 `method` 是一个从 `getattr` 返回的函数，而函数是对象。(我知道您已经听腻了，我发誓，一旦我们停止寻找新的使用方法来为我们服务时，我就决不再提它了。) 这时，函数对象作为一个参数传入这个分发方法，这个方法反过来再调用这个函数。在这里，我们不需要知道函数是什么，叫什么名字，或是在哪时定义的；我们只需要知道用一个参数 `attrs` 调用它。 |

现在回到我们已经计划好的程序：`Dialectizer`。当我们跑题时，我们定义了特别的处理方法来处理 `&lt;pre&gt;` 和 `&lt;/pre&gt;` 标记。还有一件事没有做，那就是用我们预定义的替换处理来处理文本块。为了实现它，我们需要覆盖 `handle_data` 方法。

## 例 8.19\. 覆盖 `handle_data` 方法

```py
 def handle_data(self, text):                                         
        self.pieces.append(self.verbatim and text or self.process(text)) 
```

|  |  |
| --- | --- |
| [1] | `handle_data` 在调用时只使用一个参数：要处理的文本。 |
| [2] | 在祖先类 `BaseHTMLProcessor` 中，`handle_data` 方法只是将文本追加到输出缓冲区 `self.pieces` 之后。这里的逻辑稍微有点复杂。如果我们处于 `&lt;pre&gt;`...`&lt;/pre&gt;` 块的中间，`self.verbatim` 将是大于 `0` 的某个值，接着我们想要将文本不作改动地传入输出缓冲区。否则，我们将调用另一个单独的方法来进行替换处理，然后将处理结果放入输出缓冲区中。在 Python 中，这是一个一行代码，它使用了`and-or` 技巧。 |

我们已经接近了对 `Dialectizer` 的全面理解。唯一缺少的一个环节是文本替换的特性。如果您知道点 Perl，您就会知道当需要复杂的文本替换时，唯一有效的解决方法就是正则表达式。在 `dialect.py` 文件后面的几个类中定义了一连串的正则表达式来操作 HTML 标记中的文本。我们已经学习过了正则表达式中的所有字符。我们不必重复学习正则表达式的艰难历程了，不是吗？上帝知道我反正不需要。我想现在这章您已经学得差不多了。

# 8.9\. 全部放在一起

# 8.9\. 全部放在一起

到了将迄今为止我们已经学过并用得不错的东西放在一起的时候了。我希望您专心些。

## 例 8.20\. `translate` 函数，第 1 部分

```py
 def translate(url, dialectName="chef"): 
    import urllib                       
    sock = urllib.urlopen(url)          
    htmlSource = sock.read()           
    sock.close() 
```

|  |  |
| --- | --- |
| [1] | 这个 `translate` 函数有一个可选参数 `dialectName`，它是一个字符串，指出我们将使用的方言。一会我们就会看到它是如何使用的。 |
| [2] | 嘿，等一下，在这个函数中有一个 `import` 语句！它在 Python 中完全合法。您已经习惯了在一个程序的前面看到 `import` 语句，它意味着导入的模块在程序的任何地方都是可用的。但您也可以在一个函数中导入模块，这意味着导入的模块只能在函数中使用。如果您有一个只能用在一个函数中的模块，这是一个简便的方法，使您的代码更模块化。(当发现您周末的加班已经变成了一个 800 行的艺术作品，并且决定将其分割成一打可重用的模块时，您会感谢它的。) |
| [3] | 现在我们得到了给定的 URL 源文件。 |

## 例 8.21\. `translate` 函数，第 2 部分：奇妙而又奇妙

```py
 parserName = "%sDialectizer" % dialectName.capitalize() 
    parserClass = globals()[parserName]                     
    parser = parserClass() 
```

|  |  |
| --- | --- |
| [1] | `capitalize` 是一个我们以前未曾见过的字符串方法；它只是将一个字符串的第一个字母变成大写，将其它的字母强制变成小写。再使用字符串格式化，我们就得到了一种方言的名字，并将它转化为了相应的方言变换器类的名字。如果 `dialectName` 是字符串 `'chef'`，`parserName` 将是字符串 `'ChefDialectizer'`。 |
| [2] | 我们有了一个字符串形式 (`parserName`) 的类名称，还有一个 dictionary (`globals`()) 形式的全局名字空间。合起来后，我们可以得到以前者命名的类的引用。(回想一下，类是对象，并且它们可以像其它对象一样赋值给一个变量。) 如果 `parserName` 是字符串 `'ChefDialectizer'`，`parserClass` 将是类 `ChefDialectizer`。 |
| [3] | 最后，我们拥有了一个类对象 (`parserClass`)，接着我们想要生成这个类的一个实例。好，我们已经知道如何去做了：像函数一样调用类。这个类保存在一个局部变量中，但这个事实完全不会有什么影响；我们只是像函数一样调用这个局部变量，取出这个类的一个实例。如果 `parserClass` 是类 `ChefDialectizer`，`parser` 将是类 `ChefDialectizer` 的一个实例。 |

何必这么麻烦？毕竟只有三个 `Dialectizer` 类；为什么不只使用一个 `case` 语句？ (噢，在 Python 中不存在 `case` 语句，但为什么不只使用一组 `if` 语句呢？) 理由之一是：可扩展性。这个 `translate` 函数完全不用关心我们定义了多少个方言变换器类。设想一下，如果我们明天定义了一个新的 `FooDialectizer` 类，把 `'foo'` 作为 `dialectName` 传给 `translate` ，`translate` 也能工作。

甚至会更好。设想将 `FooDialectizer` 放进一个独立的模块中，使用 `from _module_ import` 将其导入。我们已经知道了，这样会将它包含在 `globals`() 中 ，所以不用修改 `translate` ，它仍然可以正确运行，尽管 `FooDialectizer` 位于一个独立的文件中。

现在设想一下方言的名字是从程序外面的某个地方来的，也许是从一个数据库中，或从一个表格中的用户输入的值中。您可以使用任意多的服务端 Python 脚本架构来动态地生成网页；这个函数将接收在页面请求的查询字符串中的一个 URL 和一个方言名字 (两个都是字符串) ，接着输出 “翻译” 后的网页。

最后，设想一下，使用了一种插件架构的 `Dialectizer` 框架。您可以将每个 `Dialectizer` 类放在分别放在独立的文件中，在 `dialect.py` 中只留下 `translate` 函数。假定一种统一的命名模式，这个 `translate` 函数能够动态地从合适的文件中导入合适的类，除了方言名字外什么都不用给出。(虽然您还没有看过动态导入，但我保证在后面的一章中会涉及到它。) 如果要加入一种新的方言，您只要在插件目录下加入一个以合适的名字命名的文件 (像 `foodialect.py`，它包含了 `FooDialectizer` 类) 。使用方言名 `'foo'` 来调用这个 `translate` 函数，将会查找 `foodialect.py` 模块，导入 `FooDialectizer` 类，这样就行了。

## 例 8.22\. `translate` 函数，第 3 部分

```py
 parser.feed(htmlSource) 
    parser.close()          
    return parser.output() 
```

|  |  |
| --- | --- |
| [1] | 剩下的工作似乎会非常无聊，但实际上，`feed` 函数执行了全部的转换工作。我们拥有存在于单个字符串中的全部 HTML 源代码，所以我们只需要调用 `feed` 一次。然而，您可以按您的需要经常调用 `feed`，分析器将不停地进行分析。所以如果我们担心内存的使用 (或者我们已经知道了将要处理非常巨大的 HTML 页面) ，我们可以在一个循环中调用它，即我们读出一点 HTML 字节，就将其送进分析器。结果会是一样的。 |
| [2] | 因为 `feed` 维护着一个内部缓冲区，当您完成时，应该总是调用分析器的 `close` 方法 (那怕您像我们做的一样，一次就全部送出) 。否则您可能会发现，输出丢掉了最后几个字节。 |
| [3] | 回想一下，`output` 是我们在 `BaseHTMLProcessor` 上定义的函数，用来将所有缓冲的输出片段连接起来并且以单个字符串返回。 |

像这样，我们已经 “翻译” 了一个网页，除了给出一个 URL 和一种方言的名字外，什么都没有给出。

## 进一步阅读

*   您可能会认为我的服务端脚本编程的想法是开玩笑。在我发现这个[基于 web 的方言转换器](http://rinkworks.com/dialect/)之前，的确是这样想的。不幸的是，看不到它的源代码。

# 8.10\. 小结

# 8.10\. 小结

Python 向您提供了一个强大工具，`sgmllib.py`，可以通过将 HTML 结构转变为一种对象模型来进行处理。可以以许多不同的方式来使用这个工具。

*   对 HTML 进行分析，搜索特别的东西
*   摘录结果，如 URL lister
*   在处理过程中顺便调整结构，如给属性值加引号
*   将 HTML 转换为其它的东西，通过对文本进行处理，同时保留标记，如 `Dialectizer`

学过了这些例子之后，您应该无障碍地完成下面的事情：

*   使用 `locals`() 和 `globals`() 来访问名字空间
*   使用基于 dictionary 替换的字符串格式化