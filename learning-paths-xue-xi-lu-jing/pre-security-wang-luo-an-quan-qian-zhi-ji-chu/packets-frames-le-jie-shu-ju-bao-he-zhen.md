---
description: 本文相关内容：了解数据如何被分成更小的部分并通过网络传输到另一台设备。
---

# Packets & Frames(了解数据包和帧)

THM实验房间链接：https://tryhackme.com/room/packetsframes



## 什么是数据包和帧？

数据包和帧是小块的数据，当它们组合在一起时，就构成了更大的信息或消息；然而，数据包和帧在OSI模型中是两种不同的东西，帧位于OSI的第2层——数据链路层，这意味着帧不包含IP地址这样的信息。类比案例：你可以想象把一个信封放在另外一个信封里，然后邮寄完整的信封，此完整信封对应的是发送的数据包，一旦打开完整信封，里面仍然会存在并包含一些数据(也就是一个帧)。

数据包和帧的组合过程被称为封装，在这个阶段中，我们可以安全地假设：任何包含IP地址的信息指的都是数据包，而当封装好的信息被剥离时，我们谈论的信息则是帧本身。

数据包是跨网络设备传输数据的有效方式，由于这些数据是分小块传输的，所以此时在整个网络中发生数据传输阻塞的几率 比一次性发送大消息进而造成传输阻塞的几率要小。

当你从网站加载图像时，该图像不会作为一个整体发送到你的计算机，而是在你的计算机上由小块图像开始重建进而得到一个完整图像；下面的图片说明了以上过程，来自于网站的图像将被分成三个数据包，当数据包到达计算机时，图像就会开始重建，最终形成完整的图像。

![image-20230326193818536](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230326193818536.png)

数据包有不同的结构，这些结构取决于正在发送的数据包的类型，网络上充满了各种标准和协议，这些标准和协议充当了设备上如何处理数据包的一组规则。

使用了协议的数据包将有一组标头，其中包含跨网络发送的数据的附加信息。

一些值得注意的标头信息包括：

* Time to Live（生存时间）：该字段为数据包设置了一个过期计时器，即使数据包永远无法到达目标主机，这也不会阻塞你的网络。
* Checksum（校验和）：该字段能够为TCP/IP等协议提供完整性检查，如果数据被更改了，那么此时的Checksum值将变得与预期的Checksum值不同。
* Source Address（源地址）：该字段表示发送数据包的设备的IP地址，这样数据就知道该返回到哪里。
* Destination Address（目标地址）：数据包要发送到的设备的IP地址，以便数据知道下一步要传输到哪里。

### **答题**

![image-20230326201110685](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230326201110685.png)

## TCP/IP（三次握手）介绍

传输控制协议(TCP-Transmission Control Protocol)是在网络中使用的一个规则。

基于TCP/IP协议集的TCP/IP模型与OSI模型非常相似，TCP/IP模型由四层分级组成，可以说是OSI模型的一个简化版本，TCP/IP模型所包含的分层有:

* Application（应用层）
* Transport（传输层）
* Internet（网际层）
* Network Interface（网络接口层）

与OSI模型的工作原理非常相似，数据包在经过TCP/IP模型的每一层时都会被添加部分信息——这个过程被称为封装，而与此过程相反的是解封装。

**关于TCP协议**

TCP协议的一个定义特征是它是基于连接的，这意味着在发送数据之前，TCP必须在客户端和作为服务器的设备之间建立起一个持续的连接；因此，TCP能够保证发送的任何数据都会在另一端被正确地接收到，这个数据发送和接收过程被称为TCP/IP三次握手。

下面是关于TCP优点和缺点的比较:

* TCP的优点：能够保证数据的准确性；能够同步两个设备，以防止彼此被数据淹没；将执行更多流程，以确保数据传输的可靠性。
* TCP的缺点：要求两台设备之间有可靠的连接，如果没有接收到一小块数据，那么整个数据块就不能使用（必须重新发送）；TCP的慢速连接可能会导致其他设备达到数据传输的瓶颈，因为在数据传输完毕之前——相关的TCP连接将一直保留在接收端计算机上；TCP 比 UDP 慢得多，因为使用此协议的设备必须完成更多的工作（计算）。

TCP数据包还将包含 在封装过程中所添加的被称为标头的信息部分，接下来让我们了解一些重要的TCP标头信息：

* Source Port（源端口）：发送方为发送TCP数据包而开放的端口，该值是随机选择的(从0-65535之间 选择尚未使用的端口)。
* Destination Port（目标端口）：远程主机（接收数据的主机）上运行的应用程序或服务的端口号，例如，默认运行 Web 服务器的端口 80；与源端口不同，目标端口号并不是随机选择的。
* Source IP（源IP地址）：这是发送数据包的设备的IP地址。
* Destination IP（目标IP地址）：这是数据包要到达的设备的IP地址。
* Sequence Number（序列号）：当TCP连接建立时，所传输的第一段数据将被赋予一个随机数。
* Acknowledgement Number（确认号）：在一段数据被赋予序列号之后，下一段数据的序列号值将会是序列号+ 1。
* Checksum（校验和）：这个值可用于对TCP数据包的完整性进行检查，如果校验和的值在数据传输过程中发生了变化，则说明数据已经被损坏。
* Data（数据）：这个标头是存储数据的地方，所存储的是正在传输的文件的字节。
* Flag（标志）：这个TCP标头决定了在三次握手过程中任一设备应该如何处理数据包，特定的标志将决定特定的行为。

**TCP/IP三次握手**

接下来，我们将讨论TCP/IP三次握手——这是用于在两个设备之间建立连接的过程的术语。

TCP/IP三次握手会使用一些特殊的信息进行交流，下面是一些主要的信息：

* 1-SYN：SYN消息是客户端在握手过程中发送的初始数据包。这个包用于发起连接并同步两个设备。
* 2-SYN/ACK：此数据包将由接收设备(服务器)发送，以确认来自客户端的同步尝试。
* 3-ACK：客户端或服务器都可以使用确认数据包来确认已经成功接收了一系列消息/数据包。
* 4-DATA：一旦两个设备建立了连接，数据(例如文件的字节)就会通过“DATA”消息发送。
* 5-FIN：此数据包可用于在TCP连接完成后 干净地(正确地)关闭TCP连接。
* RST：RST数据包能够立刻终止所有的TCP通信，如果使用了RST数据包则表明在TCP握手过程中存在一些问题——比如发生了服务或应用程序异常以及系统资源不足等故障。

下图显示了Alice和Bob之间正常的TCP三次握手过程，在现实生活中，这将发生在两个设备之间：

![image-20230326214033986](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230326214033986.png)

在TCP三次握手过程中，客户端所发送的数据将被赋予一个随机数序列，这个数字序列接下来将进行重构并递增1；两台计算机设备必须就相同的数字序列达成一致，才能以正确的顺序发送数据，这个顺序是在三个步骤中商定的（下面步骤中的序列号是随机值）：

* SYN-Client：客户端将设置用于同步的SYN标志位并生成一个随机初始序列号(ISN)——0 。
* SYN/ACK-Server：服务器端将设置SYN标志位和ACK标志位并生成一个随机初始序列号(5000)，并对客户端的初始序列号(0)进行确认（ACK=0+1）。
* ACK-Client： 客户端设置ACK标志位，并对服务器端的初始序列号(5000)进行确认（ACK=5000+1）。

tips：SYNchronise-SYN(同步)，ACKnowledge-ACK(确认)，Initial Sequence Number-ISN(初始序列号)；以上步骤的结果——客户端ISN+1，服务器端ISN+1。

![image-20230326221426557](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230326221426557.png)

**TCP连接的关闭**

接下来，让我们简单介绍一下TCP连接关闭背后的过程。

首先，一旦源设备确定目标设备成功接收了所有数据，那么TCP连接就将要被关闭；因为TCP会在设备上占用一些系统资源，所以最好在不需要的时候尽快关闭TCP连接。

为了启动TCP连接的关闭过程，设备将发送一个“FIN”数据包到另一个设备，当然，这仍基于TCP协议，其他设备也必须确认这个“FIN”数据包。

让我们像前文一样使用Alice和Bob来展示这个过程：

![image-20230326223520503](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230326223520503.png)

在上图中，我们可以看到Alice首先向Bob发送一个“FIN”包，当Bob收到这个消息之后，Bob会让Alice知道他已经收到消息(ACK)，并且Bob将表示他也想要关闭连接(FIN)，而Alice在收到来自于Bob的消息(ACK+FIN)之后，将进行最终的确认(ACK)——结果是TCP连接被成功关闭。

### **答题**

![image-20230326224720813](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230326224720813.png)

## 关于三次握手的模拟练习

在和本文相关的TryHackMe实验房间中部署实验环境并回答问题。

_tips：以正确的顺序重新组装TCP握手过程，以帮助 Alice 和 Bob 进行通信，在 Alice 与 Bob 的对话结束后会给出一个flag值。_

### **答题**

_**Part1：**_

![image-20230326225206394](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230326225206394.png)

_**Part2：**_

![image-20230326225315154](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230326225315154.png)

_**Part3：**_

![image-20230326225411070](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230326225411070.png)

_**Part4：**_

![image-20230326225512847](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230326225512847.png)

_**Part5：**_

![image-20230326225632300](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230326225632300.png)

_**Part6：**_

![image-20230326225759731](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230326225759731.png)

_**Part7：**_

![image-20230326225838033](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230326225838033.png)

_**Part8：**_

![image-20230326225910031](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230326225910031.png)

对话结束，得到一个flag：

![image-20230326225933032](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230326225933032.png)

> 最后得到的flag值为：THM{TCP\_CHATTER} 。

![image-20230326225109100](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230326225109100.png)

## UDP/IP介绍

用户数据报协议(UDP-**U**ser **D**atagram **P**rotocol)是另一种用于设备之间进行数据通信的协议。

与TCP不同，UDP是一种无状态协议，它不需要通过在两个设备之间保持恒定的连接来发送数据；UDP协议不会发生三次握手过程，两个互相通信的设备之间也不存在任何同步。

UDP主要用于应用程序可以容忍数据丢失的情况下(如视频流程序或语音聊天程序)，下面是对UDP协议的优缺点比较：

* UDP的优点：UDP比TCP快得多；UDP让应用层(用户软件)决定是否控制数据包的发送速度；UDP不像TCP那样在设备上保留连续连接。
* UDP的缺点：UDP不关心发送的数据是否成功被接收，容易发生数据丢失；UDP这种不稳定的数据传输方式可能会给用户糟糕的体验。

如前所述，在两台设备之间建立UDP连接时不会发生任何过程，这意味着UDP不考虑数据是否被接收，也没有TCP所提供的保护措施（例如保护数据完整性）。

UDP数据包比TCP数据包简单得多，并且有更少的标头，然而，这两个协议也能共享一些标准的标头，如下所示：

* Time to Live (TTL-生存时间)：该字段为数据包设置了一个过期计时器，即使数据包永远无法到达目标主机，这也不会阻塞你的网络。
* Source Address(源地址)：发送数据包的设备的IP地址，以便数据知道返回到哪里。
* Destination Address(目标地址)：数据包要发送到的设备的IP地址，以便数据知道下一步要传输到哪里。
* Source Port(源端口)：此值是发送方为发送TCP数据包而打开的端口，该端口号的值是随机选择的(从0-65535之间 选择尚未被使用的端口)。
* Destination Port(目标端口)：远程主机（接收数据的主机）上运行的应用程序或服务的端口号，例如，默认运行 Web 服务器的端口 80；与源端口不同，目标端口号并不是随机选择的。
* Data(数据)：这个标头是存储数据的地方，所存储的是正在传输的文件的字节。

通过UDP通信与通过TCP通信的过程是不同的，我们应该记住UDP协议是无状态的，在UDP传输数据期间并不会发送确认数据包（ACK）

下图显示了Alice和Bob之间的正常UDP通信，在现实生活中，UDP通信过程将发生在两个设备之间。

![image-20230326234005977](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230326234005977.png)

**答题**

![image-20230326234125159](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230326234125159.png)

## 端口基础介绍&相关实例练习

**端口基础介绍**

Port(端口)是数据进行交换的关键点，Port也有港口的含义，此名称或许恰如其分；你可以想象在海边有一个又一个港口，所有想要靠岸停放的船舶都必须前往与船舶尺寸和船舶用途相匹配的港口，例如，一艘游轮不能停靠在为渔船而建造的港口处，反之亦然。

这些端口会规定什么服务才可以使用什么端口号——如果某个服务和某个端口不兼容，就不能为该服务分配此端口；网络设备在相互通信时也将使用端口来执行一些严格的规则，当两个设备建立起连接之后，设备所发送或接收的任何数据都将通过端口进行通信。

在计算机中，端口号所对应的是 0 到 65535 之间的数值。

由于端口的范围可以是0-65535之间的任何数值，因此很快就会出现难以跟踪哪个应用程序正在使用哪个端口的风险，值得庆幸的是，我们能够将应用程序、软件和行为与一组标准规则相关联；例如，通过强制任何web浏览器的数据都必须通过端口80发送，软件开发人员可以设计出多种使用80端口进行通信的web浏览器，如Chrome浏览器、Firefox浏览器等，而这些浏览器最终都能以相同的方式解释数据。

这意味着现在所有的浏览器都有一个共同的标准规则——通过80端口发送数据；但是浏览器的外观、使用感觉和易用性则取决于设计师或用户的个人选择。

除了web浏览器的标准规则是“使用端口80进行通信”之外，其他一些协议也已经分配了对应的标准规则，接下来我们将探讨 和一些其他协议相对应的标准规则：

_tips：任何在 0 到 1024 之间的端口都被称为普通端口。_

* **F**ile **T**ransfer **P**rotocol (**FTP**协议)：文件传输协议；默认使用21端口；此协议用于构建在客户机-服务器(C/S)模型上的文件共享应用程序，这意味着你可以从FTP服务器上下载文件。
* **S**ecure **Sh**ell (**SSH**协议)：安全外壳协议；默认使用22端口；该协议可用于通过基于文本的管理界面安全地登录目标系统。
* **H**yper**T**ext Transfer Protocol (**HTTP**协议)：超文本传输协议；默认使用80端口；这个协议为万维网(WWW)提供了基础，你的浏览器能够使用该协议来访问或者下载网页上的文本、图像和视频等资源。
* **H**yper**T**ext **T**ransfer **P**rotocol **S**ecure (**HTTPS**协议)：超文本传输安全协议；默认使用443端口；该协议与HTTP协议相同，但是另外还增加了安全加密特性。
* **S**erver **M**essage **B**lock (**SMB**协议)：服务器消息块协议，是一种网络文件系统访问协议；默认使用445端口；该协议类似于文件传输协议(FTP)，但是，除了文件之外，SMB协议还允许你共享打印机等设备。
* **R**emote **D**esktop **P**rotocol (**RDP**协议)：远程桌面协议；默认使用3389端口；该协议是一种使用可视桌面界面登录目标系统的安全方法(与SSH协议基于文本界面的限制不同)。

我们在此只是简单地介绍了网络安全中的一些常见协议，你可以在以下链接中找到1024个常用端口列表：

> http://www.vmaxx.net/techinfo/ports.htm

注意：上述这些协议只是默认会遵循标准规则来使用端口，也就是说，你其实可以在不同的端口上来管理与协议交互的应用程序，而不只是使用标准端口(比如在8080端口上而不是在80标准端口上运行web服务器)；但是应用程序将假定遵循标准规则来使用端口，因此你在通过浏览器访问目标服务器时——必须在URL的最后提供冒号(:)并加上具体的端口号。

**实例练习**

在和本文相关的TryHackMe实验房间中部署实验环境并回答问题。

**答题**

_tips：通过端口“1234”连接到目标IP地址“8.8.8.8”，最后将收到一个flag。_

![image-20230327011252012](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230327011252012.png)

> 最后得到的flag是：THM{YOU\_CONNECTED\_TO\_A\_PORT} 。

![image-20230327011307845](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230327011307845.png)
