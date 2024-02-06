# Windows Fundamentals 1(Windows基础知识1)

本文相关的TryHackMe实验房间链接：https://tryhackme.com/room/windowsfundamentals1xbx

本文介绍：本文所涉及的内容是Windows 基础模块的第 1 部分，我们将了解 Windows 桌面、NTFS 文件系统、UAC、控制面板等Windows基础组件。

![img](https://assets.tryhackme.com/room-banners/windows.png)

## 简介

Windows操作系统(OS)是一个复杂的产品，有许多系统文件、实用程序、设置、功能等。

本文将尝试对Windows操作系统作一般概述，比如浏览用户界面，对系统进行更改等。

启动本文相关实验房间中所附加的Windows虚拟机（你可以直接在浏览器中访问），如果你希望通过[远程桌面](https://www.cyberark.com/resources/threat-research-blog/explain-like-i-m-5-remote-desktop-protocol-rdp)访问Windows虚拟机，请使用以下凭据：

* Machine IP: `MACHINE_IP`（在实验房间中启动Windows虚拟机之后，你将获得一个相关的ip地址）
* User: `administrator`
* Password: `letmein123!`

![image-20230331090333532](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331090333532.png)

当上述界面弹出提示时，点击接受证书，然后你现在应该可以登录到远程系统。

## Windows版本

Windows操作系统有很长的历史，可以追溯到1985年，目前，它是家庭和公司网络的主要操作系统。正因为如此，Windows操作系统一直是黑客和恶意软件作者的目标。

Windows XP是Windows的一个流行版本，运行时间很长；微软还发布过Windows Vista，Vista是对Windows操作系统的一次彻底改造。Windows Vista存在很多问题，因此它在Windows用户中反响不佳，并很快就被淘汰了。

当微软(Microsoft)宣布Windows XP寿终待寝时，许多用户都陷入了恐慌。企业、医院等组织争先恐后地在许多其他硬件和设备上测试下一个可行的Windows版本，即Windows 7；厂商们不得不争分夺秒地工作，以确保他们的产品与Windows 7能够兼容，如果他们做不到，他们的客户就不得不撕毁协议，并寻找另一家可升级他们的产品以兼容Windows 7的供应商。

Windows 7虽然很快就发布了，但它也被标记上了终止支持的日期，然后是Windows 8.x，8.x出现时间很短暂，就像Vista一样。

后面出现了Windows 10，这是目前普遍用于台式电脑的Windows操作系统版本。Windows 10有两种版本，家庭版和专业版，[家庭版和专业版存在一些区别](https://www.microsoft.com/en-us/windows/compare-windows-10-home-vs-pro)。

tips：尽管我们没有谈论服务器，但也有可用于服务器的Windows操作系统，比如[Windows Server 2022](https://www.microsoft.com/en-us/windows-server)。

许多批评人士喜欢抨击微软，但微软每一个新版本的Windows操作系统的可用性和安全性方面都取得了长足进步。

注：本文所探索的Windows虚拟机的操作系统版本为“Windows Server 2019 Standard”，详见此Windows虚拟机中的“系统信息”界面。

截至2021年6月，微软宣布了[Windows 10的退休日期](https://docs.microsoft.com/en-us/lifecycle/products/windows-10-home-and-pro?ranMID=24542\&ranEAID=kXQk6\*ivFEQ\&ranSiteID=kXQk6.ivFEQ-M28j3qbUhtM2JFCT2wmhOA\&epi=kXQk6.ivFEQ-M28j3qbUhtM2JFCT2wmhOA\&irgwc=1\&OCID=AID2000142\_aff\_7593\_1243925\&tduid=%28ir\_\_uszrgcddyskfqz3fkk0sohz3wv2xuurc01kgzkod00%29%287593%29%281243925%29%28kXQk6.ivFEQ-M28j3qbUhtM2JFCT2wmhOA%29%28%29\&irclickid=\_uszrgcddyskfqz3fkk0sohz3wv2xuurc01kgzkod00\&ranMID=24542\&ranEAID=kXQk6\*ivFEQ\&ranSiteID=kXQk6.ivFEQ-4cKUPfbv9lM\_IR2EX7K\_hw\&epi=kXQk6.ivFEQ-4cKUPfbv9lM\_IR2EX7K\_hw\&irgwc=1\&OCID=AID2000142\_aff\_7593\_1243925\&tduid=%28ir\_\_feexvhocigkfqna9kk0sohznb32xutanagupypus00%29%287593%29%281243925%29%28kXQk6.ivFEQ-4cKUPfbv9lM\_IR2EX7K\_hw%29%28%29\&irclickid=\_feexvhocigkfqna9kk0sohznb32xutanagupypus00)：“微软将继续支持至少一个Windows 10版本，直到2025年10月14日。”

截至2021年10月5日，Windows 11已经成为终端用户可用的Windows操作系统。

访问以下链接，可阅读更多关于Windows 11的信息：https://www.microsoft.com/en-us/windows?wa=wsignin1.0

**答题**

阅读以下链接，查看Windows 10家庭版和专业版的区别：

https://www.microsoft.com/en-us/windows/compare-windows-10-home-vs-pro

![image-20230331100052164](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331100052164.png)

> 答案：BitLocker

![image-20230331095806462](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331095806462.png)

## 桌面（图形用户界面）

Windows 桌面，简称Windows图形用户界面(GUI)，是你登录 Windows 计算机后能看到并使用的主屏幕界面。

通常情况下，你需要先通过登录界面完成系统登录，然后才能看到Windows桌面。在登录界面上，你需要输入有效的帐户名称和登录凭据，这通常是该特定系统使用者的用户名和密码或者是Active Directory环境(如果是加入域的机器)中先前存在的Windows帐户的用户名和密码。

![image-20230331100826045](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331100826045.png)

上面的截图就是一个典型的Windows Desktop的例子，以下是组成上述GUI的一些组件名称：

1. The Desktop（主桌面）
2. Start Menu（开始菜单）
3. Search Box (搜索框-Cortana)
4. Task View（任务视图）
5. Taskbar（任务栏）
6. Toolbars（工具栏）
7. Notification Area（通知区域）

**The Desktop（主桌面）**

桌面是你可以快捷进入程序、文件夹、文件等的地方。计算机程序和文件的图标要么按照字母顺序排列在文件夹中，要么随机分散在桌面上，并没有特定的组织；但是，放在文件夹中的程序通常也可以通过桌面上的对应快捷方式被用户快速访问。

桌面的外观和风格可以根据用户的喜好自行进行更改。你可以右键单击桌面上的任何位置，然后就能出现一个上下文菜单，此菜单将允许你更改桌面图标的大小、指定图标的排列方式，并且还能允许你将项目复制/粘贴到桌面，以及在桌面上创建新项目(如创建文件夹、快捷方式或文本文档)等等。

![image-20230331183522221](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331183522221.png)

通过“显示设置-Display settings”功能，你还可以更改屏幕的分辨率和屏幕方向，如果你有多个电脑屏幕，你可以在这里对多屏幕设置进行配置。

![image-20230331183648774](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331183648774.png)

_注意:在远程桌面会话中，某些显示设置将被禁用。_

![image-20230331184113446](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230331184113446.png)

你也可以通过选择“个性化-Personalize”来更改壁纸。

![image-20230331183829228](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230331183829228.png)

通过“个性化-Personalize”功能，你可以更改桌面的背景图像，以及更改字体、主题、配色方案等。

![image-20230331184043358](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230331184043358.png)

**The Start Menu（开始菜单）**

在之前的Windows版本中，桌面GUI的左下角还可以看到“开始-Start ”一词；但是，在Windows 10等现代版本的操作系统中，“开始-Start ”一词不再出现，取而代之的是一个Windows Logo。尽管开始菜单的外观发生了变化，但其总体功能还是和以前一样的。

“开始菜单”为计算机用户提供了对所有最有用的程序、文件、实用工具等的访问，当你点击Windows logo之后，开始菜单就会打开，它将由三个部分共同组成。

![image-20230331184759638](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230331184759638.png)

**1.**“开始菜单”的第一个部分(Start Menu从左到右的第一个区域)为你对帐户或登录会话的可执行操作提供了快速快捷方式，例如更改用户帐户、锁定屏幕或注销帐户，其他特定于你的帐户的快捷方式还有“文档”文件夹(文档图标)和“图片”文件夹(图片图标)，最后，点击齿轮图标将允许你进入设置屏幕界面，点击电源图标将允许你关闭计算机、重新启动计算机或者断开远程桌面会话(如果存在远程桌面会话)。

如下图所示，你可以看到多个附加了功能的图标，如果你要展开此部分的更多内容，请单击顶部的图标：

![image-20230331190016903](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230331190016903.png)

**2.**“开始菜单”的第二个部分(Start Menu从左到右的第二个区域)将在顶部显示所有最近添加的程序和所有已安装的程序(可配置为出现在开始菜单中)，在此部分中，应用程序/程序将会按字母顺序被列出。

![image-20230331190757774](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230331190757774.png)

如上图所示，第一个框是最近添加的应用程序/程序将出现的地方，第二个框将显示所有已安装的应用程序/程序 并且按字母排列。

如果你有一个很长的已安装程序列表，你可以点击字母网格中的字母标题，从而实现跳转到已安装程序列表中的特定部分。

![image-20230331191141034](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230331191141034.png)

_注意：上图中的白色字母与已安装程序的字母标题相匹配。_

**3.**“开始菜单”的第三个部分(Start Menu从左到右的第三个区域)，也就是开始菜单的右侧，此区域是你可以找到特定应用程序/程序或者实用程序的图标的地方；这些图标被称为磁贴，一些磁贴在默认情况下会被添加到此区域。

右键单击任何一个磁贴，都会出现一个菜单，这个菜单可以允许你对所选磁贴执行更多操作：例如调整平铺大小，从开始菜单中取消固定，查看其属性等。

![image-20230331192202260](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230331192202260.png)

在前述的“开始菜单”第二个区域中的应用程序/程序，都可以通过右键单击并选择“Pin to Start”以形成新的磁贴，而这些新的磁贴会出现在“开始菜单”的第三个区域中。

![image-20230331192854403](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230331192854403.png)

**The Taskbar（任务栏）**

在任务栏中(任务栏一般在桌面GUI的底部)，某些组件是默认启用且可见的，例如，下图中的工具栏（Toolbars）就是为了演示目的而启用的。

如果你想要禁用一些任务栏可用组件，你可以右键单击任务栏，这将弹出一个上下文菜单以允许你进行更改。

![image-20230331193428534](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230331193428534.png)

所有你已经打开或者启动的任何应用程序/程序、文件夹、文件等都会出现在任务栏中。

![image-20230331194347701](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331194347701.png)

当你将鼠标悬停在任务栏中的应用程序图标上时，这将提供关于此应用程序的预览运行缩略图以及提示信息，这个提示信息是有用的，如果你打开了很多不同的或者相同的应用/程序，如多个谷歌Chrome浏览器界面，你可能会希望找到一个你想要的谷歌Chrome实例，此时你就可以根据提示信息来找到目标实例。

当你关闭应用程序时，相关的应用程序的图标将从任务栏中消失(除非你显式地将应用程序图标固定到任务栏)。

**The Notification Area（通知区域）**

通知区域通常位于Windows屏幕的右下方，它是显示日期和时间的地方。在通知区域中，你可能看到的其他图标还包括音量图标、网络/无线图标、电池状态图标等等，你可以通过任务栏设置来添加或删除通知区域中的图标。

![image-20230331195748338](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331195748338.png)

进入任务栏设置界面，然后向下滚动到通知区域部分即可进行更改。

![image-20230331195842358](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331195842358.png)

下面是微软官方关于用户桌面GUI中的“开始菜单”和“通知区域”的简要文档。

* https://support.microsoft.com/en-us/windows/see-what-s-on-the-start-menu-a8ccb400-ad49-962b-d2b1-93f453785a13
* https://support.microsoft.com/en-us/windows/customize-the-taskbar-notification-area-e159e8d2-9ac5-b2bd-61c5-bb63c1d437c3#WindowsVersion=Windows\_10

tips：你可以右键单击任何文件夹、文件、应用程序/程序的图标来查看更多信息或者对所单击的项目执行相关操作。

**答题**

_tips：阅读本小节内容并且访问Windows虚拟机进行探索，然后回答以下问题。_

访问由实验房间所提供的Windows虚拟机，并进行相关探索。

问题1：哪个选项可以隐藏/禁用搜索框？

![image-20230331220154338](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331220154338.png)

> 答案1:Hidden

问题2：哪个选项可以隐藏/禁用任务视图按钮？

![image-20230331220607941](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331220607941.png)

> 答案2：Show Task View button

问题3：除了“时钟”和“网络”之外，通知区域中还有哪些其他图标可见？

![image-20230331220953527](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331220953527.png)

![image-20230331221057862](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331221057862.png)

![image-20230331220827367](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331220827367.png)

> 答案3：Action Center

![image-20230331200408557](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331200408557.png)

## 文件系统

在现代版本的Windows中所使用的文件系统是“New Technology File System-新技术文件系统-NT文件系统”，可简称为[NTFS](https://learn.microsoft.com/en-us/windows-server/storage/file-server/ntfs-overview)。

在NTFS之前，还有其他文件系统：比如 FAT16/FAT32(File Allocation Table-文件分配表)和HPFS(High Performance File System-高性能文件系统)。

现在你仍然可以看到FAT分区的使用，例如，你通常可以在一些USB设备、MicroSD卡中看到FAT分区，但你通常不会在个人Windows电脑或Windows服务器上看到FAT分区。

NTFS也是一种日志文件系统(具有故障恢复能力的文件系统)，当出现故障时，NTFS文件系统可以根据日志文件中所存储的信息 自动修复磁盘上的文件夹/文件（这个功能是FAT文件系统所不具备的）。

NTFS解决了以前的文件系统的许多限制，如：

* 支持大于4GB的文件
* 能够对文件夹和文件设置特定的权限
* 能够对文件夹和文件进行压缩
* 支持加密(属于EFS-[Encryption File System](https://learn.microsoft.com/en-us/windows/win32/fileio/file-encryption)-加密文件系统)

如果你正在运行Windows，你的Windows所安装使用的文件系统是什么?你可以查看操作系统的驱动器属性(右键单击)，通常是C驱动器(`C:\`)。

![image-20230331210623069](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331210623069.png)

关于文件系统的概述，你可以阅读 Microsoft [关于 FAT、HPFS 和 NTFS 的官方文档](https://docs.microsoft.com/en-us/troubleshoot/windows-client/backup-and-storage/fat-hpfs-and-ntfs-file-systems)。

接下来，让我们简要谈谈 NTFS 特有的一些功能。

在 NTFS volumes(volumes--磁盘“卷标”)上，你可以设置授予访问或者拒绝访问文件、文件夹的权限，具体的权限有：

* **Read-读**
* **Write-写**
* **Read & Execute-读、执行**
* **List folder contents-列出文件夹内容**
* **Modify-修改**
* **Full control-完全控制**

下图列出了每个权限的含义以及这些权限如何应用于文件和文件夹。 （出自[微软官方文档-文件和文件夹权限](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-2000-server/bb727008\(v=technet.10\)?redirectedfrom=MSDN)）

![image-20230331211513652](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331211513652.png)

![image-20230331211943792](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331211943792.png)

如何查看文件或文件夹的权限？

1. 右键单击要检查权限的文件或文件夹。
2. 在弹出的上下文菜单中单击选择“属性”（`Properties`）。
3. 在“属性”中，单击“安全”（`Security`）选项卡。
4. 在组或用户名（`Group or user names`）列表中，选择要查看其权限的用户、计算机或组。

在下图中，你可以看到 Windows 文件夹的用户(`Users`)组的权限。

![image-20230331213944033](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331213944033.png)

NTFS 文件系统的另一个特性是备用数据流 (ADS-Alternate Data Streams)。

备用数据流(ADS)是特定于Windows NTFS(New Technology File System-新技术文件系统)的文件属性。

每个文件都至少有一个数据流(`$DATA`)，而ADS（备用数据流）将允许文件包含多个数据流。本机[Windows文件资源管理器](https://support.microsoft.com/en-us/windows/what-s-changed-in-file-explorer-ef370130-1cca-9dc5-e0df-2f7416fe1cb1)并不会向用户显示ADS（备用数据流），你可以使用第三方可执行文件来查看ADS（备用数据流），或者你也可以选择使用[Powershell](https://learn.microsoft.com/en-us/powershell/scripting/overview?view=powershell-7.3\&viewFallbackFrom=powershell-7.1) 来查看文件的ADS（备用数据流）。

从安全的角度来看，恶意软件编写者会使用ADS来隐藏数据，但是这并非意味着所有的ADS使用都是恶意的；例如，当你从Internet(互联网)下载一个文件时，会向ADS写入标识符，以识别该文件确实是从Internet下载的。

要了解更多关于ADS的知识，请参考[MalwareBytes所提供的相关文档](https://www.malwarebytes.com/blog/news/2015/07/introduction-to-alternate-data-streams)。

**答题**

_tips：通过阅读本小节内容即可回答以下问题。_

![image-20230331222543571](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331222543571.png)

## Windows\System32文件夹

Windows文件夹(`C:\Windows`)通常被称为包含Windows操作系统的文件夹。

这个文件夹不一定要位于C盘，它可以驻留在任何其他磁盘驱动器中，在技术上还可以实现驻留在不同的文件夹中；这就是环境变量，更具体地说是系统环境变量能够发挥作用的地方，Windows文件夹(目录)的系统环境变量是`%windir%`。

根据[微软相关文档](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about\_environment\_variables?view=powershell-7.1)：“环境变量存储着有关操作系统环境的信息，这些信息包括操作系统路径、操作系统使用的处理器数量以及临时文件夹的位置等详细信息。”

在“Windows”文件夹中有很多文件夹：

![image-20230331224546618](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331224546618.png)

其中一个文件夹是System32：

![image-20230331224616743](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331224616743.png)

System32文件夹中保存着一些对操作系统至关重要的文件，在与此文件夹交互时，你应该要非常谨慎，不小心删除System32中的任何文件或者文件夹都可能会导致Windows操作系统无法正常操作。

可参考资料：https://www.howtogeek.com/346997/what-is-the-system32-directory-and-why-you-shouldnt-delete-it/

注意：Windows基础所涉及的许多工具都位于System32文件夹中。

**答题**

_tips：通过阅读本小节内容即可回答以下问题。_

![image-20230331224902472](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331224902472.png)

## 用户帐户、配置文件和权限

在典型的本地Windows系统上，用户帐户可以是以下两种类型之一：管理员（Administrator）和标准用户（Standard User）。

用户帐户的类型将决定用户可以在特定的Windows系统上执行什么操作。

* 管理员可以对系统进行更改，如添加用户、删除用户、修改组、修改系统设置等。
* 标准用户只能对归属于该用户的文件夹/文件进行更改，不能执行系统级更改。

假设你当前以管理员身份登录到Windows机器，你可以通过一些方法来确定当前系统上存在哪些用户帐户。

你可以单击`Start Menu`并输入`Other User`。

![image-20230331230819292](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331230819292.png)

如果你点击上图中的`Other Users`，则应该会出现一个设置窗口：

![image-20230331230932138](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331230932138.png)

由于你现在是管理员，你将能够看到一个将其他人添加到此PC的选项（Add someone else to this PC）。

_注意：标准用户将看不到此选项。_

继续单击本地用户帐户，此时应该会出现更多选项：更改帐户类型和删除。

![image-20230331231440182](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331231440182.png)

单击上图中的Change account type，下拉框中的当前值即表示当前的帐户类型。

![image-20230331231216995](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331231216995.png)

当系统创建用户帐户时，将同时为该用户创建一个配置文件，每个用户配置文件的文件夹位置将落在`C:\Users`下；例如，用户帐号Max的用户配置文件所在的文件夹为`C:\Users\Max`。

用户配置文件的创建会在用户账户初次登录时完成。

当一个新用户帐户第一次登录到本地系统时，他将在登录屏幕上看到几条消息，其中一条消息User Profile Service会在登录屏幕上停留一段时间，这表示它正在创建相关的用户配置文件。

![image-20230331231928100](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331231928100.png)

一旦登录成功，用户将看到一个类似于下面的对话框，这表明配置文件正在创建中。

![image-20230331232138839](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331232138839.png)

每个用户配置文件都会有一些相同的文件夹，其中的一些是：

* Desktop
* Documents
* Downloads
* Music
* Pictures

访问用户账户信息的另一种方法是使用本地用户和组管理界面（ Local User and Group Management）。

你可以右键单击“开始菜单- Start Menu”并单击Run，然后输入`lusrmgr.msc`：

![image-20230331233204501](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230331233204501.png) ![image-20230331233318791](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230331233318791.png)

_注意：Run对话框可以允许我们快速打开项目。_

使用Run对话框并访问`lusrmgr.msc`之后，我们就进入到了“本地用户和组管理”界面，在这个界面，你可以看到两个文件夹：Users和Groups。

![image-20230331233829846](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331233829846.png)

如果单击Groups，你将看到所有本地组的名称以及每个组的简要描述。每个组都有其权限，计算机用户将由管理员分配/添加到组中，当用户被分配到某个组时，该用户将会继承该组的权限，并且一个用户可以被分配给多个组。

![image-20230331234311751](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331234311751.png)

_注意：如果你从“其他用户-Other users”中单击“将其他人添加到此PC”，这将自动打开一个“本地用户和组管理”界面。_

**答题**

_tips：阅读本小节内容并且访问Windows虚拟机进行探索，然后回答以下问题。_

访问由实验房间所提供的Windows虚拟机，并进行相关探索。

查看Other users账户：

![image-20230331235802326](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230331235802326.png)

使用`win + r`打开Run对话框，然后输入`lusrmgr.msc`并点击确定，成功进入到“本地用户和组管理”界面：

![image-20230401000117411](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230401000117411.png)

> Other users所对应的账户Name为：tryhackmebilly

查看这个用户(tryhackmebilly)属于哪个组：

![image-20230401000445572](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230401000445572.png)

> tryhackmebilly用户属于：Remote Desktop Users组和Users组。

查看用于来宾用户访问的内置帐户：

![image-20230401000749028](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230401000749028.png)

> 用于来宾用户访问的内置帐户是：Guest

查看来宾用户帐户的帐户状态：

![image-20230401001211417](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401001211417.png)

> 帐户状态为：Account is disabled

![image-20230331234547668](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230331234547668.png)

## 用户帐户控制

大多数家庭用户都以本地管理员身份登录到他们的 Windows 系统，而帐户类型为管理员的用户可以对系统进行任意更改。

计算机用户在执行某些操作时，其实并不需要系统为这些操作分配一个很高的权限，例如网上冲浪、处理 Word 文档等；这种提升的权限增加了系统被攻破的风险，也使得恶意软件更容易感染系统（因为这可能会使恶意软件在一个较高的权限下得到运行）。

为了保护本地用户的权限分配安全，Microsoft 引入了用户帐户控制机制(UAC-User Account Control)，这个概念最初是在[Windows Vista](https://en.wikipedia.org/wiki/Windows\_Vista)系统中引入的，并且在随后的 Windows 版本中也得到继续使用。

注意：UAC在默认情况下不适用于内置的本地管理员帐户。

UAC是如何工作的？当具有管理员帐户类型的用户登录系统时，当前会话并不会以提升的权限运行(此时用户具有管理员权限，无需再提升)；当具有标准帐户类型的用户登录系统时，如果用户想要执行一些需要更高级别权限的操作时，UAC机制会提示用户是否允许执行 并可能需要我们输入管理员密码。

我们可以查看当前所登录的帐户（假设为内置管理员帐户）上的程序信息——右键单击程序并查看其属性即可。

在程序属性的“安全-Security ”选项卡中，我们可以看到该程序所属的用户/组以及它们对此程序文件的权限（从下图可知：标准用户没有被列出）。

![image-20230401081039434](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401081039434.png)

如果我们以标准用户登录并尝试运行上图中的程序，那么就会触发UAC机制。

注意：当我们以管理员帐户登录之后，可以通过lusrmgr.msc界面查看到标准用户的用户名和密码(知道用户名和密码之后，我们就能通过远程桌面进行登录)。

当我们作为标准用户登录时，上述可执行程序的默认图标上会有一个盾牌标志。程序上的盾牌标志代表该程序受UAC限制，当我们点击运行该程序时，会提示是否允许以更高级别的权限运行。（如果要继续运行该程序，就需要我们输入管理员密码）

![image-20230401083931178](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401083931178.png)

双击上图中的程序，你将看到UAC提示（此时需要输入管理员帐户的密码，然后才能继续运行该程序）。

![image-20230401084355843](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230401084355843.png)

一段时间无操作之后，上图中的UAC提示符就会消失，然后程序就不会运行。

UAC功能降低了恶意软件成功破坏系统的可能性，你可以阅读[微软官方文档](https://learn.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works)了解更多关于UAC的信息。

**答题**

_tips：通过阅读本小节内容即可回答以下问题。_

![image-20230401085221414](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401085221414.png)

## 设置和控制面板

在Windows系统上，进行系统更改的主要位置是“设置菜单”和“控制面板”。

长期以来，“控制面板”一直是进行系统更改(如添加打印机、卸载程序等)的首选位置；而“设置菜单”是在Windows 8 版本中引入的，此版本是第一个适用于触摸屏平板电脑的Windows操作系统，而且“设置菜单”在Windows 10中也仍然可用。事实上，如果用户现在想要进行系统更改，“设置菜单”将会是用户所使用的主要位置。

“设置菜单”和“控制面板”既有相似之处，也有不同之处。

**设置菜单**

![image-20230401090019326](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230401090019326.png)

**控制面板**

![image-20230401090327928](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401090327928.png)

注意：如果个人计算机设备上的Windows操作系统版本不同，那么“设置菜单”中的图标也可能不同。

设置菜单和控制面板都可以通过开始菜单进行访问。

![image-20230401090517708](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401090517708.png)

控制面板能够让你访问更复杂的设置和执行更复杂的操作。在某些情况下，当你使用“设置菜单”开始进行系统更改时，最后还是会进入到“控制面板”中的相关界面。

例如，你可以在“设置菜单”中单击“网络和Internet”，然后再单击“更改适配器选项”。

![image-20230401090924855](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401090924855.png)

然后你就会注意到：接下来所弹出的窗口来自于控制面板。

![image-20230401091210493](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401091210493.png)

如果你不清楚在更改设置时应该打开设置菜单还是控制面板，你可以使用开始菜单并进行搜索。

在下面的例子中，正在搜索的内容是“wallpaper”，并且最后得到的返回结果会很少。

![image-20230401091556674](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230401091556674.png)

如果我们点击上图中的“最佳匹配”，将会出现一个“设置”菜单窗口，能够让我们对壁纸进行更改。

![image-20230401091737337](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230401091737337.png)

**答题**

_tips：阅读本小节内容并且访问Windows虚拟机进行探索，然后回答以下问题。_

访问由实验房间所提供的Windows虚拟机，并进行相关探索。

进入到控制面板界面，将视图更改为小图标显示，查看最后一项设置名称：

![image-20230401093612966](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401093612966.png)

> 最后一项设置名称为：Windows Defender Firewall 。

![image-20230401091808960](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401091808960.png)

## 任务管理器

本文所涉及的最后一个主题是任务管理器。

任务管理器可以提供 有关当前在系统上运行的应用程序和进程的信息，任务管理器中还有一些其他信息也是可用的，比如正在使用多少CPU和RAM，这是属于系统性能的部分。

你可以通过右键单击任务栏来访问任务管理器。

![image-20230401092305061](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401092305061.png)

任务管理器界面将会以一个简单视图的形式打开，视图中可能并不会显示太多信息。

![image-20230401092416660](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401092416660.png)

我们可以单击上图中的`More details`，然后视图内容将会发生变化。

![image-20230401092538793](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230401092538793.png)

有关任务管理器的更多内容，请参阅以下博客文章：

> https://www.howtogeek.com/405806/windows-task-manager-the-complete-guide/
>
> https://www.howtogeek.com/66622/stupid-geek-tricks-6-ways-to-open-windows-task-manager/

**答题**

_tips：通过阅读上述链接相关的博客文章，可回答下面问题。_

![image-20230401092709123](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230401092709123.png)
