udp
	67，68：dhcp服务，67-dhcp server用来接收下级客户请求分配ip，68-dhcp client是向客户发请求或失败的回应
	dhcp可以给用户分配永久固定的ip地址
	dhcp可以同其他方法获得ip地址的主机共存（例如手动分配的今天IP）
![[Pasted image 20231101140726.png]]
dhcp工作原理：
![[Pasted image 20231101140751.png]]
从图中可以看出client只会处理第一个接收到的dhcp offer报文
注意：当时间到达租约的50%时，client会发送request包请求续约；达到租约时间87.5%时，client会发送request包再次请求续约；如果server还是没有接收续约，则要达到租期发送discover包重新租用。如果到期后没有可用的dhcp服务器，client就会自动给自己分配一个169.254.0.0（全球无用地址，只能在短期作为内网地址）网段的随机地址。**没有ip的时候发送discover包，有ip时发送request包** 
常见故障：dhcp欺骗。
![[Pasted image 20231101141918.png]]
防御：在防火墙或交换机，限制一个机器一段时间只能发送一次ip请求包；或者用户可以指定某个dhcp服务器，只接收他分配的ip。
dhcp服务搭建：
（1）server03和win7都设置为仅主机模式，然后关闭本地dhcp服务器（虚拟网络编辑器处取消勾选本地dhcp服务）
（2）给server03配置静态IP（仅主机不出网，可以不配置网关地址）
（3）添加dhcp组件（更dns服务一样，在网络服务详细信息中去找）
（4）添加地址池（作用域），右键新建作用域——》作用域名称随便写——》输入地址范围——》添加排除（即要保留那些地址，保留的地址可以手动分配给指定的主机）——》设置租约期限（默认八天）——》配置默认网关——》配置dns服务器地址——》netbios不用配直接下一步——》激活作用域
选项（下面的顺序就是，选项生效顺序）：
	服务器选项：作用于所有作用域
	作用域选项：作用于当前作用域
	保留选项：作用于当前作用域
ipconfig /renew：如果没有获取dhcp分配的地址，可以使用本命令重新申请