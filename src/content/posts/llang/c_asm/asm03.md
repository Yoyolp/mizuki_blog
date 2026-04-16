---
title: asm03
published: 2025-10-13
description: "GAS 汇编表示偏移地址, .include 伪指令"
tags: ["asm"]
category: language
draft: false
---

## 寻址方式

在 GAS 汇编中有多种寻址方式，可以给我们便利的的访问或者操纵内存中的值，下面是寻址方式总结表

| 寻址方式      | 语法示例                             | 描述                    |
| ------------- | ------------------------------------ | ----------------------- |
| 立即寻址      | `mov eax, 123`                     | 操作数在指令中          |
| 寄存器寻址    | `mov eax, ebx`                     | 操作数在寄存器中        |
| 直接内存      | `mov eax, [var]`                   | 地址在指令中            |
| 寄存器间接    | `mov eax, [esi]`                   | 地址在寄存器中          |
| 基址+偏移     | `mov eax, [ebp-8]`                 | 基址寄存器 + 偏移       |
| 基址+变址     | `mov eax, [ebx+esi]`               | 基址寄存器 + 变址寄存器 |
| 基址+比例变址 | `mov eax, [ebx+esi*4]`             | 基址 + 变址 * 比例因子  |
| 全寻址方式    | `mov eax, [base+index*scale+disp]` | 完整的寻址表达式        |

## 伪指令

在 GAS 汇编中，有丰富的伪指令

1. 段定义伪指令
2. 数据定义伪指令
3. 符号和标签伪指令
4. 对齐和位置伪指令
5. ...

## 关于段定义伪指令

| GAS 伪指令        | 等效的 `.section` | 描述                         |
| ----------------- | ------------------- | ---------------------------- |
| `.text`         | `.section .text`  | 定义代码段。                 |
| `.data`         | `.section .data`  | 定义已初始化数据段。         |
| `.bss`          | `.section .bss`   | 定义未初始化数据段。         |
| `.section 名称` | -                   | 定义自定义段或指定详细属性。 |

### `.bss` & `.data` 的区别

| 特性                     | `.data` 段                          | `.bss` 段                                               |
| ------------------------ | ------------------------------------- | --------------------------------------------------------- |
| **用途**           | 存储**已初始化**的全局/静态变量 | 存储**未初始化**或**零初始化**的全局/静态变量 |
| **可执行文件大小** | 占用磁盘空间                          | **不占用**磁盘空间（只记录大小）                    |
| **程序加载时**     | 数据从文件直接读入内存                | 内存区域被分配并**自动清零**                        |
| **汇编指令**       | `.byte`, `.word`, `.asciz` 等   | `.lcomm`, `.comm`                                     |

简单来说， **.bss 段是程序运行时的一块“空白画布”** ，操作系统保证在你开始使用它时，它已经是干净（全零）的状态，同时又不让你的可执行文件变得臃肿。这是一种非常高效的内存初始化策略。

### 自定义段 `.sections`

**自定义段**允许程序员创建非标准的、特定用途的内存区域

```asm
.section 段名称 [, 标志] [, 类型] [, 参数] 
```

段标志定义了该内存区域的属性和权限：

| 标志  | 含义             | 描述                            |
| ----- | ---------------- | ------------------------------- |
| `a` | allocatable      | 段在程序加载时会被分配内存      |
| `w` | writable         | 段可写                          |
| `x` | executable       | 段可执行                        |
| `d` | data             | 数据段（通常与 `w` 一起使用） |
| `M` | mergeable        | 段内容可合并                    |
| `S` | contains strings | 段包含字符串                    |
| `G` | member of group  | 段属于某个组                    |

### 示例 ：

```asm
.intel_syntax noprefix
.section string_data, "a" #自定义 string_data 段
msg:
  .ascii "HELLO ASM!\n"
  msg_len = . - msg

.text
.global main
.type main, @function      # 定义标签 main为函数
main:

  mov rax, 1
  mov rdi, 1
  lea rsi, [msg]
  mov rdx, msg_len
  syscall

  ret
  
.global _start
_start:
  call main

  mov rax, 60
  xor rdi, rdi
  syscall
```

## `.include` 伪指令

它用于在汇编过程中包含其他源文件

基本语法

```asm
.include "文件名"
```

我们可以简单封装一些函数，来供其他汇编程序调用，提供代码的复用性
比如我们创建一个 函数 `asm_print` 用于打印字符串, 在文件 **asm_simple_func.s**

```asm
.intel_syntax noprefix
.text

.global asm_print
.type asm_print, @function
asm_print:
  # push address[msg]
  # push msg_len
  mov rax, 1
  mov rdi, 1
  mov rsi, [rsp + 16]
  mov rdx, [rsp + 8]
  syscall 
  ret 16

```

然后 我们可以这样调用他(includeTest.s)

```asm
.intel_syntax noprefix

.include "./asm_simple_func.s"

.data
msg:
    .ascii "HELLO ASM!\n"
    msg_len = . - msg
.text
.global _start
_start:

  lea rax, [msg]
  push rax
  push msg_len
  call asm_print

  mov rax, 60
  mov rdi, 42
  syscall
```

## 大小写转化问题

### ASCII码

**ASCII码** （American Standard Code for Information Interchange，美国信息交换标准代码）是一种字符编码标准，用于在计算机中表示文本字符。

### 核心特性：

* **7位编码** ：使用7位二进制数表示字符（共128个字符）
* **标准字符集** ：包含英文字母、数字、标点符号和控制字符
* **广泛兼容** ：几乎所有计算机系统都支持

常见的ascii 码值

```asm
数字
'0' = 48 (0x30)    '1' = 49 (0x31)    ...    '9' = 57 (0x39)
大写字母
'A' = 65 (0x41)    'B' = 66 (0x42)    ...    'Z' = 90 (0x5A)
小写字母
'a' = 97 (0x61)    'b' = 98 (0x62)    ...    'z' = 122 (0x7A)
特殊字符
空格 = 32 (0x20)   换行 = 10 (0x0A)   回车 = 13 (0x0D)
'!' = 33 (0x21)    '\n' = 10 (0x0A)   '\0' = 0 (0x00)
```

下面考虑一个问题将字符串str1的字母转化为小写, str2的字母转化为大写

通过ASCII字符的分析发现大写和小写ASCII字符在二进制的第5位有差异，小写字母为1，大写字母为0，所以可以使用位运算来实现大小写的转换

如下是功能实现
```asm
.intel_syntax noprefix

.include "./asm_simple_func.s"

.data
  str1:
    .ascii "basic"
    str1_len = . - str1 # 5
  
  str2:
    .ascii "INFOMATION"
    str2_len = . - str2 # 11

  msg1:
    .ascii "\nSolve Str ...\n"
    msg1_len = . - msg1

.text

.global _start
_start:
  asm_println str1 str1_len 
  asm_println str2 str2_len 
  # 处理字符串 str1

  asm_println msg1 msg1_len
  
  mov r12, str1_len
  mov r13, 0
str1_loop:
  mov al, [str1 + r13]
  and al, 0xdf             # 0xDF = 1101 1111
  mov [char_buf_1], al

  asm_print char_buf_1 1    

  inc r13
  dec r12
  jnz str1_loop

  asm_print char_endline 1

  mov r12, str2_len
  mov r13, 0  
str2_loop:
  mov al, [str2 + r13]
  or al, 0x20           # 0x20 = 0010 0000
  mov [char_buf_1], al 
  
  asm_print char_buf_1 1

  inc r13
  dec r12
  jnz str2_loop

  asm_print char_endline 1

  # end
  mov rax, 60
  mov rdi, 42
  syscall
```
除了使用 位运算的方法 还可以使用 算数的方法来实现大小写的转化，在ASCII码中 小写字母 和大写字母 之间的差值为20H












