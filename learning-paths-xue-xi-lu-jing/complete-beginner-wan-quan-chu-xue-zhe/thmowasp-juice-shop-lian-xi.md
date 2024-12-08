---
description: 本文相关内容：基于易受攻击的 Web 应用程序 Juice Shop，学习如何识别和利用常见的 Web 应用程序漏洞。
cover: ../../.gitbook/assets/2857591-20230614175044185-1063863239.png
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

# OWASP Juice Shop-练习

TryHackMe实验房间链接：[https://tryhackme.com/room/owaspjuiceshop](https://tryhackme.com/room/owaspjuiceshop)



## Task 1 部署实验环境&前期准备

OWASP-TOP10项目介绍：https://owasp.org/www-project-top-ten/

Juice Shop 是一个大型应用程序，它并不会涵盖OWASP-TOP10中的每个主题，此应用程序和OWASP相关的主题如下：

* [Injection](https://owasp.org/www-project-top-ten/OWASP_Top_Ten_2017/Top_10-2017_A1-Injection)（注入类漏洞）
* [Broken Authentication](https://owasp.org/www-project-top-ten/OWASP_Top_Ten_2017/Top_10-2017_A2-Broken_Authentication)（损坏的身份验证机制-身份验证失效漏洞）
* [Sensitive Data Exposure](https://owasp.org/www-project-top-ten/OWASP_Top_Ten_2017/Top_10-2017_A3-Sensitive_Data_Exposure)（敏感信息泄露）
* [Broken Access Control](https://owasp.org/www-project-top-ten/OWASP_Top_Ten_2017/Top_10-2017_A5-Broken_Access_Control)（损坏的访问控制机制-越权漏洞）
* [Cross-Site Scripting XSS](https://owasp.org/www-project-top-ten/OWASP_Top_Ten_2017/Top_10-2017_A7-Cross-Site_Scripting_\(XSS\))（跨站点脚本漏洞-XSS）

注意：

从【Task3】开始都需要获得flag以完成任务要求，flag所对应的内容会在相关任务完成后自动显示在网页上。

![image-20221208214237151](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221208214237151.png)

故障排除事项：

目标web应用程序在加载时大约需要 2-5 分钟，请耐心等待！

如果使用了Burpsuite工具，则每次完成任务时 需要先在当前浏览器的代理设置中暂时禁用burp，然后再刷新页面，页面上才会显示相关的flag内容。（这不是应用程序本身的问题，而是burp工具在拦截流量时 会自动阻止flag的内容显示）

如果你正在执行XSS任务，但是发现相关页面无法产生XSS本该有的执行效果，此时你可以尝试先清除 cookie 和网站相关数据，然后再次执行XSS Payload。

如果你确定已完成对应的任务要求 但仍然无法获得flag，你可以转到【Task8】相关页面，该页面将允许你检查所有任务的完成情况。

**答题**

![image-20221208220727163](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221208220727163.png)

启动目标机器，访问目标站点：

![image-20221210181526625](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210181526625.png)

## Task 2 遍历目标应用程序

在我们进入实际的渗透操作之前，最好先遍历一下目标。 打开BurpSuite，在Burp中，将拦截模式设置为关闭，然后直接浏览目标站点，这将允许Burp记录来自服务器的不同请求，这些请求信息稍后可能对我们的渗透操作有所帮助。

以上过程被称为遍历目标应用程序，这也是侦察形式的一种！

**答题**

**Question #1: What's the Administrator's email address?**--管理员的电子邮件地址是什么？

![image-20221210181910676](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210181910676.png)

相关产品的评论将显示每个用户的电子邮件地址，通过单击 Apple Juice 产品，我们可以看到管理员的电子邮件地址！

![image-20221210181735273](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221210181735273.png)

> admin@juice-sh.op

**Question #2: What parameter is used for searching?**--用于搜索的参数是什么？

![image-20221210182108260](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210182108260.png)

单击应用程序右上角的放大镜将弹出一个搜索栏；然后我们可以输入一些文本，按回车键将在网站的产品页面 搜索刚刚输入的文本内容。

注意浏览器地址栏的URL：它现在将更新为包含我们刚刚输入的文本，我们可以在 /#/search? 之后看到搜索参数为字母q。

![image-20221210182428085](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210182428085.png)

![image-20221210182607421](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210182607421.png)

![image-20221210182631463](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210182631463.png)

> q

**Question #3: What show does Jim reference in his review?** --吉姆在他的评论中提到了什么节目？

![image-20221210182829023](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210182829023.png)

吉姆对 Green Smoothie 产品进行了评论。 我们可以看到他提到&#x4E86;_&#x72;eplicator_。

我们用谷歌搜索引擎搜索“replicator”，得到的结果表明它来自一部名为《星际迷航》的电视节目。

![image-20221210182933730](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221210182933730.png)

![image-20221210183523271](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210183523271.png)

> Star Trek

## Task 3 注入漏洞-实验

此任务将侧重于注入类型的漏洞，注入漏洞对企业来说非常危险，因为注入攻击可能会导致目标服务器停机、数据丢失等严重后果。识别目标 Web 应用程序中的注入点通常非常简单，因为大多数注入点都会返回一个错误提示，注入攻击有很多种：

* SQL Injection：SQL注入--指攻击者输入恶意或格式错误的查询以从目标数据库中检索或篡改数据，在某些情况下，这常常发生在登录框界面。
* Command Injection：命令注入--指 Web 应用程序获取了“用户输入”或用户所控制的数据并将它们作为系统命令运行，攻击者可能会篡改输入数据以执行他们自己想要的系统命令，例如，某个目标应用程序能够执行配置错误的ping命令。
* Email Injection：电子邮件注入--是一种安全漏洞，允许恶意用户在未经电子邮件服务器事先授权的情况下向受害者发送垃圾电子邮件，攻击者可以通过向邮件头字段添加额外数据来完成攻击行为；这种攻击的主要原因是不适当的用户输入验证或者目标应用程序根本没有验证、过滤机制。

在接下来的例子中，我们将关注&#x4E8E;_**SQL 注入漏洞**_。

**答题**

**Question #1: Log into the administrator account!**--尝试登录管理员帐户！

![image-20221210190416125](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210190416125.png)

在我们导航到登录页面后，在电子邮件和密码字段中输入一些数据。

![image-20221210191030455](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210191030455.png)

在单击提交登录表单之前，请确保Burp的拦截模式已打开。

![image-20221210190334439](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210190334439.png)

这将使得我们能够在BurpSuite中看到 将要发送到目标服务器的数据！

![image-20221210192553974](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210192553974.png)

将电子邮件字段“a”更改为 **' or 1=1--** ，点击Burp中的Forward转发数据到目标服务器(直到Burp中的Proxy界面所有数据都转发完毕)，实现目标站点的登陆操作。

![image-20221210192702269](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210192702269.png)

![image-20221210200503102](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210200503102.png)

_为什么这能够成功实现登陆操作？_

1. 字符 **'** 将关闭 SQL 查询中的语句。
2. 以上SQL语句插入了“**or**”操作符，如果SQL语句中的任何一方（or操作符前的语句和后面的语句）为真，则整个SQL语句将返回true；由于 1=1 始终为真，因此整个语句的结果将为true，这将告诉目标服务器 刚才输入的电子邮件字段是有效的，然后我们就能登录到用户ID为0的账户，也就是管理员帐户。
3. 字符“**--**”在SQL语句中用于注释数据，注释符之后的 任何对登录的限制手段都将不再起作用，这类似于 python 和 javascript 中的 **#** 和 **//** 注释符。

![image-20221210193042910](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210193042910.png)

> 32a5e0f21372bcc1000a6088b93b458e41f0e02a

**Question #2: Log into the Bender account!**--登录 Bender 帐户！

![image-20221210201414273](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210201414273.png)

与问题 #1 中的操作类似，我们现在将登录 Bender 帐户！ 再次使用Burp捕获登录请求，但这次我们将把：**bender@juice-sh.op'--** 作为修改之后的 电子邮件字段内容（该有效电子邮件地址是我们通过遍历目标应用程序才得知的）。

![image-20221210202141223](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210202141223.png)

然后，将修改之后的数据转发到目标服务器！

![image-20221210202359233](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210202359233.png)

![image-20221210202420466](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210202420466.png)

此处为什么不注入1=1？这是因为我们在此处输入的电子邮件地址是有效的（相关的SQL查询将自动返回 true），所以我们不需要强制SQL查询结果为 true。我们可以在电子邮件字段之后添加 **'--** 来绕过登录系统的验证机制。 注意，只有当电子邮件或用户名未知、无效时，才需要尝试使用 1=1。

> fb364762a3c102b2db932069c0e6b78e738d4066

## Task 4 身份验证失效漏洞-实验

在此任务中，我们将研究通过不同的漏洞来利用身份验证机制，此处在谈论身份验证机制中的缺陷时，也包括了一些易受攻击者操纵的机制，下面列出的这些机制是我们将要利用的：

* 高权限帐户的弱密码
* 忘记密码页面

**答题**

**Question #1: Bruteforce the Administrator account's password!**--暴力破解管理员帐户的密码！

![image-20221210205147392](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210205147392.png)

在上一个任务中，我们已经成功利用 SQL Injection 登录到 Administrator 帐户，但我们仍然不知道相关的密码明文内容，所以我们接下来将尝试使用暴力破解攻击 以得到管理员账户密码！在登录表单输入以下字段：

> admin@juice-sh.op
>
> a

![image-20221210204524852](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210204524852.png)

使用Burp捕获上图所对应的登录请求，但此次不是通过Burp中的Proxy界面修改、发送数据，而是将捕获到的表单数据先转发给Burp中的Intruder界面（通过此界面完成暴力破解攻击）。

![image-20221210204736340](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210204736340.png)

在Intruder界面点击Positions，然后选择**clear §** 按钮，在密码字段中，将两个\*\*§**符号放在引号内；注意：此处的**§§**不是指代两个单独的输入字符，而是 Burp 中的引用符号的实现，类似于**""\*\*符号。

![image-20221210204838869](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210204838869.png)

![image-20221210204934107](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210204934107.png)

对于爆破攻击所使用的有效载荷，我们将选择来自于 Seclists 项目中的 best1050.txt列表。（Seclists 可以通过以下方式安装：`apt-get install seclists`）

你可以从以下位置 加载所需列表：`/usr/share/secLists/Passwords/Common-Credentials/best1050.txt`

![image-20221210210518209](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210210518209.png)

将字典文件加载到 Burp 后，开始爆破攻击，你可以按状态码来判断请求所对应的结果：失败的请求将收到 401 Unauthorized，成功的请求将返回 200 OK。

![image-20221210211837847](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210211837847.png)

成功完成爆破攻击之后，我们可以使用刚才得到的明文密码登录管理员帐户，以获取对应的flag。

> admin@juice-sh.op
>
> admin123

![image-20221210211959256](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210211959256.png)

![image-20221210212026875](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210212026875.png)

> c2110d06dc6f81c67cd8099ff0ba601241f1ac0e

**Question #2:** **Reset Jim's password!**--重置Jim的密码！

![image-20221210212928635](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210212928635.png)

重置密码机制也可以被利用！ 当我们在“忘记密码”页面的电子邮件字段中输入Jim的Email（jim@juice-sh.op）时，可以看到Jim 的密码安全问题被设置为“你最年长的兄弟姐妹的中间名是什么？”。

![image-20221210212443775](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210212443775.png)

在任务 2 中，我们已经发现Jim可能与星际迷航有关，所以我们通过谷歌搜索“Jim Star Trek”这为我们提供了来自 Star Trek 的关于 Jame T. Kirk 的维基百科页面；翻阅相关维基百科页面 我们发现他有一个哥哥，Jim哥哥的中间名为**Samuel**，所以我们可以将其输入到“忘记密码”页面中并尝试更改Jim账户的密码。

![image-20221210212528545](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221210212528545.png)

![image-20221210212635865](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210212635865.png)

![image-20221210212753459](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221210212753459.png)

成功修改密码之后，可以获得一个flag。

![image-20221210212836933](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210212836933.png)

> 094fbc9b48e525150ba97d05b942bbf114987257

使用修改之后的密码登录Jim的账户（jim@juice-sh.op），也能得到一个flag（本文中的任务并未要求获得该flag）

![image-20221210213237975](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210213237975.png)

> da36b345cb3d661cf985428df8c2d8756c362e56

## Task 5 敏感信息泄露-实验

Web 应用程序应该安全可靠地存储和传输敏感数据，但在某些情况下，开发人员可能没有正确保护好这些敏感数据，从而使其容易被泄露。

大多数时候，数据保护并未在整个 Web 应用程序中得到一致应用，这就使得某些页面能够直接被公众所访问，所以一些敏感信息可能会在开发人员不知情的情况下已经泄露给公众访问人员，从而可能导致目标Web应用程序容易受到非法攻击。

**答题**

**Question #1:** **Access the Confidential Document!**--访问机密文件！

![image-20221210214237801](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210214237801.png)

导航至“About Us-关于我们”页面，将鼠标悬停在“查看我们的使用条款”上。

![image-20221210214326687](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221210214326687.png) ![image-20221210214624771](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221210214624771.png)

点击上图中的链接将跳转到 http://10.10.102.75/ftp/legal.md 页面（这将自动执行文件下载操作），在此页面修改url以导航到/ftp/ 目录，你会发现该目录是一个公开的可访问页面！

![image-20221210214719502](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210214719502.png)

![image-20221210214807968](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210214807968.png)

我们尝试在/ftp/页面中 下载包含敏感信息的 _**acquisitions.md**_&#x6587;件。

![image-20221210215200468](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210215200468.png)

下载完成后，我们可以导航到之前的页面以接收对应的flag。

![image-20221210215341034](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210215341034.png)

> edf9281222395a1c5fee9b89e32175f1ccf50c5b

**Question #2:** **Log into MC SafeSearch's account!**--登录 MC SafeSearch 的帐户！

TryHackMe在此提供了一个YouTube视频链接，当我们看完对应的歌曲MV视频后，我们可以得到一些关键信息：目标用户的密码本应是“_**Mr. Noodles**_”，但他已经将一些“元音字母替换为零”，这意味着将字母`o`替换为`0`，所以我们可知`mc.safesearch@juice-sh.op`帐户的密码是“_**Mr. N00dles**_”

![image-20221210221014132](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210221014132.png)

![image-20221210221103738](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210221103738.png)

tips：以上两个截图中所显示的字幕 是自动生成的字幕（不一定非常准确）。

使用已知的密码登录MC SafeSearch的帐户（`mc.safesearch@juice-sh.op`、`Mr. N00dles`），并获得一个flag：

![image-20221210221605624](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210221605624.png)

![image-20221210221726421](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210221726421.png)

> 66bdcffad9e698fd534003fbb3cc7e2b7b55d7f0

**Question #3:** **Download the Backup file!**--下载备份文件！

![image-20221210222240373](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210222240373.png)

我们现在返回至之前所提到的 http://10.10.102.75/ftp/ 页面并尝试下载 `package.json.bak`文件，但我们接收到了一个403状态码提示，该提示告诉我们只允许下载 `.md` 和 `.pdf` 格式的文件。

![image-20221210222224161](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210222224161.png)

为了解决这个问题，我们将使用一个名为“Poison Null Byte”的字符来进行绕过，该空字节内容如下所示：`%00`。

注意：由于我们在此处是使用 url 来下载文件，因此我们需要将空字节也编码为 url 编码格式；经过url编码处理的Poison Null Byte 将变为：`%2500`。

我们在url（ http://10.10.102.75/ftp/package.json.bak ）末尾添加空字节以及 `.md` ，就能绕过403 错误提示以完成文件下载操作！

```http
http://10.10.102.75/ftp/package.json.bak%2500.md
```

![image-20221210223103652](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210223103652.png)

原理：Poison Null Byte 实际上是一个 NULL 终止符，在字符串中的某个字节处放置一个 NULL 字符，那么该字符串将告诉目标服务器在NULL字符处终止读取，然后字符串的其余部分将被置为空。

返回主页面以获得一个flag：

![image-20221210223624952](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210223624952.png)

> bfc1e6b4a16579e85e06fee4c36ff8c02fb13795

## Task 6 损坏的访问控制(越权)-实验

现代系统将允许多个用户访问不同的页面，其中管理员用户可以使用管理页面以方便编辑、添加和删除网站的不同元素，当你使用 Weebly 或 Wix 等程序构建网站时，你就可能会用到管理员页面来辅助建站。

当发现损坏的访问控制漏洞或错误时，它将被归类为以下两种类型之一：

* 水平（横向）越权：当用户可以执行 操作或访问具&#x6709;_**相同权限级别**_&#x7684;另一个用户的数据时发生。
* 垂直（纵向）越权：当用户可以执行 操作或访问具&#x6709;_**更高权限级别**_&#x7684;另一个用户的数据时发生。

![image-20221210224427073](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221210224427073.png)

**答题**

**Question #1:** **Access the administration page!**--访问管理页面！

![image-20221211185152085](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211185152085.png)

首先，我们将在Firefox浏览器中打开**Debugger**（调试器）或者在谷歌浏览器中打开**Sources**，Debugger可以通过在浏览器中打开Web开发者工具找到，找到Debugger之后再刷新页面，找到并查看main-es2015.js文件，要将其内容转换为我们可以阅读的格式，单击 { } 按钮即可。

![image-20221211182647790](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221211182647790.png)

![image-20221211183254858](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211183254858.png)

![image-20221211182819067](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211182819067.png)

现在在上图的js文件内容中 搜索关键词“admin”，你可以找到几个包含“admin”的不同代码段，最终我们所发现的是“path: administration”，它指代了一个名为“/#/administration”的页面路径，但是在未登录的情况下无法进入该路径相关页面。

![image-20221211184207274](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221211184207274.png)

由于这是一个管理员页面，因此我们需要在管理员帐户中才能查看它。阻止普通用户访问管理员界面的一个好方法是 只加载普通用户所需要使用的部分应用程序，这可以防止泄露一些敏感信息或阻止查看管理页面等。

接下来根据前文所得的信息，我们先登录到管理员账户，再访问“/#/administration”页面，获取对应的flag：

> admin@juice-sh.op
>
> admin123

![image-20221211184945260](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211184945260.png)

![image-20221211185101249](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20221211185101249.png)

> 946a799363226a24822008503f5d1324536629a0

**Question #2:** **View another user's shopping basket!**--查看其他用户的购物篮！

![image-20221211191009253](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211191009253.png)

开启BurpSuite，在管理员帐户登录后的界面中单击“你的购物篮”，然后在Burp页面转发捕获到的请求，直到你看到`GET /rest/basket/1 HTTP/1.1` ，接着使用Burp将`/basket/`之后的数字 1 更改为 2，现在和该请求所对应的页面 就可以显示 UserID为2 的用户购物篮！

![image-20221211190631857](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211190631857.png)

![image-20221211190724073](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211190724073.png)

使用Burp转发所有请求，然后在浏览器中 查看修改之后的请求所对应的页面以便得到一个flag。

_**UserID为2 的用户购物篮（下图是通过Burp修改请求才得到的界面）：**_

![image-20221211190847609](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211190847609.png)

_**UserID为1 的用户购物篮（admin用户的原始购物车界面）：**_

![image-20221211191139272](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211191139272.png)

> 41b997a36cc33fbe4f0ba018474e19ae5ce52121

**Question #3: Remove all 5-star reviews!**--删除所有 5 星评论！

![image-20221211192033307](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211192033307.png)

在管理员账户下：再次导航到 http://10.10.103.124/#/administration 页面，然后单击带有 5 星评论旁边的垃圾桶图标！这将删除所有5星评论。

![image-20221211191837902](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211191837902.png)

完成上述操作之后，得到一个flag：

![image-20221211191940560](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211191940560.png)

> 50c97bcce0b895e446d61c83a21df371ac2266ef

## Task 7 跨站点脚本(XSS漏洞)-实验

XSS 或跨站点脚本是一种允许攻击者在 Web 应用程序中运行 恶意javascript脚本 的漏洞，是 Web 应用程序中最常见的漏洞类型之一；XSS的复杂度从简单到极其困难都有所包含，因为每个 Web 应用程序可能会以不同的方式来解析具有查询功能的js代码。

XSS攻击主要分为三种类型：

**基于DOM的XSS（特殊的反射型XSS）**：基于DOM的XSS (Document Object Model-based Cross-site Scripting) 可以利用 HTML 环境执行恶意 javascript代码，这种类型的XSS攻击通常会使用 `<script></script>` HTML 标记然后结合 _**支持动态代码执行的接收器**_ 来构造Payload，此外，这种类型的XSS攻击还可以通过直接修改DOM元素来构造Payload。

一些常见的接收器：

```txt
document.write()
document.writeln()
document.domain
someDOMElement.innerHTML
someDOMElement.outerHTML
someDOMElement.insertAdjacentHTML
someDOMElement.onevent
```

例子:利用innerHTML篡改页面

```html
<script>document.body.innerHTML="<div style=visibility:visible;><h1>This is DOM XSS</h1></div>";</script>
```

**持久型(存储型)XSS（服务器端）**：持久型XSS 是在服务器加载包含它的页面时才会运行的 恶意javascript脚本（js脚本已插入到网站数据库中），当服务器在将用户所输入的数据上传（upload）到网站页面 而未对用户数据进行正确过滤时，就可能会发生这种XSS攻击，此类攻击手段在博客文章页面很常见。

**反射型XSS（客户端）**：反射型XSS 是在 Web 应用程序的客户端运行的恶意javascript脚本（因受害者点击恶意链接而导致恶意js脚本运行），常见于目标服务器未正确过滤搜索（search）数据时。

**答题**

**Question #1:** **Perform a DOM XSS!**--执行基于DOM的XSS攻击

![image-20221211204425039](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211204425039.png)

我们将利用带有 javascript 警告标记(alert tag)的 iframe 元素来构造攻击载荷（以下payload中包含xss单词的符号是反引号而不是单引号）：

![image-20221211203130320](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211203130320.png)

_**注意**_：我们在此处利用的是`iframe`元素，它是Web应用程序中常见的 HTML 元素之一，还有其他能够产生相同攻击效果的HTML元素；使用以上这种Payload类型的XSS也可称为XFS（跨框架脚本-Cross-Frame Scripting），XFS是能够在 Web 应用程序中被检测到的 XSS 的最常见形式之一，一个允许用户修改 `iframe` 或者其他DOM元素的网站很容易受到XSS攻击。

_**原理**_：当用户在搜索栏中输入关键字进行搜索时，这通常会导致客户端向服务器发送相关的请求，然后服务器将根据搜索栏中的关键字来返回一些相关信息；如果web应用程序没有做一些正确的“用户输入”过滤，那么我们就能够利用搜索栏来执行 XSS 攻击。

将XSS Payload输入到搜索栏并按下回车键，这将触发并弹出一个警告框并将显示对应的flag内容。

![image-20221211203417837](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211203417837.png)

![image-20221211204319635](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211204319635.png)

> 9aaf4bbea5c30d00a1f5bbcfce4db6d4b0efe0bf

**Question #2:** **Perform a persistent XSS!**--执行持久型(存储型)XSS攻击

![image-20221211210357475](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211210357475.png)

首先，登录到管理员帐户，然后导航到“Last Login IP”页面。

![image-20221211210506830](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211210506830.png)

![image-20221211210540469](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211210540469.png)

我们可以从上图看到，最后登陆的ip地址显示为0.0.0.0 ，现在我们注销此次登陆，这样就能记录一个新的“最后登陆ip”，同时还要确保打开 Burp 拦截，以便我们捕获这个注销登陆的请求。

然后我们在Burp中转到Headers 选项卡，在其中添加一个新的header项。（以下payload中包含xss单词的符号是反引号而不是单引号）

![image-20221211213606372](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211213606372.png)

![image-20221211214658568](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211214658568.png)

修改完成之后，在Burp中将请求转发给目标服务器即可。 当我们重新登录管理员帐户并再次导航到“Last Login IP”页面时，我们就能看到一个 XSS 警告框，并能获取对应的flag。

![image-20221211215007054](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211215007054.png)

> 149aa8ce13d7a4a8a931472308e269c94dc5f156

**Question #3:** **Perform a reflected XSS!**--执行反射型XSS攻击

![image-20221211220959120](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211220959120.png)

首先，我们需要在正确的页面上执行反射型XSS！所以我们需要登录到管理员帐户并导航至“订单历史记录”页面。

![image-20221211215258168](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211215258168.png)

![image-20221211215606014](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211215606014.png)

在上图，你可以看到一个“Truck-卡车”图标，点击图标将会导航至订单的追踪结果页面，在该追踪结果界面你还能看到一个与订单配对的 ID。

![image-20221211215831384](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211215831384.png)

我们可以用以下Payload代替上图url栏中的id值。（以下payload中包含xss单词的符号是反引号而不是单引号）

![image-20221211203130320](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211203130320.png)

提交 URL 后，刷新页面，你将收到一个 XSS 警告框，并能获取到对应的flag。

![image-20221211220922027](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221211220922027.png)

> 23cefee1527bde039295b2616eeb29e1edc660a0

## Task 8 计分板

如果你想尝试本文未涵盖的一些更难的挑战，请查看 Juice-shop 上的\*\*/#/score-board/\*\*部分，在该部分你可以看到你已经完成的任务 以及 其他本文未涵盖的不同难度的任务。

![image-20221208221153945](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221208221153945.png)

**答题**

![image-20221208221225637](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221208221225637.png)

使用攻击机访问/#/score-board/页面，获取对应的flag内容：

![image-20221210183807295](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221210183807295.png)

> 7efd3174f9dd5baa03a7882027f2824d2f72d86e
