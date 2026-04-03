---
title: doubleFree
published: 2026-04-03
description: "一些经典堆攻击的介绍"
tags: ["ctfwp", "heap"]
category: ctf
draft: false
---

## FastBinAttack

fastbin Attack 是一类漏洞的利用方法，是所有基于 fastbin机制的漏洞方法。这类利用的前提是
+ 存在堆溢出, Uaf等能够控制chunk 内容的漏洞
+ 漏洞发生于 fastbin 类型的chunk 中

fastbin 存在的原因是fastbin 是使用单链表来维护释放堆块的，并且 fastbin 管理的 chunk 即使被释放，其 next_chunk 的 prev_inuse 也不会被清空, 由下面的代码可以看到 fastbin 是这样管理空闲 chunk的
```c
if (__builtin_expect (old == p, 0))
  malloc_printerr("double free or corruption")
p->fd = PROTECT (&p->fd, old)
*fb = p;
    }
  else
    do
    {
      /* Check that the top of the bin is not record we are going to add (i.e. double free)*/
      if(__buildin_expect (old == p, 0))
        malloc_printerr("double free or corruption")
      old2 = old;
      p->fd = PROTECT_PTR("double free or corruption(fasttop)")
    }
    ...
```

### House of spirit

House of spirit的主要原理是改写 free 状态的 fast chunk 的fd 指针，在下一次分配时拿到目标地址的读写权
1. 释放一个 fast chunk，并且为了防止 Tcahce的干扰，所以可能需要绕过Tcahce
2. fast chunk 被放进了 fastbin，假设此时改chunk为第一个被放进 bin 的chunk，那么ptmalloc为其fd赋值为0
3. 利用漏洞 改写fd指针为目标地址`eg: __free_hook`,
4. 申请一个被释放大小的堆块
