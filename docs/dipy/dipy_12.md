# 第十一章 HTTP Web 服务

# 第十一章 HTTP Web 服务

*   11.1\. 概览
*   11.2\. 避免通过 HTTP 重复地获取数据
*   11.3\. HTTP 的特性
    *   11.3.1\. 用户代理 (User-Agent)
    *   11.3.2\. 重定向 (Redirects)
    *   11.3.3\. Last-Modified/If-Modified-Since
    *   11.3.4\. ETag/If-None-Match
    *   11.3.5\. 压缩 (Compression)
*   11.4\. 调试 HTTP web 服务
*   11.5\. 设置 User-Agent
*   11.6\. 处理 Last-Modified 和 ETag
*   11.7\. 处理重定向
*   11.8\. 处理压缩数据
*   11.9\. 全部放在一起
*   11.10\. 小结

# 11.1\. 概览

# 11.1\. 概览

在讲解如何下载 web 页和如何从 URL 解析 XML 时，你已经学习了关于 HTML 处理和 XML 处理，接下来让我们来更全面地探讨有关 HTTP web 服务的主题。

简单地讲，HTTP web 服务是指以编程的方式直接使用 HTTP 操作从远程服务器发送和接收数据。如果你要从服务器获取数据，直接使用 HTTP GET；如果您想发送新数据到服务器，使用 HTTP POST。(一些较高级的 HTTP web 服务 API 也定义了使用 HTTP PUT 和 HTTP DELETE 修改和删除现有数据的方法。) 换句话说，构建在 HTTP 协议中的 “verbs (动作)” (GET, POST, PUT 和 DELETE) 直接映射为接收、发送、修改和删除等应用级别的操作。

这种方法的主要优点是简单，并且许多不同的站点充分印证了这样的简单性是受欢迎的。数据 (通常是 XML 数据) 能静态创建和存储，或通过服务器端脚本和所有主流计算机语言 (包括用于下载数据的 HTTP 库) 动态生成。调试也很简单，因为您可以在任意浏览器中调用网络服务来查看这些原始数据。现代浏览器甚至可以为您进行良好的格式化并漂亮地打印这些 XML 数据，以便让您快速地浏览。

HTTP web 服务上的纯 XML 应用举例：

*   Amazon API 允许您从 Amazon.com 在线商店获取产品信息。
*   National Weather Service (美国) 和 [Hong Kong Observatory](http://demo.xml.weather.gov.hk/) (香港) 通过 web 服务提供天气警报。
*   Atom API 用来管理基于 web 的内容。
*   Syndicated feeds 应用于 weblogs 和新闻站点中带给您来自众多站点的最新消息。

在后面的几章里，我们将探索使用 HTTP 进行数据发送和接收传输的 API，但是不会将应用语义映射到潜在的 HTTP 语义。(所有这些都是通过 HTTP POST 这个管道完成的。) 但是本章将关注使用 HTTP GET 从远程服务器获取数据，并且将探索几个由纯 HTTP web 服务带来最大利益的 HTTP 特性。

如下所示为上一章曾经看到过的 `openanything` 模块的更高级版本：

## 例 11.1\. `openanything.py`

如果您还没有下载本书附带的样例程序, 可以 [下载本程序和其他样例程序](http://www.woodpecker.org.cn/diveintopython/download/diveintopython-exampleszh-cn-5.4b.zip "Download example scripts")。

```py
 import urllib2, urlparse, gzip
from StringIO import StringIO
USER_AGENT = 'OpenAnything/1.0 +http://diveintopython.org/http_web_services/'
class SmartRedirectHandler(urllib2.HTTPRedirectHandler):    
    def http_error_301(self, req, fp, code, msg, headers):  
        result = urllib2.HTTPRedirectHandler.http_error_301(
            self, req, fp, code, msg, headers)              
        result.status = code                                
        return result                                       
    def http_error_302(self, req, fp, code, msg, headers):  
        result = urllib2.HTTPRedirectHandler.http_error_302(
            self, req, fp, code, msg, headers)              
        result.status = code                                
        return result                                       
class DefaultErrorHandler(urllib2.HTTPDefaultErrorHandler):   
    def http_error_default(self, req, fp, code, msg, headers):
        result = urllib2.HTTPError(                           
            req.get_full_url(), code, msg, headers, fp)       
        result.status = code                                  
        return result                                         
def openAnything(source, etag=None, lastmodified=None, agent=USER_AGENT):
    '''URL, filename, or string --> stream
    This function lets you define parsers that take any input source
    (URL, pathname to local or network file, or actual data as a string)
    and deal with it in a uniform manner.  Returned object is guaranteed
    to have all the basic stdio read methods (read, readline, readlines).
    Just .close() the object when you're done with it.
    If the etag argument is supplied, it will be used as the value of an
    If-None-Match request header.
    If the lastmodified argument is supplied, it must be a formatted
    date/time string in GMT (as returned in the Last-Modified header of
    a previous request).  The formatted date/time will be used
    as the value of an If-Modified-Since request header.
    If the agent argument is supplied, it will be used as the value of a
    User-Agent request header.
    '''
    if hasattr(source, 'read'):
        return source
    if source == '-':
        return sys.stdin
    if urlparse.urlparse(source)[0] == 'http':                                      
        # open URL with urllib2 
        request = urllib2.Request(source)                                           
        request.add_header('User-Agent', agent)                                     
        if etag:                                                                    
            request.add_header('If-None-Match', etag)                               
        if lastmodified:                                                            
            request.add_header('If-Modified-Since', lastmodified)                   
        request.add_header('Accept-encoding', 'gzip')                               
        opener = urllib2.build_opener(SmartRedirectHandler(), DefaultErrorHandler())
        return opener.open(request)                                                 
    # try to open with native open function (if source is a filename)
    try:
        return open(source)
    except (IOError, OSError):
        pass
    # treat source as string
    return StringIO(str(source))
def fetch(source, etag=None, last_modified=None, agent=USER_AGENT):  
    '''Fetch data and metadata from a URL, file, stream, or string'''
    result = {}                                                      
    f = openAnything(source, etag, last_modified, agent)             
    result['data'] = f.read()                                        
    if hasattr(f, 'headers'):                                        
        # save ETag, if the server sent one 
        result['etag'] = f.headers.get('ETag')                       
        # save Last-Modified header, if the server sent one 
        result['lastmodified'] = f.headers.get('Last-Modified')      
        if f.headers.get('content-encoding', '') == 'gzip':          
            # data came back gzip-compressed, decompress it 
            result['data'] = gzip.GzipFile(fileobj=StringIO(result['data']])).read()
    if hasattr(f, 'url'):                                            
        result['url'] = f.url                                        
        result['status'] = 200                                       
    if hasattr(f, 'status'):                                         
        result['status'] = f.status                                  
    f.close()                                                        
    return result 
```

## 进一步阅读

*   Paul Prescod 认为[纯 HTTP web 服务是 Internet 的未来](http://webservices.xml.com/pub/a/ws/2002/02/06/rest.html)。

# 11.2\. 避免通过 HTTP 重复地获取数据

# 11.2\. 避免通过 HTTP 重复地获取数据

假如说你想用 HTTP 下载资源，例如一个 Atom feed 汇聚。你不仅仅想下载一次；而是想一次又一次地下载它，如每小时一次，从提供 news feed 的站点获得最新的消息。让我们首先用一种直接而原始的方法来实现它，然后看看如何改进它。

## 例 11.2\. 用直接而原始的方法下载 feed

```py
>>> import urllib
>>> data = urllib.urlopen('http://diveintomark.org/xml/atom.xml').read()    
>>> print data
<?xml version="1.0" encoding="iso-8859-1"?>
<feed version="0.3"

  xml:lang="en">
  <title mode="escaped">dive into mark</title>
  <link rel="alternate" type="text/html" href="http://diveintomark.org/"/>
  <-- rest of feed omitted for brevity --> 
```

|  |  |
| --- | --- |
| [1] | 使用 Python 通过 HTTP 下载任何东西都简单得令人难以置信；实际上，只需要一行代码。`urllib` 模块有一个便利的 `urlopen` 函数，它接受您所要获取的页面地址，然后返回一个类文件对象，您仅仅使用 `read()` 便可获得页面的全部内容。这再简单不过了。 |

那么这种方法有何不妥之处吗？当然，在测试或开发中一次性的使用没有什么不妥。我经常这样。我想要 feed 汇聚的内容，我就获取 feed 的内容。这种方法对其他 web 页面同样有效。但是一旦你开始按照 web 服务的方式去思考有规则的访问需求时 (记住，你说你计划每小时一次地重复获取这样的 feed ) 就会发现这样的做法效率实在是太低了，并且对服务器来说也太笨了。

下面先谈论一些 HTTP 的基本特性。

# 11.3\. HTTP 的特性

# 11.3\. HTTP 的特性

*   11.3.1\. 用户代理 (User-Agent)
*   11.3.2\. 重定向 (Redirects)
*   11.3.3\. Last-Modified/If-Modified-Since
*   11.3.4\. ETag/If-None-Match
*   11.3.5\. 压缩 (Compression)

这里有五个你必须关注的 HTTP 重要特性。

## 11.3.1\. 用户代理 (`User-Agent`)

`User-Agent` 是一种客户端告知服务器谁在什么时候通过 HTTP 请求了一个 web 页、feed 汇聚或其他类型的 web 服务的简单途径。当客户端请求一个资源时，应该尽可能明确发起请求的是谁，以便当产生异常错误时，允许服务器端的管理员与客户端的开发者取得联系。

默认情况下 Python 发送一个通用的 `User-Agent`：`Python-urllib/1.15`。下一节，您将看到更加有针对性的 `User-Agent`。

## 11.3.2\. 重定向 (Redirects)

有时资源移来移去。Web 站点重组内容，页面移动到了新的地址。甚至是 web 服务重组。原来位于 `http://example.com/index.xml` 的 feed 汇聚可能被移动到 `http://example.com/xml/atom.xml`。或者因为一个机构的扩展或重组，整个域被迁移。例如，`http://www.example.com/index.xml` 可能被重定向到 `http://server-farm-1.example.com/index.xml`。

您每次从 HTTP 服务器请求任何类型的资源时，服务器的响应中均包含一个状态代码。状态代码 `200` 的意思是 “一切正常，这就是您请求的页面”。状态代码 `404` 的意思是 “页面没找到”。 (当浏览 web 时，你可能看到过 404 errors。)

HTTP 有两种不同的方法表示资源已经被移动。状态代码 `302` 表示*临时重定向*；这意味着 “哎呀，访问内容被临时移动” (然后在 `Location:` 头信息中给出临时地址)。状态代码 `301` 表示*永久重定向*；这意味着 “哎呀，访问内容被永久移动” (然后在 `Location:` 头信息中给出新地址)。如果您获得了一个 `302` 状态代码和一个新地址，HTTP 规范说您应该使用新地址获取您的请求，但是下次您要访问同一资源时，应该使用原地址重试。但是如果您获得了一个 `301` 状态代码和一个新地址，您应该从此使用新地址。

当从 HTTP 服务器接受到一个适当的状态代码时，`urllib.urlopen` 将自动 “跟踪” 重定向，但不幸的是，当它做了重定向时不会告诉你。 你将最终获得所请求的数据，却丝毫不会察觉到在这个过程中一个潜在的库 “帮助” 你做了一次重定向操作。因此你将继续不断地使用旧地址，并且每次都将获得被重定向的新地址。这一过程要往返两次而不是一次：太没效率了！本章的后面，您将看到如何改进这一点，从而适当地且有效率地处理永久重定向。

## 11.3.3\. `Last-Modified`/`If-Modified-Since`

有些数据随时都在变化。CNN.com 的主页经常几分钟就更新。另一方面，Google.com 的主页几个星期才更新一次 (当他们上传特殊的假日 logo，或为一个新服务作广告时)。 Web 服务是不变的：通常服务器知道你所请求的数据的最后修改时间，并且 HTTP 为服务器提供了一种将最近修改数据连同你请求的数据一同发送的方法。

如果你第二次 (或第三次，或第四次) 请求相同的数据，你可以告诉服务器你上一次获得的最后修改日期：在你的请求中发送一个 `If-Modified-Since` 头信息，它包含了上一次从服务器连同数据所获得的日期。如果数据从那时起没有改变，服务器将返回一个特殊的 HTTP 状态代码 `304`，这意味着 “从上一次请求后这个数据没有改变”。这一点有何进步呢？当服务器发送状态编码 `304` 时，*不再重新发送数据*。您仅仅获得了这个状态代码。所以当数据没有更新时，你不需要一次又一次地下载相同的数据；服务器假定你有本地的缓存数据。

所有现代的浏览器都支持最近修改 (last-modified) 的数据检查。如果你曾经访问过某页，一天后重新访问相同的页时发现它没有变化，并奇怪第二次访问时页面加载得如此之快——这就是原因所在。你的浏览器首次访问时会在本地缓存页面内容，当你第二次访问，浏览器自动发送首次访问时从服务器获得的最近修改日期。服务器简单地返回 `304: Not Modified` (没有修改)，因此浏览器就会知道从本地缓存加载页面。在这一点上，Web 服务也如此智能。

Python 的 URL 库没有提供内置的最近修改数据检查支持，但是你可以为每一个请求添加任意的头信息并在每一个响应中读取任意头信息，从而自己添加这种支持。

## 11.3.4\. `ETag`/`If-None-Match`

ETag 是实现与最近修改数据检查同样的功能的另一种方法：没有变化时不重新下载数据。其工作方式是：服务器发送你所请求的数据的同时，发送某种数据的 hash (在 `ETag` 头信息中给出)。hash 的确定完全取决于服务器。当第二次请求相同的数据时，你需要在 `If-None-Match:` 头信息中包含 ETag hash，如果数据没有改变，服务器将返回 `304` 状态代码。与最近修改数据检查相同，服务器*仅仅* 发送 `304` 状态代码；第二次将不为你发送相同的数据。在第二次请求时，通过包含 ETag hash，你告诉服务器：如果 hash 仍旧匹配就没有必要重新发送相同的数据，因为你还有上一次访问过的数据。

Python 的 URL 库没有对 ETag 的内置支持，但是在本章后面你将看到如何添加这种支持。

## 11.3.5\. 压缩 (Compression)

最后一个重要的 HTTP 特性是 gzip 压缩。 关于 HTTP web 服务的主题几乎总是会涉及在网络线路上传输的 XML。XML 是文本，而且还是相当冗长的文本，而文本通常可以被很好地压缩。当你通过 HTTP 请求一个资源时，可以告诉服务器，如果它有任何新数据要发送给我时，请以压缩的格式发送。在你的请求中包含 `Accept-encoding: gzip` 头信息，如果服务器支持压缩，它将返回由 gzip 压缩的数据并且使用 `Content-encoding: gzip` 头信息标记。

Python 的 URL 库本身没有内置对 gzip 压缩的支持，但是你能为请求添加任意的头信息。Python 还提供了一个独立的 `gzip` 模块，它提供了对数据进行解压缩的功能。

注意我们用于下载 feed 汇聚的小单行脚本并不支持任何这些 HTTP 特性。让我们来看看如何改善它。

# 11.4\. 调试 HTTP web 服务

# 11.4\. 调试 HTTP web 服务

首先，让我们开启 Python HTTP 库的调试特性并查看网络线路上的传输过程。这对本章的全部内容都很有用，因为你将添加越来越多的特性。

## 例 11.3\. 调试 HTTP

```py
>>> import httplib
>>> httplib.HTTPConnection.debuglevel = 1             
>>> import urllib
>>> feeddata = urllib.urlopen('http://diveintomark.org/xml/atom.xml').read()
connect: (diveintomark.org, 80)                       
send: '
GET /xml/atom.xml HTTP/1.0                            
Host: diveintomark.org                                
User-agent: Python-urllib/1.15                        
'
reply: 'HTTP/1.1 200 OK\r\n'                          
header: Date: Wed, 14 Apr 2004 22:27:30 GMT
header: Server: Apache/2.0.49 (Debian GNU/Linux)
header: Content-Type: application/atom+xml
header: Last-Modified: Wed, 14 Apr 2004 22:14:38 GMT  
header: ETag: "e8284-68e0-4de30f80"                   
header: Accept-Ranges: bytes
header: Content-Length: 26848
header: Connection: close 
```

|  |  |
| --- | --- |
| [1] | `urllib` 依赖于另一个 Python 的标准库，`httplib`。通常你不必显式地给出 `import httplib` (`urllib` 会自动导入)，但是你可以为 `HTTPConnection` 类 (`urllib` 在内部使用它来访问 HTTP 服务器) 设置调试标记。这是一种令人难以置信的有用技术。Python 其他的一些库也有类似的调试标记，但是没有命名和开启它们的特殊标准；如果有类似的特性可用，你需要阅读每一个库的文档来查看使用方法。 |
| [2] | 既然已经设置了调试标记，HTTP 的请求和响应信息会实时地被打印出来。首先告诉你的是你连接服务器 `diveintomark.org` 的 80 端口，这是 HTTP 的标准端口。 |
| [3] | 当你请求 Atom feed 时，`urllib` 向服务器发送三行信息。第一行指出你使用的 HTTP verb 和资源的路径 (除去域名)。在本章中所有的请求都将使用 `GET`，但是在下一章的 SOAP 中，你会看到所有的请求都使用 `POST` 。除了请求的动词不同之外，基本的语法是相同的。 |
| [4] | 第二行是 `Host` 头信息，它指出你所访问的服务的域名。这一点很重要，因为一个独立的 HTTP 服务器可以服务于多个不同的域。当前我的服务器服务于 12 个域；其他的服务器可以服务于成百乃至上千个域。 |
| [5] | 第三行是 `User-Agent` 头信息。在此你看到的是由 `urllib` 库默认添加的普通的 `User-Agent` 。在下一节，你会看到如何自定义它的更多细节。 |
| [6] | 服务器用状态代码和一系列的头信息答复 (其中一些数据可能会被存储到 `feeddata` 变量中)。这里的状态代码是 `200`，意味着 “一切正常，这就是您请求的数据”。服务器也会告诉你响应请求的数据、一些有关服务器自身的信息，以及传给你的数据的内容类型。根据你的应用不同，这或许有用，或许没用。这充分确认了你所请求的是一个 Atom feed，瞧，你获得了 Atom feed (`application/atom+xml`，它是已经注册的有关 Atom feeds 的内容类型)。 |
| [7] | 当此 Atom feed 有最近的修改，服务器会告诉你 (本例中，大约发生在 13 分钟之前)。当下次请求同样的 feed 时，你可以这个日期再发送给服务器，服务器将做最近修改数据检查。 |
| [8] | 服务器也会告诉你这个 Atom feed 有一个值为 `"e8284-68e0-4de30f80"` 的 ETag hash。这个 hash 自身没有任何意义；除了在下次访问相同的 feed 时将它送还给服务器之外，你也不需要用它做什么。然后服务器使用它告诉你修改日期是否被改变了。 |

# 11.5\. 设置 `User-Agent`

# 11.5\. 设置 `User-Agent`

改善你的 HTTP web 服务客户端的第一步就是用 `User-Agent` 适当地鉴别你自己。为了做到这一点，你需要远离基本的 `urllib` 而深入到 `urllib2`。

## 例 11.4\. `urllib2` 介绍

```py
>>> import httplib
>>> httplib.HTTPConnection.debuglevel = 1                             
>>> import urllib2
>>> request = urllib2.Request('http://diveintomark.org/xml/atom.xml') 
>>> opener = urllib2.build_opener()                                   
>>> feeddata = opener.open(request).read()                            
connect: (diveintomark.org, 80)
send: '
GET /xml/atom.xml HTTP/1.0
Host: diveintomark.org
User-agent: Python-urllib/2.1
'
reply: 'HTTP/1.1 200 OK\r\n'
header: Date: Wed, 14 Apr 2004 23:23:12 GMT
header: Server: Apache/2.0.49 (Debian GNU/Linux)
header: Content-Type: application/atom+xml
header: Last-Modified: Wed, 14 Apr 2004 22:14:38 GMT
header: ETag: "e8284-68e0-4de30f80"
header: Accept-Ranges: bytes
header: Content-Length: 26848
header: Connection: close 
```

|  |  |
| --- | --- |
| [1] | 如果你的 Python IDE 仍旧为上一节的例子而打开着，你可以略过这一步，在开启 HTTP 调试时你能看到网络线路上的实际传输过程。 |
| [2] | 使用 `urllib2` 获取 HTTP 资源包括三个处理步骤，这会有助于你理解这一过程。 第一步是创建 `Request` 对象，它接受一个你最终想要获取资源的 URL。注意这一步实际上还不能获取任何东西。 |
| [3] | 第二步是创建一个 URL 开启器 (opener)。它可以接受任何数量的处理器来控制响应的处理。但你也可以创建一个没有任何自定义处理器的开启器，在这儿你就是这么做的。你将在本章后面探究重定向的部分看到如何定义和使用自定义处理器的内容。 |
| [4] | 最后一个步骤是，使用你创建的 `Request` 对象告诉开启器打开 URL。因为你能从获得的信息中看到所有调试信息，这个步骤实际上获得了资源并且把返回数据存储在了 `feeddata` 中。 |

## 例 11.5\. 给 `Request` 添加头信息

```py
>>> request                                                
<urllib2.Request instance at 0x00250AA8>
>>> request.get_full_url()
http://diveintomark.org/xml/atom.xml
>>> request.add_header('User-Agent',
`...` 'OpenAnything/1.0 +http://diveintopython.org/')    
>>> feeddata = opener.open(request).read()                 
connect: (diveintomark.org, 80)
send: '
GET /xml/atom.xml HTTP/1.0
Host: diveintomark.org
User-agent: OpenAnything/1.0 +http://diveintopython.org/   
'
reply: 'HTTP/1.1 200 OK\r\n'
header: Date: Wed, 14 Apr 2004 23:45:17 GMT
header: Server: Apache/2.0.49 (Debian GNU/Linux)
header: Content-Type: application/atom+xml
header: Last-Modified: Wed, 14 Apr 2004 22:14:38 GMT
header: ETag: "e8284-68e0-4de30f80"
header: Accept-Ranges: bytes
header: Content-Length: 26848
header: Connection: close 
```

|  |  |
| --- | --- |
| [1] | 继续前面的例子；你已经用你要访问的 URL 创建了 `Request` 。 |
| [2] | 使用`Request` 对象的 `add_header` 方法，你能向请求中添加任意的 HTTP 头信息。第一个参数是头信息，第二个参数是头信息的值。`User-Agent` 的约定格式是：应用名，跟一个斜线，跟版本号。剩下的是自由的格式，你将看到许多疯狂的变化，但通常这里应该包含你的应用的 URL。和你的请求的其他信息一样，`User-Agent` 会被服务器纪录下来，其中包含你的应用的 URL。如果发生错误，服务器管理员就能通过查看他们的访问日志与你联系。 |
| [3] | 之前你创建的`opener` 对象也可以再生，且它将再次获得相同的 feed，但这次使用了你自定义的 `User-Agent` 头信息。 |
| [4] | 这就是你发送的自定义的 `User-Agent`，代替了 Python 默认发送的一般的 `User-Agent`。若你继续看，会注意到你定义的是 `User-Agent` 头信息，但实际上发送的是 `User-agent` 头信息。看看有何不同？`urllib2` 改变了大小写所以只有首字母是大写的。这没问题，因为 HTTP 规定头信息的字段名是大小写无关的。 |

# 11.6\. 处理 `Last-Modified` 和 `ETag`

# 11.6\. 处理 `Last-Modified` 和 `ETag`

既然你知道如何在你的 web 服务请求中添加自定义的 HTTP 头信息，接下来看看如何添加 `Last-Modified` 和 `ETag` 头信息的支持。

下面的这些例子将以调试标记置为关闭的状态来显示输出结果。如果你还停留在上一部分的开启状态，可以使用 `httplib.HTTPConnection.debuglevel = 0` 将其设置为关闭状态。或者，如果你认为有帮助也可以保持为开启状态。

## 例 11.6\. 测试 `Last-Modified`

```py
>>> import urllib2
>>> request = urllib2.Request('http://diveintomark.org/xml/atom.xml')
>>> opener = urllib2.build_opener()
>>> firstdatastream = opener.open(request)
>>> firstdatastream.headers.dict                       
{'date': 'Thu, 15 Apr 2004 20:42:41 GMT', 
 'server': 'Apache/2.0.49 (Debian GNU/Linux)', 
 'content-type': 'application/atom+xml',
 'last-modified': 'Thu, 15 Apr 2004 19:45:21 GMT', 
 'etag': '"e842a-3e53-55d97640"',
 'content-length': '15955', 
 'accept-ranges': 'bytes', 
 'connection': 'close'}
>>> request.add_header('If-Modified-Since',
... firstdatastream.headers.get('Last-Modified'))  
>>> seconddatastream = opener.open(request)            
Traceback (most recent call last):
  File "<stdin>", line 1, in ?
  File "c:\python23\lib\urllib2.py", line 326, in open
    '_open', req)
  File "c:\python23\lib\urllib2.py", line 306, in _call_chain
    result = func(*args)
  File "c:\python23\lib\urllib2.py", line 901, in http_open
    return self.do_open(httplib.HTTP, req)
  File "c:\python23\lib\urllib2.py", line 895, in do_open
    return self.parent.error('http', req, fp, code, msg, hdrs)
  File "c:\python23\lib\urllib2.py", line 352, in error
    return self._call_chain(*args)
  File "c:\python23\lib\urllib2.py", line 306, in _call_chain
    result = func(*args)
  File "c:\python23\lib\urllib2.py", line 412, in http_error_default
    raise HTTPError(req.get_full_url(), code, msg, hdrs, fp)
urllib2.HTTPError: HTTP Error 304: Not Modified 
```

|  |  |
| --- | --- |
| [1] | 还记得当调试标记设置为开启时所有那些你看到的 HTTP 头信息打印输出吗？ 这里便是用编程方式访问它们的方法： `firstdatastream.headers` 是一个类似 dictionary 行为的对象并且允许你获得任何个别的从 HTTP 服务器返回的头信息。 |
| [2] | 在第二次请求时，你用第一次请求获得的最近修改时间添加了 `If-Modified-Since` 头信息。如果数据没被改变，服务器应该返回一个 `304` 状态代码。 |
| [3] | 毫无疑问，数据没被改变。你可以从跟踪返回结果看到 `urllib2` 抛出了一个特殊异常，`HTTPError`，以响应 `304` 状态代码。这有点不寻常，并且完全没有任何帮助。毕竟，它不是个错误；你明确地询问服务器如果没有变化就不要发送任何数据，并且数据没有变化，所以服务器告诉你它没有为你发送任何数据。那不是个错误；实际上也正是你所期望的。 |

`urllib2` 也为你认为是错误的其他条件引发 `HTTPError` 异常，比如 `404` (page not found)。实际上，它将为*任何* 除了状态代码 `200` (OK)、`301` (permanent redirect)或 `302` (temporary redirect) 之外的状态引发 `HTTPError`。捕获状态代码并简单返回它，而不是抛出异常，这应该对你很有帮助。为了实现它，你将需要自定义一个 URL 处理器。

## 例 11.7\. 定义 URL 处理器

这个自定义的 URL 处理器是 `openanything.py` 的一部分。

```py
 class DefaultErrorHandler(urllib2.HTTPDefaultErrorHandler):    
    def http_error_default(self, req, fp, code, msg, headers): 
        result = urllib2.HTTPError(                           
            req.get_full_url(), code, msg, headers, fp)       
        result.status = code                                   
        return result 
```

|  |  |
| --- | --- |
| [1] | `urllib2` 是围绕 URL 处理器而设计的。每一个处理器就是一个能定义任意数量方法的类。当某事件发生时——比如一个 HTTP 错误，甚至是 `304` 代码——`urllib2` 审视用于处理它的 一系列已定义的处理器方法。在此要用到自省，与 第九章 *XML 处理*中为不同节点类型定义不同处理器类似。但是 `urllib2` 是很灵活的，还可以内省为当前请求所定义的所有处理器。 |
| [2] | 当从服务器接收到一个 `304` 状态代码时，`urllib2` 查找定义的操作并调用 `http_error_default` 方法。通过定义一个自定义的错误处理，你可以阻止 `urllib2` 引发异常。取而代之的是，你创建 `HTTPError` 对象，返回它而不是引发异常。 |
| [3] | 这是关键部分：返回之前，你保存从 HTTP 服务器返回的状态代码。这将使你从主调程序轻而易举地访问它。 |

## 例 11.8\. 使用自定义 URL 处理器

```py
>>> request.headers                           
{'If-modified-since': 'Thu, 15 Apr 2004 19:45:21 GMT'}
>>> import openanything
>>> opener = urllib2.build_opener(
... openanything.DefaultErrorHandler())   
>>> seconddatastream = opener.open(request)
>>> seconddatastream.status                   
304
>>> seconddatastream.read()                   
'' 
```

|  |  |
| --- | --- |
| [1] | 继续前面的例子，`Request` 对象已经被设置，并且你已经添加了 `If-Modified-Since` 头信息。 |
| [2] | 这是关键所在：既然已经定义了你的自定义 URL 处理器，你需要告诉 `urllib2` 来使用它。还记得我怎么说 `urllib2` 将一个 HTTP 资源的访问过程分解为三个步骤的正当理由吗？这便是为什么构建 HTTP 开启器是其步骤之一，因为你能用你自定义的 URL 操作覆盖 `urllib2` 的默认行为来创建它。 |
| [3] | 现在你可以快速地打开一个资源，返回给你的对象既包括常规头信息 (使用 `seconddatastream.headers.dict` 访问它们)，也包括 HTTP 状态代码。在此，正如你所期望的，状态代码是 `304`，意味着此数据自从上次请求后没有被修改。 |
| [4] | 注意当服务器返回 `304` 状态代码时，并没有重新发送数据。这就是全部的关键：没有重新下载未修改的数据，从而节省了带宽。因此若你确实想要那个数据，你需要在首次获得它时在本地缓存数据。 |

处理 `ETag` 的工作也非常相似，只不过不是检查 `Last-Modified` 并发送 `If-Modified-Since`，而是检查 `ETag` 并发送 `If-None-Match`。让我们打开一个新的 IDE 会话。

## 例 11.9\. 支持 `ETag`/`If-None-Match`

```py
>>> import urllib2, openanything
>>> request = urllib2.Request('http://diveintomark.org/xml/atom.xml')
>>> opener = urllib2.build_opener(
... openanything.DefaultErrorHandler())
>>> firstdatastream = opener.open(request)
>>> firstdatastream.headers.get('ETag')        
'"e842a-3e53-55d97640"'
>>> firstdata = firstdatastream.read()
>>> print firstdata                            
<?xml version="1.0" encoding="iso-8859-1"?>
<feed version="0.3"

  xml:lang="en">
  <title mode="escaped">dive into mark</title>
  <link rel="alternate" type="text/html" href="http://diveintomark.org/"/>
  <-- rest of feed omitted for brevity -->
>>> request.add_header('If-None-Match',
... firstdatastream.headers.get('ETag'))   
>>> seconddatastream = opener.open(request)
>>> seconddatastream.status                    
304
>>> seconddatastream.read()                    
'' 
```

|  |  |
| --- | --- |
| [1] | 使用 `firstdatastream.headers` 伪字典，你可以获得从服务器返回的 `ETag` (如果服务器没有返回 `ETag` 会发生什么？答案是，这一行代码将返回 `None`。) |
| [2] | OK，你获得了数据。 |
| [3] | 现在进行第二次调用，将 `If-None-Match` 头信息设置为你第一次调用获得的 `ETag`。 |
| [4] | 第二次调用静静地成功了 (没有出现任何的异常)，并且你又一次看到了从服务器返回的 `304` 状态代码。你第二次基于 `ETag` 发送请求，服务器知道数据没有被改变。 |
| [5] | 无论 `304` 是被 `Last-Modified` 数据检查还是 `ETag` hash 匹配触发的，获得 `304` 的同时都不会下载数据。这就是重点所在。 |

> 注意
> 在这些例子中，HTTP 服务器同时支持 `Last-Modified` 和 `ETag` 头信息，但并非所有的服务器皆如此。作为一个 web 服务的客户端，你应该为支持两种头信息做准备，但是你的程序也应该为服务器仅支持其中一种头信息或两种头信息都不支持而做准备。

# 11.7\. 处理重定向

# 11.7\. 处理重定向

你可以使用两种不同的自定义 URL 处理器来处理永久重定向和临时重定向。

首先，让我们来看看重定向处理的必要性。

## 例 11.10\. 没有重定向处理的情况下，访问 web 服务

```py
>>> import urllib2, httplib
>>> httplib.HTTPConnection.debuglevel = 1           
>>> request = urllib2.Request(
... 'http://diveintomark.org/redir/example301.xml') 
>>> opener = urllib2.build_opener()
>>> f = opener.open(request)
connect: (diveintomark.org, 80)
send: '
GET /redir/example301.xml HTTP/1.0
Host: diveintomark.org
User-agent: Python-urllib/2.1
'
reply: 'HTTP/1.1 301 Moved Permanently\r\n'             
header: Date: Thu, 15 Apr 2004 22:06:25 GMT
header: Server: Apache/2.0.49 (Debian GNU/Linux)
header: Location: http://diveintomark.org/xml/atom.xml  
header: Content-Length: 338
header: Connection: close
header: Content-Type: text/html; charset=iso-8859-1
connect: (diveintomark.org, 80)
send: '
GET /xml/atom.xml HTTP/1.0                              
Host: diveintomark.org
User-agent: Python-urllib/2.1
'
reply: 'HTTP/1.1 200 OK\r\n'
header: Date: Thu, 15 Apr 2004 22:06:25 GMT
header: Server: Apache/2.0.49 (Debian GNU/Linux)
header: Last-Modified: Thu, 15 Apr 2004 19:45:21 GMT
header: ETag: "e842a-3e53-55d97640"
header: Accept-Ranges: bytes
header: Content-Length: 15955
header: Connection: close
header: Content-Type: application/atom+xml
>>> f.url                                               
'http://diveintomark.org/xml/atom.xml'
>>> f.headers.dict
{'content-length': '15955', 
'accept-ranges': 'bytes', 
'server': 'Apache/2.0.49 (Debian GNU/Linux)', 
'last-modified': 'Thu, 15 Apr 2004 19:45:21 GMT', 
'connection': 'close', 
'etag': '"e842a-3e53-55d97640"', 
'date': 'Thu, 15 Apr 2004 22:06:25 GMT', 
'content-type': 'application/atom+xml'}
>>> f.status
Traceback (most recent call last):
  File "<stdin>", line 1, in ?
AttributeError: addinfourl instance has no attribute 'status' 
```

|  |  |
| --- | --- |
| [1] | 你最好开启调试状态，看看发生了什么。 |
| [2] | 这是一个我已经设置了永久重定向到我的 Atom feed `http://diveintomark.org/xml/atom.xml` 的 URL。 |
| [3] | 毫无疑问，当你试图从那个地址下载数据时，服务器会返回 `301` 状态代码，告诉你你访问的资源已经被永久移动了。 |
| [4] | 服务器同时返回 `Location:` 头信息，它给出了这个数据的新地址。 |
| [5] | `urllib2` 注意到了重定向状态代码并会自动从`Location:` 头信息中给出的新地址获取数据。 |
| [6] | 从 `opener` 返回的对象包括新的永久地址和第二次请求获得的所有头信息 (从一个新的永久地址获得)。但是状态代码不见了，因此你无从知晓重定向到底是永久重定向还是临时重定向。这是至关重要的：如果这是临时重定向，那么你应该继续使用旧地址访问数据。但是如果是永久重定向 (正如本例)，你应该从现在起使用新地址访问数据。 |

这不太理想，但很容易改进。实际上当 `urllib2` 遇到 `301` 或 `302` 时的行为并不是我们所期望的，所以让我们来覆盖这些行为。如何实现呢？用一个自定义的处理器，正如你处理 `304` 代码所做的。

## 例 11.11\. 定义重定向处理器

这个类定义在 `openanything.py`。

```py
 class SmartRedirectHandler(urllib2.HTTPRedirectHandler):     
    def http_error_301(self, req, fp, code, msg, headers):  
        result = urllib2.HTTPRedirectHandler.http_error_301( 
            self, req, fp, code, msg, headers)              
        result.status = code                                 
        return result                                       
    def http_error_302(self, req, fp, code, msg, headers):   
        result = urllib2.HTTPRedirectHandler.http_error_302(
            self, req, fp, code, msg, headers)              
        result.status = code                                
        return result 
```

|  |  |
| --- | --- |
| [1] | 重定向行为定义在 `urllib2` 的一个叫做 `HTTPRedirectHandler` 的类中。我们不想完全地覆盖这些行为，只想做点扩展，所以我们子类化 `HTTPRedirectHandler`，从而我们仍然可以调用祖先类来实现所有原来的功能。 |
| [2] | 当从服务器获得 `301` 状态代码，`urllib2` 将搜索处理器并调用 `http_error_301` 方法。我们首先要做的就是在祖先中调用 `http_error_301` 方法，它将处理查找 `Location:` 头信息的工作并跟踪重定向到新地址。 |
| [3] | 这是关键：返回之前，你存储了状态代码 (`301`)，所以主调程序稍后就可以访问它了。 |
| [4] | 临时重定向 (状态代码 `302`) 以相同的方式工作：覆盖 `http_error_302` 方法，调用祖先，并在返回之前保存状态代码。 |

这将为我们带来什么？现在你可以用自定义重定向处理器构造一个的 URL 开启器，并且它依然能自动跟踪重定向，也能展示出重定向状态代码。

## 例 11.12\. 使用重定向处理器检查永久重定向

```py
>>> request = urllib2.Request('http://diveintomark.org/redir/example301.xml')
>>> import openanything, httplib
>>> httplib.HTTPConnection.debuglevel = 1
>>> opener = urllib2.build_opener(
... openanything.SmartRedirectHandler())           
>>> f = opener.open(request)
connect: (diveintomark.org, 80)
send: 'GET /redir/example301.xml HTTP/1.0
Host: diveintomark.org
User-agent: Python-urllib/2.1
'
reply: 'HTTP/1.1 301 Moved Permanently\r\n'            
header: Date: Thu, 15 Apr 2004 22:13:21 GMT
header: Server: Apache/2.0.49 (Debian GNU/Linux)
header: Location: http://diveintomark.org/xml/atom.xml
header: Content-Length: 338
header: Connection: close
header: Content-Type: text/html; charset=iso-8859-1
connect: (diveintomark.org, 80)
send: '
GET /xml/atom.xml HTTP/1.0
Host: diveintomark.org
User-agent: Python-urllib/2.1
'
reply: 'HTTP/1.1 200 OK\r\n'
header: Date: Thu, 15 Apr 2004 22:13:21 GMT
header: Server: Apache/2.0.49 (Debian GNU/Linux)
header: Last-Modified: Thu, 15 Apr 2004 19:45:21 GMT
header: ETag: "e842a-3e53-55d97640"
header: Accept-Ranges: bytes
header: Content-Length: 15955
header: Connection: close
header: Content-Type: application/atom+xml 
>>> f.status                                           
301
>>> f.url
'http://diveintomark.org/xml/atom.xml' 
```

|  |  |
| --- | --- |
| [1] | 首先，用刚刚定义的重定向处理器创建一个 URL 开启器。 |
| [2] | 你发送了一个请求，并在响应中获得了 `301` 状态代码。 如此一来，`http_error_301` 方法就被调用了。你调用了祖先类，跟踪了重定向并且发送了一个新地址 (`http://diveintomark.org/xml/atom.xml`) 请求。 |
| [3] | 这是决定性的一步：现在，你不仅做到了访问一个新 URL，而且获得了重定向的状态代码，所以你可以断定这是一个永久重定向。下一次你请求这个数据时，就应该使用 `f.url` 指定的新地址 (`http://diveintomark.org/xml/atom.xml`)。如果你已经在配置文件或数据库中存储了这个地址，就需要更新旧地址而不是反复地使用旧地址请求服务。现在是更新你的地址簿的时候了。 |

同样的重定向处理也可以告诉你*不该* 更新你的地址簿。

## 例 11.13\. 使用重定向处理器检查临时重定向

```py
>>> request = urllib2.Request(
... 'http://diveintomark.org/redir/example302.xml')   
>>> f = opener.open(request)
connect: (diveintomark.org, 80)
send: '
GET /redir/example302.xml HTTP/1.0
Host: diveintomark.org
User-agent: Python-urllib/2.1
'
reply: 'HTTP/1.1 302 Found\r\n'                           
header: Date: Thu, 15 Apr 2004 22:18:21 GMT
header: Server: Apache/2.0.49 (Debian GNU/Linux)
header: Location: http://diveintomark.org/xml/atom.xml
header: Content-Length: 314
header: Connection: close
header: Content-Type: text/html; charset=iso-8859-1
connect: (diveintomark.org, 80)
send: '
GET /xml/atom.xml HTTP/1.0                                
Host: diveintomark.org
User-agent: Python-urllib/2.1
'
reply: 'HTTP/1.1 200 OK\r\n'
header: Date: Thu, 15 Apr 2004 22:18:21 GMT
header: Server: Apache/2.0.49 (Debian GNU/Linux)
header: Last-Modified: Thu, 15 Apr 2004 19:45:21 GMT
header: ETag: "e842a-3e53-55d97640"
header: Accept-Ranges: bytes
header: Content-Length: 15955
header: Connection: close
header: Content-Type: application/atom+xml
>>> f.status                                              
302
>>> f.url
http://diveintomark.org/xml/atom.xml 
```

|  |  |
| --- | --- |
| [1] | 这是一个 URL，我已经设置了它，让它告诉客户端*临时* 重定向到 `http://diveintomark.org/xml/atom.xml`。 |
| [2] | 服务器返回 `302` 状态代码，标识出一个临时重定向。数据的临时新地址在 `Location:` 头信息中给出。 |
| [3] | `urllib2` 调用你的 `http_error_302` 方法，它调用了 `urllib2.HTTPRedirectHandler` 中的同名的祖先方法，跟踪重定向到一个新地址。然后你的 `http_error_302` 方法存储状态代码 (`302`) 使主调程序在稍后可以获得它。 |
| [4] | 此时，已经成功追踪重定向到 `http://diveintomark.org/xml/atom.xml`。`f.status` 告诉你这是一个临时重定向，这意味着你应该继续使用原来的地址 (`http://diveintomark.org/redir/example302.xml`) 请求数据。也许下一次它仍然被重定向，也许不会。也许会重定向到不同的地址。这也不好说。服务器说这个重定向仅仅是临时的，你应该尊重它。并且现在你获得了能使主调程序尊重它的充分信息。 |

# 11.8\. 处理压缩数据

# 11.8\. 处理压缩数据

你要支持的最后一个重要的 HTTP 特性是压缩。许多 web 服务具有发送压缩数据的能力，这可以将网络线路上传输的大量数据消减 60% 以上。这尤其适用于 XML web 服务，因为 XML 数据 的压缩率可以很高。

服务器不会为你发送压缩数据，除非你告诉服务器你可以处理压缩数据。

## 例 11.14\. 告诉服务器你想获得压缩数据

```py
>>> import urllib2, httplib
>>> httplib.HTTPConnection.debuglevel = 1
>>> request = urllib2.Request('http://diveintomark.org/xml/atom.xml')
>>> request.add_header('Accept-encoding', 'gzip')        
>>> opener = urllib2.build_opener()
>>> f = opener.open(request)
connect: (diveintomark.org, 80)
send: '
GET /xml/atom.xml HTTP/1.0
Host: diveintomark.org
User-agent: Python-urllib/2.1
Accept-encoding: gzip                                    
'
reply: 'HTTP/1.1 200 OK\r\n'
header: Date: Thu, 15 Apr 2004 22:24:39 GMT
header: Server: Apache/2.0.49 (Debian GNU/Linux)
header: Last-Modified: Thu, 15 Apr 2004 19:45:21 GMT
header: ETag: "e842a-3e53-55d97640"
header: Accept-Ranges: bytes
header: Vary: Accept-Encoding
header: Content-Encoding: gzip                           
header: Content-Length: 6289                             
header: Connection: close
header: Content-Type: application/atom+xml 
```

|  |  |
| --- | --- |
| [1] | 这是关键：一创建了 `Request` 对象，就添加一个 `Accept-encoding` 头信息告诉服务器你能接受 gzip 压缩数据。`gzip` 是你使用的压缩算法的名称。理论上你可以使用其它的压缩算法，但是 `gzip` 是 web 服务器上使用率高达 99% 的一种。 |
| [2] | 这是你的头信息传越网络线路的过程。 |
| [3] | 这是服务器的返回信息：`Content-Encoding: gzip` 头信息意味着你要回得的数据已经被 gzip 压缩了。 |
| [4] | `Content-Length` 头信息是已压缩数据的长度，并非解压缩数据的长度。一会儿你会看到，实际的解压缩数据长度为 15955，因此 gzip 压缩节省了 60% 以上的网络带宽！ |

## 例 11.15\. 解压缩数据

```py
>>> compresseddata = f.read()                              
>>> len(compresseddata)
6289
>>> import StringIO
>>> compressedstream = StringIO.StringIO(compresseddata)   
>>> import gzip
>>> gzipper = gzip.GzipFile(fileobj=compressedstream)      
>>> data = gzipper.read()                                  
>>> print data                                             
<?xml version="1.0" encoding="iso-8859-1"?>
<feed version="0.3"

  xml:lang="en">
  <title mode="escaped">dive into mark</title>
  <link rel="alternate" type="text/html" href="http://diveintomark.org/"/>
  <-- rest of feed omitted for brevity -->
>>> len(data)
15955 
```

|  |  |
| --- | --- |
| [1] | 继续上面的例子，`f` 是一个从 URL 开启器返回的类文件对象。使用它的 `read()` 方法将正常地获得非压缩数据，但是因为这个数据已经被 gzip 压缩过，所以这只是获得你想要的最终数据的第一步。 |
| [2] | 好吧，只是先得有点儿凌乱的步骤。Python 有一个 `gzip` 模块，它能读取 (当然也能写入) 磁盘上的 gzip 压缩文件。但是磁盘上还没有文件，只在内存里有一个 gzip 压缩缓冲区，并且你不想仅仅为了解压缩而写出一个临时文件。那么怎么做来从内存数据 (`compresseddata`) 创建类文件对象呢？这需要使用 `StringIO` 模块。你首次看到 `StringIO` 模块是在上一章，但现在你会发现它的另一种用法。 |
| [3] | 现在你可以创建 `GzipFile` 的一个实例，并且告诉它其中的 “文件” 是一个类文件对象 `compressedstream`。 |
| [4] | 这是做所有工作的一行：从 `GzipFile` 中 “读取” 将会解压缩数据。感到奇妙吗？是的，它确实解压缩了数据。`gzipper` 是一个类文件对象，它代表一个 gzip 压缩文件。尽管这个 “文件” 并非一个磁盘上的真实文件；但 `gzipper` 还是从你用 `StringIO` 包装了压缩数据的类文件对象中 “读取” 数据，而它仅仅是内存中的变量 `compresseddata`。压缩的数据来自哪呢？最初你从远程 HTTP 服务器下载它，通过从用 `urllib2.build_opener` 创建的类文件对象中 “读取”。令人吃惊吧，这就是所有的步骤。链条上的每一步都完全不知道上一步在造假。 |
| [5] | 看看吧，实际的数据 (实际为 15955 bytes)。 |

“等等!” 我听见你在叫。“还能更简单吗！” 我知道你在想什么。你在，既然 `opener.open` 返回一个类文件对象，那么为什么不抛弃中间件 `StringIO` 而通过 `f` 直接访问 `GzipFile` 呢？OK，或许你没想到，但是别为此担心，因为那样无法工作。

## 例 11.16\. 从服务器直接解压缩数据

```py
>>> f = opener.open(request)                  
>>> f.headers.get('Content-Encoding')         
'gzip'
>>> data = gzip.GzipFile(fileobj=f).read()    
Traceback (most recent call last):
  File "<stdin>", line 1, in ?
  File "c:\python23\lib\gzip.py", line 217, in read
    self._read(readsize)
  File "c:\python23\lib\gzip.py", line 252, in _read
    pos = self.fileobj.tell()   # Save current position
AttributeError: addinfourl instance has no attribute 'tell' 
```

|  |  |
| --- | --- |
| [1] | 继续前面的例子，你已经有一个设置了 `Accept-encoding: gzip` 头信息的 `Request` 对象。 |
| [2] | 简单地打开请求将获得你的头信息 (虽然还没下载任何数据)。正如你从 `Content-Encoding` 头信息所看到的，这个数据被要求用 gzip 压缩发送。 |
| [3] | 从 `opener.open` 返回了一个类文件对象，从头信息中你可以获知，你将获得 gzip 压缩数据。为什么不简单地通过那个类文件对象直接访问 `GzipFile` 呢？当你从 `GzipFile` 实例 “读取” 时，它将从远程 HTTP 服务器 “读取” 被压缩的数据并且立即解压缩。这是个好主意，但是不行。由 gzip 压缩的工作方式所致，`GzipFile` 需要存储其位置并在压缩文件上往返游走。当 “文件” 是来自远程服务器的字节流时无法工作；你能用它做的所有工作就是一次返回一个字节流，而不是在字节流上往返。所以使用 `StringIO` 这种看上去不太优雅的手段是最好的解决方案：下载压缩的数据，用 `StringIO` 创建一个类文件对象，并从中解压缩数据。 |

# 11.9\. 全部放在一起

# 11.9\. 全部放在一起

你已经看到了构造一个智能的 HTTP web 客户端的所有片断。现在让我们看看如何将它们整合到一起。

## 例 11.17\. `openanything` 函数

这个函数定义在 `openanything.py` 中。

```py
 def openAnything(source, etag=None, lastmodified=None, agent=USER_AGENT):
    # non-HTTP code omitted for brevity
    if urlparse.urlparse(source)[0] == 'http':                                       
        # open URL with urllib2 
        request = urllib2.Request(source)                                           
        request.add_header('User-Agent', agent)                                      
        if etag:                                                                    
            request.add_header('If-None-Match', etag)                                
        if lastmodified:                                                            
            request.add_header('If-Modified-Since', lastmodified)                    
        request.add_header('Accept-encoding', 'gzip')                                
        opener = urllib2.build_opener(SmartRedirectHandler(), DefaultErrorHandler()) 
        return opener.open(request) 
```

|  |  |
| --- | --- |
| [1] | `urlparse` 是一个解析 URL 的便捷的工具模块。它的主要函数也叫 `urlparse`，接受一个 URL 并将其拆分为 tuple (scheme (协议), domain (域名), path (路径), params (参数), query string parameters (请求字符串参数), fragment identifier (片段效验符))。当然，你唯一需要注意的就是 scheme，确认你处理的是一个 HTTP URL (`urllib2` 才能处理)。 |
| [2] | 通过调用函数使用 `User-Agent` 向 HTTP 服务器确定你的身份。如果没有 `User-Agent` 被指定，你会使用一个默认的，就是定义在早期的 `openanything.py` 模块中的那个。你从来不会使用到默认的定义在 `urllib2` 中的那个。 |
| [3] | 如果给出了 `ETag`，要在 `If-None-Match` 头信息中发送它。 |
| [4] | 如果给出了最近修改日期，要在 `If-Modified-Since` 头信息中发送它。 |
| [5] | 如果可能要告诉服务器你要获取压缩数据。 |
| [6] | 使用*两个* 自定义 URL 处理器创建一个 URL 开启器：`SmartRedirectHandler` 终于处理 `301` 和 `302` 重定向，而 `DefaultErrorHandler` 用于处理 `304`, `404` 以及其它的错误条件。 |
| [7] | 就是这样！打开 URL 并返回一个类文件对象给调用者。 |

## 例 11.18\. `fetch` 函数

这个函数定义在 `openanything.py` 中。

```py
 def fetch(source, etag=None, last_modified=None, agent=USER_AGENT):  
    '''Fetch data and metadata from a URL, file, stream, or string'''
    result = {}                                                      
    f = openAnything(source, etag, last_modified, agent)              
    result['data'] = f.read()                                         
    if hasattr(f, 'headers'):                                        
        # save ETag, if the server sent one 
        result['etag'] = f.headers.get('ETag')                        
        # save Last-Modified header, if the server sent one 
        result['lastmodified'] = f.headers.get('Last-Modified')       
        if f.headers.get('content-encoding', '') == 'gzip':           
            # data came back gzip-compressed, decompress it 
            result['data'] = gzip.GzipFile(fileobj=StringIO(result['data']])).read()
    if hasattr(f, 'url'):                                             
        result['url'] = f.url                                        
        result['status'] = 200                                       
    if hasattr(f, 'status'):                                          
        result['status'] = f.status                                  
    f.close()                                                        
    return result 
```

|  |  |
| --- | --- |
| [1] | 首先，你用 URL、`ETag` hash、`Last-Modified` 日期和 `User-Agent` 调用 `openAnything` 函数。 |
| [2] | 读取从服务器返回的真实数据。这可能是被压缩的；如果是，将在后面进行解压缩。 |
| [3] | 保存从服务器返回的 `ETag` hash，这样主调程序下一次就能把它传递给你，然后再传递给 `openAnything`，放到 `If-None-Match` 头信息里发送给远程服务器。 |
| [4] | 也要保存 `Last-Modified` 数据。 |
| [5] | 如果服务器说它发送的是压缩数据，就执行解压缩。 |
| [6] | 如果你的服务器返回一个 URL 就保存它，并在查明之前假定状态代码为 `200`。 |
| [7] | 如果其中一个自定义 URL 处理器捕获了一个状态代码，也要保存下来。 |

## 例 11.19\. 使用 `openanything.py`

```py
>>> import openanything
>>> useragent = 'MyHTTPWebServicesApp/1.0'
>>> url = 'http://diveintopython.org/redir/example301.xml'
>>> params = openanything.fetch(url, agent=useragent)              
>>> params                                                         
{'url': 'http://diveintomark.org/xml/atom.xml', 
'lastmodified': 'Thu, 15 Apr 2004 19:45:21 GMT', 
'etag': '"e842a-3e53-55d97640"', 
'status': 301,
'data': '<?xml version="1.0" encoding="iso-8859-1"?>
<feed version="0.3"
<-- rest of data omitted for brevity -->'}
>>> if params['status'] == 301:                                    
... url = params['url']
>>> newparams = openanything.fetch(
... url, params['etag'], params['lastmodified'], useragent)    
>>> newparams
{'url': 'http://diveintomark.org/xml/atom.xml', 
'lastmodified': None, 
'etag': '"e842a-3e53-55d97640"', 
'status': 304,
'data': ''} 
```

|  |  |
| --- | --- |
| [1] | 第一次获取资源时，你没有 `ETag` hash 或 `Last-Modified` 日期，所以你不用使用这些参数。 (它们是可选参数。) |
| [2] | 你获得了一个 dictionary，它包括几个有用的头信息、HTTP 状态代码和从服务器返回的真实数据。`openanything` 在内部处理 gzip 压缩；在本级别上你不必关心它。 |
| [3] | 如果你得到一个 `301` 状态代码，表示是个永久重定向，你需要把你的 URL 更新为新地址。 |
| [4] | 第二次获取相同的资源时，你已经从以往获得了各种信息：URL (可能被更新了)、从上一次访问获得的 `ETag`、从上一次访问获得的 `Last-Modified` 日期，当然还有 `User-Agent`。 |
| [5] | 你重新获取了这个 dictionary，但是数据没有改变，所以你得到了一个 `304` 状态代码而没有数据。 |

# 11.10\. 小结

# 11.10\. 小结

`openanything.py` 及其函数现在可以完美地工作了。

每个客户端都应该支持 HTTP web 服务的以下 5 个重要特性：

*   通过设置适当的 `User-Agent` 识别你的应用。
*   适当地处理永久重定向。
*   支持 `Last-Modified` 日期检查从而避免在数据未改变的情况下重新下载数据。
*   支持 `ETag` hash 从而避免在数据未改变的情况下重新下载数据。
*   支持 gzip 压缩从而在数据*已经* 改变的情况下尽可能地减少传输带宽。