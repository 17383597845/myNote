### 一、python命令行编程
**注意：这里只展示了可选参数，没有展示位置参数，关于命令解析的详细信息，参考这里的链接：[[optparse和argparse模块]]**

```shell
#怎么解析这些命令(在demo.py执行中怎么拿到命令的选项值)
python demo.py --port 8080 --host 172... --
```
getopt  命令解析 面向过程
optparse 命令解析 面向对象 -h 模块默认的帮助命令，不能从新定义
```python
import optparse
#python 2.py -i 127.0.0.1 -p 8080
#生成一个命令解析的对象
parser= optparse.OptionParser('<Usage> *** this is usage ****') #这里的参数就是一个提示信息
#设置终端命令
parser.add_option('-i',dest='ip',type='string',help='target ip')
parser.add_option('-p',dest='port',type='int',help='target port')
#解析终端的命令
#t = parser.parse_args()
#print(t)   #这里的t里面有一个存放option的‘字典’，和一个存放其他数据的数组
#其他数据：python 2.py -i 127.0.0.1 -p 8080  102 34 234  在这前面的102 34 234就会存放到那个数组中
opts,args = parser.parse_args()
ip = opts.ip
port = opts.port
print(ip,port)
```
结果：
![[Pasted image 20231125090512.png]]
### 二、数据库的连接
浏览器的记录保存在哪里呢？
[[firefox配置文件目录]]
#### 1、python连接sqlite数据库
```python
import sqlite3
#创建连接对象
conn = sqlite3.connect('day02\source\cookies.sqlite')
#游标，拿到游标对象
c = conn.cursor()
sql  = 'select host,name,value from moz_cookies'
#调用游标的方法，执行sql语句
c.execute(sql) #这句话生成的结果集，就保存在c对象中
for t in c:
    print(t)
```





