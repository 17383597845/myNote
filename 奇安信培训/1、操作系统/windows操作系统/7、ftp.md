常见远程服务端口
tcp：
	21——ftp，与大多数internet服务一样，ftp也分为客户端和服务器端。
	会建立两个连接：
		控制信息连接：不进行数据传输
		数据连接：
		
![[Pasted image 20231101155519.png]]

![[Pasted image 20231101155622.png]]

![[Pasted image 20231101155819.png]]

### ftp服务器搭建：
ftp的搭建不需要静态ip
添加ftp组件：在应用程序服务中点击详细信息——》选中internet信息服务点击详情——》添加ftp组件
配置ftp：
管理工具——》iis管理器——》右键属性默认站点——》安全账户决定是否允许匿名ftp用户——》主目录的路径下是站点权限，匿名用户权限受站点权限控制，非匿名用户的权限受站点权限和ntfs权限共同控制——》目录安全性，限制ip访问
新建站点：
在C盘下新建一个文件夹（如果放在其他盘下？）——》右键新建ftp站点——》给站点分配ip地址（下拉框选中即可，这个地址一定是当前主机的地址）——》用户隔离（可以让部分用户可以访问特定的目录，在特定需求下才会隔离用户）
安全配置：
1、取消匿名访问：
取消匿名访问（站点属性安全账户下）——》创建一个访问用户——》配置ntfs权限（给访问用户站点主目录的访问权限）
2、配置目录安全性
配置目录安全性（站点属性安全账户下）——》限制部分ip或允许部分ip的访问

### ftp访问:
	两种方法访问：
	1、ftp://ip/
	2、
	ftp 服务器端口
	匿名用户：anonymous
	下载文件：get 文件名