# Chapter 16 打包 Python 类库

# Chapter 16 打包 Python 类库

> " You’ll find the shame is like the pain; you only feel it once. " — Marquise de Merteuil, [Dangerous Liaisons](http://www.imdb.com/title/tt0094947/quotes)

## 深入

读到这里，你可能是想要发布一个 Python 脚本，库，框架，或者应用程序。太棒了！世界需要更多的 Python 代码。

Python 3 自带一个名为 Distutils 的打包框架。Distutils 包含许多功能：构建工具（为你所准备），安装工具（为用户所准备），数据包格式（为搜索引擎所准备）等。它集成了 [Python 安装包索引](http://pypi.python.org/)（“PyPI”），一个开源 Python 类库的中央资料库。

这些 Distutils 的不同功能以*setup script*为中心，一般被命名为 `setup.py`。事实上，你已经在本书中见过一些 Distutils 安装脚本。在 《HTTP Web Services》 一章中，我们使用 Distutils 来安装 `httplib2` ，而在《案例研究：将 `chardet` 移植到 Python 3》一章中，我们用它安装 `chardet` 。

在本章中，你将学习 `chardet` 和 `httplib2` 的安装脚本如何工作，并将逐步（学会）发布自己的 Python 软件。

```py
# chardet's setup.py
from distutils.core import setup
setup(
    name = "chardet",
    packages = ["chardet"],
    version = "1.0.2",
    description = "Universal encoding detector",
    author = "Mark Pilgrim",
    author_email = "mark@diveintomark.org",
    url = "http://chardet.feedparser.org/",
    download_url = "http://chardet.feedparser.org/download/python3-chardet-1.0.1.tgz",
    keywords = ["encoding", "i18n", "xml"],
    classifiers = [
        "Programming Language :: Python",
        "Programming Language :: Python :: 3",
        "Development Status :: 4 - Beta",
        "Environment :: Other Environment",
        "Intended Audience :: Developers",
        "License :: OSI Approved :: GNU Library or Lesser General Public License (LGPL)",
        "Operating System :: OS Independent",
        "Topic :: Software Development :: Libraries :: Python Modules",
        "Topic :: Text Processing :: Linguistic",
        ],
    long_description = """\
Universal character encoding detector
-------------------------------------

Detects
 - ASCII, UTF-8, UTF-16 (2 variants), UTF-32 (4 variants)
 - Big5, GB2312, EUC-TW, HZ-GB-2312, ISO-2022-CN (Traditional and Simplified Chinese)
 - EUC-JP, SHIFT_JIS, ISO-2022-JP (Japanese)
 - EUC-KR, ISO-2022-KR (Korean)
 - KOI8-R, MacCyrillic, IBM855, IBM866, ISO-8859-5, windows-1251 (Cyrillic)
 - ISO-8859-2, windows-1250 (Hungarian)
 - ISO-8859-5, windows-1251 (Bulgarian)
 - windows-1252 (English)
 - ISO-8859-7, windows-1253 (Greek)
 - ISO-8859-8, windows-1255 (Visual and Logical Hebrew)
 - TIS-620 (Thai)

This version requires Python 3 or later; a Python 2 version is available separately.
"""
) 
```

> ☞`chardet` 和 `httplib2` 都是开源的，但这并没有要求你在特定的许可下发布你自己的 Python 库。本章所描述的过程对任何 Python 软件都适用，无论它使用什么许可证

## Distutils 无法为你完成的工作

发布第一个 Python 包是一项艰巨的过程。（发布第二个相对容易一些。）Distutils 试图尽可能多的自动完成一些工作，但是仍然有一些事情你必须自己做。

*   **选择一种许可协议 。**. 这是一个复杂的话题，充满了派别斗争和危险。如果想将软件发布为开源软件，我冒昧地提出五点忠告：
    1.  不要撰写自己的许可证。
    2.  不要撰写自己的许可证。
    3.  不要撰写自己的许可证。
    4.  许可证并不一定必须是 GPL ，但它需要与 GPL 兼容 。
    5.  不要撰写自己的许可证。
*   使用 PyPI 分类系统**对软件进行分类**。我将在本章后面的部分解释这是什么意思。
*   **写“自述”(read me)文件 。**不要在这一点吝惜精力投入。至少，它应该让你的用户了解你的软件可以干什么并知道如何安装它。

## 目录结构

要开始打包 Python 软件，必须先将文件和目录安排好。 `httplib2` 的目录树如下：

```py
 |
   +--__init__.py
   |
   +--iri2uri.py 
```

1.  创建根目录来保存所有的目录和文件。将其以 Python 模块的名字命名。
2.  为了适应 Windows 用户，"自述"文件应包含 `.txt` 扩展名，而且它应该使用 Windows 风格回车符。不能仅仅因为*你*使用了一个优秀的文本编辑器，它从命令行运行并包括它自己的宏语言，而需要让你的用户为难。（你的用户使用记事本。虽然可悲，但却是事实。）即使你工作在 Linux 或 Mac OS X 环境下，优秀的文本编辑器毫无疑问地会有一个选项，允许将文件以 Windows 风格回车符来保存。
3.  Distutils 安装脚本应命名为 setup.py，除非你有一个很好的理由不这样做。但你并没有一个很好的理由不这样做。
4.  如果你的 Python 软件只包含一个单一的 `.py` 文件，你应该把它和"自述"文件以及安装脚本放到根目录下。但 `httplib2` 并不是单一的 `.py` 文件，它是一个多文件模块 。但是没关系！只需在根目录下放置 `httplib2` 目录，这样在 `httplib2/` 根目录下就会有一个包含 `__init__.py` 文件的 `httplib2/` 目录。这并不是一个难题，事实上，它可以简化打包过程。

`chardet` 目录看起来有些不同。像 `httplib2` 一样，它是一个多文件模块 ，所以在 `chardet/` 根目录下有一个 `chardet/` 目录。除了 `README.txt` 文件，在 `docs/` 目录下， `chardet` 还有 HTML ——格式化文档。该 `docs/` 目录包含多个 `.html` 和`.css` 文件和 `images/` 子目录，其中包含几个 `.png` 和 `.gif` 文件。（稍后你会发现，这将是很重要的。）此外，对于 (L)GPL 许可的软件，它包含一个单独的 `COPYING.txt` 文件，其中包含 LGPL 许可证的完整内容。

```py
chardet/
|
+--COPYING.txt
|
+--setup.py
|
+--README.txt
|
+--docs/
|  |
|  +--index.html
|  |
|  +--usage.html
|  |
|  +--images/ ...
|
+--chardet/
   |
   +--__init__.py
   |
   +--big5freq.py
   |
   +--... 
```

## 编写安装脚本

Distutils 安装脚本是一份 Python 脚本。从理论上讲，它可以做任何 Python 可以做的事情。在实践中，安装脚本应该做尽可能少的事情并尽可能按标准的方式做。安装脚本应该简单。安装过程越奇异，错误报告也会更奇特。

每个 Distutils 安装脚本的第一行总是相同的：

```py
from distutils.core import setup 
```

该行导入 `setup()` 函数，这是 Distutils 的主入口点。95％ 的 Distutils 安装脚本仅由一个对 `setup()` 方法的调用组成。（这完全是我臆造的统计，但如果你的 Distutils 安装脚本所做的比仅仅调用 `setup()` 方法更多，你会有一个好的理由。你有一个好的理由吗？我并不这么认为。）

`setup()` 方法[可以有几十个参数](http://docs.python.org/3.1/distutils/apiref.html#distutils.core.setup) 。为了使每个参与者都能清楚，你必须对每个参数使用命名变量 。这不只是一项约定，还是一项硬性要求。如果尝试以非命名变量调用 `setup()` 方法，安装脚本会崩溃。

下面的命名变量是必需的：

*   **name**，安装包的名称。
*   **version**，安装包的版本。
*   **author**，您的全名。
*   **author_email**，您的邮件地址。
*   **url**，项目主页。如果没有一个单独的项目网站，这里可以是安装包的 [PyPI](http://pypi.python.org/) 的页面地址。

虽然以下内容不是必须的，但我也建议你把他们包括在你的安装脚本里：

*   **description**，在线的项目摘要。
*   **long_description**，以 [reStructuredText format](http://docutils.sourceforge.net/rst.html) 格式编写的多行字符串。[PyPI](http://pypi.python.org/) 将其转换为 HTML 并在安装包中显示它。
*   **classifiers**，下一节中将讲述的特别格式化字符串。

> ☞安装脚本中用到的元数据具体定义在 [PEP 314](http://www.python.org/dev/peps/pep-0314/) 中。

现在让我们看看 `chardet` 的安装脚本。它包含所有这些要求的和建议的参数，还有一个我没有提到： `packages` 。

```py
from distutils.core import setup
setup(
    name = 'chardet',
    <mark>packages = ['chardet']</mark>,
    version = '1.0.2',
    description = 'Universal encoding detector',
    author='Mark Pilgrim',
    ...
) 
```

在分发过程中，这个 `packages` 参数凸显出一个不幸的词汇表重叠。我们一直在谈论正在构建的“安装包”（并将潜在地出现在 Python 包索引中）。但是，这并不是 `packages` 参数所指代的。它指代的是 `chardet` 模块是一个多文件模块这一事实 ，有时也被称为...“包”。`packages` 参数告诉 Distutils 去包含`chardet/` 目录，它的 `__init__.py` 文件，以及所有其他构成 `chardet` 模块的 `.py` 文件。这还算比较重要；如果你忘记了包含实际的代码，那么所有这些关于文件和元数据的愉快交谈都将是无关紧要的。

## 将包分类

Python 包索引（“PyPI”）包含成千上万的 Python 库。正确的分类数据将让人们更容易找到你的包。PyPI 让你[以类别的形式浏览包](http://pypi.python.org/pypi?:action=browse) 。你甚至可以选择多个类别来缩小搜索范围。分类不是你可以忽略的不可见的元数据！

你可以通过传递 `classifiers` 参数给 Distutils 的 `setup()` 方法来给你的软件分类。`classifers` 参数是一个字符串列表。这些字符串*不是*任意形式的。所有的分类字符串应该来自 [PyPI 上的列表](http://pypi.python.org/pypi?:action=list_classifiers) 。

分类是可选的。你可以写一个不包含任何分类的 Distutils 安装脚本。**不要这样做。** 你应该*总是*至少包括以下分类：

*   <b0 编程语言. 特别的，你应该包括`"Programming Language :: Python"`和"`Programming Language :: Python :: 3`"。如果你不包括这些，你的包将不会出现在[兼容 Python 3 的库](http://pypi.python.org/pypi?:action=browse&c=533&show=all)列表中，它链接自每个`pypi.python.org`单页的侧边拦。
*   **许可证**. 当我评价一个第三方库的时候，这*绝对是我寻找的第一个*东西。不要让我（花太多时间）寻找这个重要的信息。不要包含一个以上的许可证分类，除非你的软件明确地在多许可证下分发。（不要在多许可证下发布你的软件，除非你不得不这样做。不要强迫别人这样做。许可证已经足够让人头痛了，不要使情况变得更糟。）
*   **操作系统 0**. 如果你的软件只能运行于 Windows（或 Mac OS X 或 Linux），我想要尽早知道。如果你的软件不包含任何特定平台的代码并可以在任何平台运行，请使用分类 `"Operating System :: OS Independent"`。`多操作系统` 分类仅在你的软件在不同平台需要特别支持时使用。（这并不常见。）

我还建议你包括以下分类：

*   **开发状态**. 你的软件品质适合 beta 发布么？适合 Alpha 发布么？还是 Pre-alpha？在这里面选择一个吧。要诚实点。
*   **目标用户**. 谁会下载你的软件？最常见的选项包括： `Developers`、 `End Users/Desktop`、 `Science/Research` 和 `System Administrators`。
*   **框架**. 如果你的软件是像 [Django](http://www.djangoproject.com/) 或 [Zope](http://www.zope.org/) 这样较大的框架的插件，请包含适当的 `Framework` 分类。如果不是，请忽略它。
*   **主题**. 有 [大量的主题](http://pypi.python.org/pypi?:action=list_classifiers) 可供选择 ，选择所有的适用项。

### 包分类的优秀范例

作为例子，下面是 [Django](http://pypi.python.org/pypi/Django/) 的分类。它是一个运行在 Web 服务器上的，可用于生产环境的，跨平台的，使用 BSD 授权的 Web 应用程序框架。（Django 还没有与 Python 3 兼容，因此， 并没有列出 `Programming Language :: Python :: 3` 分类。）

```py
Programming Language :: Python
License :: OSI Approved :: BSD License
Operating System :: OS Independent
Development Status :: 5 - Production/Stable
Environment :: Web Environment
Framework :: Django
Intended Audience :: Developers
Topic :: Internet :: WWW/HTTP
Topic :: Internet :: WWW/HTTP :: Dynamic Content
Topic :: Internet :: WWW/HTTP :: WSGI
Topic :: Software Development :: Libraries :: Python Modules 
```

下面是 [`chardet`](http://pypi.python.org/pypi/chardet) 的分类。它就是在《案例研究：将 `chardet` 移植到 Python 3》一章提到的字符编码检测库。`chardet` 是高质量的，跨平台的，与 Python 3 兼容的， LGPL 许可的库。它旨在让开发者将其集成进自己的产品。

```py
Programming Language :: Python
Programming Language :: Python :: 3
License :: OSI Approved :: GNU Library or Lesser General Public License (LGPL)
Operating System :: OS Independent
Development Status :: 4 - Beta
Environment :: Other Environment
Intended Audience :: Developers
Topic :: Text Processing :: Linguistic
Topic :: Software Development :: Libraries :: Python Modules 
```

以下是在本章开头我提到的 [`httplib2`](http://pypi.python.org/pypi/httplib2) 模块——HTTP 的分类。`httplib2` 是一个测试品质的，跨平台的，MIT 许可证授权的，为 Python 开发者准备的模块。

```py
Programming Language :: Python
Programming Language :: Python :: 3
License :: OSI Approved :: MIT License
Operating System :: OS Independent
Development Status :: 4 - Beta
Environment :: Web Environment
Intended Audience :: Developers
Topic :: Internet :: WWW/HTTP
Topic :: Software Development :: Libraries :: Python Modules 
```

## 通过清单指定附加文件

默认情况下，Distutils 将把下列文件包含在你的发布包中：

*   `README.txt`
*   `setup.py`
*   由列在 `packages` 参数中的多模块文件所需的 `.py` 文件
*   在 `py_modules` 参数中列出的单独 `.py` 文件

这将覆盖`httplib2` 项目的所有文件。但对于 `chardet` 项目，我们还希望包含 `COPYING.txt` 许可文件和含有图像与 HTML 文件的整个 `docs/` 目录。要让 Distutils 在构建 `chardet` 发布包时包含这些额外的文件和目录，你需要创建一个 *manifest file* 。

清单文件是一个名为 `MANIFEST.in` 的文本文件。将它放置在项目的根目录下，同 `README.txt` 和 `setup.py` 一起。清单文件并*不是* Python 脚本，它是文本文件，其中包含一系列 Distutils 定义格式的命令。清单命令允许你包含或排除特定的文件和目录。

以下是 `chardet` 项目的全部清单文件：

1.  第一行是不言自明的：包含项目根目录的 `COPYING.txt` 文件。
2.  第二行有些复杂。`recursive-include` 命令需要一个目录名和至少一个文件名。文件名并不限于特定的文件，可以包含通配符。这行的意思是“看到在项目根目录下的 `docs/` 目录了吗？在该目录下（递归地）查找 `.html`、 `.css`、 `.png` 和 `.gif` 文件。我希望将他们都包含在我的发布包中。”

所有的清单命令都将保持你在项目目录中所设置的目录结构。`recursive-include` 命令不会将一组 `.html` 和 `.png` 文件放置在你的发布包的根目录下。它将保持现有的 `docs/` 目录结构，但只包含该目录内匹配给定的通配符的文件。（之前我并没有提到， `chardet` 的文档实际上由 XML 语言写成，并由一个单独的脚本转换为 HTML 。我不想在发布包中包含 XML 文件，只包含 HTML 文件和图像。)

> ☞清单文件有自己独特的格式。详见 [分发指定文件](http://docs.python.org/3.1/distutils/sourcedist.html#manifest) 和[清单文件命令](http://docs.python.org/3.1/distutils/commandref.html#sdist-cmd)。

重申：仅仅在你需要包含一些 Distutils 不会默认包含的文件时才创建清单文件。I 如果你确实需要一个清单文件，它应该只包含那些 Distutils 不会自动包含的文件和目录。

## 检查安装脚本的错误

有许多事情需要留意。Distutils 带有一个内置的验证命令，它检查是否所有必须的元数据都体现在你的安装脚本中。例如，如果你忘记包含 `version` 参数，Distutils 会提醒你。

```py
c:\Users\pilgrim\chardet> c:\python31\python.exe setup.py check
running check
warning: check: missing required meta-data: version 
```

当你包含了 `version` 参数（和所有其他所需的元数据）时， `check` 命令将如下所示：

```py
c:\Users\pilgrim\chardet> c:\python31\python.exe setup.py check
running check 
```

## 创建发布源

Distutils 支持构建多种类型的发布包。至少，你应该建立一个“源代码分发”，其中包含源代码，你的 Distutils 安装脚本，“read me ”文件和你想要包含其他文件 。为了建立一个源代码分发，传递 `sdist` 命令给你的 Distutils 安装脚本。

```py
c:\Users\pilgrim\chardet> <mark>c:\python31\python.exe setup.py sdist</mark>
running sdist
running check
reading manifest template 'MANIFEST.in'
writing manifest file 'MANIFEST'
creating chardet-1.0.2
creating chardet-1.0.2\chardet
creating chardet-1.0.2\docs
creating chardet-1.0.2\docs\images
copying files to chardet-1.0.2...
copying COPYING -> chardet-1.0.2
copying README.txt -> chardet-1.0.2
copying setup.py -> chardet-1.0.2
copying chardet\__init__.py -> chardet-1.0.2\chardet
copying chardet\big5freq.py -> chardet-1.0.2\chardet
...
copying chardet\universaldetector.py -> chardet-1.0.2\chardet
copying chardet\utf8prober.py -> chardet-1.0.2\chardet
copying docs\faq.html -> chardet-1.0.2\docs
copying docs\history.html -> chardet-1.0.2\docs
copying docs\how-it-works.html -> chardet-1.0.2\docs
copying docs\index.html -> chardet-1.0.2\docs
copying docs\license.html -> chardet-1.0.2\docs
copying docs\supported-encodings.html -> chardet-1.0.2\docs
copying docs\usage.html -> chardet-1.0.2\docs
copying docs\images\caution.png -> chardet-1.0.2\docs\images
copying docs\images\important.png -> chardet-1.0.2\docs\images
copying docs\images\note.png -> chardet-1.0.2\docs\images
copying docs\images\permalink.gif -> chardet-1.0.2\docs\images
copying docs\images\tip.png -> chardet-1.0.2\docs\images
copying docs\images\warning.png -> chardet-1.0.2\docs\images
creating dist
creating 'dist\chardet-1.0.2.zip' and adding 'chardet-1.0.2' to it
adding 'chardet-1.0.2\COPYING'
adding 'chardet-1.0.2\PKG-INFO'
adding 'chardet-1.0.2\README.txt'
adding 'chardet-1.0.2\setup.py'
adding 'chardet-1.0.2\chardet\big5freq.py'
adding 'chardet-1.0.2\chardet\big5prober.py'
...
adding 'chardet-1.0.2\chardet\universaldetector.py'
adding 'chardet-1.0.2\chardet\utf8prober.py'
adding 'chardet-1.0.2\chardet\__init__.py'
adding 'chardet-1.0.2\docs\faq.html'
adding 'chardet-1.0.2\docs\history.html'
adding 'chardet-1.0.2\docs\how-it-works.html'
adding 'chardet-1.0.2\docs\index.html'
adding 'chardet-1.0.2\docs\license.html'
adding 'chardet-1.0.2\docs\supported-encodings.html'
adding 'chardet-1.0.2\docs\usage.html'
adding 'chardet-1.0.2\docs\images\caution.png'
adding 'chardet-1.0.2\docs\images\important.png'
adding 'chardet-1.0.2\docs\images\note.png'
adding 'chardet-1.0.2\docs\images\permalink.gif'
adding 'chardet-1.0.2\docs\images\tip.png'
adding 'chardet-1.0.2\docs\images\warning.png'
removing 'chardet-1.0.2' (and everything under it) 
```

有几件事情需要注意：

*   Distutils 发现了清单文件( `MANIFEST.in` )
*   Distutils 成功地解析了清单文件，并添加了我们所需要的文件—— `COPYING.txt` 和在 `docs/` 目录下的 HTML 与图像文件。
*   如果你进入你的项目目录，你会看到 Distutils 创建了一个 `dist/` 目录。你可以分发在 `dist/` 目录中的 `.zip` 文件。

```py
c:\Users\pilgrim\chardet> <mark>dir dist</mark>
 Volume in drive C has no label.
 Volume Serial Number is DED5-B4F8

 Directory of c:\Users\pilgrim\chardet\dist

07/30/2009  06:29 PM    <DIR>          .
07/30/2009  06:29 PM    <DIR>          ..
07/30/2009  06:29 PM           206,440 <mark>chardet-1.0.2.zip</mark>
               1 File(s)        206,440 bytes
               2 Dir(s)  61,424,635,904 bytes free 
```

## 创建图形化安装程序

在我看来，每一个 Python 库都应该为 Windows 用户提供图形安装程序。这很容易做（即使你并没有运行 Windows ），而且 Windows 用户会对此表示感激。

通过传递 `bdist_wininst` 命令到你的 Distutils 安装脚本，它可以[为你创建一个图形化的 Windows 安装程序](http://docs.python.org/3.1/distutils/builtdist.html#creating-windows-installers) 。

```py
c:\Users\pilgrim\chardet> <mark>c:\python31\python.exe setup.py bdist_wininst</mark>
running bdist_wininst
running build
running build_py
creating build
creating build\lib
creating build\lib\chardet
copying chardet\big5freq.py -> build\lib\chardet
copying chardet\big5prober.py -> build\lib\chardet
...
copying chardet\universaldetector.py -> build\lib\chardet
copying chardet\utf8prober.py -> build\lib\chardet
copying chardet\__init__.py -> build\lib\chardet
installing to build\bdist.win32\wininst
running install_lib
creating build\bdist.win32
creating build\bdist.win32\wininst
creating build\bdist.win32\wininst\PURELIB
creating build\bdist.win32\wininst\PURELIB\chardet
copying build\lib\chardet\big5freq.py -> build\bdist.win32\wininst\PURELIB\chardet
copying build\lib\chardet\big5prober.py -> build\bdist.win32\wininst\PURELIB\chardet
...
copying build\lib\chardet\universaldetector.py -> build\bdist.win32\wininst\PURELIB\chardet
copying build\lib\chardet\utf8prober.py -> build\bdist.win32\wininst\PURELIB\chardet
copying build\lib\chardet\__init__.py -> build\bdist.win32\wininst\PURELIB\chardet
running install_egg_info
Writing build\bdist.win32\wininst\PURELIB\chardet-1.0.2-py3.1.egg-info
creating 'c:\users\pilgrim\appdata\local\temp\tmp2f4h7e.zip' and adding '.' to it
adding 'PURELIB\chardet-1.0.2-py3.1.egg-info'
adding 'PURELIB\chardet\big5freq.py'
adding 'PURELIB\chardet\big5prober.py'
...
adding 'PURELIB\chardet\universaldetector.py'
adding 'PURELIB\chardet\utf8prober.py'
adding 'PURELIB\chardet\__init__.py'
removing 'build\bdist.win32\wininst' (and everything under it)
c:\Users\pilgrim\chardet> <mark>dir dist</mark>
c:\Users\pilgrim\chardet>dir dist
 Volume in drive C has no label.
 Volume Serial Number is AADE-E29F

 Directory of c:\Users\pilgrim\chardet\dist

07/30/2009  10:14 PM    <DIR>          .
07/30/2009  10:14 PM    <DIR>          ..
07/30/2009  10:14 PM           371,236 <mark>chardet-1.0.2.win32.exe</mark>
07/30/2009  06:29 PM           206,440 chardet-1.0.2.zip
               2 File(s)        577,676 bytes
               2 Dir(s)  61,424,070,656 bytes free 
```

### 为其它操作系统编译安装包

Distutils 可以帮助你[为 Linux 用户构建可安装包](http://docs.python.org/3.1/distutils/builtdist.html#creating-rpm-packages) 。我认为，这可能不值得你浪费时间。如果你希望在 Linux 中分发你的软件，你最好将时间花在与那些社区成员进行交流上，他们专门为主流 Linux 发行版打包软件。

例如，我的 `chardet` 库包含在 Debian GNU/Linux 软件仓库中（因而也包含[在 Ubuntu 的软件仓库](http://packages.ubuntu.com/python-chardet)中）。我不曾做任何事情，我只在那里将安装包展示了一天。Debian 社区拥有[他们自己的关于打包 Python 库的政策](http://www.debian.org/doc/packaging-manuals/python-policy/)，并且 Debian 的 `python-chardet` 包被设计为遵循这些公约。由于这个包存在在 Debian 的软件仓库中，依赖于 Debian 用户所选择的管理自己计算机的系统设置，他们会收到该包的安全更新和（或）新版本。

Distutils 构建的包不具有 Linux 包所提供的任何优势。你的时间最好花在其他地方。

## 将软件添加到 Python 安装包列表

上传软件到 Python 包索引需要三个步骤。

1.  注册你自己
2.  注册你的软件
3.  上传你通过 `setup.py sdist` 和 `setup.py bdist_*` 创建的包。

要注册自己，访问 [PyPI 用户注册页面](http://pypi.python.org/pypi?:action=register_form)。输入你想要的用户名和密码，提供一个有效的电子邮件地址，然后点击 `Register` 按钮。（如果你有一个 PGP 或 GPG 密钥，你也可以提供。如果你没有或者不知道这是什么意思，不用担心。）检查你的电子邮件，在几分钟之内，你应该会收到一封来自 PyPI 的包含验证链接的邮件。点击链接以完成注册过程。

现在，你需要在 PyPI 注册你的软件并上传它。你可以用一步完成。

```py
 running register
We need to know who you are, so please choose either:
 1\. use your existing login,
 2\. register as a new user,
 3\. have the server generate a new password for you (and email it to you), or
 4\. quit

Password:

Server response (200): OK

... output trimmed for brevity ...

... output trimmed for brevity ...

Submitting dist\chardet-1.0.2.zip to http://pypi.python.org/pypi
Server response (200): OK
Submitting dist\chardet-1.0.2.win32.exe to http://pypi.python.org/pypi
Server response (200): OK
I can store your PyPI login so future submissions will be faster.
(the login will be stored in c:\home\.pypirc) 
```

1.  当你第一次发布你的项目时，Distutils 会将你的软件加入到 Python 包索引中并给出它的 URL。在这之后，它只会用你在 `setup.py` 参数所做的任何改变来更新项目的元数据。之后，它构建一个源代码发布 (`sdist`) 和一个 Windows 安装程序 (`bdist_wininst`) 并把他们上传到 PyPI (`upload`)。
2.  键入 `1` 或 `ENTER` 选择“ 使用已有的账户登录【use your existing login.】”。
3.  输入你在 [PyPI 用户注册页面](http://pypi.python.org/pypi?:action=register_form)所选择的用户名和密码。Distuils 不会回显你的密码，它甚至不会在相应的位置显示星号。只需输入你的密码，然后按 `回车键` 。
4.  Distutils 在 Python 包索引注册你的包……
5.  ……构建源代码分发……
6.  ……构建 Windows 安装程序……
7.  ……并把它们上传至 Python 包索引。
8.  如果你想自动完成发布新版本的过程，你需要将你的 PyPI 凭据保存在一个本地文件中。这完全是不安全的而且是完全可选的。

恭喜你，现在，在 Python 包索引中有你自己的页面了！地址是 `http://pypi.python.org/pypi/_NAME_`，其中 *NAME* 是你在 `setup.py` 文件中 `name` 参数所传递的字符串。

如果你想发布一个新版本，只需以新的版本号更新 `setup.py` 文件，然后再一次运行相同的上传命令:

```py
c:\Users\pilgrim\chardet> c:\python31\python.exe setup.py register sdist bdist_wininst upload 
```

## Python 打包工具的一些可能的将来

Distutils 并非是一个代替所有并终结所有的 Python 打包，但在写本书时（2009 年 8 月），它是唯一可以工作在 Python 3 下的打包框架。对于 Python 2，还有许多其他的框架，有的重在安装，有的重在测试，还有的重在部署。在未来，它们中的一部分或全体都将移植到 Python 3。

以下框架重在安装：

*   [Setuptools](http://pypi.python.org/pypi/setuptools)
*   [Pip](http://pypi.python.org/pypi/pip)
*   [Distribute](http://bitbucket.org/tarek/distribute/)

以下框架重在测试和部署：

*   [`virtualenv`](http://pypi.python.org/pypi/virtualenv)
*   [`zc.buildout`](http://pypi.python.org/pypi/zc.buildout)
*   [Paver](http://www.blueskyonmars.com/projects/paver/)
*   [Fabric](http://fabfile.org/)
*   [`py2exe`](http://www.py2exe.org/)

## 深入阅读

关于 Distutils：

*   [通过 Distutils 发布 Python 模块](http://docs.python.org/3.1/distutils/)
*   [核心发布功能](http://docs.python.org/3.1/distutils/apiref.html#module-distutils.core) 列出了 `setup()` 函数的所有可能参数
*   [Distutils 食谱](http://wiki.python.org/moin/Distutils/Cookbook)
*   [PEP 370: 每用户 `site-packages` 目录](http://www.python.org/dev/peps/pep-0370/)
*   [PEP 370 和 “environment stew”](http://jessenoller.com/2009/07/19/pep-370-per-user-site-packages-and-environment-stew/)

其它打包框架：

*   [Python 打包生态系统](http://groups.google.com/group/django-developers/msg/5407cdb400157259)
*   [关于打包](http://www.b-list.org/weblog/2008/dec/14/packaging/)
*   [对 “关于打包” 的几点纠错](http://blog.ianbicking.org/2008/12/14/a-few-corrections-to-on-packaging/)
*   [我为什么喜欢 Pip](http://www.b-list.org/weblog/2008/dec/15/pip/)
*   [Python 打包：几点看法](http://cournape.wordpress.com/2009/04/01/python-packaging-a-few-observations-cabal-for-a-solution/)
*   [没有人期望 Python 打包！](http://jacobian.org/writing/nobody-expects-python-packaging/)

你的位置: Home ‣ Dive Into Python 3 ‣

难度等级: ♦♦♦♦♦