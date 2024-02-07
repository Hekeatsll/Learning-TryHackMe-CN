---
description: 本文相关内容：踏上学习 Linux 基础知识的旅程，学习在交互式终端上运行一些最基本的Linux命令。
---

# Linux Fundamentals Part1(Linux基础知识1)

THM实验房间链接：https://tryhackme.com/room/linuxfundamentalspart1



## 简介

欢迎来到“Linux 基础”系列的第一部分。你很可能使用的是 Windows 或 Mac 机器，这两者在视觉设计和操作方式上都有很多不同，就像 Windows、iOS 和 MacOS 一样，Linux只是另一种操作系统，也是世界上最流行的操作系统之一，Linux系统正在为智能汽车、安卓设备、超级计算机、家用电器、企业服务器等提供动力。

本文将为你提供以下Linux基础内容：

* 在交互式Linux机器中运行你的第一个命令
* 介绍一些用于与文件系统交互的基本Linux命令
* 向你介绍用户和组如何在Linux上工作（以及这对渗透测试人员意味着什么）

## Linux的一些背景知识

**Linux在哪里使用?**

相比于Windows系统，Linux操作系统要轻量级得多，你会惊讶地发现，你每天都有机会以某种形式使用Linux系统，Linux能够支持以下功能：

* 用于搭建用户访问的网站
* 用于车载娱乐信息系统和相关控制面板
* 销售点(PoS-Point of Sale)系统，如商店的收银台
* 关键基础设施，如交通灯控制器或工业传感器

**Linux的特色**

“Linux”这个名字实际上是基于UNIX(另一种操作系统)的多个操作系统的总称，由于UNIX是开源的，所以Linux的变体也各种各样——操作系统的不同用途催生出了不同的Linux变体。

例如，Ubuntu和Debian是比较常见的Linux发行版，Linux的可扩展非常强，也就是说，你可以把Ubuntu作为服务器(比如网站和web应用程序)运行，也可以把它作为一个成熟的个人操作系统运行。在本文中，我们将使用Ubuntu系统进行练习。

tips：Ubuntu服务器可以在只有512MB内存的系统上运行。

tips：类似于不同版本的Windows(7、8和10)，Linux也有许多不同的版本以及发行版。

**答题**

![image-20230402214844561](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402214844561.png)

## 运行简单的Linux命令

正如我们之前所讨论的那样，使用 Ubuntu 等Linux操作系统的关键原因是由于Linux操作系统的轻量级，但是，这并非意味着Linux并非没有缺点，例如，如果我们没有为Linux系统安装GUI，那么在Linux操作系统中通常就没有 GUI（图形用户界面，能提供与机器交互的桌面环境）可供我们使用。

如果我们想要与没有GUI的Linux操作系统发生交互，我们就必须通过“终端”来完成（即使系统安装了GUI，还是免不了依赖于“终端”来和Linux系统进行交互）。

“终端”是纯粹基于文本的，Linux操作系统的“终端”（Terminal）界面类似于下图：

![image-20230402220603219](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402220603219.png)

我们可以通过终端命令来实现一些基本的操作，如导航到文件、输出文件内容以及创建文件等，这些命令的含义有时候是不言自明的。

让我们先了解以下两个基本命令：

* `echo`：输出我们所提供的任何文本内容；
* `whoami`：显示我们当前登录的用户名。

以下是相关命令的示例：

![image-20230402221322991](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402221322991.png)

![image-20230402221333903](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402221333903.png)

**答题**

使用和本文相关的实验房间所提供的Linux虚拟机进行操作：

![image-20230402221632147](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402221632147.png)

![image-20230402221505119](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402221505119.png)

## 使用Linux命令与文件系统交互

我们只介绍了“ echo”和“ whoami”命令，这还不能帮助我们实现导航到文件系统、读取文件内容以及写入内容到文件等操作；所以我们还需要学习更多的命令。

**与文件系统交互**

如前所述，能够在不依赖桌面环境的情况下浏览所登录的Linux机器是非常重要的，我们可以使用以下命令：

* `ls`：该命令的全名是“listing”；
* `cd`：该命令的全名是“change directory”；
* `cat`：该命令的全名是“concatenate”；
* `pwd`：该命令的全名是“print working directory”。

tips：上述四个命令是不言自明的，我们可以从上面四个命令的全名中猜测出各个命令的用途。

**列出当前目录中的文件(ls)**

在我们查找任何文件或文件夹的内容之前，我们需要知道Linux机器的当前目录下存在什么，我们可以使用`ls`命令(ls是listing的缩写)来列出当前目录中的文件。

![image-20230402225034052](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402225034052.png)

在上面的截图中，我们可以看到当前有以下目录/文件夹：

* Important Files
* My Documents
* Notes
* Pictures

根据以上文件夹的名称，你可能会猜测到这些文件夹将包含什么内容。

tips：你可以直接列出某个目录(文件夹)下的内容，而无需事先导航到对应目录，如使用`ls Pictures`命令将直接列出Pictures目录下的内容。

**更改当前目录(cd)**

现在我们知道当前目录下存在哪些文件夹，我们可以使用`cd`命令(change directory的简称)来切换到对应目录中；例如，如果我们想打开“Pictures”目录，就可以执行`cd Pictures`命令来切换目录，然后再使用`ls`命令——即可列出“Pictures”目录下的内容。

![image-20230402230130734](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402230130734.png)

如上图所示，在Pictures目录下有四个jpg图像文件。

**查看文件内容(cat)**

当我们知道文件的存在之后，我们还可能想要查看文件的具体内容，此时我们可以使用`cat`命令，该命令可以简单地查看文本文件(或其他文件)的内容。

“`cat`”是concatating的缩写，该命令可用于输出文件的具体内容(不仅仅是文本文件)。

在下面的截图中，我们可以看到如何使用`ls`和`cat`来查看“Documents”目录下的文件具体内容：

![image-20230402230319216](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402230319216.png)

让我们来分析一下上图内容：

1. 我们首先使用了`ls`命令，这让我们知道这台机器的“Documents”文件夹中有哪些文件可用，如上图中的“todo.txt”文件；
2. 然后，我们使用`cat todo.txt`命令来输出这个“todo.txt”文件的内容，随后显示的内容为“Here's something important for me to do later!”

tips：我们也可以使用`cat /home/ubuntu/Documents/todo.txt`命令直接输出目标文本文件的内容。

用户名、密码、配置设置等敏感信息有时候也可能会存储在文件中，因此，我们可以尝试使用`cat`命令来查看一些可能包含敏感信息的文件的具体内容。

**查找当前工作目录的完整路径(pwd)**

在你使用终端浏览Linux机器的过程中，你当前的工作目录名称可以通过终端提示符得知；但是我们很容易忘记我们当前在文件系统中的确切位置（即当前工作目录的完整路径），这时我们就可以使用`pwd`命令来进行查看，`pwd`表示打印工作目录的完整路径（**p**rint **w**orking **d**irectory）。

假设我们目前在“Documents”文件夹中——但是该文件夹在Linux机器文件系统的哪个具体位置呢?

我们可以使用`pwd`命令进行查看，如下图所示：

![image-20230402223817265](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402223817265.png)

让我们来分析一下上图内容：

1. 根据当前的终端提示符，我们已经知道我们在“Documents”目录中，假设我们不知道“Documents”在系统中的具体位置。
2. 我们可以使用`pwd`(打印工作目录)命令找到这个“Documents”文件夹的完整路径。
3. 在输入`pwd`命令之后，Linux会告诉我们这个“Documents”目录存储在当前机器上的“/home/ubuntu/Documents”路径。
4. 现在，如果我们发现自己在其他目录中，我们就可以使用`cd /home/ubuntu/Documents`将当前工作目录重新更改为“Documents”目录。

**答题**

使用和本文相关的实验房间所提供的Linux虚拟机进行操作：

![image-20230402224702955](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402224702955.png)

![image-20230402224506668](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402224506668.png)

## 查找文件

Linux的一个可取之处是使用它可能会帮助我们提高工作效率，话虽如此，你的效率当然取决于你对Linux命令的熟悉程度；随着时间的推移，当你习惯与操作系统(如Ubuntu)进行交互时，那些常用的Linux命令将开始变成你的肌肉记忆。

在Linux终端界面中，我们可以使用一组命令来快速搜索用户（在整个系统中）可以访问的文件，而不只是一直使用`cd`和`ls`来查找文件位置。

**使用“find”命令**

find命令既可以非常简单地使用，也可以和其他参数或命令组合使用，这取决于你具体想要做什么。

假设我们可以看到如下的一个可用目录列表：

![image-20230403080111411](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230403080111411.png)

1. Desktop
2. Documents
3. Pictures
4. folder1

一个目录可以包含更多的目录或文件，当我们为了寻找特定的文件而不得不查看每一个目录时，这就变得令人头疼了，所以我们可以使用find命令来简化上述过程。

假设我们已经知道我们正在寻找的文件的名称-但我们不知道它的确切位置，比如，我们想找到"password.txt"文件。

如果我们已经知道文件名，我们就可以简单地使用`find -name password .txt`命令，这将在当前目录下的每个文件夹中查找指定的文件，如下所示：

![image-20230403080529617](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230403080529617.png)

如上图所示，find命令已经设法找到目标文件——相关路径是folder1/passwords.txt；但是，如果我们不知道文件的具体名称或者想要搜索每个具有相同扩展名(如“.txt”)的文件，我们应该怎么做？

我们可以简单地使用通配符(`*`)来搜索任何以“.txt”（或者其他扩展名）结尾的文件和目录。

假设我们希望找到当前目录中的每个.txt文件，我们就可以构造`find -name *.txt`命令，这样我们就能够找到当前目录下的每个.txt文件以及它们的具体位置：

![image-20230403081402271](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230403081402271.png)

如上图所示，我们已经成功找到了：

1. 位于./folder1目录下的passwords.txt文件；
2. 位于./Document目录下的todo.txt文件。

**使用“grep”命令**

另一个值得学习的实用程序是`grep`命令的使用，`grep`命令能够允许我们在文件的内容中搜索我们正在寻找的特定值。

以web服务器的访问日志为例，假设某个web服务器的access.log有244条日志记录。

![image-20230403082019508](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230403082019508.png)

在此处，使用像`cat`这样的命令并不能很好地解决内容查找问题，例如，如果我们想搜索这个日志文件，以查看某个用户/IP地址访问了哪些内容，考虑到我们想要找到一个特定的值，所以用`cat`命令直接查看这个包含244个条目的日志文件内容并不是那么有效。

我们可以尝试使用`grep`来搜索一个文件的全部内容，以找到我们想要查找的特定值；以上述的访问日志文件为例，假设我们希望找到IP地址“81.143.211.90”访问过的所有内容(注意，这是虚构的)，我们可以如下操作：使用`grep "81.143.211.90" access.log`命令。

![image-20230403082642585](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230403082642585.png)

如上图所示，我们已经使用`grep`命令搜索了access.log文件，然后终端界面向我们显示了一些条目，这些条目都是在access.log日志文件中与目标IP相关的内容。

tips：“grep”的全称为Global search REgular expression and Print out the line（全局搜索正则表达式并打印成行）

**答题**

使用和本文相关的实验房间所提供的Linux虚拟机进行操作：

```shell
grep "THM" access.log
```

![image-20230403084038773](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230403084038773.png)

![image-20230403084117471](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230403084117471.png)

## Shell操作符介绍

学习Linux操作符能够加强你对Linux命令的了解，有一些重要的操作符值得我们注意。

本小节将简单介绍以下几个基础操作符：

* `&`：此操作符允许你在终端的后台运行命令。
* `&&`：此操作符允许你在终端的一行中组合多个命令。
* `>`：这个操作符是一个重定向器——这意味着我们可以从命令中获取输出(例如使用cat输出一个文件)并将其定向到其他地方，当我们将输出重定向到一个有内容的文本文件时，原文件的文本内容会被自动覆盖。
* `>>`：此操作符执行与`>`操作符相同的功能，但最终效果是附加输出内容到原文件内容的结尾而不是直接替换(这意味着不会覆盖任何内容)。

接下来，让我们简单地介绍一下以上四种操作符。

**操作符 "&"**

这个操作符允许我们在后台执行命令。

假设我们现在想复制一个大文件，这将花费很长时间，并且在复制文件期间我们无法做任何其他事情；因此，我们可以选择使用"&"操作符将命令后台化，该操作符的作用是允许我们执行一个命令并将其放置在后台，而当命令在后台执行时，我们就可以继续利用终端来做其他事情。

**操作符"&&"**

我们可以使用"&&"操作符来组合我们要执行的命令列表，并且使用该操作符组合的每个有效命令都会得到执行，例如`command1 && command2`，但是要注意的一点是：只有在`command1`命令执行成功时，`command2`命令才会随后得到执行。

**操作符 ">"**

这个操作符就是所谓的重定向器。使用该操作符时，我们可以从命令中获取输出内容，并能将该输出内容发送到其他地方。

一个很好的例子是使用">"操作符重定向`echo`命令的输出，假设我们想要创建一个名为，并且要求它的内容为“hey”，我们就可以使用以下命令：`echo hey > welcome`

![image-20230403090231983](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230403090231983.png)

tips：如果上图例子中的“welcome”文件本身就有内容，那么使用上述之后，“welcome”文件的原内容将被“hey”所覆盖。

**操作符 ">>"**

这个操作符其实也是一个输出重定向器，就像我们前面所讨论的操作符(">")一样；然而， ">>"操作符的不同之处在于：它不会覆盖原文件中的任何内容，而是会将输出内容附加到原文件内容的最后。

在前面的例子中，我们有一个包含“hey”内容的文件“welcome”，如果我们使用`echo`命令组合 ">"操作符向文件中添加“hello”，那么“welcome”文件的内容将只有“hello”而没有“hey”（因为使用">"操作符会覆盖原文件的内容）。

当我们使用 ">>"操作符时，重定向的输出内容将附加到原文件内容的末尾，而不会直接替换（覆盖）原文件的内容：`echo hello >> welcome`

![image-20230403090722301](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230403090722301.png)

**答题**

阅读本小节的内容，并且使用和本文相关的实验房间所提供的Linux虚拟机进行操作。

![image-20230403092012788](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230403092012788.png)

![image-20230403092433553](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230403092433553.png)

## 小结

本文所介绍的内容是你在与 Linux 机器交互时将要使用的最基本功能，只要你经常使用这些Linux命令，你将能够很快熟悉这些Linux基础内容。

现在快速回顾一下，本文简单介绍了以下内容：

* 了解关于Linux的一些简单背景知识。
* 与Linux机器进行简单交互。
* 运行一些最基本的Linux命令。
* 介绍如何简单地与Linux中的文件系统进行交互，使用`find`和`grep`等命令来更有效地查找目标文件。
* 通过了解一些 shell 操作符来增强你的命令使用基础。
