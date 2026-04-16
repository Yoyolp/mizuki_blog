---
title: asm06
published: 2025-11-13
description: "nop, .rept 指令的使用"
tags: ["asm"]
category: language
draft: false
---

## .rept 伪指令

通过GUN 汇编手册的介绍发现，通过该伪指令可以重复多次选定的代码

> ## [`.rept count`](https://ftp.gnu.org/old-gnu/Manuals/gas-2.9.1/html_mono/as.html#TOC116)

> []()Repeat the sequence of lines between the `.rept` directive and the next `.endr` directive `count` times.

> For example, assembling

```
        .rept   3
        .long   0
        .endr
```

通过手册中的例子发现 将  指令：分配4字节大小的空间 重复了3次

```
        .long 0
        .long 0
        .long 0
```


## nop 指令
这个指令是空指令，即什么都不做，占用一个CPU的时钟周期，和一个字节的空间

## movsxd & movzx 的使用
 + movsxd rax, ebx   将 32位的值赋值到64位中 多出来的位置 根据符号位来补齐即 如果是正数补0 负数补1       
 + movzx  rax, ebx   将 32 位的值赋值到64位中，多出来的位置使用0来补齐

