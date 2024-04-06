#### 1、grant
mysql权限表：
mysql中权限的全部信息都保存到权限表中，包括：user表，db表等等。mysql权限表通过用户标识（**用户名+主机名**）来做权限验证。
	user表：最重要的权限表，记录了允许连接到服务器的账户信息，权限是全局的
	db表：储存了用户对某个数据库的操作权限
	host表：储存了某个主机对数据库的操作权限
	tables_pri表：用于对表设置操作权限
	columns_pri表：用来对表的某一列设置权限
	procs_pri表：存储过程和函数有关的权限
	
用户权限级别：
	服务管理类：super（管理员权限），usage（只有连接数据库的权限）
权限管理命令：
```sql
-- 将所有数据库所有表的全部权限，包括grant和revoke权限给root@%用户标识，并给这个用户标识设置密码
-- 'root'@'%' 主机名 %：表示允许任意地址登录  localhost：只允许本地登录  具体ip：只允许这个ip的主机登录
grant all privileges on *.* to 'root'@'%' identified by 'Admin@123' with grant option;

```
如何确定用户标识是否存在，mysql.user:
```sql
select user,host from mysql.user;
```
![[Pasted image 20231115144227.png]]
案例：
```sql
select user,host from mysql.user;
grant select,insert on school.* to 'user1'@'localhost' identified by 'Admin@123' with grant option;
flush privileges;
show grants for 'user1'@'localhost';
-- 赋予user1用户school数据库下student表中sno和snmae的update权限
grant update(sno,sname) on scholl.student to 'user1'@'localhost';
-- user2账户允许任意地址登录，赋予user2账户school数据库下course表中cno和cname的select权限，并且使得该用户拥有将自己的权限赋予给其他人的能力，查看权限、查看用户
grant select(cno,cname) on scholl.course to 'user2'@'%' identified 'Admin@123' with grant option;
```
#### 2、revoke
```sql
-- 收回user2@%用户school数据库下course表的select权限
revoke select on school.course from 'user2'@'%';
flush privileges;
-- 收回grant权限
revoke grant option on school.course from 'user2'@'%';
```


commit
rollback