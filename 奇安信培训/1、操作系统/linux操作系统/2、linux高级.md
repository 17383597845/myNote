### 一、安装软件
### 1、源码安装：
		编译：使用gcc编译器将源码编译成目标文件
		链接：再次使用gcc编译器将目标文件链接成二进制文件
		make程序：一次编译多个文件夹，make会在当前目录下搜索makefile这个文本文件，而makefile里面则记录了源码如何编译的详细信息。make会自动判别源码是否经过变动了而自动更新执行文件
		config：一个检测程序来检测用户的操作系统（通常软件开放商都会写一个检测程序来检测用户的操作环境），该检测程序完成后，就会主动新建makefile的规则文件。
		tarball文件：就是将源代码先tar打包，然后通过压缩技术压缩（最常见的就是gzip压缩）
	安装步骤：
		1、将软件源码或tarball文件下载至/usr/local，并解压
		2、解压后查看install域readme文件，这两个文件中详细的介绍了本软件的安装方法和注意事项
		3、执行configure命令，生成makefile（./confiure  --prefix=/usr/local/apr   指明安装路径，因为configure命令会生成makefile文件所以在这里指定安装目录是对的）
		4、执行make clean；make将源码编译为二进制文件
		5、执行make install 将上一步编译号的二进制文件安装到指定目录中去
		**注：** 第四步和第五步使用 make && make install一个命令，&&代表两个命令要不一起成功，要么一起失败
	校验文件md5值：
			linux：md5sum 文件名
			windows：certutil -hashfile 文件名 md5
### 2、jdk的安装
	jdk的安装只需要解压缩之后，配置环境变量即可：
```bash
	cp ~/jdk-7u80-linux-x64.tar.gz /usr/local
	tar -zxvf jdk-7u80-linux-x64.tar.gz
	rpm -qa | grep java  # 查看当前操作系统已安装的java包
	rpm -e --nodeps 要卸载的软件包     #-e参数表示卸载，--nodeps是强制卸载
	mv jdk-7u80 jdk  #重命名
```
	java环境变量配置三个：
										1、javahome
										2、path
										3、classpath
	配置系统环境变量：
```bash
		vim /etc/profile
		export JAVA_HOME=/usr/local/jdk            #在文件中输入
		export PATH=$PATH:${JAVA_HOME}/bin
		export CLASSPATH=.:${JAVA_HOME}/lib
		source /etc/profile #使配置生效
```

虚拟机文件共享：
	开启共享：（在虚拟机关机的情况下）虚拟机设置——》选项——》共享文件夹启用——》添加一个宿主机路径作为共享路径，名称一定不能有中文——》启用此共享
	挂载共享目录：
			创建挂载点（mkdir  /mnt/share）——》vmware -hgfsclient（检测文件共享是否开启）——》挂载共享文件（vmhgfs -fuse  .host:/ /mnt/share/）——》进入挂载点（cd /mnt/share）

### 3、rpm：编译好的的软件，然后打包成一个特定的包格式
	特点：
	已经编译过程序和设置文件等数据，不用重新编译
	rpm文件本身提供了软件版本、依赖属性软件名称、软件用途等信息
	rpm在安装前，会先检测系统的硬盘容量，系统版本等，避免文件被错误安装
	rpm管理的方式使用数据库记录rpm文件的相关参数，便于查询，删除，和修改
rpm默认安装路径：
	![[Pasted image 20231104103806.png]]
rpm安装：
```bash
rpm -ivh rmp包文件名 # 一般这个命令就行了
```
![[Pasted image 20231104104018.png]]

rpm查询：
![[Pasted image 20231104104046.png]]

rpm卸载和重建数据库：
![[Pasted image 20231104104234.png]]

rpm安装mysql：
注意rpm的包既可以使用rmp安装，也可以使用yum安装，建议使用yum安装（因为yum会自动拉去依赖）
```bash
yum install -y mysql57-community-release-el7-10.noarch.rpm #安装完成二进制文件
yum -y install mysql-community-server #安装mysql的客户端可服务器端
#如果报错官方密钥出错，则重新导入官方密钥
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
service mysqld start #开启服务
service mysqld status #查看服务状态
# 在日志文件/var/log/mysqld.log中查看初始密码
cat /var/log/mysqld.log | grep password
#进入mysql，修改密码
set password=password("Admin@123");
```

### 4、yum安装：
![[Pasted image 20231104104601.png]]

更换yum源：
源的所在地：/etc/yum.repos.d/
备份原有yum源——》下载国内的yum源，并重命名为默认yum源（如：CentOS-Base.repo）——》yum makecache生成缓存
如果不允许出网的情况下，就只能使用本地yum源
更改本地yum源
```bash
mkdir /mnt/cdrom  #创建挂载点
mount /dev/sr0 /mnt/cdrom/ #将光盘挂载
mv CentOS-Base.repo CentOS-Bse.repo.163
mv CentOS-Base.repo CentOS-Bse.repo.bak CentOS-Base.repo CentOS-Bse.repo
vim CentOS-Base.repo 
[base]
# mirrorlist
baseurl=file:///mnt/cdrom
[updates]
# mirrorlist
baseurl=file:///mnt/cdrom
[extras]
# mirrorlist
baseurl=file:///mnt/cdrom
[centosplus]
# mirrorlist
baseurl=file:///mnt/cdrom
enabled=1#设置为开机自启动
yum clean all #清除缓存
yum makecache #生成缓存
```
yum命令：
```bash
yum search # 查询yum仓库是否存在某个软件
yum install --installroot=/some/path#配置安装路径
yum remove 软件名#移除一个软件
```




## 5、mysql安装：
	两种安装方法：
		1、源码安装，优点是安全包比较小，只有十多M，缺点是安装的依赖的库多，安装编译时间长，安装步骤复杂容易出错
		2、使用官网编译好的额二进制文件安装
	启动mysql服务的方法：
		1、使用service启动service mysqld start
		2、使用功能mysqld脚本启动：/etc/init.d/mysql start
		3、使用safe_mysqld启动，先使用命令cd /usr/bin进入bin目录下，在使用命令./mysqld_safe
		开机启动：chkconfig -add mysqld
	关闭mysql服务
		![[Pasted image 20231104140828.png]]
	重启mysql服务：
	![[Pasted image 20231104140845.png]]
	mysql目录结构：
	mysql设置远程登录：
		1、修改/etc/mysql/my.conf，找到bind-address=127.0.0.1改为0.0.0.0
		2、给用户权限：
```sql
	grant all on *.* to root@"%" identified by '123455'with grant option;
	grant all priviliges on *.* to 'root'@'%' identified by '123456' with grant option;
```
![[Pasted image 20231104141552.png]]

mysql修改启动用户
	步骤：
	killmysql进程——》切换到普通用户——》启动mysql
	![[Pasted image 20231104141747.png]]
	修改mysql默认端口：
	/etc/my.conf


## ssh服务：
	配置文件位置：/etc/ssh/sshd_config
	![[Pasted image 20231104155626.png]]
### 二、iptables防火墙
#### iptables防火墙：
工作在网络层，针对tcp/ip数据包实施过滤和限制，是典型的包过滤防火墙
![[Pasted image 20231104142138.png]]
四表，五链。
规则：指的是网络管理员预定义的条件

![[Pasted image 20231104143158.png]]
![[Pasted image 20231104144030.png]]


![[Pasted image 20231104143543.png]]
![[Pasted image 20231104144301.png]]



![[Pasted image 20231104144155.png]]


![[Pasted image 20231104144341.png]]


![[Pasted image 20231104144448.png]]

```bash
service firewalld stop
service firewalld disable
systemctl start iptables.service
systemctl status iptables
iptables -L --line-number
iptables -F#清空规则
iptables -A INPUT -s 192.168.1.1 -j REJECT#-A在末尾追加
iptables -I INPUT -s 192.168.1.1 -P tcp -d 192.168.111.50 --dport 22 -j ACCEPT#-I指的是在首行添加规则
iptables -R INPUT 2 -s 192.168.1.1 -j DROP#-R修改规则,2是规则编号
iptables -D INPUT 2#-D删除规则，2表示规则编号
iptables -P FORWORD DROP #修改链的默认策略
service iptables save #保存防火墙规则
history | grep iptables #查看带有iptables的历史指令
```





### 三、shell编程

![[Pasted image 20231104163040.png]]
![[Pasted image 20231104163107.png]]
![[Pasted image 20231104163445.png]]
![[Pasted image 20231104164315.png]]

![[Pasted image 20231104170440.png]]

#### Docker：

配置