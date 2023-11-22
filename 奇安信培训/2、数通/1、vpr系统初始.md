# 一、vpr概念
结业的时候：防火墙会考试
学习要求：要求：知道数据包在网络上是怎么跑的
概念：vrp是华为数通产品的操作系统平台，如：路由，安全，无线，交换这些都由这个操作系统设置
# 二、文件系统、存储设备、设备管理和用户级别
### 1、文件系统：
进入内网关注的就是配置文件
![[Pasted image 20231107093724.png]]

### 2、存储设备：
![[Pasted image 20231107094402.png]]
### 3、设备管理：
![[Pasted image 20231107094428.png]]
console口：
	不能用网线，要用串口线，走的也不是网络通信而是串口通信
eth：网线口
mgt：管理口
### 4、用户级别：
![[Pasted image 20231107095111.png]]

# 三、视图和命令
### 1、视图
一共有用户视图，系统视图，接口视图，协议视图这四个视图
![[Pasted image 20231107102625.png]]

### 2、常用命令
命令结构： 关键字 参数列表（参数名 参数值）
#### a、？的用法
```bash
	di? #查询di开头的名
	dis ?#查询命令用法
```
b、管道符|的使用
```bash
<Huawei>display current-configuration | ?
	  begin    Begin with the line that matches
	  exclude  Match the character strings excluding with the regular expression
	  include  Match the character strings including with the regular expression
```
#### c、视图切换
```bash
system-view #进入系统视图，简写：sys
interface GigabitEthernet 0/0/1 #进入端口，简写：inter

return #直接退回到用户视图ctrl+z也行
sysname hhh #修改主机名
```
#### d、文件系统操作（都是在用户视图里面）
```bash
pwd
dir
copy
move
rename
delete
undelete
reset 
reboot
save
```
