### 一、文件包含漏洞：
[[ctf中截断在文件包含中的应用]]
通常分为本地文件包含（LFI）和远程文件包含（RFI）两种类型。

本地文件包含（LFI）：
本地文件包含漏洞指的是攻击者能够通过应用程序，动态包含本地文件系统中的文件，通俗来说就是通过浏览器包含web服务器上的文件。这种漏洞最常见于服务器端脚本语言如PHP中的代码，攻击者可以通过构造恶意的文件路径来包含本地文件。成因通常是由于开发者在编写代码时没有正确过滤用户输入，或者将用户输入直接用于文件包含操作而未进行充分验证。

远程文件包含（RFI）：
远程文件包含漏洞则是指应用程序对外部资源的引用没有经过充分的验证，导致攻击者可以向应用程序注入远程文件路径，并使应用程序从远程服务器上下载并执行恶意代码。这种漏洞可能会导致严重的安全问题，包括执行任意远程代码。成因通常是由于应用程序在包含外部资源时没有进行足够的验证，并且对用户提供的输入信任过高。

### 二、文件包含php伪协议：
#### 1，
遇到文件包含需要读取源码可以使用php://filter协议:

读：php://filter/resource=文件名

php://filter/read=convert.base64-encode/resource=文件名

写：php://filter/resource=文件名&txt=文件内容

php://filter/write=convert.base64-encode/resource=文件名&txt=文件内容

#### 2，
data://和php://input作用相同，如果碰到input被过滤的情况可以用其替代。和php:input不同的是data://需要 allow_url_fopen、allow_url_include都需要打开（on）。

格式也稍稍有点不同，全部是在GET实现的

#### 3，data://text/plain ，是固定格式，后面如果有过滤可以用base64，内容可以是命令，也可以输出一些东西，看题目行事

file.php?file=data:text/plain,<?php phpinfo();?>

file.php?file=data://text/plain;base64,SSBsb3ZlIFBIUAo=

#### 4，phar://伪协议：

这个参数是就是php解压缩包的一个函数，不管后缀是什么，都会当做压缩包来解压。

用法：?file=phar://压缩包/内部文件 phar://xxx.png/shell.php （PHP > =5.3.0 压缩包需要是zip协议压缩，rar不行），将木马文件压缩后，改为其他任意格式的文件都可以正常使用。

步骤： 写一个一句话木马文件shell.php，然后用zip协议压缩为shell.zip，然后将后缀改为png等其他格式。

#### 5，zip://伪协议：

zip伪协议和phar协议类似，但是用法不一样。

用法：?file=zip://[压缩文件绝对路径]#[压缩文件内的子文件名] zip://xxx.png#shell.php。

但使用zip协议，需要指定绝对路径，同时将#编码为%23，之后填上压缩包内的文件。

?file=zip://C:\phpStudy\WWW\FileInclusion\phpinfo.zip%23phpinfo.txt
条件： PHP > =5.3.0，注意在windows下测试要5.3.0<PHP<5.4 才可以 #在浏览器中要编码为%23，否则浏览器默认不会传输特殊字符。

#### 6，包含session

利用条件：session文件路径已知，且其中内容部分可控。
思路：结合phpmyadmin，因为phpmyadmin每次登录时，会带上session

session文件的绝对路径可在phpinfo中查看，session.save_path

常见的php-session存放位置：

/var/lib/php/sess_PHPSESSID
/var/lib/php/sess_PHPSESSID
/tmp/sess_PHPSESSID
/tmp/sessions/sess_PHPSESSID

在基本的LFI攻击中，我们可以使用（../）或简单地（/）从目录中直接读取文件的内容，../../../../敲的足够多就能跳转到根目录下../../../../etc/passwd

