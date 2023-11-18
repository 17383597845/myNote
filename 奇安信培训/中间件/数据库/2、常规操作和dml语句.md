mysql登录：
	mysql -u用户名 -p密码 -P端口 -H主机名
information_schema数据库：
	schemata                所有数据库的名字
		schema_name  数据库名称
	tables
		table_schema  表所属数据库的名字
		table_name  表的名字
	columns
		table_shema  字段所属数据库的名字
		table_name   字段所属表的名称
		column_name   字段的名字
四种注释方式：
		```
		#：单行注释
		-- ：单行注释，注意这里--后有一个空格
		/* */：多行注释
		/*！ */：内联注释
		```
	关于/*！ */的使用（绕过有用）：[[内联注释的使用]]

sql语言功能：
![[Pasted image 20231113142746.png]]
支持的字符集：
![[Pasted image 20231113143336.png]]
gbk一个汉字占两个字节，utf8一个汉字占三个字节

修改数据库的字符集：
```sql
show character set; #查看数据库支持的字符集
alter database mysql character set gbk;#修改数据库的字符集，注意在修改之前的表的字符集不会修改，只有后面的新表才生效；也就是说，他是从当前开始生效。
show create database mysql;#查看数据库的所有属性
```
在创建数据库的同时指定字符集：
```sql
create database qax default character set utf8;
show create database qax;
#上面语句的简写
create database qax character set utf8;
create database qax charset utf8;#最简写
```

创建表：
	三要素：字段名、数据类型、约束条件
约束条件：
	表级约束：当一张表中吗多个字段为主键时就要使用表级约束
	实体完整性：
		 主键约束：primary key（字段名1，字段名2）
		 唯一性约束：unique（字段名），可以插入空值，但是空值也只能插入一次
		自增约束：auto_increment
	参照完整性：
		外键约束：foreign key(子表中的外键字段名) references 主表（父表字段名）  这里的父表字段必须是有唯一性约束的
	域完整性：
		默认值约束
		非空约束
		检查约束：check ——————mysql不支持
		
```sql
CREATE TABLE student(
sno CHAR(5) PRIMARY KEY,
sname VARCHAR(20) NOT NULL,
birth DATE,
sage INT,
ssex CHAR(5) DEFAULT '男'
)
/*
注意constraint hh是在给外键取别名
*/
CREATE TABLE score(
sno CHAR(5),
cname VARCHAR(20) NOT null,
score int,
CONSTRAINT hh FOREIGN KEY(sno) REFERENCES student(sno)
)

#设置联合主键
CREATE TABLE sc(
sno CHAR(5),  
cno CHAR(5),
sc INT NOT NULL,
PRIMARY KEY(sno,cno)
)
```

复制表：
```sql
create 新表 like 源表 #复制的是表结构，不复制数据
create table 新表名 select * form 原表 #复制表中的数据，结构可能有丢失
create table 新表名 select * form 原表 where 1=1
```
修改表结构：
![[Pasted image 20231113163446.png]]

![[Pasted image 20231113164534.png]]

练习：
![[Pasted image 20231113165107.png]]


```sql
create table 新表名 like 源表名      #只复制表结构，不复制表数据

create table 新表名 select * from 源表名   #复制表中的数据，但是结构（约束条件）会改变或丢失
或：
create table 新表名 select * from 源表名 where 1=1

```




































修改字段相关信息:  
删除字段: alter table 表名 drop 字段名  
  
添加新字段: alter table 表名 add 新字段名 新数据类型 [新约束条件] [ first |       after 旧字段名]  
  
修改字段名(或者数据类型) : alter table 表名 change 旧字段名 新字段名 新数据类型  
  
修改数据类型: alter table 表名 modify 字段名 新数据类型   
  
修改约束条件:  
添加约束条件: alter table 表名 add [constraint约束名] 约束类型 (字段名)  
eg:alter table scoure add foreign key (sno) references student(sno);  
删除约束条件: alter table 表名 drop primary key  
alter table 表名 drop foreign key 约束名  #要删除外键要有外键名称  
show create table 表名  #用来查看外键名称  
  
修改表名:  
rename table 旧表名 to 新表名  
alter table 旧表名 rename 新表名