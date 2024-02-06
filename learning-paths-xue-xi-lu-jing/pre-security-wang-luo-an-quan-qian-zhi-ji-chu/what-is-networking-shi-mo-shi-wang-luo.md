# What is Networking？(什么是网络？)

本文相关的TryHackMe实验房间链接：https://tryhackme.com/room/whatisnetworking

本文相关内容：学习一些关于计算机网络的基础知识。

![img](https://assets.tryhackme.com/room-banners/networking.png)

## 简介

网络是连接在一起的东西，例如，你的朋友圈：你们都因为相似的兴趣、爱好、技能和类型而联系在一起。

网络可以在各行各业中找到:

* 城市公共交通系统
* 国家电网等基础设施
* 你和邻居会见和互相问候
* 邮寄信件和包裹的邮政系统

在计算机领域，网络的概念和上述例子类似，只是分散到了各个技术设备上；以你的手机为例，你拥有它的原因是为了获取东西。我们将介绍这些设备如何相互通信以及它们遵循什么规则。

在计算机领域，一个网络可以由2个到数十亿个设备组成，这些设备包括你的笔记本电脑和手机，安全摄像头，交通信号灯，甚至农场。

网络已经融入了我们的日常生活，无论是收集天气数据，还是为家庭供电，甚至是确定谁在道路上有优先通行权都和网络有所关联；由于网络是嵌入在现代社会中的，所以网络也是网络安全中必须掌握的一个基本概念。

以下图为例，Alice, Bob和Jim已经组成了他们的网络。

![image-20230325111701678](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230325111701678.png)

网络有各种各样的形状和大小，这也是我们将在本文中讨论的内容。

**答题**

![image-20230325111758447](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230325111758447.png)

## 什么是互联网？

现在我们已经了解了什么是网络，以及在计算机中网络是如何定义的(在计算机领域：网络表示多个连接的设备)，让我们来探索什么是互联网。

互联网是一个巨大的网络，它由许许多多小网络组成。基于上一小节中的例子，现在让我们想象 Alice 交了一些名叫 Zayn 和 Toby 的新朋友，她想把他们介绍给Bob和Jim，而问题是Alice是唯一一个和Zayn和Toby说同一种语言的人，所以Alice必须成为传达信息的信使。

![image-20230325112358537](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230325112358537.png)

因为Alice会说两种语言，上图中的其他人可以通过Alice实现互相交流——这从而形成一个新的网络。

互联网的第一次迭代是在20世纪60年代末的 ARPANET 项目中，该项目由美国国防部资助，是第一个记录在案的计算机网络；然而，直到1989年，蒂姆·伯纳斯·李 才通过创建万维网(WWW-**W**orld **W**ide **W**eb) 发明了我们现在所知道的互联网，至此，互联网才开始被用作存储和共享信息的存储库(就像今天一样)。

让我们将上面例子中的Alice的朋友网络与计算机设备联系起来，互联网看起来将像下面这类图表的放大版:

![image-20230325113424671](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230325113424671.png)

如前所述，互联网是由许多小型网络连接在一起组成的；这些小网络被称为专用网络(private networks)，连接这些小网络的网络被称为公共网络(private networks)——也就是Internet 。所以，我们可以概括得出，计算机网络可以是以下两种类型之一：

* Private network（专用网络）
* Public network（公共网络）

tips：在网络上，计算机设备将使用一组标签来标识自己。

**答题**

![image-20230325114031504](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230325114031504.png)

## 识别网络上的设备

为了通信和维持秩序，网络上的设备必须具有识别性和可识别性，如果你都不知道你在和谁说话，那么网络又有什么用呢?

计算机网络中的设备与人类非常相似，人类有两种被识别的方式:

* 名字
* 指纹

我们可以改变名字，但是我们不能改变指纹，每个人都有自己的一套指纹，这意味着即使改了名字，我们仍然有唯一的身份标识；在网络中的计算机设备具有类似的被识别方式：

* IP地址
* 媒体访问控制(MAC-Media Access Control)地址——可以把它看作是一个类似于序列号的地址。

### **IP Addresses**

简单地说，IP地址(或Internet协议)可以在一段时间内用作识别网络上的主机的方式，然后该IP地址也可以（在前一个设备使用该IP地址的时间过期后）与另一个设备相关联，而不会使IP地址发生改变。

让我们精确地划分IP地址：

![image-20230325182903071](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230325182903071.png)

IP地址是一组共32位的二进制数字，可分为4个8位二进制数，这些 8位二进制数(_**octet**_) 的值将被汇总为网络设备的IP地址；这组数字是通过被称为 IP 寻址和子网划分的技术计算出来的，此处需要理解的重点是：IP 地址可以因设备而异，但不能在同一个网络中同时被多个设备使用。

tips：一个字节由8个二进制位组成，所以一个IPv4地址共四个字节。

IP地址遵循一组被称为协议的标准，这些协议是网络的主干，迫使许多设备使用同一种语言进行通信；设备既可以在私有网络（专用网络）上，也可以在公共网络上。根据设备所处的位置，将决定它们所拥有的IP地址类型：公共IP地址或者私有IP地址。

公共地址可用于在Internet上识别设备，而私有地址可用于在其他设备之中识别设备。以下面的表格和截图为例，假如我们有两个设备在专用网络（私有网络）上:

![image-20230325185159663](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230325185159663.png)

以上两台设备将能够使用它们的私有IP地址相互通信，但是，从这两台设备发送到Internet的任何数据都将被相同的公共IP地址识别。公共IP地址可由你的互联网服务提供商(ISP-Internet Service Provider)提供，并且按月收费。

![image-20230325185615792](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230325185615792.png)

随着越来越多的设备联网，想要获得一个未被使用的公共地址也变得越来越难。根据网络行业巨头思科公司的估计，到2021年底，将有大约500亿台设备连接到互联网([思科, 2021](https://www.cisco.com/c/dam/en\_us/about/ac79/docs/innov/IoT\_IBSG\_0411FINAL.pdf))。但是IP地址不止一个版本，到目前为止，我们只讨论了一个版本的互联网协议寻址方案，即 IPv4 ，它使用的是一个具有2^32(42.9亿)个IP地址的编号系统。

IPv6是互联网协议寻址方案的新迭代版本，能够帮助解决IP地址短缺的问题，IPv6有一些好处：

* 支持最多2^128(340万亿+)个IP地址，有助于解决IPv4面临的地址短缺问题。
* 由于采用了新的方法，IPv6的效率更高。

下面的截图显示了IPv6地址和IPv4地址的实例：

![image-20230325191000099](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230325191000099.png)

### **MAC Addresses**

网络上的设备都有一个物理网络接口，和接口对应的是该设备主板上的一个微型芯片板；设备的物理网络接口会由制造芯片的工厂分配一个唯一的地址，此地址被称为MAC(媒体访问控制-**M**edia **A**ccess **C**ontrol)地址。MAC地址是一个具有十二个字符的十六进制数字(十六进制：是一种以16为基数的数字系统，可用于在计算机中表示数字)，每一位十六进制的数字都代表两个字符，在具体表示十六进制数时会用冒号分隔，这些冒号是分隔符，例如：`a4:c3:f0:85:ac:2d`。

MAC地址的前六个字符（前三位MAC地址）代表制作物理网络接口的公司，MAC地址的后六个字符则是唯一的数字。

![image-20230325192759185](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230325192759185.png)

然而，关于MAC地址的一个有趣的事是，它们可以被伪造或“欺骗”。当一台联网设备使用MAC地址伪装成另一台设备时，就会发生MAC地址欺骗，当MAC地址欺骗发生时，通常会破坏一些实现得很差的安全设计，这些（安全性差的）安全设计会假设在网络上通信的设备是可信的。以下面的场景为例：如果防火墙已被配置为允许管理员 MAC 地址之间发生任何通信，如果一个设备伪造或“欺骗”管理员的MAC地址，防火墙就会认为它正在接收来自管理员的通信（而实际上并没有）。

咖啡馆和酒店等场所在使用“Guest”或“Public”Wi-Fi时经常会使用MAC地址进行控制，这种配置可以提供更好的服务，也就是说，如果你愿意为每台设备支付费用，就可以以一定的价格获得更快的连接。

### **答题**

在和本文相关的TryHackMe实验房间中 部署交互式实验环境并回答下面问题。

_tips：此交互式实验环境模拟了酒店的Wi-Fi网络，你必须为上网服务付费。你可以注意到路由器不允许Bob的数据包(蓝色)进入TryHackMe网站，并将这些数据包放入垃圾箱中，但是Alice的数据包(绿色)却可以通过，因为Alice已经支付了使用Wi-Fi所需的费用；现在我们尝试将Bob的MAC地址更改为与Alice的MAC地址相同，看看会发生什么事情。_

正常的通信情况：

![image-20230325200205269](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230325200205269.png)

伪装MAC地址进行通信：

![image-20230325200413832](https://c/Users/Vimalano2ise/AppData/Roaming/Typora/typora-user-images/image-20230325200413832.png)

得到flag：

![image-20230325200323536](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230325200323536.png)

> 最后得到的flag为：THM{YOU\_GOT\_ON\_TRYHACKME} 。

![image-20230325200012653](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230325200012653.png)

## 关于Ping (ICMP)

Ping是我们可用的最基本的网络工具之一。Ping使用ICMP (Internet Control Message Protocol)报文来判断设备之间连接的性能，例如连接是否存在或者连接是否可靠。

如下面的截图所示，ICMP报文在设备之间传输所花费的时间可通过ping来测量；这种测量是使用 ICMP 的回波数据包(echo packet)和来自目标设备的 ICMP 回波应答(echo reply)来完成的。

![image-20230325115450514](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230325115450514.png)

Ping可以针对网络上的设备执行，例如你的家庭网络或者网站等资源。ping工具易于使用，并能安装在Linux和Windows等操作系统上，执行 ping 命令的简单语法是 `ping IP address or website URL` 。

基于上面的ping命令示例截图：我们正在ping一个私有地址为192.168.1.254的设备，我们可以看到已发送了6个ICMP数据包（手动按ctrl+c按键停止ping命令），所有这些数据包是在平均5.3秒的时间内收到的。

**答题**

在和本文相关的TryHackMe实验房间中部署实验环境并回答下面问题。

_tips：在部署的实验环境中对地址“8.8.8.8”使用ping命令。_

![image-20230325120527199](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230325120527199.png)

> 执行ping命令得到的flag为：THM{I\_PINGED\_THE\_SERVER} 。

![image-20230325120413513](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20230325120413513.png)
