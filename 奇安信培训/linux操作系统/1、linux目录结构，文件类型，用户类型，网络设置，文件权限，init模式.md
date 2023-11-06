
# 一、linux的init模式和关机命令（使用init 1找回密码）
### 1、linux的init模式：
init0：系统停机模式，系统默认运行级别不能设置为0，否则系统不能正常启动，机器关闭
init1：单用户模式，root权限（单用户就是root用户），用于系统维护，禁止远程登录，就像windows下的安全模式登录 *注* 找回密码就是使用的这个模式
init2：多用户模式，没nfs网络支持
init3：完整的多用户文本模式，有nfs网络支持，登录后进入控制台命令行模式
init4：系统未使用，一些特殊的情况可以用它来做一些事情
init5：图形化模式，登录后进入图形gui模式
init6：重启模式，默认运行级别不能设置为6，否则不能正常启动
2、开机时按e键进入一个编辑模式——》找到ro改成rw（写权限）——》在UTF-8后面加：（空格）init=/bin/bash——》ctrl-x进入单用户模式（单用户模式的当前密码是root）——》更改当前用户密码：passwd——》写入配置文件中：touch /.autorelabel——》重启：exec /sbin/init
# 二、网络设置
## 1、配置静态ip：
两种方法：
	方法一：
	修改配置文件：
	修改/etc/sysconfig/network-scripts/
	例：
	设置静态ip：
	![[Pasted image 20231103111311.png]]
	设置动态IP：
	![[Pasted image 20231103112724.png]]

方法二：
```bash
nmtui
```
### 3、静态路由设置
```bash
route #查看路由表
#修改路由
route [add|del] [-net|host] target [netmask  Nm] [gw GW] [[dev] if]
```
### 4、dns配置（dns服务器的搭建，不用搭建，了解即可）
件。
	1、安装
	2、配置区域解析文件
	3、创建a记录和ptr记录
	![[Pasted image 20231103114010.png]]
网卡配置文件的配置会覆盖方法二中的dns配置文件。
linu域名解析顺序：
	查找缓存——》hosts——》找dns服务器（网卡配置文件或dns配置文件找dns配置）
	![[Pasted image 20231103114500.png]]

# 三、常见命令
关机命令：
```bash
	shutdown
	poweroff
	init 0
```
重启命令：
```bash
	reboot
	init 6
	shutdown -r
```
隐藏权限：https://blog.51cto.com/u_15292540/3124767
```bash
lsattr #用户查看文件的隐藏权限
chattr +a 文件名  #修改文件的隐藏权限
```
修改文件的所属组：
命令格式：chgrp [选项] 新组 文件/目录
	选项：
	`-R`：递归操作，将目录中的文件和子目录一并更改为新组。
    `--reference=file`：使用参考文件的组来更改指定文件的组。
    `-v`：显示执行的操作。
```bash
chgrp 组名 文件名 #修改文件或目录的所属组
chown 属主:属组 文件名#用法和chgrp类似
```
其他命令：
```bash
cd - #返回上一级工作目录
last #查看近一个月登录者的信息
gunzip#解压缩文件的程序
tar -zcvf 1.rar 1.txt  #压缩1.txt文件到1.rar
tar -zxvf 1.rar 1.txt #解压1.rar到1.txt
ps #查看进程
kill 参数 进程号#删除进程，强制用-9参数
top#类似windows的任务管理器
ifconfig ens33 down（up）#关闭（开启）网卡
#netstat ：查看网络套接字状态** 查找结果：
netstat -at  | grep  ssh
head -n 文件#查看文件的前n行
tail -n  文件#查看文件后n行
cat -n/-b 文件名#查询结果按行编号
find /home  -atime +10#查找前十天的内容
date#显示当前服务器的时间
	date +%Y-%m-%d %H:%M:%S  #以特定的格式展示时间
	date -s "2024-11-3 16:40:00"  # 修改时间
	ntpdate ntp2.aliyun.com          #在能够正常dns解析，并能正常出网的情况下可以使用和时间服务器同步时间


wc -l 文件 #统计文件有多少行
gpasswd -a和usermod -aG的区别：
gpasswd -a是将用户添加到组，用户原有组不变，usermod -aG是将用户移入组，用户不再属于原有组
```
netstat命令：
![[Pasted image 20231103115715.png]]
磁盘空间查看：df命令
![[Pasted image 20231103160243.png]]
du：查看文件和目录的磁盘使用情况
	du 选项  文件
	![[Pasted image 20231103160338.png]]





# 四、linux的目录结构、文件类型和用户类型，挂载和开机挂载

### 1、目录结构
	1. `/`（根目录）：Linux文件系统的根目录，所有其他目录和文件都从这里开始。
    
2. `/bin`：存放系统启动和运行时需要的基本命令，如ls、cp、mv等。
    
3. `/boot`：包含系统引导程序和内核文件。
    
4. `/dev`：包含设备文件，表示系统上的各种硬件设备。
    
5. `/etc`：存放系统配置文件，包括网络配置、用户账户配置等。
    
6. `/home`：用户的主目录，每个用户有一个单独的子目录，通常以用户名命名。
    
7. `/lib`：包含系统启动和运行时需要的共享库文件。
    
8. `/media`：用于挂载可移动设备，如光盘、USB驱动器等。
    
9. `/mnt`：用于挂载临时文件系统。
    
10. `/opt`：用于安装第三方软件的目录。
    
11. `/proc`：虚拟文件系统，提供有关正在运行的进程和内核状态的信息。
    
12. `/root`：超级用户（管理员）的主目录。
    
13. `/sbin`：存放系统启动和运行时需要的系统命令，通常只能由管理员访问。
    
14. `/srv`：包含服务相关的数据文件。
    
15. `/tmp`：用于存放临时文件。
    
16. `/usr`：存放系统用户和应用程序文件，包括用户安装的软件。
    
17. `/var`：包含经常变化的文件，如日志文件、数据库文件等。
### 2、文件类型
	普通文件：-
	目录：d
	字符设备文件：c
	块设备文件：b
	符号链接文件：l
	
![[Pasted image 20231103141609.png]]
### 3、用户类型


用户类型：
	![[Pasted image 20231103141756.png]]
	普通用户：
	![[Pasted image 20231103142016.png]]
	虚拟用户：
	![[Pasted image 20231103142038.png]]

### 4、挂载和开机挂载

	**什么是挂载？** 
		将光盘/U盘/分区/网络存储等设备装到某个linux目录
		关于分区/U盘/光盘等设备文件？
		分区/dev/sda1   /dev/sda2等等；U盘，一般是/dev/sdb1等
		光盘，一般是/dev/cdrom，指向/dev/sr0
		查看分区和设备：lsblk
		挂载设备
		准备挂载点-----------------mkdir /mnt/dvd
		挂载光盘-------------------mount /dev/cdrom /mnt/dvd
		通过挂载点访问光盘---------ls /mnt/dvd
		卸载挂载设备---------------umount /mnt/dvd
	
	**配置开机挂载：**
		开机挂载解析
		在/etc/fstab文件中添加设备记录
		文件系统类型：iso9660、xfs、swap、ext4、...
		设备路径                           挂载点                             文件系统类型              挂载选项            0       0
		/dev/cdrom/          /repos/openEuler    iso9660           defaults，ro
		
		创建挂载点       配置fstab    检查配置            重启验证
		创建挂载点：mkdir -p /repos/openEuler
		配置fstab：vim  /etc/fstab
		mount -a 
		重启系统，检查开机挂载：reboot          ls  /repos/openEuler

# 五、文件权限
### 1、特殊文件权限
	suid：冒险位，通常用在可执行文件上，以指示该文件再执行时应以文件的所有者权限来运行（chmod u+s   文件名）
	sgid：强制位让其他用户拥有文件拥有组的权限，对于目录来说，该目录下所有文件的所属组均为父文件的所属组，但是属主还是创建用户；。（chmod g+s 目录名或文件名）
	sbit：粘滞位，仅对目录有效。若目录有此权限，目录的其他用户的x权限位置变成t（chmod +t 目录名）。
	
![[Pasted image 20231103150238.png]]
2、文件隐藏权限：
	lsattr：用户查看文件的隐藏权限
	chattr：修改文件的隐藏权限
![[Pasted image 20231103150416.png]]



# 六、练习

![[Pasted image 20231103153925.png]]
# 七、总结
![[Pasted image 20231103160428.png]]