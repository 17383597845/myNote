在python中任何东西都可以看成对象。
### 一、数据类型：
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

### 二、常用函数：
#### 1、列表中的函数
```python
#列表方法
ls = ['a','b',1,2,3,4]
ls.append('100') #添加一个元素
print(ls)

i = ls.index('b') #查找列表中的元素，返回索引，如果不存在报错
print(i)

a = ls.pop() #弹出列表中最后一个元素
ls.remove('b') #删除列表中的元素，元素不存在，报错
```
#### 2、len方法
```python
a = 'hello中文'
b = ['a','b','c']
c = ('a','b','c')
print(len(a),len(b),len(c)) #7 3 3   一个中文还是一个长度
print(a[-2]) #中

d = {'name':'tom','age':18}
print(len(d)) #获取键值对的数量
```
### 三、语法特点
####  1、缩进
python中没有{}，都是以缩进来表示
#### 2、关于in（判断元素是否在序列中）
```python
a = 'hello'
b = ['a','b','c']
c = ('a','b','c')
#判断元素是否在一个序列中
print('h' in a)
print('a' in b)
print('b' in c)
#判断不再列表中  and or noe ---->python中的逻辑
print('h' not in a)


d = {'name':'tom','age':18}
print('name' : d) #注意只能判断key，不能判断value
```
