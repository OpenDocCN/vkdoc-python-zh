# 附录 D. 示例清单

# 附录 D. 示例清单

第一章 安装 Python

*   1.3\. Mac OS X 上的 Python
    *   例 1.1\. 两个 Python 版本
*   1.5\. RedHat Linux 上的 Python
    *   例 1.2\. 在 RedHat Linux 9 上安装
*   1.6\. Debian GNU/Linux 上的 Python
    *   例 1.3\. 在 Debian GNU/Linux 上安装
*   1.7\. 从源代码安装 Python
    *   例 1.4\. 从源代码安装
*   1.8\. 使用 Python 的交互 Shell
    *   例 1.5\. 初次使用交互 Shell

第二章 第一个 Python 程序

*   2.1\. 概览
    *   例 2.1\. odbchelper.py
*   2.3\. 文档化函数
    *   例 2.2\. 定义 buildConnectionString 函数的 doc string
*   2.4\. 万物皆对象
    *   例 2.3\. 访问 buildConnectionString 函数的 doc string
*   2.4.1\. 模块导入的搜索路径
    *   例 2.4\. 模块导入的搜索路径
*   2.5\. 代码缩进
    *   例 2.5\. 缩进 buildConnectionString 函数
    *   例 2.6\. if 语句

第三章 内置数据类型

*   3.1.1\. Dictionary 的定义
    *   例 3.1\. 定义 Dictionary
*   3.1.2\. Dictionary 的修改
    *   例 3.2\. 修改 Dictionary
    *   例 3.3\. Dictionary 的 key 是大小写敏感的
    *   例 3.4\. 在 dictionary 中混用数据类型
*   3.1.3\. 从 dictionary 中删除元素
    *   例 3.5\. 从 dictionary 中删除元素
*   3.2.1\. List 的定义
    *   例 3.6\. 定义 List
    *   例 3.7\. 负的 list 索引
    *   例 3.8\. list 的分片 (slice)
    *   例 3.9\. Slice 简写
*   3.2.2\. 向 list 中增加元素
    *   例 3.10\. 向 list 中增加元素
    *   例 3.11\. extend (扩展) 与 append (追加) 的差别
*   3.2.3\. 在 list 中搜索
    *   例 3.12\. 搜索 list
*   3.2.4\. 从 list 中删除元素
    *   例 3.13\. 从 list 中删除元素
*   3.2.5\. 使用 list 的运算符
    *   例 3.14\. List 运算符
*   3.3\. Tuple 介绍
    *   例 3.15\. 定义 tuple
    *   例 3.16\. Tuple 没有方法
*   3.4\. 变量声明
    *   例 3.17\. 定义 myParams 变量
*   3.4.1\. 变量引用
    *   例 3.18\. 引用未赋值的变量
*   3.4.2\. 一次赋多值
    *   例 3.19\. 一次赋多值
    *   例 3.20\. 连续值赋值
*   3.5\. 格式化字符串
    *   例 3.21\. 字符串的格式化
    *   例 3.22\. 字符串格式化与字符串连接的比较
    *   例 3.23\. 数值的格式化
*   3.6\. 映射 list
    *   例 3.24\. List 解析介绍
    *   例 3.25\. keys, values 和 items 函数
    *   例 3.26\. buildConnectionString 中的 list 解析
*   3.7\. 连接 list 与分割字符串
    *   例 3.27\. odbchelper.py 的输出结果
    *   例 3.28\. 分割字符串

第四章 自省的威力

*   4.1\. 概览
    *   例 4.1\. apihelper.py
    *   例 4.2\. apihelper.py 的用法示例
    *   例 4.3\. apihelper.py 的高级用法
*   4.2\. 使用可选参数和命名参数
    *   例 4.4\. info 的有效调用
*   4.3.1\. type 函数
    *   例 4.5\. type 介绍
*   4.3.2\. str 函数
    *   例 4.6\. str 介绍
    *   例 4.7\. dir 介绍
    *   例 4.8\. callable 介绍
*   4.3.3\. 内置函数
    *   例 4.9\. 内置属性和内置函数
*   4.4\. 通过 getattr 获取对象引用
    *   例 4.10\. getattr 介绍
*   4.4.1\. 用于模块的 getattr
    *   例 4.11\. apihelper.py 中的 getattr 函数
*   4.4.2\. getattr 作为一个分发者
    *   例 4.12\. 使用 getattr 创建分发者
    *   例 4.13\. getattr 缺省值
*   4.5\. 过滤列表
    *   例 4.14\. 列表过滤介绍
*   4.6\. and 和 or 的特殊性质
    *   例 4.15\. and 介绍
    *   例 4.16\. or 介绍
*   4.6.1\. 使用 and-or 技巧
    *   例 4.17\. and-or 技巧介绍
    *   例 4.18\. and-or 技巧无效的场合
    *   例 4.19\. 安全使用 and-or 技巧
*   4.7\. 使用 lambda 函数
    *   例 4.20\. lambda 函数介绍
*   4.7.1\. 真实世界中的 lambda 函数
    *   例 4.21\. split 不带参数
*   4.8\. 全部放在一起
    *   例 4.22\. 动态得到 doc string
    *   例 4.23\. 为什么对一个 doc string 使用 str ？
    *   例 4.24\. ljust 方法介绍
    *   例 4.25\. 打印列表

第五章 对象和面向对象

*   5.1\. 概览
    *   例 5.1\. fileinfo.py
*   5.2\. 使用 from module import 导入模块
    *   例 5.2\. import module vs. from module import
*   5.3\. 类的定义
    *   例 5.3\. 最简单的 Python 类
    *   例 5.4\. 定义 FileInfo 类
*   5.3.1\. 初始化并开始类编码
    *   例 5.5\. 初始化 FileInfo 类
    *   例 5.6\. 编写 FileInfo 类
*   5.4\. 类的实例化
    *   例 5.7\. 创建 FileInfo 实例
*   5.4.1\. 垃圾回收
    *   例 5.8\. 尝试实现内存泄漏
*   5.5\. 探索 UserDict：一个封装类
    *   例 5.9\. 定义 UserDict 类
    *   例 5.10\. UserDict 常规方法
    *   例 5.11\. 直接继承自内建数据类型 dict
*   5.6.1\. 获得和设置数据项
    *   例 5.12\. **getitem** 专用方法
    *   例 5.13\. **setitem** 专用方法
    *   例 5.14\. 在 MP3FileInfo 中覆盖 **setitem**
    *   例 5.15\. 设置 MP3FileInfo 的 name
*   5.7\. 高级专用类方法
    *   例 5.16\. UserDict 中更多的专用方法
*   5.8\. 类属性介绍
    *   例 5.17\. 类属性介绍
    *   例 5.18\. 修改类属性
*   5.9\. 私有函数
    *   例 5.19\. 尝试调用一个私有方法

第六章 异常和文件处理

*   6.1\. 异常处理
    *   例 6.1\. 打开一个不存在的文件
*   6.1.1\. 为其他用途使用异常
    *   例 6.2\. 支持特定平台功能
*   6.2\. 与文件对象共事
    *   例 6.3\. 打开文件
*   6.2.1\. 读取文件
    *   例 6.4\. 读取文件
*   6.2.2\. 关闭文件
    *   例 6.5\. 关闭文件
*   6.2.3\. 处理 I/O 错误
    *   例 6.6\. MP3FileInfo 中的文件对象
*   6.2.4\. 写入文件
    *   例 6.7\. 写入文件
*   6.3\. for 循环
    *   例 6.8\. for 循环介绍
    *   例 6.9\. 简单计数
    *   例 6.10\. 遍历 dictionary
    *   例 6.11\. MP3FileInfo 中的 for 循环
*   6.4\. 使用 sys.modules
    *   例 6.12\. sys.modules 介绍
    *   例 6.13\. 使用 sys.modules
    *   例 6.14\. **module** 类属性
    *   例 6.15\. fileinfo.py 中的 sys.modules
*   6.5\. 与目录共事
    *   例 6.16\. 构造路径名
    *   例 6.17\. 分割路径名
    *   例 6.18\. 列出目录
    *   例 6.19\. 在 fileinfo.py 中列出目录
    *   例 6.20\. 使用 glob 列出目录
*   6.6\. 全部放在一起
    *   例 6.21\. listDirectory

第七章 正则表达式

*   7.2\. 个案研究：街道地址
    *   例 7.1\. 在字符串的结尾匹配
    *   例 7.2\. 匹配整个单词
*   7.3.1\. 校验千位数
    *   例 7.3\. 校验千位数
*   7.3.2\. 校验百位数
    *   例 7.4\. 检验百位数
*   7.4\. 使用 {n,m} 语法
    *   例 7.5\. 老方法：每一个字符都是可选的
    *   例 7.6\. 一个新的方法：从 n 到 m
*   7.4.1\. 校验十位数和个位数
    *   例 7.7\. 校验十位数
    *   例 7.8\. 用 {n,m} 语法确认罗马数字
*   7.5\. 松散正则表达式
    *   例 7.9\. 带有内联注释 (Inline Comments) 的正则表达式
*   7.6\. 个案研究：解析电话号码
    *   例 7.10\. 发现数字
    *   例 7.11\. 发现分机号
    *   例 7.12\. 处理不同分隔符
    *   例 7.13\. 处理没有分隔符的数字
    *   例 7.14\. 处理开始字符
    *   例 7.15\. 电话号码，无论何时我都要找到它
    *   例 7.16\. 解析电话号码 (最终版本)

第八章 HTML 处理

*   8.1\. 概览
    *   例 8.1\. BaseHTMLProcessor.py
    *   例 8.2\. dialect.py
    *   例 8.3\. dialect.py 的输出结果
*   8.2\. sgmllib.py 介绍
    *   例 8.4\. sgmllib.py 的样例测试
*   8.3\. 从 HTML 文档中提取数据
    *   例 8.5\. urllib 介绍
    *   例 8.6\. urllister.py 介绍
    *   例 8.7\. 使用 urllister.py
*   8.4\. BaseHTMLProcessor.py 介绍
    *   例 8.8\. BaseHTMLProcessor 介绍
    *   例 8.9\. BaseHTMLProcessor 输出结果
*   8.5\. locals 和 globals
    *   例 8.10\. locals 介绍
    *   例 8.11\. globals 介绍
    *   例 8.12\. locals 是只读的，globals 不是
*   8.6\. 基于 dictionary 的字符串格式化
    *   例 8.13\. 基于 dictionary 的字符串格式化介绍
    *   例 8.14\. BaseHTMLProcessor.py 中的基于 dictionary 的字符串格式化
    *   例 8.15\. 基于 dictionary 的字符串格式化的更多内容
*   8.7\. 给属性值加引号
    *   例 8.16\. 给属性值加引号
*   8.8\. dialect.py 介绍
    *   例 8.17\. 处理特别标记
    *   例 8.18\. SGMLParser
    *   例 8.19\. 覆盖 handle_data 方法
*   8.9\. 全部放在一起
    *   例 8.20\. translate 函数，第一部分
    *   例 8.21\. translate 函数，第二部分：奇妙而又奇妙
    *   例 8.22\. translate 函数，第三部分

第九章 XML 处理

*   9.1\. 概览
    *   例 9.1\. kgp.py
    *   例 9.2\. toolbox.py
    *   例 9.3\. kgp.py 的样例输出
    *   例 9.4\. kgp.py 的简单输出
*   9.2\. 包
    *   例 9.5\. 载入一个 XML 文档 (偷瞥一下)
    *   例 9.6\. 包的文件布局
    *   例 9.7\. 包也是模块
*   9.3\. XML 解析
    *   例 9.8\. 载入一个 XML 文档 (这次是真的)
    *   例 9.9\. 获取子节点
    *   例 9.10\. toxml 用于任何节点
    *   例 9.11\. 子节点可以是文本
    *   例 9.12\. 把文本挖出来
*   9.4\. Unicode
    *   例 9.13\. unicode 介绍
    *   例 9.14\. 存储非 ASCII 字符
    *   例 9.15\. sitecustomize.py
    *   例 9.16\. 设置默认编码的效果
    *   例 9.17\. 指定.py 文件的编码
    *   例 9.18\. russiansample.xml
    *   例 9.19\. 解析 russiansample.xml
*   9.5\. 搜索元素
    *   例 9.20\. binary.xml
    *   例 9.21\. getElementsByTagName 介绍
    *   例 9.22\. 每个元素都是可搜索的
    *   例 9.23\. 搜索实际上是递归的
*   9.6\. 访问元素属性
    *   例 9.24\. 访问元素属性
    *   例 9.25\. 访问单个属性

第十章 脚本和流

*   10.1\. 抽象输入源
    *   例 10.1\. 从文件中解析 XML
    *   例 10.2\. 解析来自 URL 的 XML
    *   例 10.3\. 解析字符串 XML (容易但不灵活的方式)
    *   例 10.4\. StringIO 介绍
    *   例 10.5\. 解析字符串 XML (类文件对象方式)
    *   例 10.6\. openAnything
    *   例 10.7\. 在 kgp.py 中使用 openAnything
*   10.2\. 标准输入、输出和错误
    *   例 10.8\. stdout 和 stderr 介绍
    *   例 10.9\. 重定向输出
    *   例 10.10\. 重定向错误信息
    *   例 10.11\. 打印到 stderr
    *   例 10.12\. 链接命令
    *   例 10.13\. 在 kgp.py 中从标准输入读取
*   10.3\. 查询缓冲节点
    *   例 10.14\. loadGrammar
    *   例 10.15\. 使用 ref 元素缓冲
*   10.4\. 查找节点的直接子节点
    *   例 10.16\. 查找直接子元素
*   10.5\. 根据节点类型创建不同的处理器
    *   例 10.17\. 已解析 XML 对象的类名
    *   例 10.18\. parse，通用 XML 节点分发器
    *   例 10.19\. parse 分发器调用的函数
*   10.6\. 处理命令行参数
    *   例 10.20\. sys.argv 介绍
    *   例 10.21\. sys.argv 的内容
    *   例 10.22\. getopt 介绍
    *   例 10.23\. 在 kgp.py 中处理命令行参数

第十一章 HTTP Web 服务

*   11.1\. 概览
    *   例 11.1\. openanything.py
*   11.2\. 避免通过 HTTP 重复地获取数据
    *   例 11.2\. 用直接而原始的方法下载 feed
*   11.4\. 调试 HTTP web 服务
    *   例 11.3\. 调试 HTTP
*   11.5\. 设置 User-Agent
    *   例 11.4\. urllib2 介绍
    *   例 11.5\. 给 Request 添加头信息
*   11.6\. 处理 Last-Modified 和 ETag
    *   例 11.6\. 测试 Last-Modified
    *   例 11.7\. 定义 URL 处理器
    *   例 11.8\. 使用自定义 URL 处理器
    *   例 11.9\. 支持 ETag/If-None-Match
*   11.7\. 处理重定向
    *   例 11.10\. 没有重定向处理的情况下，访问 web 服务
    *   例 11.11\. 定义重定向处理器
    *   例 11.12\. 使用重定向处理器检查永久重定向
    *   例 11.13\. 使用重定向处理器检查临时重定向
*   11.8\. 处理压缩数据
    *   例 11.14\. 告诉服务器你想获得压缩数据
    *   例 11.15\. 解压缩数据
    *   例 11.16\. 从服务器直接解压缩数据
*   11.9\. 全部放在一起
    *   例 11.17\. openanything 函数
    *   例 11.18\. fetch 函数
    *   例 11.19\. 使用 openanything.py

第十二章 SOAP Web 服务

*   12.1\. 概览
    *   例 12.1\. search.py
    *   例 12.2\. search.py 的使用样例
*   12.2.1\. 安装 PyXML
    *   例 12.3\. 检验 PyXML 安装
*   12.2.2\. 安装 fpconst
    *   例 12.4\. 检验 fpconst 安装
*   12.2.3\. 安装 SOAPpy
    *   例 12.5\. 检验 SOAPpy 安装
*   12.3\. 步入 SOAP
    *   例 12.6\. 获得现在的气温
*   12.4\. SOAP 网络服务查错
    *   例 12.7\. SOAP 网络服务查错
*   12.6\. 以 WSDL 进行 SOAP 内省
    *   例 12.8\. 揭示有效方法
    *   例 12.9\. 揭示一个方法的参数
    *   例 12.10\. 揭示方法返回值
    *   例 12.11\. 通过 WSDL proxy 调用一个 SOAP 网络服务
*   12.7\. 搜索 Google
    *   例 12.12\. 内省 Google 网络服务
    *   例 12.13\. 搜索 Google
    *   例 12.14\. 从 Google 获得次要信息
*   12.8\. SOAP 网络服务故障排除
    *   例 12.15\. 以错误的设置调用 Proxy 方法
    *   例 12.16\. 以错误参数调用方法
    *   例 12.17\. 调用时方法所期待的返回值个数错误
    *   例 12.18\. 调用方法返回一个应用特定的错误

第十三章 单元测试

*   13.3\. romantest.py 介绍
    *   例 13.1\. romantest.py
*   13.4\. 正面测试 (Testing for success)
    *   例 13.2\. testToRomanKnownValues
*   13.5\. 负面测试 (Testing for failure)
    *   例 13.3\. 测试 toRoman 的无效输入
    *   例 13.4\. 测试 fromRoman 的无效输入
*   13.6\. 完备性检测 (Testing for sanity)
    *   例 13.5\. 以 toRoman 测试 fromRoman 的输出
    *   例 13.6\. 大小写测试

第十四章 测试优先编程

*   14.1\. roman.py, 第 1 阶段
    *   例 14.1\. roman1.py
    *   例 14.2\. 以 romantest1.py 测试 roman1.py 的输出
*   14.2\. roman.py, 第 2 阶段
    *   例 14.3\. roman2.py
    *   例 14.4\. toRoman 如何工作
    *   例 14.5\. 以 romantest2.py 测试 roman2.py 的输出
*   14.3\. roman.py, 第 3 阶段
    *   例 14.6\. roman3.py
    *   例 14.7\. 观察 toRoman 如何处理无效输入
    *   例 14.8\. 用 romantest3.py 测试 roman3.py 的结果
*   14.4\. roman.py, 第 4 阶段
    *   例 14.9\. roman4.py
    *   例 14.10\. fromRoman 如何工作
    *   例 14.11\. 用 romantest4.py 测试 roman4.py 的结果
*   14.5\. roman.py, 第 5 阶段
    *   例 14.12\. roman5.py
    *   例 14.13\. 用 romantest5.py 测试 roman5.py 的结果

第十五章 重构

*   15.1\. 处理 bugs
    *   例 15.1\. 关于 Bug
    *   例 15.2\. 测试 bug (romantest61.py)
    *   例 15.3\. 用 romantest61.py 测试 roman61.py 的结果
    *   例 15.4\. 修改 Bug (roman62.py)
    *   例 15.5\. 用 romantest62.py 测试 roman62.py 的结果
*   15.2\. 应对需求变化
    *   例 15.6\. 修改测试用例以适应新需求 (romantest71.py)
    *   例 15.7\. 用 romantest71.py 测试 roman71.py 的结果
    *   例 15.8\. 为新的需求编写代码 (roman72.py)
    *   例 15.9\. 用 romantest72.py 测试 roman72.py 的结果
*   15.3\. 重构
    *   例 15.10\. 编译正则表达式
    *   例 15.11\. roman81.py 中已编译的正则表达式
    *   例 15.12\. 用 romantest81.py 测试 roman81.py 的结果
    *   例 15.13\. roman82.py
    *   例 15.14\. 以 romantest82.py 测试 roman82.py 的结果
    *   例 15.15\. roman83.py
    *   例 15.16\. 用 romantest83.py 测试 roman83.py 的结果
*   15.4\. 后记
    *   例 15.17\. roman9.py
    *   例 15.18\. 用 romantest9.py 测试 roman9.py 的结果

第十六章 函数编程

*   16.1\. 概览
    *   例 16.1\. regression.py
    *   例 16.2\. regression.py 的样例输出
*   16.2\. 找到路径
    *   例 16.3\. fullpath.py
    *   例 16.4\. os.path.abspath 的进一步解释
    *   例 16.5\. fullpath.py 的样例输出
    *   例 16.6\. 在当前目录运行脚本
*   16.3\. 重识列表过滤
    *   例 16.7\. filter 介绍
    *   例 16.8\. regression.py 中的 filter
    *   例 16.9\. 以列表解析法过滤
*   16.4\. 重识列表映射
    *   例 16.10\. map 介绍
    *   例 16.11\. map 与混合数据类型的列表
    *   例 16.12\. regression.py 中的 map
*   16.6\. 动态导入模块
    *   例 16.13\. 同时导入多个模块
    *   例 16.14\. 动态导入模块
    *   例 16.15\. 动态导入模块列表
*   16.7\. 全部放在一起
    *   例 16.16\. regressionTest 函数
    *   例 16.17\. 步骤 1：获得所有文件
    *   例 16.18\. 步骤 2：找到你关注的多个文件
    *   例 16.19\. 步骤 3：映射文件名到模块名
    *   例 16.20\. 步骤 4：映射模块名到模块
    *   例 16.21\. 步骤 5：将模块载入测试套件
    *   例 16.22\. 步骤 6：告知 unittest 使用你的测试套件

第十七章 动态函数

*   17.2\. plural.py, 第 1 阶段
    *   例 17.1\. plural1.py
    *   例 17.2\. re.sub 介绍
    *   例 17.3\. 回到 plural1.py
    *   例 17.4\. 正则表达式中否定的更多应用
    *   例 17.5\. 更多的 re.sub
*   17.3\. plural.py, 第 2 阶段
    *   例 17.6\. plural2.py
    *   例 17.7\. 剖析 plural 函数
*   17.4\. plural.py, 第 3 阶段
    *   例 17.8\. plural3.py
*   17.5\. plural.py, 第 4 阶段
    *   例 17.9\. plural4.py
    *   例 17.10\. plural4.py 继续
    *   例 17.11\. 剖析规则定义
    *   例 17.12\. plural4.py 的完成
    *   例 17.13\. 回头看 buildMatchAndApplyFunctions
    *   例 17.14\. 调用函数时展开元组
*   17.6\. plural.py, 第 5 阶段
    *   例 17.15\. rules.en
    *   例 17.16\. plural5.py
*   17.7\. plural.py, 第 6 阶段
    *   例 17.17\. plural6.py
    *   例 17.18\. 介绍生成器
    *   例 17.19\. 使用生成器替代递归
    *   例 17.20\. for 循环中的生成器
    *   例 17.21\. 生成器生成动态函数

第十八章 性能优化

*   18.1\. 概览
    *   例 18.1\. soundex/stage1/soundex1a.py
*   18.2\. 使用 timeit 模块
    *   例 18.2\. timeit 介绍
*   18.3\. 优化正则表达式
    *   例 18.3\. 目前为止最好的结果：soundex/stage1/soundex1e.py
*   18.4\. 优化字典查找
    *   例 18.4\. 目前的最佳结果：soundex/stage2/soundex2c.py
*   18.5\. 优化列表操作
    *   例 18.5\. 目前的最佳结果：soundex/stage2/soundex2c.py