![[Pasted image 20231115155121.png]]

```bash
#只备份数据库表，不备份数据库，并恢复
mysqldump -u用户名 -p密码 数据库名 > 存储路径
mysql -u 用户名 -p密码 数据库名 < 存储路径

#备份数据表，同时备份数据库，并恢复
mysqldump -u用户名 -p密码 -B 数据库名 > 存储路径;
mysql -u 用户名 -p密码  < 存储路径;
```