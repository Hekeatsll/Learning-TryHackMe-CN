---
description: 本文相关内容：本文所涉及的内容是Windows 基础模块的第 2 部分，了解有关系统配置、UAC 设置、资源监控、Windows 注册表等更多信息。
cover: ../../.gitbook/assets/4094ed0a54f8dc274b9b4f602c57b152.svg
coverY: 0
layout:
  cover:
    visible: true
    size: hero
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Windows Fundamentals 2(Windows基础知识2)

TryHackMe实验房间链接：[https://tryhackme.com/room/windowsfundamentals2x0x](https://tryhackme.com/room/windowsfundamentals2x0x)



## 简介

在Windows Fundamentals 1中，我们已经介绍了Windows的桌面、文件系统、用户帐户控制、控制面板、设置和任务管理器。

本文将继续尝试概述 Windows 操作系统中可用的其他一些实用程序以及访问这些实用程序的不同方法。

启动本文相关实验房间中所附加的Windows虚拟机（你可以直接在浏览器中访问），如果你希望通过远程桌面访问虚拟机，请使用以下凭据：

* Machine IP: `MACHINE_IP`（在实验房间中启动Windows虚拟机之后，你将获得一个相关的ip地址）
* User: `administrator`
* Password: `letmein123!`

![image-20230331102855386](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331102855386.png)

当上述界面弹出提示时，点击接受证书，然后你现在应该可以登录到远程系统。

## 系统配置面板（ System Configuration）

系统配置实用程序 (`MSConfig`)可用于高级故障排除，其主要目的是帮助诊断启动问题，系统配置面板可以帮助我们导航到其他 Windows 应用程序。

有关系统配置实用程序的更多信息，请参阅下面的文档。

> https://learn.microsoft.com/en-us/troubleshoot/windows-client/performance/system-configuration-utility-troubleshoot-configuration-errors

有几种方法可以启动系统配置，其中一种方法是从开始菜单启动--在开始菜单处输入MSConfig即可。

![image-20230401185721094](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230401185721094.png)

注意:你需要本地管理员权限才能打开系统配置实用程序。

该实用程序顶部有五个选项卡，下面是每个选项卡的名称，在本小节中，我们将简要对每个选项卡进行介绍。

* General-常规
* Boot-引导
* Services-服务
* Startup-启动
* Tools-工具

![image-20230401190049438](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230401190049438.png)

在General选项卡中，我们可以选择Windows启动时要加载的设备和服务，相关的选项包括:标准（Normal,）、诊断性（Diagnostic）以及选择性（Selective）。

在Boot选项卡中，我们可以为操作系统定义各种启动选项。

![image-20230401190444870](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230401190444870.png)

在Services选项卡中，会列出系统中配置的所有服务，而不考虑其状态(运行或停止)，服务是指在后台运行的一种特殊类型的应用程序。Services选项卡允许我们启用或禁用出现在列表中的服务。

![image-20230401190621227](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230401190621227.png)

Startup选项卡会将用户引导至任务管理器以管理启动项，下面是本地机器上MSConfig的Startup选项卡的截图。

![image-20230401190728279](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230401190728279.png)

如上图所示，微软会建议你使用任务管理器(`taskmgr`)来管理(启用/禁用)启动项，系统配置实用程序并不是一个启动管理程序。

在Tools选项卡中，有一个关于各种实用程序(工具)的列表，我们可以运行这些工具来进一步配置操作系统。Tools选项卡中的每个工具都有一个简短的描述，以让我们对工具用途有一些了解。

![image-20230401191148145](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230401191148145.png)

注意上图中的`Selected command`部分，此文本框中的信息将根据工具的不同而变化。

关于工具的运行：我们可以通过Run提示符使用命令运行工具，也可以通过命令提示符使用命令运行工具，或者可以通过单击上图中的`launch`按钮启动对应工具。

**答题**

_tips：阅读本小节内容并且访问Windows虚拟机进行探索，然后回答以下问题。_

访问由实验房间所提供的Windows虚拟机，并进行相关探索。

_**问题1：将 Systems Internals 列为制造商的服务的名称是什么？**_

导航到系统配置面板(`MSConfig`)的“服务”选项卡并单击“制造商-Manufacturer”，这将按服务制造商的字母顺序对服务进行排序，然后寻找由“Systems Internals”所制造的服务。

![image-20230401201614902](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401201614902.png)

> 由Systems Internals所制造的服务名称为：PsShutdown

_**问题2：Windows 许可证注册给谁？**_

导航到系统配置面板的“工具”选项卡并选择“关于 Windows”工具。

单击“启动”按钮启动该工具，你将看到有关操作系统的信息，包括了Windows许可证的注册对象。

![image-20230401201811501](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401201811501.png)

![image-20230401201840541](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401201840541.png)

> Windows许可证的注册对象是：Windows User

_**问题3：Windows 故障排除的命令是什么？**_

在系统配置面板的“工具”选项卡中 选择Windows 故障排除工具，注意此时选项卡的“Selected command”部分，我们可以使用选项卡中对应的命令启动工具（在cmd或者Powershell中运行命令即可）。

![image-20230401203319568](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401203319568.png)

> Windows 故障排除的命令是：C:\Windows\System32\control.exe /name Microsoft.Troubleshooting

_**问题4：什么命令将打开控制面板？（答案是.exe的名字，并非全路径）**_

虽然不是很明显，但是有多个工具的启动命令都引用了同一个exe 文件——系统属性以及Windows 故障排除，两者都使用了control.exe。

![image-20230401203904805](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401203904805.png)

![image-20230401203932302](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401203932302.png)

> 打开控制面板的命令中将包含control.exe

![image-20230401191632414](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401191632414.png)

## 更改UAC设置

我们将继续探索可通过“系统配置”面板使用的工具——Change UAC Settings。

用户帐户控制(UAC-User Account Control) 机制和Windows中的权限分配有关，UAC能够在日常使用时 为具有管理员访问权限的用户帐户保持较低级别的权限，并能在实际需要管理员访问权限时临时提升权限级别。

UAC可以通过UAC设置进行更改，甚至能够完全关闭(不推荐完全关闭UAC)。

我们能够导航至MSConfig 系统配置实用程序中的“工具”选项卡，然后选择对应程序以更改 UAC 设置。

在UAC设置界面，你可以通过移动滑块来改变UAC提示的优先级（从总是提示到完全不提示）。

![image-20230401204502203](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401204502203.png)

**答题**

_tips：阅读本小节内容并且访问Windows虚拟机进行探索，然后回答以下问题。_

访问由实验房间所提供的Windows虚拟机，并进行相关探索。

单击MSConfig的“工具”选项卡中的“更改 UAC 设置”工具，查看“Selected command”部分。

![image-20230401205525109](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401205525109.png)

> 与启动UAC设置的命令 相关联的程序为：UserAccountControlSettings.exe

![image-20230401204813864](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401204813864.png)

## 计算机管理界面

我们将继续探索可通过“系统配置”面板使用的工具——Computer Management（`compmgmt`）。

计算机管理（`compmgmt`) 实用程序包含三个主要部分：系统工具(System Tools)、 存储(Storage)以及服务和应用程序(Services and Applications)。

![image-20230401205855916](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401205855916.png)

**System Tools（系统工具）**

系统工具菜单由六部分组成：_**Task Scheduler、Event Viewer、Shared Folders、Local Users and Groups、Performance、Device Manager**_。

让我们首先从任务调度器(_**Task Scheduler**_)开始，根据微软的说法：通过任务调度器，我们可以创建和管理 计算机在指定的时间自动执行的常见任务。

任务可以是运行应用程序、脚本等，并且任务可以被配置为在任何时间运行。比如计算机任务可以在用户登录或注销时运行，任务也可以被配置为按特定的时间表运行，例如，设置每五分钟运行一次指定的计算机任务。

如果你要创建一个基本任务，可以单击“操作-Actions”(计算机管理页面的右侧窗格)下的“创建基本任务-Create Basic Task”。

![image-20230401211603366](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401211603366.png)

接下来的系统工具是事件查看器（_**Event Viewer**_）。

事件查看器允许我们查看计算机上发生的事件，这些事件的记录可以被看作是一种审计跟踪，能够用来了解计算机系统的活动；这些事件信息通常可用于诊断问题以及调查系统上已执行的操作。

![image-20230401225150680](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401225150680.png)

事件查看器有三个窗格。

1. 左边的窗格显示了一个关于事件日志提供程序的分层树列表。(如上图所示)
2. 中间的窗格将显示一个总体概述和特定于选定提供者的事件摘要。
3. 右边的窗格是一个操作（Action）界面。

可以记录的事件有五种类型，下面是来自[docs.microsoft.com](https://docs.microsoft.com/en-us/windows/win32/eventlog/event-types)的一个表格，提供了对每种事件类型的简要描述。

![image-20230401212813748](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401212813748.png)

![image-20230402103926626](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402103926626.png)

事件日志能够在 Windows logs下可见，下面是来自于[微软官方文档](https://docs.microsoft.com/en-us/windows/win32/eventlog/eventlog-key)的一个表格，它提供了对标准日志和自定义日志的简要描述。

![image-20230401213048340](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401213048340.png)

![image-20230402104244808](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402104244808.png)

接下来的系统工具是共享文件夹（_**Shared Folders**_），在此系统工具中——你将看到其他人可以连接到的共享和共享文件夹的完整列表。

![image-20230401224757803](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230401224757803.png)

![image-20230401213444809](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401213444809.png)

上图是打开Shares文件夹之后所看到的共享列表，此列表会显示 Windows 的默认共享`C$`和 Windows 所创建的默认远程管理共享，如`ADMIN$`。

与Windows中的任何对象一样，你可以右键单击共享文件夹以查看其属性，例如权限属性(谁可以访问共享资源)。

在Sessions文件夹下，你将看到当前连接到“共享”的用户列表（为空）；而正在连接共享的用户可访问的文件夹和文件将在“Open Files”下被列出。

![image-20230401225023683](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401225023683.png)

接下来的系统工具是本地用户和组（_**Local Users and Groups**_），我们可通过在Run提示符下输入`lusrmgr.msc`打开本地用户和组界面（在之前的文章中已有介绍，此处不再赘述）。

接下来的系统工具是性能（_**Performance**_），你将看到一个名为Performance Monitor (`perfmon`)的实用程序。

Perfmon（即性能监视器）可用于实时或者从日志文件中查看性能数据，此实用程序可用于排除计算机系统(本地或远程)上的性能问题。

![image-20230401220426705](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401220426705.png)

我们最后要介绍的系统工具是 设备管理器（_**Device Manager**_），它允许我们查看和配置硬件，例如允许我们禁用任何连接到计算机的硬件。

![image-20230401225612103](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401225612103.png)

**Storage（存储）**

在“存储”菜单下的是“Windows服务器备份”和“磁盘管理”，在这里，我们只讨论“磁盘管理”部分。

注意：由于本文相关实验所使用的虚拟机是 Windows Server，所以会有一些在Windows 10系统中通常看不到的实用程序。

![image-20230401220930240](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401220930240.png)

磁盘管理是Windows中的一个系统实用程序，它使你能够执行高级存储任务，这些任务包括：

* Set up a new drive——建立一个新的驱动器；
* Extend a partition——扩展分区；
* Shrink a partition——缩小分区；
* Assign or change a drive letter (ex. E:) ——分配或更改驱动器盘符(例如E:)

**Services and Applications（服务和应用程序）**

![image-20230401221421693](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401221421693.png)

如上图所示，计算机管理页面的“服务和应用程序”菜单列表将包含“路由和远程访问”、“服务”和“WMI控制”三部分。

“服务”是在后台运行的一种特殊类型的应用程序，通过计算机管理页面的“服务和应用程序”菜单，你不仅可以启用和禁用服务，还可以查看服务的Properties（属性）。

![image-20230401221511583](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230401221511583.png)

此外，“服务和应用程序”菜单中的“WMI Control”将用于配置和控制WMI服务。

_tips：WMI是指Windows管理工具，它的全称为——Windows Management Instrumentation。_

根据维基百科的说法：“WMI允许脚本语言(如VBScript或Windows PowerShell)在本地和远程 对Microsoft Windows个人电脑与服务器进行管理，微软还为WMI提供了一个命令行接口，被称为Windows管理工具命令行(WMIC：Windows Management Instrumentation Command-line)。”

注意：WMIC工具在Windows 10 21H1版本中已弃用，Windows PowerShell取代了WMI服务。

**答题**

_tips：阅读本小节内容并且访问Windows虚拟机进行探索，然后回答以下问题。_

访问由实验房间所提供的Windows虚拟机，并进行相关探索。

_**问题1：打开计算机管理界面的命令是什么？（使用.msc 文件名称作答即可）**_

单击 MSConfig 的“工具”选项卡中的“计算机管理”工具，查看“Selected command”部分即可。

![image-20230401225315207](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401225315207.png)

> 答案1：compmgmt.msc

_**问题2：GoogleUpdateTaskMachineUA 任务配置为每天什么时间运行？**_

通过系统配置面板启动“计算机管理”工具并导航到“任务调度器”，在计划任务列表中，你可以看到“GoogleUpdateTaskMachineUA”任务，和此任务关联的运行时间可以在“触发器-Triggers”栏目下找到。

![image-20230401225451929](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401225451929.png)

![image-20230401225532208](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401225532208.png)

> 答案2：6:15 AM

_**问题3：共享的隐藏文件夹的名称是什么？**_

通过系统配置面板启动“计算机管理”工具并导航到系统工具下的“共享文件夹”部分，打开包含共享列表的Shares文件夹：

![image-20230401225743498](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401225743498.png)

> 答案3：sh4r3dF0Ld3r

![image-20230401222619478](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401222619478.png)

## 系统信息界面

我们将继续探索可通过“系统配置”面板使用的工具—— System Information（`msinfo32`）。

**什么是系统信息( `msinfo32`) 工具？**

根据 Microsoft 的说法，“ Windows 包含一个名为Microsoft System Information (Msinfo32.exe) 的工具，此工具可收集有关你的计算机的信息并显示计算机硬件、系统组件和软件环境的综合视图，你可以使用它来诊断计算机问题。”

系统信息工具的主窗口是系统摘要（System Summary），它提供了很多关于操作系统、系统、处理器、BIOS、内存等的信息。

系统摘要(System Summary)中的信息分为以下三部分：

* 硬件资源：有关系统硬件的高级信息；
* 组件：有关计算机上安装的不同设备的信息，如存储、显示器、键盘、鼠标、打印机等；
* 软件环境：提供有关系统上安装的软件的详细信息，包括驱动程序、服务、任务和启动程序等，还有一个被称为环境变量的部分，环境变量用于存储有关操作系统的详细信息。

系统摘要(System Summary)将显示计算机所使用的一般技术规格，例如计算机的处理器品牌和型号等。

![image-20230401233204758](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401233204758.png)

“硬件资源”中显示的信息不适用于普通计算机用户，如果你想了解更多关于此部分的信息，请参阅[Microsoft官方页面](https://docs.microsoft.com/en-us/windows-hardware/drivers/kernel/hardware-resources)。

![image-20230401234718952](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401234718952.png)

在“组件”下，你可以看到有关计算机上安装的硬件设备的特定信息（此处存在部分信息不会显示）。

![image-20230401234913754](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401234913754.png)

在Software Environment部分中，你可以看到有关嵌入到操作系统中的软件(如：驱动程序)和已安装软件的信息，在此处还可以看到其他详细信息，例如环境变量和网络连接等。

![image-20230401235447963](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401235447963.png)

根据微软的说法，“环境变量存储着有关操作系统环境的信息，这些信息包括操作系统路径、操作系统使用的处理器数量以及临时文件夹的位置等详细信息。具体而言：环境变量存储着操作系统和其他程序所使用的数据，例如，WINDIR环境变量（`%windir%`）所包含的是Windows安装目录的位置，程序可以通过查询这个变量的值来确定Windows操作系统文件的位置。”

单击Environment Variables即可查看已经为计算机分配好的环境变量值。

![image-20230402000123298](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402000123298.png)

可用于查看环境变量的方法还包括：

1. Control Panel > System and Security > System > Advanced system settings > Environment Variables
2. Settings > System > About > system info > Advanced system settings > Environment Variables

![image-20230402000305026](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230402000305026.png)

在系统信息实用程序（`msinfo32`）的最底部，还有一个搜索栏可以帮助我们快速定位，例如：我们可以选择“组件”并搜索“`IP address`”。

![image-20230402001026708](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402001026708.png)

**答题**

_tips：阅读本小节内容并且访问Windows虚拟机进行探索，然后回答以下问题。_

访问由实验房间所提供的Windows虚拟机，并进行相关探索。

_**问题1：打开系统信息的命令是什么？（以相关的exe文件名称作答即可）**_

导航到MSConfig面板的“工具”选项卡并找到“系统信息”工具，查看“Selected command”部分即可。

![image-20230402002650839](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402002650839.png)

> 答案1：msinfo32.exe

_**问题2：系统名称下列出的是什么？**_

导航到MSConfig面板的工具选项卡并启动“系统信息”工具，然后选择“系统摘要”并查看“系统名称”条目。

![image-20230402002833201](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402002833201.png)

> 答案2：THM-WINFUN2

_**问题3：ComSpec的环境变量值是多少？**_

我们导航到MSConfig面板的工具选项卡并启动“系统信息”工具，然后展开“软件环境”菜单并查看“环境变量”页面，以找到ComSpec条目和相应的变量值。

ComSpec 环境变量将指向命令行解释器，即 cmd.exe。

![image-20230402003006483](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402003006483.png)

> 答案3：%SystemRoot%\system32\cmd.exe

![image-20230402001112874](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402001112874.png)

## 资源监视器

我们将继续探索可通过“系统配置”面板使用的工具——Resource Monitor（`resmon`）。

**什么是资源监视器( `resmon`)？**

根据Microsoft的说法：“资源监视器用于显示每个进程和聚合 CPU、内存、磁盘、网络的使用信息，此外它还提供有关哪些进程正在使用单个文件句柄和模块的详细信息；它的高级过滤功能将允许用户隔离与一个或多个进程(应用程序或服务)相关的数据，并能启动、停止、暂停和恢复服务，以及从用户界面关闭无响应的应用程序；它还包括一个进程分析功能，可以帮助识别死锁进程和文件锁定冲突，以便用户可以尝试解决冲突，而不是直接关闭应用程序并遭受丢失数据的风险。 ”

正如本文所提到的其他一些工具一样，资源监视器实用程序主要面向——那些需要在计算机系统上执行高级故障排除的用户。

在概述(Overview)选项卡中，Resmon 有四个部分：

* CPU-中央处理器
* Disk-磁盘
* Network-网络
* Memory-内存

![image-20230402003146897](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402003146897.png)

上图中的四个部分在资源监视器界面的顶部 也有相应的选项卡。

![image-20230402003326346](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402003326346.png)

请注意，上图每个选项卡都有各自的附加信息，四个选项卡的具体图像信息如下所示。

**中央处理器**

![image-20230402003355546](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402003355546.png)

**内存**

![image-20230402003410757](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402003410757.png)

**磁盘**

![image-20230402003427038](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402003427038.png)

**网络**

![image-20230402003441094](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402003441094.png)

资源监视器界面的最右侧还有窗格区域：当你查看概述选项卡时，窗格区域会实时显示和以上四个部分相关的图形视图；当你查看单个选项卡时，窗格区域则会实时显示和选项卡类型对应的图形视图。

![image-20230402085550790](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402085550790.png)

![image-20230402085443134](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402085443134.png)

**答题**

_tips：阅读本小节内容并且访问Windows虚拟机进行探索，然后回答以下问题。_

访问由实验房间所提供的Windows虚拟机，并进行相关探索。

**问题：打开资源监视器的命令是什么？（以相关的exe文件名称作答即可）**

导航到MSConfig面板的工具选项卡并找到“资源监视器”工具，查看“Selected command”部分即可。

![image-20230402004626723](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402004626723.png)

> 答案：resmon.exe

![image-20230402004955219](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402004955219.png)

## 命令提示符界面

我们将继续探索可通过“系统配置”面板使用的工具——Command Prompt（`cmd`）。

命令提示符(`cmd`)一开始似乎令人生畏，但一旦你了解如何与它进行交互，你就会发现它没有那么难理解。

在早期的操作系统中，命令行是与操作系统交互的唯一方式，随着GUI(图形用户界面)被引入，用户就能够使用GUI并点击一些按钮来执行复杂的计算机任务，用户也不再局限于 只能使用命令提示符来与操作系统发生交互。

即使现在GUI是与操作系统进行交互的主要方式，但是计算机用户仍然可以通过命令提示符和操作系统发生交互。

在本小节中，我们将只讨论计算机用户可以在命令提示符中运行以获取有关计算机系统的信息的几个简单命令。

让我们从一些简单的命令开始，比如`hostname`和`whoami`。

命令`hostname`将输出计算机名。

![image-20230402085935253](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402085935253.png)

命令`whoami`将输出登录用户的名称。

![image-20230402090001407](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402090001407.png)

接下来，让我们看看在故障排除时有用的一些命令。

经常使用的命令是`ipconfig`，这个命令将显示计算机的网络地址设置。

![image-20230402090043093](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402090043093.png)

每个命令都有一个帮助手册，用于解释正确执行命令所需的语法，以及可添加到命令中以扩展其执行的任何其他参数。

检索命令帮助手册的命令是`/?`

例如，如果要查看`ipconfig`的帮助手册，可以使用如下命令：`ipconfig /?`

![image-20230402090216859](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402090216859.png)

注意：清除命令提示符界面的命令为`cls`。

下一个命令是`netstat`，根据帮助手册，此命令将显示协议统计信息和当前TCP/IP网络连接信息。

![image-20230402090331950](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402090331950.png)

上图中的红框部分向我们展示了`netstat`命令的示例语法，此语法结构告诉我们`netstat`命令可以单独运行，也可以带参数运行，例如`-a`、`-b`、`-e`等。

当任何参数附加到根命令(本例中为`netstat`)时，对应的命令输出也将发生变化。

接下来我们介绍`net`命令，此命令主要用于管理网络资源，`net`命令支持子命令。

如果输入`net`而不带子命令，则输出结果将显示根命令的语法，并会显示一些你可以使用的子命令。

![image-20230402090824244](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402090824244.png)

对于`net`命令，显示帮助手册`/?`不会起作用，在这种情况下，我们需要使用不同的语法以查看帮助信息，即`net help`。

![image-20230402090931154](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402090931154.png)

如果你希望查看`net user`的帮助信息，则相关的命令为`net help user`。

![image-20230402091028314](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402091028314.png)

你可以使用类似的命令查看其他有用的`net`子命令的帮助信息，例如`localgroup`、`use`、`share`和`session`。

请参考下面的链接，以查看你可以在命令提示符中执行的命令的相关列表： https://ss64.com/nt/

**答题**

_tips：阅读本小节内容并且访问Windows虚拟机进行探索，然后回答以下问题。_

访问由实验房间所提供的Windows虚拟机，并进行相关探索。

_**问题1：在系统配置中，Internet 协议配置的完整命令是什么？**_

导航到MSConfig面板的“工具”选项卡并找到“Internet 协议配置”工具，查看“Selected command”部分即可。

![image-20230402092105770](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402092105770.png)

> 答案1：C:\Windows\System32\cmd.exe /k %windir%\system32\ipconfig.exe

_**问题2：对于 ipconfig 命令，如何显示其详细信息？**_

启动cmd工具，并输入`ipconfig /?`命令以查看相关的帮助信息：

tips：ipconfig 命令将显示计算机的网络地址设置。

![image-20230402094749041](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402094749041.png)

![image-20230402094951203](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402094951203.png)

![image-20230402095102296](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402095102296.png)

> 答案2：ipconfig /all

![image-20230402005656334](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402005656334.png)

## 注册表编辑器

我们将继续探索可通过“系统配置”面板使用的工具——Registry Editor （`regedit`）。

Windows注册表是一个中央分层数据库，它用于存储配置系统所需的信息，这些信息适用于一个或多个用户、应用程序和硬件设备。

注册表包含 Windows 在运行期间不断引用的信息，例如：

* 每个用户的配置文件。
* 计算机上安装的应用程序和每个应用程序可以创建的文档类型。
* 文件夹和应用程序图标的属性表设置。
* 系统上存在哪些硬件。
* 正在使用的端口。

警告：注册表仅适用于高级计算机用户设置，随意更改注册表会影响正常的计算机操作。

有多种方法可以查看/编辑注册表。其中一种方法是使用注册表编辑器( `regedit`)。

![image-20230402005424423](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402005424423.png)

请参阅[Microsoft相关文档](https://docs.microsoft.com/en-us/troubleshoot/windows-server/performance/windows-registry-advanced-users)以了解更多有关Windows注册表的信息。

**答题**

_tips：阅读本小节内容并且访问Windows虚拟机进行探索，然后回答以下问题。_

访问由实验房间所提供的Windows虚拟机，并进行相关探索。

**问题：打开注册表编辑器的命令是什么？（以相关的exe文件名称作答即可）**

导航到MSConfig面板的“工具”选项卡并找到“注册表编辑器”工具，查看“Selected command”部分即可。

![image-20230402085801146](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402085801146.png)

> 答案：regedt32.exe

![image-20230402005539209](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230402005539209.png)

## 小结

回顾：本文所介绍的主要内容是一些可以从Windows`MSConfig` 启动的工具。

在Window操作系统中，实用程序的命令和快捷方式是共享的，这意味着你不必启动`MSConfig`（系统配置）也可运行一些实用程序。

比如：你也可以直接从“开始菜单”界面运行其中一些实用程序：

![image-20230331104050610](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331104050610.png)

`MSConfig`中所列出的工具并未在本文中全部提及，你可以自行探索`MSConfig`中的其他工具。
