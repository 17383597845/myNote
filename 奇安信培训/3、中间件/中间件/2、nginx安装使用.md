### 1、特点：
	高性能：官方5w连接，实际2W连接
	配置简单，易上手
	内存占有小，成本低    
	不需要安装，只需要解压缩即可
### 2、目录结构
conf：配置文件
docs：说明文档
html：存放html文件，是默认根目录
logs：存储日志文件
temp：存放临时文件
### 3、关闭nginx

![[Pasted image 20231116110005.png]]
### 4、安全配置
在nginx.conf中：
	修改默认端口号
	修改默认根目录
	修改默认页面
```bash
tasklist | find "nginx.exe"
nginx -s stop
taskkill /f /t /im nginx.exe
start nginx.exe
```