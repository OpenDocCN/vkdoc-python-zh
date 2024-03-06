# 第一章 安装 Python

# 第一章 安装 Python

*   1.1\. 哪一种 Python 适合您？
*   1.2\. Windows 上的 Python
*   1.3\. Mac OS X 上的 Python
*   1.4\. Mac OS 9 上的 Python
*   1.5\. RedHat Linux 上的 Python
*   1.6\. Debian GNU/Linux 上的 Python
*   1.7\. 从源代码安装 Python
*   1.8\. 使用 Python 的交互 Shell
*   1.9\. 小结

欢迎来到 Python 世界，让我们开始吧。在本章中，将学习适合您的 Python 安装。

# 1.1\. 哪一种 Python 适合您？

# 1.1\. 哪一种 Python 适合您？

学习 Python 的第一件事就是安装，不是吗？

如果您在公网的服务器上有个用户账号，那么您的 ISP 或许已经安装了 Python。 大多数 Linux 发行版在默认安装的情况下就已经提供了 Python。 虽然您可能希望在苹果机上安装一个拥有类 Mac 的图形操作界面，但在 Mac OS X 10.2 或更高的版本上已经包含了一个 Python 的命令行版本。

Windows 环境默认不提供任何版本的 Python，但是不要担心！本章将提供几种 Windows 环境下安装 Python 的方法。

正像您所看到的，Python 可以运行于很多操作系统平台。包括 Windows、Mac OS、Mac OS X、所有免费的类 UNIX 变种 (如 Linux)。也有运行于 Sun Solaris、AS/400、Amiga、OS/2、BeOS 的版本，甚至是您从来没听说过的其他操作系统平台。

有太多的平台可以运行 Python 了。在一种平台下编写的 Python 程序稍作修改，就可以运行于*任何* 其他支持的平台。例如，我通常在 Windows 平台上开发 Python 程序，然后适当配置后使之能在 Linux 平台上运行。

回到开始的问题，“哪一种 Python 适合您？” 回答是：哪一个已经安装在您计算机上均可。

# 1.2\. Windows 上的 Python

# 1.2\. Windows 上的 Python

在 Windows 上，安装 Python 有两种选择。

ActiveState 制作的 ActivePython 是专门针对 Windows 的 Python 套件，它包含了一个完整的 Python 发布、一个适用于 Python 编程的 IDE 以及一些 Python 的 Windows 扩展，提供了全部的访问 Windows APIs 的服务，以及 Windows 注册表的注册信息。

虽然 ActivePython 不是开源软件，但它可以自由下载。ActivePython 是我学习 Python 时使用过的 IDE。除非有别的原因，我建议您使用它。可能的一个原因是：ActiveState 通常要在新的 Python 版本发布几个月以后才更新它的安装程序。如果您就需要 Python 的最新版本，并且 ActivePython 仍然落后于最新版本的话，您应该直接跳到在 Windows 上安装 Python 的第二种选项。

第二种选择是使用由 Python 发布的 “官方” Python 安装程序。她是可自由下载的开源软件，并且您总是可以获得当前 Python 的最新版本。

## 过程 1.1\. 选项 1：安装 ActivePython

下面描述 ActivePython 的安装过程：

1.  从 [`www.activestate.com/Products/ActivePython/`](http://www.activestate.com/Products/ActivePython/) 下载 ActivePython 。

2.  如果您正在使用 Windows 95、Windows 98 或 Windows ME，还需要在安装 ActivePython 之前下载并安装[Windows Installer 2.0](http://download.microsoft.com/download/WindowsInstaller/Install/2.0/W9XMe/EN-US/InstMsiA.exe) 。

3.  双击安装程序 `ActivePython-2.2.2-224-win32-ix86.msi`。

4.  按照安装程序的提示信息一步步地执行。

5.  如果磁盘空间不足，您可以执行定制安装，不选文档，但是笔者不建议您这样做，除非您实在是挤不出 14M 空间来。

6.  在安装完后之后，关闭安装程序，打开 开始->程序->ActiveState ActivePython 2.2->PythonWin IDE。您将看到类似如下的信息：

```py
PythonWin 2.2.2 (#37, Nov 26 2002, 10:24:37) [MSC 32 bit (Intel)] on win32.
Portions Copyright 1994-2001 Mark Hammond (mhammond@skippinet.com.au) -
see 'Help/About PythonWin' for further copyright information.
>>> 
```

## 过程 1.2\. 选项 2：安装来自 [Python.org](http://www.python.org/ "Python language home page") 的 Python

1.  从 [`www.python.org/ftp/python/`](http://www.python.org/ftp/python/) 选择最新的 Python Windows 安装程序，下载 `.exe` 安装文件。

2.  双击安装程序 `Python-2.xxx.yyy.exe`。文件名依赖于您所下载的 Python 安装程序文件。

3.  按照安装程序的提示信息一步步地执行。

4.  如果磁盘空间不足，可以取消 HTMLHelp 文件、实用脚本 (`Tools/`)、和/或测试套件 (`Lib/test/`)。

5.  如果您没有机器的管理员权限，您可以选择 Advanced Options，然后选择 Non-Admin Install。这只会对登记注册表和开始菜单中创建的快捷方式有影响。

6.  在安装完成之后，关闭安装程序，打开 开始->程序->Python 2.3->IDLE (Python GUI)。您将看到类似如下的信息：

```py
Python 2.3.2 (#49, Oct  2 2003, 20:02:00) [MSC v.1200 32 bit (Intel)] on win32
Type "copyright", "credits" or "license()" for more information.
    ****************************************************************
    Personal firewall software may warn about the connection IDLE
    makes to its subprocess using this computer's internal loopback
    interface.  This connection is not visible on any external
    interface and no data is sent to or received from the Internet.
    ****************************************************************
IDLE 1.0
>>> 
```

# 1.3\. Mac OS X 上的 Python

# 1.3\. Mac OS X 上的 Python

在 Mac OS X 上，对于安装 Python 有两种选择：安装或不安装。您可能想要安装它。

Mac OS X 10.2 及其后续版本已经预装了一个 Python 的命令行版本。如果您习惯使用命令行，那么您可以使用它学完本书的三分之一。然而，预安装的版本不带 XML 解析器，所以当您学到 XML 的章节时，您会需要安装完整版。

您还可以安装优于预装版本的最新的包含图形界面 Shell 的完整版本。

## 过程 1.3\. 在 Mac OS X 上运行预装版本的 Python

使用预装的 Python 版本的步骤：

1.  打开 `/Applications` 文件夹。

2.  打开 `Utilities` 文件夹。

3.  双击 `Terminal` 打开一个终端进入命令行窗口。

4.  在提示符下键入 **`python`**。

试验:

```py
Welcome to Darwin!
[localhost:~] you% python
Python 2.2 (#1, 07/14/02, 23:25:09)
[GCC Apple cpp-precomp 6.14] on darwin
Type "help", "copyright", "credits", or "license" for more information.
>>> [press Ctrl+D to get back to the command prompt]
[localhost:~] you% 
```

## 过程 1.4\. 在 Mac OS X 上安装最新版的 Python

下面介绍下载并安装 Python 最新版本的过程:

1.  从 [`homepages.cwi.nl/~jack/macpython/download.html`](http://homepages.cwi.nl/~jack/macpython/download.html) 下载 `MacPython-OSX` 磁盘镜像 。

2.  下载完毕，双击 `MacPython-OSX-2.3-1.dmg` 将磁盘镜像挂载到桌面。

3.  双击安装程序 `MacPython-OSX.pkg`.

4.  安装程序将提示要求您的管理员用户名和口令。

5.  按照安装程序的提示一步步执行。

6.  安装完毕后，关闭安装程序，打开 `/Applications` 文件夹。

7.  打开 `MacPython-2.3` 文件夹。

8.  双击 `PythonIDE` 来运行 Python 。

MacPython IDE 将显示一个弹出屏幕界面将您带进交互 shell。如果交互 shell 没有出现，选择 Window->Python Interactive (****Cmd**-0**)。您将看到类似如下的信息:

```py
Python 2.3 (#2, Jul 30 2003, 11:45:28)
[GCC 3.1 20020420 (prerelease)]
Type "copyright", "credits" or "license" for more information.
MacPython IDE 1.0.1
>>> 
```

请注意，安装完最新版本后，预装版本仍然存在。如果您从命令行运行脚本，那您需要知道正在使用的是哪一个版本的 Python 。

## 例 1.1\. 两个 Python 版本

```py
[localhost:~] you% python
Python 2.2 (#1, 07/14/02, 23:25:09)
[GCC Apple cpp-precomp 6.14] on darwin
Type "help", "copyright", "credits", or "license" for more information.
>>> [press Ctrl+D to get back to the command prompt]
[localhost:~] you% /usr/local/bin/python
Python 2.3 (#2, Jul 30 2003, 11:45:28)
[GCC 3.1 20020420 (prerelease)] on darwin
Type "help", "copyright", "credits", or "license" for more information.
>>> [press Ctrl+D to get back to the command prompt]
[localhost:~] you% 
```

# 1.4\. Mac OS 9 上的 Python

# 1.4\. Mac OS 9 上的 Python

Mac OS 9 上没有预装任何版本的 Python，安装相对简单，只有一种选择。

下面介绍在 Mac OS 9 上安装 Python 的过程:

1.  从 [`homepages.cwi.nl/~jack/macpython/download.html`](http://homepages.cwi.nl/~jack/macpython/download.html) 下载 `MacPython23full.bin`。

2.  如果浏览器不能自动解压文件，那么双击 `MacPython23full.bin` 用 Stuffit Expander 解压。

3.  双击安装程序 `MacPython23full`。

4.  按照安装程序的提示一步步执行。

5.  安装完毕后，关闭安装程序，打开 `/Applications` 文件夹。

6.  打开 `MacPython-OS9 2.3` 文件夹。

7.  双击 `PythonIDE` 来运行 Python 。

MacPython IDE 将显示一个弹出屏幕界面将您带进交互 shell。如果交互 shell 没有出现，选择 Window->Python Interactive (****Cmd**-0**)。您将看到类似如下的信息:

```py
Python 2.3 (#2, Jul 30 2003, 11:45:28)
[GCC 3.1 20020420 (prerelease)]
Type "copyright", "credits" or "license" for more information.
MacPython IDE 1.0.1
>>> 
```

# 1.5\. RedHat Linux 上的 Python

# 1.5\. RedHat Linux 上的 Python

在类 UNIX 的操作系统 (如 Linux) 上安装二进制包很容易。预编译好的二进制包对大多数 Linux 发行版是可用的。或者您可以通过源码进行编译。

在 [`www.python.org/ftp/python/`](http://www.python.org/ftp/python/) 选择列出的最新的版本号, 然后选择 其中的`rpms/` 目录下载最新的 Python RPM 包。 使用 **rpm** 命令进行安装，操作如下所示:

## 例 1.2\. 在 RedHat Linux 9 上安装

```py
localhost:~$ su -
Password: [enter your root password]
[root@localhost root]# wget http://python.org/ftp/python/2.3/rpms/redhat-9/python2.3-2.3-5pydotorg.i386.rpm
Resolving python.org... done.
Connecting to python.org[194.109.137.226]:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 7,495,111 [application/octet-stream]
...
[root@localhost root]# rpm -Uvh python2.3-2.3-5pydotorg.i386.rpm
Preparing...                ########################################### [100%]
   1:python2.3              ########################################### [100%]
[root@localhost root]# python          
Python 2.2.2 (#1, Feb 24 2003, 19:13:11)
[GCC 3.2.2 20030222 (Red Hat Linux 3.2.2-4)] on linux2
Type "help", "copyright", "credits", or "license" for more information.
>>> [press Ctrl+D to exit]
[root@localhost root]# python2.3       
Python 2.3 (#1, Sep 12 2003, 10:53:56)
[GCC 3.2.2 20030222 (Red Hat Linux 3.2.2-5)] on linux2
Type "help", "copyright", "credits", or "license" for more information.
>>> [press Ctrl+D to exit]
[root@localhost root]# which python2.3 
/usr/bin/python2.3 
```

|  |  |
| --- | --- |
| [1] | 仅仅键入 **`python`** 运行的是老版本的 Python ――它是缺省安装的版本。它不是我们想要的。 |
| [2] | 截止到笔者写作时，新的版本是 **`python2.3`**。您可能会需要修改示例脚本的第一行的路径指向新版本。 |
| [3] | 这是我们刚安装的 Python 新版本的全路径。在 `#!` 行中 (每个脚本的第一行) 使用它来确保脚本运行在最新版的 Python 下，并且确保敲入的是 **`python2.3`** 进入交互 shell。 |

# 1.6\. Debian GNU/Linux 上的 Python

# 1.6\. Debian GNU/Linux 上的 Python

如果您运行在 Debian GNU/Linux 上，安装 Python 需要使用 **apt** 命令。

## 例 1.3\. 在 Debian GNU/Linux 上安装

```py
localhost:~$ su -
Password: [enter your root password]
localhost:~# apt-get install python
Reading Package Lists... Done
Building Dependency Tree... Done
The following extra packages will be installed:
  python2.3
Suggested packages:
  python-tk python2.3-doc
The following NEW packages will be installed:
  python python2.3
0 upgraded, 2 newly installed, 0 to remove and 3 not upgraded.
Need to get 0B/2880kB of archives.
After unpacking 9351kB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Selecting previously deselected package python2.3.
(Reading database ... 22848 files and directories currently installed.)
Unpacking python2.3 (from .../python2.3_2.3.1-1_i386.deb) ...
Selecting previously deselected package python.
Unpacking python (from .../python_2.3.1-1_all.deb) ...
Setting up python (2.3.1-1) ...
Setting up python2.3 (2.3.1-1) ...
Compiling python modules in /usr/lib/python2.3 ...
Compiling optimized python modules in /usr/lib/python2.3 ...
localhost:~# exit
logout
localhost:~$ python
Python 2.3.1 (#2, Sep 24 2003, 11:39:14)
[GCC 3.3.2 20030908 (Debian prerelease)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> [press Ctrl+D to exit] 
```

# 1.7\. 从源代码安装 Python

# 1.7\. 从源代码安装 Python

如果您宁愿从源码创建，可以从 [`www.python.org/ftp/python/`](http://www.python.org/ftp/python/)下载 Python 的源代码。选择最新的版本，下载`.tgz` 文件，执行通常的 **`configure`**, **`make`**, **`make install`** 步骤。

## 例 1.4\. 从源代码安装

```py
localhost:~$ su -
Password: [enter your root password]
localhost:~# wget http://www.python.org/ftp/python/2.3/Python-2.3.tgz
Resolving www.python.org... done.
Connecting to www.python.org[194.109.137.226]:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 8,436,880 [application/x-tar]
...
localhost:~# tar xfz Python-2.3.tgz
localhost:~# cd Python-2.3
localhost:~/Python-2.3# ./configure
checking MACHDEP... linux2
checking EXTRAPLATDIR...
checking for --without-gcc... no
...
localhost:~/Python-2.3# make
gcc -pthread -c -fno-strict-aliasing -DNDEBUG -g -O3 -Wall -Wstrict-prototypes
-I. -I./Include  -DPy_BUILD_CORE -o Modules/python.o Modules/python.c
gcc -pthread -c -fno-strict-aliasing -DNDEBUG -g -O3 -Wall -Wstrict-prototypes
-I. -I./Include  -DPy_BUILD_CORE -o Parser/acceler.o Parser/acceler.c
gcc -pthread -c -fno-strict-aliasing -DNDEBUG -g -O3 -Wall -Wstrict-prototypes
-I. -I./Include  -DPy_BUILD_CORE -o Parser/grammar1.o Parser/grammar1.c
...
localhost:~/Python-2.3# make install
/usr/bin/install -c python /usr/local/bin/python2.3
...
localhost:~/Python-2.3# exit
logout
localhost:~$ which python
/usr/local/bin/python
localhost:~$ python
Python 2.3.1 (#2, Sep 24 2003, 11:39:14)
[GCC 3.3.2 20030908 (Debian prerelease)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> [press Ctrl+D to get back to the command prompt]
localhost:~$ 
```

# 1.8\. 使用 Python 的交互 Shell

# 1.8\. 使用 Python 的交互 Shell

既然我们已经安装了 Python，那么我们运行的这个交互 shell 是什么东西呢？

Python 扮演着两种角色。首先它是一个脚本解释器，可以从命令行运行脚本，也可以在脚本上双击，像运行其他应用程序一样。它还是一个交互 shell，可以执行任意的语句和表达式。这一点对调试、快速组建和测试相当有用。我甚至知道一些人把 Python 的交互 shell 当作计算器来使用！

在您的计算机平台上启动 Python 的交互 shell，接下来让我们尝试着做些操作：

## 例 1.5\. 初次使用交互 Shell

```py
>>> 1 + 1               
2
>>> print 'hello world' 
hello world
>>> x = 1               
>>> y = 2
>>> x + y
3 
```

|  |  |
| --- | --- |
| [1] | Python 的交互 shell 可以计算任意的 Python 表达式，包括任何基本的数学表达式。 |
| [2] | 交互 shell 可以执行任意的 Python 语句，包括 **print** 语句。 |
| [3] | 也可以给变量赋值，并且变量值在 shell 打开时一直有效 (一旦关毕交互 Sheel，变量值将丢失)。 |

# 1.9\. 小结

# 1.9\. 小结

您现在应该已经安装了一个可以工作的 Python 版本了。

根据您的运行平台，您可能安装有不止一个 Python 版本。那样的话，您需要知道 Python 的路径。若在命令行简单地键入 **python** 没有运行您想使用的 Python 版本，则需要输入想要的版本的全路径。

最后祝贺您，欢迎来到 Python 世界。