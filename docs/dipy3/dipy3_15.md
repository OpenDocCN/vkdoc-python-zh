# Chapter 12 XML

# Chapter 12 XML

> " In the archonship of Aristaechmus, Draco enacted his ordinances. " — [Aristotle](http://www.perseus.tufts.edu/cgi-bin/ptext?doc=Perseus:text:1999.01.0046;query=chapter%3D%235;layout=;loc=3.1)

## 概述

这本书的大部分章节都是以样例代码为中心的。但是 XML 这章不是；它以数据为中心。最常见的 XML 应用为“聚合订阅(syndication feeds)”，它用来展示博客，论坛或者其他会经常更新的网站的最新内容。大多数的博客软件都会在新文章，新的讨论区，或者新博文发布的时候自动生成和更新 feed。我们可以通过“订阅(subscribe)”feed 来关注它们，还可以使用专门的“[feed 聚合工具(feed aggregator)](http://en.wikipedia.org/wiki/List_of_feed_aggregators)”，比如[Google Reader](http://www.google.com/reader/)。

以下的 XML 数据是我们这一章中要用到的。它是一个 feed — 更确切地说是一个[Atom 聚合 feed](http://atompub.org/rfc4287.html)

```py
<?xml version='1.0' encoding='utf-8'?>
<feed xmlns='http://www.w3.org/2005/Atom' xml:lang='en'>
  <title>dive into mark</title>
  <subtitle>currently between addictions</subtitle>
  <id>tag:diveintomark.org,2001-07-29:/</id>
  <updated>2009-03-27T21:56:07Z</updated>
  <link rel='alternate' type='text/html' href='http://diveintomark.org/'/>
  <link rel='self' type='application/atom+xml' href='http://diveintomark.org/feed/'/>
  <entry>
    <author>
      <name>Mark</name>
      <uri>http://diveintomark.org/</uri>
    </author>
    <title>Dive into history, 2009 edition</title>
    <link rel='alternate' type='text/html'
      href='http://diveintomark.org/archives/2009/03/27/dive-into-history-2009-edition'/>
    <id>tag:diveintomark.org,2009-03-27:/archives/20090327172042</id>
    <updated>2009-03-27T21:56:07Z</updated>
    <published>2009-03-27T17:20:42Z</published>
    <category scheme='http://diveintomark.org' term='diveintopython'/>
    <category scheme='http://diveintomark.org' term='docbook'/>
    <category scheme='http://diveintomark.org' term='html'/>
  <summary type='html'>Putting an entire chapter on one page sounds
    bloated, but consider this &amp;mdash; my longest chapter so far
    would be 75 printed pages, and it loads in under 5 seconds&amp;hellip;
    On dialup.</summary>
  </entry>
  <entry>
    <author>
      <name>Mark</name>
      <uri>http://diveintomark.org/</uri>
    </author>
    <title>Accessibility is a harsh mistress</title>
    <link rel='alternate' type='text/html'
      href='http://diveintomark.org/archives/2009/03/21/accessibility-is-a-harsh-mistress'/>
    <id>tag:diveintomark.org,2009-03-21:/archives/20090321200928</id>
    <updated>2009-03-22T01:05:37Z</updated>
    <published>2009-03-21T20:09:28Z</published>
    <category scheme='http://diveintomark.org' term='accessibility'/>
    <summary type='html'>The accessibility orthodoxy does not permit people to
      question the value of features that are rarely useful and rarely used.</summary>
  </entry>
  <entry>
    <author>
      <name>Mark</name>
    </author>
    <title>A gentle introduction to video encoding, part 1: container formats</title>
    <link rel='alternate' type='text/html'
      href='http://diveintomark.org/archives/2008/12/18/give-part-1-container-formats'/>
    <id>tag:diveintomark.org,2008-12-18:/archives/20081218155422</id>
    <updated>2009-01-11T19:39:22Z</updated>
    <published>2008-12-18T15:54:22Z</published>
    <category scheme='http://diveintomark.org' term='asf'/>
    <category scheme='http://diveintomark.org' term='avi'/>
    <category scheme='http://diveintomark.org' term='encoding'/>
    <category scheme='http://diveintomark.org' term='flv'/>
    <category scheme='http://diveintomark.org' term='GIVE'/>
    <category scheme='http://diveintomark.org' term='mp4'/>
    <category scheme='http://diveintomark.org' term='ogg'/>
    <category scheme='http://diveintomark.org' term='video'/>
    <summary type='html'>These notes will eventually become part of a
      tech talk on video encoding.</summary>
  </entry>
</feed> 
```

## 5 分钟 XML 速成

如果你已经了解 XML，可以跳过这一部分。

XML 是一种描述层次结构化数据的通用方法。XML*文档*包含由*起始和结束标签(tag)*分隔的一个或多个*元素(element)*。以下也是一个完整的(虽然空洞)XML 文件：

1.  这是`foo`元素的*起始标签*。
2.  这是`foo`元素对应的*结束标签*。就如写作、数学或者代码中需要平衡括号一样，每一个起始标签必须有对应的结束标签来*闭合*（匹配）。

元素可以*嵌套*到任意层次。位于`foo`中的元素`bar`可以被称作其*子元素*。

```py
<foo>
  <mark><bar></bar></mark>
</foo> 
```

XML 文档中的第一个元素叫做*根元素(root element)*。并且每份 XML 文档只能有一个根元素。以下不是一个 XML 文档，因为它存在两个“根元素”。

```py
<foo></foo>
<bar></bar> 
```

元素可以有其*属性(attribute)*，它们是一些名字-值(name-value)对。属性由空格分隔列举在元素的起始标签中。一个元素中*属性名*不能重复。*属性值*必须用引号包围起来。单引号、双引号都是可以。

```py
</foo> 
```

1.  `foo`元素有一个叫做`lang`的属性。`lang`的值为`en`
2.  `bar`元素则有两个属性，分别为`id`和`lang`。其中`lang`属性的值为`fr`。它不会与`foo`的那个属性产生冲突。每个元素都其独立的属性集。

如果元素有多个属性，书写的顺序并不重要。元素的属性是一个无序的键-值对集，跟 Python 中的列表对象一样。另外，元素中属性的个数是没有限制的。

元素可以有其*文本内容(text content)*

```py
<foo lang='en'>
  <bar lang='fr'><mark>PapayaWhip</mark></bar>
</foo> 
```

如果某一元素既没有文本内容，也没有子元素，它也叫做*空元素*。

```py
<foo></foo> 
```

表达空元素有一种简洁的方法。通过在起始标签的尾部添加`/`字符，我们可以省略结束标签。上一个例子中的 XML 文档可以写成这样：

```py
<foo<mark>/</mark>> 
```

就像 Python 函数可以在不同的*模块(modules)*中声明一样，也可以在不同的*名字空间(namespace)*中声明 XML 元素。XML 文档的名字空间通常看起来像 URL。我们可以通过声明`xmlns`来定义*默认名字空间*。名字空间声明跟元素属性看起来很相似，但是它们的作用是不一样的。

```py
</feed> 
```

1.  `feed`元素处在名字空间`http://www.w3.org/2005/Atom`中。
2.  `title`元素也是。名字空间声明不仅会作用于当前声明它的元素，还会影响到该元素的所有子元素。

也可以通过`xmlns:`prefix``声明来定义一个名字空间并取其名为 _prefix_。然后该名字空间中的每个元素都必须显式地使用这个前缀(`prefix`)来声明。

```py
 </atom:feed> 
```

1.  `feed`元素属于名字空间`http://www.w3.org/2005/Atom`。
2.  `title`元素也在那个名字空间。

对于 XML 解析器而言，以上两个 XML 文档是*一样的*。名字空间 + 元素名 = XML 标识。前缀只是用来引用名字空间的，所以对于解析器来说，这些前缀名(`atom:`)其实无关紧要的。名字空间相同，元素名相同，属性（或者没有属性）相同，每个元素的文本内容相同，则 XML 文档相同。

最后，在根元素之前，字符编码信息可以出现在 XML 文档的第一行。（这里存在一个两难的局面(catch-22)，直观上来说，解析 XML 文档需要这些编码信息，而这些信息又存在于 XML 文档中，如果你对 XML 如何解决此问题有兴趣，请参阅[XML 规范中 F 章节](http://www.w3.org/TR/REC-xml/#sec-guessing-no-ext-info)）

```py
<?xml version='1.0' <mark>encoding='utf-8'</mark>?> 
```

现在我们已经知道足够多的 XML 知识，可以开始探险了！

## Atom Feed 的结构

想像一下网络上的博客，或者互联网上任何需要频繁更新的网站，比如[CNN.com](http://www.cnn.com/)。该站点有一个标题(“CNN.com”)，一个子标题(“Breaking News, U.S., World, Weather, Entertainment *&* Video News”)，包含上次更新的日期(“updated 12:43 p.m. EDT, Sat May 16, 2009”)，还有在不同时期发布的文章的列表。每一篇文章也有自己的标题，第一次发布的日期（如果曾经修订过或者改正过某个输入错误，或许也有一个上次更新的日期），并且每篇文章有自己唯一的 URL。

Atom 聚合格式被设计成可以包含所有这些信息的标准格式。我的博客无论在设计，主题还是读者上都与 CNN.com 大不相同，但是它们的基本结构是相同的。CNN.com 能做的事情，我的博客也能做…

每一个 Atom 订阅都共享着一个*根元素*：即在名字空间`http://www.w3.org/2005/Atom`中的元素`feed`。

1.  `http://www.w3.org/2005/Atom`表示名字空间 Atom。
2.  每一个元素都可以包含`xml:lang`属性，它用来声明该元素及其子元素使用的语言。在当前样例中，`xml:lang`在根元素中被声明了一次，也就意味着，整个 feed 都使用英文。

描述 Atom feed 自身的一些信息在根元素`feed`的子元素中被声明。

```py
<feed xmlns='http://www.w3.org/2005/Atom' xml:lang='en'> 
```

1.  该行表示这个 feed 的标题为`dive into mark`。
2.  这一行表示子标题为`currently between addictions`。
3.  每一个 feed 都要有一个全局唯一标识符(globally unique identifier)。想要知道如何创建它，请查阅[RFC 4151](http://www.ietf.org/rfc/rfc4151.txt)。
4.  表示当前 feed 上次更新的时间为 March 27, 2009, at 21:56 GMT。通常来说，它与最近一篇文章最后一次被修改的时间是一样的。
5.  事情开始变得有趣了…`link`元素没有文本内容，但是它有三个属性：`rel`，`type`和`href`。`rel`元素的值能告诉我们链接的类型；`rel='alternate'`表示这个链接指向当前 feed 的另外一个版本。`type='text/html'`表示链接的目标是一个 HTML 页面。然后目标地址在`href`属性中指出。

现在我们知道这个 feed 上一更新是在 on March 27, 2009，它是为一个叫做“dive into mark”的站点准备的，并且站点的地址为[`http://diveintomark.org/`](http://diveintomark.org/)。

> ☞在有一些 XML 文档中，元素的排列顺序是有意义的，但是 Atom feed 中不需要这样做。

feed 级的元数据后边就是最近文章的列表了。单独的一篇文章就像这样：

```py
<entry>

    <name>Mark</name>
    <uri>http://diveintomark.org/</uri>
  </author>

    href='http://diveintomark.org/archives/2009/03/27/dive-into-history-2009-edition'/>

  <published>2009-03-27T17:20:42Z</published>        

  <category scheme='http://diveintomark.org' term='docbook'/>
  <category scheme='http://diveintomark.org' term='html'/>

    bloated, but consider this &amp;mdash; my longest chapter so far
    would be 75 printed pages, and it loads in under 5 seconds&amp;hellip;
    On dialup.</summary> 
```

1.  `author`元素指示文章的作者：一个叫做 Mark 的伙计，并且我们可以在`http://diveintomark.org/`找到他的事迹。（这就像是 feed 元素里的备用链接，但是没有规定一定要这样。许多网络日志由多个作者完成，他们都有自己的个人主页。）* `title`元素给出这篇文章的标题，即“Dive into history, 2009 edition”。
2.  `如`feed`元素中的备用链接一样，`link`元素给出这篇文章的 HTML`版本地址。
3.  每个条目也像 feed 一样，需要一个唯一的标识。
4.  每个条目有两个日期与其相关：第一次发布日期(`published`)和上次修改日期(`updated`)。
5.  条目可以属于任意多个类别。这篇文章被归类到`diveintopython`，`docbook`，和`html`。
6.  `summary`元素中有这篇文章的概要性描述。（还有一个元素这里没有展示出来，即`content`，我们可以把整篇文章的内容都放在里边。）当前样例中，`summary`元素含有一个 Atom 特有的`type='html'`属性，它用来告知这份概要为 HTML 格式，而非纯文本。这非常重要，因为概要内容中包含了 HTML 中特有的实体（`&mdash;`和`&hellip;`），它们不应该以纯文本直接显示，正确的形式应该为“—”和“…”。
7.  最后就是`entry`元素的结束标记了，它指示文章元数据的结尾。

## 解析 XML

Python 可以使用几种不同的方式解析 XML 文档。它包含了[DOM](http://en.wikipedia.org/wiki/XML#DOM)和[SAX](http://en.wikipedia.org/wiki/Simple_API_for_XML)解析器，但是我们焦点将放在另外一个叫做 ElementTree 的库上边。

```py
 <Element {http://www.w3.org/2005/Atom}feed at cd1eb0> 
```

1.  ElementTree 属于 Python 标准库的一部分，它的位置为`xml.etree.ElementTree`。
2.  `parse()`函数是 ElementTree 库的主要入口，它使用文件名或者流对象作为参数。`parse()`函数会立即解析完整个文档。如果内存资源紧张，也可以[增量式地解析 XML 文档](http://effbot.org/zone/element-iterparse.htm)
3.  `parse()`函数会返回一个能代表整篇文档的对象。这*不是*根元素。要获得根元素的引用可以调用`getroot()`方法。
4.  如预期的那样，根元素即`http://www.w3.org/2005/Atom`名字空间中的`feed`。该字符串表示再次重申了非常重要的一点：XML 元素由名字空间和标签名（也称作*本地名(local name)*）组成。这篇文档中的每个元素都在名字空间 Atom 中，所以根元素被表示为`{http://www.w3.org/2005/Atom}feed`。

> ☞ElementTree 使用`{`namespace`}`localname``来表达 XML 元素。我们将会在 ElementTree 的 API 中多次见到这种形式。

### 元素即列表

在 ElementTree API 中，元素的行为就像列表一样。列表中的项即该元素的子元素。

```py
# continued from the previous example

'{http://www.w3.org/2005/Atom}feed'

8

... 
<Element {http://www.w3.org/2005/Atom}title at e2b5d0>
<Element {http://www.w3.org/2005/Atom}subtitle at e2b4e0>
<Element {http://www.w3.org/2005/Atom}id at e2b6c0>
<Element {http://www.w3.org/2005/Atom}updated at e2b6f0>
<Element {http://www.w3.org/2005/Atom}link at e2b4b0>
<Element {http://www.w3.org/2005/Atom}entry at e2b720>
<Element {http://www.w3.org/2005/Atom}entry at e2b510>
<Element {http://www.w3.org/2005/Atom}entry at e2b750> 
```

1.  紧接前一例子，根元素为`{http://www.w3.org/2005/Atom}feed`。
2.  根元素的“长度”即子元素的个数。
3.  我们可以像使用迭代器一样来遍历其子元素。
4.  从输出可以看到，根元素总共有 8 个子元素：所有 feed 级的元数据（`title`，`subtitle`，`id`，`updated`和`link`），还有紧接着的三个`entry`元素。

也许你已经注意到了，但我还是想要指出来：该列表只包含*直接*子元素。每一个`entry`元素都有其子元素，但是并没有包括在这个列表中。这些子元素本可以包括在`entry`元素的列表中，但是确实不属于`feed`的子元素。但是，无论这些元素嵌套的层次有多深，总是有办法定位到它们的；在这章的后续部分我们会介绍两种方法。

### 属性即字典

XML 不只是元素的集合；每一个元素还有其属性集。一旦获取了某个元素的引用，我们可以像操作 Python 的字典一样轻松获取到其属性。

```py
# continuing from the previous example

{'{http://www.w3.org/XML/1998/namespace}lang': 'en'}

<Element {http://www.w3.org/2005/Atom}link at e181b0>

{'href': 'http://diveintomark.org/',
 'type': 'text/html',
 'rel': 'alternate'}

<Element {http://www.w3.org/2005/Atom}updated at e2b4e0>

{} 
```

1.  `attrib`是一个代表元素属性的字典。这个地方原来的标记语言是这样描述的：`&lt;feed xmlns='http://www.w3.org/2005/Atom' xml:lang='en'&gt;`。前缀`xml:`指示一个内置的名字空间，每一个 XML 不需要声明就可以使用它。
2.  第五个子元素 — 以 0 为起始的列表中即`[4]` — 为元素`link`。
3.  `link`元素有三个属性：`href`，`type`，和`rel`。
4.  第四个子元素 — `[3]` — 为`updated`。
5.  元素`updated`没有子元素，所以`.attrib`是一个空的字典对象。

## 在 XML 文档中查找结点

到目前为止，我们已经“自顶向下“地从根元素开始，一直到其子元素，走完了整个文档。但是许多情况下我们需要找到 XML 中特定的元素。Etree 也能完成这项工作。

```py
>>> import xml.etree.ElementTree as etree
>>> tree = etree.parse('examples/feed.xml')
>>> root = tree.getroot()

[<Element {http://www.w3.org/2005/Atom}entry at e2b4e0>,
 <Element {http://www.w3.org/2005/Atom}entry at e2b510>,
 <Element {http://www.w3.org/2005/Atom}entry at e2b540>]
>>> root.tag
'{http://www.w3.org/2005/Atom}feed'

[]

[] 
```

1.  `findfall()`方法查找匹配特定格式的子元素。（关于查询的格式稍后会讲到。）
2.  每个元素 — 包括根元素及其子元素 — 都有`findall()`方法。它会找到所有匹配的子元素。但是为什么没有看到任何结果呢？也许不太明显，这个查询只会搜索其子元素。由于根元素`feed`中不存在任何叫做`feed`的子元素，所以查询的结果为一个空的列表。
3.  这个结果也许也在你的意料之外。在这篇文档中确实存在`author`元素；事实上总共有三个（每个`entry`元素中都有一个）。但是那些`author`元素不是根元素的*直接子元素*。我们可以在任意嵌套层次中查找`author`元素，但是查询的格式会有些不同。

```py
 [<Element {http://www.w3.org/2005/Atom}entry at e2b4e0>,
 <Element {http://www.w3.org/2005/Atom}entry at e2b510>,
 <Element {http://www.w3.org/2005/Atom}entry at e2b540>]

[] 
```

1.  为了方便，对象`tree`（调用`etree.parse()`的返回值）中的一些方法是根元素中这些方法的镜像。在这里，如果调用`tree.getroot().findall()`，则返回值是一样的。
2.  也许有些意外，这个查询请求也没有找到文档中的`author`元素。为什么没有呢？因为它只是`tree.getroot().findall('{http://www.w3.org/2005/Atom}author')`的一种简洁表示，即“查询所有是根元素的子元素的`author`”。因为这些`author`是`entry`元素的子元素，所以查询没有找到任何匹配的。

`find()`方法用来返回第一个匹配到的元素。当我们认为只会有一个匹配，或者有多个匹配但我们只关心第一个的时候，这个方法是很有用的。

```py
 >>> len(entries)
3

>>> title_element.text
'Dive into history, 2009 edition'

>>> foo_element
>>> type(foo_element)
<class 'NoneType'> 
```

1.  在前一样例中已经看到。这一句返回所有的`atom:entry`元素。
2.  `find()`方法使用 ElementTree 作为参数，返回第一个匹配到的元素。
3.  在`entries[0]`中没有叫做`foo`的元素，所以返回值为`None`。

> ☞可逮住你了，在这里`find()`方法非常容易被误解。在布尔上下文中，如果 ElementTree 元素对象不包含子元素，其值则会被认为是`False`（*即*如果`len(element)`等于 0）。这就意味着`if element.find('...')`并非在测试是否`find()`方法找到了匹配项；这条语句是在测试匹配到的元素是否包含子元素！想要测试`find()`方法是否返回了一个元素，则需使用`if element.find('...') is not None`。

也*可以*在所有*派生(descendant)*元素中搜索，*即*任意嵌套层次的子元素，孙子元素等…

```py
 >>> all_links
[<Element {http://www.w3.org/2005/Atom}link at e181b0>,
 <Element {http://www.w3.org/2005/Atom}link at e2b570>,
 <Element {http://www.w3.org/2005/Atom}link at e2b480>,
 <Element {http://www.w3.org/2005/Atom}link at e2b5a0>]

{'href': 'http://diveintomark.org/',
 'type': 'text/html',
 'rel': 'alternate'}

{'href': 'http://diveintomark.org/archives/2009/03/27/dive-into-history-2009-edition',
 'type': 'text/html',
 'rel': 'alternate'}
>>> all_links[2].attrib
{'href': 'http://diveintomark.org/archives/2009/03/21/accessibility-is-a-harsh-mistress',
 'type': 'text/html',
 'rel': 'alternate'}
>>> all_links[3].attrib
{'href': 'http://diveintomark.org/archives/2008/12/18/give-part-1-container-formats',
 'type': 'text/html',
 'rel': 'alternate'} 
```

1.  `//{http://www.w3.org/2005/Atom}link`与前一样例很相似，除了开头的两条斜线。这两条斜线告诉`findall()`方法“不要只在直接子元素中查找；查找的范围可以是*任意*嵌套层次”。
2.  查询到的第一个结果*是*根元素的直接子元素。从它的属性中可以看出，它是一个指向该 feed 的 HTML 版本的备用链接。
3.  其他的三个结果分别是低一级的备用链接。每一个`entry`都有单独一个`link`子元素，由于在查询语句前的两条斜线的作用，我们也能定位到他们。

总的来说，ElementTree 的`findall()`方法是其一个非常强大的特性，但是它的查询语言却让人有些出乎意料。官方描述它为“[有限的 XPath 支持](http://effbot.org/zone/element-xpath.htm)。”[XPath](http://www.w3.org/TR/xpath)是一种用于查询 XML 文档的 W3C 标准。对于基础地查询来说，ElementTree 与 XPath 语法上足够相似，但是如果已经会 XPath 的话，它们之间的差异可能会使你感到不快。现在，我们来看一看另外一个第三方 XML 库，它扩展了 ElementTree 的 API 以提供对 XPath 的全面支持。

## 深入 lxml

[`lxml`](http://codespeak.net/lxml/)是一个开源的第三方库，以流行的[libxml2 解析器](http://www.xmlsoft.org/)为基础开发。提供了与 ElementTree 完全兼容的 API，并且扩展它以提供了对 XPath 1.0 的全面支持，以及改进了一些其他精巧的细节。提供[Windows 的安装程序](http://pypi.python.org/pypi/lxml/)；Linux 用户推荐使用特定发行版自带的工具比如`yum`或者`apt-get`从它们的程序库中安装预编译好了的二进制文件。要不然，你就得手工[安装](http://codespeak.net/lxml/installation.html)他们了。

```py
 [<Element {http://www.w3.org/2005/Atom}entry at e2b4e0>,
 <Element {http://www.w3.org/2005/Atom}entry at e2b510>,
 <Element {http://www.w3.org/2005/Atom}entry at e2b540>] 
```

1.  导入`lxml`以后，可以发现它与内置的 ElementTree 库提供相同的 API。
2.  `parse()`函数：与 ElementTree 相同。
3.  `getroot()`方法：相同。
4.  `findall()`方法：完全相同。

对于大型的 XML 文档，`lxml`明显比内置的 ElementTree 快了许多。如果现在只用到了 ElementTree 的 API，并且想要使用其最快的实现(implementation)，我们可以尝试导入`lxml`，并且将内置的 ElementTree 作为备用。

```py
try:
    from lxml import etree
except ImportError:
    import xml.etree.ElementTree as etree 
```

但是`lxml`不只是一个更快速的 ElementTree。它的`findall()`方法能够支持更加复杂的表达式。

```py
 >>> tree = lxml.etree.parse('examples/feed.xml')

[<Element {http://www.w3.org/2005/Atom}link at eeb8a0>,
 <Element {http://www.w3.org/2005/Atom}link at eeb990>,
 <Element {http://www.w3.org/2005/Atom}link at eeb960>,
 <Element {http://www.w3.org/2005/Atom}link at eeb9c0>]

[<Element {http://www.w3.org/2005/Atom}link at eeb930>]
>>> NS = '{http://www.w3.org/2005/Atom}'

[<Element {http://www.w3.org/2005/Atom}author at eeba80>,
 <Element {http://www.w3.org/2005/Atom}author at eebba0>] 
```

1.  在这个样例中，我使用了`import lxml.etree`（而非`from lxml import etree`），以强调这些特性只限于`lxml`。
2.  这一句在整个文档范围内搜索名字空间 Atom 中具有`href`属性的所有元素。在查询语句开头的`//`表示“搜索的范围为整个文档（不只是根元素的子元素）。” `{http://www.w3.org/2005/Atom}`指示“搜索范围仅在名字空间 Atom 中。” `*` 表示“任意本地名(local name)的元素。” `[@href]`表示“含有`href`属性。”
3.  该查询找出所有包含`href`属性并且其值为`http://diveintomark.org/`的 Atom 元素。
4.  在简单的字符串格式化后（要不然这条复合查询语句会变得特别长），它搜索名字空间 Atom 中包含`uri`元素作为子元素的`author`元素。该条语句只返回了第一个和第二个`entry`元素中的`author`元素。最后一个`entry`元素中的`author`只包含有`name`属性，没有`uri`。

仍然不够用？`lxml`也集成了对任意 XPath 1.0 表达式的支持。我们不会深入讲解 XPath 的语法；那可能需要一整本书！但是我会给你展示它是如何集成到`lxml`去的。

```py
>>> import lxml.etree
>>> tree = lxml.etree.parse('examples/feed.xml')

...     namespaces=NSMAP)

[<Element {http://www.w3.org/2005/Atom}entry at e2b630>]
>>> entry = entries[0]

['Accessibility is a harsh mistress'] 
```

1.  要查询名字空间中的元素，首先需要定义一个名字空间前缀映射。它就是一个 Python 字典对象。
2.  这就是一个 XPath 查询请求。这个 XPath 表达式目的在于搜索`category`元素，并且该元素包含有值为`accessibility`的`term`属性。但是那并不是查询的结果。请看查询字符串的尾端；是否注意到了`/..`这一块？它的意思是，“然后返回已经找到的`category`元素的父元素。”所以这条 XPath 查询语句会找到所有包含`&lt;category term='accessibility'&gt;`作为子元素的条目。
3.  `xpath()`函数返回一个 ElementTree 对象列表。在这篇文档中，只有一个`category`元素，并且它的`term`属性值为`accessibility`。
4.  XPath 表达式并不总是会返回一个元素列表。技术上说，一个解析了的 XML 文档的 DOM 模型并不包含元素；它只包含*结点(node)*。依据它们的类型，结点可以是元素，属性，甚至是文本内容。XPath 查询的结果是一个结点列表。当前查询返回一个文本结点列表：`title`元素(`atom:title`)的文本内容(`text()`)，并且`title`元素必须是当前元素的子元素(`./`)。

## 生成 XML

Python 对 XML 的支持不只限于解析已存在的文档。我们也可以从头来创建 XML 文档。

```py
>>> import xml.etree.ElementTree as etree

<ns0:feed xmlns:ns0='http://www.w3.org/2005/Atom' xml:lang='en'/> 
```

1.  实例化`Element`类来创建一个新元素。可以将元素的名字（名字空间 + 本地名）作为其第一个参数。当前语句在 Atom 名字空间中创建一个`feed`元素。它将会成为我们文档的根元素。
2.  将属性名和值构成的字典对象传递给`attrib`参数，这样就可以给新创建的元素添加属性。请注意，属性名应该使用标准的 ElementTree 格式，`{`namespace`}`localname``。
3.  在任何时候，我们可以使用 ElementTree 的`tostring()`函数序列化任意元素（还有它的子元素）。

这种序列化结果有使你感到意外吗？技术上说，ElementTree 使用的序列化方法是精确的，但却不是最理想的。在本章开头给出的 XML 样例文档中定义了一个*默认名字空间(default namespace)*(`xmlns='http://www.w3.org/2005/Atom'`)。对于每个元素都在同一个名字空间中的文档 — 比如 Atom feeds — 定义默认的名字空间非常有用，因为只需要声明一次名字空间，然后在声明每个元素的时候只需要使用其本地名即可(`&lt;feed&gt;`，`&lt;link&gt;`，`&lt;entry&gt;`)。除非想要定义另外一个名字空间中的元素，否则没有必要使用前缀。

对于 XML 解析器来说，它不会“注意”到使用默认名字空间和使用前缀名字空间的 XML 文档之间有什么不同。当前序列化结果的 DOM 为：

```py
<ns0:feed xmlns:ns0='http://www.w3.org/2005/Atom' xml:lang='en'/> 
```

与下列序列化的 DOM 是一模一样的：

```py
<feed xmlns='http://www.w3.org/2005/Atom' xml:lang='en'/> 
```

实际上唯一不同的只是第二个序列化短了几个字符长度。如果我们改动整个样例 feed，使每一个起始和结束标签都有一个`ns0:`前缀，这将为每个起始标签增加 4 个字符 × 79 个标签 + 4 个名字空间声明本身用到的字符，总共 320 个字符。假设我们使用 UTF-8 编码，那将是 320 个额外的字节。（使用 gzip 压缩以后，大小可以降到 21 个字节，但是，21 个字节也是字节。）也许对个人来说这算不了什么，但是对于像 Atom feed 这样的东西，只要稍有改变就有可能被下载上千次，每一个请求节约的几个字节就会迅速累加起来。

内置的 ElementTree 库没有提供细粒度地对序列化时名字空间内的元素的控制，但是`lxml`有这样的功能。

```py
>>> import lxml.etree

<feed xmlns='http://www.w3.org/2005/Atom'/>

>>> print(lxml.etree.tounicode(new_feed))
<feed xmlns='http://www.w3.org/2005/Atom' xml:lang='en'/> 
```

1.  首先，定义一个用于名字空间映射的字典对象。其值为名字空间；字典中的键即为所需要的前缀。使用`None`作为前缀来定义默认的名字空间。
2.  现在我们可以在创建元素的时候，给`lxml`专有的`nsmap`参数传值，并且`lxml`会参照我们所定义的名字空间前缀。
3.  如所预期的那样，该序列化使用 Atom 作为默认的名字空间，并且在声明`feed`元素的时候没有使用名字空间前缀。
4.  啊噢… 我们忘了加上`xml:lang`属性。我们可以使用`set()`方法来随时给元素添加所需属性。该方法使用两个参数：标准 ElementTree 格式的属性名，然后，属性值。（该方法不是`lxml`特有的。在该样例中，只有`nsmap 参数是`lxml`特有的，它用来控制序列化输出时名字空间的前缀。）`

难道每个 XML 文档只能有一个元素吗？当然不了。我们可以创建子元素。

```py
 >>> print(lxml.etree.tounicode(new_feed))
<feed xmlns='http://www.w3.org/2005/Atom' xml:lang='en'><title type='html'/></feed>

<feed xmlns='http://www.w3.org/2005/Atom' xml:lang='en'><title type='html'>dive into &amp;hellip;</title></feed>

<feed xmlns='http://www.w3.org/2005/Atom' xml:lang='en'>
<title type='html'>dive into&amp;hellip;</title>
</feed> 
```

1.  给已有元素创建子元素，我们需要实例化`SubElement`类。它只要求两个参数，父元素（即该样例中的`new_feed`）和子元素的名字。由于该子元素会从父元素那儿继承名字空间的映射关系，所以这里不需要再声明名字空间前缀。
2.  我们也可以传递属性字典给它。字典的键即属性名；值为属性的值。
3.  如预期的那样，新创建的`title`元素在 Atom 名字空间中，并且它作为子元素插入到`feed`元素中。由于`title`元素没有文件内容，也没有其子元素，所以`lxml`将其序列化为一个空元素（使用`/&gt;`）。
4.  设定元素的文本内容，只需要设定其`.text`属性。
5.  当前`title`元素序列化的时候就使用了其文本内容。任何包含了`&lt;`或者`&`符号的内容在序列化的时候需要被转义。`lxml`会自动处理转义。
6.  我们也可以在序列化的时候应用“漂亮的输出(pretty printing)”，这会在每个结束标签的末尾，或者含有子元素但没有文本内容的标签的末尾添加换行符。用术语说就是，`lxml`添加“无意义的空白(insignificant whitespace)”以使输出更具可读性。

> ☞你也许也想要看一看[xmlwitch](http://github.com/galvez/xmlwitch/tree/master)，它也是用来生成 XML 的另外一个第三方库。它大量地使用了`with`语句来使生成的 XML 代码更具可读性。

## 解析破损的 XML

XML 规范文档中指出，要求所有遵循 XML 规范的解析器使用“严厉的(draconian)错误处理”。即，当它们在 XML 文档中检测到任何编排良好性(wellformedness)错误的时候，应当立即停止解析。编排良好性错误包括不匹配的起始和结束标签，未定义的实体(entity)，非法的 Unicode 字符，还有一些只有内行才懂的规则(esoteric rules)。这与其他的常见格式，比如 HTML，形成了鲜明的对比 — 即使忘记了封闭 HTML 标签，或者在属性值中忘了转义`&`字符，我们的浏览器也不会停止渲染一个 Web 页面。（通常大家认为 HTML 没有错误处理机制，这是一个常见的误解。[HTML 的错误处理](http://www.whatwg.org/specs/web-apps/current-work/multipage/syntax.html#parsing)实际上被很好的定义了，但是它比“遇见第一个错误即停止”这种机制要复杂得多。）

一些人（包括我自己）认为 XML 的设计者强制实行这种严格的错误处理本身是一个失误。请不要误解我；我当然能看到简化错误处理机制的优势。但是在现实中，“编排良好性”这种构想比乍听上去更加复杂，特别是对 XML（比如 Atom feeds）这种发布在网络上，通过 HTTP 传播的文档。早在 1997 年 XML 就标准化了这种严厉的错误处理，尽管 XML 已经非常成熟，研究一直表明，网络上相当一部分的 Atom feeds 仍然存在着编排完整性错误。

所以，从理论上和实际应用两种角度来看，我有理由“不惜任何代价”来解析 XML 文档，即，当遇到编排良好性错误时，*不会*中断解析操作。如果你认为你也需要这样做，`lxml`可以助你一臂之力。

以下是一个破损的 XML 文档的片断。其中的编排良好性错误已经被高亮标出来了。

```py
<?xml version='1.0' encoding='utf-8'?>
<feed xmlns='http://www.w3.org/2005/Atom' xml:lang='en'>
  <title>dive into <mark>&hellip;</mark></title>
...
</feed> 
```

因为实体`&hellip;`并没有在 XML 中被定义，所以这算作一个错误。（它在 HTML 中被定义。）如果我们尝试使用默认的设置来解析该破损的 feed，`lxml`会因为这个未定义的实体而停下来。

```py
>>> import lxml.etree
>>> tree = lxml.etree.parse('examples/feed-broken.xml')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "lxml.etree.pyx", line 2693, in lxml.etree.parse (src/lxml/lxml.etree.c:52591)
  File "parser.pxi", line 1478, in lxml.etree._parseDocument (src/lxml/lxml.etree.c:75665)
  File "parser.pxi", line 1507, in lxml.etree._parseDocumentFromURL (src/lxml/lxml.etree.c:75993)
  File "parser.pxi", line 1407, in lxml.etree._parseDocFromFile (src/lxml/lxml.etree.c:75002)
  File "parser.pxi", line 965, in lxml.etree._BaseParser._parseDocFromFile (src/lxml/lxml.etree.c:72023)
  File "parser.pxi", line 539, in lxml.etree._ParserContext._handleParseResultDoc (src/lxml/lxml.etree.c:67830)
  File "parser.pxi", line 625, in lxml.etree._handleParseResult (src/lxml/lxml.etree.c:68877)
  File "parser.pxi", line 565, in lxml.etree._raiseParseError (src/lxml/lxml.etree.c:68125)
lxml.etree.XMLSyntaxError: Entity 'hellip' not defined, line 3, column 28 
```

为了解析该破损的 XML 文档，忽略它的编排良好性错误，我们需要创建一个自定义的 XML 解析器。

```py
 examples/feed-broken.xml:3:28:FATAL:PARSER:ERR_UNDECLARED_ENTITY: Entity 'hellip' not defined
>>> tree.findall('{http://www.w3.org/2005/Atom}title')
[<Element {http://www.w3.org/2005/Atom}title at ead510>]
>>> title = tree.findall('{http://www.w3.org/2005/Atom}title')[0]

'dive into '

<feed xmlns='http://www.w3.org/2005/Atom' xml:lang='en'>
  <title>dive into </title>
.
. [rest of serialization snipped for brevity]
. 
```

1.  实例化`lxml.etree.XMLParser`类来创建一个自定义的解析器。它可以使用[许多不同的命名参数](http://codespeak.net/lxml/parsing.html#parser-options)。在此，我们感兴趣的为`recover`参数。当它的值被设为`True`，XML 解析器会尽力尝试从编排良好性错误中“恢复”。
2.  为使用自定的解析器来处理 XML 文档，将对象`parser`作为第二个参数传递给`parse()`函数。注意，`lxml`没有因为那个未定义的`&hellip;`实体而抛出异常。
3.  解析器会记录它所遇到的所有编排良好性错误。（无论它是否被设置为需要从错误中恢复，这个记录总会存在。）
4.  由于不知道如果处理该未定义的`&hellip;`实体，解析器默认会将其省略掉。`title`元素的文本内容变成了`'dive into '`。
5.  从序列化的结果可以看出，实体`&hellip;`并没有被移到其他地方去；它就是被省略了。

在此，必须反复强调，这种“可恢复的”XML 解析器没有**互用性(interoperability)保证**。另一个不同的解析器可能就会认为`&hellip;`来自 HTML，然后将其替换为`&amp;hellip;`。这样“更好”吗？也许吧。这样“更正确”吗？不，两种处理方法都不正确。正确的行为（根据 XML 规范）应该是终止解析操作。如果你已经决定不按规范来，你得自己负责。

## 进一步阅读

*   [维基百科上的词条　XML](http://en.wikipedia.org/wiki/XML)
*   [ElementTree 的 XML API](http://docs.python.org/3.1/library/xml.etree.elementtree.html)
*   [元素和树状元素](http://effbot.org/zone/element.htm)
*   [ElementTree 中对 XPath 的支持](http://effbot.org/zone/element-xpath.htm)
*   [ElementTree 的迭代式解析(iterparse)功能](http://effbot.org/zone/element-iterparse.htm)
*   [`lxml`](http://codespeak.net/lxml/)
*   [使用`lxml`解析 XML 和 HTML with](http://codespeak.net/lxml/1.3/parsing.html)
*   [使用`lxml`解析 XPath 和 XSLT](http://codespeak.net/lxml/1.3/xpathxslt.html)
*   [xmlwitch](http://github.com/galvez/xmlwitch/tree/master)