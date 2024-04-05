今天学了什么？
linux的常用命令，查看文件，分页查看文件，移动文件，复制文件，删除文件；用户和组管理，文件权限管理；挂载外部存储设备，更改yum软件源；禁用root远程连接，；su权限管理，启用wheel用户组；查看安全日志；使用visudo命令，进入/etc/sudoers .tmp文件，修改文件可以修改用户或组的sudo权限 eg：%hyl ALL=（ALL：ALL）ALL；
SELinux模式，查看，设置。


一、基础操作：
查看默认网关记录 ---------------------route -n
查看dns/主机名信息-------------------cat /etc/resolv.conf      查看主机可以直接使用这个命令，不用找配置文件hostname（查看help）
修改网络连接-------------------------nmtui

list命令
list 目录路径
查看所有文件包括隐藏文件-------------- -a
以长格式显示------------------------- -l
带单位-------------------------------  -h
只列出目录本身------------------------ -d

cat命令
cat 文件路径
可分页显示长文件---------------------  less
翻页--------------------------------- pgUp/PgDn

mkdir命令
连同父目录递归创建------------------ -p

cp命令
cp 【选项】 源文件 目标路径
递归复制----------------- -r


rm删除文档
rm 【选项】 文件或目录
递归删除------------------------ -r
强制删除------------------------ -f

mv移动目录
mv 【选项】 源文件 目标路径

二、用户和组和权限管理
添加用户、删除用户、设置密码-------useradd  userdel  passwd   注：彻底删除userdel -r 用户名
查看用户id信息 --------------------id 用户名
添加组、删除组--------------------groupadd groupdel
给组添加成员、删除成员-------------gpasswd 【-a|-d】 用户 组

访问控制：
更改文档归属----------------------chown 用户名：组名 /用户名：组名  文档路径
查看文档--------------------------chmod 【ugo】 【+-=】【rwx】 文档路径


day03
什么是挂载？
将光盘/U盘/分区/网络存储等设备装到某个linux目录
关于分区/U盘/光盘等设备文件？
分区/dev/sda1   /dev/sda2等等；U盘，一般是/dev/sdb1等
光盘，一般是/dev/cdrom，指向/dev/sr0

挂载设备
准备挂载点-----------------mkdir /mnt/dvd
挂载光盘-------------------mount /dev/cdrom /mnt/dvd
通过挂载点访问光盘---------ls /mnt/dvd
卸载挂载设备---------------umount /mnt/dvd


配置开机挂载：
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



openEuler安装盘已经预先配置成软件仓库，可以直接使用
比如挂载或复制到本机目录/repos/openEuler：mkdir -p /repos/openEuler           mount /dev/cdrom  /repos/openEuler

三步搞定软件源：
1、禁用无效源
配置位置：/etc/yum.repos.d/*.repo
查看仓库列表：yum repolist -v
删除不用的仓库：rm -rf /etc/yum.repos.d/*.repo
2、设置有效源
添加软件源：yum-config-manager --add  软件源url地址：yum-config-manager --add file：///repos/openEuler
查看配置是否生效：cat /etc/yum.repos.d/repos_openEuler.repo
3、关闭软件来源检查
vim /etc/yum.conf
gpgcheck=0 //将1改为0可以关闭检查
重新获取源数据，确保有可用仓库：yum repolist -v


yum命令：
yum list 【软件名】
yum info 【软件名】
yum provides 文件名
yum install 软件名
yum remove 软件名
yum reinstall 软件名



快速安装lamp平台
联网：yum -y install httpd mariadb-server php-fpm php-mysqlnd
没有联网：yum -y install /root/lamp_oe1_pkgs/*.rpm（例如lamp_oe1_pkgs/子目录）

SELinux状态控制：
查看SELinux运行状态：getenforce
切换为宽松模式：setenforce 0
切换为强制模式：setenforce 1
开机时自动切换：
	需要修改/etc/selinux/config配置，重启后生效：
		vim /etc/setlinux/config
		设置：SELINUX=disabled//无需SELinux时，可以设置为禁用   SELINUXTYPE=targeted//默认使用靶标保护方式

day04：
设置，禁止root远程登陆:vim /etc/ssh/sshd_config   设置PermitRootLogin  no
重启远程服务：systemctl restart sshd

禁止滥用su切换权限：
启用wheel组限制：
vim /etc/pam.d/su
# auth required pam_wheel.so use_uid //删除此行注释

查看安全日志/var/log/secure文件
less /var/log/secure

配置sudo授权
使用visudo专用工具
查看自己的sudo权限：sudo -l

开启sudo日志：
visudo：添加Defaults logfile=/var/log/sudo
查看日志：cat /var/log/sudo


grant all on zabbix.* to zabbix@localhost identified by 'zbx@1234';

kali修改镜像源：/etc/apt/sources.list