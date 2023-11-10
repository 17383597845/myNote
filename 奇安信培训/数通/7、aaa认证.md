
### 1、aaa概念：
认证，授权和计费的简称，是网络安全的一种管理机制，提供了认证授权和计费三种安全功能。
### 2、AAA常见架构：
![[Pasted image 20231110095650.png]]
远端认证：有网络，有客户机，有服务器


### 3、认证方式：
	本地认证：
		在nas上认证，就是在路由器或交换机上面认证
	远端认证：
		服务器认证
![[Pasted image 20231110095931.png]]

授权：
计费：

AAA实现协议-radius（二层认证）
准入系统
![[Pasted image 20231110100127.png]]
### 4、本地认证配置方法
```bash
aaa配置
aaa #进入aaa界面
local-user xx password cipher 密码
local-user yonhuming privilege level 15 #设置用户级别
local-user yonhuming service-type telnet #设置允许的服务
全局配置（系统视图）
telnet server enable #开启telnet
user-interface vty 0 4
protocol inbound telnet #允许telnet可以控制0-4个终端
authentication-mode aaa #使用aaa认证

#本地验证
telnet 127.0.0.1 
dis users
#远程验证
telent ip
```
![[Pasted image 20231110102943.png]]




