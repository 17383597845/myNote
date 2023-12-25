


测试文件：[https://adworld.xctf.org.cn/media/task/attachments/5d4117b968684b9483d0d4464e0a6fea](https://adworld.xctf.org.cn/media/task/attachments/5d4117b968684b9483d0d4464e0a6fea)

这道题要使用到gdb文件调试，gdb调试相关知识：[https://www.cnblogs.com/Mayfly-nymph/p/11403150.html](https://www.cnblogs.com/Mayfly-nymph/p/11403150.html)

## 1.准备

![](https://img2018.cnblogs.com/blog/1228809/201908/1228809-20190823233142508-1560232090.png)

获得信息

1. 32位文件

## 2.IDA打开

获得main函数的C语言代码

![复制代码](https://common.cnblogs.com/images/copycode.gif)

int __cdecl main(int argc, const char **argv, const char **envp)
{
  setlocale(6, &locale);
  banner();
  prompt_authentication();
  authenticate();
  return 0;
}

![复制代码](https://common.cnblogs.com/images/copycode.gif)

### 2.1 代码分析

通过分析，可知authenticate();函数，储存着flag，进入函数

![复制代码](https://common.cnblogs.com/images/copycode.gif)

 1 void authenticate() 2 {
 3   int ws[8192]; // [esp+1Ch] [ebp-800Ch]
 4   wchar_t *s2; // [esp+801Ch] [ebp-Ch]
 5 
 6   s2 = decrypt(&s, &dword_8048A90);
 7   if ( fgetws(ws, 0x2000, stdin) )
 8   {
 9     ws[wcslen(ws) - 1] = 0;
10     if ( !wcscmp(ws, s2) )
11       wprintf(&unk_8048B44);
12     else
13       wprintf(&unk_8048BA4);
14 }
15   free(s2);
16 }

![复制代码](https://common.cnblogs.com/images/copycode.gif)

通过第10~13行代码，我们可以知道s2就是我们需要flag(ws为输入值)

**unk_8048B44内容**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)View Code

**unk_8048BA4内容**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)View Code

### 2.2 authenticate()函数

回到authenticate()函数的汇编代码

![复制代码](https://common.cnblogs.com/images/copycode.gif)

 1 .text:08048708                 push    ebp 2 .text:08048709                 mov     ebp, esp 3 .text:0804870B                 sub     esp, 8028h 4 .text:08048711                 mov     dword ptr [esp+4], offset dword_8048A90 ; wchar_t *
 5 .text:08048719                 mov     dword ptr [esp], offset s ; s
 6 .text:08048720                 call    decrypt 7 .text:08048725                 mov     [ebp+s2], eax 8 .text:08048728                 mov     eax, ds:stdin@@GLIBC_2_0
 9 .text:0804872D                 mov     [esp+8], eax    ; stream
10 .text:08048731                 mov     dword ptr [esp+4], 2000h ; n
11 .text:08048739                 lea     eax, [ebp+ws]
12 .text:0804873F                 mov     [esp], eax      ; ws
13 .text:08048742                 call    _fgetws
14 .text:08048747                 test    eax, eax
15 .text:08048749                 jz      short loc_804879C
16 .text:0804874B                 lea     eax, [ebp+ws]
17 .text:08048751                 mov     [esp], eax      ; s
18 .text:08048754                 call    _wcslen
19 .text:08048759                 sub     eax, 1
20 .text:0804875C                 mov     [ebp+eax*4+ws], 0
21 .text:08048767                 mov     eax, [ebp+s2]
22 .text:0804876A                 mov     [esp+4], eax    ; s2
23 .text:0804876E                 lea     eax, [ebp+ws]
24 .text:08048774                 mov     [esp], eax      ; s1
25 .text:08048777                 call    _wcscmp
26 .text:0804877C                 test    eax, eax
27 .text:0804877E                 jnz     short loc_804878F
28 .text:08048780                 mov     eax, offset unk_8048B44
29 .text:08048785                 mov     [esp], eax
30 .text:08048788                 call    _wprintf
31 .text:0804878D                 jmp     short loc_804879C

![复制代码](https://common.cnblogs.com/images/copycode.gif)

通过第6~7行代码，我们可以知道eax储存着decryp函数返回的flag值，再保存到s2

**decrypt函数**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

wchar_t *__cdecl decrypt(wchar_t *s, wchar_t *a2)
{
  size_t v2; // eax
  signed int v4; // [esp+1Ch] [ebp-1Ch]
  signed int i; // [esp+20h] [ebp-18h]
  signed int v6; // [esp+24h] [ebp-14h]
  signed int v7; // [esp+28h] [ebp-10h]
  wchar_t *dest; // [esp+2Ch] [ebp-Ch]
  v6 = wcslen(s);
  v7 = wcslen(a2);
  v2 = wcslen(s);
  dest = (wchar_t *)malloc(v2 + 1);
  wcscpy(dest, s);
  while ( v4 < v6 )
  {
    for ( i = 0; i < v7 && v4 < v6; ++i )
      dest[v4++] -= a2[i];
  }
  return dest;
}

![复制代码](https://common.cnblogs.com/images/copycode.gif)

### 2.3 gdb调试准备

综上所述，我们需要的flag保存在eax中，因此我们可以将断点设置在decrypt函数处，单步执行后，eax保存着我们需要的值，再读取eax值即可。

## 3.gdb调试

### 3.1 调试文件

gdb pro -q

![](https://img2018.cnblogs.com/blog/1228809/201908/1228809-20190823235152156-21473286.png)

**设置断点**

b decrypt

**执行到断点**

r

单步执行decrypt

n

![](https://img2018.cnblogs.com/blog/1228809/201908/1228809-20190824001330583-26876450.png)

**显示寄存器**

i r

![复制代码](https://common.cnblogs.com/images/copycode.gif)

eax            0x804e800           134539264
ecx            0x1480              5248
edx            0x7d                125
ebx            0x0                 0
esp            0xffff5300          0xffff5300
ebp            0xffffd328          0xffffd328
esi            0xf7fac000          -134561792
edi            0xf7fac000          -134561792
eip            0x8048725           0x8048725 <authenticate+29>
eflags         0x282               [ SF IF ]
cs             0x23                35
ss             0x2b                43
ds             0x2b                43
es             0x2b                43
fs             0x0                 0
gs             0x63                99

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**查看eax的值**

x/6sw $eax

6：显示6行数据

s：字符串形式

w：word（4字节）形式

![复制代码](https://common.cnblogs.com/images/copycode.gif)

0x804e800:    U"9447{you_are_an_international_mystery}"
0x804e89c:    U"W\001\xf7ade1e8\xf7ade1ea\xf7ade1ec\xf7ade1ee\xf7ade1f0\xf7ade1f2\xf7ade1f4\xf7ade1f6\xf7ade1f8\xf7ade1fa\001\xf7ade200\xf7ade204\xf7ade208\xf7ade20c\xf7ade210\xf7ade214\xf7ade218\xf7ade21c\xf7ade220\xf7ade224\xf7ade228\xf7ade22a\xf7ade22c\xf7ade22e\xf7ade230\xf7ade232\xf7ade234\xf7ade236\xf7ade238\xf7ade23a\060\061\062\063\064\065\066\067\070\071\x175a\xf7ade268\xf7ae3fd0\xf7aefaa0\xf7af5808\001\xf7b07f4c"
0x804e968:    U"\xf7b07f54"
0x804e970:    U""
0x804e974:    U"\xf7b07f7c\xf7b088b8\xf7b09234\xf7b0aa74\xf7b0acec\xf7b0af64\xf7b0b29c\xf7b0d194\xf7b0f08c\xf7b0f3c4\xf7b0f67c\xf7b10f74\xf7b127b4\xf7b138e4\xf7b14854\xf7b14b6c\xf7b1a988\xf7b1fba4\021\x435f687a\x54552e4e\x382d46\021\x435f687a\x54552e4e\x382d46!\x804ea00\001\x804ea80\001"
0x804e9f4:    U""

![复制代码](https://common.cnblogs.com/images/copycode.gif)

## 4.get flag！

> 9447{you_are_an_international_mystery}