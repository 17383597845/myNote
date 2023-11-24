在python中任何东西都可以看成对象。
数据类型：
list():[]
```python
#列表
ls = [1,2,'a','b',12.1]
print(ls)
#通过索引访问
print(ls[0],ls[1])
```
dict():{}    **key是唯一的**
```python
dic = {'name':'tom','age':18}
print(dic['name'])
```
tuple():元组  **元组中的数据不可变**
```python
t = (1,2,'a','b',12.1);
t[0]=2 #这里会报错，因为元组中的数据不可变
```
	set():集合 {} -------注意要区别于字典
```python
s={'a','b',1,2,3,4,4}
#集合可以去重，输出只有一个4了
print(s)
#注意只写{}默认是一个字典，如何创建一个空集合？
a = set();
```

常用函数：
```python
#列表方法
ls = ['a','b',1,2,3,4]
ls.append('100') #添加一个元素
print(ls)

i = ls.index('b')
```
