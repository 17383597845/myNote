vpn：
	虚拟专用网络
vpn分类：
	注意mpls是需要独立建设通信网络的
![[Pasted image 20231111084313.png]]	
![[Pasted image 20231111084447.png]]
实现的网络层次：
![[Pasted image 20231111084515.png]]

实现技术：
	vpn技术的基本原理是利用tunnel技术，对传输报文进行封装，利用vpn骨干网建立专用数据传输通道，实现报文的安全传输；位于隧道两端的vpn网关，对数据进行封装和解封装
![[Pasted image 20231111085003.png]]


ipsec协议体系：
![[Pasted image 20231111085205.png]]

ipsec基本原理：
![[Pasted image 20231111085852.png]]


gre概述：
![[Pasted image 20231111090534.png]]

弊端：数据不加密
gre+ipsec：gre不加密，但是可以实现ipv4，ipv6转化；而ipsec可以加密，但是不能实现转化
![[Pasted image 20231111090823.png]]


mpls vpn 概述：
![[Pasted image 20231111091141.png]]


