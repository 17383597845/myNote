
telnet：底层是tcp协议
![[Pasted image 20231101171817.png]]

![[Pasted image 20231101172048.png]]
3389端口（rdp协议）：是windows远程桌面的服务端口，可以说就是一个图形化的telnet
![[Pasted image 20231101172403.png]]

如何使用telnet？
1、开启telnet
	在win2003上
		services.msc：打开服务，找到telnet，启用
	在win7上
		1、添加telnet组件：控制面板——》程序——》程序和功能（打开或关闭windows功能）——》选中telnet服务器端和客户端
2、新建用户，并将该用户添加到telnetclient组中，并将用户从原有组中移除（避免telnet用户有太高权限）
打开本地用户和组：有一个telnetclients组
3、使用 telnet  ip的方式连接

远程桌面怎么使用？
1、添加一个新用户，移入remoteDesktopUsers组，并从原有组中移除他。
2、开启远程桌面
3、连接
	mstsc——》输入远程连接目标的ip——》输入用户名密码


http协议：80
https协议：443
使用iis发布web服务或使用软件（如phpstudy）

查看进程id：netstat -ano | find "80"
tasklist | find "进程号" //查看是否是系统进程，如果是系统进程不要关闭
taskkill  进程号        //杀死进程

![[Pasted image 20231101174740.png]]







