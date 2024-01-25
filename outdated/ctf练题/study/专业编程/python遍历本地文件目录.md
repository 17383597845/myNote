使用Python遍历文件非常简单，核心函数是使用 os.walk(folder) 函数，folder就是你想要搜索的文件夹的最顶层。

例如，我们有一个文件夹叫base：

```text
base/
├── fileA.txt
├── fileA2.xls
├── fileA3.xls
├── fileA4.pdf
├── sub1
│   ├── fileB.txt
│   ├── fileB2.xls
│   └── fileB3.pdf
└── sub2
    ├── fileB.txt
    ├── fileC2.xls
    └── fileC3.pdf
```

base/ 文件夹下面有4个文件（"fileA.txt", "fileA2.xls", "fileA3.xls", "fileA4.pdf"）和两个文件夹（sub1/ 和 sub2/）。sub1/ 文件夹里面有3个文件（"fileB.txt", "fileB2.xls", "fileB3.pdf"），sub2/ 文件夹你们也有3个文件（"fileB.txt", "fileC2.xls", "fileC3.pdf"）。注意，文件 "fileB.txt" 出现了两次，但是在不同的文件夹中。

现在我们想拿到这里面所有的文件，如何实现？

迭代生成 base 文件夹下所有的文件的名称，可以使用：

```python
def findAllFile(base):
    for root, ds, fs in os.walk(base):
        for f in fs:
            yield f
```

我们可以在 base/ 文件夹同级的位置写个脚本测试一下：

```python
# file: all_file.py

import os

def findAllFile(base):
    for root, ds, fs in os.walk(base):
        for f in fs:
            yield f

def main():
    base = './base/'
    for i in findAllFile(base):
        print(i)

if __name__ == '__main__':
    main()
```

运行效果：

```text
$ python all_file.py
fileA.txt
fileA2.xls
fileA3.xls
fileA4.pdf
fileB3.pdf
fileB.txt
fileB2.xls
fileC.txt
fileC3.pdf
fileC2.xls
```

到这一步，其实并没有什么用处，因为这里没有文件对应的路径信息。也就是说，如果你想打开文件的话，程序是找不到的。

所以就需要稍微改动一下：

```python
def findAllFile(base):
    for root, ds, fs in os.walk(base):
        for f in fs:
            fullname = os.path.join(root, f)
            yield fullname
```

运行效果：

```text
$ python all_file.py
./base/fileA.txt
./base/fileA2.xls
./base/fileA3.xls
./base/fileA4.pdf
./base/sub1/fileB3.pdf
./base/sub1/fileB.txt
./base/sub1/fileB2.xls
./base/sub2/fileC.txt
./base/sub2/fileC3.pdf
./base/sub2/fileC2.xls
```

接下来你就可以随心所欲的在脚本里操作文件啦。

比如，我想找出所有的 Excel 文件（后缀名 .xls）：

```python
# file: all_file.py

import os

def findAllFile(base):
    for root, ds, fs in os.walk(base):
        for f in fs:
            if f.endswith('.xls'):
                fullname = os.path.join(root, f)
                yield fullname

def main():
    base = './base/'
    for i in findAllFile(base):
        print(i)

if __name__ == '__main__':
    main()
```

运行结果：

```text
$ python all_file.py
./base/fileA2.xls
./base/fileA3.xls
./base/sub1/fileB2.xls
./base/sub2/fileC2.xls
```

如果你想要更加复杂的文件名匹配方案，可能使用正则表达式会比较方便，导入 re 模块即可。文件名在程序可以当成字符串对象来处理，很方便。

比如，有一个需求，需要找出 base/ 文件夹下所有文件名中包含数字的文件：

```python
# file: all_file.py

import os,re

def findAllFile(base):
    for root, ds, fs in os.walk(base):
        for f in fs:
            if re.match(r'.*\d.*', f):
                fullname = os.path.join(root, f)
                yield fullname

def main():
    base = './base/'
    for i in findAllFile(base):
        print(i)

if __name__ == '__main__':
    main()
```

执行效果：

```text
$ python all_file.py
./base/fileA2.xls
./base/fileA3.xls
./base/fileA4.pdf
./base/sub1/fileB3.pdf
./base/sub1/fileB2.xls
./base/sub2/fileC3.pdf
./base/sub2/fileC2.xls
```

—— 完 2019-12-19 ——

一年半后续更：

很多人对 for root, ds, fs in os.walk(base): 这里面的内容很好奇，我解释一下。

首先，os.walk(base) 返回一个可迭代的对象，所以能用for循环遍历，每次得到的是一个元组（tuple），这个tuple包含三个元素：第一个是root，表示“走”到了哪个文件夹位置，第二个是在这个位置里面能看到哪些文件夹（是个列表），第三个是在这个位置里面能看到哪些文件（也是列表）。

其实这个是很直觉的：当打开任意一个文件夹（root）的时候，里面能看到的东西无非就两类：文件夹（ds）和文件（fs）。所以，我们用 os.path.join(root, f) 就能得到完整的文件地址，ds这个变量，这里是不需要用到的，除非你对搜索文件夹名称感兴趣。