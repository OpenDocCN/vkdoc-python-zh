# Chapter 0 安装 Python

# Chapter 0 安装 Python

> " *Tempora mutantur nos et mutamur in illis.* （时光流转，吾等亦随之而变。） " — 古罗马谚语

## 深入

欢迎来到 Python 3 的世界。让我们继续深入。本章中，您将安装适合自己的 Python 3 版本。

## 何种版本的 Python 适合您？

对 Python 要做的第一件事情是安装。还是说已经装了？

如果使用的是托管服务器上的帐号， ISP［互联网供应商］ 可能已经安装了 Python 3 。如果是在家运行的 Linux ，也可能已经安装了 Python 3 。多数流行的 GNU/Linux 发行包在缺省安装中都包括了 Python 2 ；为数不多但却不断增加的发行包中同时也包括了 Python 3 。Mac OS X 包括了命令行版本的 Python 2，但直至本书写作之时止，其尚未提供 Python 3。Microsoft Windows 未安装任何版本之 Python 。但是不要绝望！无论是何种操作系统，均可通过安装 Python 来开启通向光明的道路。

在 Linux 或 Mac OS X 系统上检测 Python 3 的最简单办法是进入命令行。在 Linux 中，可从 **`Application［应用程序］`** 菜单找到叫一个做 **`Terminal［终端］`** 的程序。（它也有可能位于像 **`Accessories［附件］`** 或 **`System［系统］`** 这样的子菜单内。） 在 Mac OS X 中，在 `/Application/Utilities/` 文件夹中有一个叫做 **`Terminal.app`** 的应用程序。

见到命令行提示符之后，只需输入 `python3` （全部字母小写、无空格），并观察接下来发生的事情。我家中的 Linux 系统已经安装了 Python 3 ，运行该命令将把我带入*Python 交互式 shell* 中。

```py
mark@atlantis:~$ python3
Python 3.0.1+ (r301:69556, Apr 15 2009, 17:25:52)
[GCC 4.3.3] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> 
```

（输入 `exit()` 并按下 `回车键` 可退出 Python 交互式 shell。）

我选择的 [虚拟主机服务商](http://cornerhost.com/) 也运行 Linux 并提供命令行访问，但我的服务器未安装 Python 3 。（嘘！）

```py
mark@manganese:~$ python3
bash: python3: command not found 
```

因此无论在计算机上已安装了哪个版本，让我们回到本节开始时提到的问题：“哪种 Python 版本适合你？“，

［阅读关于 Windows 的指导，或者跳到 在 Mac OS X 上安装、 在 Ubuntu Linux 上安装 或 在其它平台上安装。］

## 在 Microsoft Windows 上安装

当前 Windows 有两种架构： 32 位 和 64 位。当然，还有很多不同的 Windows *版本* — XP、 Vista、 Windows 7 — 而 Python 可在所有这些版本上运行。The more important distinction is 32-bit v. 64-bit. 如果不知道目前正在运行何种架构，那么多半是 32 位的。

访问 [`python.org/download/`](http://python.org/download/) 并下载与计算机架构对应的 Python 3 Windows 安装程序。面对的选择可能包括下面这些：

*   **Python 3.1 Windows 安装程序** （Windows 二进制 — 不包括源码）
*   **Python 3.1 Windows AMD64 安装程序** （Windows AMD64 二进制 — 不包括源码）

未在此处提供直接下载链接是因为 Python 总是在进行小的更新，而我又不想为您错过更新负责。应该总是安装最新的 Python 3.x 版本，除非您有特别的理由不这么做。

1.  ![[Windows 对话框：打开文件安全警告]](win-install-0-security-warning.png)

    下载完成后，双击该 `.msi` 文件。由于正要运行的是可执行代码，Windows 将弹出一个安全警告。官方 Python 安装程序由负责 Python 开发的非盈利性组织 [Python 软件基金会](http://www.python.org/psf/) 进行数字签名。千万别接受山寨版！

    点击 `Run［运行］` 按钮启动 Python 3 安装程序。

2.  ![[Python 安装程序：选择是否为本计算机的所有用户安装 Python 3.1。]](win-install-1-all-users-or-just-me.png)

    安装程序将会询问的第一个问题是：是为所有用户，还是仅为您自己安装 Python 3。缺省的选项是 “为所有用户安装”，如果没有更好理由选择其它选项，这是最好的选择。（想要”只为我安装“的一个可能原因是：正往公司的计算机上安装 Python 而您的 Windows 帐号又没有 Administrator 权限。不过，您又为啥未经公司 Windows 管理员的许可而安装 Python 呢？这个问题上不要给我惹麻烦！）

    点击 `Next［下一步］` 按钮接受对安装类型的选择。

3.  ![[Python 安装程序：选择目标目录]](win-install-2-destination-directory.png)

    接下来，安装程序将会提示选择一个目标目录。所有 Python 3.1.x 版本缺省的目标目录是： `C:\Python31\`，这对绝大多数用户都是合适的，除非您有特别的理由修改它。如果有单独的磁盘驱动器用于安装应用程序，可通过嵌入式控件找到它，或直接在下方的文本框中输入该路径名。如果在 `C:` 盘安装 Python 受限；可在其它盘的任何目录下安装。

    点击 `Next [下一步]` 按钮接受对目标目录的选择。

4.  ![[Python 安装程序: 定制 Python 3.1]](win-install-3-customize.png)

    接下来的页面看着有点复杂，但其实并不真的复杂。和其它安装程序一样，您可以选择不安装 Python 3 每个单独部件。如果磁盘空间特别紧张，可以将某些部件排除在外。

    *   **Register Extensions ［注册扩展名］** 允许通过双击 Python 脚本 (`.py` files) 来运行它们。建议选上，但不是必需的。（该选项不占用任何磁盘空间，因此排除它没有任何意义。）
    *   **Tcl/Tk** 是 Python Shell 使用的图形化类库，您将在整本书都用到它。强烈建议保留该选项。
    *   **Documentation ［文档］** 安装的帮助文件包括大量来自 [`docs.python.org`](http://docs.python.org/) 信息。如果使用拨号上网或者互联网访问受限的话，建议保留。
    *   **Utility Scripts［实用脚本］** 包括本书稍后将学到的 `2to3.py` 脚本。如果想学习如何将现有 Python 2 代码移植到 Python 3 ，这是必需的部件。若无现有的 Python 2 代码，可略过该选项。
    *   **Test Suite ［测试套件］** 是用于测试 Python 解释器的脚本集合。本书中将不会用到，而且我在用 Python 编程的过程中也从未用到。完全是可选的。
5.  ![[Python 安装程序：磁盘空间需求]](win-install-3a-disk-usage.png)

    如果不确定有多少磁盘空间，点击 `Disk Usage［磁盘使用情况］`按钮。安装程序将列出所有驱动器盘符，并计算每个驱动器上有多少可用空间，以及安装后会剩下多少空间。

    点击 `OK［确定］` 按钮返回“Customizing Python［自定义 Python］” 页面。

6.  ![[Python 安装程序：删除测试套件可以省出 7908KB 硬盘空间。]](win-install-3b-test-suite.png)

    如果决心排除某选项，选择选项之前的下拉选项按钮并选中 “Entire feature will be unavailable.［整个功能将不可用］”选项。例如，排除 Test Suite ［测试套件］将节省高达 7908KB 的磁盘空间。

    点击 `Next［下一步］` 按钮接受对所选内容的选择。

7.  ![[Python 安装程序: 进度表]](win-install-4-copying.png)

    安装程序将把所有必需的文件拷贝到所选择的目标目录中。（该过程非常快捷，以至于我不得不试了三遍才捕捉到它的屏幕截图！）

8.  ![[Python 安装程序：安装完成特别视窗归功于 Mark Hammond，没有这些经年累月免费共享的 Windows 专业知识，Python for Windows 可能还只是 Python for DOS 。]](win-install-5-finish.png)

    点击 `Finish［完成］` 按钮退出该安装程序。

9.  ![[Windows Python Shell, Python 的图形化交互 Shell]](win-interactive-shell.png)

    在 `开始` 菜单中，将会出现一条名为 `Python 3.1` 的新菜单项。在其中有一个名为 IDLE 的程序。选择此菜单项以运行交互式 Python Shell 。

跳到 [使用 Python Shell]

## 在 Mac OS X 上安装

所有的现代麦金塔计算机使用英特尔芯片（像大多数 Windows PC 一样)。旧款的苹果电脑使用 PowerPC 芯片。你无须理解其中区别，因为所有苹果电脑只有一种 Mac Python 安装程序。

访问 [`python.org/download/`](http://python.org/download/) 并下载 Mac 安装程序。它可能被叫做 **Python 3.1 Mac Installer Disk Image** 之类的名字，尽管版本号可能会不同。请确定下载的是 3.x 版，而不是 2.x 版。

1.  ![[Python 安装程序磁盘映像的内容]](mac-install-0-dmg-contents.png)

    浏览器可以自动挂载磁盘映像，并打开一个 Finder 窗口展示其内容。（如果没有发生这样的情形，则需要在下载目录中找到磁盘映像，并双击挂载。它可能被命名为 `python-3.1.dmg` 之类的名称。）磁盘映像包括一些文本文件（`Build.txt`、 `License.txt`、 `ReadMe.txt`），以及实际的安装程序包，`Python.mpkg`。

    双击 `Python.mpkg` 安装程序包以启动 Mac Python 安装程序。

2.  ![[Python 安装程序: 欢迎词画面]](mac-install-1-welcome.png)

    安装程序的第一页就 Python 本身给出了一段简要描述，然后提示您参阅 `ReadMe.txt` 文件（您没有读过该文件，不是吗？）以掌握更多细节。

    点击 `Continue［继续］` 按钮进入下一步。

3.  ![Python 安装程序：所支持的架构、磁盘空间及可接受目标目录等相关信息］

    接下来的页面实际包含一些重要信息： Python 必须安装在 Mac OS X 10.3 或其后续版本之上。如果仍在使用 Mac OS X 10.2，那就真的需要升级一下了。苹果公司已经不再为（Mac OS X 10.2）操作系统提供安全更新了，而且如果曾经上网的话，您的计算机可能已经处于危险之中了。此外，您也无法运行 Python 3 。

    点击 `Continue［继续］` 按钮继续前进。

4.  ![[Python 安装程序：软件许可协议]](mac-install-3-license.png)

    如同所有优秀的安装程序，Python 安装程序列出了软件许可协议。Python 是开源软件，其许可协议由 [Open Source Initiative［开源软件促进会］](http://opensource.org/licenses/) 提供。历史上，Python 有过一些所有者和赞助者，每个都在软件许可协议之上留下了痕迹。但最终结果是：Python 是开源的，可在任何平台上为任何目的使用它，而无需付费或承担对等义务。

    再次点击 `Continue［继续］` 按钮。

5.  ![[Python 安装程序：接受许可协议对话框]](mac-install-4-license-dialog.png)

    根据苹果安装程序框架的习惯，必须“agree［同意］” 软件许可协议以完成安装。由于 Python 是开源的，实际上您所“同意”的只是授予您额外的权利，而不是剥夺它们。

    点击 `Agree［同意］` 按钮以继续安装。

6.  ![[Python 安装程序: 标准安装画面]](mac-install-5-standard-install.png)

    下一个画面允许您修改安装位置。**必须** 将 Python 安装到启动驱动器上，但由于安装程序的限制，它并没有强迫这么做。说实话，我从来没有需要过修改安装位置。

    从该画面中，您还可以自定义安装以剔除特定功能。如果想这么做，点击 `Customize［自定义］` 按钮；否则点击 `Install［安装］` 按钮。

7.  ![[Python 安装程序: 定制安装画面]](mac-install-6-custom-install.png)

    如果选择了自定义安装，安装程序将为您提供下列功能：

    *   **Python Framework ［Python 框架］**. 这是 Python 的核心所在，由于必须被安装，它已经被选中并处于无法取消状态。
    *   **GUI Applications［GUI 应用程序］** 包括 IDLE，即本书通篇将用到的图形化 Python Shell 。强烈建议保留该选项。
    *   **UNIX command-line tools［UNIX 命令行工具］** 包括了 `python3` 命令行应用程序。同样强烈建议保留该选项。
    *   **Python Documentation［Python 文档］** 包含了来自 [`docs.python.org`](http://docs.python.org/) 的许多信息。如果使用拨号上网或者互联网访问受限的话，建议保留。
    *   **Shell profile updater［Shell 文档更新程序］** 控制是否更新 shell 设置（用于 `Terminal.app` 中）以确保此版本的 Python 位于 Shell 的搜索路径当中。您可能不需要修改该项设置。
    *   **Fix system Python［修复系统 Python］** 不应作变更。（它告诉 Mac 将 Python 3 用作所有脚本的缺省 Python ，包括来自苹果公司的内置系统脚本。这将会导致非常糟糕的结果，因为多数这些脚本是为 Python 2 编写的，在 Python 3 环境中将无法正确运行。）

    点击 `Install［安装］` 按钮以继续。

8.  ![[Python 安装程序：输入管理员密码的对话框]](mac-install-7-admin-password.png)

    由于是安装系统级的框架，且二进制文件被安装至 `/usr/local/bin/` 之中，安装程序将会向您询问管理员口令。没有管理员权限是无法安装 Mac Python 的。

    点击 `OK［确定］` 按钮开始安装。

9.  ![[Python 安装程序: 进度表]](mac-install-8-progress.png)

    在安装所选功能时，安装程序将会显示进度条。

10.  ![[Python 安装程序：安装成功]](mac-install-9-succeeded.png)

    假定一切顺利，安装程序将会展示一个很大的绿色对号，告知安装成功完成。

    点击 `Close［关闭］` 按钮退出该安装程序。

11.  ![[contents of /Applications/Python 3.1/ folder]](mac-install-10-application-folder.png)

    加入没有修改安装位置，您可以在 `/Applications` 目录下的 `Python 3.1` 目录中找到新安装的文件。 最重要的部分是图形化 Python Shell IDLE。

    双击 IDLE 以启动 Python Shell。

12.  ![[Windows Python Shell, Python 的图形化交互 Shell]](mac-interactive-shell.png)

    Python Shell 是您探索 Python 过程中花费时间最多的地方。本书中所有的例子都假定您能够找到进入 Python Shell 的方法。

跳到 [使用 Python Shell]

## 在 Ubuntu Linux 上安装

现代的 Linux 发行版背后都有着大型的预编译应用程序仓库，随时可用于安装。具体的细节各发行版均不同。对于 Ubuntu Linux 而言，安装 Python 3 的最简单途径是通过 `Applications` 菜单中的`增加／删除` 应用程序。

1.  ![[增加／删除：Add/Remove: Canonical 维护的应用程序]](ubu-install-0-add-remove-programs.png)

    在首次运行 `增加／删除` 应用程序时，它将展示一份分成多类的预选程序清单。有的已经安装；多数还没有。因为该仓库包括超过 10,000 种应用程序，所以可以使用过滤器参看仓库的不同部分。默认过滤器是“由 Canonical 维护的应用程序”，它是创建及维护 Ubuntu Linux 的 Canonical 公司官方所支持的大量应用程序中的一个小子集。

2.  ![[增加／删除：所有的开源应用程序]](ubu-install-1-all-open-source-applications.png)

    Python 3 并非由 Canonical 维护，因此第一个步骤是下拉过滤器菜单，并选择“所有开源应用程序”。

3.  ![[增加／删除：搜索 Python 3]](ubu-install-2-search-python-3.png)

    放宽过滤器以包括所有开源应用程序之后，使用进紧挨着过滤器菜单的”搜索“框来搜索 `Python 3`。

4.  ![[增加／删除：选择 Python 3.0 安装包]](ubu-install-3-select-python-3.png)

    现在应用程序列表收窄为仅包括匹配 `Python 3` 的那些内容。您将查看两个安装包。第一个是 `Python (v3.0)` 。该安装包包含了 Python 解释器自身。

5.  ![[增加／删除：为 Python 3.0 安装包选择 IDLE]](ubu-install-4-select-idle.png)

    第二个要安装的包就在正上方： `IDLE (using Python-3.0)`。这是你在整本书都要用到的图形化 Python Shell 。

    选好这两个包后，点击 `Apply Changes［应用修改］` 按钮以继续。

6.  ![[增加／删除：应用修改]](ubu-install-5-apply-changes.png)

    该软件包管理器将会要求您确认是否要添加 `IDLE (using Python-3.0)` 和 `Python (v3.0)` 。

    点击 `Apply［应用］` 按钮以继续。

7.  ![[增加／删除：下载进度条]](ubu-install-6-download-progress.png)

    在从 Canonical 互联网仓库下载所需安装包时，软件包管理器将显示一个进度条。

8.  ![[增加／删除：安装进度条]](ubu-install-7-install-progress.png)

    下好安装包后，软件包管理器将会自动开始安装。

9.  ![[增加／删除：新的应用程序已安装。]](ubu-install-8-success.png)

    如果一切顺利，软件包管理器将确认两个安装包都已安装成功。从此，您可双击 IDLE 启动 Python Shell，或者点击 `Close［关闭］` 按钮退出软件包管理器。

    您还可以从 `Applications［应用程序］` 菜单，然后进入`Programming` 子菜单并选择 IDLE，以重新启动 Python Shell。

10.  ![[Linux Python Shell, Python 的图形化交互 Shell]](ubu-interactive-shell.png)

    Python Shell 是您探索 Python 过程中花费时间最多的地方。本书中所有的例子都假定您能够找到进入 Python Shell 的方法。

跳到 [使用 Python Shell]

## 在其它平台上安装

Python 3 还可在一些其它平台上安装。特别要指出的是，它几乎可以在所有的 Linux、 BSD 和基于 Solaris 的发行版纸上安装。例如，RedHat Linux 使用 `yum` 软件包管理器；FreeBSD 有 [移植和软件包集合](http://www.freebsd.org/ports/)；Solaris 有 `pkgadd` 和 friends 。在网上快速搜索 `Python 3` + *您的操作系统* 将会告诉你是否存在该平台的 Python 以及如何安装。

## 使用 Python Shell

Python Shell 是您探索 Python 语法，通过命令获取交互式帮助以及调试段程序的地方。图形化 Python Shell （名为 IDLE）还包括了一个不错的文本编辑器，它支持 Python 语法着色并与 Python Shell 进行了整和。如果还没有喜欢的文本编辑器，不妨试用下 IDLE 。

重中之重。Python Shell 本身是一款了不起的互动环境。在本书中，您将看到下面这样的例子：

```py
>>> 1 + 1
2 
```

这三个尖括号， `&gt;&gt;&gt;`，表示 Python Shell 提示符。不要输入该部分。它只是让您知道该例要在 Python Shell 中运行。

`1 + 1` 是您输入的部分。您可在 Python Shell 中输入任何有效的 Python 表达式和命令。别怕羞，它不会咬你！最糟糕的事情也不过看到一条错误信息。命令将立即得到执行（一旦您按下 `ENTER［回车键］`）；表达式的值将立即得到计算，而 Python Shell 将输出结果。

`2` 是该表达式的计算结果。事实上，`1 + 1` 是一个有效的 Python 不等式。结果，当然，是 `2` 。

让我们尝试下另一个例子.

```py
>>> print('Hello world!')
Hello world! 
```

很简单，不是吗？但你在 Python shell 中可完成的工作要多得多。如果您被困住了——无法想起某个命令，或者无法想起如何正确给某个函数传递参数——您可寻求 Python Shell 的交互式帮助。只需输入 `help` 并按下 `回车键` 。

```py
>>> help
Type help() for interactive help, or help(object) for help about object. 
```

有两种帮助模式。您可以获得某个对象的帮助，这样将只打印出文档并返回 Python Shell 提示符。您也可以输入 *help mode*，系统将不会计算 Python 表达式，您只需输入关键字或命令名称，系统将会输出关于该命令它所知道的内容。

要进入交互帮助模式，仅需输入 `help()` 并按下 `回车键`。

```py
>>> help()
Welcome to Python 3.0!This is the online help utility.

If this is your first time using Python, you should definitely check out
the tutorial on the Internet at http://docs.python.org/tutorial/.

Enter the name of any module, keyword, or topic to get help on writing
Python programs and using Python modules.  To quit this help utility and
return to the interpreter, just type "quit".

To get a list of available modules, keywords, or topics, type "modules",
"keywords", or "topics".  Each module also comes with a one-line summary
of what it does; to list the modules whose summaries contain a given word
such as "spam", type "modules spam". 
help> 
```

请注意提示符是如何从 `&gt;&gt;&gt;` 改变为 `help&gt;` 的。该提示符提醒您目前正处于交互式帮助模式。现在您可以输入任何关键字、命令、模块名称、函数名称 — 几乎任何 Python 能够理解的一切 — 然后阅读其文档。

```py
 Help on built-in function print in module builtins:

print(...)
    print(value, ..., sep=' ', end='\n', file=sys.stdout)

    Prints the values to a stream, or to sys.stdout by default.
    Optional keyword arguments:
    file: a file-like object (stream); defaults to the current sys.stdout.
    sep:  string inserted between values, default a space.
    end:  string appended after the last value, default a newline. 

no Python documentation found for 'PapayaWhip' 

 You are now leaving help and returning to the Python interpreter.
If you want to ask for help on a particular object directly from the
interpreter, you can type "help(object)".  Executing "help('string')"
has the same effect as typing a particular string at the help> prompt. 
```

1.  要获取 `print()` 函数的文档，仅需输入 `print` 然后按下 `回车键` 。该交互式帮助模式将会显示类似 man 页面的内容：函数名称、简要内容、函数的参数及缺省值等等。如果文档看起来很难懂，千万别慌。您将在后面不远的章节中学到关于这些概念的更多内容。
2.  当然，交互式帮助模式并不知道一切。如果您所输入的不是 Python 的命令、模块、函数或者其它内建关键字，交互式帮助模式将只能耸耸虚拟的肩膀。
3.  要退出交互帮助模式，仅需输入 `quit()` 并按下 `回车键`。
4.  提示符将变回 `&gt;&gt;&gt;` 以提示您已经离开交互帮助模式，并返回到了 Python Shell 。

图形化的 Python Shell —— IDLE,同样带有一个 Python 相关的文本编辑器。

## Python 编辑器和集成开发环境

如果要以 Python 编写程序，IDLE 并不是唯一的编辑器选择。尽管它对于初学该语言非常有帮助，但许多开发人员更喜欢其它文本编辑器或集成开发环境。（IDEs）在此我不想展开阐述，Python 社区维护了一份 [Python 相关编辑器的清单](http://wiki.python.org/moin/PythonEditors)，涵盖了各种各样支持平台和软件许可协议。

您可能也想查看一下这份 [Python 相关 IDEs](http://wiki.python.org/moin/IntegratedDevelopmentEnvironments) 的清单，尽管其中还只有少数才支持 Python 3 。其中之一是 [PyDev](http://pydev.sourceforge.net/)，[Eclipse](http://eclipse.org/) 的一种插件，它将 Eclipse 变成了一种成熟的 Python IDE。Eclipse 和 PyDev 都是跨平台的开源软件。

在商业方面，有 ActiveState 公司的 [Komodo IDE](http://www.activestate.com/komodo/) 。它需要用户为单位的授权许可，但学生可以得到折扣，同时还有时间受限的免费试用版。

在用 Python 编程的九年中，我使用 [GNU Emacs](http://www.gnu.org/software/emacs/) 编辑 Python 程序，并在命令行 Python Shell 中进行调试。对于使用 Python 开发来说，编辑器之选没有绝对的正确和错误。重要的是找到适合自己的道路！