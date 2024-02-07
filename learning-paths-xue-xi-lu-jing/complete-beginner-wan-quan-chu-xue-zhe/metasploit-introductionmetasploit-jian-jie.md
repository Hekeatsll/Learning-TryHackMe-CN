# Metasploit: Introduction(Metasploit简介)

TryHackMe实验房间链接：https://tryhackme.com/room/metasploitintro

## 介绍

Metasploit 是应用最广泛的利用框架之一，它是一个强大的工具，可以支持渗透测试的所有阶段，从信息收集到后渗透。

Metasploit 有两个主要版本:

* Metasploit Pro: 促进任务自动化和管理的商业版。这个版本有一个图形用户界面(GUI)。
* Metasploit Framework: 从命令行界面开始工作的开源版本，本文主要关注这个版本，MSF在Kali上默认安装 。

Metasploit 框架是一组允许进行信息收集、扫描、漏洞利用、exp开发、后渗透等操作的工具，虽然 Metasploit 框架的主要用途集中在渗透测试领域，但它也有助于漏洞研究和exp开发。

Metasploit 框架的主要组成部分可概括如下：

* msfconsole：主命令行界面。
* Modules（模块）: msf框架支持的功能模块，如漏洞利用，扫描，有效载荷（payload）等。
* Tools（工具）：有助于漏洞研究、漏洞评估或渗透测试的独立工具。其中一些工具是msfvenom、pattern\_create 、pattern\_offset。其中pattern\_create 和pattern\_offset在exp开发阶段是很有用的工具，我们在这里主要介绍msfvenom。

本文将介绍 Metasploit 的主要组成部分，了解如何在目标系统上找到相关的漏洞、设定msf中的一些参数、对易受攻击的服务进行利用等。

## Metasploit 的主要组成部分

在使用 Metasploit 框架时，你将主要与 Metasploit 控制台进行交互。你可以使用`msfconsole`命令从kali Linux终端启动Metasploit控制台，这个控制台是你与 Metasploit 框架的不同模块进行交互的主界面。

模块（Modules ）是 Metasploit 框架中用于执行特定任务的小组件，比如利用漏洞、扫描目标、或者执行穷举法（暴力破解）等。

在深入研究模块之前，明晰一些未来将重复出现的概念会很有帮助：漏洞、漏洞利用（exp）和有效载荷（payload）。

* 漏洞：影响目标系统的设计、编码或逻辑缺陷，利用漏洞可能导致泄露机密信息或者允许攻击者在目标系统上执行任意代码。
* 漏洞利用（exp）：对目标系统上存在的漏洞进行利用的一段代码
* 有效载荷（payload）：一个exp会利用一个漏洞，然而，如果我们希望执行exp能够得到我们想要的结果（进入目标系统，阅读机密信息等），我们就需要使用有效载荷，有效载荷是将在目标系统上运行的代码。

下面列出了MSF中的一些模块和类别，仅供参考，你可以通过 Metasploit 控制台（msfconsole）与它们进行交互。

**Auxiliary（辅助）**：任何功能支持模块，如扫描器，爬虫功能和 fuzz功能，都可以在这里找到。

![image-20221002232549493](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221002232549493.png)

**Encoders（编码器）**：编码器将允许你对exp和payload进行编码，以期望绕过基于特征的反病毒解决方案的限制。

基于特征的反病毒和安全解决方案有一个已知威胁的数据库，它们通过将可疑文件与此数据库进行比较来检测威胁，如果有匹配就发出警报。因为反病毒解决方案可以执行额外的检查，所以编码器的成功率是有限的。

![image-20221002233121271](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221002233121271.png)

**Evasion（逃避）**：虽然编码器能够对payload进行编码，但是使用编码器不应该被认为是一个直接逃避杀毒软件的尝试；在另一方面，使用Evasion（逃避）模块才会尝试直接逃避杀毒软件，这个功能模块或多或少会成功。

![image-20221002235657666](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221002235657666.png)

**Exploits（exp）**：漏洞利用，主要针对目标系统。

![image-20221003000242685](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003000242685.png)

**NOPs**：**NOPs (No OPeration)** ，无操作。

它们在 Intel x86 CPU 系列中用0x90表示，接下来的一个周期中 CPU 将不执行任何操作，该模块通常用作缓冲区，以实现一致的有效负载（payload）大小。

![image-20221003000615444](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003000615444.png)

**Payloads（有效载荷）**：有效载荷是在目标系统上运行的代码。

exp会利用目标系统的漏洞，但是为了达到预期的结果，我们需要一个有效载荷，例如，获取一个 shell，将恶意软件或后门程序加载到目标系统，运行一个命令，或者启动 calc.exe 作为概念验证（POC-proof of concept）以便添加到渗透测试报告中。通过启动 calc.exe 应用程序来远程启动目标系统上的计算器是一种良好的POC方式，这样可以证明我们能够在目标系统上运行命令。

在目标系统上运行命令已经是一个重要的步骤，但是建立一个交互式连接，允许你输入能在目标系统上执行的命令更佳，这种交互式命令行环境被称为“ shell”。

Metasploit 提供了发送不同有效载荷（payload）的能力，这些有效载荷有些可以在目标系统上打开 shell。

![image-20221003001643536](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003001643536.png)

你将在有效载荷模块下看到三个不同的目录: singles、stagers 和stages。

* Singles：不需要下载其他组件就可以运行的自包含有效载荷(能实现添加用户、启动 notepad.exe 程序等)。
* Stagers：负责建立 Metasploit 与目标系统之间的连接渠道。在处理分段有效载荷时非常有用，“分段 payloads” 会先上传一个payload片段到目标系统上，然后再下载剩余的payload片段。这样处理有一些优点，因为与一次发送全部有效载荷相比，分段有效载荷的初始大小将相对较小。
* Stages： Downloaded by the stager，这将允许你使用更大的有效载荷。

Metasploit 有一种微妙的方法来帮助你识别单一(也称为“内联”)有效载荷和分段有效载荷。

> generic/shell\_reverse\_tcp和windows/x64/shell/reverse\_tcp，两者都能用于建立 Windows反向shell。
>
> 前者是内联(或单一)有效载荷,这可以从"shell"和"reverse"之间的"\_"符号得出判断结果，后者则是分段有效载荷，这可以从 "shell" 和"reverse"之间的"/"符号得出判断结果。

**Post**：Post 模块将有助于以上渗透测试过程的最后阶段以及后渗透。

![image-20221003004842215](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003004842215.png)

如果你希望进一步熟悉这些模块，可以在 Metasploit 安装的模块文件夹下找到它们，对于 TryHackMe提供的AtackBox，这些模块位于/opt/metasploit-framework-5101/modules路径下（如上面的图片所示）。

**答题**

![image-20221003005532578](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003005532578.png)

## Msfconsole

如前所述，控制台是 Metasploit 框架的主要接口。你可以在TryHackMe提供的AtackBox 终端或者在已经安装了Metasploit Framework的任何系统上使用`msfconsole` 命令来启动MSF控制台。

![image-20221003005851414](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003005851414.png)

一旦成功启动控制台，你将看到命令行的开头会被更改为 msf5(或 msf6，这取决于你安装的 Metasploit 版本)。Metasploit 控制台(msfconsole)可以像普通的命令行 shell 一样使用，如下所示：

第一个命令是`ls`，列出了使用 msfconsole命令启动的Metasploit 所在文件夹的内容；然后是一个发送到 Google's DNS（谷歌的域名服务器）IP 地址(8.8.8.8)的 `ping`。

当我们用TryHackMe提供的 AtackBox (Linux)进行ping操作时，我们添加了 `-c 1`选项，这样只会发送一个 ping，否则，ping 进程将持续进行，直到使用 CTRL + C 才能让它停止。

![image-20221003011226640](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003011226640.png)

msfconsole将支持大多数 Linux 命令,包括clear命令（清除终端屏幕），但是它不允许你使用常规命令行的某些特性（比如：不支持输出重定向），如下图所示：

![image-20221003011705085](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003011705085.png)

说到这个（上图中输入了help），help 命令可以单独使用，也可以用于特定的命令，下面是set 命令的帮助（help）菜单：

![image-20221003011926352](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003011926352.png)

你可以使用 history 命令查看之前键入的命令：

![image-20221003012153574](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003012153574.png)

msfconsole 的一个重要特性是支持选项卡补全，这将在以后使用 Metasploit 命令或处理模块时派上用场。例如，如果你输入he然后按下 Tab 键，你将看到它会自动补全成help命令。

msfconsole 是由上下文进行管理的，这意味着，除非将参数设置为全局变量，否则如果更改了决定使用的模块，所有原模块的参数设置都将丢失。

在下面的示例中，我们使用了 ms17\_010\_eternalblue的exp，并且设置了RHOSTS等参数，如果我们现在切换到另一个模块(例如端口扫描模块) ，我们就需要再次设置 RHOSTS参数值，因为我们之前所做的所有参数更改都保留在ms17\_010\_eternalblue的exp的上下文中。

让我们看看下面的示例，以便更好地理解这个特性，我们将使用 MS17-010 永恒之蓝的exp作为例子。

当你输入 use exploit/windows/smb/ms17\_010\_eternalblue 命令，

你将看到命令行提示符从 msf6 更改为 "msf6 exploit(windows/smb/ms17\_010\_eternalblue)" 。

“永恒之蓝”是美国国家安全局(NSA)开发的一个漏洞，该漏洞影响到众多 Windows 系统上的 SMBv1服务器。SMB (服务器消息块)在 Windows 网络中广泛用于文件共享，甚至用于向打印机发送文件。2017年4月，网络犯罪集团“影子经纪人”(Shadow Brokers)泄露了永恒之蓝。2017年5月，这个漏洞在 WannaCry 勒索软件攻击中被全世界利用。

![image-20221003013852763](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003013852763.png)

你也可以使用 use 命令选择要使用的模块，并在搜索结果行的开头输入exp编号。

虽然命令行提示符已经更改，但是你能注意到我们仍然可以运行前面提到的普通命令，如 ls命令，这意味着我们并没有像你通常所期望的那样在操作系统命令行中“进入”了一个文件夹。

![image-20221003014249405](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003014249405.png)

命令行提示符告诉我们现在有了一个上下文环境，我们将在其中工作，你可以通过键入 show options 命令看到这一点。

![image-20221003014516400](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003014516400.png)

这将打印与我们之前选择的漏洞exp相关的选项，Show options 命令将根据其所用的上文下具有不同的输出。上面的例子表明，这个漏洞exp需要我们设置变量，如 RHOSTS 和 RPORT。另一方面，后渗透模块可能只需要我们设置一个 SESSION ID (参见下面的屏幕截图)。会话是目标系统的现有连接，后期漏洞利用模块将使用该连接。

![image-20221003101927605](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003101927605.png)

show 命令可以在任何上下文中使用，show命令后面跟模块类型(auxiliary、payload、exp等)以列出可用模块。下面的示例中列出了可用于 ms17-010 Eternalblue 漏洞利用的有效载荷。

![image-20221003102500829](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003102500829.png)

如果在msfconsole命令行提示符环境下（msf6>）使用，show 命令将列出所有模块。

到目前为止，我们在 Metasploit 看到的所有模块的use命令和show options命令的效果都是相同的。

你可以使用 back 命令离开当前的上下文环境。

![image-20221003104050171](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003104050171.png)

可以通过在其上下文中键入 info 命令来获得关于任何模块的进一步信息。

```shell
msf6 exploit(windows/smb/ms17_010_eternalblue) > info

       Name: MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
     Module: exploit/windows/smb/ms17_010_eternalblue
   Platform: Windows
       Arch: 
 Privileged: Yes
    License: Metasploit Framework License (BSD)
       Rank: Average
  Disclosed: 2017-03-14

Provided by:
  Sean Dillon 
  Dylan Davis 
  Equation Group
  Shadow Brokers
  thelightcosine

Available targets:
  Id  Name
  --  ----
  0   Windows 7 and Server 2008 R2 (x64) All Service Packs

Check supported:
  Yes

Basic options:
  Name           Current Setting  Required  Description
  ----           ---------------  --------  -----------
  RHOSTS                          yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:'
  RPORT          445              yes       The target port (TCP)
  SMBDomain      .                no        (Optional) The Windows domain to use for authentication
  SMBPass                         no        (Optional) The password for the specified username
  SMBUser                         no        (Optional) The username to authenticate as
  VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target.
  VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target.

Payload information:
  Space: 2000

Description:
  This module is a port of the Equation Group ETERNALBLUE exploit, 
  part of the FuzzBunch toolkit released by Shadow Brokers. There is a 
  buffer overflow memmove operation in Srv!SrvOs2FeaToNt. The size is 
  calculated in Srv!SrvOs2FeaListSizeToNt, with mathematical error 
  where a DWORD is subtracted into a WORD. The kernel pool is groomed 
  so that overflow is well laid-out to overwrite an SMBv1 buffer. 
  Actual RIP hijack is later completed in 
  srvnet!SrvNetWskReceiveComplete. This exploit, like the original may 
  not trigger 100% of the time, and should be run continuously until 
  triggered. It seems like the pool will get hot streaks and need a 
  cool down period before the shells rain in again. The module will 
  attempt to use Anonymous login, by default, to authenticate to 
  perform the exploit. If the user supplies credentials in the 
  SMBUser, SMBPass, and SMBDomain options it will use those instead. 
  On some systems, this module may cause system instability and 
  crashes, such as a BSOD or a reboot. This may be more likely with 
  some payloads.

References:
  https://docs.microsoft.com/en-us/security-updates/SecurityBulletins/2017/MS17-010
  https://cvedetails.com/cve/CVE-2017-0143/
  https://cvedetails.com/cve/CVE-2017-0144/
  https://cvedetails.com/cve/CVE-2017-0145/
  https://cvedetails.com/cve/CVE-2017-0146/
  https://cvedetails.com/cve/CVE-2017-0147/
  https://cvedetails.com/cve/CVE-2017-0148/
  https://github.com/RiskSense-Ops/MS17-010

Also known as:
  ETERNALBLUE

msf6 exploit(windows/smb/ms17_010_eternalblue) > 
```

或者，你可以使用 info 命令，后面跟着msfconsole命令行提示符中模块的路径(例如，info exploit/windows/smb/ms17\_010\_eternalblue)。Info 不是帮助菜单，它会显示模块的详细信息，如作者、相关资源等

**Search**

msfconsole中最有用的命令之一是search。此命令将在 Metasploit Framework 数据库中搜索与给定搜索参数相关的模块。

你可以使用 CVE 编号进行搜索，或者搜索exp名称(eternalblue永恒之蓝，heartbleed心脏滴血等) ，或者搜索目标系统的类型。

![image-20221003110317219](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003110317219.png)

search 命令的输出会提供每个返回的模块的概述，你可能注意到“ name”字段已经提供了比模块名更多的信息。你能看到模块的类型（auxiliary、exp等）以及模块的类别（scanner, admin, windows, Unix等）。

你可以使用搜索结果中返回的任何模块，输入命令 use 加上对应的数字即可（例如：使用 use 0代替use auxiliary/admin/smb/ms17\_010\_command）。

返回的另一个重要信息是“rank”字段，exp是根据它们的可靠性来评定的，下表提供了它们各自的描述：

![image-20221003112726237](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003112726237.png)

参考链接：https://github.com/rapid7/metasploit-framework/wiki/Exploit-Ranking

你可以使用类型(type)和平台(platform)等关键字来指导搜索功能。

例如，如果我们希望搜索结果只包含auxiliary（辅助）模块，我们可以将类型设置为auxiliary，下面的屏幕截图显示了搜索类型: auxiliary telnet 命令的输出结果

![image-20221003115845510](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003115845510.png)

请记住，exp利用了目标系统上的一个漏洞，并且可能总是显示出意想不到的行为，一个低等级的exp可能会完美地运行，而一个优秀等级的exp可能不会运行的很好，或者更糟糕----会破坏目标系统。

**答题**

使用的命令如下：

```
msfconsole
search Apache
info  auxiliary/scanner/ssh/ssh_login
```

![image-20221003121628643](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003121628643.png)

![image-20221003121417326](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003121417326.png)

## 使用模块（Working with modules ）

如前所述，使用 use 命令后跟模块名称进入模块的上下文后，将需要设置参数。下面列出了你将使用的最常用参数，记住,根据你使用的模块，你可能需要设置其他或不同的参数，最好使用 show options 命令列出当前模块所需的参数。

所有参数都使用相同的命令语法设置:

```shell
set PARAMETER_NAME VALUE
```

在继续操作之前，请记住始终检查 msfconsole 的提示符，以确保你处于正确的上下文中，在使用 Metasploit 时，你可能会看到五种不同的命令行提示符:

**常规命令提示符**：这里不能使用 Metasploit 命令（这是进入msf控制台之前的命令行状态）。

![image-20221003123115567](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003123115567.png)

**msfconsole提示符**：msf5(或 msf6，取决于你所安装的msf版本)是msf控制台的提示符，可以看到，此处没有设置上下文，因此不能在这里使用特定于上下文的命令来设置参数和运行模块。

![image-20221003123504552](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003123504552.png)

**模块的上下文提示符**:一旦你决定使用某个模块并用 set 命令来选择它，msf控制台将显示该模块的上下文。你可以在此处使用特定于上下文的命令(例如，set RHOSTS 10.10.x.x)。

![image-20221003123742241](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003123742241.png)

**Meterpreter 提示符**：Meterpreter 是一个重要的有效负载，我们将在本模块的后面详细介绍，这个提示符意味着 Meterpreter 代理已加载到目标系统并将回连到你的攻击机，你可以在此处使用 Meterpreter 特定的命令。

![image-20221003124148826](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003124148826.png)

**目标系统上的shell**：一旦exp执行完成，你可能就可以访问目标系统上的命令 shell，这是一个普通的命令行，这里输入的所有命令都将在目标系统上运行。

![image-20221003124444467](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003124444467.png)

如前所述，show options 命令将列出所有可用的参数。

![image-20221003124614769](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003124614769.png)

正如你在上面的截图中看到的，为了让exp能够成功执行，其中一些参数需要设置一个值，一些必需的参数值将被预先填充，请检查一下以确定对于你的目标而言 这些参数值是否应该保持不变。

例如，一个 web 漏洞可能有一个 RPORT参数（remote port远程端口: 目标系统上的端口，Metasploit将尝试连接它并运行exp利用程序），它的预设值为80端口，但是你的目标 Web 应用程序也可能会使用8080端口。

在本例中，我们将使用 set 命令，把 RHOSTS 参数设置为目标系统的 IP 地址。

![image-20221003125803792](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003125803792.png)

设置好参数后，可以使用 show options 命令检查参数值是否设置正确。

你经常可能使用的参数如下:

**RHOSTS**： "Remote host" 远程主机，即目标系统的 IP 地址，可以设置单个IP地址或者ip段（一个网段）。此处支持 CIDR表示法（/24，/16等）或者直接写网络段（10.10.10.x – 10.10.10.y），你还可以使用一个"列出目标ip地址"的文件，使用file:/path/of/the/target\_file.txt，在文件内容中每行都有一个目标ip，如下所示：

> CIDR是Classless Inter-Domain Routing的缩写，中文意思是：无类别域间路由。

![image-20221003131530265](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003131530265.png)

**RPORT:** "Remote port"远程端口，易受攻击的应用程序正在运行的目标系统上的端口。

**PAYLOAD**:你将用于攻击的有效载荷。

**LHOST**："Local host"本地主机，你使用的攻击机的IP地址。

**LPORT**："Local port"本地端口，用于反向shell的回连端口，这是你的攻击机器上的一个端口，你可以将其设置为任一其他应用程序没有正在使用的端口。

**SESSION**：使用 Metasploit 建立到目标系统的每个连接都将有一个会话 ID，你将在后期漏洞利用模块中使用它，它将使用现有连接来连接到目标系统。

你可以再次使用 set 命令，以不同的值重写任何已经设置好的参数，也可以使用 unset 命令清除任一指定的参数值或者使用unset all命令清除所有设定好的参数值。

![image-20221003132934885](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003132934885.png)

你可以使用setg命令设置用于所有模块的值，setg命令的使用方法和set命令类似，区别在于：如果使用 set 命令来设置当前模块的参数值，当你切换到另一个模块时，原模块的参数值就会失效；而setg 命令则允许你设置一个默认值，这个值能够跨越不同的模块使用。如果想清除 使用setg命令所设置的值，请用unsetg命令。

下面的示例使用以下流程：

1.使用ms17\_010\_eternalblue的exp

2.使用 setg 命令而不是 set 命令设置 RHOSTS 变量

3.使用 back 命令离开exp的上下文环境

4.使用auxiliary/scanner/smb/smb\_ms17\_010 (此模块是一个扫描器，用于发现 MS17-010漏洞)

5.使用show options 命令，结果显示 RHOSTS 参数填充了之前设置的目标系统 IP 地址。

![image-20221003134955849](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003134955849.png)

由上图可知：setg 命令设置的是一个全局值，该值将会一直使用 直到你退出 Metasploit 或者使用 unsetg 命令清除它为止。

**使用模块（Using modules）**

当你设置好了该exp模块的所有参数，你就可以使用exploit命令启动该exp模块。Metasploit 还支持 run 命令，这是为exploit命令创建的别名 ，因为当你使用的模块不是exploits 时，exploit这个词就没有意义了（比如当你在使用端口扫描、漏洞扫描等模块时）

在使用exploit命令时，你可以选择不带任何参数或者使用"-z "参数。

当你使用 exploit -z 命令时，将执行exp并会对打开的会话进行 立即后台化处理 。

![image-20221003140928697](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003140928697.png)

这将返回到你运行exp的界面并显示模块的上下文提示符。

有些模块支持check（检查）选项，这将检查目标系统是否易受攻击，而不进行漏洞利用（只检查而不执行exp）。

**Sessions**

一旦漏洞被成功利用，将创建一个会话，这是Metasploit与目标系统之间建立的通信渠道。

你可以使用 background 命令对会话提示符进行后台化处理，然后返回至 原模块的上下文提示符界面：

![image-20221003141857035](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003141857035.png)

或者也可以使用CTRL+Z来使会话后台化。

使用 sessions命令，可以在 msfconsole 提示符界面或者在模块的上下文提示符界面查看现有的会话。

![image-20221003142629243](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003142629243.png)

你可以使用 sessions -i 命令后跟所需的会话号，来与任何已存在的会话进行交互，

![image-20221003142938745](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003142938745.png)

**答题**

![image-20221003122545338](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221003122545338.png)

## 总结

正如我们迄今所看到的，Metasploit 是促进漏洞利用过程的有力工具，这个过程包括三个主要步骤:

找到漏洞利用（找exp）、定制漏洞利用（设置exp参数）和利用易受攻击的服务（执行exp）。

Metasploit 提供了许多模块，可以用于漏洞利用过程的每个步骤，通过本节内容学习，我们看到了 Metasploit 的基本组成部分及其各自的用途。
