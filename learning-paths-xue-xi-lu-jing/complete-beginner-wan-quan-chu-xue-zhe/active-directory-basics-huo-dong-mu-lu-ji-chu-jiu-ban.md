---
description: 本文相关内容：了解Active Directory(活动目录)的基础知识及其在现实世界中的使用方式。
cover: ../../.gitbook/assets/2857591-20230516214618727-1819928694.png
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

# Active Directory Basics(活动目录基础·旧版)

TryHackMe实验房间链接：[https://tryhackme.com/room/activedirectorybasics](https://tryhackme.com/room/activedirectorybasics)



## 简介

Active Directory(活动目录)是Windows域网络的目录服务，它被当今许多顶级公司使用，并且是我们在攻击Windows机器时需要理解的重要技能。

注意：本文可能不会涉及有关活动目录的特定用途和特定对象的细节信息，因为本文只是关于Active Directory(活动目录)的一般概述。

![image-20230514114423317](https://img2023.cnblogs.com/blog/2857591/202305/2857591-20230516214619233-278755284.png)

**什么是活动目录(Active Directory)？**

Active Directory是域内互相连接的计算机和服务器的集合，同时也是组成AD网络的更大域森林的集合部分。Active Directory包含了许多功能组件，我们将在下文介绍其中的大部分。为了概述我们将要讨论的内容，请查看以下的Active Directory组件列表，并熟悉Active Directory的各个组成部分：

* Domain Controllers(域控制器)
* Forests, Trees, Domains(森林、树、域)
* Users + Groups(用户+组)
* Trusts(信任)
* Policies(策略)
* Domain Services(域服务)

所有这些Active Directory的组成部分形成了一个庞大的计算机和服务器网络。

**为什么要使用活动目录(Active Directory)？**

大多数大公司使用Active Directory是因为它允许通过一个域控制器来控制和监视员工用户的电脑。

域控制器允许单个用户登录到活动目录(AD)网络上的任何计算机，并能让用户有权访问他在服务器上所存储的文件和文件夹，以及访问已登录的计算机上的本地存储内容。当一家公司使用了Active Directory之后，就能允许该公司中的任何员工用户使用公司所拥有的任何机器，而不必在一台机器上设置多个用户帐户。

现在我们已经知道了Active Directory的组件和用途，让我们在下文中继续讨论它的工作原理和功能。

## 活动目录物理组件

Active Directory物理组件是企业本地的服务器和计算机，它们可以是域控制器、存储服务器、域用户计算机等除了软件之外的Active Directory环境所需的任何硬件资源。

![image-20230514183719796](https://img2023.cnblogs.com/blog/2857591/202305/2857591-20230516214619685-366686399.png)

**域控制器**

域控制器是指安装了Active Directory域服务(AD DS)并且已经在森林中升级为域控制器的Windows服务器，域控制器是 Active Directory(活动目录)的中心——它能够控制域的其余部分。以下是关于域控制器的工作内容的概述：

* 可以控制AD DS(Active Directory Domain Services)数据存储 ；
* 能够处理身份验证机制以及授权服务 ；
* 能够基于森林中的其他域控制器进行复制更新；
* 允许管理员访问并管理域资源。

![image-20230514190946958](https://img2023.cnblogs.com/blog/2857591/202305/2857591-20230516214620086-462795813.png)

**AD DS 数据存储**

Active Directory域服务(Domain Services)数据存储能够保存管理目录信息(如用户、组和服务等)所需的数据库和进程，下面概述了AD DS数据存储的一些内容和特征:

* 包含了NTDS.dit数据库，该数据库将会存储Active Directory域控制器的所有信息以及域用户的密码哈希值。
* 默认存储在%SystemRoot%\NTDS中。
* 只有域控制器才能访问。

**答题**

阅读本小节内容，回答以下问题。

![image-20230514192423344](https://img2023.cnblogs.com/blog/2857591/202305/2857591-20230516214620457-1360255343.png)

## 森林(Forest)

﻿森林是将AD网络中的所有其他部分连接在一起的容器——没有森林，所有树和域将无法交互。森林既是一个物质的东西，也是一个比喻的东西，在本文中它仅仅是描述在AD网络中的树和域之间创建连接的一种方式。

![image-20230515070348128](https://img2023.cnblogs.com/blog/2857591/202305/2857591-20230516214620847-1715853892.png)

**森林概述**

森林是Active Directory网络中一个或多个域树的集合，它用于将AD网络的各个组成部分归类为一个整体。

森林通常可由以下这些部分组成(我们将在下文做进一步的介绍)：

* Trees(树) - Active Directory域服务中域的层次结构；
* Domains(域) - 用于对对象进行分组和管理 ；
* Organizational Units (OUs-组织单元) - 用于组、计算机、用户、打印机和其他 OU 的容器；
* Trusts(信任) - 允许用户访问其他域中的资源；
* Objects(对象) - 用户、组、打印机、计算机、共享等；
* Domain Services(域服务) - DNS服务器，LLMNR，IPv6等；
* Domain Schema(域架构) - 创建对象的规则。

**答题**

阅读本小节内容，回答以下问题。

![image-20230515070324396](https://img2023.cnblogs.com/blog/2857591/202305/2857591-20230516214621262-2026798211.png)

## 用户和组(Users + Groups)

活动目录(Active Directory)中的用户和组可以由AD使用者自己设置，当我们在活动目录网络中创建了一个域控制器时，它通常会带有默认的组和两个默认用户：管理员(Administrator)和来宾(guest)；而是否创建新用户以及是否创建要向其添加用户的新组则主要取决于我们自己。

**用户概述**

用户是Active Directory(活动目录)的核心，我们可以在Active Directory网络中找到四种主要类型的用户; 但是，根据不同企业管理其AD网络中用户权限的方式，有时我们也可以找到更多类型的用户。在Active Directory网络中，四种主要类型的用户分别是：

* Domain Admins(域管理员) - 这些用户可以控制整个域，同时也是唯一可以访问域控制器的用户群体。
* Service Accounts (服务帐户，有时可以是域管理员) - 除了在服务维护时需要之外，大多数情况下服务帐户可能从未被使用过，Windows系统会将这些服务帐户用于SQL之类的具体计算机服务，并且特定的服务与特定服务帐户具有对应关系。
* Local Administrators(本地管理员) - 这些用户可以作为管理员对本地计算机进行更改，甚至可以控制其他普通用户，但是他们不能访问域控制器。
* Domain Users(域用户) - 这些是域中的日常用户，他们可以登录他们有权访问的域内计算机，并且可以根据企业的分配而拥有一些计算机的本地管理员权限。

![image-20230515161836488](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230515161836488.png)

**组概述**

通过将用户和对象组织成具有指定权限的组，我们可以更加容易地向它们(用户和对象)授予权限，在Active Directory中有两种主要类型的组：

* Security Groups(安全组) - 这些组可用于为大量用户指定权限。
* Distribution Groups(通讯组) - 这些组用于指定电子邮件通讯列表；此类组对于攻击者而言用处有限，但是它们可能有助于我们进行枚举操作。

![image-20230515161929992](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230515161929992.png)

**默认安全组**

有很多默认的安全组，因此在此我们不对每个安全组进行过多的详细介绍，而是简要描述它们为所分配的组提供的权限。以下是关于常见的一些安全组的简要概述：

* Domain Controllers(域控制器组) - 此组中的成员包括域中的所有域控制器。
* Domain Guests(域来宾组) - 此组中的成员包括所有域来宾用户。
* Domain Users(域用户组) - 此组中的成员包括所有域用户。
* Domain Computers(域计算机组) - 此组中的成员包括加入域的所有工作站和服务器。
* Domain Admins(域管理员组) - 此组中的成员包括指定的域管理员。
* Enterprise Admins(企业管理员组) - 此组中的成员包括指定的企业管理员。
* Schema Admins(架构管理员组) - 此组中的成员包括指定的架构管理员。
* DNS Admins(DNS管理员组) - 此组中的成员为DNS管理员。
* DNS Update Proxy(DNS更新代理组) - 包括允许代表其他客户端(如 DHCP 服务器)执行动态更新的 DNS 客户端。
* Allowed RODC Password Replication Group(允许 RODC 密码复制组) - 此组中的成员可以将其密码复制到域中的所有只读域控制器。
* Group Policy Creator Owners(组策略创建所有者组) - 此组中的成员可以修改域的组策略。
* Denied RODC Password Replication Group(拒绝 RODC 密码复制组) - 此组中的成员不能将其密码复制到域中的任何只读域控制器。
* Protected Users(受保护的用户组) - 这个组的成员可以得到额外的保护以防御身份验证安全威胁，更多信息参见 http://go.microsoft.com/fwlink/?LinkId=298939 。
* Cert Publishers(证书发布组) - 这个组的成员被允许将证书发布到目录。
* Read-Only Domain Controllers(只读域控制器组) - 此组的成员是域中的只读域控制器。
* Enterprise Read-Only Domain Controllers(企业只读域控制器组) - 此组的成员是企业中的只读域控制器。
* Key Admins(密钥管理员组) - 此组中的成员可以对域中的密钥对象执行管理操作。
* Enterprise Key Admins(企业密钥管理员组) - 此组中的成员可以对森林中的密钥对象执行管理操作。
* Cloneable Domain Controllers(可克隆的域控制器组) - 此组中作为域控制器的成员可以被克隆。
* RAS and IAS Servers(RAS 和 IAS 服务器组) - 此组中的服务器可以访问用户的远程访问属性。

你可以通过访问以下链接来获取关于默认安全组的完整列表：

> https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-groups

**答题**

阅读本小节内容，回答以下问题。

![image-20230516085406862](https://img2023.cnblogs.com/blog/2857591/202305/2857591-20230516214621630-1353689027.png)

## 信任和策略(Trusts + Policies)

信任和策略相辅相成，它们能够帮助域和树互相通信，并且能够在域网络内部更好地维护域的“安全性”，它们会将规则应用于森林内部的域如何互相交互、外部的森林如何与当前森林进行交互以及描述一个域必须遵循的总体域规则或域策略。

**域信任概述**

信任是一种适当的机制，能够让域网络中的用户获得对域中其他资源的访问权限。在大多数情况下，信任概述了同一森林中的域之间相互通信的方式，在某些环境中，信任也可以扩展到外部的域，甚至可以扩展到外部的森林。

有两种类型的域信任可以决定域之间是如何互相通信的，以下是相关的概述：

* (Directional)方向性的信任：域信任的方向将从一个信任域流向另外一个被信任域。
* (Transitive)传递性的信任：域信任关系可以扩展到两个域之外，从而能够包括其他受信任的域。

已经设置的域信任类型能够决定森林中的域和树如何进行通信以及发送数据，在攻击活动目录环境时，我们有时可以滥用这些信任关系，以便在整个AD网络中执行横向移动操作。

**域策略概述**

域策略是Active Directory的重要组成部分，它们可以规定服务器如何操作，以及服务器将遵循和不遵循哪些具体规则。我们可以将域策略想象成域组，只不过它们不包含权限，而是包含规则，并且域策略不仅仅是应用于一组用户，而是会应用于整个域。

域策略可以作为AD的规则手册，然后域管理员可以根据需要对域策略进行修改，以保持域网络平稳、安全地运行。除了很长的默认域策略列表之外，域管理员还可以选择添加他们自己的域控制器策略，例如，如果你想禁用域中所有机器的Windows Defender，你可以创建一个新的组策略对象来禁用Windows Defender。

关于域策略的选项非常多，在枚举 Active Directory网络时，域策略是我们作为攻击者要考虑的一个重要因素。接下来，我们将概述一些默认的域策略，换言之我们可以在Active Directory环境中创建以下域策略：

* 禁用Windows Defender - 可以禁用域内所有机器的Windows Defender。
* 数字签名通信(Always) - 可以在域控制器上禁用或启用SMB签名

**答题**

阅读本小节内容，回答以下问题。

![image-20230516122712793](https://img2023.cnblogs.com/blog/2857591/202305/2857591-20230516214622017-956437388.png)

## Active Directory域服务和域身份验证

Active Directory域服务是Active Directory网络的核心功能，AD域服务能够允许管理域、安全证书、 LDAP等等。通过使用AD域服务，域控制器可以决定它想要做什么，以及它想要为域提供什么服务。

![image-20230516123316829](https://img2023.cnblogs.com/blog/2857591/202305/2857591-20230516214622476-1091172284.png)

**域服务概述**

域服务是域控制器向域或树的其余部分所提供的服务。虽然有各种各样的服务可以被添加到域控制器中，但是，本文只会介绍在设置Windows服务器作为一个域控制器时的一些默认服务。以下是关于默认的域服务的简述：

* LDAP：轻型目录访问协议(Lightweight Directory Access Protocol)，能够提供应用程序与目录服务之间的通信。
* Certificate Services(证书服务)：能够允许域控制器创建、验证和撤销公钥证书。
* DNS, LLMNR, NBT-NS：能够识别IP主机名的域名服务(Domain Name Services)。

**域认证概述**

已经设置的身份验证协议是Active Directory最重要的部分以及Active Directory最脆弱的部分，Active Directory有两种主要的身份验证类型：NTLM 和 Kerberos。

* Kerberos：Kerberos是Active Directory的默认身份验证服务，它会使用票据授予票据(TGT)和服务票据(ST)对用户进行身份验证，并能允许用户访问域中的其他资源。
* NTLM：NTLM是默认的Windows身份验证协议，它是使用加密技术的询问/响应协议

Active Directory域服务是攻击者的主要访问点，并且AD域服务会包含一些易受攻击的Active Directory协议。

**答题**

阅读本小节内容，回答以下问题。

![image-20230516190745945](https://img2023.cnblogs.com/blog/2857591/202305/2857591-20230516214622995-1570300239.png)

## 云AD简单介绍

除了通常的AD网络之外，企业还能够基于Active Directory使用云网络。最著名的云AD服务提供商是Azure AD，它的默认设置比企业内部部署的Active Directory网络要安全得多，但是，云AD仍然可能存在安全漏洞。

![image-20230516191147900](https://img2023.cnblogs.com/blog/2857591/202305/2857591-20230516214623310-793534738.png)

**Azure AD概述**

Azure能够充当物理的Active Directory和用户登录之间的中间人，这允许在域之间进行更安全的事务处理，从而可以使许多常规的Active Directory攻击手段无效。

![image-20230516191402880](https://img2023.cnblogs.com/blog/2857591/202305/2857591-20230516214623667-1579833753.png)

**云安全概述**

为了更好地展示云AD如何采取更安全的预防措施，我们可以将云Active Directory环境与常规的Active Directory网络进行比较：

| **Windows Server AD**     | **Azure AD**   |
| ------------------------- | -------------- |
| LDAP                      | Rest APIs      |
| NTLM                      | OAuth/SAML     |
| Kerberos                  | OpenID         |
| OU Tree                   | Flat Structure |
| Domains and Forests(域和森林) | Tenants        |
| Trusts(信任)                | Guests         |

可供参考的官方文档：https://learn.microsoft.com/zh-cn/azure/active-directory/fundamentals/active-directory-whatis

以上只是对云Active Directory的简要概述，并未详细介绍云AD中的常见协议。

**答题**

阅读本小节内容，回答以下问题。

![image-20230516193955466](https://img2023.cnblogs.com/blog/2857591/202305/2857591-20230516214624066-862835166.png)

## 实例练习

接下来，我们将通过使用PowerShell命令查看AD域中的计算机、用户和组来了解Active Directory的内部结构。

**实验设置**

1. 部署目标机器；
2. 使用SSH或RDP访问目标机器，也可以基于TryHackMe实验房间所提供的浏览器界面访问目标机。

登录凭据如下：

* 用户：`Administrator`
* 密码：`password123@`
* 域：`CONTROLLER.local`

**PowerView设置**

先安装[PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)工具(在本文的实验环境中，该工具已经被放置在目标机器中)。

1. `cd Downloads`：导航到PowerView所在目录；
2. `powershell -ep bypass`：加载一个绕过执行策略的powershell shell；
3. `. .\PowerView.ps1`：导入PowerView模块。

![image-20230516195106326](https://img2023.cnblogs.com/blog/2857591/202305/2857591-20230516214624466-306726595.png)

**实验概述**

以下是一些示例命令：

* `Get-NetComputer -fulldata | select operatingsystem`：用于获取域中所有操作系统的列表。

![image-20230516200934907](https://img2023.cnblogs.com/blog/2857591/202305/2857591-20230516214624806-1717888475.png)

* `Get-NetUser | select cn`：用于获取域中所有用户的列表。

![image-20230516200947163](https://img2023.cnblogs.com/blog/2857591/202305/2857591-20230516214625129-2073567137.png)

可供参考的PowerView命令备忘单：https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993

**答题**

部署好目标机器，并在目标机上的Powershell环境下导入PowerView模块以便查看Active Directory的内部结构。

```powershell
cd Downloads  #导航到PowerView所在目录

powershell -ep bypass  #加载一个绕过执行策略的powershell shell

. .\PowerView.ps1  #导入PowerView模块(注意两个.符号之间有空格)
```

![image-20230516210304465](https://img2023.cnblogs.com/blog/2857591/202305/2857591-20230516214625494-1050084712.png)

使用`Get-NetComputer -fulldata | select operatingsystem`获取域中所有操作系统的列表：

![image-20230516210541222](https://img2023.cnblogs.com/blog/2857591/202305/2857591-20230516214625859-1136671690.png)

使用`Get-NetUser | select cn`获取域中所有用户的列表：

![image-20230516210836234](https://img2023.cnblogs.com/blog/2857591/202305/2857591-20230516214626174-498034043.png)

使用`Get-NetGroup -GroupName *`获取域中的组名称：

![image-20230516211131689](https://img2023.cnblogs.com/blog/2857591/202305/2857591-20230516214626612-2046393942.png)

使用`Get-NetUser -SPN | ?{$_.memberof -match 'Domain Admins'}`或`Get-ADUser -identity SQLService -properties *`查看域中的SQL服务信息：

![image-20230516211258391](https://img2023.cnblogs.com/blog/2857591/202305/2857591-20230516214627090-1494859144.png)

![image-20230516211919007](https://img2023.cnblogs.com/blog/2857591/202305/2857591-20230516214627618-1980409466.png)

![image-20230516212036034](https://img2023.cnblogs.com/blog/2857591/202305/2857591-20230516214628044-1742452860.png)

![image-20230516211802863](https://img2023.cnblogs.com/blog/2857591/202305/2857591-20230516214628532-1023986349.png)
