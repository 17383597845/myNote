
## [攻防世界-fileinclude](https://www.cnblogs.com/niyani/p/16954008.html)

一道简单的文件包含题目，源代码如下

 ![](https://img2023.cnblogs.com/blog/3014598/202212/3014598-20221205233802733-806789308.png)

一、代码分析

此题中关键代码为

![](https://img2023.cnblogs.com/blog/3014598/202212/3014598-20221205233844352-1485366544.png)

 分析此处代码可知，$lan的值是cookie中language所对应的值，当该值不为english时，会将$lan的值与.php字符串进行拼接之后include进来

二、payload构造

使用火狐浏览器的网站调试工具（F12），在网络这一栏，向请求头中添加cookie=‘language=flag’后重发。但是，这样是不对的，应该还要使用php伪协议再加密一下（可以使用伪协议的函数include，include、require、include_once、require_once、highlight_file、show_source、file_get_contents、fopen、file、readfile），防止flag.php执行 

故应该将cookie改为'language=php://filter/read=convert.base64-encode/resource=flag'  这样就会执行include(php://filter/read=convert.bas64-encode/resource=flag.php)

![](https://img2023.cnblogs.com/blog/3014598/202212/3014598-20221206002459949-2033352140.png)

 发送之后便可以得到响应

![](https://img2023.cnblogs.com/blog/3014598/202212/3014598-20221206002532238-645554393.png)

 然后base64解码一下得flag

<?php  
$flag="cyberpeace{2f2a0a7b4bd29eac116458757262f422}";  
?>