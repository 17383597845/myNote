# SQL注入常规Fuzz全记录
[[fuzz字典]]
### **前言**

**本篇文章是在做ctf bugku的一道sql 盲注的题（题目地址:注入题目）中运用了fuzz的思路，完整记录整个fuzz的过程，给师傅们当点心，方便大家加深对web sql注入 fuzz的理解。**

![](https://ask.qcloudimg.com/http-save/yehe-1268449/rrzu3pndd7.jpeg)

### **进入主题**

1.访问题目，是个典型的登录框

![](https://ask.qcloudimg.com/http-save/yehe-1268449/uuauh9tggh.jpeg)

2.尝试输入admin/123456,提示密码错误，因此可以确定存在用户admin，这里可能会有师傅要爆破了，但这里题目要求sql注入，我们就按照预期解来吧。

![](https://ask.qcloudimg.com/http-save/yehe-1268449/1wijqep8cl.jpeg)

3.我自己写了个简单的fuzz burp插件，先将登陆请求包发送到插件扫描，可以看到是存在盲注的，payload的形式为:

![](https://ask.qcloudimg.com/http-save/yehe-1268449/gcctwgeccv.jpeg)

4.fuzz

(1)从payload的形式可以猜测题目应该是过滤了注释符(--+和#)

(2)fuzz一遍特殊字符，看看过滤了什么

当存在过滤的字符时，响应包是这样的

![](https://ask.qcloudimg.com/http-save/yehe-1268449/hl1o7wbjun.jpeg)

因此可以作为fuzz的判断(当然有些waf是静默waf,就是照样接收你的数据但自己做了处理，返回正常页面，这种fuzz的判断有时候就需要设计下你的payload，这种在以后的文章继续讨论)

fuzz特殊字符，结果如下，可以看到长度为370的是被wa了的，过滤了相当多的字符，特别是内联注释 注释符 空格 %0a %0b %0d %a0这些比较常用的绕过关键组件，尤其注意过滤了逗号

![](https://ask.qcloudimg.com/http-save/yehe-1268449/wp39rtg4pv.jpeg)

(3)fuzz一遍关键字，过滤了and or order union for 等等,因此取数据常用的mid( xx from xx for xx)就不能用了，之前逗号也被过滤了也就不能用mid(xx,1,1)。

![](https://ask.qcloudimg.com/http-save/yehe-1268449/zy1ron5zkk.jpeg)

(4)fuzz函数名和操作符(由于插件的扫描结果没有过滤sleep,直觉上是没有对函数做过滤)

![](https://ask.qcloudimg.com/http-save/yehe-1268449/z5gy5hj1sp.jpeg)

![](https://ask.qcloudimg.com/http-save/yehe-1268449/zx2qad5zrx.jpeg)

不出意外，确实是只有包含关键字or and等的函数被wa了，其他基本没有，其实这里我们也可以联想到跑表经常要用的information_schema表是存在or关键字的，因此后面构造语句的时候也就不能直接用information_schema

(5)尝试用时间盲注跑数据

```javascript
if(1=1,sleep(5),0)
```

复制

由于不能用逗号需要变为

```javascript
CASE WHEN (1=1) THEN (sleep(5)) ELSE (2) END
```

复制

但空格也被过滤了，需要用括号代替空格（/_!_/ 空格 tab %a0 %0d%0a均被过滤了）

```javascript
(CASE WHEN(1=1)THEN(sleep(1))ELSE(1)END);
```

复制

最后本地测试的时候发现case when之间不能用括号，做一下字符fuzz，从%00到%ff

![](https://ask.qcloudimg.com/http-save/yehe-1268449/xqnvtjd10c.jpeg)

可以看到结果是确实不行，并不能产生延时(有的直接被wa,有的没被wa但sql语句无法生效)，因此基本可以确认不能用时间盲注跑数据，于是我们只能考虑布尔盲注

(6)尝试布尔盲注

由于无法使用if或者case/when,只能使用题目自带的bool盲注做逻辑判断(=) 比如我们一开始就注意到存在admin用户，改造插件的payload: '+sleep(5)+' (注意把+换为%2b)

```javascript
admin'+1+' (false,注意把+换为%2b)
admin'+0+' (true,注意把+换为%2b)
```

复制

```javascript
select * from user where name='admin'+1+'' and passwd='123456';(为false) ==>提示用户名错误
select * from user where name='admin'+0+'' and passwd='123456';(为true) ==>提示密码错误
```

复制

这里是mysql的一个特性，可能有不明白的师傅，可以做下实验

```javascript
select 'admin'='admin'+0 union select 'admin'='admin'+1;
```

复制

前者为1后者为0，先对右边的等式做运算，发生强制转换，结果为数字，然后再和左边的admin字符做比较，又发生了强制转换，因此出现1和0的区别。

这样子我们就解决了布尔盲注的判断了

(7)解决下跑数据的问题

这里不能用mid(xxx,1,1)也不能用mid(xxx from 1 for 1),但查手册发现可以使用mid(xxx from 1),表示从第一位开始取剩下的所有字符,取ascii函数的时候会发生截断，因此利用ascii(mid(xxx from 1))可以取第一位的ascii码，ascii(mid(xxx from 2))可以取第二位的ascii，依次类推

![](https://ask.qcloudimg.com/http-save/yehe-1268449/6yzcarh8b0.jpeg)

(8)burp跑数据

a.判断passwd字段的长度: 跑出长度为32

(这里可以猜字段，根据post请求包中的passwd猜测[数据库](https://cloud.tencent.com/solution/database?from_column=20065&from=20065)的字段应该也是passwd，这样就可以不用去跑information_schema，直接在登陆查询语句中获取passwd)

```javascript
admin'-(length(passwd)=48)-'
```

复制

![](https://ask.qcloudimg.com/http-save/yehe-1268449/yml9p7xq60.jpeg)

b.跑第一位

这里的payload我用的不是上面的，从最后面开始倒着取数据然后再reverse一下，那时候做题没转过弯，其实都一样的,用下面的payload的好处是假如ascii不支持截断的情况下是不会报错的(用于其他数据库的时候)

```javascript
=admin'-(ascii(mid(REVERSE(MID((passwd)from(-1)))from(-1)))=48)-'
```

复制

用这一个也可以的

```javascript
=admin'-(ascii(mid(passwd)from(1))=48)-'
```

复制

![](https://ask.qcloudimg.com/http-save/yehe-1268449/rifxwqv67c.jpeg)

重复上述操作修改偏移，即可获取32位密码005b81fd960f61505237dbb7a3202910解码得到admin123，登陆即可获取flag,到这里解题过程结束。

### **总结**

1.上述用到的fuzz字典均可在sqlmap的字典以及mysql官方手册中收集

2.这里仅仅是常规的fuzz，但大多数fuzz其实都是相通的，主要是fuzz的判断，fuzz的位置,fuzz payload的构造技巧等等

3.欢迎各位大师傅一起交流讨论！