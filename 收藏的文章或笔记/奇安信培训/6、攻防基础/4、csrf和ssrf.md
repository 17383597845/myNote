## csrf ------ 跨站请求伪造
### 一、概述
#### 1、漏洞原理

CSRF(跨站请求伪造)，攻击者通过一些技术手段欺骗用户的浏览器去访问一个自己以前认证过的站点并运行一些操作（如发邮件，发消息，甚至财产操作（如转账和购买商品））。因为浏览器之前认证过，所以被访问的站点会认为这是真正的用户操作而去运行。
![[Pasted image 20231221193905.png]]

#### 2、利用条件
1. 用户已经登陆了网站，具有执行各个功能的权限。
2. 攻击者知道功能的数据包。
3. 用户访问了攻击者构造的恶意URL。
#### 3、利用
1. burp插件：CSRF Scanner
2. 自己手动编写html页面
#### 4、csrf漏洞案例：
burp抓取创建用户功能点数据包：
![[Pasted image 20231221194138.png]]

使用CSRF Scanner插件生成恶意的html页面，并html页面放在自己服务上：
![[Pasted image 20231221194205.png]]

用户在登陆情况下，访问了恶意的html页面，将会创建一个新用户
http://xx.xx.xx.xx:8080/user.html
![[Pasted image 20231221194249.png]]



### 二、csrf漏洞探测和防御
#### 1、漏洞探测：
1.利用工具生成poc，提交数据，能修改表示存在csrf漏洞
2.手动删除referer字段值，如果能修改成功，表示存在csrf漏洞
#### 2、防御
1.当用户修改密码时需要输入原密码
2.检验referer来源，验证是否为管理员本人在操作
3.设置随机token验证（推荐）
4.设置验证码
5.http头中自定义属性并验证

## ssrf ----------服务端请求伪造

### 一、概述
#### 1、ssrf漏洞基础
服务端请求伪造(SSRF)是指一种由攻击者构造形成由服务端发起请求的一个安全漏洞，一般情况下，SSRF攻击的目标都是从外网无法访问的内部系统，借助服务端发起伪造的请求，可以访问到与它相连而与外网隔离的内部系统。
**漏洞原理：**
由于服务端提供了从其他服务器应用获取数据的功能且没有对目标地址做过滤和限制。
#### 2、ssrf--挖掘思路
##### 从功能点探测：
1.分享：通过URL地址分享网页内容
2.转码服务：通过URL地址把原地址的网页内容调优使其适合手机屏幕浏览
3.在线翻译：通过URL地址翻译对应文本的内容
4.图片加载和下载：通过URL地址加载或下载图片
5.图片、文章收藏功能
6.未公开的api实现以及其他调用URL的功能
##### 从url关键字中探测：
share wap url link src source target display sourceURL imageURL domain 等
### 二、ssrf漏洞利用
#### 1、端口扫描
```
http://100.100.100.55:83/vul/ssrf/ssrf_curl.php?url=http://127.0.0.1:8005
```
bp抓包，爆破端口号
#### 2、内网探测
```
http://100.100.100.55:83/vul/ssrf/ssrf_curl.php?url=http://172.16.0.11
```
bp抓包，爆破网段
#### 3、文件读取
使用file伪协议
```
http://100.100.100.55:83/vul/ssrf/ssrf_curl.php?url=file:///c:/windows/win.ini
```
#### 4、常用协议
```
1. http://172.16.0.1:80

2. ftp://127.0.0.1:21

3. file:///etc/passwd

4. dict://127.0.0.1:3306/info

5. gopher://ip:port/_TCP/IP数据流
```
#### 5、使用gopher协议攻击redis、mysql、fastcgi、smtp等服务、
##### gopher协议格式：
```
gopher://ip:port/_TCP/IP数据流
```
注意：
	1. gopher协议数据流中，url编码使用%0d%0a替换字符串中的回车换行
	2. 数据流末尾使用%0d%0a代表消息结束
##### gopher攻击redis：
写入定时任务：
![[Pasted image 20231221195525.png]]

写入webshell（ctf-me.org）：
![[Pasted image 20231221195615.png]]


#### 6、ssrf利用工具
```
1. https://github.com/evilAdan0s/GopherGo

2. https://github.com/tarunkant/Gopherus

3. https://github.com/assetnote/blind-ssrf-chains#redis
```
### 三、ssrf绕过和防御
#### 1、绕过：
添加端口：http://127.0.0.1:80
短网址：https://my5353.com/
数值绕过：
1. 八进制：0177.00.00.01
2. 十六进制：0x7f.0x0.0x0.0x1
3. 十进制：2130706433
跳转绕过，加@可以跳转到指定的URL：
http://www.baidu.com@127.0.0.1 会跳转到127.0.0.1SSRF-绕过
302跳转绕过：
http://challenge-719da0c18563793e.sandbox.ctfhub.com:10800/?url=https://my5353.com/GcXUL

#### 2、防御
1、过滤返回的信息，如果web应用是去获取某一种类型的文件。那么在把返回结果展示给
用户之前先验证返回的信息是否符合标准。
2、统一错误信息，避免用户可以根据错误信息来判断远程服务器的端口状态。
3、限制请求的端口，比如80,443,8080,8090。
4、禁止不常用的协议，仅仅允许http和https请求。可以防止类似于file:///,gopher://,ftp://
等引起的问题。
5、使用DNS缓存或者Host白名单的方式。