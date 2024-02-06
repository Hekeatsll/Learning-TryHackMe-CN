# Burp Suite：The Basics(Burp Suite基础·更新版)

本文相关的TryHackMe实验房间链接：https://tryhackme.com/room/burpsuitebasics

本文相关内容：介绍如何使用 Burp Suite 进行 Web 应用程序渗透测试。

![image-20231219185445169](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231219185445169.png)

## 简介

本文旨在了解Burp Suite web应用程序安全测试框架的基础知识。文章内容将重点围绕以下几个主要方面展开：

* 全面介绍Burp Suite；
* 全面概述Burp Suite框架中可用的各种工具；
* 有关在系统上安装Burp Suite的过程的详细指南(正版安装手段)；
* 导航和配置Burp Suite。

我们还将介绍Burp Suite框架的核心，即Burp Proxy(Burp 代理)。

本文所涉及的内容主要是关于Burp Suite的基础知识，所以文中的理论知识部分会更多一些。

## 什么是Burp Suite？

从本质上讲，Burp Suite是一个基于Java的框架，旨在作为进行Web应用程序渗透测试的综合解决方案。Burp Suite已经成为针对Web和移动(mobile )应用程序包括依赖于应用程序编程接口(APIs)的应用程序进行实际安全评估的行业标准工具。

简而言之，Burp Suite可以捕获并操作浏览器和Web服务器之间的所有HTTP/HTTPS流量，这一基本功能构成了Burp Suite框架的主干。在拦截web请求之后，用户可以灵活地将它们发送到Burp Suite框架内的各种组件中，我们将在下文内容中对此进行探讨。

Burp Suite可以在Web请求到达目标服务器之前拦截、查看和修改Web请求(requests)，甚至可以在浏览器接收到响应之前对响应消息(responses)进行操作，这种能力使得Burp Suite成为手动测试Web应用程序的宝贵工具。

![image-20231219214927327](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231219214927327.png)

Burp Suite有不同版本，在本文中我们将重点使用和介绍Burp Suite社区版，该版本可以在法律规定的范围内免费用于任何非商业用途；此外，Burp Suite还为用户提供了专业版和企业版，这两个版本都需要用户支付一定的许可费用，但与此同时它们也拥有一些强大的额外功能：

1.  Burp Suite专业版——我们可以把Burp Suite专业版理解为不受限制的Burp Suite社区版本，它具有以下功能：

    * 能够作为一个自动化的漏洞扫描器（拥有Scanner功能模块）；
    * 能够作为一个不受速率限制的模糊器(fuzzer)/暴力破解攻击器(bruteforcer)；
    * 能够保存项目以供将来使用，并且能够生成报告；
    * 拥有允许与其他工具集成的内置API；
    * 能够不受限制地访问并添加新的扩展模块，从而让Burp Suite获得更强大的功能；
    * 可以访问Burp Suite Collaborator(协作器)，从而能有效地提供一个自托管或运行在Portswigger所拥有的服务器上的独特的请求捕获器。

    简而言之，Burp Suite Pro是一个非常强大的工具，这就是为什么它的一年订阅费为319英镑/ 399美元；Burp Pro通常供专业人士使用(相关的工具许可证通常可由渗透测试人员的雇主提供)，是Web渗透测试领域专业人士的首选。
2.  Burp Suite企业版——与社区版和专业版相比，Burp Suite企业版主要用于持续扫描，它能提供一个自动扫描器，从而可以定期扫描目标web应用程序是否存在漏洞，这类似于使用Nessus等工具来执行自动基础设施扫描；其他版本的Burp Suite通常是允许用户在本地计算机上执行手动渗透测试，而企业版的Burp则不同，它驻留在服务器上，并能不断扫描目标web应用程序是否存在潜在漏洞。

    ![image-20231219221205712](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231219221205712.png)

### **答题**

请阅读本小节内容并回答以下问题：

![image-20230407083518499](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230407083518499.png)

## Burp Suite社区版的功能

虽然与Burp Suite专业版对比，Burp Suite社区版提供的功能集更为有限，但它仍然提供了一系列可用的优秀功能模块，主要有：

* Proxy(代理)：Proxy允许我们在与Web应用程序交互时拦截、查看和修改web请求/响应(requests/responses)；它是一个能够拦截 HTTP/HTTPS消息的代理服务器，能够作为一个在浏览器和目标Web应用程序之间的中间人。
* Repeater(中继器、转发器、重放器)：Repeater允许我们在捕获、修改web请求之后，实现多次重新发送同一Web请求；这个模块的功能绝对是无价的，特别是当我们需要通过反复试验来构建payload(如测试SQL注入漏洞)或者在测试端点的功能缺陷时。
* Intruder(入侵者、攻击器)：虽然在Burp Suite社区版中严格限制了发送请求的速率，但Intruder仍允许我们(在速率限制内)向目标端点喷洒请求；这个功能通常用于暴力破解攻击或者对端点进行模糊测试。
* Decoder(解码器)：这个模块能够充当一个解码器以帮助转换数据格式，Decoder既可以解码Burp Suite所捕获的信息，又能在将payload发送到目标之前对其进行编码；虽然其他工具也可以完成数据格式转换，但直接在Burp Suite中使用Decoder来完成这项工作会更加高效。
* Comparer(对比器、比较器)：这个模块可以充当一个比较器，Comparer允许我们在字符级别或字节级别上比较两个数据片段；虽然这项工作也能通过使用其他工具来完成，但是使用Burp Suite的Comparer模块来完成"数据比较"会更加高效。
* Sequencer(定序器)：这个模块能够充当定序器，我们通常可使用Sequencer来评估令牌(tokens)的随机性，比如评估会话的cookie值或者其他假定随机生成的数据值；如果目标Web应用程序用于生成这些值的算法缺乏安全随机性，那么就可能会向攻击者暴露一些毁灭性的攻击途径。
* Spider(蜘蛛爬虫)：Spider能够充当网络爬虫，它能用于爬取目标web应用程序的内容信息和功能信息。

除了内置功能模块之外，我们还可以编写一些扩展来增加Burp Suite框架的功能，我们通常可以用Java、Python(使用Java [Jython](https://www.jython.org/)解释器)、Ruby(使用Java [JRuby](https://www.jruby.org/)解释器)来编写Burp Suite扩展。

通过Burp Suite的Extender模块可以快速轻松地将扩展加载到Burp框架中，Extender模块还提供了一个可以下载第三方模块的商店(被称为BApp Store)。虽然许多扩展都需要Burp专业版的许可证才能进行下载和添加，但仍有相当数量的扩展可以与Burp社区版免费集成，例如，我们可以使用[Logger++扩展](https://github.com/nccgroup/LoggerPlusPlus)来强化Burp Suite的内置日志功能。

### **答题**

通过阅读本小节内容，可帮助我们回答以下问题。

![image-20231221000858388](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231221000858388.png)

## 安装Burp Suite社区版

Burp Suite是一种非常有用的工具，无论是用于web或移动应用程序评估、渗透测试、漏洞赏金狩猎，还是用于在Web应用程序开发时进行功能调试，随后我们将介绍在不同计算机系统平台上安装Burp Suite的指南。

**下载**

对于大多数操作系统，要下载最新的Burp Suite社区版，通常可以通过网页直接访问下载链接：https://portswigger.net/burp/releases/

Kali Linux操作系统：Burp Suite通常是预装在Kali Linux中的，如果你的Kali中缺少该工具，你可以轻松地使用命令从Kali apt 存储库中获取并安装它。

macOS和Windows操作系统：对于其他操作系统，Burp Suite的开发商[PortSwigger](https://portswigger.net/)在[相关下载页面](https://portswigger.net/burp/releases/)中提供了Community版和Professional版的专用安装程序，我们可以在下拉菜单中选择我们的操作系统类别，然后选择社区版进行下载即可。

tips：[PortSwigger](https://portswigger.net/)官网能为多种操作系统提供专用的Burp Suite安装程序，因为Burp Suite是一个Java应用程序，所以它还可以作为JAR归档文件被下载，并且能够在任何支持Java运行时环境的计算机上有效运行。

![image-20231220105934069](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231220105934069.png)

**安装**

在完成Burp Suite下载之后(建议验证已下载文件的完整性)，我们需要使用适合我们系统的方法来安装Burp Suite。

在Windows操作系统中，我们可以直接运行相关可执行文件来进行Burp安装；在Linux系统中，我们通常需要在终端界面执行相关脚本(命令中可能需要带`sudo`)来安装Burp。

注意:如果在Linux系统中安装Burp Suite，你可以自主选择使用或者不使用root用户权限来进行安装；如果你决定在执行脚本时不使用`sudo`, 那么Burp Suite将安装在你的主目录`~/BurpSuiteCommunity/BurpSuiteCommunity`中，并且相关的安装路径不会被添加到系统的`PATH`变量中。

Burp Suite的安装向导非常直观，无论你所使用的操作系统是什么，接受默认的安装建议通常会是安全的选择，但是仔细阅读安装程序说明也仍然不失为明智之举。

成功安装了Burp Suite之后，我们就可以启动该应用程序，在下一小节中，我们将探讨Burp suite的初始设置和配置。

## Burp Suite Dashboard(仪表面板)

当我们打开Burp Suite并接受相应的默认条款时，我们会看到一个项目选择窗口：在Burp Pro中，此窗口将允许我们将新项目保存到磁盘，或者选择加载先前已保存的项目；然而，在Burp Suite社区版中，我们在这里所能做的就是点击"下一步"按钮。

![image-20231220212308871](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231220212308871.png)

接下来的窗口将允许我们为Burp Suite选择配置，在大多数情况下，保持默认配置值即可。

![image-20231220212342541](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231220212342541.png)

接着我们点击上图中的"Start Burp"，等待一段时间后将自动打开Burp Suite的主界面，第一次打开Burp Suite时，我们可能会看到一个带有教学性质的Learn页面，后继打开Burp Suite时，默认出现的会是Burp Suite Dashboard(仪表面板)界面：

![image-20231220212644636](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231220212644636.png)

![image-20231220212851065](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231220212851065.png)

简而言之，Dashboard界面分为四个象限区域：

![image-20230407134132092](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230407134132092.png)

1. **Tasks**(任务)：左上角的Tasks菜单允许我们定义后台任务，这样当我们使用Burp Suite时会在后台执行既定任务；在Burp Suite社区版本中，我们可以使用默认的"Live passive crawl-动态被动爬取"任务来自动记录我们访问过的web页面，而Pro版本还允许我们在Task菜单中创建按需扫描任务。
2. **Event log**(事件日志)：左下角的Event log会告诉我们关于Burp Suite已执行的操作的信息，例如启动代理，以及有关通过Burp建立的连接的详细信息。
3. **Issue Activity**：右上角的Issue Activity部分专属于pro版本，当我们使用Burp Suite社区版时，我们将无法使用该部分；在专业版本中，Issue Activity部分会列出自动扫描器发现的所有漏洞，这些漏洞会根据严重程度进行排序，并会根据Burp Suite对"目标组件易受攻击"的确定程度进行过滤。
4. **Advisory**：右下角的Advisory部分提供了关于已识别的漏洞的更多详细信息，包括参考资料、建议的漏洞修复措施，我们可以将这些信息导出到一个报告中；此部分在Burp Suite社区版中可能不会显示任何关于已识别的漏洞的信息。

我们可以点击Issue Activity部分中的一个示例漏洞，然后Advisory部分就会出现一些和示例漏洞相关的信息：

![image-20230407144742713](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230407144742713.png)

在Burp Suite的各个选项卡和窗口中，你会发现有一个帮助图标：内含一个问号的圆圈标志。

![image-20230407141157666](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230407141157666.png)

点击帮助图标将打开一个包含对应部分帮助信息的新窗口，例如：

![image-20230407155054946](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230407155054946.png)

如果你不知道Burp Suite中某个功能是干什么的，查看相关的帮助信息可能会非常有用。

### 答题

请阅读本小节内容并回答以下问题：

![image-20231220112609240](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231220112609240.png)

## Burp Suite界面导航

在默认情况下，Burp Suite的界面导航主要可以通过顶部菜单栏来完成，它允许我们在不同模块之间进行切换，并能让我们访问每个模块内的各种子选项卡，子选项卡会出现在模块主菜单栏正下方的小菜单栏中：

![image-20231220213216645](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231220213216645.png)

Burp Suite界面导航包括：

1.模块选择：顶部菜单栏将显示Burp Suite中的可用模块，我们可以通过单击顶部菜单栏中的模块选项卡按钮来在每个模块之间进行切换。

2.子选项卡选择：如果我们所选定的功能模块有多个子选项卡，那么我们可以通过模块主菜单栏正下方出现的小菜单栏来访问它们；这些子选项卡通常包含一些特定于功能模块的设置和选项，例如我们可以在Burp Proxy模块中选择Proxy Intercept子选项卡。

![image-20231220214233620](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231220214233620.png)

3.隐藏或分离选项卡：在较新版本的Burp Suite中，如果我们希望单独查看某一选项卡，那么我们可以用鼠标右键单击指定的模块选项卡，然后选择将该选项卡分离到单独的窗口中；如果我们想隐藏某一选项卡，也可以用鼠标右键单击指定的模块选项卡，然后选择将该选项卡隐藏起来。

![image-20231220220200150](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231220220200150.png)

除了顶部的模块选择菜单栏之外，在Burp Suite中还可以使用快捷键来快速导航到一些关键的模块选项卡中：

![image-20230407143515687](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230407143515687.png)

### 答题

请阅读本小节内容并回答以下问题：

![image-20231220214315696](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231220214315696.png)

## Burp Suite设置选项

在开始学习Burp Proxy之前，让我们先看一下可用于配置Burp Suite的设置选项(options)。

* **Global Settings**(全局设置，也称为用户设置)：这些设置会影响整个Burp Suite，并会在每次启动Burp应用程序时应用，它们将为我们的Burp Suite环境提供基准配置，这些设置可以在Settings选项卡下面的"User-用户"选项中找到；
* **Project Settings**(项目设置)：这些设置特定于当前项目，并且仅在会话期间应用，但是因为Burp Suite社区版并不支持保持项目，所以当我们关闭Burp时，任何特定于项目的选项设置都将会自动丢失，这些设置可以在Settings选项卡下面的"Project-项目"选项中找到。

_tips：在较新版本的Burp Suite中，User选项和Project选项都在顶部菜单栏中的Settings选项卡的相关界面下。_

简而言之：Global Settings将适用于我们每次打开Burp Suite时，而Project Settings则只能应用于某个具体的Burp项目；鉴于我们并不能在社区版的Burp Suite中保存Burp项目，我们的所设置的Project Settings将会在每次关闭Burp Suite时自动重置。

要访问Burp Suite的设置界面，我们需要单击顶部导航栏中的**Settings**按钮，这将为我们打开一个单独的设置窗口：

![image-20231220220938499](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231220220938499.png)

下图显示了单独的设置窗口。

![image-20231220221028867](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231220221028867.png)

在设置窗口中，我们可以在页面左侧找到一个菜单，此菜单允许我们在不同类型的设置之间切换，包括：

1. Search(搜索)：允许我们使用关键字搜索特定设置。
2. Type filter(类型过滤器)：过滤关于用户和项目选项的设置。
   * User：显示影响整个Burp Suite应用程序的设置，即用户设置。
   * Project：显示特定于当前项目的设置，即项目设置。
3. Categories(类别)：允许我们按类别选择设置。

![image-20231220232135471](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231220232135471.png)

值得注意的是，Burp Suite 中的许多功能模块都提供了有关特定类别设置的快捷方式，例如，Proxy模块的子选项卡包括了一个Proxy settings按钮，点击该按钮可以直接打开关于Proxy部分的设置窗口。

![image-20231220232748507](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231220232748507.png)

虽然User选项中的用户设置适用于全局(即这些设置能作用于整个Burp Suite——而不仅仅是某个Burp项目)，但是许多用户设置都可以被Project选项中的对应设置所覆盖。

User选项中的常用设置(在较新版本的Burp Suite中不止这些)：

* Connections：这个设置的内容允许我们控制Burp Suite如何连接到目标，例如，我们可以为Burp Suite设置一个代理来连接目标。
* TLS：这个设置允许我们启用和禁用各种TLS(传输层安全)选项，并且能够给我们提供一个可以上传客户端证书的地方(如果某个web应用程序需要我们使用证书来建立连接)。
* Hotkeys：键盘绑定列表(热键)，此设置可以允许我们查看和更改Burp Suite所使用的键盘快捷键，使用键盘绑定可以大大加快我们的工作流程。
*   Display：这个设置允许我们改变Burp Suite的外观，包括更改字体大小、为Burp设置主题(如黑暗模式)、是否选择隐藏Learn选项卡以及控制Burp在高分辨率显示器上的外观。

    ![image-20231220235531972](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231220235531972.png)

Project 选项中的常用设置(在较新版本的Burp Suite中不止这些)：

* Sessions：此处的设置内容可为我们提供处理会话的选项，它允许我们定义Burp Suite如何获取、保存和使用它从目标站点接收到的会话cookie，它还允许我们定义宏(Macros)，我们可以用这些宏(Macros)来自动化一些事情(比如登录到web应用程序)。
* Connections：该设置的作用和User选项中的对应设置类似，并且此部分设置可以覆盖User选项中的对应设置，例如你可以仅为当前项目设置一个代理，这能够覆盖你在User选项中的任何代理设置；此外，此部分内容中还有一些额外选项，例如它包含了"Hostname Resolution-主机名解析"选项(允许你在Burp Suite中将域名映射为IP地址)。
* TLS：此部分中的设置内容能够覆盖Burp应用程序范围内的TLS设置，并且可以展示用于我们访问过的站点的一个公共服务器证书列表。
*   HTTP：此处的设置内容定义了Burp Suite将如何处理HTTP协议的各个方面，例如，是否会跟随重定向或者如何处理不寻常的响应代码。

    ![image-20231220235713841](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231220235713841.png)

### **答题**

查看Burp Suite社区版界面并回答以下问题：

_**问题1：你可以在哪个类别的设置中找到对"Cookie jar"的引用？**_

![image-20231221000553113](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231221000553113.png)

> Sessions

_**问题2：你可以在哪个基本类别的设置中找到控制Burp Suite更新行为的“Updates”子类别？**_

![image-20231221000649076](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231221000649076.png)

> Suite

_**问题3：允许你更改Burp Suite中快捷键的键盘绑定的子设置类别的名称是什么？**_

![image-20231221000748648](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231221000748648.png)

> Hotkeys

![image-20231221000426081](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231221000426081.png)

## Burp Proxy(代理)介绍

Burp Proxy(代理)是BurpSuite中最基本也是最重要的功能模块，它能够捕获用户客户端和目标Web服务器之间的请求消息和响应消息，然后，我们就可以对这些被拦截的流量进行操作，我们可以将这些数据发送到Burp Suite的其他功能模块做进一步处理，最后我们可以再允许将这些数据继续发往目的地。

例如，如果我们通过Burp Proxy(代理)向https://tryhackme.com发出请求，然后我们的请求将被捕获，在我们明确允许请求消息通过之前，该请求将无法继续发送到TryHackMe服务器（我们也可以选择对来自服务器的响应消息执行类似的捕获操作）。

**了解Burp Proxy的要点：**

*   拦截请求：在开启拦截功能之后，当Web请求经过Burp Proxy发出时，它们将会被拦截并会被阻止到达目标服务器，然后被拦截的请求会出现在Proxy选项卡的界面中，从而允许我们针对其做进一步操作，例如转发、删除、编辑或者将它们发送到其他Burp模块中或者将请求的内容复制为cURL命令并保存为文件；如果你想要禁用拦截功能并允许Web请求能够不间断地通过Proxy，则可以尝试将拦截功能关闭，我们用鼠标左键单击 `Intercept is on` 按钮即可关闭Proxy的拦截功能。

    ![image-20231221162928614](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231221162928614.png)

    tips：在Proxy(代理)激活并开启拦截功能之后，我们接着使用浏览器向TryHackMe网站发出一个请求，此时，发出请求的浏览器将会挂起，该请求消息则会出现在Proxy选项卡的界面中，我们可以看到如上图所示的画面。
* 控制流量：Burp Proxy这种拦截请求的能力使得渗透测试人员能够完全控制与目标服务器相关的网络流量，这对于测试web应用程序来说是非常宝贵的。
* 捕获和记录：在默认情况下，Burp Suite会捕获并记录通过Proxy发出的请求，即使在关闭拦截状态下也是如此；这种日志记录功能可以帮助我们以后进行内容分析以及审查先前的请求消息。
* 支持处理WebSocket：Burp Suite还能捕获并记录WebSocket通信，这在我们分析Web应用程序时能够提供额外的帮助。
*   日志和历史记录：已经捕获的请求消息可以在HTTP history和WebSockets history子选项卡的界面中查看，这能允许我们进行回顾性分析，以便根据需要将已捕获的请求发送到其他Burp功能模块中。

    ![image-20231221164222623](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231221164222623.png)

我们还可以通过单击Proxy settings按钮(该按钮可以在Proxy选项卡正下方的子选项卡一栏中找到)来访问特定于Proxy功能模块的设置选项，这些选项能够为我们提供对Proxy的行为和功能的广泛控制，熟悉这些设置选项可以帮助我们优化Burp Proxy的使用。

**在Proxy Settings中，一些值得注意的功能：**

*   响应拦截：在默认情况下，Proxy不会主动拦截来自于目标服务器的响应消息，除非我们在每个Web请求的基础上进行明确要求；我们可以使用Proxy设置界面中的"Intercept responses based on the following rules-基于以下规则拦截响应"复选框以及一些已定义的规则来更加灵活地设置响应拦截功能。

    ![image-20231221165316158](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231221165316158.png)

    tips：我们可以勾选"基于以下规则截取响应"复选框并选择其中的一个或多个规则来覆盖默认设置，例如配置"`Or` `Request` `Was interrupted`"规则，此规则可用于捕获被Proxy拦截的所有请求的相关响应，除了上图的规则配置外，我们也可以制定自己的规则来控制Proxy的操作方式。
* 匹配和替换：Proxy设置界面中的“匹配和替换”部分允许我们使用正则表达式(regex)来修改传入和传出的Web请求，该功能允许我们进行一些动态更改，修改用户代理或操作cookie等；例如，我们可以自动更改用户代理以便在传出请求中模拟不同的web浏览器，或者删除传入请求中设置的所有cookie，同样，你也可以在这部分设置中使用自定义规则。

## 配置常规流量代理(FoxyProxy)

要使用Burp Suite代理，我们还需要配置本地web浏览器，如此才能尝试通过Burp Suite重定向流量。

在本小节中，我们将重点介绍如何使用Firefox浏览器中的FoxProxy扩展插件来配置Burp代理。

有两种方法可以通过Burp Suite代理我们的常规流量：

1. 使用Burp Suite的内置嵌入式浏览器(本小节对此不做赘述)；
2. 配置本地web浏览器，然后通过Burp代理我们的流量(此方法更加常用，本小节主要介绍这种方法)。

在本地Kali机中，Burp Proxy（代理）默认会通过在`127.0.0.1:8080`端口上打开Web界面来开展相关工作，所以，我们需要通过8080端口重定向所有的本地浏览器流量，然后才能开始使用Burp拦截流量；我们可以通过改变浏览器设置来做到这一点，或者使用一个名为[FoxyProxy](https://getfoxyproxy.org/)的浏览器扩展进行配置，[FoxyProxy](https://getfoxyproxy.org/)插件允许我们保存代理配置文件，这意味着我们可以很方便地切换到我们的"Burp Proxy"配置文件——通过点击勾选代理配置文件，就能快速启用或关闭Burp代理。

FoxyProxy插件有两个版本：基础版和标准版。这两个版本都允许我们更改代理设置，然而，FoxyProxy标准版能够更好地控制什么流量可通过代理发送；例如，标准版的FoxyProxy将允许你设置模式匹配规则来确定某些Web请求是否应该被代理——这比FoxyProxy基础版所提供的简单代理设置要复杂得多。

使用FoxyProxy基础版对我们来说绰绰有余，FoxyProxy是通过_**Firefox浏览器**_来安装和配置的，相关的插件下载链接如下：

> https://addons.mozilla.org/en-US/firefox/addon/foxyproxy-basic/
>
> https://addons.mozilla.org/zh-CN/firefox/addon/foxyproxy-basic/

如果想使用谷歌浏览器来完成本小节的代理配置，请使用以下插件(配置方法与在Firefox浏览器中使用FoxyProxy插件类似)：

> https://chromewebstore.google.com/detail/padekgcemlokbadohgkifijomclgjgif?hl=zh-CN\&utm\_source=ext\_sidebar

以下是在Firefox浏览器中使用 FoxyProxy 配置 Burp Suite 代理(Proxy)的步骤：

1. 安装 FoxyProxy：下载并安装[FoxyProxy Basic 扩展](https://addons.mozilla.org/zh-CN/firefox/addon/foxyproxy-basic/)。
2.  访问FoxyProxy插件中的选项：在Firefox浏览器中完成插件安装之后，浏览器的界面右上角会出现一个插件图标按钮，这将允许我们访问代理配置(要点击右上角的按钮，才能看到如下界面)：

    ![image-20230408054258418](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230408054258418.png)
3.  在FoxyProxy插件中创建 Burp 代理配置：FoxyProxy中没有默认配置(没有可供选择的初始配置)，我们可以点击上图中的"Options-选项"按钮来创建我们的代理配置，这将打开一个新的浏览器选项卡：

    ![image-20230408054445834](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230408054445834.png)
4.  添加代理配置的详细信息：接着我们点击上图中的"Add-添加"按钮，并在"Add Proxy"页面填写以下值：

    Title：`Burp`（或者填写其他你想设置的名称）

    Proxy Type：`HTTP`

    Proxy IP：`127.0.0.1`

    Port：`8080`

    ![image-20230408055342297](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230408055342297.png)
5. 保存配置：单击上图中的“Save保存”按钮以保存我们的 Burp 代理配置(这里的Burp是我们的配置名称，见上图)。
6.  激活插件中的代理配置：现在点击Firefox浏览器顶部的FoxyProxy插件图标，就会看到有一个可用于Burp的配置，如果我们单击勾选下图中的"Burp"配置，那么本地浏览器就可以开始通过`127.0.0.1:8080`重定向常规流量(注意，当我们激活插件中的"Burp"配置时，Burp Suite必须正在运行状态)。

    ![image-20231221194435130](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231221194435130.png)

    tips：如果Burp Suite并未启动运行，那么本地浏览器将无法在插件中的"Burp"代理配置激活时发出任何web请求，即FoxyProxy插件中的代理配置选项需要和Burp Suite应用程序中的Proxy模块配合使用。
7.  在 Burp Suite 中启用代理(Proxy)拦截：成功激活浏览器插件中的Burp代理配置选项之后，让我们切换到Burp Suite应用程序中，此时我们要确保Proxy模块中的拦截选项是打开的。

    ![image-20231221194408340](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231221194408340.png)
8. 测试代理(Proxy)：现在，当我们尝试在本地Firefox浏览器中访问 http://MACHINE\_IP/ 目标主页时(这会向目标站点发出一个请求)，我们的本地浏览器会被自动挂起，同时我们的代理将填充相关的HTTP请求，最终我们在本地浏览器中发出的Web请求将会被Burp Suite所拦截。

请记住以下几点：

* 当插件中的代理配置激活，并且在Burp Suite中打开了Proxy的拦截选项时，那么无论何时发出Web请求，本地浏览器都会挂起。
* 注意不要在无意中打开Burp Proxy的拦截选项，因为它会阻止浏览器发出任何请求。
* 在Burp Suite中右键单击已经被拦截的请求消息时，将允许我们执行各种特定操作，例如转发请求、删除请求以及发送该请求消息到Burp Suite的其他功能模块中等等。

注意：在启用Burp中的拦截功能之前，请考虑关闭本地浏览器中的其他网页选项卡，否则我们将在Burp中收到一些并非来自于目标服务器的其他WebSocket请求。

### **答题**

在本地Kali机上查看BurpSuite界面：

![image-20231221195429776](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231221195429776.png)

![image-20230408061453610](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230408061453610.png)

## 配置HTTPS流量代理

通过上一小节中的配置，我们已经可以拦截HTTP流量——接下来，我们需要尝试拦截HTTPS流量。

在我们使用Burp Suite拦截HTTP流量时，如果我们导航到启用了TLS(也就是使用了HTTPS协议)的站点会发生什么？例如 https://google.com/ ：

![image-20230408062352280](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230408062352280.png)

如上图所示，我们会得到一个错误提示，具体而言，Firefox会告诉我们Portswigger证书颁发机构(CA)没有被授权保护此连接。

为了解决上述问题，我们需要让Firefox浏览器信任Portswigger证书有权保护HTTPS连接，因此我们可以手动将此CA证书添加到浏览器的受信任证书颁发机构列表中，操作方法如下：

1.  下载CA证书：首先，在Burp代理配置激活(未开启拦截功能)的时候访问 http://burp/cert ，这将下载一个名为`cacert.der`的文件，我们将此文件保存到计算机上的某个位置。

    ![image-20230408063455946](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230408063455946.png)
2.  访问 Firefox 证书设置：在 Firefox URL 栏中输入`about:preferences`并按下Enter键，这将把我们带到FireFox的设置页面，然后我们再在设置页面中搜索"certificates-证书"，然后单击"View certificates-查看证书"按钮。

    ![image-20230408063727051](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230408063727051.png)

    ![image-20230408063750188](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230408063750188.png)
3.  导入CA证书：在"Certificate Manager-证书管理器"窗口，单击“Import-导入”按钮，然后选择我们在前述步骤中所下载的`cacert.der`文件。

    ![image-20230408064739349](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230408064739349.png)

    ![image-20230408064716405](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230408064716405.png)
4.  设置信任CA证书：在随后出现的窗口中，选中"Trust this CA to identify websites-信任此CA以识别网站"框，然后点击"OK"按钮。

    ![image-20231224114412746](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231224114412746.png)

通过完成上述步骤，我们已经将PortSwigger CA证书添加到了受信任的证书颁发机构列表中；现在，我们应该能够访问任何启用TLS的站点，而不会遇到证书错误。

如下图所示，我们现在可以在激活了Burp代理配置(未开启拦截功能)的情况下，自由地访问任何启用了TLS(也就是使用了HTTPS协议)的网站。

![image-20230408065356294](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230408065356294.png)

## Burp Suite内置浏览器

除了修改我们的本地Web浏览器以使用Burp Proxy(代理)之外，Burp Suite还包括了一个内置的Chromium浏览器，该浏览器已经预先配置好能够使用Burp代理，而无需我们再进行任何修改。

我们可以通过Burp Proxy(代理)模块的"Intercept"子选项卡中的"Open Browser-打开浏览器"按钮来启动Burp Suite内置浏览器：

![image-20231221205704324](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231221205704324.png)

点击"Open Browser-打开浏览器"按钮之后，将会自动弹出一个Chromium窗口，我们在这个浏览器中发出的任何请求都将自动通过Burp Proxy(代理)。

![image-20231221205823389](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231221205823389.png)

注意：在项目选项和用户选项的设置中有许多与Burp Suite内置浏览器相关的设置，请根据需要探索和定制它们。

如果我们在Linux上以root用户运行Burp Suite，则可能会遇到由于无法创建沙箱环境而导致Burp Suite内置浏览器无法启动的错误。

![image-20230408070918316](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230408070918316.png)

对此有两个简单的解决方案：

* 明智的选择：我们可以在Linux上创建一个新用户，并在低权限帐户下运行Burp Suite，这将允许Burp Browser运行而不会出现问题。
*   简单的选择：我们可以进入"Settings"->" Tools"->"Burp's browser"，并勾选"Allow Burp's browser to run without a sandbox-允许Burp's浏览器在没有沙箱环境的情况下运行"，启用此选项将允许Burp Browser在root用户权限下启动，但请注意，出于安全原因，默认情况下该选项是被禁用的。

    ![image-20231221210956217](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231221210956217.png)

## Target和Scoping

本小节我们将查看使用Burp Proxy时的一个很重要的部分——Scoping(范围界定)。

我们可能不需要使用Burp Proxy(代理)来记录浏览器上的所有流量，当选择让Burp记录所有流量内容时会包含很多和目标网站无关的流量信息，这就可能会混淆我们在Burp Suite中所记录的日志内容；简而言之，如果我们只想关注特定的Web应用程序，那么我们应该让Burp Proxy(代理)记录一定范围内的流量而并非全部流量。

因此，在必要的情况下，我们需要使用Scope来控制Burp Proxy对于流量的记录范围。

通过在Burp Suite的项目(Project)设置中修改范围(Scope)，我们可以定义在Burp Suite中代理(Proxy)和记录的内容，从而限制Burp Suite仅针对我们想要测试的特定Web应用程序；相关步骤如下：我们先切换到`Target`选项卡，右键单击Site map左侧列表中的目标站点，并且选择`Add To Scope`，然后Burp将提示我们是否要停止记录范围(scope)之外的任何内容——我们选择`yes`即可。

![image-20231222182632284](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231222182632284.png)

![image-20231222182739191](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231222182739191.png)

现在我们切换到Target选项卡下的"Scope settings"子选项卡中，然后就可以检查流量记录范围(可以看到我们刚才添加的站点)：

![image-20231222182843961](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231222182843961.png)

如上图所示：Scope settings子选项卡允许我们通过包含或者排除“域名/ip地址”来控制Burp Proxy(代理)所记录的流量范围。

我们刚才只是选择禁用范围外流量的日志记录，但是当我们启用Burp代理的拦截功能时，仍然会拦截浏览器中的所有流量；如果要关闭拦截全部流量的功能，我们需要转到Proxy settings子选项卡，然后在"Intercept Client Requests-拦截客户端请求"部分中配置"`And` `URL` `Is in target scope`"规则，这样Burp仅会拦截目标范围内的流量。

![image-20231222183331689](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20231222183331689.png)

按照上图选项成功配置流量拦截规则之后，Burp代理将完全忽略在已定义的范围之外的任何流量内容(既不记录范围外的流量日志，也不拦截范围外的浏览器请求)，从而能在Burp Suite中为我们提供更清晰的流量视图。

tips：我们可以将`http://MACHINE_IP/`(目标站点)添加到我们的Burp Suite测试范围内，并且更改Burp Suite中的代理(Proxy)设置以实现仅拦截流向范围内目标的流量。

## Site Map和Issue Definitions

使用Scope settings子选项卡可以控制Burp Suite的测试范围，这可能是Target选项卡中最有用的部分，但是Burp Suite中的Target选项卡不仅仅能提供对测试范围的控制，它一共由三个子选项卡组成：

*   Site map(站点地图)： 这个子选项卡允许我们以树形结构绘制目标web应用程序，我们在Burp Proxy处于活动状态时所访问的每个页面都会显示在站点地图的列表中，因此，此功能使我们能够通过简单地浏览Web应用程序的页面来自动生成相关的站点地图；在Burp专业版中，我们还可以使用站点地图来针对目标执行自动爬取，探索页面之间的链接并绘制出尽可能多的网站内容，即使使用的是Burp社区版，我们仍然可以在初始信息枚举时使用站点地图来积累数据；这个功能对于APIs绘制也特别有用，因为目标Web应用程序所访问的任何API端点也都会被捕获并记录在站点地图中。

    tips：当我们访问一个页面时，在页面加载过程中用于检索数据的任何API端点都会显示在Site map中。
* Issue Definitions(问题定义)：在Burp Suite社区版中，虽然我们无法使用完整的Burp Suite漏洞扫描功能，但是我们仍然可以访问Burp扫描器所能够查找的所有漏洞列表；Issue Definitions子选项卡能够为我们提供一个web漏洞列表(包括漏洞的描述和参考)，如果我们需要书写漏洞报告或者描述在手动测试期间可能已识别的特定漏洞，那么我们就可以从Issue Definitions子选项卡中获得一定帮助。
* Scope settings(范围设置)：这个子选项卡能允许我们控制Burp Suite中的目标范围，它使得我们能够包含或排除特定的域名/IP以定义我们的测试范围；通过对目标范围进行管理，我们就可以专注于测试我们的目标Web应用程序，而避免捕获过多的、不必要的流量。

### **答题**

启动目标机器，在攻击机上激活Burp代理插件并且不启用拦截功能，使用本地浏览器访问目标站点( http://10.10.53.186/ )的每个页面，然后检查Burp中Target选项卡下的Site map界面(我们想找到一个可疑端点，并想查看其响应消息)：

![image-20230408095919709](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230408095919709.png)

![image-20230408100013480](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230408100013480.png)

> 找到的flag为：THM{NmNlZTliNGE1MWU1ZTQzMzgzNmFiNWVk} 。

查看Burp Suite中的Issue Definitions列表(它是Target选项卡下的子选项卡)，找到易受攻击的JavaScript依赖项(Vulnerable JavaScript dependency) 所对应的漏洞严重性：

![image-20230408100801502](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230408100801502.png)

> 严重性为：Low

![image-20230408101245071](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230408101245071.png)

## 实例练习

在现实世界的web应用程序中，我们会测试目标站点 是否存在各种各样的漏洞，其中一个漏洞就是跨站点脚本(XSS)漏洞。

简单地理解XSS漏洞：如果我们能将一个客户端脚本(通常是Javascript)注入到目标网站的页面中，并且可以执行已注入的脚本，那么目标网站就存在XSS漏洞。

XSS漏洞有很多种——我们在本小节实例中所测试的XSS类型被称为“反射型”XSS，因为它只会影响发出Web请求的用户。

### **操作**

查看目标站点的support页面： http://10.10.53.186/ticket/

![image-20230408101452114](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230408101452114.png)

![image-20230408101512232](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230408101512232.png)

在上图中的"Contact Email"输入框中，输入以下内容：

```js
<script>alert("Succ3ssful XSS")</script>
```

我们会发现此处有一个过滤器在限制我们的输入内容，该过滤器可以阻止我们添加“任何不允许在电子邮件地址中使用的特殊字符”。

幸运的是，此处的客户端过滤器非常容易绕过，有多种方法可以禁用该过滤器脚本或者我们在一开始就能阻止过滤器脚本加载。

接下来让我们尝试使用Burp Suite来绕过刚才提及的客户端过滤器。

我们打开Burp Suite并启动Burp代理插件，然后开启Burp Proxy中的拦截功能，然后再在浏览器中通过目标站点的Support页面发送请求：

![image-20230408103110742](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230408103110742.png)

成功捕获到请求后，修改email字段为 `<script>alert("Succ3ssful XSS")</script>`，然后使用Ctrl + U快捷键对修改之后的内容进行一次URL编码，以使其稍后可以安全地被发送：

![image-20230408103801145](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230408103801145.png)

我们点击上图中的"Forward"按钮，这将使得浏览器继续向目标站点发送修改之后的请求，如果我们在浏览器页面中得到一个js弹框(代表我们插入的js脚本成功被执行)，那么就说明目标站点确实存在“反射型”XSS漏洞：

![image-20230408104254596](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230408104254596.png)

## 小结\&Pro版下载链接

通过学习本文内容，能够很好地掌握Burp Suite的主要界面内容和选项配置，并可以了解Burp Proxy(代理)的基本工作流程。

关于Burp Suite专业版的第三方获取链接如下(验证：52pj)：

* https://www.123pan.com/s/F2W5Vv-Rk7Vv.html
* https://pan.baidu.com/s/1J\_CUxLKqC0h3Ypg4sQV0\_g#list/path=%2F
