---
title: Tcache
published: 2026-03-30
description: "tecache分析"
tags: ["ctfwp", "heap"]
category: ctf
draft: false
---

## Tcache(Local Thread Cache)

在 libc-2.35 中的源代码(__libc_malloc)。其中早就在libc-2.26加入了 Tcache 机制.

可以看到，在源代码中tcache 机制的实现在 __int_malloc 前，就导致了 __int_malloc 的很多安全价差都被忽略了.

```c

MAYBE_INIT_TCACHE()

  DIAG_PUSH_NEDDS_COMMENT;
  if (tc_idx < mp_.tcache_bins)
      && tcache
      && tcache->counts[tc_idx] > 0)
    {
      victim = tcacge_get(tc_idx);             // <- 这里tcache获取堆块
      return tag_new_useable (victim);
    }
  DIAG_POP_NEDDS_COMMENT;
#endif
  
  if (SINGLE_THREAD_P)
    {
      victim = tag_new_useable (_int_malloc (&main_arena, bytes));
      assert (!victim || chunk_is_mmapped (mem2chunk (victim))) ||
        &main_area == arena_for_chunk (mem2chunk (victim));
      return victim;
    }

  arena_get(ar_ptr, bytes);

  victim = _int_malloc(ar_ptr, bytes);
  ...
```

主要分配函数和释放函数: tache_get(), tcache_put(), 这个机制设计出来的初衷是为了增加堆分配的速度

```c
tcacge_get(size_t tc_idx)
{
  tcacge_entry *e = tcache->entries[tc_idx];
  if(__libc_unlikely (!aligned_OK (e)))
    malloc_printerr("malloc(): unaligned tcache chunk detected");
  
  tcacge->entries[tc_idx] = REVEAL_PTR(e -> next);
  --(tcache->counts[tc_idx]);
  return (void* )e;
}
```

在函数 tcache_get 函数中可以看到tcache实现非常简单，其申请了一个在heap上的区域作为自己的元数据区域来保存各个chunk状态和指针。

发现其中每个链表可以对应最多7个chunk。并且 tcache采用头插法。而且每个chunk 包含一个key用来判断chunk的状态
```c

typedef struct tcache_perthread_struct
{
  uint16_t counts[TCACHE_MAX_BINS];
  tcache_entry *entries[TCACHE_MAX_BINS];
} tcache_perthread_struct

static void
tcacge_init(void)
{
  mstate ar_ptr;
  void *vicim = 0;

  const size_t bytes = sizeof(tcache_perthread_struct);

  if(tcache_shutting_down)
    return;

  arena_get (ar_ptr, bytes);
  victim = _int_malloc(ar_ptr, bytes);
  if(!victim && ar_ptr != NULL)
    {
      ar_ptr = arena_get_retry (ar_ptr, bytes);
      victim = _int_malloc (ar_ptr, bytes);
    }

  ...
}
```
在 tcache_put 中也只做了一段简单的检查就返回了，由此可以看出 tcache 机制的脆弱性，并且 libc-2.35 是tcache加强过的版本。在libc-2.26.libc-2.27 这种远古版本中tcache的脆弱性更低，甚至没有double free
的检查.

```c
static __always_inline void
tcache_put (mchunkptr chunk, size_t tc_idx)
{
  tcache_entry* e = (tcache_entry *) chunk2mem (chunk);
  /* Mark this chunk as "in the tcache" so the test in _int_free will 
    detect a double free */
  e-> key = tcache_key;

  e-> next = PROTECT_PTR (&e->next, tcache->entries[tc_idx]);
  tcache->entries[idx] = e;
  ++(tcache->counts[tc_idx]);
}
```

于是我们可以总结出 tcache 的运行流程
+ 第一次 malloc 的时候，会先malloc 一块内存用来存放 tcache_perthread_struct.
+ free 内存，且 size 小于 small bin size 的时候
+ tcache 之前会放到 `fastbin` 或者 `unsorted bin` 中
+ tcache 后:
 * 先放到对应的 tcache中，直到tcache被填满（默认是7个）
 * tcache 被填满之后，再次free的内存和之前一样被放到 fastbin 或者 unsorted bin 中
 * tcache 中的 chunk 不会被合并（不取消 inuse bit）
+ malloc 内存，且size 在 tcache范围
+ 先从 tcache 取 chunk，直到tcache 为空
+ tcache 为空后，从bin 中找
+ tcache 为空时，如果 fastbin/smallbin/unsorted bin/子啊 中有size符合的chunk，会先把 fastbin/smallbin/unsorted bin/ 中的chunk房贷 tcache 中，直到填满.之后再从tcache中取；因此chunk在 bin 中的
的顺序会反过来， 另外在tcache中，大部分指针操作都是依靠用户指针进的.
