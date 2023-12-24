## Cat
## 题目描述

抓住那只猫

## 思路

打开页面，有个输入框输入域名，输入baidu.com进行测试

![20190828225655.png](https://i.loli.net/2019/08/28/6wrVvTDcmJb5dWt.png)

发现无任何回显，输入127.0.0.1进行测试。

![20190828225750.png](https://i.loli.net/2019/08/28/NXtyEwI3cdqUSVB.png)

发现已经执行成功，执行的是一个ping命令，一开始想的应该是命令拼接执行，但是输入任何有关拼接的字符串都会提示`invalid url`

![20190828225940.png](https://i.loli.net/2019/08/28/JaATjL4Ng6Cc2bh.png)

说明系统对这些字符串都进行了过滤，fuzz测试后发现只有`@`没有被过滤。

![20190828230310.png](https://i.loli.net/2019/08/28/duRCLkM9OBStQUo.png)

且当输入`@`时，会将`@`编码为`%40`

![20190828230426.png](https://i.loli.net/2019/08/28/slSeL9pakKZD67T.png)

尝试在url处输入宽字符，比如`%bf`

![20190828232825.png](https://i.loli.net/2019/08/28/9KpdI3T8ReHChfJ.png)

出现报错信息，是一段html代码，将这些代码复制出来打开。

![20190828232945.png](https://i.loli.net/2019/08/28/4vRZJBoLwVOWS6d.png)

看到了熟悉的django报错页面，看来是将输入的参数传到了后端的django服务中进行解析，而django设置了编码为gbk导致错误编码了宽字符（超过了ascii码范围）。

到这一步后，联系到前面的`@`字符没有被过滤，然后看了大佬们的解题思路（这里太坑了，原题目上其实是由提示的）

![20190828234335.png](https://i.loli.net/2019/08/28/gLfXo6iGKIVHbkt.png)

意思是可以用`@`读取文件内容。

结合django的报错得知了项目的绝对路径为`/opt/api`

![20190828234553.png](https://i.loli.net/2019/08/28/Dua5mChZ81Az4BG.png)

这里还需要懂一些django开发的基本知识，我感觉这道题涉及的面有点广了，django项目下一般有个settings.py文件是设置网站数据库路径（django默认使用的的是sqlites数据库），如果使用的是其它数据库的话settings.py则设置用户名和密码。除此外settings.py还会对项目整体的设置进行定义。

读取settings.py文件，这里需要注意django项目生成时settings.py会存放在以项目目录下再以项目名称命名的文件夹下面。

![20190828235147.png](https://i.loli.net/2019/08/28/laRICAjKb7h58Mm.png)

同上将报错信息已html文件打开，可看到一些敏感信息：

![20190828235523.png](https://i.loli.net/2019/08/28/KBIMhHVtlpFubdx.png)

同样在使用`@`读取数据库信息

![20190828235757.png](https://i.loli.net/2019/08/28/Xm9nwxjvKrtEQeO.png)

在报错信息中搜索CTF得到flag。

![20190828235832.png](https://i.loli.net/2019/08/28/AbSCOlWmfojDeZk.png)

以上就是这道题的解法，我只能说大佬们的思路真野！