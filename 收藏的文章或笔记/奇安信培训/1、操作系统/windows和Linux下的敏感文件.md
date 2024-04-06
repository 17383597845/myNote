## 0x01、 基本信息

遇到文件包含、任意文件下载等漏洞的时候，可以结合这篇文章，进行下一步攻击。

## 0x02、配置文件

### 查找文件

如果能够命令执行，直接使用查找命令吧。。。

**Linux 相关：**

```bash
# 查找文件
find / -name filename.ext

# 全盘查找含有 flag 的文件
grep flag -r /
```

**Windows 相关：**

```bash
# 全盘查找文件，一定要加一个星号！
for /r c:\ %i in (password.txt*) do @echo %i
for /r c:\ %i in (*.ini) do @echo %i

# 查找 C 盘中包含 password 字样的文件，一定要双引号！
findstr /s /n "password" c:\*

# 查找 pwd.txt 中是否包含 password 字样，一定要双引号！
find /N /I "password" pwd.txt
```

### 常见的配置文件名

```bash
# apache
httpd.conf

# MySQL
my.ini

# 虚拟主机配置
httpd-vhosts.conf

# IIS
metabase.xml
applicationHost.config

# ssh
/etc/ssh/sshd_config

# nginx
/etc/nginx/nginx.conf
/etc/nginx/sites-enabled/default

# PHP
php.ini
# weblogic 读密码
./security/SerializedSystemIni.dat
./config/config.xml
```

### Apache

```bash
# 配置文件路径
/etc/httpd/conf/httpd.conf

# 默认站点路径
/var/www/html/

# ubuntu 下配置文件
/etc/apache2/apache2.conf

# 访问日志和错误日志
/private/var/log/apache2/error_log
/private/var/log/apache2/access_log
```

### IIS

```bash
# 配置文件
web.config
```

### MySQL

```bash
# 配置文件
/etc/my.cnf
/etc/mysql/my.cnf 
```

### phpMyAdmin

```bash
# 配置文件
config.inc.php

# 默认路径
/var/www/phpmyadmin/config.inc.php
```

### XAMPP 建站

相关路径

```bash
# 网站默认路径
xampp\htdocs

# Apache 基本配置
xampp\apache\conf\httpd.conf

# Apache SSL
xampp\apache\conf\ssl.conf

# Apache Perl（仅限插件）
xampp\apache\conf\perl.conf

# Apache Tomcat（仅限插件）
xampp\apache\conf\java.conf

# Apache Python（仅限插件）
xampp\apache\conf\python.conf

# 虚拟主机
xampp/apache/conf/extra/httpd-vhosts.conf

# PHP
xampp\php\php.ini

# 数据库默认路径
xampp\mysql\data

# MySQL
xampp\mysql\bin\my.ini 

# phpMyAdmin
xampp\phpMyAdmin\config.inc.php

# FileZilla FTP 服务器
xampp\FileZilla

# FTP\FileZilla 
Server.xml Mercury 

# 邮件服务器基本配置
xampp\MercuryMail\MERCURY.INI 

# Sendmail
xampp\sendmail\sendmail.ini 
```

默认密码

```bash
# MySQL
User: root   Password:（空）

# FileZilla FTP
User: newuser   Password: wampp
User: anonymous   Password: some@mail.net

# Mercury
Postmaster: postmaster (postmaster@localhost) 
Administrator: Admin (admin@localhost) 
TestUser: newuser   Password: wampp 

# WEBDAV
User: wampp   Password: xampp
```

### phpStudy 建站

还记得几年前用 phpStudy 建站，贼费劲，可能是当时技术太差了，端口占用、数据库管理啥的都很乱，今天（2019年08月02日）在 Windows 上又搭了一次，发现啥问题也没遇到，技术、产品的更新换代真的太快了。

现在还出了个 pro 版本，所以路径也相对的有了变化，本文以 Pro 版为例，如果是普通版，去掉 Pro 即可。

相关路径

```bash
# 根目录
phpstudy\WWW
phpstudy_pro\WWW

# phpMyAdmin
phpstudy_pro\WWW\phpMyAdmin4.8.5

# php：Pro 版本，以扩展的方式来显示插件。
phpstudy_pro\Extensions\php\php7.3.4nts\php.ini
```

## 0x03、敏感文件

### 探针等信息

在使用 `XAMPP/LAMPP/phpStudy/PHPnow` 建站时，可能留下来一些探针页面，可以找到一些可用的信息，比如 `Document_Root` 代表网站根目录，`session.save_path` 存放 `Session` 信息。

```text
1.php
l.php
p.php
u.ph
tz.php
test.php
info.php
ceshi.php
tanzhen.php
phpinfo.php
```

### Windows

```bash
# 查看系统版本
c:\boot.ini

# IIS配置文件
c:\windows\system32\inetsrv\MetaBase.xml 

# 存储Windows系统初次安装的密码
c:\windows\repair\sam 

# MySQL配置
c:\ProgramFiles\mysql\my.ini 

# MySQL root密码
c:\ProgramFiles\mysql\data\mysql\user.MYD 

# php 配置信息
c:\windows\php.ini 
```

### linux

[Basic Linux Privilege Escalation](http://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/)

```bash
# 账户信息
/etc/passwd

# 账户密码文件
/etc/shadow

# Apache2默认配置文件
/usr/local/app/apache2/conf/httpd.conf

# 虚拟网站配置
/usr/local/app/apache2/conf/extra/httpd-vhost.conf

# PHP 配置文件
/usr/local/app/php5/lib/php.ini

# Apache 配置文件
/etc/httpd/conf/httpd.conf

# MySQL 配置文件
/etc/my.conf       
```

## 0x04、常见 CMS

### DeDeCms

```text
/member/templets/menulit.php
/plus/paycenter/alipay/return_url.php
/plus/paycenter/cbpayment/autoreceive.php
/paycenter/nps/config_pay_nps.php
/plus/task/dede-maketimehtml.php
/plus/task/dede-optimize-table.php
/plus/task/dede-upcache.php
```

### WordPress

```text
/wp-admin/includes/file.php
/wp-content/themes/baiaogu-seo/footer.php
```

### Ecshop

```text
/api/cron.php
/wap/goods.php
/temp/compiled/ur_here.lbi.php
/temp/compiled/pages.lbi.php
/temp/compiled/user_transaction.dwt.php
/temp/compiled/history.lbi.php
/temp/compiled/page_footer.lbi.php
/temp/compiled/goods.dwt.php
/temp/compiled/user_clips.dwt.php
/temp/compiled/goods_article.lbi.php
/temp/compiled/comments_list.lbi.php
/temp/compiled/recommend_promotion.lbi.php
/temp/compiled/search.dwt.php
/temp/compiled/category_tree.lbi.php
/temp/compiled/user_passport.dwt.php
/temp/compiled/promotion_info.lbi.php
/temp/compiled/user_menu.lbi.php
/temp/compiled/message.dwt.php
/temp/compiled/admin/pagefooter.htm.php
/temp/compiled/admin/page.htm.php
/temp/compiled/admin/start.htm.php
/temp/compiled/admin/goods_search.htm.php
/temp/compiled/admin/index.htm.php
/temp/compiled/admin/order_list.htm.php
/temp/compiled/admin/menu.htm.php
/temp/compiled/admin/login.htm.php
/temp/compiled/admin/message.htm.php
/temp/compiled/admin/goods_list.htm.php
/temp/compiled/admin/pageheader.htm.php
/temp/compiled/admin/top.htm.php
/temp/compiled/top10.lbi.php
/temp/compiled/member_info.lbi.php
/temp/compiled/bought_goods.lbi.php
/temp/compiled/goods_related.lbi.php
/temp/compiled/page_header.lbi.php
/temp/compiled/goods_script.html.php
/temp/compiled/index.dwt.php
/temp/compiled/goods_fittings.lbi.php
/temp/compiled/myship.dwt.php
/temp/compiled/brands.lbi.php
/temp/compiled/help.lbi.php
/temp/compiled/goods_gallery.lbi.php
/temp/compiled/comments.lbi.php
/temp/compiled/myship.lbi.php
/includes/fckeditor/editor/dialog/fck_spellerpages/spellerpages/server-scripts/spellchecker.php
/includes/modules/cron/auto_manage.php
/includes/modules/cron/ipdel.php
```

### PHP168

```text
/admin/inc/hack/count.php?job=list
/admin/inc/hack/search.php?job=getcode
/admin/inc/ajax/bencandy.php?job=do
/cache/MysqlTime.txt
/PHPcms2008-sp4
```

### CMSeasy

```text
/lib/mods/celive/menu_top.php
/lib/default/ballot_act.php
/lib/default/special_act.php
```