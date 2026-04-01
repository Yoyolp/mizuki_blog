---
title: ida01
published: 2025-11-18
description: "ida的简单使用"
tags: ["tools"]
category: ctf
draft: false
---

## IDA
IDA Pro（Interactive Disassembler Professional）是由Hex-Rays公司开发的交互式反汇编工具，被广泛视为逆向工程领域的行业标准工具。该软件支持Windows、Linux、macOS操作系统，采用递归下降反汇编技术，具有可编程和可扩展特性，支持包括x86、ARM、MIPS、PowerPC、Z80、68000及c8051等数十种处理器架构。其核心功能包含静态分析、动态调试模块，提供Hex View窗口、Strings窗口等分析界面，兼容BinDiff插件并支持反编译生成类C代码 [4-6] [8]。
作为多平台逆向分析工具，IDA Pro最初以MS-DOS控制台程序形态出现，采用水印追踪与密钥文件认证机制，支持命名许可证与计算机许可证两种授权模式。软件安装包包含32位与64位版本，生成包含.id0、.id1等扩展名的分析数据库，通过ida.key密钥文件激活后可实现跨平台二进制文件分析与汇编代码重构

## 基本界面操作 1. 文件操作

   + 打开文件 File -> Open 或者拖拽文件到IDA 窗口
   + 新建数据库： 首次打开文件会创建IDB 数据库文件
   + 保存数据库：File -> Save Database
2. 视图导航

   + 反汇编视图：主代码显示窗口
   + 函数窗口： View -> Open subvuews -> Functions 显示所有函数
   + 字符串窗口： Shift + f12 显示所有字符串
   + 导入/ 导出表: View -> Open subviews -> Imports/Exports

## 常用快捷键

```txt
分析操作
空格键 - 在图形视图和文本视图间切换
F5 - 将汇编代码反编译为伪代码（需要 Hex-Rays 插件）
; - 添加注释
N - 重命名变量/函数
X - 查看交叉引用
G - 跳转到地址

分析操作
Esc - 返回上一个位置
Ctrl+Enter - 跳转到函数开始
Tab - 在伪代码和汇编代码间切换（Hex-Rays）


; 创建函数
P - 将选中的代码定义为函数

; 函数操作
Y - 修改函数原型
Ctrl+K - 查看栈变量布局



D - 将数据转换为字节、字、双字等
A - 将数据转换为字符串
* - 创建数组


Insert - 添加结构体
D - 在结构体中添加成员
Alt+Q - 将变量应用结构体类型


# 启动调试
F9 - 开始调试
F2 - 设置/取消断点
F7 - 单步进入
F8 - 单步跳过
```

## 实践：

### CTF+ [SignIn](https://www.ctfplus.cn/problem-detail/1975492213970309120/description)

#### 标签： 0xGame2025 Re

在下在题目提供的附件后 发现是一个 名为  **SignIn.exe** 的文件

将其拖入IDA 后 经过简单查找 就找到FLAG

![ida_01_1](/images/ida/01_1.png)

### CTF+ [EasyXor](https://www.ctfplus.cn/problem-detail/1975492219204800512/description)

#### 标签 0xGame2025 Re

将其拖入IDA 中经过 查找main文件
发现
![ida_01_1](/images/ida/01_2.png)

其中

```c
if ( (char)(v3 ^ key[n43_1 % _raputa0xGame2025_]) + n43 != str[n43] )
```

发现 这个for 循环将用户的输入经过一定加密来与 str 中的字符串进行逐一比对，结合信息:puts("Now please give me your flag:");
可以合理猜测 这个这个str 存储的就是flag

跳转到 str

```txt
data:0000000000004060 ; unsigned __int8 str[48]
.data:0000000000004060 str             db 42h, 1Ah, 39h, 17h, 1Dh, 9, 51h, 55h, 2Ch, 5Fh, 63h
.data:0000000000004060                                         ; DATA XREF: main+BC↑o
.data:000000000000406B                 db 0Ch, 0Dh, 16h, 62h, 27h, 55h, 64h, 55h, 26h, 6Dh, 6Ah 
.data:0000000000004076                 db 18h, 34h, 88h, 65h, 6Eh, 1Ch, 21h, 6Eh, 3Dh, 23h, 6Ah
.data:0000000000004081                 db 25h, 6Bh, 63h, 68h, 7Eh, 77h, 75h, 9Ah, 7Dh, 39h, 43h
.data:000000000000408C                 db 4 dup(0)
```

这是一段16进制数组

接下来由加密公式 来推导解密公式，
加密公式:

$$ (s[i] \oplus \text{key}[i\mod L]) + i = \text{flag}[i] $$

通过简单移项 和 异或运算的自逆性质
得出公式

$$
s[i] \oplus \text{key}[i\mod L] = \text{flag}[i] - i \\
(s[i] \oplus \text{key}[i\mod L]) \oplus \text{key}[i\mod L]
  = (\text{flag}[i] - i) \oplus \text{key}[i\mod L] \\
s[i] = (\text{flag}[i] - i) \oplus \text{key}[i\mod L] 
$$

由此得出pyhton 脚本

```c
key = b"raputa0xGame2025"
str_array = [
  0x42, 0x1A, 0x39, 0x17, 0x1D, 0x09, 0x51, 0x55, 0x2C, 0x5F, 0x63,
  0x0C, 0x0D, 0x16, 0x62, 0x27, 0x55, 0x64, 0x55, 0x26, 0x6D, 0x6A,
  0x18, 0x34, 0x88, 0x65, 0x6E, 0x1C, 0x21, 0x6E, 0x3D, 0x23, 0x6A,
  0x25, 0x6B, 0x63, 0x68, 0x7E, 0x77, 0x75, 0x9A, 0x7D, 0x39, 0x43
]

flag = ""
for i in range(44):
  s = (str_array[i] - i) ^ key[i % len(key)]
  flag += chr(s)

print(flag)
```

最终获得flag: 0xGame{6c74d39f-723f-42e7-9d7a-18e9508a655b}
