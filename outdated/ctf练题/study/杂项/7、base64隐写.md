# [密码学 base64stego 还不懂base64的隐写？详解12行代码带你领略](https://www.cnblogs.com/asmurmur/p/14778548.html)

本文主要讲述密码学 base64隐写的python脚本解密的方法，以及一些python函数的用法。

隐写是密码学的一个概念，现代，我们可以把数据藏到一张图片中，一段文字中。**隐写对于数据取证，数据隐藏有着十分重要的价值，主要用于军工，和安全方面。**

网上写了很多多关于xctf MISC新手篇的base64Stego隐写的教程，但大都不太清楚，基本上都是讲了一段隐写原理，直接上代码了。但是代码是这道题的关键，代码讲了如何解码这个隐写的完整流程，

## 这次我以一个python的源码的解释，实现隐写过程的解释。

可能会花费你很长时间，大约一天半天，但是我们要有信心，恒心！

### base64 隐写原理 + 破解隐写的代码

仔细看！！！！！！！  
**[Tr0y's Blog baseStego](https://www.tr0y.wang/2017/06/14/Base64steg/)**  
存在隐写的编码末尾都存在 = ，一个 = 隐写 2bit

隐写的编码，解码后，再编码，最后挨着 = 的字符会发生变化。

# 史上最完全的源码解析

真小白级此题的隐写解码的python解析,

### 代码分析

```python
# -*- coding: utf-8 -*-
import base64
b64chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
with open('stego.txt', 'rb') as f:
    bin_str = ''
    for line in f.readlines():
        stegb64 = str(line, "utf-8").strip("\n")
        tureb64 =  str(base64.b64encode(base64.b64decode(stegb64)), "utf-8").strip("\n")
        offset = abs(b64chars.index(stegb64.replace('=','')[-1])-b64chars.index(tureb64.replace('=','')[-1]))
        equalnum = stegb64.count('=') #no equalnum no offset
        if equalnum:
            bin_str += bin(offset)[2:].zfill(equalnum * 2)
        print(''.join([chr(int(bin_str[i:i + 8], 2)) for i in range(0, len(bin_str), 8)])) 
```

#### 1 python 3.8.无法保存

```none
# -*- coding: utf-8 -*-
```

在 python 3.8 IDE编写的程序文件无法保存存在中文字符的代码，也就无法运行，加上这一行就可以了保存了。

#### 2 这一行为后面求隐写数据提供了标尺，后面再解释

```none
b64chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
```

#### 3 python 文件读写

```none
with open('1.txt', 'rb') as f:
```

python提供的打开文件的方法，不需要关闭文件，即不需要写 **f.close()** ,但要注意**文件操作的代码**都写到 f:下面，有格式要求，有缩进。

注意stego.txt要和脚本放到同一目录下。

"r" - 读取 - 默认值。打开文件进行读取，如果文件不存在则报错。  
"b" - 二进制 - 二进制模式（例如图像）。  
以二进制读入文件数据，也可以直接读入文本数据。

[w3school 文件读写](https://www.w3school.com.cn/python/python_file_handling.asp)  
[博主 有梦就要去实现它 **with open() as f:**](https://www.cnblogs.com/ymjyqsx/p/6554817.html)

#### 4 隐写数据二进制字符串

```none
bin_str = ''
```

用来存储，隐藏的字符flag, 在后面所有求出的隐写二进制数据都将追加到 bin_str 的尾部

#### 5 readlines()

```none
for line in f.readlines():
```

到底二进制读出来的line是什么呢！  
源数据：IHdyaXRpbmcgaGlkZGVuIG1lc3NhZ2VzIGluIHN1Y2ggYSB3YXkgdGhhdCBubyBvbmV=\n

print(line)：b'IHdyaXRpbmcgaGlkZGVuIG1lc3NhZ2VzIGluIHN1Y2ggYSB3YXkgdGhhdCBubyBvbmV=\n' **class 'bytes'**

可以使用 readlines() 方法返回所有行：  
循环读入文件，每次读取一行，下面就是对每一次读入的二进制数据的一些操作。

#### 6 strip("\n")

```none
stegb64 = str(line, "utf-8").strip("\n")  //将读入的二进制串编成文本串，此时和stego.txt中的base64隐写串一样，去除了\n换行符 假！
tureb64 =  str(base64.b64encode(base64.b64decode(stegb64)), "utf-8").strip("\n")  //解码后的又编码的base64串，即原来的base64 真！
```

可以理解为 **utf-8**的英文字符 和 **ASCII**的英文字符　编码是一致的。　在任何一种编码格式中　**0-127所代表的字符都是一样的**

在base64隐写中，如果存在隐写的数据，隐写数据后的base64 和 没有隐写数据的base64 在最后一个字符会发生变化，即= 旁边

一个 = 隐藏 2bit数据。集齐8bit,就可以拼出一个字符串

**eg.程序数据中的隐写**

stegb64 = IHdyaXRpbmcgaGlkZGVuIG1lc3NhZ2VzIGluIHN1Y2ggYSB3YXkgdGhhdCBubyBvbmV=

tureb64 = IHdyaXRpbmcgaGlkZGVuIG1lc3NhZ2VzIGluIHN1Y2ggYSB3YXkgdGhhdCBubyBvbmU=

**str(bytes, encoding, errors)这个很重要**

- **str(line)**  
    b'IHdyaXRpbmcgaGlkZGVuIG1lc3NhZ2VzIGluIHN1Y2ggYSB3YXkgdGhhdCBubyBvbmU=\n'
    
- **str(line,'utf-8')**  
    IHdyaXRpbmcgaGlkZGVuIG1lc3NhZ2VzIGluIHN1Y2ggYSB3YXkgdGhhdCBubyBvbmU=
    

如果既没有给出encoding也没有给出errors, str(object)返回object.__str__()，这是object的“非正式”或可打印的字符串表示。对于字符串对象，这是字符串本身。如果object没有__str__()方法，则str()返回repr(object)。

如果至少给出了encoding或errors中的一个，object应该是bytes-like object(例如bytes或bytearray)。在这种情况下，如果object是一个bytes(或bytearray)对象，则str(bytes, encoding, errors)等价于bytes.decode(encoding, errors)

这里隐写了数据 **'01'**

**特别**！如果没有变化，也算是一种隐写 ==->'0000' =->'00' 这个可能根据不同的隐藏方法有关。我们也可以定义只有不同的base64真假编码可以藏数据。但这里是只要有 = 就有隐写！

- **eg.strip()**  
    a=" gho stwwl\n"  
    a.strip("\n") = ' gho stwwl'  
    去掉一行首部和尾部的换行符，若要去一边的话还有 rstrip() lstrip()

#### 7 offset 偏离(数字类型)

```none
offset = abs(b64chars.index(stegb64.replace('=','')[-1])-b64chars.index(
(ture64.replace('=','')[-1]))
```

- abs() 返回绝对值 V的位置 - U的位置
- stegb64.replace('=','')[-1] 去掉末尾的'=' 并且返回它的最后一个字符 V
- rowb64.replace('=','')[-1] 去掉末尾的'=' 并且返回它的最后一个字符 U
- index() 返回这个字符在 b64chars 中的位置 ，**这就是b64chars的使用之处**

#### 8 计算 '=' 的数量

```none
equalnum = stegb64.count('=') #no equalnum no offset
if equalnum:
            bin_str += bin(offset)[2:].zfill(equalnum * 2)
```

如果存在等号表示隐藏了数据，我们把隐藏的数据转换成二进制存到 bin_str 中 以追加的方式

- **bin(x)** 返回一个整数 int 或者长整数 long int 的二进制表示。  
    **bin(1)='0b1'** 上面的例子就是这个(U V)  
    bin(2)='0b10'  
    bin(4)='0b100'  
    因为返回的字符串都有 '0b' 但我们只要二进制数据  
    [2:] 从 '0b' 之后截取 我们取到'1'  
    但是这个隐写了 2bit 所以用到了 zfill()
    
- **.zfill(equalnum * 2)** 方法返回指定长度的字符串，原字符串右对齐，前面填充0。
    
- str = '1'  
    str.zfill(2) = '01'  
    str.zfill(4) = '0001'
    

经过这次的转换 我们求解了 '01' 的隐藏数据

#### 经过几个循环

IHdyaXRpbmcgaGlkZGVuIG1lc3NhZ2VzIGluIHN1Y2ggYSB3YXkgdGhhdCBubyBvbmV= **'01'**

LCBhcGFydCBmcm9tIHRoZSBzZW5kZXIgYW5kIGludGVuZGVkIHJlY2lwaWVudCwgc3VzcGU= **'00'**

Y3RzIHRoZSBleGlzdGVuY2Ugb2YgdGhlIG1lc3M= **'00'**

YWdlLCBhIGZvcm0gb2Ygc2VjdXJpdHkgdGhyb3VnaCBvYnNjdXJpdHkuIFS= **'11'**

我们得到了 **B 0100 0011** 这是 码ascii

#### 输出

```none
print(''.join([chr(int(bin_str[i:i + 8], 2)) for i in range(0, len(bin_str), 8)])) 
```

- **int() 函数用于将一个字符串或数字转换为整型。**  
    int(x, base=10)  
    x -- 字符串或数字。  
    base -- 进制数，默认十进制。
- **join()**

Python join() 方法用于将序列中的元素以指定的字符连接生成一个新的字符串。  
str.join(sequence)  
sequence -- 要连接的元素序列。

str = "-";  
seq = ("a", "b", "c"); # 字符串序列  
print str.join( seq );  
结果 ： a-b-c

#### 在Python中函数是一类公民

```none
def create_adder(x):
    def adder(y):
        return x + y
    return adder

add_10 = create_adder(10)
add_10(3)

13
```

#### 匿名函数

```none
(lambda x: x > 2)(3)
True

(lambda x, y: x ** 2 + y ** 2)(2, 1)
5
```

```none
高阶函数

list(map(add_10, [1, 2, 3]))
[11, 12, 13]

list(map(max, [1, 2, 3], [4, 2, 1]))
[4, 2, 3]

list(filter(lambda x: x > 5, [3, 4, 5, 6, 7]))
[6, 7]
```

列表推导是`map`和`filter`的语法糖，请你自己找出对照关系

### **[.. for in range(10)]**

```none
[add_10(i) for i in [1, 2, 3]]
[11, 12, 13]

[x for x in [3, 4, 5, 6, 7] if x > 5]
[6, 7]

列表推导也可以用在集合和字典上

{x for x in 'abcddeef' if x not in 'abc'}
{'d', 'e', 'f'}

{x: x**2 for x in range(5)}
{0: 0, 1: 1, 2: 4, 3: 9, 4: 16}
```

为了匹配 sequence 生成一个字符列表 以便用于 join();

最后，这些解码的字符就连接到一起了。并且，每读一行数据，加工后输出一次隐写数据。

# 懂(mei)了(dong)吗

---

---

---

---

如果我的工作对您有帮助，您想回馈一些东西，你可以考虑通过分享这篇文章来支持我。我非常感谢您的支持，真的。谢谢！

**作者**：[Dba_sys](https://www.cnblogs.com/asmurmur/) (Jarmony)

**转载**以及**引用**请**注明原文链接**：[https://www.cnblogs.com/asmurmur/p/14778548.html](https://www.cnblogs.com/asmurmur/p/14778548.html)