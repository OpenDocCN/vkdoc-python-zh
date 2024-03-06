# Chapter -1 《深入 Python 3》中有何新内容

# Chapter -1 《深入 Python 3》中有何新内容

> " 这不正是我们进来的地方吗？ " — 《迷墙》

## *又叫做* “the minus level”

你读过原版的 “[深入 Python](http://diveintopython.org/)” 并可能甚至买了纸版的。（谢谢！）你差不多已经了解 Python 2 了。你准备好了投入到 Python 3 里面。… 如果所有这些都成立，继续读。（如果没有一个是成立的，你最好从头开始。）

Python 3 提供了一个脚本叫做 `2to3`。学习它。喜欢它。使用它。用 `2to3` 移植代码到 Python 3 是一个有关 `2to3` 工具能够自动整理的所有东西的参考手册。很多这些东西都是语法的变更，因此了解 Python 3 里面许多的语法变更是一个好的起点。（`print` 现在是一个函数，`x` 不能使用，等等。）

案例分析：移植 `chardet` 到 Python 3 记录了我努力（最终成功）把一个不平常的库从 Python 2 移植到 Python 3 的过程。它也许能帮助你；也许不能。这里存在一个相当陡的学习曲线，由于你首先需要稍微理解一下这个库，那样你才可以理解为什么它会损坏以及我如何修复它的。围绕字符串有很多损坏的地方。说到这个…

字符串。吆。从哪儿开始呢。Python 2 有 “strings” 和 “Unicode strings”。Python 3 有 “bytes” 和 “strings”。也就是说，现在所有字符串都是 Unicode 的字符串，那么如果你想处理一个字节包，你可以使用新的 `bytes` 类型。Python 3 *从不会*在 strings 和 bytes 之间进行隐式的转换，因此在任何时候如果你不确信你拥有的是什么类型，你的代码几乎无疑的将会出问题。阅读 Strings 的章节 了解更多细节信息。

贯穿整个这本书，Bytes 和 strings 的对比会一次又一次的出现。

*   在文件这章，你将了解到通过“二进制”模式和“文本”模式读取文件的区别；在文本模式下读取（和写入！）文件需要提供一个 `encoding` 参数。一些文本文件方法按照字符来计数，而另一些方法按照字节计数。如果你的代码采取一个字符等于一个字节的方式，那么在多字节表示一个字符的情况下*将会*出问题。
*   在 HTTP Web 服务这章，`httplib2` 模块通过 HTTP 获取头信息和数据。HTTP 头信息返回的是字符串，而 HTTP 正文则返回的是字节。
*   在序列化 Python 对象这章，你将了解到为什么 Python 3 里面的 `pickle` 模块定义了一个和 Python 2 向后不兼容的新的数据类型。（提示：这就是因为字节和字符串的原因。） 同样 JSON 也根本不支持字节类型。我将向你展示如何解决这个问题。
*   在案例分析：移植 `chardet` 到 Python 3 这章，到处都是一大堆一大堆关于字节和字符串的东西。

即使你不关心 Unicode （但实际上你会的），你也会想阅读一下 Python 3 里面的字符串格式，这和 Python 2 里面的完全不一样。

迭代在 Python 3 里面无处不在，比起五年之前我写“深入 Python” 的时候，我现在能更好的理解它们。你也需要理解他们，因为过去经常在 Python 2 里面返回列表的很多函数，在 Python 3 里面将返回迭代。至少，你应该阅读一下迭代章节的下半部分和高级迭代章节的下半部分。

根据大家的要求，我已经添加了一个关于特殊方法名称的附录，有点像 [Python 文档的 “数据模型”章节](http://www.python.org/doc/3.1/reference/datamodel.html#special-method-names)但是包含更多的内容。

当我在撰写“深入 Python”的时候，所有可用的 XML 库都很糟糕。接着 Fredrik Lundh 编写了非常优秀的 [ElementTree](http://effbot.org/zone/element-index.htm)。Python 的专家们聪明的把 [ElementTree 变成了标准库的一部分](http://docs.python.org/3.1/library/xml.etree.elementtree.html)，然后现在它构成了我的新的 XML 章节的基础。解析 XML 的那些老的方式仍然可用，但是你应该避免使用它们，因为他们很糟糕！

除此之外，还有个关于 Python 的新东西 — 不是语言上的，而是社区中的 — 像 [Python 包装索引](http://pypi.python.org/)(PyPI)的出现。Python 提供了实用工具类用来将你的代码打包成标准格式，并分发那些包到 PyPI 中。阅读 打包 Python 库了解详细信息。