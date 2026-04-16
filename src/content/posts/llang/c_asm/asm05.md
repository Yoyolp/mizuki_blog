---
title: asm05
published: 2025-11-13
description: "跳转指令 CMP, JNC, JC"
tags: ["asm"]
category: language
draft: false
---

## cmp - 比较指令

```asm

cmp <src>, <dest>

cmp rax, 10
==>
sub rax, 10  # 但是 rax 不保存结果

mov rax, 15
cmp rax, 10    # 15 - 10 = 5 (正数)
# 结果：ZF=0, CF=0, SF=0

mov rax, 5
cmp rax, 10    # 5 - 10 = -5 (需要借位)
# 结果：ZF=0, CF=1, SF=1

mov rax, 10
cmp rax, 10    # 10 - 10 = 0
# 结果：ZF=1, CF=0, SF=0
```

### 影响的标志位

* **ZF (零标志)** ：如果结果为 0，则置 1 (`destination == source`)
* **CF (进位标志)** ：如果发生借位/进位，则置 1 (`destination < source` 无符号比较时)
* **SF (符号标志)** ：如果结果为负，则置 1
* **OF (溢出标志)** ：如果发生有符号溢出，则置 1

## je, jz, jne, jnz, jns, jo, jno, jp, jnp, jb/jnae/jc, jnb/jae/jnc

| 指令        | 含义                      | 测试条件 |
| ----------- | ------------------------- | -------- |
| je/jz       | 相等，结果为0             | ZF = 1   |
| jne/jnz     | 不等/结果不为0            | ZF = 0   |
| js          | 结果为负                  | SF = 1   |
| jns         | 结果非负                  | SP = 0   |
| jo          | 结果没有溢出              | OF = 1   |
| jno         | 结果溢出                  | OF = 0   |
| jp          | 奇偶位为1                 | PF = 1   |
| jnp         | 奇偶位不为1               | PF = 0   |
| jb/jnae/jc  | 低于/不高于/不等于/有借位 | CF = 1   |
| jnb/jae/jnc | 不低于/高于等于/无借位    | CF = 0   |

## div指令

```asm
# 64位除法：RDX:RAX / RCX
mov rdx, 0       # 被除数高64位
mov rax, 100     # 被除数低64位
mov rcx, 3       # 除数
div rcx          # 结果：RAX=商, RDX=余数
# RAX = 33, RDX = 1
```

在使用div 指令的时候效率比较低，不过假如除数为 $2^x$的形式的话，那么他的余数显然是这个除数的低位 我们可以使用 访问这个被除数的低位来获取余数

## mul 指令

```asm
.intel_syntax noprefix
mov rax, 10000000000   # RAX = 10,000,000,000
mov rbx, 300           # RBX = 300
mul rbx                # RDX:RAX = 10,000,000,000 * 300 = 3,000,000,000,000
```

### 标志位影响

`mul` 指令会影响以下标志位：

* **CF（进位标志）** ：如果结果的高半部分（DX/EDX/RDX）不为0，则CF=1
* **OF（溢出标志）** ：与CF相同
