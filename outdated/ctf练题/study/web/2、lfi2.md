### 文章目录

- [一、文件包含漏洞分类](https://www.cnblogs.com/Hardworking666/p/15866130.html#_7)
- [二、文件包含漏洞原理](https://www.cnblogs.com/Hardworking666/p/15866130.html#_23)
- [三、文件包含函数](https://www.cnblogs.com/Hardworking666/p/15866130.html#_38)
- [四、测试是否存在本地文件包含（LFI）漏洞](https://www.cnblogs.com/Hardworking666/p/15866130.html#LFI_70)
- [五、文件包含漏洞实例](https://www.cnblogs.com/Hardworking666/p/15866130.html#_100)
- - [“百度杯”CTF比赛 2017 二月场include](https://www.cnblogs.com/Hardworking666/p/15866130.html#CTF_2017_include_101)

# 一、文件包含漏洞分类

LFI(Local File Inclusion)  
**本地文件包含漏洞**，指的是能打开并包含本地文件的漏洞。大部分情况下遇到的文件包含漏洞都是LFI。  
为了方便本文把LFI直接称为文件包含漏洞。

RFI(Remote File Inclusion)  
**远程文件包含漏洞**。是指能够包含远程服务器上的文件并执行。由于远程服务器的文件是我们可控的，因此漏洞一旦存在危害性会很大。但RFI的利用条件较为苛刻，需要php.ini中进行配置

```php
allow_url_fopen = On
allow_url_include = On
```

两个配置选项均需要为On，才能远程包含文件成功。  
在php.ini中，allow_url_fopen默认一直是On，而allow_url_include从php5.2之后就默认为Off。

# 二、文件包含漏洞原理

本地文件包含（Local File Inclusion）漏洞，是程序员在网站设计中，为方便自己在设计构架时，使用了一些包含的函数，在文件中包含一个文件。

服务器执行PHP文件时，可以通过**文件包含函数**加载另一个文件中的PHP代码，并且当PHP来执行，这会为开发者节省大量的时间。

这意味着可以创建供所有网页引用的标准页眉或菜单文件。当页眉需要更新时，只更新一个包含文件就可以了，或者当向网站添加一张新页面时，仅仅需要修改一下菜单文件（而不是更新所有网页中的链接）。

LFI 产生的原因是由于程序员未对**用户可控变量**进行输入检查，此漏洞的影响可能导致泄露服务器上的敏感文件等。  
如若攻击者能够通过其他方式在Web服务器上放置代码，那么他们便可以执行任意命令

```bash
 Directory traversal attack is also called a Local File Inclusion or LFI.
```

翻译：**目录遍历攻击**也称为**本地文件包含攻击**或**LFI**。

# 三、文件包含函数

PHP中文件包含函数有以下四种：

```php
require() // 只在执行到此函数时才去包含文件，若包含的文件不存在产生警告，程序继续运行

require_once() // 如果一个文件已经被包含过，则不会在包含它

include() // 程序一运行文件便会包含进来，若包含文件不存在产生致命错误，程序终止运行

include_once() // 如果一个文件已经被包含过，则不会在包含它
```

include和require区别主要是，include在包含的过程中如果出现错误，会抛出一个警告，程序**继续正常运行**；  
而require函数出现错误的时候，会直接**报错并退出**程序的执行。

而include_once()，require_once()这两个函数，与前两个的不同之处在于这两个函数**只包含一次**，适用于在脚本执行期间同一个文件有可能被包括超过一次的情况下，你想确保它只被包括一次以避免函数重定义，变量重新赋值等问题。

这四个函数可以将任意类型的文件当做 PHP 文件进行解析。  
示例代码：

```php
<?php
    $filename  = $_GET['filename']; // 存在可控变量
    include($filename); // 存在动态变量
?>
```

例如：

$_GET[‘filename’]参数开发者没有经过严格的过滤，直接带入了include的函数，  
攻击者可以修改  
$_GET[‘filename’]的**值**，执行非预期的操作。

# 四、测试是否存在本地文件包含（LFI）漏洞

使用 …/ 上一级目录测试：

```php
?page=../
```

会返回以下错误：

```php
Warning: include(C:\Users\Administrator\Documents\php): failed to open stream: Permission denied in C:\Users\Administrator\Documents\php\LFI\LFI_base.php on line 10
```

这意味着很有可能存在 LFI 漏洞

使用 常见 页面

```php
?page=index.html
```

若是返回 index.html 页面，也就意味着存在 LFI 漏洞

直接读取 /etc/passwd 文件(Linux)：

```php
?page=/etc/passwd
```

若是出现了 passwd 数据，那证明此网站容易受到本地文件包含的影响

# 五、文件包含漏洞实例

## “百度杯”CTF比赛 2017 二月场include

题目  
![](https://img-blog.csdnimg.cn/20210510211405868.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0hhcmR3b3JraW5nNjY2,size_16,color_FFFFFF,t_70)  
既然是文件包含，我们搜索下文件包含**关键性函数的状态**  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210510211652730.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0hhcmR3b3JraW5nNjY2,size_16,color_FFFFFF,t_70)

既然 allow_url_include打开，意味着直接能使php://input用包含post中的代码。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210510230318387.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0hhcmR3b3JraW5nNjY2,size_16,color_FFFFFF,t_70)  
发现文件：dle345aae.php  
查看此文件内容：使用php://filter协议查看曝露出来的文件的内容，因为PHP文件不能直接显示内容所以先base64显示出来，然后再解码。  
构造语句:

```php
?path=php://filter/read=convert.base64-encode/resource=dle345aae.php
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021051023054162.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0hhcmR3b3JraW5nNjY2,size_16,color_FFFFFF,t_70)  
然后base64解密即可flag

https://blog.csdn.net/Hardworking666 本人主要使用CSDN，地址献上，请多多指教。