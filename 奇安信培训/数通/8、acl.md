acl：access control list
注意：acl的作用只是将流量抓出来，它就是一个抓流量的工具，实际中还有一个工具叫做前缀列表，和acl很像
	是由一系列permit或deny语句组成的，有序规则列表。他是一个匹配工具，能够对报文进行匹配和区分
	背景：针对数据流量
	![[Pasted image 20231110140342.png]]
acl应用：
	![[Pasted image 20231110140843.png]]

acl的分类：
![[Pasted image 20231110141544.png]]

acl的组成：
![[Pasted image 20231110141816.png]]

规则编号：
	![[Pasted image 20231110141944.png]]

匹配规则：
![[Pasted image 20231110142421.png]]
![[Pasted image 20231110142601.png]]
![[Pasted image 20231110144656.png]]
基本acl和高级acl：
![[Pasted image 20231110144817.png]]
acl的匹配机制：
前面的rule匹配了，马上停止匹配，后面的rule就没有用了
![[Pasted image 20231110144839.png]]

![[Pasted image 20231110145605.png]]

```bash
#拒绝流量的配置
acl 2000   #创建一个初级的acl工具对象
rule deny source 10.0.0.2 0 #配置规则
dis acl 2000 #查看acl2000的配置
int g0/0/0
traffic-filter inbound acl 2000 #进站流量绑定刚刚创建的2000对象

#在telnet里的使用
aaa #进入aaa视图
local-user x password cipher x #创建aaa用户并设置密码
local-user x privilege level 15 #给创建的用户设置一个权限
local-user x service-type telnet #允许该用户使用telnet服务
user-interface vty 0 4  #user接口下配置虚拟终端
protocol inbound telnet #配置允许远程登录方式
authentication-mode aaa #配置认证方式
acl 2000 inbound 
#注意，acl末尾有一个隐藏的deny
rule permit source any
```
