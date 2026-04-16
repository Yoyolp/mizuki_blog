---
title: ropper
published: 2026-04-07
description: "ropper的使用"
tags: ["ctf", "pwn"]
category: ctf
draft: false
---
## ropper

Ropper 是一个功能强大的二进制文件分析和 ROP（Return-Oriented Programming，返回导向编程）链生成工具

### 安装

```bash
pip install ropper
```

命令行选项主要分为以下几类：

| 类别                       | 关键参数                                  | 说明                                             |
| -------------------------- | ----------------------------------------- | ------------------------------------------------ |
| **基本信息**         | `-f <file>`                             | 指定要分析的文件                                 |
|                            | `-i`,`-s`,`--imports`,`--symbols` | 显示文件头、节区、导入表、符号表等信息           |
| **Gadget 搜索**      | `--search <regex>`                      | 通过正则表达式搜索 Gadget                        |
|                            | `--opcode <opcode>`                     | 通过字节码（如 `ffe4`，支持通配符 `?`）搜索  |
|                            | `--instructions <ins>`                  | 通过指令字符串（如 `"pop eax; ret"`）搜索      |
|                            | `--semantic <约束>`                     | 语义搜索（如 `eax==1`）                        |
| **Gadget 过滤/控制** | `--inst-count <n>`                      | 设置 Gadget 的最大指令条数（默认6）              |
|                            | `-b <badbytes>`                         | 排除包含特定坏字节（如 `00`、`0a`）的 Gadget |
|                            | `--type <rop/jop/sys/all>`              | 指定 Gadget 类型                                 |
|                            | `--detailed`,`--all`                  | 输出详情或包含重复项                             |
| **特定搜索**         | `-p, --ppr`                             | 搜索 `pop reg; pop reg; ret`（常用于 x86/x64） |
|                            | `-j <reg>, --jmp <reg>`                 | 搜索 `jmp reg`                                 |
|                            | `--stack-pivot`                         | 搜索栈迁移 Gadget                                |
| **ROP 链生成**       | `--chain execve`                        | 生成 `execve("/bin/sh")`的 ROP 链（Linux）     |
|                            | `--chain mprotect=<地址>:<大小>`        | 生成调用 `mprotect`的 ROP 链                   |
|                            | `--chain virtualprotect=...`            | 生成调用 `VirtualProtect`的 ROP 链（Windows）  |
| **汇编/反汇编**      | `--asm "jmp esp"`                       | 将指令汇编为字节码                               |
|                            | `--disasm ffe4`                         | 将字节码反汇编为指令                             |
| **其他**             | `--console`                             | 启动交互式命令行模式                             |
|                            | `--nocolor`                             | 禁用彩色输出                                     |
|                            | `--clear-cache`                         | 清除缓存                                         |

### 常用命令
```bash
# 1. 查看文件基本信息（架构、保护机制）
ropper --file ./challenge --info

# 2. 查看导入函数（判断可用的危险函数）
ropper --file ./challenge --imports

# 3. 查看段/节区信息（找可写可执行段）
ropper --file ./challenge --sections
ropper --file ./challenge --segments

# x64: 第一个参数是 rdi
ropper --file ./challenge --search "pop rdi"
# 或更通用
ropper --file ./challenge --search "pop %"

# x64: rsi, rdx, rcx, r8, r9
ropper --file ./challenge --search "pop rsi"
ropper --file ./challenge --search "pop rdx"

# 一次性找多个
ropper --file ./challenge --search "pop rdi; pop rsi;"

# 先在二进制里找（如果有）
ropper --file ./challenge --string "/bin/sh"
# 检查导入表
ropper --file ./challenge --imports | grep system

# 只找 2-3 条指令的短 gadget（更稳定）
ropper --file ./challenge --inst-count 3 --search "pop"
```
### 对二进制操作
```bash
# 让栈变为可执行（Linux）
ropper --file ./challenge --chain "mprotect address=0x7ffffffde000 size=0x21000"

# Windows 环境
ropper --file ./challenge.exe --chain "virtualprotect address=0x401000 size=0x1000"

# x86 生成 execve("/bin/sh") 的 ROP 链
ropper --file ./challenge --chain execve

# 自定义命令
ropper --file ./challenge --chain "execve cmd=/bin/cat flag.txt"

```
