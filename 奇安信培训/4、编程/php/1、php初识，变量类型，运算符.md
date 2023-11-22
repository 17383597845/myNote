### 一、ide配置
vscode：
	open php —— 打开浏览器
				setting——》搜索open php插件——》Open-PHP-HTML-JS-in-browser:配置网站根目录
	php debug ——debug用的
	php intelephense  ——用于自动补全
php特点：
	服务器端语言
	嵌入到html语言中
	脚本语言
	php不强制区分大小写，官方推荐小写
配置var_dump输出格式：
在php.ini中添加以下行（添加xdebug扩展）
![[Pasted image 20231120142153.png]]
php很多功能都是依赖扩展的，扩展在windows中存放在ext文件夹中，在这个文件夹中有的扩展在php.ini中正确配置后就可以使用了。
### 二、php数据类型
#### 1、变量的声明：
$变量名
#### 2、数据类型
四种标量类型：
	boolean
	integer
	float
	string
	注意：这四种类型可以运算，php会对混合的表达式进行隐式的类型转换；不会报错可能会导致出现你不想要的结果（也许会warning）
复合数据类型：
	array
	object
两种特殊类型：
	resource:操作文件，操作数据库时
	NULL:定义变量，不赋值，它就是空值；echo空值不会有回显，这时可以使用var_dump()函数进行输出
在类型强制转换时，可以使用函数intval(),floatval()，strval()，也可以使用（string），（int），（float）

```php
<?php 
$string1 = ""
$interger1 = 123

$Float1 = 1.1
$Float2 = 123e2
$Float3=123e-3

$Boolean1=true/false

class car{
	var $color;
	function __construct($color="green"){
		$this->color = $color;
	}
	function showColor(){
		return $this->color;
	}
}                                                                                                  
?>
```
#### 3、可变变量（动态变量）
如果一个变量保存的值刚好时另一个变量的名字，那么可以直接通过访问一个变量得到另一个变量的值(注意这个值指的是字符串)；在变量前面再多加一个$符号
```php
<?php
$a='b';
$b='c';
echo $$a；

$hi = 'hello';
$$hi = 'world';
echo "$hi $hello"."<br>";
echo "$hi ${$hi}";
```

**注意：**
.运算符表示连接（不一定是字符串之间的连接，整型等均可以连接；不是+号，+号会被解析为运算）
''和""的区别：
	双引号可以解析变量，单引号是不能解析变量的;如果使用双引号还不想让他解析变量可以使用转义字符\
```php
<?php
$hi = 'hello';
$$hi = 'world';
echo '$hi $hello'.'<br>';
echo "$hi $hello"."<br>";
echo "\$hi \$hello"
```

unset:对变量使用后，这个变量的值变成空值。
#### 4、常量
定义方式：
	define('常量名'，'常量值')；这里需要引号
	const 常量名 = 值；这里不需要引号
	两种方式的常量在使用的时候都不需要加引号，同时常量和普通变量不同的是，它不需要$
内置常量：**双下划线开头的**
```PHP
__FILE__ //目录加脚本名
__DIR__  //目录
```
#### 5、数组
数组创建：
```php
//索引数组：key值全部为数字
$Array1 = array("1","2","3");
$Array3=['a','b','c'];#使用方括号创建索引数组
var_dump($Array1);
print_r($Array2);

//关联数组:类似于map
$Array2=array('001'=>'a','002'=>'b','003'=>'c');
$Array4=['001'=>'a','002'=>'b','003'=>'c'];#使用方括号创建关联数组
var_dump($Array2);
//第三种创建数组的方式
$Array5[]='a';
$Array5['001']='a';
$Array5[100]='b';
$Array5['002']='b';
$Array5[]='c';
var_dump($Array5);
结果：
**array** _(size=5)_
  0 => string 'a' _(length=1)_
  '001' => string 'a' _(length=1)_
  100 => string 'b' _(length=1)_
  '002' => string 'b' _(length=1)_
  101 => string 'c' _(length=1)_

//二维数组以下面的例子类推
$Array6 = ['1'=>[],'2'=>[]];
/*
内置数组：
$_GET:处理get请求
$_POST
$_REQUEST
$_SERVER
$GLOBALS
$_FILES
$_COOKIE
$_SESSION
*/

```
数组遍历：
```php
//while  配合list函数和each函数遍历数组
$a1 = ['tom',18,170];
while(list($key,$value) = each($a1)){
	echo $key,$value,'<br>';
}
//for  适合遍历索引数组 要知道数组长度,count函数可以返回数组的长度 
for($i=0;i<count($a1);i++){
	echo $a1[$i].'<br>';
}
//foreach
//只遍历value
foreach (a1 as $value){

}
//即遍历key值又遍历value值
foreach（a1 as $key=>$value）{

}
//list()可以对数组进行解析
list($name,$age,$heigt) = $a1;#将数组中的元素取到变量中


//foreach遍历二维数组
$array=[
	[1,2],
	[3,4],
];
foreach($array as list($a,$b)){
	echo 'a = ' . $a .'b = ' . $b;
}

//each() 专业处理数组
error_reporting(0);#报错信息不显示，在生产环境中报错信息不能给客户显示
	$a2 = array('001'=>'a','002'=?'b','003'=>'c');
var_dump(each($c));
```
数组方法：
```php
in_array()
array_pop():弹出最后一个值，出栈
array_push() ：压入一个值，入栈
array_slice()：从数组中截取一段
array_keys()：拿到数组所有的或部分的key值
sort()：数组排序
```
### 三、运算符
```
其他的和java相同，需要注意的是==和===
==只是比较值
===不仅比较值而且比较类型
```
![[Pasted image 20231120102138.png]]
自增自减：
```php
<?php
$a=1;
$a++;
++$a;
```

### 时间格式化：
```
date('H:i:s Y-m-d',time());
$hour = date('H');
echo $hour;
```

### 四、分支语句,循环语句
#### 1、swith case
```
swith($a){
	case '100':
		echo 'a';
		break;
	case '100s':
		echo 'b';
		break;
	default:
		echo 'c';
}
```
#### 2、while循环
```php
$out=0;
while($out <= 10){


$out++;
}
```
#### 3、for循环
这个和java也是一样的
