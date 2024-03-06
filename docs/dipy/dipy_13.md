# 第十二章 SOAP Web 服务

# 第十二章 SOAP Web 服务

*   12.1\. 概览
*   12.2\. 安装 SOAP 库
    *   12.2.1\. 安装 PyXML
    *   12.2.2\. 安装 fpconst
    *   12.2.3\. 安装 SOAPpy
*   12.3\. 步入 SOAP
*   12.4\. SOAP 网络服务查错
*   12.5\. WSDL 介绍
*   12.6\. 以 WSDL 进行 SOAP 内省
*   12.7\. 搜索 Google
*   12.8\. SOAP 网络服务故障排除
*   12.9\. 小结

第十一章 关注 HTTP 上面向文档的 web 服务。“输入参数” 是 URL，“返回值” 是需要你来解析的一个实际的 XML 文档。

本章将关注更加结构化的 SOAP web 服务。SOAP 不需要你直接与 HTTP 请求和 XML 文档打交道，而是允许你模拟返回原始数据类型的函数调用。正像你将要看到的，这个描述恰如其份；你可以使用标准 Python 调用语法通过 SOAP 库去调用一个函数，这个函数也自然会返回 Python 对象和值。但揭开这层面纱，SOAP 库实际上执行了一个多个 XML 文档和远程服务器参与的复杂处理过程。

SOAP 的贴切定义很复杂，不要误认为 SOAP 就是用于调用远程函数。有些人觉得应该补充上：SOAP 还允许单向异步的信息通过，以及面向文档的 Web 服务。有这样想法的人是正确的，SOAP 的确是这样，但却不止于此。但这一章的重点在于所谓的 “RPC-style” SOAP――调用远程函数获得返回结果。

# 12.1\. 概览

# 12.1\. 概览

你用 Google，对吧？它是一个很流行的搜索引擎。你是否希望能以程序化的方式访问 Google 的搜索结果呢？现在你能做到了。下面是一个用 Python 搜索 Google 的程序。

## 例 12.1\. `search.py`

```py
from SOAPpy import WSDL
# you'll need to configure these two values;
# see http://www.google.com/apis/
WSDLFILE = '/path/to/copy/of/GoogleSearch.wsdl'
APIKEY = 'YOUR_GOOGLE_API_KEY'
_server = WSDL.Proxy(WSDLFILE)
def search(q):
    """Search Google and return list of {title, link, description}"""
    results = _server.doGoogleSearch(
        APIKEY, q, 0, 10, False, "", False, "", "utf-8", "utf-8")
    return [{"title": r.title.encode("utf-8"),
             "link": r.URL.encode("utf-8"),
             "description": r.snippet.encode("utf-8")}
            for r in results.resultElements]
if __name__ == '__main__':
    import sys
    for r in search(sys.argv[1])[:5]:
        print r['title']
        print r['link']
        print r['description']
        print 
```

你可以在较大的程序中以模块导入并使用它，也可以在命令行上运行这个脚本。在命令行上，需要把查询字符串作为命令行参数使用，之后就会打印出最前面的五个 Google 查询结果，包括：URL、标题和描述信息。

下面是以 “python” 作为命令行参数的查询结果。

## 例 12.2\. `search.py` 的使用样例

```py
C:\diveintopython\common\py> python search.py "python"
<b>Python</b> Programming Language
http://www.python.org/
Home page for <b>Python</b>, an interpreted, interactive, object-oriented,
extensible<br> programming language. <b>...</b> <b>Python</b>
is OSI Certified Open Source: OSI Certified.
<b>Python</b> Documentation Index
http://www.python.org/doc/
 <b>...</b> New-style classes (aka descrintro). Regular expressions. Database
API. Email Us.<br> docs@<b>python</b>.org. (c) 2004\. <b>Python</b>
Software Foundation. <b>Python</b> Documentation. <b>...</b>
Download <b>Python</b> Software
http://www.python.org/download/
Download Standard <b>Python</b> Software. <b>Python</b> 2.3.3 is the
current production<br> version of <b>Python</b>. <b>...</b>
<b>Python</b> is OSI Certified Open Source:
Pythonline
http://www.pythonline.com/
Dive Into <b>Python</b>
http://diveintopython.org/
Dive Into <b>Python</b>. <b>Python</b> from novice to pro. Find:
<b>...</b> It is also available in multiple<br> languages. Read
Dive Into <b>Python</b>. This book is still being written. <b>...</b> 
```

## 进一步阅读

*   [`www.xmethods.net/`](http://www.xmethods.net/) 是一个访问 SOAP web 服务的公共知识库。
*   SOAP 规范相当可读，如果你喜欢这类东西的话。

# 12.2\. 安装 SOAP 库

# 12.2\. 安装 SOAP 库

*   12.2.1\. 安装 PyXML
*   12.2.2\. 安装 fpconst
*   12.2.3\. 安装 SOAPpy

与本书中的其他代码不同，本章依赖的库不是 Python 预安装的。

在深入学习 SOAP web 服务之前，你需要安装三个库：PyXML、fpconst 和 SOAPpy。

## 12.2.1\. 安装 PyXML

你要用到的第一个库是 PyXML，它是 XML 库的一个高级组件，提供了比我们在 第九章 学习的 XML 内建库更多的功能。

## 过程 12.1.

下面是安装 PyXML 的步骤：

1.  访问 [`pyxml.sourceforge.net/`](http://pyxml.sourceforge.net/)，点击 Downloads，下载适合你所使用操作系统的最新版本。

2.  如果你所使用的是 Windows，那么你有多个选择。一定要确保你所下载的 PyXML 和你所使用的 Python 版本匹配。

3.  双击安装程序。如果你下载的是为 Windows 提供的 PyXML 0.8.3，并且你所使用的是 Python 2.3，这个安装程序应该是 `PyXML-0.8.3.win32-py2.3.exe`。

4.  深入安装过程。

5.  安装完成后，关闭安装程序，没有任何安装成功的昭示 (并没有在开始菜单、快捷栏或桌面出现图标)。因为 PyXML 仅仅是被其他程序调用的 XML 的库集合。

要检验 PyXML 安装得是否正确，可以运行 Python IDE，下面的指令可以看到 XML 库的安装版本。

## 例 12.3\. 检验 PyXML 安装

```py
>>> import xml
>>> xml.__version__
'0.8.3' 
```

这个安装版本号应该和你所下载并安装的 PyXML 安装程序版本号一致。

## 12.2.2\. 安装 fpconst

你所需要安装的第二个库是 fpconst，它是一系列支持 IEEE754 double-precision 特殊值的常量和函数，提供了对 Not-a-Number (NaN), Positive Infinity (Inf) 和 Negative Infinity (-Inf) 等特殊值的支持，而这是 SOAP 数据类型规范的组成部分。

## 过程 12.2.

下面是 fpconst 的安装过程：

1.  从 [`www.analytics.washington.edu/statcomp/projects/rzope/fpconst/`](http://www.analytics.washington.edu/statcomp/projects/rzope/fpconst/) 下载 fpconst 的最新版本。

2.  提供了两种格式的下载：`.tar.gz` 和 `.zip`。如果你使用的是 Windows 操作系统，下载 `.zip` 文件；其他情况下应该下载 `.tar.gz` 文件。

3.  对这个文件进行解压缩。在 Windows XP 上你可以鼠标右键单击这个文件并选择“解压文件”；在较早的 Windows 版本上则需要 WinZip 之类的第三方解压程序。在 Mac OS X 上，可以右键单击压缩文件进行解压。

4.  打开命令提示符窗口并定位到解压目录。

5.  键入 **`python setup.py install`** 运行安装程序。

要检验 fpconst 安装得是否正确，运行 Python IDE 并查看版本号。

## 例 12.4\. 检验 fpconst 安装

```py
>>> import fpconst
>>> fpconst.__version__
'0.6.0' 
```

这个安装版本号应该和你所下载并用于安装的 fpconst 压缩包版本号一致。

## 12.2.3\. 安装 SOAPpy

第三个，也是最后一个需要安装的库是 SOAP 库本身：SOAPpy。

## 过程 12.3.

下面是安装 SOAPpy 的过程：

1.  访问 [`pywebsvcs.sourceforge.net/`](http://pywebsvcs.sourceforge.net/) 并选择 SOAPpy 部分中最新的官方发布。

2.  提供了两种格式的下载。如果你使用的是 Windows，那么下载 `.zip` 文件；其他情况则下载 `.tar.gz` 文件。

3.  和安装 fpconst 时一样先解压下载的文件．

4.  打开命令提示符窗口并定位到解压 SOAPpy 文件的目录。

5.  键入 **`python setup.py install`** 运行安装程序。

要检验 SOAPpy 安装得是否正确，运行 Python IDE 并查看版本号。

## 例 12.5\. 检验 SOAPpy 安装

```py
>>> import SOAPpy
>>> SOAPpy.__version__
'0.11.4' 
```

这个安装版本号应该和你所下载并用于安装的 SOAPpy 压缩包版本号一致。

# 12.3\. 步入 SOAP

# 12.3\. 步入 SOAP

调用远程函数是 SOAP 的核心功能。有很多提供公开 SOAP 访问的服务器提供用于展示的简单功能。

最受欢迎的 SOAP 公开访问服务器是 [`www.xmethods.net/`](http://www.xmethods.net/)。这个例子使用了一个展示函数，可以根据美国邮政编码返回当地气温。

## 例 12.6\. 获得现在的气温

```py
>>> from SOAPpy import SOAPProxy            
>>> url = 'http://services.xmethods.net:80/soap/servlet/rpcrouter'
>>> namespace = 'urn:xmethods-Temperature'  
>>> server = SOAPProxy(url, namespace)      
>>> server.getTemp('27502')                 
80.0 
```

|  |  |
| --- | --- |
| [1] | 你通过 `SOAPProxy` 这个代理 (proxy) 类访问远程 SOAP 服务器。这个代理处理了所有的 SOAP 内部事务，其中包括：根据函数名和参数列表创建 XML 请求文档，并将这个请求文档通过 HTTP 发送到远程 SOAP 服务器；解析 XML 返回文档，并创建本地的 Python 返回值。在下一节中你将看到这个 XML 文档。 |
| [2] | 每个 SOAP 服务都有一个 URL 用以处理所有请求。相同的 URL 可以用于所有的函数请求。每个特定服务则只有一个函数。但稍后你将看到的 Google API 却有多个函数。这个服务的 URL 提供给所有函数分享。每个 SOAP 服务都有一个命名空间 (namespace)，这个命名空间是由服务器任意命名的。这不过是为调用 SOAP 方法设置的。它使得服务器让多个不相关的服务共享服务 URL 和路径请求成为可能。这与 Python 中模块相对于包的关系类似。 |
| [3] | 这里你创建了包含服务 URL 和服务命名空间的 `SOAPProxy`。此时还不会连接到 SOAP 服务器；仅仅是建立了一个本地 Python 对象。 |
| [4] | 到此为止，如果你的设置完全正确，应该可以向调用本地函数一样调用远程 SOAP 方法。这和给普通函数传递参数并接收返回值一样，但在背后却隐藏着很多的工作。 |

让我们看一看这些背后的工作。

# 12.4\. SOAP 网络服务查错

# 12.4\. SOAP 网络服务查错

SOAP 提供了一个很方便的方法用以查看背后的情形。

`SOAPProxy` 的两个小设置就可以打开查错模式。

## 例 12.7\. SOAP 网络服务查错

```py
>>> from SOAPpy import SOAPProxy
>>> url = 'http://services.xmethods.net:80/soap/servlet/rpcrouter'
>>> n = 'urn:xmethods-Temperature'
>>> server = SOAPProxy(url, namespace=n)     
>>> server.config.dumpSOAPOut = 1            
>>> server.config.dumpSOAPIn = 1
>>> temperature = server.getTemp('27502')    
*** Outgoing SOAP ******************************************************
<?xml version="1.0" encoding="UTF-8"?>
<SOAP-ENV:Envelope SOAP-ENV:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"
  xmlns:SOAP-ENC="http://schemas.xmlsoap.org/soap/encoding/"

  xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/"
  >
<SOAP-ENV:Body>
<ns1:getTemp  SOAP-ENC:root="1">
<v1 xsi:type="xsd:string">27502</v1>
</ns1:getTemp>
</SOAP-ENV:Body>
</SOAP-ENV:Envelope>
************************************************************************
*** Incoming SOAP ******************************************************
<?xml version='1.0' encoding='UTF-8'?>
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/"

  >
<SOAP-ENV:Body>
<ns1:getTempResponse 
  SOAP-ENV:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
<return xsi:type="xsd:float">80.0</return>
</ns1:getTempResponse>
</SOAP-ENV:Body>
</SOAP-ENV:Envelope>
************************************************************************ 
>>> temperature
80.0 
```

|  |  |
| --- | --- |
| [1] | 首先，和平常一样，建立带有服务 URL 和命名空间的 `SOAPProxy`。 |
| [2] | 然后，通过设置 `server.config.dumpSOAPIn` 和 `server.config.dumpSOAPOut` 打开查错模式。 |
| [3] | 最后，和平常一样，调用远程 SOAP 方法。SOAP 库将会输出送出的 XML 请求文档和收到的 XML 返回文档。这是 `SOAPProxy` 为你做的所有工作。有点恐怖，不是吗？让我们来分析一下。 |

大部分 XML 请求文档都基于模板文件。忽略所有命名空间声明这些对于所有 SOAP 调用都一成不变的东西。这个 “函数调用” 的核心是`&lt;Body&gt;` 当中的部分：

```py
<ns1:getTemp                                 

  SOAP-ENC:root="1">
<v1 xsi:type="xsd:string">27502</v1>         
</ns1:getTemp> 
```

|  |  |
| --- | --- |
| [1] | 这个元素名 `getTemp` 就是函数名。`SOAPProxy` 使用 `getattr` 作为分发器。有别于使用方法名分别调用本地方法，这里使用方法名构造了一个 XML 请求文档。 |
| [2] | 函数的 XML 元素被存储于一个特别的命名空间，这个命名空间就是你在建立 `SOAPProxy` 对象时所指定的那个命名空间。也不必为 `SOAP-ENC:root` 而苦恼，因为它也是基于模板文件的。 |
| [3] | 函数的参数也被记入 XML 文档。`SOAPProxy` 查看并确定每个参数的数据类型 (这里是 string 字符串类型)。参数的数据类型记入 `xsi:type` 属性，并在其后记入实际的字符串值。 |

返回的 XML 文档同样容易理解，重点在于知道应该忽略掉哪些内容。把注意力集中在 `&lt;Body&gt;` 部分：

```py
<ns1:getTempResponse                             

  SOAP-ENV:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
<return xsi:type="xsd:float">80.0</return>       
</ns1:getTempResponse> 
```

|  |  |
| --- | --- |
| [1] | 服务器传回的值记录在 `&lt;getTempResponse&gt;` 部分的几行中。通常包括函数名和回应 `(Response)`。当然其他的内容也可能出现在这里，但 `SOAPProxy` 所重视的不是这里的元素名，而是命名空间。 |
| [2] | 服务器返回时所使用的命名空间就是在请求时所用的命名空间，也就是在创建 `SOAPProxy` 对象时所指定的命名空间。本章稍后的部分中，我们将看到在创建 `SOAPProxy` 对象时忘记指定功能名空间会怎样。 |
| [3] | 这是返回值和它的数据类型 (浮点类型 float)。`SOAPProxy` 使用显式数据类型创建一个本地数据类型的 Python 对象并返回之。 |

# 12.5\. WSDL 介绍

# 12.5\. WSDL 介绍

`SOAPProxy` 类本地方法调用并透明地转向到远程 SOAP 方法。正如你所看到的，这是很多的工作，`SOAPProxy` 快速和透明地完成他们。它没有做到的是提供方法自省的手段。

考虑一下：前面两部分所展现的调用只有一个参数和返回的简单远程 SOAP 方法。服务 URL 和一系列参数及它们的数据类型需要被知道并跟踪。任何的缺失或错误都会导致整体的失败。

这并没有什么可惊讶的。如果我要调用一个本地函数，我需要知道函数所在的包和模块名 (与之对应的则是服务 URL 和命名空间)。我还需要知道正确的函数名以及其函数个数。Python 精妙地不需明示类型，但我还是需要知道有多少个参数需要传递，多少个值将被返回。

最大的区别就在于内省。就像你在 第四章 看到的那样，Python 擅长于让你实时地去探索模块和函数的情况。你可以对一个模块中的所有函数进行列表，并不费吹灰之力地明了函数的声明和参数情况。

WSDL 允许你对 SOAP 网络服务做相同的事情。WSDL 是 “网络服务描述语言 (Web Services Description Language)”的缩写。它尽管是为自如地表述多种类型的网络服务而设定，却也经常用于描述 SOAP 网络服务。

一个 WSDL 文件不过就是一个文件。更具体地讲，是一个 XML 文件。通常存储于你所访问的 SOAP 网络服务这个被描述对象所在的服务器上，并没有什么特殊之处。在本章稍后的位置，我们将下载 Google API 的 WSDL 文件并在本地使用它。这并不意味着本地调用 Google，这个 WSDL 文件所描述的仍旧是 Google 服务器上的远程函数。

在 WSDL 文件中描述了调用相应的 SOAP 网络服务的一切：

*   服务 URL 和命名空间
*   网络服务的类型 (可能是 SOAP 的函数调用，但我说过，WSDL 足够自如地去描述网络服务的广泛内容)
*   有效函数列表
*   每个函数的参数
*   每个参数的类型
*   每个函数的返回值及其数据类型

换言之，一个 WSDL 文件告诉你调用 SOAP 所需要知道的一切。

# 12.6\. 以 WSDL 进行 SOAP 内省

# 12.6\. 以 WSDL 进行 SOAP 内省

就像网络服务舞台上的所有事物，WSDL 也经历了一个充满明争暗斗而且漫长多变的历史。我不打算讲述这段令我伤心的历史。还有一些其他的标准提供相同的支持，但 WSDL 还是胜出，所以我们还是来学习一下如何使用它。

WSDL 最基本的功能便是让你揭示 SOAP 服务器所提供的有效方法。

## 例 12.8\. 揭示有效方法

```py
>>> from SOAPpy import WSDL          
>>> wsdlFile = 'http://www.xmethods.net/sd/2001/TemperatureService.wsdl'
>>> server = WSDL.Proxy(wsdlFile)    
>>> server.methods.keys()            
[u'getTemp'] 
```

|  |  |
| --- | --- |
| [1] | SOAPpy 包含一个 WSDL 解析器。在本书写作之时，它被标示为开发的初级阶段，但我从来没有在解析任何 WSDL 文件时遇到问题。 |
| [2] | 使用一个 WSDL 文件，你还是要用到一个 proxy 类：`WSDL.Proxy`，它只需一个参数：WSDL 文件。我指定的是存储在远程服务器上的 WSDL 的 URL，但是这个 proxy 类对于本地的 WSDL 副本工作同样出色。创建 WSDL proxy 将会下载 WSDL 文件并解析它，所以如果 WSDL 文件有任何问题 (或者由于网络问题不能获得) 你会立刻知道。 |
| [3] | WSDL proxy 类通过 Python 字典 `server.methods` 揭示有效函数。所以列出有效方法只需调用字典方法 `keys()`。 |

好的，你知道这个 SOAP 服务器提供一个方法：`getTemp`。但是如何去调用它呢？WSDL 也在这方面提供信息。

## 例 12.9\. 揭示一个方法的参数

```py
>>> callInfo = server.methods['getTemp']  
>>> callInfo.inparams                     
[<SOAPpy.wstools.WSDLTools.ParameterInfo instance at 0x00CF3AD0>]
>>> callInfo.inparams[0].name             
u'zipcode'
>>> callInfo.inparams[0].type             
(u'http://www.w3.org/2001/XMLSchema', u'string') 
```

|  |  |
| --- | --- |
| [1] | `server.methods` 字典中记录一个 SOAPpy 的特别结构，被称为 `CallInfo`。`CallInfo` 对象中包含着特定函数和函数参数的信息。 |
| [2] | 函数参数信息存储在 `callInfo.inparams` 中，这是一个记录每一个参数信息的 `ParameterInfo` 对象的 Python 列表。 |
| [3] | 每个 `ParameterInfo` 对象包含一个 `name` 属性，这便是参数名。在通过 SOAP 调用函数时，你不需要知道参数名，但 SOAP 支持在调用函数时使用参数名 (类似于 Python)。如果使用参数名，`WSDL.Proxy` 将会正确地把这些参数关联到远程函数。 |
| [4] | 每个参数都是都是显式类型的，使用的是在 XML Schema 定义的数据类型。你可以在上一节中发现这一点：XML Schema 命名空间是我让你忽略的模版的一部分。就目前而言，你还是可以继续忽略它。`zipcode` 参数是一个字符串，如果你向 `WSDL.Proxy` 对象传递一个 Python 字符串，它会被正确地关联和传递到服务器。 |

WSDL 还允许你自省函数的返回值。

## 例 12.10\. 揭示方法返回值

```py
>>> callInfo.outparams            
[<SOAPpy.wstools.WSDLTools.ParameterInfo instance at 0x00CF3AF8>]
>>> callInfo.outparams[0].name    
u'return'
>>> callInfo.outparams[0].type
(u'http://www.w3.org/2001/XMLSchema', u'float') 
```

|  |  |
| --- | --- |
| [1] | 与揭示函数参数的 `callInfo.inparams` 对应的是揭示返回值的 `callInfo.outparams`。它也同样是一个列表，因为通过 SOAP 调用函数时可以返回多个值，就像 Python 函数一样。 |
| [2] | `ParameterInfo` 对象包含 `name` 和 `type`。这个函数返回一个浮点值，它的名字是 `return`。 |

让我们整合一下，通过 WSDL proxy 调用一个 SOAP 网络服务。

## 例 12.11\. 通过 WSDL proxy 调用一个 SOAP 网络服务

```py
>>> from SOAPpy import WSDL
>>> wsdlFile = 'http://www.xmethods.net/sd/2001/TemperatureService.wsdl')
>>> server = WSDL.Proxy(wsdlFile)               
>>> server.getTemp('90210')                     
66.0
>>> server.soapproxy.config.dumpSOAPOut = 1     
>>> server.soapproxy.config.dumpSOAPIn = 1
>>> temperature = server.getTemp('90210')
*** Outgoing SOAP ******************************************************
<?xml version="1.0" encoding="UTF-8"?>
<SOAP-ENV:Envelope SOAP-ENV:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"
  xmlns:SOAP-ENC="http://schemas.xmlsoap.org/soap/encoding/"

  xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/"
  >
<SOAP-ENV:Body>
<ns1:getTemp  SOAP-ENC:root="1">
<v1 xsi:type="xsd:string">90210</v1>
</ns1:getTemp>
</SOAP-ENV:Body>
</SOAP-ENV:Envelope>
************************************************************************
*** Incoming SOAP ******************************************************
<?xml version='1.0' encoding='UTF-8'?>
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/"

  >
<SOAP-ENV:Body>
<ns1:getTempResponse 
  SOAP-ENV:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
<return xsi:type="xsd:float">66.0</return>
</ns1:getTempResponse>
</SOAP-ENV:Body>
</SOAP-ENV:Envelope>
************************************************************************ 
>>> temperature
66.0 
```

|  |  |
| --- | --- |
| [1] | 这比直接调用 SOAP 服务时的设置简单，因为在 WSDL 文件中包含着调用服务所需要的服务 URL 和命名空间。创建 `WSDL.Proxy` 对象将会下载 WSDL 文件，解析之，并设置一个用以调用实际的 SOAP 网络服务的 `SOAPProxy` 对象。 |
| [2] | 只要创建了 `WSDL.Proxy` 对象，你就可以像调用 `SOAPProxy` 对象一样简单地调用一个函数。这并不奇怪，`WSDL.Proxy` 就是一个具有自省方法的 `SOAPProxy` 封装套件，所以调用函数的语法也是一样的。 |
| [3] | 你可以通过 `server.soapproxy` 访问 `WSDL.Proxy` 的 `SOAPProxy`。这对于打开查错模式很重要，这样一来当你通过 WSDL proxy 调用函数时，它的 `SOAPProxy` 将会把线路上来往的 XML 文档甩下来。 |

# 12.7\. 搜索 Google

# 12.7\. 搜索 Google

让我们回到这章开始时你看到的那段代码，获得比当前气温更有价值和令人振奋的信息。

Google 提供了一个 SOAP API，以便通过程序进行 Google 搜索。使用它的前提是，你注册了 Google 网络服务。

## 过程 12.4\. 注册 Google 网络服务

1.  访问 [`www.google.com/apis/`](http://www.google.com/apis/) 并创建一个账号。唯一的需要是提供一个 E-mail 地址。注册之后，你将通过 E-mail 收到你的 Google API 许可证 (license key)。你需要在调用 Google 搜索函数时使用这个许可证。

2.  还是在 [`www.google.com/apis/`](http://www.google.com/apis/) 上，下载 Google 网络 APIs 开发工具包 (Google Web APIs developer kit)。它包含着包括 Python 在内的多种语言的样例代码，更重要的是它包含着 WSDL 文件。

3.  解压这个开发工具包并找到 `GoogleSearch.wsdl`。将这个文件拷贝到你本地驱动器的一个永久地址。在本章后面位置你会用到它。

你有了开发许可证和 Google WSDL 文件之后就可以和 Google 网络服务打交道了。

## 例 12.12\. 内省 Google 网络服务

```py
>>> from SOAPpy import WSDL
>>> server = WSDL.Proxy('/path/to/your/GoogleSearch.wsdl') 
>>> server.methods.keys()                                  
[u'doGoogleSearch', u'doGetCachedPage', u'doSpellingSuggestion']
>>> callInfo = server.methods['doGoogleSearch']
>>> for arg in callInfo.inparams:                          
... print arg.name.ljust(15), arg.type
key             (u'http://www.w3.org/2001/XMLSchema', u'string')
q               (u'http://www.w3.org/2001/XMLSchema', u'string')
start           (u'http://www.w3.org/2001/XMLSchema', u'int')
maxResults      (u'http://www.w3.org/2001/XMLSchema', u'int')
filter          (u'http://www.w3.org/2001/XMLSchema', u'boolean')
restrict        (u'http://www.w3.org/2001/XMLSchema', u'string')
safeSearch      (u'http://www.w3.org/2001/XMLSchema', u'boolean')
lr              (u'http://www.w3.org/2001/XMLSchema', u'string')
ie              (u'http://www.w3.org/2001/XMLSchema', u'string')
oe              (u'http://www.w3.org/2001/XMLSchema', u'string') 
```

|  |  |
| --- | --- |
| [1] | 步入 Google 网络服务很简单：建立一个 `WSDL.Proxy` 对象并指向到你复制到本地的 Google WSDL 文件。 |
| [2] | 由 WSDL 文件可知，Google 提供三个函数：`doGoogleSearch`、`doGetCachedPage` 和 `doSpellingSuggestion`。顾名思义，执行 Google 搜索并返回结果；获得 Google 最后一次扫描该页时获得的缓存；基于常见拼写错误提出单词拼写建议。 |
| [3] | `doGoogleSearch` 函数需要一系列不同类型的参数。注意：WSDL 文件可以告诉你有哪些参数和他们的参数类型，但不能告诉你它们的含义和使用方法。在参数值有限定的情况下，理论上它能够告诉你参数的取值范围，但 Google 的 WSDL 没有那么细化。`WSDL.Proxy` 不会变魔术，它只能给你 WSDL 文件中提供的信息。 |

这里简要地列出了 `doGoogleSearch` 函数的所有参数：

*   `key`――你注册 Google 网络服务时获得的 Google API 许可证。
*   `q`――你要搜索的词或词组。其语法与 Google 的网站表单处完全相同，你所知道的高级搜索语法和技巧这里完全适用。
*   `start`――起始的结果编号。与使用 Google 网页交互搜索时相同，这个函数每次返回 10 个结果。如果你需要查看 “第二” 页结果则需要将 `start` 设置为 10。
*   `maxResults`――返回的结果个数。目前的值是 10，当然如果你只对少数返回结果感兴趣或者希望节省网络带宽，也可以定义为返回更少的结果。
*   `filter`――如果设置为 `True`，Google 将会过滤结果中重复的页面。
*   `restrict`――这里设置 `country` 并跟上一个国家代码可以限定只返回特定国家的结果。例如：`countryUK` 用于在英国搜索页面。你也可以设定 `linux`，`mac` 或者 `bsd` 以便搜索 Google 定义的技术站点组，或者设为 `unclesam` 来搜索美国政府站点。
*   `safeSearch`――如果设置为 `True`，Google 将会过滤掉色情站点。
*   `lr` (“language restrict”，语言限制)――这里设置语言限定值返回特定语言的站点。
*   `ie` 和 `oe` (“input encoding”，输入编码和 “output encoding”，输出编码)――不赞成使用，都应该是 `utf-8`。

## 例 12.13\. 搜索 Google

```py
>>> from SOAPpy import WSDL
>>> server = WSDL.Proxy('/path/to/your/GoogleSearch.wsdl')
>>> key = 'YOUR_GOOGLE_API_KEY'
>>> results = server.doGoogleSearch(key, 'mark', 0, 10, False, "",
... False, "", "utf-8", "utf-8")             
>>> len(results.resultElements)                  
10
>>> results.resultElements[0].URL                
'http://diveintomark.org/'
>>> results.resultElements[0].title
'dive into <b>mark</b>' 
```

|  |  |
| --- | --- |
| [1] | 在设置好 `WSDL.Proxy` 对象之后，你可以使用十个参数来调用 `server.doGoogleSearch`。记住要使用你注册 Google 网络服务时授权给你自己的 Google API 许可证。 |
| [2] | 有很多的返回信息，但我们还是先来看一下实际的返回结果。它们被存储于 `results.resultElements` 之中，你可以像使用普通的 Python 列表那样来调用它。 |
| [3] | `resultElements` 中的每个元素都是一个包含 `URL`、`title`、`snippet` 以及其他属性的对象。基于这一点，你可以使用诸如 **`dir(results.resultElements[0])`** 的普通 Python 自省技术来查看有效属性，或者通过 WSDL proxy 对象查看函数的 `outparams`。不同的方法能带给你相同的结果。 |

`results` 对象中所加载的不仅仅是实际的搜索结果。它也含有搜索行为自身的信息，比如耗时和总结果数等 (尽管只返回了 10 条结果)。Google 网页界面中显示了这些信息，通过程序你也同样能获得它们。

## 例 12.14\. 从 Google 获得次要信息

```py
>>> results.searchTime                     
0.224919
>>> results.estimatedTotalResultsCount     
29800000
>>> results.directoryCategories            
[<SOAPpy.Types.structType item at 14367400>:
 {'fullViewableName':
  'Top/Arts/Literature/World_Literature/American/19th_Century/Twain,_Mark',
  'specialEncoding': ''}]
>>> results.directoryCategories[0].fullViewableName
'Top/Arts/Literature/World_Literature/American/19th_Century/Twain,_Mark' 
```

|  |  |
| --- | --- |
| [1] | 这个搜索耗时 0.224919 秒。这不包括用于发送和接收 SOAP XML 文档的时间，仅仅是 Google 在接到搜索请求后执行搜索所花费的时间。 |
| [2] | 总共有接近 30,000,000 个结果信息。通过让 `start` 参数以 10 递增来重复调用 `server.doGoogleSearch`，你能够获得全部的结果。 |
| [3] | 对于有些请求，Google 还返回一个 [Google Directory](http://directory.google.com/) 中的类别列表。你可以用这些 URLs 到 [`directory.google.com/`](http://directory.google.com/) 建立到 directory category 页面的链接。 |

# 12.8\. SOAP 网络服务故障排除

# 12.8\. SOAP 网络服务故障排除

是的，SOAP 网络服务的世界中也不总是欢乐和阳光。有时候也会有故障。

正如你在本章中看到的，SOAP 牵扯了很多层面。SOAP 向 HTTP 服务器发送 XML 文档并接收返回的 XML 文档时需要用到 HTTP 层。这样一来，你在 第十一章 *HTTP Web 服务* 学到的调试技术在这里都有了用武之地。你可以 **`import httplib`** 并设置 **`httplib.HTTPConnection.debuglevel = 1`** 来查看潜在的 HTTP 传输。

在 HTTP 层之上，还有几个可能发生问题的地方。SOAPpy 隐藏 SOAP 语法的本领令你惊叹不已，但也意味着在发生问题时更难确定问题所在。

下面的这些例子是我在使用 SOAP 网络服务时犯过的一些常见错误以及所产生的错误信息。

## 例 12.15\. 以错误的设置调用 Proxy 方法

```py
>>> from SOAPpy import SOAPProxy
>>> url = 'http://services.xmethods.net:80/soap/servlet/rpcrouter'
>>> server = SOAPProxy(url)                                        
>>> server.getTemp('27502')                                        
<Fault SOAP-ENV:Server.BadTargetObjectURI:
Unable to determine object id from call: is the method element namespaced?>
Traceback (most recent call last):
  File "<stdin>", line 1, in ?
  File "c:\python23\Lib\site-packages\SOAPpy\Client.py", line 453, in __call__
    return self.__r_call(*args, **kw)
  File "c:\python23\Lib\site-packages\SOAPpy\Client.py", line 475, in __r_call
    self.__hd, self.__ma)
  File "c:\python23\Lib\site-packages\SOAPpy\Client.py", line 389, in __call
    raise p
SOAPpy.Types.faultType: <Fault SOAP-ENV:Server.BadTargetObjectURI:
Unable to determine object id from call: is the method element namespaced?> 
```

|  |  |
| --- | --- |
| [1] | 你看出错误了吗？你手工地创建了一个 `SOAPProxy`，你正确地指定了服务 URL，但是你没有指定命名空间。由于多个服务可能被路由到相同的服务 URL，命名空间是确定你所调用的服务和方法的重要内容。 |
| [2] | 服务器返回的是一个 SOAP 错误 (Fault)，SOAPpy 把它转换为 Python 异常 `SOAPpy.Types.faultType`。从任何 SOAP 服务器返回的错误都是 SOAP 错误，因此你可以轻易地捕获这个异常。就此处而言，我们能从 SOAP 错误信息中看出端倪：由于源 `SOAPProxy` 对象没有设置服务命名空间，因此方法元素也就没有了命名空间。 |

错误配置 SOAP 服务的基本元素是 WSDL 着眼解决的问题。WSDL 文件包含服务 URL 和命名空间，所以你应该不会在这里犯错。但是，还有其他可能出错的地方。

## 例 12.16\. 以错误参数调用方法

```py
>>> wsdlFile = 'http://www.xmethods.net/sd/2001/TemperatureService.wsdl'
>>> server = WSDL.Proxy(wsdlFile)
>>> temperature = server.getTemp(27502)                                
<Fault SOAP-ENV:Server: Exception while handling service request:
services.temperature.TempService.getTemp(int) -- no signature match>   [2]
Traceback (most recent call last):
  File "<stdin>", line 1, in ?
  File "c:\python23\Lib\site-packages\SOAPpy\Client.py", line 453, in __call__
    return self.__r_call(*args, **kw)
  File "c:\python23\Lib\site-packages\SOAPpy\Client.py", line 475, in __r_call
    self.__hd, self.__ma)
  File "c:\python23\Lib\site-packages\SOAPpy\Client.py", line 389, in __call
    raise p
SOAPpy.Types.faultType: <Fault SOAP-ENV:Server: Exception while handling service request:
services.temperature.TempService.getTemp(int) -- no signature match> 
```

|  |  |
| --- | --- |
| [1] | 你看出错误了吗？这是一个不易察觉的错误：你在使用整数而不是字符串来调用 `server.getTemp` 。自省 WSDL 文件不难发现，`getTemp()` 这个 SOAP 函数接受一个参数 `zipcode`，这是一个字符串参数。`WSDL.Proxy` *不* 会为你强制转换数据类型；你需要根据服务器需要的数据类型传递数据。 |
| [2] | 又是这样，服务器传回一个 SOAP 错误，你能从 SOAP 错误信息中看出端倪：你在使用整数类型的参数调用 `getTemp` 函数，但却没有一个以此命名的函数接收整数参数。理论上讲，SOAP 允许你重载 (*overload*) 函数，也就是可以在同一个 SOAP 服务中存在同名函数，并且参数个数也相同，但是参数的数据类型不同。这就是数据类型必须匹配的原因，也说明了为什么 `WSDL.Proxy` 不强制地为你改变数据类型。如果真的强制改变了数据类型，发生这样的错误时，调用的可能是另外一个不相干的函数。看来产生这样的错误是件幸运的事。对于数据类型多加注意会让事情简单很多，一旦搞错了数据类型便立刻会发生错误。 |

Python 所期待的返回值个数与远程函数的实际返回值个数不同是另一种可能的错误。

## 例 12.17\. 调用时方法所期待的返回值个数错误

```py
>>> wsdlFile = 'http://www.xmethods.net/sd/2001/TemperatureService.wsdl'
>>> server = WSDL.Proxy(wsdlFile)
>>> (city, temperature) = server.getTemp(27502)  
Traceback (most recent call last):
  File "<stdin>", line 1, in ?
TypeError: unpack non-sequence 
```

|  |  |
| --- | --- |
| [1] | 你看出错误了吗？`server.getTemp` 只返回一个浮点值，但你写的代码却期待着获得两个值，并把它们赋值给不同的两个变量。注意这不是一个 SOAP 错误。就远程服务器而言没有发生任何错误。错误发生在完成 SOAP 交割*之后*。`WSDL.Proxy` 返回一个浮点数，你本地的 Python 解释器试图将这个浮点数分成两个变量。由于函数只返回了一个值，你在试图分割它时所获得的是一个 Python 异常，而不是 SOAP 错误。 |

那么 Google 网络服务方面又如何呢？我曾经犯过的最常见的错误是忘记正确设置应用许可证。

## 例 12.18\. 调用方法返回一个应用特定的错误

```py
>>> from SOAPpy import WSDL
>>> server = WSDL.Proxy(r'/path/to/local/GoogleSearch.wsdl')
>>> results = server.doGoogleSearch('foo', 'mark', 0, 10, False, "", 
... False, "", "utf-8", "utf-8")
<Fault SOAP-ENV:Server:                                              [2]
 Exception from service object: Invalid authorization key: foo:
 <SOAPpy.Types.structType detail at 14164616>:
 {'stackTrace':
  'com.google.soap.search.GoogleSearchFault: Invalid authorization key: foo
   at com.google.soap.search.QueryLimits.lookUpAndLoadFromINSIfNeedBe(
     QueryLimits.java:220)
   at com.google.soap.search.QueryLimits.validateKey(QueryLimits.java:127)
   at com.google.soap.search.GoogleSearchService.doPublicMethodChecks(
     GoogleSearchService.java:825)
   at com.google.soap.search.GoogleSearchService.doGoogleSearch(
     GoogleSearchService.java:121)
   at sun.reflect.GeneratedMethodAccessor13.invoke(Unknown Source)
   at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
   at java.lang.reflect.Method.invoke(Unknown Source)
   at org.apache.soap.server.RPCRouter.invoke(RPCRouter.java:146)
   at org.apache.soap.providers.RPCJavaProvider.invoke(
     RPCJavaProvider.java:129)
   at org.apache.soap.server.http.RPCRouterServlet.doPost(
     RPCRouterServlet.java:288)
   at javax.servlet.http.HttpServlet.service(HttpServlet.java:760)
   at javax.servlet.http.HttpServlet.service(HttpServlet.java:853)
   at com.google.gse.HttpConnection.runServlet(HttpConnection.java:237)
   at com.google.gse.HttpConnection.run(HttpConnection.java:195)
   at com.google.gse.DispatchQueue$WorkerThread.run(DispatchQueue.java:201)
Caused by: com.google.soap.search.UserKeyInvalidException: Key was of wrong size.
   at com.google.soap.search.UserKey.<init>(UserKey.java:59)
   at com.google.soap.search.QueryLimits.lookUpAndLoadFromINSIfNeedBe(
     QueryLimits.java:217)
   ... 14 more
'}>
Traceback (most recent call last):
  File "<stdin>", line 1, in ?
  File "c:\python23\Lib\site-packages\SOAPpy\Client.py", line 453, in __call__
    return self.__r_call(*args, **kw)
  File "c:\python23\Lib\site-packages\SOAPpy\Client.py", line 475, in __r_call
    self.__hd, self.__ma)
  File "c:\python23\Lib\site-packages\SOAPpy\Client.py", line 389, in __call
    raise p
SOAPpy.Types.faultType: <Fault SOAP-ENV:Server: Exception from service object:
Invalid authorization key: foo:
<SOAPpy.Types.structType detail at 14164616>:
{'stackTrace':
  'com.google.soap.search.GoogleSearchFault: Invalid authorization key: foo
   at com.google.soap.search.QueryLimits.lookUpAndLoadFromINSIfNeedBe(
     QueryLimits.java:220)
   at com.google.soap.search.QueryLimits.validateKey(QueryLimits.java:127)
   at com.google.soap.search.GoogleSearchService.doPublicMethodChecks(
     GoogleSearchService.java:825)
   at com.google.soap.search.GoogleSearchService.doGoogleSearch(
     GoogleSearchService.java:121)
   at sun.reflect.GeneratedMethodAccessor13.invoke(Unknown Source)
   at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
   at java.lang.reflect.Method.invoke(Unknown Source)
   at org.apache.soap.server.RPCRouter.invoke(RPCRouter.java:146)
   at org.apache.soap.providers.RPCJavaProvider.invoke(
     RPCJavaProvider.java:129)
   at org.apache.soap.server.http.RPCRouterServlet.doPost(
     RPCRouterServlet.java:288)
   at javax.servlet.http.HttpServlet.service(HttpServlet.java:760)
   at javax.servlet.http.HttpServlet.service(HttpServlet.java:853)
   at com.google.gse.HttpConnection.runServlet(HttpConnection.java:237)
   at com.google.gse.HttpConnection.run(HttpConnection.java:195)
   at com.google.gse.DispatchQueue$WorkerThread.run(DispatchQueue.java:201)
Caused by: com.google.soap.search.UserKeyInvalidException: Key was of wrong size.
   at com.google.soap.search.UserKey.<init>(UserKey.java:59)
   at com.google.soap.search.QueryLimits.lookUpAndLoadFromINSIfNeedBe(
     QueryLimits.java:217)
   ... 14 more
'}> 
```

|  |  |
| --- | --- |
| [1] | 你看出错误了吗？调用的语法，参数个数以及数据类型都没有错误。这个问题是应用特定的：第一个参数应该是我的应用许可证，但 `foo` 不是一个有效的 Google 许可证。 |
| [2] | Google 服务器返回的是一个 SOAP 错误和一大串特别长的错误信息，其中包含了完整的 Java 堆栈跟踪。记住*所有* 的 SOAP 错误都被标示为 SOAP Faults: errors in configuration (设置错误), errors in function arguments (函数参数错误)，或者是应用特定的错误 (这里就是) 等等。在其中埋藏的至关重要信息是：`Invalid authorization key: foo` (非有效授权许可证：foo)。 |

## 进一步阅读

*   New developments for SOAPpy 一步步连接到另一个不名副其实的 SOAP 服务。

# 12.9\. 小结

# 12.9\. 小结

SOAP 网络服务是很复杂的，雄心勃勃的它试图涵盖网络服务的很多不同应用。这一章我们接触了它的一个简单应用。

在开始下一章的学习之前，确保你能自如地做如下工作：

*   连接到 SOAP 服务器并调用远程方法
*   通过 WSDL 文件自省远程方法
*   有效排除 SOAP 调用中的错误
*   排除常见的 SOAP 相关错误