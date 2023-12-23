# 让Python自动下载网站所有文件

发布于 2020-11-25 10:09:01

3.4K0

举报

阅读本文大概需要 3 分钟。

最近维基 jie mi 彻底公开了网站的全部文件，我就在想如何使用 Python 将其下载到本地永久保存，于是就有了这篇文章，写爬虫会遇到很多坑，借鉴他人经验，考虑越全面，出错的概率就越小。

![](https://ask.qcloudimg.com/http-save/yehe-2196272/sdek6z0478.jpeg)

假如一个网站，里面有很多链接，有指向文件的，有指向新链接的，新的链接点击进去后，仍然是有指向文件的，有指向新链接的，类似一个文件夹，里面即有文件，又有目录，目录中又有文件和目录。如何从这样的网站上下载所有的文件，并按网站的目录结构来保存这些文件呢？

关键词：Python、下载、正则表达式、递归。

按照**自顶向下**来设计程序，我们整理自己的思路，然后使用 Python 语言来翻译下即可。

思路：由于目录的深度不固定，也不可能穷举，且每一个目录的处理方式和子目录父目录的处理流程都是一样的，因此我们可以使用递归来下载所有文件。

递归代码必须要有退出条件，退出条件要放在前面，本例中的递归退出条件就是：如果是文件就下载，下载完递归函数即完成任务。

##### **总体思路：**

1、给定一个 url，判断是否是文件，如果是文件，下载即可，然后函数结束。

2、如果给定 url 不是文件，那么访问该 url，并获取它下面的所有链接。

3、遍历步骤 2 产生的所有链接，递归的执行步骤 1 和 2，直到程序运行结束。

以上思路，用代码描述如下：

```javascript
import urllib.request
import requests
import re, os


def get_file(url):
    '''
    递归下载网站的文件
    :param url:
    :return:
    '''
    if isFile(url):
        print(url)
        try:
            download(url)
        except:
            pass
    else:
        urls = get_url(url)
        for u in urls:
            get_file(u)
```

复制

前面导入的包在接下来函数中会用到，下面就是逐渐层向下，实现子功能。

##### **判断链接是否指向文件：**

这里总结 url 规律，很容易写出。

```javascript
def isFile(url):
    '''
    判断一个链接是否是文件
    :param url:
    :return:
    '''
    if url.endswith('/'):
        return False
    else:
        return True
```

复制

##### **下载文件：**

下载文件时要从 url 中获取文件应该存储的位置，并使用 os.makedirs 来创建多级目录。然后使用 urllib.request.urlretrieve 来下载文件。

```javascript
def download(url):
    '''
    :param url:文件链接
    :return: 下载文件，自动创建目录
    '''
    full_name = url.split('//')[-1]
    filename = full_name.split('/')[-1]
    dirname = "/".join(full_name.split('/')[:-1])
    if os.path.exists(dirname):
        pass
    else:
        os.makedirs(dirname, exist_ok=True)
    urllib.request.urlretrieve(url, full_name)
```

复制

##### **获取 url 下的所有链接：**

这里要具体网站具体分析，看看如何使用正则表达式获取网页中的链接，这样的正则表达式可以说是再简单不过了。

```javascript
def get_url(base_url):
    '''
    :param base_url:给定一个网址
    :return: 获取给定网址中的所有链接
    '''
    text = ''
    try:
        text = requests.get(base_url).text
    except Exception as e:
        print("error - > ",base_url,e)
        pass
    reg = '<a href="(.*)">.*</a>'
    urls = [base_url + url for url in re.findall(reg, text) if url != '../']
    return urls
```

复制

这里有个小坑，就是网站有个链接是返回上级页面的，url 的后辍是 '../' 这样的链接要去掉，否则递归函数就限入了死循环。

接下来就是写主函数，执行任务了，慢慢等它下载完吧。

```javascript
if __name__ == '__main__':
    get_file('https://file.wikileaks.org/file/')
```

复制

其实，还会存两个问题：

1、假如网站某页有个链接它指向了首页，那么递归程序仍然会限入一个死循环，解决方法就是将访问过的 url 保存在一个列表里（或者其他数据结构），如果接下来要访问的 url 不在此列表中，那么就访问，否则就忽略。

2、如果下载的过程中程序突然报错退出了，由于下载文件较慢，为了节约时间，那么如何让程序从报错处继续运行呢？这里可采用分层递归，一开始时先获取网站的所有一级 url 链接，顺序遍历这些一级 url 链接，执行上述的 get_file(url) ，每访问一次一级 url 就将其索引位置加1（索引位置默认为0，存储在文件中或[数据库](https://cloud.tencent.com/solution/database?from_column=20065&from=20065)中），程序中断后再运行时先读取索引，然后从索引处开始执行即可。另外，每下载成功一个文件，就把对应的 url 也保存在文件中或数据库中，如果一级 url 下的链接已经下载过文件，那么就不需要重新下载了。