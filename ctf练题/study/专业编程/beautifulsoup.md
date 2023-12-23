# 使用Beautiful Soup

在前面的文章中已经讲过了正则表达式的使用方法了，但是如果正则表达式出现问题，那么得到的结果就不是我们想要的内容。熟悉前端的朋友肯定知道，对于一个网页来说，都有一定的特殊结构和层级关系，而且很多节点都用id和class来区分。所以可以借助网页的结构和属性来提取数据。

Beautiful Soup是一个可以从HTML或XML中提取数据的Python库。它可以通过你喜欢的转换器快速帮你解析并查找整个HTML文档。

> Beautiful Soup自动将输入文档转为Unicode编码，输出文档转为UTF-8编码。因此你不需要考虑编码方式。
> 
> 除非文档没有指定一个编码方式，这时你只要说明一下原始的编码方式就可以了。

## 准备工作

在开始之前，确保已经安装好Beautiful Soup和lxml。如果没有安装，请参考下面的安装教程。

```undefined
pip install bs4
pip install lxml
```

## 解析器

Beautiful在解析时依赖解析器，它除了支持Python标准库中的HTML解析器外，还支持一些第三方库（比如lxml）。

下面简单的介绍Beautiful Soup 支持的解析器。

|解析器|使用方法|优势|劣势|
|---|---|---|---|
|Python标准库|BeautifulSoup(markup, 'html.parser')|python内置的标准库，执行速度适中|Python3.2.2之前的版本容错能力差|
|lxml HTML解析器|BeautifulSoup(markup, 'lxml')|速度快、文档容错能力强|需要安装C语言库|
|lxml XML解析器|BeautifulSoup(markup 'xml')|速度快，唯一支持XML的解析器|需要安装C语言库|
|html5lib|BeautifulSoup(markup, 'html5lib')|最好的容错性、以浏览器的方式解析文档、生成HTML5格式的文档|速度慢，不依赖外部拓展|

从上面的表格可以看出，lxml解析器可以解析HTML和XML文档，并且速度快，容错能力强，所有推荐使用它。

如果使用lxml，那么在初始化的BeautifulSoup时候，可以把第二个参数设为lxml即可。

具体代码如下所示：

```python
from bs4 import BeautifulSoup

soup = BeautifulSoup('<p>Hello world</p>', 'lxml')
print(soup.p)
```

## 基本用法

下面先用示例来看看Beautiful Soup的基本用法

```python
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""

from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'lxml')
print(soup.prettify())
print(soup.title.string)
```

首先对上面的代码做简单的说明。

眼尖的小伙伴会发现，在声明的 html_doc 变量中是一串HTML代码，但是html标签和body标签并没有闭合。

接着，将html_doc传入BeautifulSoup并指定'lxml'为解析器。这样就成功创建了BeautifulSoup对象，将这个对象赋值给soup。

接下来就可以调用soup的各个方法和属性来解析这串HTML代码了。

首先，调用prettify( )方法。这个方法可以把要解析的字符串以标准的缩进格式输出。这里需要注意的是，输出结果里面包含body、html节点，也就是说对于不标准的HTML字符串，BeautifulSoup可以自动更正格式。

**这一步不是由prettify( )方法做成的**，而是在创建BeautifulSoup时就完成。

然后调用soup.title.string，这实际上是输出HTML中title节点的文本内容。

## 节点选择器

### 选择元素

下面再用一个例子详细说明选择元素的方法：

```python
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""

from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'lxml')
print(soup.title)
print(type(soup.title))
print(soup.title.string)
print(soup.head)
print(soup.p)
print(type(soup.p))
```

运行结果：

```xml
<title>The Dormouse's story</title>
<class 'bs4.element.Tag'>
The Dormouse's story
<head><title>The Dormouse's story</title></head>
<p class="title"><b>The Dormouse's story</b></p>
<class 'bs4.element.Tag'>
```

下面就对上面的代码做简要的描述，这里依然选用之前的HTML代码，首先打印输出title节点的选择结果，你会发现选择结果是**Tag**类型，该类型有很多方法和属性，比如`string`属性，输出title节点的文本内容。

其他代码都是选择节点，并打印节点及其内部的所有内容。

最后要注意的是当有多个节点时，这种选择方式只会匹配到第一个节点，例如：p节点。

### 提取节点信息

从上面的代码我们知道可以使用string属性获取文本的内容。但是有些时候我需要获取节点属性的值，或者节点名。

（1）获取名称

可以利用name属性获取节点的名称。

具体代码如下所示：

```python
soup = BeautifulSoup('<b class="boldest">Extremely bold</b>')
tag = soup.b
print(tag.name)
```

通过运行上面的代码，你会发现成功获取到了b节点的名称。

（2）获取属性

每个节点可能有多个属性，比如id和class等，选择这个节点元素之后，可以调用attrs获取所有的属性。

具体代码示例如下所示：

```python
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="Dormouse"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""

from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'lxml')
print(soup.p.attrs)
print(soup.p.attrs['name'])
```

运行结果

```bash
{'class': ['title'], 'name': 'Dormouse'}
Dormouse
```

从上面的运行结果你会发现属性值返回的是字典类型。

class属性使用列表保存，这是为什么呢？

原因是:class这个属性可以有多个值，所以将其保存在列表中

（4）获取内容

可以利用string属性获取节点元素包含的文本内容，比如要获取第一个p节点的文本。

```python
print(soup.p.string)
```

### 获取子节点

获取子节点也可以理解为嵌套选择，我们知道在一个节点中可能包含其他的节点，BeautifulSoup提供了许多操作和遍历子节点的属性。

比如我们可以获取HTML中的head元素还可以继续获得head元素内部的节点元素。

```python
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="Dormouse"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""

from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'lxml')
print(soup.head.title)
print(soup.head.title.string)
```

### 关联选择

在做选择的时候，有时候不能做到一步就获取到我想要的节点元素，需要选取某一个节点元素，然后以这个节点为基准再选取它的子节点、父节点、兄弟节点等。

（1）选取子节点和子孙节点

选取节点元素之后，想要获取它的直接子节点可以调用contents属性。

具体代码示例如下：

```python
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="Dormouse"><b>The Dormouse's story</b></p>


<p class="story">...</p>
"""

from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'lxml')
print(soup.p.contents)
```

```python
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="Dormouse"><b>The Dormouse's story</b>
</p>


<p class="story">...</p>
"""

from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'lxml')
print(soup.p.contents)
```

相信眼尖的小伙伴看上面两段代码的很容易就看出区别了吧。

第一段代码的`p`节点没有换行，但是第二段代码的`p`节点是存在换行符的。所以当你尝试运行上面代码的时候会发现，直接子节点保存在列表中，并且第二段代码存在换行符。

相同的功能还可以通过调用children属性来获取。

```python
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>


<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""

from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'lxml')
print(soup.p.children)
print(list(soup.p.children))
for i in soup.p.children:
    print(i)
```

上面的代码通过调用children属性来获取选择结果，返回的类型是生成器类型，可以转为list类型或者是for循环将其输出。

如果想要获取子孙的节点的话，可以调用descendants属性来获取输出内容。

具体代码示例如下所示：

```python
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>


<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><span>Elsie</span>Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""

from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'lxml')
print(soup.p.descendants)
for child in soup.p.descendants:
    print(child)
```

此时返回的结果依然还是生成器类型，遍历输出之后，你会发现可以单独输出人名，若子节点内还有子节点也会单独输出。

（2）父节点和祖先节点

如果想要获取某个节点的父节点可以直接调用parent属性。

具体代码示例如下所示：

```python
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>


<p class="story">
Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>
<p>
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a>
</p>
</p>

<p class="story">...</p>
"""

from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'lxml')
print(soup.a.parent)
```

试着运行上面的代码，你会发现，获取的父节点是第一个a节点的直接父节点。而且也不会去访问祖先节点。

如果想要获取所有的祖先节点可以调用parents属性。

具体代码示例如下所示：

```python
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>


<p class="story">
Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>
<p>
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a>
</p>
</p>

<p class="story">...</p>
"""

from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'lxml')
print(soup.a.parents)
for i, parent in enumerate(soup.a.parents):
    print(i, parent)
```

获取祖先节点，依然返回的类型仍然是生成器类型。所以通过循环可以遍历出每一个内容。

试着运行上面的代码，你会发现，输出结果包含了body节点和html节点。

（3） 兄弟节点

上面的两个了例子说明了父节点与子节点的获取方法。那假如我需要获取同级节点该怎么办呢？可以使用**next_sibling、previous_sibling、next_siblings、previous_siblings**这四个属性来获取。

具体代码示例如下：

```python
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>


<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>hello
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""

from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'lxml')
print(soup.a.next_sibling)
print(list(soup.a.next_siblings))
print(soup.a.previous_sibling)
print(list(soup.a.previous_siblings))
```

从上面的代码可以发现，这里调用了4个属性，分别是next_sibling和previous_sibling，这个两个属性分别获取节点的上一个兄弟元素和下一个兄弟元素。

而next_siblings和previous_siblings是获取前面和后面的兄弟节点，返回的类型依然是生成器类型。

## 方法选择器

前面所讲的内容都是通过**属性**来选择的，这种方法非常快，但是如果是较为复杂的选择，那上面的选择方法就可能显得繁琐。因此，Beautiful Soup为我们提供了查询方法，比如:find_all()和find()等。调用它们，传入相应的参数。

- **find_all()**

它的API如下：

find_all(name, attrs, recursive, text, **kwargs)

（1）name

可以根据节点名称来选择参数

具体代码示例如下：

```python
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="Dormouse"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""

from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'lxml')
print(soup.find_all('a'))
print(len(soup.find_all('a')))
```

上面的代码调用了find_all( )方法，传入了name参数，参数值为a，

试着运行上面的代码，我们想要获取的所有a节点，返回结果是列表类型，长度为3。

```python
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="Dormouse"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><span>Elsie</span></a>,
<a href="http://example.com/lacie" class="sister" id="link2"><span>Lacie</span></a> and
<a href="http://example.com/tillie" class="sister" id="link3"><span>Tillie</span></a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""

from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'lxml')
print(soup.find_all('a'))
for a in soup.find_all('a'):
    print(a.find_all('span'))
    print(a.string)
```

将上面的代码做些许修改。

试着运行上面的代码，你会发现可以通过a节点去获取span节点，同样的也可以获取a节点的文本内容。

（2）attrs

除了根据节点名查询的话，同样的也可以通过属性来查询。

具体代码示例如下所示：

```python
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="Dormouse"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><span>Elsie</span></a>,
<a href="http://example.com/lacie" class="sister" id="link2"><span>Lacie</span></a> and
<a href="http://example.com/tillie" class="sister" id="link3"><span>Tillie</span></a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""

from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'lxml')
print(soup.find_all(attrs={'id': 'link1'}))
print(soup.find_all(attrs={'name': 'Dormouse'}))
```

这里查询的时候要传入的参数是attrs参数，参数的类型是字典类型。

对于常用的属性比如class，我们可以直接传入class这个参数，还是上面的文本，具体代码示例如下：

```python
from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'lxml')
print(soup.find_all(class_ = 'sister'))
```

在这里需要注意的是class是Python的保留字，所以在class的后面加上下划线。

同样的，其实`id`属性也可以这样操作，还是上面的文本，具体代码示例如下：

```python
from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'lxml')
print(soup.find_all(id = 'link2'))
```

- **find( )**

除了find_all( )方法，还有find( )方法，前者返回的是多个元素，以列表形式返回，后缀是返回一个元素。

具体代码示例如下：

```python
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="Dormouse"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><span>Elsie</span></a>,
<a href="http://example.com/lacie" class="sister" id="link2"><span>Lacie</span></a> and
<a href="http://example.com/tillie" class="sister" id="link3"><span>Tillie</span></a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""

from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'lxml')
print(soup.find(name='a'))
print(type(soup.find(name='a')))
```

试着运行上面的代码，你会发现，find ( )方法返回第一个a节点的元素，类型是Tag类型。

find( )与find_all( )的使用方法相同。

还有其他方法选择器，在这里做一下简单的介绍。

**find_parents() 和find_parent()**：前者返回所有祖先节点，后者返回直接父节点。

**find_next_siblings()和find_next_sibling()**：前者返回后面的所有兄弟节点，后者返回后面第一个兄弟节点。

**find_previous_siblings和find_previous_sibling()**：前者返回前面的所有兄弟节点，后者返回前面第一个兄弟节点。

## CSS选择器

Beautiful Soup还为我们提供了另一种选择器，就是CSS选择器。熟悉前端开发的小伙伴来说，CSS选择器肯定也不陌生。

使用CSS选择器的时候，需要调用select( ) 方法，将属性值或者是节点名称传入选择器即可。

具体代码示例如下：

```python
html_doc = """
<div class="panel">
    <div class="panel-heading">
        <h4>Hello World</h4>   
    </div>
    
    <div class="panel-body">
        <ul class="list" id="list-1">
           <li class="element">Foo</li>
           <li class="element">Bar</li>
           <li class="element">Jay</li>
        </ul>
        
        <ul class="list list-samll" id="list-2">
           <li class="element">Foo</li>
           <li class="element">Bar</li>
           <li class="element">Jay</li>
        </ul>
    </div>
    </div>
</div>
"""

from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'lxml')
print(soup.select('.panel .panel-heading')) # 获取class为panel-heading的节点
print(soup.select('ul li')) # 获取ul下的li节点
print(soup.select('#list-2 li')) # 获取id为list-2下的li节点
print(soup.select('ul'))    # 获取所有的ul节点
print(type(soup.select('ul')[0]))
```

试着运行上面的代码，查看运行结果之后，很多内容你就明白了。

最后一句输出列表中元素的类型，你会发现依然还是Tag类型。

### 嵌套选择

select( )方法同样支持嵌套选择，例如，会选择所有的ul节点，在对ul节点进行遍历，选择li节点。

与上面的html文本相同，具体代码如下所示：

```python
from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'lxml')
for ul in soup.select('ul'):
    print(ul.select('li'))
```

试着运行上面的结果，输出所有ul节点下的所有li节点组成的列表。

### 获取属性

从上面的几个例子中相信大家应该明白了，所有的节点类型都是Tag类型，所以获取属性依然可以使用以前的方法，仍然是上面的HTML文本，这里尝试获取每个ul节点下的id属性。

具体代码示例如下所示：

```python
from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'lxml')
for ul in soup.select('ul'):
    print(ul['id'])
    print(ul.attrs['id'])
```

从上面的代码可以看出，可以直接向中括号传入属性名，或者通过attrs属性获取属性值。

### 获取文本

要获取文本除了之前所说的string属性，另外，还可以调用get_text()方法。

依然还是前面的html文本具体代码示例如下所示：

```python
from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'lxml')
for li in soup.select('li'):
    print('String:', li.string)
    print('get text:', li.get_text())
```

## 小结

Beautiful Soup到这里基本上就结束了。

在编写爬虫的时候一般使用find_all( )和find( )方法获取指定节点。

如果对css选择器熟悉的话也可以使用select( )方法。

# 实战

## 前言

如果你看到了这里，那么恭喜你完成了很多人不能做到的坚持，因为很少人能够看完上面杂而多的知识。

这次的实战内容，我带来的是爬取B站视频弹幕。

为什么是这个实战内容呢？很简单就是为了迎合我们刚刚学完的Beautiful Soup。

## 准备工作

**工欲善其事，必先利其器**，写爬虫也是同样的道理。

首先，安装好两个必要的库：requests, bs4

```undefined
pip install requests
pip install bs4
```

## 关于B站弹幕限制

以前B站的弹幕很快可以通过抓包获取到，但是现在B站有了限制，就获取不到了，不过不用担心，我拿到以前的API接口依然是可以获取到B站弹幕的。

## 爬取内容

在2020年的最后一天，郭敬明和于正在早期由于抄袭分别向庄羽和琼瑶道歉。当时看了一下还上了微博的热搜。

所以，我今天就默默的打开B站，看看UP主们是怎么样分析这次道歉的以及观众对这次分析发表的言论，所有就来爬取视频的弹幕。

视频链接如下：

`https://www.bilibili.com/video/BV1XK411M7ka?from=search&seid=17596321343783034307`

## 需求分析

### 获取弹幕API接口

![](//upload-images.jianshu.io/upload_images/25386579-16c83e988fa07db3.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image

通过抓包，我们需要的获取内容就是**oid**信息。

我拿了以前的API接口，进行获取弹幕，现在我也将这个接口分享给大家。

```cpp
https://api.bilibili.com/x/v1/dm/list.so?oid=276746872
```

每一个视频的弹幕都可以通过修改oid的值去获取。

将上面的链接输入到浏览器就会可以看到弹幕信息了。

![](//upload-images.jianshu.io/upload_images/25386579-c6c99070fbd48b08.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image

### 爬取弹幕

既然我们在上面所讲的内容是Beautiful Soup，那肯定是通过Beautiful Soup进行数据解析，文本内容保存下来。获取弹幕的写法肯定会有很多种，我在下面就先列出一种。

## 功能实现

同样的，我们需要对上面的链接发起请求。再通过Beautiful Soup获取文本内容，保存至txt文档。

## 具体代码

```python
import requests
from bs4 import BeautifulSoup


class DanMu(object):
    def __init__(self):
        self.headers = {
            'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.66 Safari/537.36'
        }
        self.url = 'https://api.bilibili.com/x/v1/dm/list.so?oid=276746872'

    # 获取网页信息
    def get_html(self):
        response = requests.get(self.url, headers=self.headers)
        html = response.content.decode('utf-8')
        return html

    # 保存弹幕
    def get_info(self):
        html = self.get_html()
        soup = BeautifulSoup(html, 'lxml')
        file = open('弹幕.txt', 'w', encoding='utf-8')
        for d in soup.find_all(name='d'):
            danmu = d.string
            file.write(danmu)
            file.write('\n')



if __name__ == '__main__':
    danmu = DanMu()
    danmu.get_info()
```

![](//upload-images.jianshu.io/upload_images/25386579-3b474badd19825f1.png?imageMogr2/auto-orient/strip|imageView2/2/w/858/format/webp)

image

通过上面的代码只获取到了3000条弹幕，但是弹幕有6000多条，这也算是一种反爬措施。当我写到反爬的时候，会给大家做分析。

  
  
作者：小志Codings  
链接：https://www.jianshu.com/p/424e037c5dd8  
来源：简书  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。