---
description: 本文相关内容：探索Web应用程序(即Web网站)中的一些基本的文件上传漏洞。
---

# Upload Vulnerabilities(文件上传漏洞)

TryHackMe实验房间链接：https://tryhackme.com/room/uploadvulns





## 配置在线练习环境

使用你喜欢的文本编辑器，打开设备上的 hosts 文件:

在 Linux 和 MacOS 上，hosts文件可以通过终端在 /etc/hosts 路径下找到。 在 Windows 上，hosts文件可以在 C:\Windows\System32\drivers\etc\hosts 路径下找到（可以不通过终端）。

在 Linux 或 MacOS 上，您将需要在终端界面使用 sudo 打开文件进行写入。 在 Windows 中，你需要用“ Run as Administrator”打开该文件（先在电脑中找到该文件）。

```
这里主要介绍在kali等linux系统中的操作方法：

使用命令,在hosts文件的内容末尾添加代码行:
echo 10.10.190.98    overwrite.uploadvulns.thm shell.uploadvulns.thm java.uploadvulns.thm annex.uploadvulns.thm magic.uploadvulns.thm jewel.uploadvulns.thm demo.uploadvulns.thm >>/etc/hosts

注意: 
如果你在此之前已经完成了上述步骤，那么你必须删除以前的host条目。
hosts文件中应该只有一行包含上述 URL。
例如，下面的示例将不起作用:

10.10.10.10    overwrite.uploadvulns.thm shell.uploadvulns.thm java.uploadvulns.thm annex.uploadvulns.thm magic.uploadvulns.thm jewel.uploadvulns.thm demo.uploadvulns.thm

10.10.190.98    overwrite.uploadvulns.thm shell.uploadvulns.thm java.uploadvulns.thm annex.uploadvulns.thm magic.uploadvulns.thm jewel.uploadvulns.thm demo.uploadvulns.thm
```

![image-20220924184856252](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220924184856252.png)

## 上传漏洞简介和一般利用方法

**上传漏洞简介**

将文件上传到服务器的能力已经成为我们如何与 Web应用程序进行交互的一个组成部分。 无论是社交媒体网站的个人资料图片、上传到云存储的报告，还是在 Github 上保存项目，具有文件上传功能的应用程序都是无限的。

不幸的是，如果处理不当，文件上传也会在服务器中引发严重的漏洞，这可能会导致一系列麻烦的问题的出现。 如果攻击者设法上传并执行 shell，则可能导致完全的远程代码执行(Remote Code Execution，RCE)。 通过不受限制地上传到服务器(以及随意检索数据的能力) ，攻击者可以破坏或以其他方式改变现有内容——包括注入恶意网页，从而导致进一步的漏洞，如 XSS 或 CSRF。 通过上传任意文件，攻击者还可能使用服务器来承载和/或提供非法内容，或泄漏敏感信息。 实际上，攻击者如果能够将自己选择的文件上传到你的服务器上(没有任何限制) ，那么这种攻击实际上是非常危险的。

本文的目的是探讨由于不正确(或不充分)处理文件上传操作 而导致的一些漏洞。具体来说，我们将关注: 1.文件上传 覆盖服务器上的现有文件 2.文件上传 在服务器上上传和执行 Shell 3.文件上传 绕过客户端过滤 4.文件上传 绕过各种服务器端过滤 5.文件上传 欺骗内容类型验证检查

**一般利用方法**

```
那么，我们现在发现一个网站上有一个文件上传点，我们该如何利用它呢？

对于任何类型的黑客攻击，枚举都是关键。我们越了解我们的环境，我们就越能利用它。
查看页面的源代码有助于了解是否应用了任何类型的客户端筛选。
使用像 Gobuster 这样的目录爆破工具进行扫描 通常对网络攻击很有帮助，并且可能会显示文件上传到了哪里，
Kali默认情况下不会安装Gobuster，但是可以通过命令：sudo apt install gobuster 进行gobuster的安装。
使用Burpsuit拦截上传请求也会派上用场。
诸如Wappalyzer之类的浏览器扩展插件可以提供关于目标站点的有价值的信息。

对网站如何处理我们的输入有一个基本的了解，然后我们就可以尝试四处查看，看看我们能上传什么，不能上传什么。
如果网站使用了客户端过滤，那么我们可以很容易地查看过滤器的代码，并寻找绕过它，
如果网站有服务器端过滤的地方，那么我们可能需要猜测一下过滤器在寻找什么，上传一个文件，如果上传失败，再根据错误消息尝试不同的方法。
上传能够引发错误的文件可以帮助判断服务器端过滤的情况。
在这个阶段，像 Burpsuite或者OWASP Zap这样的工具非常有用。
```

## 覆盖服务器上的现有文件

### 理论讲解

当文件上传到服务器时，服务器应该进行一系列检查，以确保文件不会覆盖服务器上已经存在的任何内容。 通常的做法是给文件分配一个新名称（一般是随机的），或者将上传的日期和时间添加到原始文件名的开头或结尾， 或者可以应用检查来查看文件名是否已经存在于服务器上，如果同名文件已经存在，那么服务器将返回一个错误消息，要求用户选择一个不同的文件名。 当保护现有文件不被覆盖时，文件权限也会发挥作用：例如，Web用户不应该对 Web 页面有可写权限，从而防止web页面被攻击者上传的恶意版本所覆盖。

如果目标没有采取这样的预防措施，那么我们可能就能够覆盖目标服务器上的现有文件。 实际上，服务器上的文件权限可能会防止这种情况成为一个严重的漏洞，在挖掘漏洞的时候 关于服务器的文件权限值得我们密切关注。

例子：

在下图中，我们有一个带有上传表单的网页:

![image-20220924165157794](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220924165157794.png)

对于真正的挑战，你可能需要列举更多内容; 但是，这只是一个简单的例子，在这个实例中，让我们看一下页面的源代码:

![image-20220924165313511](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220924165313511.png)

在上图的红色框中，我们能看到负责显示页面上的图像的代码。图片来源于一个名为“ spaniel.jpg”的文件，该图片在一个名为“ images”的目录中。

现在我们知道图像是从哪里提取的了，我们能覆盖它吗？

让我们从互联网上下载另一张图片，命名为 spineel.jpg。然后我们会把它上传到网站，看看是否能覆盖现有的图片:

![image-20220924165623733](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220924165623733.png)

![image-20220924165646810](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220924165646810.png)

### 实践操作

任务要求：打开你的网页浏览器并导航到overwrite.uploadvulns.thm ，你的目标是用你自己上传的文件 覆盖服务器上的文件。

![image-20220924185206709](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220924185206709.png)

![image-20220924185713701](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220924185713701.png)

上传成功，获得flag

![image-20220924185816261](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220924185816261.png)

## 远程代码执行

### 理论讲解

如果可以完全覆盖服务器上已经存在的文件，那么对于维护站点的人来说是一个麻烦，因为文件覆盖可能会导致一些其他漏洞，

现在让我们更进一步，尝试验证远程代码执行漏洞：

```
远程代码执行，顾名思义，就是允许我们在 Web 服务器上执行任意代码
虽然这可能是一个低权限的 Web 用户帐户(如 Linux 服务器上的 www-data账户) ,但是这仍然是一个非常严重的漏洞。

通过 Web 应用程序中的上传漏洞远程执行代码，往往利用的是上传与网站后端使用相同语言编写的程序（或者是服务器能够理解并能执行的另一种语言）
传统的后端语言是PHP，然而，其他后端语言也已经变得更加普遍（主要的例子有Python Django、Node.js 形式的 Javascript等）

值得注意的是，在路由应用程序中(路由是通过编程方式定义的应用程序，而不是映射到文件系统的应用程序)这种攻击方法变得更加复杂，发生的可能性也大大降低。
大多数现代 Web 框架都是通过编程方式实现路由的。

在利用文件上传漏洞时，有两种基本的方法可以在 Web 服务器上实现 RCE（远程代码执行）：Webshell和反向/绑定 shell
实际上，一个功能完备的反向/绑定 shell 是攻击者的理想目标；
然而，有时候webshell可能是唯一可用的选择（例如在以下环境中：目标服务器对上传功能施加了文件长度限制，或者防火墙规则阻止任何基于网络的 shell）

作为一种通用的方法论，我们希望上传一种或另一种 shell，然后激活它，如果服务器允许，我们就能直接导航到该文件（限制不足的非路由应用程序）；
或者以其他方式强制 webapp 为我们运行脚本（这在路由应用程序中是必需的）。
```

**webshell实现RCE**

假设我们找到了一个带有上传表单的网页:

![image-20220924192650805](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220924192650805.png)

我们接下来怎么办? 我们从扫描开始:

![image-20220924192730491](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220924192730491.png)

我们看到了两个比较值得注意的目录：uploads和assets

我们上传的任何文件似乎都将放置在“uploads”目录中，我们先上传一个合法的图像文件：

![image-20220924193316086](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220924193316086.png)

![image-20220924194029486](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220924194029486.png)

现在，如果我们去 http://demo.uploadvulns.thm/uploads 页面，我们应该能够看到照片已经上传了！

![image-20220924194854826](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220924194854826.png)

![image-20220924194936039](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220924194936039.png)

我们已经验证了可以成功上传我们自己的图片，现在让我们尝试一个 webshell。

实际上，我们知道这个 webserver 是用PHP开发后端页面的，所以我们将直接跳到创建和上传PHP的 webshell。

在实战中，我们可能需要多做一些枚举，PHP是一个很好的起点。

一个简单的 webshell 通过获取一个参数并将其作为系统命令执行来工作，在 PHP 中，这样做的语法是：

```php
<?php
    echo system($_GET["cmd"]);
?>
```

此代码将接受一个 GET 参数，并将其作为系统命令执行，然后将输出回显到屏幕上。

让我们尝试把它上传到网站，然后用它来显示我们当前的用户和工作目录的内容:

![image-20220924200128060](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220924200128060.png)

我们现在可以使用这个 shell 从系统中读取文件，或者从这里升级到反向 shell。现在我们有了 RCE，选择是无限的。

注意：在使用 webshell 时，查看页面的源代码通常更容易查看输出，这大大改进了输出的格式。

**反向shell实现RCE**

上传反向 shell 的过程与上传 webshell 的过程几乎完全相同，我们将使用Pentest Monkey文档建立反向 shell，这在Kali Linux 上是默认安装的，

该文档的链接是：https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php

你需要编辑这个 shell 的第49行：$ip = '127.0.0.1'; // CHANGE THIS

将127.0.0.1更改为 TryHackMe提供的 tun0 IP 地址（在https://tryhackme.com/access 页面可以找到，这代表的是攻击机的内部虚拟 IP 地址）

修改完shell文件后，接下来我们需要做的是启动一个 Netcat 监听器来接收连接，输入命令： nc -lvnp 1234

![image-20220924202051550](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220924202051550.png)

现在，让我们上传 shell，然后通过浏览器url导航到http://demo.uploadvulns.thm/uploads/shell.php

这个 shell 的名字，就是你自己命令的shell文件的名字（默认情况下是php-reverse-shell.php）

此时网站会被挂起，无法正常加载内容，当我们切换回我们的终端界面时，我们可以看到我们获取了一个反向shell：

![image-20220924202734098](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220924202734098.png)

### 实践操作

任务要求：打开你的网页浏览器并导航到shell.uploadvulns.thm，实现一个简单的RCE。

```
先进行目录扫描，使用gobuster，输入以下命令：
gobuster dir -u http://shell.uploadvulns.thm -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

![image-20220924204803595](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220924204803595.png)

发现/resources目录是文件上传点

准备好一个反向shell文件（https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php ）

将shell文件中的ip地址为攻击机的虚拟内部地址（https://tryhackme.com/access ）：

![image-20220924212035175](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220924212035175.png)

在攻击机终端设置监听，利用目标的文件上传点——上传shell文件，上传完毕之后，先访问shell文件，再查看终端界面：

![image-20220924212830528](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220924212830528.png)

![image-20220924212952319](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220924212952319.png)

使用shell界面，找到flag文件：

![image-20220924213305430](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220924213305430.png)

## 关于“过滤”的介绍

首先，让我们讨论一下客户端过滤和服务器端过滤之间的区别。

**客户端过滤：**

当我们谈论一个“客户端”的脚本时,在 Web 应用程序的环境中，我们的意思是它运行在用户的浏览器上，而不是运行在Web 服务器上。主流的客户端脚本语言是JavaScript 。

无论使用何种语言，客户端脚本都会在浏览器中运行，在文件上传的上下文中，这意味着过滤发生在文件上传到服务器之前。

理论上讲，这应该是件好事，然而，因为过滤是在我们的计算机上发生的，所以我们很容易绕过它。因此，像这样通过客户端过滤来验证上载的文件是否为恶意文件，本身就是一种高度不安全的方法。

**服务器端过滤：**

服务器端脚本将在服务器上运行。在传统上，PHP 是主要的服务器端语言(微软IIS的ASP则紧随其后），最近几年，其他几种服务端脚本语言（C # ，Node.js，Python，Ruby on Rails等等）也得到了广泛地应用。服务器端过滤往往更难绕过，因为你面前没有代码。

当代码在服务器上执行时，在大多数情况下，也不可能完全绕过过滤；所以我们必须在符合过滤的地方 形成一个有效载荷（payload），从而让服务器端仍然允许我们执行代码。

谈完了客户端过滤和服务器端过滤，让我们看看具体的一些不同类型的过滤：

**文件扩展名验证：**

```
文件扩展名(理论上)用于标识文件的内容。
在实践中，它们很容易改变，所以实际上并没有多大意义。
然而，MS Windows 仍然使用它们来识别文件类型，尽管基于 Unix 的系统倾向于依赖其他方法，
检查文件扩展名的筛选器有两种工作方式。它们要么是黑名单扩展(即有一个不允许的扩展名列表) ，要么是白名单扩展(即有一个允许的扩展名列表，并拒绝所有其他扩展名)。
```

**文件类型过滤：**

文件类型过滤 类似于文件扩展名验证，但是更加密集，文件类型过滤会再次验证文件的内容是否可以上传。 我们将研究两种类型的文件类型验证:

```
MIME 验证:
MIME (多用途 Internet 邮件扩展)类型用作文件的标识符——最初是在通过电子邮件作为附件传输时使用，现在也在通过 HTTP (S)传输文件时使用。
文件上传的 MIME 类型附加在请求的头部，看起来像这样:
```

![image-20220924231155849](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220924231155849.png)

```
MIME类型遵循的格式为： < type >/< subtype > 。
在上面的请求中，你可以看到图片“ Spaniel.jpg”已经上传到服务器。
作为一个合法的 JPEG 图像，这次上传的 MIME 类型是“ image/JPEG”。
文件的 MIME 类型可以在客户端和/或服务器端进行检查，但是，由于 MIME 是基于文件的扩展名的，因此也很容易绕过它。
```

```
魔术数字验证:
魔术数字是确定文件内容的更准确的方法，尽管它们并非不可伪造。
文件的“魔术数字”是指标识文件内容 在文件内容开头位置的一串字节。
例如，一个 PNG 文件在文件的最顶部有这些字节:89 50 4E 47 0D 0A 1A 0A
```

![image-20220924231259948](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220924231259948.png)

```
与Windows系统不同的是，Unix系统使用魔法数字来识别文件。
Unix类型的系统 在处理文件上传时，会检查上传文件的魔法数字，以确保文件能够被安全接受。
这绝不是一个有保证的解决方案，但它比检查文件的扩展名更有效。
```

**文件长度过滤:**

```
文件长度过滤 用于防止通过上传表单将大型文件上传到服务器(因为这可能导致服务器资源短缺)。
在大多数情况下，当我们上传 shell 时，这不会给我们带来任何问题。
但是，值得注意的是，如果上传表单只希望上传一个非常小的文件，可能会有一个长度过滤，以确保遵守文件长度要求。
作为一个例子，我们前一个任务中的PHP反向shell文件 大小为5.4 Kb ——相对较小，但是如果表单期望的文件最大为2Kb，那么我们需要找到一个替代的 shell 来上传。
```

**文件名过滤:**

```
如前所述，上传到服务器的文件应该是唯一的。
通常这意味着向文件名添加一些随机字符，但是，另一种策略是检查服务器上是否已经存在同名文件，如果存在，则返回给用户一个错误提示。
此外，文件名应该在上传时进行消毒，以确保它们不包含任何“坏字符”，“坏字符”可能会在上传时对文件系统造成潜在的问题(例如，Linux 上的空字节或正斜杠，以及控制字符";" 和潜在的Unicode 字符......)。
这对我们来说意味着，在一个管理良好的系统上，我们上传后的文件不太可能与上传前的文件名称相同，所以要注意，如果你设法绕过了过滤，你可能还要找到上传后的shell。
```

**文件内容过滤:**

```
更复杂的过滤系统可能会扫描上传文件的全部内容，以确保文件不会在扩展名、 MIME 类型和 Magic Number上面进行欺骗。
这是一个明显比大多数基本过滤系统更复杂的过滤过程。
```

值得注意的是，这些过滤本身都不是完美的——它们通常会相互结合使用，提供一个多层过滤，从而大大提高了上传的安全性。这些过滤器的任何一个都可以单独应用于客户端、服务器端或两者兼而有之（同时应用在前端和后端）。

类似地，不同的框架和语言都有自己固有的过滤和验证上传文件的方法。因此，可能会出现特定于语言的漏洞。

例如，在 PHP 主要版本php5之前，通过附加一个null空字节能够绕过文件扩展名过滤，后面再跟有效的扩展名.php，从而形成一个恶意的php webshell文件。

还可以将 PHP 代码注入到本来有效的图像文件的 exif 数据中，然后强制服务器执行它。

## 绕过客户端过滤

### 理论讲解

正如前面所提到的，客户端过滤非常容易绕过，因为它完全发生在你所控制的机器上，当你能够访问代码时，就很容易去修改它。

有四种简单的方法可以绕过一般的客户端文件上传过滤:

```
1.关闭浏览器中的 Javascript以绕过客户端过滤 ——如果网站不需要 Javascript 来提供基本的功能，那么这种绕过方法就是有效的。
如果完全关闭 Javascript 将阻止站点工作，那么你可能需要选择其他方法来进行绕过客户端过滤，

2.拦截并修改将要加载的web页面。使用 Burpsuit，我们可以拦截传入的Web页面，并在 Javascript 过滤器有机会运行之前就删除这个过滤器。

3.拦截并修改文件然后再上传。前一个方法在加载网页之前工作，这个方法则允许网页正常加载，在文件已经通过，并被过滤器接受之后，再拦截文件上传操作（upload）。

4.直接将文件发送到上传点，当你可以直接使用像 curl 这样的工具发送文件时，为什么要使用带有过滤器的网页呢？这种将数据直接发布到包含处理文件上传代码的页面是完全绕过客户端过滤器的另一种有效方法。
在这里，我们不会深入讨论这种方法，但是，这种命令的语法应该是这样的:
curl -X POST -F "submit:<value>" -F "<file-parameter>:@<path-to-file>" <site>
要使用这种方法，首先要拦截一次成功的上传(使用 BurpSuite 或浏览器控制台) ，查看上传中使用的参数，然后将这些参数插入上面的命令。
```

我们将在下面深入讨论第二和第三种方法。

让我们再次假设，我们在一个网站上找到了一个上传页面:

![image-20220925164435363](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925164435363.png)

像往常一样，我们先看一下源代码。这里我们看到一个基本的 Javascript 函数 它用于检查上传文件的 MIME 类型:

![image-20220925164501313](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925164501313.png)

在这个例子中，我们可以看到过滤器使用了一个白名单来排除任何不是 image/jpeg 的 MIME 类型。

我们的下一步操作是尝试进行文件上传——正如预期的那样，如果我们选择一个 jpeg图片，过滤器函数就会接受它，否则上传行为将会被拒绝。

了解了这些信息之后，让我们启动 Burpsuite 的拦截功能 并重新加载原网页。在Burpsuite中我们将看到我们对站点的请求消息，但是我们真正想看到的是服务器的响应消息，所以右键单击截获的数据，向下滚动到“ Do Intercept”，然后选择“ Response to this request”:

![image-20220925164617843](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925164617843.png)

当我们点击窗口顶部的“Forward”按钮时，我们将看到服务器对我们的请求的响应。在这里，我们可以删除、注释掉 Javascript 过滤器函数，或者在它有机会加载之前中断它:

![image-20220925164651853](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925164651853.png)

删除过滤函数后，我们再次点击“Forward”，直到浏览器中的网站页面完成加载。现在我们可以自由上传任何类型的文件到网站:

![image-20220925164718959](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925164718959.png)

这里值得注意的是，在默认情况下，Burpsuite 不会拦截 Web 页面正在加载的任何外部 Javascript 文件。如果你需要编辑一个不在正在加载的主页面中的脚本（也就是外部js脚本），你需要点击 Burpsuite窗口顶部的“ Options”选项卡，然后在“ Intercept Client Request”（拦截客户端请求）部分，编辑第一行的“文件扩展名-不匹配”的条件（condition），删除^js$| :

![image-20220925164819834](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925164819834.png)

我们已经通过在加载页面之前拦截和删除过滤器函数绕过了这个过滤，现在让我们尝试先上传一个具有合法扩展名和 MIME 类型的文件，然后再用 Burpsuite 进行拦截和更正上传。

重新加载网页（这时过滤器函数将重新生效），让我们使用之前使用的反向 shell 并将其重命名为“ shell.jpg”。当 MIME 类型(基于文件扩展名)自动检查完成时，客户端的过滤器会让我们的payload（有效载荷）通过:

![image-20220925164850109](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925164850109.png)

激活 Burpsuite 拦截功能，然后在网页中单击“uplpad（上传）”，Burpsuite 就会自动捕捉该上传请求:

![image-20220925164914830](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925164914830.png)

请注意，我们的 PHP shell 的 MIME 类型当前是 image/jpeg。我们将其更改为 text/x-php，并将文件扩展名从.jpg改成.php，然后再将这个请求转发到服务器:

![image-20220925165002906](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925165002906.png)

我们先在攻击机终端设置好netcat 监听器，然后我们再导航到 http://demo.uploadvulns.thm/uploads/shell.php 页面，由下图能够看到攻击机终端已经成功接收到一个连接，反向shell已建立！

![image-20220925165030643](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925165030643.png)

### 实践操作

任务要求：使用浏览器导航到java.uploadvulns.thm页面，并绕过过滤器获得一个反向 shell。请记住，并非所有客户端脚本都是内联的（意思就是有时候会存在外部脚本）！

使用gobuster对目标站点进行目录爆破：

```
gobuster dir -u http://java.uploadvulns.thm -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

![image-20220925222043825](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925222043825.png)

修改要上传的shell文件的内容，将IP地址和端口设置一下（IP地址设置为TryHackMe配置给攻击机的虚拟内部地址，端口自定义即可），在攻击机上打开一个终端窗口，输入命令设置一个Netcat监听器，监听1234端口：

```
nc -nlvp 1234
```

![image-20220925212329121](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925212329121.png)

在浏览器中打开给定的目标网址，查看网页源码，可以发现过滤函数为一个外部js脚本，点击该脚本地址 跳转查看过滤函数内容，可以看到是设置了一个白名单机制 只允许图片格式的文件进行上传：

![image-20220925214225752](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925214225752.png)

![image-20220925214342018](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925214342018.png)

使用Burpsuite 拦截网站的数据包，由于Burp默认不拦截外部的js脚本，所以先修改Burp的Proxy功能中的options，取消对js文件的不匹配。

![image-20220925214951749](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925214951749.png)

回到原网站刷新浏览器页面，此时Burp会拦截浏览器加载页面，在Burp中查看服务器的响应消息，将实现过滤功能的脚本注释掉或者删除掉，再重新转发给服务器：

![image-20220925215525151](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925215525151.png)

![image-20220925220838729](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925220838729.png)

此时回到原网站 可以进行任意文件上传，将shell文件上传至目标站点，访问网站目录中的shell文件，成功建立反向shell：

![image-20220925220144576](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925220144576.png)

![image-20220925222248997](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925222248997.png)

利用该反向shell，在攻击机终端找到flag文件：

![image-20220925222502895](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925222502895.png)

## 绕过服务器端过滤: 文件扩展名

### 理论讲解

当过滤设置在服务器端时，比客户端过滤更难绕过，因为你完全看不到代码。

为了绕过服务器端的过滤，我们需要进行很多尝试 弄清楚它的过滤机制，然后才能逐渐组装出一个符合限制条件的payload（有效载荷）。

假设有一个使用 文件扩展名黑名单 作为服务器端过滤器的网站，它的后端代码如下所示：

```php
<?php
    //Get the extension
    $extension = pathinfo($_FILES["fileToUpload"]["name"])["extension"];
    //Check the extension against the blacklist -- .php and .phtml
    switch($extension){
        case "php":
        case "phtml":
        case NULL:
            $uploadFail = True;
            break;
        default:
            $uploadFail = False;
    }
?>
```

在此实例中，代码正在查找最后一个句点(.)并使用该文件名确认扩展名，这就是我们要绕过的。

这些代码的其他工作方式包括: 搜索文件名中的第一个句点(.)，或者在每个句点(.)后面分割文件名，检查是否有被列入黑名单的扩展名出现。

我们可以看到代码正在过滤掉.php 和.phtml 扩展名,因此，如果我们想要上传一个 PHP 脚本，我们将不得不寻找另一个扩展名。

PHP 的维基百科页面（https://en.wikipedia.org/wiki/PHP）提供了一些常见的扩展，我们可以尝试使用; 然而，实际上还有很多其他很少使用的扩展，网络服务器仍然可以识别出来，其中包括：

```
.php3, .php4, .php5, .php7, .phps, .php-s, .pht and .phar
```

其中许多都能绕过过滤器(.php 和.phtml被黑名单过滤拦截) ，但有时候服务器会被配置为 不识别一些扩展名为 PHP 文件，如下例所示:

![image-20220925225244128](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925225244128.png)

这实际上是 Apache2服务器的默认配置（不识别一些扩展名为 PHP 文件），但是，系统管理员可能已经更改了默认配置（或服务器可能已经过时），所以使用其他可能会被解析成php的文件扩展名 还是值得一试。

最终在本例中，我们发现.phar扩展名能够绕过过滤器，并让我们获得一个shell：

![image-20220925225925059](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925225925059.png)

让我们看看另一个例子，这次我们将完全黑箱操作:没有后端代码可供参考。

在这个例子中，我们有一个上传表单：

![image-20220925230255823](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925230255823.png)

我们从完全合法的上传开始，让我们尝试上传spaniel.jpg：

![image-20220925230346469](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925230346469.png)

这说明 JPEGS 是可以被接受的，我们选择一个肯定会被拒绝的吧：shell.php

![image-20220925230539046](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925230539046.png)

通过多次尝试上传不同文件格式 的文件，我们能够大概了解 过滤器会接受什么格式的文件或者会拒绝什么格式的文件，我们发现没有既执行又不被过滤的php shell 文件扩展名。

在前面的示例中，我们看到后端代码使用 pathinfo ()函数 来得到" . "后面的最后几个字符,但是如果它对 略有不同的输入内容 进行过滤时会发生什么呢？

让我们尝试上传一个名为shell.jpg.php的文件，我们已经知道 JPEG 文件能够被接受,如果过滤器只是检查.jpg文件是否在输入内容中的某个地方 那会发生什么呢？

这种过滤器的伪代码可能是这样的:

```
ACCEPT FILE FROM THE USER -- SAVE FILENAME IN VARIABLE userInput
IF STRING ".jpg" IS IN VARIABLE userInput:
    SAVE THE FILE
ELSE:
    RETURN ERROR MESSAGE

伪代码解读：
接受用户的文件--在变量userInput中保存文件名
如果字符串“ . jpg”在变量userInput中
则保存该文件
否则 返回错误信息
```

当上传shell.jpg.php文件之后，我们得到一个成功的消息。使用浏览器导航到/uploads目录下，确认payload（有效载荷）已成功上传:

![image-20220925232259443](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925232259443.png)

点击这个shell文件，成功建立反向shell：

![image-20220925232416755](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925232416755.png)

这绝不是一个与文件扩展名相关的上传漏洞的详尽列表。与黑客行为中的所有内容一样，我们也在寻找其他人编写的代码中的缺陷; 这些代码很可能是为手头的任务编写的唯一代码。这是从这个任务中学到的真正重要的一点: 在编程时，有成千上万种不同的方法来实现相同的特性——您的开发必须针对手边的过滤器进行定制。绕过任何类型的服务器端过滤器的关键是枚举并查看哪些是允许的，以及哪些是阻塞的; 然后尝试构建一个有效载荷，以便能够通过过滤器所寻找的条件。

### 实践操作

任务要求：绕过过滤器上传并激活一个 shell，在/var/www/路径下找到flag文件，目标站点为：annex.uploadvulns.thm （tips：在找上传后的shell文件时，网站目录并不总是能够提供索引）

进行目录爆破：

```
 gobuster dir -u http://annex.uploadvulns.thm -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
```

![image-20220925234842112](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925234842112.png)

尝试修改shell文件扩展名进行绕过：

```
上传.jpg.php失败
上传.phar失败
上传.pht失败
上传.php-s失败
上传.phps失败
上传.php7失败
上传.php5成功
```

![image-20220925235117393](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925235117393.png)

使用建立的shell，找到flag文件并查看内容：

![image-20220925235249962](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925235249962.png)

## 绕过服务器端过滤: 魔术数字

### 理论讲解

我们已经了解了服务器端的扩展名过滤，接下来了解一下 魔术数字验证作为服务器端过滤器的情况。

如前所述，魔术数字是用作文件的更准确的标识符。文件的魔术数字是一个十六进制字符串，并且总是文件中的第一个元素。了解了这一点，就可以使用魔术数字来验证文件上传，只需读取前几个字节并将它们与白名单或黑名单进行比较即可。请记住，这种技术对于基于 PHP 的 Web 服务器非常有效; 但是，对于其他类型的 Web 服务器，它有时会失败。

让我们来看一个例子，我们现在有一个上传页面:

![image-20220926000803240](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220926000803240.png)

如果我们上传标准的 shell.php 文件，就会得到一个错误，但是上传JPEG文件是可以的，所以让我们尝试添加 JPEG 的魔术数字到我们的shell.php文件的顶部，

查看网站：https://en.wikipedia.org/wiki/List\_of\_file\_signatures ，这个网站向我们展示了 JPEG 文件的几个魔术数字，选择其中一个即可，如：FF D8 FF DB

我们可以直接把这些数字的 ASCII 表示（ÿØÿÛ）添加到文件的顶部，但是直接使用十六进制表示通常更容易。

在开始之前，让我们使用 Linux file 命令来检查 shell 的文件类型:

![image-20220926004117605](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220926004117605.png)

我们选择的魔术数字是四个字节长度，所以让我们打开反向 shell 脚本，在第一行添加四个随机字符。这些字符并不重要，所以在这个例子中我们用了四个“ A”:

![image-20220926003240345](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220926003240345.png)

保存并退出，使用hexeditor（kali默认已安装）或者其他类似工具打开这个文件，这个工具可以让你以十六进制的形式打开并且编辑shell文件内容：

![image-20220926003613598](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220926003613598.png)

注意上图红色框中的四个字节: 它们都是41，这是大写的“ A”的十六进制代码--正是我们之前在文件顶部添加的内容，将其更改为我们之前查找到的 JPEG 文件的魔术数字:FF D8 FF DB

![image-20220926003936182](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220926003936182.png)

现在，如果我们保存并退出文件(Ctrl + x)，我们可以再次使用 file命令，并且可以看到我们已经成功地欺骗了 shell 的 filetype：

![image-20220926004049542](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220926004049542.png)

很好，现在让我们尝试上传修改后的 shell，看看它是否绕过过滤器:

![image-20220926004208195](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220926004208195.png)

我们绕过了服务器端的魔法数字过滤器，得到了一个反向 shell。

### 实践操作

任务要求：绕过魔术数字过滤器上传一个 shell，找到上传的 shell 的位置并激活它，最后在/var/www/路径下找到并查看flag文件，目标站点为magic.uploadvulns.thm

先对目标站点进行目录爆破：

```
 gobuster dir -u http://magic.uploadvulns.thm -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
```

![image-20220926010048800](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220926010048800.png)

上传一个JPEG文件到目标站点，目标站点给的提示信息为 只接收gif文件：

![image-20220926005811324](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220926005811324.png)

修改shell文件（先在php文件的顶部添加AAAAAA六个字符，然后使用 hexeditor 修改AAAAAA对应的十六进制字符为gif文件的魔术数字）：

![image-20220926005329194](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220926005329194.png)

![image-20220926010343653](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220926010343653.png)

在攻击机终端设置Netcat监听器，上传文件到目标站点，访问之前爆破得到的目录路径，发现服务器上的索引关闭，因此我们需要输入以下内容激活shell：

```
http://magic.uploadvulns.thm/graphics/shell.php
```

![image-20220926020458363](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220926020458363.png)

![image-20220926020859860](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220926020859860.png)

成功建立反向shell，使用命令查找flag文件并查看其中的内容：

![image-20220926020634615](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220926020634615.png)

## 示例方法

我们现在已经看到了各种不同类型的过滤器——客户端和服务器端——以及用于文件上传攻击的一般方法。在下一个任务中，你将要完成一个黑盒文件上传挑战，因此让我们借此机会讨论一个示例方法，以便更深入地处理此类挑战。你可以开发自己的方法来替代这种方法，但是，如果你是文件上传攻击的新手，你可能会发现以下示例方法非常有用。

我们把这看作是一个循序渐进的过程，假设我们有一个网站要进行安全审计。

​ 1.我们要做的第一件事就是把网站作为一个整体来看。使用浏览器扩展插件，比如前面提到的 Wappalyzer(或者手动) ，我们可以寻找到 Web 应用程序可能使用了什么语言和框架。 Wappalyzer并不总是100% 准确，通过向网站发出请求来手动枚举这些信息是一个很好的开始，同时可以使用 Burpsuite来截取响应消息。消息头中的server或者x-powered-by可用于获取有关服务器的信息，我们还要寻找攻击的载体，比如，一个上传页面。

​ 2.找到一个上传页面之后，我们的目标是进一步检查它。查看客户端脚本的源代码，以确定是否有任何要绕过的客户端过滤器，客户端完全在我们的控制之中。

​ 3.然后我们会尝试上传一个完全无害的文件。从这里我们将看到如何才能访问我们的文件，换句话说，我们可以直接在上传文件夹中访问它吗？上传的文件是不是嵌在某个页面里了？网站对所上传的文件的命名方案是什么？如果上传之后，文件所在的位置不是很明显，就需要使用Gobuster 这样的目录扫描工具。这一步非常重要，因为它不仅提高了我们对正在攻击的目标的认识，还给了我们一个“可接受”文件的基准，我们可以在此基础上做进一步测试。

​ gobuster在这里的一个常用参数选项是-x 选项，该选项可用来查找具有特定扩展名的文件，例如，如果将 -x php，txt，html 添加到 gobuster 命令中，则该工具将附加.php，.txt和.html到选定的字典内容中的每一项。如果你已经设法上传了一个有效负载，并且服务器会更改上传文件的名称，那么这将非常有用。

​ 4.确定了如何以及在哪里可以访问我们上传的文件之后，我们将尝试进行恶意文件上传，绕过我们在第二步中发现的任何客户端过滤器。如果此时存在服务器端过滤器使文件上传停止，那么它给我们的报错消息对于确定接下来的步骤非常有用。

假设我们的恶意文件上传操作已经被服务器端过滤所停止，以下有一些方法可以确定服务器端使用的是哪种类型的过滤:

​ 1.如果你可以成功地上传一个文件扩展名完全无效的文件（`testingimage.invalidfileextension`)，服务器很可能使用了"扩展名黑名单"来过滤文件，反之，如果该上传失败，那么服务器可能使用的是"扩展名白名单"的方式过滤文件。

​ 2.尝试重新上传最初被接受的完全无害文件，但这一次将该文件的魔术数字修改为shell文件类型（如php类型）的魔术数字。如果上传失败，那么说明服务器正在使用一个基于魔术数字验证的过滤器。

​ 3.与前一点一样，再次尝试上传无害的文件，但是使用 BurpSuite 拦截请求，并将上传的 MIME 类型更改为你希望通过过滤的MIME类型。如果上传失败，那么说明服务器端是基于 MIME 类型验证进行过滤的。

​ 4.枚举服务器端是否存在文件长度过滤，就是上传一个小文件，然后逐渐上传更大的文件，直到你触发了过滤器。到那时你就会知道可接受的极限文件大小是多少。如果你非常幸运，那么第一次上传失败的错误信息可能会直接告诉你文件大小限制是多少。请注意，一个小的文件长度限制可能会阻止你上传shell文件----比如：到目前为止我们一直在使用的反向 shell文件。

## 挑战任务

要求：目标站点为 jewel.uploadvulns.thm ，上传文件并建立反向shell，利用反向shell界面 找到/var/www/路径下的flag文件并查看其内容。

tips：

这里需要一个字典 UploadVulnsWordlist.txt，由TryHackme网页提供下载。

还需要使用github上的nodejs反向shell，地址如下（访问该地址复制前面的function部分）： https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#nodejs

我们需要的shell文件内容如下所示（已经将ip和端口修改好了）：

![image-20220926230004608](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220926230004608.png)

首先对目标站点进行目录爆破：

```
gobuster dir -u http://jewel.uploadvulns.thm -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

![image-20220926211905094](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220926211905094.png)

> 扫描发现四个目录： /content /modules /admin /assets

在浏览器中访问目标站点查看其网站源代码：

![image-20220926210849782](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220926210849782.png)

![image-20220926211005676](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220926211005676.png)

> 由过滤脚本可知：客户端存在三种过滤，验证文件大小、验证魔术数字、验证文件扩展名

使用Wappalyzer浏览器插件检测到该网站属于nodejs站点，所以需要一个nodejs反向shell文件（前文已经给出下载链接）

![image-20220926213700914](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220926213700914.png)

对该shell文件进行修改以便能够绕过一些过滤：

```
打开shell文件，在其内容的顶部添加AAA（占位用），
使用hexeditor工具，将AAA对应的十六进制数值改为jpg文件对应的魔术数字的前三位十六进制值FF D8 FF（对应的ASCII码为ÿØÿ）--因为js过滤器中的代码只检测了前三位
修改文件名为shell.jpg
查看文件的大小是否超过文件大小过滤器的限定值
```

![image-20220926230711555](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220926230711555.png)

进行文件上传操作，打开Burpsuite拦截该上传,因为shell的内容中添加了多余的魔术数字，所以为了使shell文件能被激活，最终还需要删除这些字符，php的shell文件不需要这一步操作----因为魔术数字能被添加在php的开始标志"\<?php"之外。

![image-20220926220220577](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220926220220577.png)

> 通过Burpsuite拦截到的内容，我们可以看到目标站点对文件内容进行了base64编码。
>
> 对nodejs的反向shell文件内容进行base64编码，替换掉Burpsuite中的编码内容，然后再点击"Forward"，将该请求消息转发给服务器端。
>
> 其实删除掉/9j/就行，这几个字符是jpg魔术数字前三位对应的ASCII码（前三位ÿØÿ）的base64编码。

尝试访问之前爆破得到的目录，发现只有一个目录可以访问，即：/admin ，根据页面提示 我们需要找到要访问的具体文件路径，然后输入到下图中的输入框中：

![image-20220926221954657](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220926221954657.png)

我们要找的是jpg格式的文件路径，所以我们先看一下网站原有图片的路径：

![image-20220926223445601](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220926223445601.png)

> 从上图可以看到网站原有的图片在/content目录下

现在使用TryHackme提供的字典进行目录爆破 指定目录为/content 并指定文件格式为.jpg，尝试找到我们已经上传的shell文件，因为之前检测过shell文件的大小，所以我们找size在400左右的jpg文件进行访问：

```
gobuster dir -u http://jewel.uploadvulns.thm/content -w ./UploadVulnsWordlist.txt -x .jpg
```

![image-20220927000530281](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220927000530281.png)

在攻击机终端使用Netcat建立监听器，在目标站点下/admin页面中的输入框中输入../content/xxx.jpg ，建立反向shell，使用该shell界面查找flag文件并查看其内容：

![image-20220927001020219](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220927001020219.png)

![image-20220927001046177](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220927001046177.png)

![image-20220927003006447](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220927003006447.png)

## 恢复hosts文件配置

```
在完成练习之后，需要恢复在第一节中对hosts文件所做的修改。

在Linux或MacOS上，使用以下命令:
sudo sed -i '$d' /etc/hosts        #该行命令的意思是：删除文本的最后一列 也就是说把之前在第一节中添加的那一行记录删除。

在Windows上使用以下命令:
(GC C:\Windows\System32\drivers\etc\hosts | select -Skiplast 1) | SC C:\Windows\System32\drivers\etc\hosts
```

![image-20220927003603556](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220927003603556.png)

![image-20220927003620151](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220927003620151.png)

## 关卡完整答案

![image-20220924190030075](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220924190030075.png)

![image-20220924213350439](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220924213350439.png)

![image-20220925222528104](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925222528104.png)

![image-20220925235324298](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220925235324298.png)

![image-20220927003325771](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220927003325771.png)
