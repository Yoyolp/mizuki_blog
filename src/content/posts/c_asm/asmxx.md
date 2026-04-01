---
title: asmxx
published: 2025-11-13
description: "为方便写汇编代码，所以将一些常用的功能封装到 `asm_simple_func.s` 中， 这是其中的内容"
tags: ["asm"]
category: language
draft: false
---

同时这是 [GAS汇编器](https://ftp.gnu.org/old-gnu/Manuals/gas-2.9.1)文档地址

## `asm_simple_func.s` :

```asm
.intel_syntax noprefix

.data
  char_endline:
    .ascii "\n"
    char_endline_len = . - char_endline

  char_buf_1: .byte 0             # char 缓冲区 大小1
  char_buf_1ln: .byte 0, 0x0a     # char 缓冲区 大小2 + (换行符)

.text

# print 函数 实现原型
.global asm_print
.type asm_print, @function
__asm_print_old:
  # push address[msg]
  # push msg_len
  mov rax, 1
  mov rdi, 1
  mov rsi, [rsp + 16]
  mov rdx, [rsp + 8]
  syscall 
  ret 16

.global __asm_func_print_stack
.type __asm_func_print_stack, @function
__asm_func_print_stack:
  push rbp
  mov rbp, rsp

  mov rax, 1
  mov rdi, 1
  mov rsi, [rbp + 24]
  mov rdx, [rbp + 16]
  syscall

  leave
  ret 16

# 打印字符
.macro asm_print _str _str_len
  push offset \_str
  push \_str_len
  call __asm_func_print_stack  
.endm

# 打印字符并在末尾添加换行
.macro asm_println _str _str_len
  push offset \_str
  push \_str_len
  call __asm_print

  push offset char_endline
  push char_endline_len
  call __asm_func_print_stack 
.endm
```
