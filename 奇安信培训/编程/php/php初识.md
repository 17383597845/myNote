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
$Array1 = array("1","2","3")

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
如果一个变量保存的值刚好时另一个变量的名字，那么可以直接通过访问一个变量得到另一个变量的值；在变量前面再多加一个$符号
