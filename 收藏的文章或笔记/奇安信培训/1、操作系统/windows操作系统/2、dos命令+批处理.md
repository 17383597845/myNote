
## 一、dos命令
ncpa.cpl:打开网络适配器界面
secpol.msc:打开本地安全策略
四个常用命令：ipconfig、ping、netstat、systeminfo
文件夹相关命令：E:、cd、dir、mkdir（md）、rd
文件相关命令：
	添加文件：echo  >或>>  注：>是覆盖而>>是
	查看文件：type
	删除文件：del       /s/q：强制删除
	复制文件：copy
	移动文件：move
	重命名：ren
	修改关联：assoc   例如：assoc  .txt=exefile

获取所有用户名（包括隐藏用户）：wmic useraccount get name。。

用户：
			非隐藏用户：net user
			隐藏用户：
				深隐藏用户（即克隆用户）：
				浅隐藏用户：
克隆用户（克隆用户必须是隐藏用户）：
1、
net user tom$ 123456 /add   #这里创建一个隐藏目录
2、
regedit  #打开注册表项 
给sam表项权限：右键sam选择权限，勾选完全控制，应用，确定。
上面步骤后就会在sam中出现新的表项
将administrator的权限表复制粘贴到tom目录的权限表下
导出权限表
在本地用户和组中，删除tom用户
重新导入注册表项
此时，在本地用户和组中，是没有该用户的，但是在sam注册表项中，是有tom用户的。
### 注意：隐藏用户中$是用户名的一部分
使用tom用户，发现可以登录，登录后whomi， 并且可以更改c盘文件，有系统权限
wmic useraccount get name #查看隐藏用户（包括深隐藏和克隆用户）


作业：
![[EJ2@9[]LV640CLXYFSK(HAB.png]]
```shell
net localgroup student /add
net localgroup teacher /add
net user john 123456 /add
net user john /expires:2024-10-28
net user bob 123456 /add
net localgroup teacher bob /add
net user bob /active:no
net user hacker$ 123456 /add
net localgroup teacher hacker$
```

shutdown命令：
关机和定时关机：
shutdown  -s -t 100 #100s后关机
shutdown -a  #取消关机
shutdown -l  #注销
shutdown -r #重启 
## 二、批处理
1、批处理定义：批处理的每一行都是一条dos命令。批处理文件就是dos的外部命令，可以将批处理文件添加到环境变量path下就可以在任意位置执行批处理文件了。
2、批处理文件的运行：双击或在dos窗口输入批处理文件名
3、特点：以".bat"或“.cmd”结尾的文本文件，在这个文件中的每一行都是一条dos指令，批处理文件对大小写不敏感（dos不敏感，批处理自然也不敏感），批处理有流程控制语句（if  goto）和循环语句（for）。

批处理注释：rem（多行注释）或者使用双冒号::(单行注释)
命令：
	pause：暂停dos页面，需要按任意键继续
	@echo off：关闭dos指令的回显	
	>nul：关闭输出流，用于关闭输出结果的回显，如果>nul不写参数，默认参为1
				0    标准输入流
				1    标准输出流（执行成功，返回的结果）
				2    标准错误输出流（执行失败后，返回的结果）
	title：添加标题
	echo：输出字符串
			示例程序1：[[1.bat]]
	set /p : 定义变量
			[[set命令详解]]
	if  ：进行条件判断
	exit：退出
	go to：跳转到标签位置（使用冒号定义标签）
			示例程序2：[[2.bat]]
	start：后面跟文件路径，用于打开文件（如果不跟参数则默认打开一个dos命令框）