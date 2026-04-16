---
title: asm02
published: 2025-10-13
description: "gdb 的使用 Liunx syscall 系统调用约定， loop， Call&Ret, Stack栈的使用"
tags: ["asm"]
category: language
draft: false
---

## GDB

**GDB** 全称是  **GNU Debugger** ，是 GNU 软件系统的一部分。它是一个功能强大的 **程序调试工具** 。

简单来说，GDB 就像一个程序的“手术刀”和“X光机”，它允许你深入到程序内部，观察其运行时的每一个细节。你可以让程序在任何你指定的地方暂停，然后检查当时的状态（比如变量的值、内存的内容、函数的调用关系等），也可以一步一步地执行代码，看程序究竟是如何运行的。

### GDB 能做什么？（主要功能）

1. **控制程序的运行**
   * 可以让程序从指定位置开始运行，或者在任何你希望的地方暂停。
   * 可以逐行执行代码，精确跟踪执行流程。
2. **设置断点**
   * 这是最核心的功能。你可以在特定的 **行号** 、**函数名**甚至满足某个**条件**时设置断点。程序运行到断点处就会自动暂停，让你进行观察。
3. **检查程序状态**
   * 当程序暂停时，你可以查看 **变量的值** 、 **寄存器的内容** 、**内存数据**等。
   * 可以查看程序正在执行哪一段代码。
4. **动态改变程序**
   * 你可以在调试过程中 **修改变量的值** ，或者强制调用某个函数。这可以用于测试不同输入对程序行为的影响，而无需重新编译。
5. **分析程序崩溃**
   * 当程序发生**段错误（Segmentation Fault）** 等严重错误而崩溃时，GDB 可以连接到程序产生的**核心转储（Core Dump）** 文件，告诉你程序在崩溃时正在执行什么代码，调用栈是什么，从而快速定位问题根源。
6. **多线程/多进程调试**
   * 支持调试多线程程序和多个进程，可以查看所有线程的状态，并在特定线程中设置断点。

### gdb 基础命令

1. **基本运行和断点命令**

| 命令                 | 简写    | 说明                                                            | 示例                                                                                       |
| -------------------- | ------- | --------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| `run`              | `r`   | **开始运行程序** ，可以带参数                             | `(gdb) run arg1 arg2`                                                                    |
| `start`            |         | 开始运行程序，并在 `main` 函数开头暂停                        | `(gdb) start`                                                                            |
| `break`            | `b`   | **设置断点**                                              | `(gdb) b main` (在函数处)`(gdb) b 10` (在行号处)`(gdb) b file.c:15` (在指定文件行号) |
| `info breakpoints` | `i b` | 查看所有已设置的断点信息                                        | `(gdb) i b`                                                                              |
| `delete`           | `d`   | **删除断点** （接断点编号）                               | `(gdb) d 1` (删除 1 号断点)`(gdb) d` (删除所有断点)                                    |
| `disablevar`       | `dis` | 禁用断点（接断点编号）                                          | `(gdb) dis 1`                                                                            |
| `enable`           | `en`  | 启用断点（接断点编号）                                          | `(gdb) en 1`                                                                             |
| `continue`         | `c`   | **从当前断点继续运行到下一个断点**                        | `(gdb) c`                                                                                |
| `step`             | `s`   | **单步执行** ，**会进入函数内部**                   | `(gdb) s`                                                                                |
| `next`             | `n`   | **单步执行** ， **不会进入函数** ，将其视为一个整体 | `(gdb) n`                                                                                |
| `finish`           | `fin` | **执行完当前函数，并返回到调用它的地方**                  | `(gdb) fin`                                                                              |
| `until`            | `u`   | 运行直到退出当前循环或到达指定行                                | `(gdb) u 20` (运行到第20行)                                                              |
| `kill`             | `k`   | 终止正在调试的程序，但不退出 GDB                                | `(gdb) k`                                                                                |
| `quit`             | `q`   | **退出 GDB**                                              | `(gdb) q`                                                                                |

2. **查看代码与信息**

| 命令               | 简写     | 说明                                    | 示例                                                                               |
| ------------------ | -------- | --------------------------------------- | ---------------------------------------------------------------------------------- |
| `list`           | `l`    | **列出源代码**                    | `(gdb) l` (当前行附近)`(gdb) l 10` (第10行附近)`(gdb) l main` (main函数附近) |
| `backtrace`      | `bt`   | **显示函数调用栈** （非常有用！） | `(gdb) bt`                                                                       |
| `frame`          | `f`    | 选择栈帧（接栈帧编号）                  | `(gdb) f 0` (切换到 0 号栈帧)                                                    |
| `info registers` | `i r`  | 查看寄存器的值                          | `(gdb) i r`                                                                      |
| `info args`      | `i a`  | 查看当前函数的参数                      | `(gdb) i a`                                                                      |
| `info locals`    | `i lo` | 查看当前函数的局部变量                  | `(gdb) i lo`                                                                     |
| `info variables` | `i va` | 查看所有全局/静态变量                   | `(gdb) i va`                                                                     |
| `layout`         |          | 文本用户界面模式                        | `(gdb) layout src ` `（源代码视图 -g）`                                       |

或者 使用gdb -tui 也可以直接进入用户界面

3. **查看与修改变量**

| 命令             | 简写       | 说明                                             | 示例                                                                       |
| ---------------- | ---------- | ------------------------------------------------ | -------------------------------------------------------------------------- |
| `print`        | `p`      | **打印变量或表达式的值**                   | `(gdb) p variable``(gdb) p *ptr``(gdb) p array[5]@10` (打印数组10个元素) |
| `display`      | `disp`   | **每次程序暂停时，自动打印指定表达式的值** | `(gdb) display variable`                                                 |
| `info display` | `i disp` | 查看所有自动显示项                               | `(gdb) i disp`                                                           |
| `undisplay`    |            | 取消自动显示（接显示编号）                       | `(gdb) undisplay 1`                                                      |
| `set variable` | `set`    | **修改变量的值**                           | `(gdb) set var i=10``(gdb) set var ptr=0x0`                              |
| `whatis`       |            | 查看变量的类型                                   | `(gdb) whatis variable`                                                  |
| `ptype`        |            | 更详细地查看变量的类型定义                       | `(gdb) ptype struct_name`                                                |

4. **高级功能与多进程/线程调试**

| 命令                    | 简写     | 说明                                                       |
| ----------------------- | -------- | ---------------------------------------------------------- |
| `watch`               |          | **设置观察点** ，当变量被**写入**时中断        |
| `rwatch`              |          | 设置观察点，当变量被**读取**时中断                   |
| `awatch`              |          | 设置观察点，当变量被**读写**时中断                   |
| `thread apply all bt` |          | **打印所有线程的调用栈** （排查死锁/卡顿时极其有用） |
| `thread <id>`         |          | 切换到指定 ID 的线程                                       |
| `info threads`        | `i th` | 查看所有线程信息                                           |
| `attach <pid>`        |          | 附加到正在运行的进程（接进程PID）                          |
| `detach`              |          | 从已附加的进程分离                                         |
| `shell`               |          | 在 GDB 中执行 Shell 命令                                   |

### `set`命令调用技巧

1. 修改普通变量，指针指向的值

```txt
set var a = 20
set a = 20

set var *ptr = 100
set var ptr = 0x0
```

2. 修改数组元素

```txt
set var arr[0] = 99
set var set[1] = 'H'
```

2. 直接修改内存地址

```txt
# 修改单个字节
set *(char *)0x7fffffffe24c = 0x41
set *(char *)(0x7fffffffe250 + 1) = 'Y'
```

### X 命令

> x/\[数量\]\[格式\]\[单位\] \[内存地址或者表达式\]
> 参数说明：

+ 数量：要显示的内存单元数量
+ 格式：显示数据的格式 （10进制）
+ 单位：每个内存单元的大小
+ 地址：要查看的内存地址，如果省略则继续查看上次查看的位置

#### 单位（Size）详解

| 单位字符 | 含义     | 字节数 | 示例      |
| -------- | -------- | ------ | --------- |
| `b`    | byte     | 1字节  | `x/4xb` |
| `h`    | halfword | 2字节  | `x/4xh` |
| `w`    | word     | 4字节  | `x/4xw` |
| `g`    | giant    | 8字节  | `x/4xg` |

 **注意** ：单位大小可能因架构而异，上述是 x86/x86_64 的典型值。

#### 格式（Format）详解

| 格式字符 | 含义           | 示例       |
| -------- | -------------- | ---------- |
| `x`    | 十六进制       | `x/4xw`  |
| `d`    | 有符号十进制   | `x/4dw`  |
| `u`    | 无符号十进制   | `x/4uw`  |
| `o`    | 八进制         | `x/4ow`  |
| `t`    | 二进制         | `x/4tw`  |
| `f`    | 浮点数         | `x/4fw`  |
| `a`    | 地址           | `x/4aw`  |
| `i`    | 指令（反汇编） | `x/4i`   |
| `s`    | 字符串         | `x/s`    |
| `c`    | 字符           | `x/10cb` |

### 直接汇编并执行代码

使用assemble 命令

```txt
assemble $code_addr
mov $0x1, %eax
ret
end

disassemble main 查看 main 函数的反汇编指令
```

### layout 常用命令

```
tui disable/enable          # 关闭启用

(gdb) layout next    # 切换到下一个布局
(gdb) layout prev    # 切换到上一个布局

(gdb) focus cmd      # 焦点切换到命令窗口
(gdb) focus src      # 焦点切换到源代码窗口
(gdb) focus asm      # 焦点切换到汇编窗口
(gdb) focus reg      # 焦点切换到寄存器窗口

(gdb) backtrace full # 查看完整调用栈


# 重置显示 当出现异常的时候
(gdb) refresh
# 或
Ctrl + l
快速切换
Ctrl + x, a
```

### 如何使用gdb 调试汇编

> 环境：ubuntu-gcc13.3.0, WSL-Ubuntu, gdb15.0.50

存在如下汇编文件 `hello.s`

```asm
.intel_syntax noprefix
.data
msg:
  .ascii "Hello_World!\n"
  len = . - msg

.text
  .global _start

_start:
  # 写入
  mov rax, 1 # 系统调用 - write
  mov rdi, 1
  lea rsi, [msg]
  mov rdx, len
  syscall

  # exit
  mov rax, 60
  mov rdi, 0
  syscall
```

> as ./hello.s -o ./hello.o
> ld -g ./hello.o -o ./hello

使用 `info files` 查看文件所拥有的段

```bash
(gdb) info files
Symbols from "/home/yoyo/yoyo_dir/myobject/aboutPwn/hello".
Local exec file:
        `/home/yoyo/yoyo_dir/myobject/aboutPwn/hello', file type elf64-x86-64.
        Entry point: 0x401000
        0x0000000000401000 - 0x000000000040102f is .text
        0x0000000000402000 - 0x000000000040200d is .data
(gdb)
```

发现存在 代码段 `.text`和初始化数据段 `.data`
使用 `shell objdump -t ./hello` 查看./hello 具有哪些标识
进入_start

```txt
disas _start
# 或者
disas 0x401000, 0x40102f
```

得到

```asm
Dump of assembler code from 0x401000 to 0x40200d:
   0x0000000000401000 <_start+0>:       mov    $0x1,%rax
   0x0000000000401007 <_start+7>:       mov    $0x1,%rdi
   0x000000000040100e <_start+14>:      lea    0x402000,%rsi
   0x0000000000401016 <_start+22>:      mov    $0xd,%rdx
   0x000000000040101d <_start+29>:      syscall
   0x000000000040101f <_start+31>:      mov    $0x3c,%rax
   0x0000000000401026 <_start+38>:      mov    $0x0,%rdi
   0x000000000040102d <_start+45>:      syscall
   0x000000000040102f:
```

可以看出 在 `0x40100e` 出调用了指向 `.data`中的msg的地址也就是helloworld 这个字符串
使用 `x/10s 0x402000` 最近10行的字符串信息

```asm
(gdb) x/10s 0x402000
0x402000:       "Hello_World!\n"<error: Cannot access memory at address 0x40200d>
0x40200d:       <error: Cannot access memory at address 0x40200d>
0x40200d:       <error: Cannot access memory at address 0x40200d>
0x40200d:       <error: Cannot access memory at address 0x40200d>
0x40200d:       <error: Cannot access memory at address 0x40200d>
0x40200d:       <error: Cannot access memory at address 0x40200d>
0x40200d:       <error: Cannot access memory at address 0x40200d>
0x40200d:       <error: Cannot access memory at address 0x40200d>
0x40200d:       <error: Cannot access memory at address 0x40200d>
0x40200d:       <error: Cannot access memory at address 0x40200d>
```

在 `0x401000` 设置断点 `b *0x401000` 或者 `b _start`
使用 `stepi` `nexti` 或者 `si` `ni` 开始调试

## Liunx syscall 调用约定

在 Linux 系统中，系统调用（syscall）的约定根据架构不同而有所差异。以下以x86-64 架构系统调用约定为例子

1. **系统调用约定**

| 寄存器        | 用途       | 说明                 |
| ------------- | ---------- | -------------------- |
| **rax** | 系统调用号 | 指定要调用的系统调用 |
| **rdi** | 第一个参数 | arg0                 |
| **rsi** | 第二个参数 | arg1                 |
| **rdx** | 第三个参数 | arg2                 |
| **r10** | 第四个参数 | arg3                 |
| **r8**  | 第五个参数 | arg4                 |
| **r9**  | 第六个参数 | arg5                 |

2. **系统调用指令**

> syscall

3. **返回值**

* 成功：`rax` 包含返回值
* 错误：`rax` 包含负的错误码

### 常用的系统调用表

| 系统调用         | 编号 | 参数                              | 描述         |
| ---------------- | ---- | --------------------------------- | ------------ |
| **read**   | 0    | rdi=fd, rsi=buf, rdx=count        | 读取数据     |
| **write**  | 1    | rdi=fd, rsi=buf, rdx=count        | 写入数据     |
| **open**   | 2    | rdi=filename, rsi=flags, rdx=mode | 打开文件     |
| **close**  | 3    | rdi=fd                            | 关闭文件     |
| **exit**   | 60   | rdi=status                        | 退出进程     |
| **brk**    | 12   | rdi=addr                          | 调整堆指针   |
| **mmap**   | 9    | 复杂参数                          | 内存映射     |
| **munmap** | 11   | rdi=addr, rsi=length              | 取消内存映射 |

## `loop`指令使用

loop 指令 使用 rcx/ecx/cx 作为迭代器，在使用loop进行循环的时候需要注意防止syscall等破坏rcx 导致错误

下列是一个实例

> 使用环境：ubuntu-gcc13.3.0, WSL-Ubuntu 下同

```asm
.intel_syntax noprefix
.data
msg:
  .ascii "Hello_World\n"
  len = . - msg

.text
  .global _start
  # loop循环设置
_start:
  # 循环5次
  mov rcx, 5  
print_loop:
    # print
  mov rax, 1
  mov rdi, 1
  lea rsi, [msg]
  mov rdx, len
  
  # 防止系统调用破坏rcx 导致死循环
  mov r12, rcx

  syscall

  mov rcx, r12
  mov r12, 0
  loop print_loop

  # return 0;
  mov rax, 60
  mov rdi, 0
  syscall
```

### 其他循环的实现，使用jnz 指令

某些指令比如 `add`,`sub`,`or`,`and`,`xor`,`dec`,`inc` ,等会影响 `ZF`(零标志指令)，而 `jnz` 通过判断ZF 来决定是否跳转,所以可以通过这种方法来进行循环的实现

修改之前的循环代码

```asm
.intel_syntax noprefix
.data
msg:
  .ascii "Hello_World\n"
  len = . - msg

.text
  .global _start
  # loop循环设置
_start:
  # 循环5次
  mov r12, 5  
print_loop:
  # print
  mov rax, 1
  mov rdi, 1
  lea rsi, [msg]
  mov rdx, len
  syscall

  dec r12
  jnz print_loop

  # return 0;
  mov rax, 60
  mov rdi, 0
  syscall
```

## 在代码中使用栈 `Stack`

栈是一种先进后出的数据结构，在内存中专门用于：

+ 函数的调用管理
+ 局部变量的存储
+ 寄存器的保存
+ 参数的传递

### 帧指针 vs 栈指针

栈指针寄存器：`rsp` / `esp`

栈帧指针：`rbp` /`ebp`

| 寄存器            | 名称   | 作用             | 特点     |
| ----------------- | ------ | ---------------- | -------- |
| **ESP/RSP** | 栈指针 | 指向栈顶         | 动态变化 |
| **EBP/RBP** | 帧指针 | 指向当前栈帧基址 | 相对固定 |

### 总结：

程序栈是管理汇编编程的核心概念

+ 管理函数调用：返回地址，帧指针
+ 数据存储：局部变量，临时数据
+ 状态保存：寄存器保护
+ 必须小心管理：栈指针平衡，对齐要求
+ 性能要求：栈操作速度较快

### 实践：

栈的使用通过指令 `pop`, `push` 来决定

下面是一个使用栈的例子

```asm
.intel_syntax noprefix
.data
msg_1:
  .ascii "PUSH NUMBER IN TO STACK!\n"
  len_1 = . - msg_1

char_buf: .byte 0, 0x0A # 字符缓冲区域

.text
  .global _start
_start:
  mov rax, 1
  mov rdi, 1
  lea rsi, [msg_1]
  mov rdx, len_1
  syscall

  mov r12, len_1
stack_push_loop:
  mov rax, [msg_1 + r12 - 1]
  push rax
  dec r12
  jnz stack_push_loop

  # 打印栈中字符串
  mov r12, len_1
print_stack_str_loop:
  pop rax
  mov [char_buf], al      # 由于汇编语言没有自动截断功能，所以必须使用低位的寄存器 al 是 ax 的低位
  
  mov rax, 1
  mov rdi, 1
  lea rsi, [char_buf]
  mov rdx, 2
  syscall

  dec r12
  jnz print_stack_str_loop

  mov rax, 60
  mov rdi, 0
  syscall
```

这段代码实现了将字符串倒序压入栈中，然后通过循环拿出这一功能，注意 栈不能无限制压入数据一般来说Liunx 栈的大小为 **8MB**

## CALL & RET 指令

`call` 和 `ret`指令都是转移指令，都修改rip / eip / ip 等 他们经常杯共同用来实现子程序的设计，可以把它们理解为类似于高级语言的函数调用

### call&ret 语法格式

```asm
call target      # 直接调用
call [rax]       # 间接调用（通过寄存器）
call [rsp+8]     # 间接调用（通过内存）

ret           # 简单返回
ret imm16     # 返回并调整栈指针
```

### 总结:

* **`call`** ：保存返回地址 + 跳转到函数
* **`ret`** ：恢复返回地址 + 返回调用者
* **必须保持栈平衡** ：push/pop 数量匹配
* **性能关键** ：影响函数调用开销
* **调试友好** ：正确的帧指针管理便于调试

### 实践：

下面是一个使用 `call` & `ret` 的例子:

```asm
.intel_syntax noprefix
.data
msg:
  .ascii "This is a MSG!\n"
  msg_len = . - msg  

.text
  .global _start

_start:
  call _func_print_msg

  mov rax, 60
  mov rdi, 0
  syscall

_func_print_msg:
  mov rax, 1
  mov rdi, 1
  lea rsi, [msg]
  mov rdx, msg_len
  syscall
  ret
```
