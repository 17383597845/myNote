### 1、安装和使用

首先apache要能解析php，然后给php添加mysql的扩展；怎么验证php是否可以使用mysql呢，使用phpmyadmin
解析php配置步骤：
	在httpd.conf中添加：
	a、添加解析php模块
```
	LoadMOdule php5_module "C:/WAMP/php5/php5apache2_2.dll"	
```
		
	b、添加解析类型
```
	AddType application/x-httpd-php .php
```
	c、php家目录
```
	PHPInidir "C:/WAMP/php5"
```
	![[Pasted image 20231117102322.png]]
```
添加连接mysql的扩展
	a、配置文件
	在php根目录有一个php.ini-development文件，将他改名为php.ini
	b、扩展目录:php.ini
```
	extension_dir = "C:/WAMP/php5/ext"
```
		![[Pasted image 20231117103641.png]]
	
	b、添加扩展(取消两行注释即可)
	
	![[Pasted image 20231117103758.png]]
apache原本只能处理静态的资源，但是添加了模块后可以支持动态资源
目录：
	conf:配置文件目录
	htdocs：网页默认根目录
	logs：日志目录
### 2、安全配置项
	修改网站根目录：httpd.conf
		DocumentRoot：修改为要修改的根目录
		Directory：修改为要修改的根目录
	修改网站端口：httpd.conf
```bash
	Listen 127.0.0.1:8000 #只能本机能访问，访问到8000
	Listen 8000 #指定端口
```
修改默认页面：
```bash
<IfModule dir_module>
	DireactoryIndex index.html index.php
</IfModule>
```
### 3、日志目录
linux中，日志在/var/log/httpd中
	访问日志（access.log）：访问信息，服务器处理的所有请求
		格式：远程主机ip，空白（email），空白（登录名），请求时间，请求方法+资源+协议版本，状态代码，发送字节数
	错误日志(error.log)：
	安装日志(insall.log):
	