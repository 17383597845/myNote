# 靶机渗透测试（covfefe）

  

**靶机渗透测试（covfefe）：**

**Vulnhub靶机 covfefe**  
靶机：修改靶机的网络配置为桥接模式。  
攻击机：Kali虚拟机，同样使用桥接模式，即可访问靶机。  
靶机难度：（==Intermediate==）  
目标：==Covfefe is my Debian 9 based B2R VM, originally created as a CTF for SecTalks_BNE. It has three flags…==  
flag进度：==3/3==

---

---

**渗透流程：**

#### 1. 探测靶机ip地址（netdiscover -i eth0 -r 网关）

> 命令：netdiscover -i eth0 -r 192.168.10.0

靶机ip为： 192.168.10.70  
![在这里插入图片描述](https://i2.wp.com/img-blog.csdnimg.cn/20201113150838168.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjcwMDA0Mg==,size_16,color_FFFFFF,t_70#pic_center)

---

---

#### 2. nmap进行靶机端口服务扫描

> 命令： nmap -sS -Pn -A -p- -n 192.168.10.70

靶机开放22/ssh、 80/http、 31337/http等端口服务

![在这里插入图片描述](https://i2.wp.com/img-blog.csdnimg.cn/20201113151108437.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjcwMDA0Mg==,size_16,color_FFFFFF,t_70#pic_center)

---

---

#### 3. 根据端口服务进行渗透（80/http端口服务）

访问80端口页面，发现为nginx的配置页面，查看一波源代码，无果！！！  
使用御剑，dirb工具进行目录扫描，仍然没有任何有价值的信息！！！

![在这里插入图片描述](https://i2.wp.com/img-blog.csdnimg.cn/20201113151201517.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjcwMDA0Mg==,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://i2.wp.com/img-blog.csdnimg.cn/20201113151209790.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjcwMDA0Mg==,size_16,color_FFFFFF,t_70#pic_center)

---

---

#### 4. 端口渗透（31337/http端口服务）

**4.1. 信息收集~31337端口**

访问31337端口网页，发现一段提示Url信息，于是使用dirb进行目录扫描；  
可以看到扫描出了robots.txt 、 .ssh等敏感目录；

![在这里插入图片描述](https://i2.wp.com/img-blog.csdnimg.cn/20201113151525722.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjcwMDA0Mg==,size_16,color_FFFFFF,t_70#pic_center)  
![在这里插入图片描述](https://i2.wp.com/img-blog.csdnimg.cn/20201113151533480.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjcwMDA0Mg==,size_16,color_FFFFFF,t_70#pic_center)

访问robots.txt，在/taxes目录中得到第一项flag1:

> flag1{make_america_great_again}

![在这里插入图片描述](https://i2.wp.com/img-blog.csdnimg.cn/20201113151541458.PNG#pic_center)

![在这里插入图片描述](https://i2.wp.com/img-blog.csdnimg.cn/20201113151547335.PNG#pic_center)  
继而访问另外两个目录，下载得到两个文件（未利用）  
![在这里插入图片描述](https://i2.wp.com/img-blog.csdnimg.cn/20201113151552932.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjcwMDA0Mg==,size_16,color_FFFFFF,t_70#pic_center)

---

**4.2. 访问.ssh目录**

访问.ssh目录，可以看到一项信息，内容大致是ssh登陆的公钥与私钥信息；

![在这里插入图片描述](https://i2.wp.com/img-blog.csdnimg.cn/20201113151559408.PNG#pic_center)

继续访问，下载三项ssh密钥文件：

![在这里插入图片描述](https://i2.wp.com/img-blog.csdnimg.cn/20201113151607380.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjcwMDA0Mg==,size_16,color_FFFFFF,t_70#pic_center)  
在id_rsa.pub中保存有ssh公钥信息，可以看到用户名==simon==（猜测为ssh登陆的用户名）

![在这里插入图片描述](https://i2.wp.com/img-blog.csdnimg.cn/20201113151613247.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjcwMDA0Mg==,size_16,color_FFFFFF,t_70#pic_center)

---

**4.3. 爆破ssh私钥密码**

我们可以使用ssh2john命令将私钥文件生成为一段Hash，然后使用john工具进行密钥密码的爆破:

> 命令：  
> ssh2john.py id_rsa > key

> john key

成功爆破出私钥密码为==starwars==

![在这里插入图片描述](https://i2.wp.com/img-blog.csdnimg.cn/20201113152408637.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjcwMDA0Mg==,size_16,color_FFFFFF,t_70#pic_center)

---

如果kali中没有发现命令，可以使用locate ssh2john命令找到存放地址，然后ln -s 执行一下就可以啦！(==添加软链接==)

![在这里插入图片描述](https://i2.wp.com/img-blog.csdnimg.cn/20201113152427717.PNG#pic_center)

---

**4.4. ssh密钥登陆**

使用命令进行ssh密钥登陆：

> ssh -i id_rsa simon@192.168.10.70

不知道为什么一直提示报错，貌似是因为靶机ip没有存档在hosts文件中（？？不懂，希望可以得到大佬们的指教）

![在这里插入图片描述](https://i2.wp.com/img-blog.csdnimg.cn/20201113152434420.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjcwMDA0Mg==,size_16,color_FFFFFF,t_70#pic_center)

---

---

#### 5. FinalShell登陆

**5.1. ssh私钥登陆**

使用finalShell程序进行ssh密钥登陆，导入密钥id_rsa文件，然后输入密码starwars，进行ssh远程连接；

连接成功，得到simon低权限用户！！！

![在这里插入图片描述](https://i2.wp.com/img-blog.csdnimg.cn/20201113154721723.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjcwMDA0Mg==,size_16,color_FFFFFF,t_70#pic_center)

cd到root目录下，ls发现存在flag.txt（没有权限打开），然后发现旁边还存在一项read_message.c文件；

![在这里插入图片描述](https://i2.wp.com/img-blog.csdnimg.cn/20201113154727830.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjcwMDA0Mg==,size_16,color_FFFFFF,t_70#pic_center)

cat查看read_message.c文件一波，发现flag2:

> flag2{use_the_source_luke}

![在这里插入图片描述](https://i2.wp.com/img-blog.csdnimg.cn/20201113154735436.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjcwMDA0Mg==,size_16,color_FFFFFF,t_70#pic_center)

---

**5.2. 溢出漏洞提权（缓冲区溢出）**

使用find命令查找可以SUID执行的文件，看到了熟悉的read_message,继续审计read_message.c中的代码，发现为代码溢出漏洞！！！

![在这里插入图片描述](https://i2.wp.com/img-blog.csdnimg.cn/20201113154741705.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjcwMDA0Mg==,size_16,color_FFFFFF,t_70#pic_center)

代码中只匹配前五个字符，并且前五个字符为Simon用户名。超过20个字符就会执行溢出漏洞；

其缺陷就在于执行字符串判断操作时未将目的字符串大小进行判断导致了缓冲区溢出攻击的发生，此例使用了缓冲区溢出攻击将具有root执行权限的read_message程序调用了外部shell，从而获得了root权限。

![在这里插入图片描述](https://i2.wp.com/img-blog.csdnimg.cn/20201113154747830.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjcwMDA0Mg==,size_16,color_FFFFFF,t_70#pic_center)  
该文件为编译后的文件，执行后，发现提示信息：

> 执行suid文件：  
> /usr/local/bin/read_message

然后输入name：

> Simonaaaaaaaaaaaaaaa/bin/sh

成功获取root权限！！！  
得到flag3：

> flag3{das_bof_meister}

![在这里插入图片描述](https://i2.wp.com/img-blog.csdnimg.cn/20201113154754483.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjcwMDA0Mg==,size_16,color_FFFFFF,t_70#pic_center)

实验结束！

---

---

#### 实验总结：

---

**代码原理:**

当我们输入一个字符串时, 它将与Simon 一起检查字符串的前5字符。如果匹配, 它将会运行程序/usr/local/bin/read_message。

现在它被分配为20字节。所以我们溢出堆栈进入超过20个字节的数据就可执行溢出漏洞。

使用前5个字符是 “Simon”, 然后添加15 个任意字符, 最后加入 “/bin/sh” 在第21字节，溢出提权。

---

本次实验中，在前期信息收集时，robots.txt敏感文件以及得到ssh公钥私钥的文件是实验的关键，在得到ssh私钥时，使用john爆破出私钥密码，并使用finalshell远程登陆ssh’

在提权中，本次漏洞的利用方法为超过缓冲区20字符限制。