---
title: ewrok01
published: 2025-10-22
description: "第二次考核的错题"
tags: ["ctf", "c", "pwn"]
category: ctf
draft: false
---

## 3. 现在有一段神秘的伪代码，但是不用管这个函数实现了啥功能，根据上面回复出来的伪代码与下面没有进行回复的代码进行比较，恢复这个名为VM的结构体,结构如下:

```c
struct VM
{
  unsigned char mem[4096];
  
  unsigned short stack[256];
  unsigned short pc；
  unsigned short sp；
  unsigned short flags；

  unsigned short reg[3];
}
```

代码1

```c

unsigned __int16 __fastcall vm::step(VM *this)
{
  unsigned int n0xFC; // eax
  unsigned __int16 v2; // dx
  unsigned __int16 sp; // ax
  unsigned __int16 v4; // cx
  unsigned __int16 v5; // cx
  unsigned __int16 v6; // ax
  unsigned __int16 result; // ax
  unsigned __int16 v8; // [rsp+1Eh] [rbp-2h]

  v8 = this->regs[this->mem[this->pc + 2]];
  n0xFC = this->mem[this->pc];
  if ( n0xFC > 0xFC )
    return 0;
  if ( this->mem[this->pc] < 0xB0u )
  {
    if ( this->mem[this->pc] <= 0xA0u )
    {
      if ( this->mem[this->pc] >= 0x80u )
      {
        switch ( this->mem[this->pc] )
        {
          case 0x80u:
            this->regs[this->mem[this->pc + 1]] = v8;
            this->pc += 3;
            break;
          case 0x81u:
            this->regs[this->mem[this->pc + 1]] = (this->mem[this->pc + 3] << 8) | this->mem[this->pc + 2];
            this->pc += 4;
            break;
          case 0x90u:
            this->regs[this->mem[this->pc + 1]] += v8;
            this->pc += 3;
            break;
          case 0x91u:
            this->regs[this->mem[this->pc + 1]] += this->mem[this->pc + 2] | (this->mem[this->pc + 3] << 8);
            this->pc += 4;
            break;
          case 0xA0u:
            this->regs[this->mem[this->pc + 1]] -= v8;
            this->pc += 3;
            break;
          default:
            return 0;
        }
      }
      else if ( n0xFC == 0x4C )
      {
        sp = this->sp;
        this->sp = sp - 1;
        this->regs[this->mem[this->pc + 1]] = this->stack[sp]; /// STACK !!!!!!!!!!!
        this->pc += 2;
      }
      else if ( this->mem[this->pc] <= 0x4Cu )
      {
        if ( this->mem[this->pc] > 0x29u )
        {
          if ( n0xFC == 0x48 )
          {
            v2 = this->regs[this->mem[this->pc + 1]];
            this->stack[++this->sp] = v2;
            this->pc += 2;
          }
        }
        else if ( this->mem[this->pc] >= 8u )
        {
          switch ( this->mem[this->pc] )
          {
            case 8u:
              this->regs[this->mem[this->pc + 1]] = this->mem[v8];
              this->pc += 3;
              break;
            case 0xAu:
              this->regs[this->mem[this->pc + 1]] = (this->mem[v8 + 1] << 8) | this->mem[v8];
              this->pc += 3;
              break;
            case 0x18u:
              this->mem[this->regs[this->mem[this->pc + 1]]] = v8;
              this->pc += 3;
              break;
            case 0x1Au:
              this->mem[this->regs[this->mem[this->pc + 1]]] = v8;
              this->mem[this->regs[this->mem[this->pc + 1]] + 1] = HIBYTE(v8);
              this->pc += 3;
              break;
            case 0x28u:
              this->flags = v8 == this->regs[this->mem[this->pc + 1]];
              this->pc += 3;
              break;
            case 0x29u:
              this->flags = this->regs[this->mem[this->pc + 1]] == (this->mem[this->pc + 2]
                                                                  | (this->mem[this->pc + 3] << 8));
              this->pc += 4;
              break;
            default:
              return 0;
          }
        }
      }
    }
    return 0;
  }
  switch ( this->mem[this->pc] )
  {
    case 0xB0u:
      this->regs[this->mem[this->pc + 1]] ^= v8;
      this->pc += 3;
      return 0;
    case 0xC1u:                           //  uint16        |          
      this->regs[this->mem[this->pc + 1]] *= this->mem[this->pc + 2] | (unsigned __int16)(this->mem[this->pc + 3] << 8);
      this->pc += 4;
      return 0;
    case 0xE0u:
      this->pc = this->mem[this->pc + 1] | (this->mem[this->pc + 2] << 8);
      return 0;
    case 0xE8u:
      if ( this->flags )
        this->pc = this->mem[this->pc + 1] | (this->mem[this->pc + 2] << 8);
      else
        this->pc += 3;
      return 0;
    case 0xECu:
      if ( this->flags )
        this->pc += 3;
      else
        this->pc = this->mem[this->pc + 1] | (this->mem[this->pc + 2] << 8);
      return 0;
    case 0xEEu:
      v4 = this->pc + 2;
      this->stack[++this->sp] = v4;
      this->pc = this->regs[this->mem[this->pc + 1]];
      return 0;
    case 0xEFu:
      v5 = this->pc + 3;
      this->stack[++this->sp] = v5;
      this->pc = this->mem[this->pc + 1] | (this->mem[this->pc + 2] << 8);
      return 0;
    case 0xF0u:
      v6 = this->sp;
      this->sp = v6 - 1;
      this->pc = this->stack[v6];
      return 0;
    case 0xF8u:
      result = -1;
      break;
    case 0xFCu:
      if ( this->regs[0] )
        this->regs[0] = read(0, &this->mem[this->regs[1]], this->regs[2]);
      else
        write(1, &this->mem[this->regs[1]], this->regs[2]);
      ++this->pc;
      return 0;
    default:
      return 0;
  }
  return result;
}
```

代码2

```c
unsigned __int16 __fastcall vm::step(unsigned __int8 *this)
{
  unsigned int n0xFC; // eax
  __int16 v2; // dx
  unsigned __int16 v3; // ax
  __int16 v4; // cx
  __int16 v5; // cx
  unsigned __int16 v6; // ax
  unsigned __int16 result; // ax
  unsigned __int16 v8; // [rsp+1Eh] [rbp-2h]

  v8 = *((_WORD *)this + *(this + *((unsigned __int16 *)this + 2304) + 2) + 2307);
  n0xFC = *(this + *((unsigned __int16 *)this + 2304));
  if ( n0xFC > 0xFC )
    return 0;
  if ( *(this + *((unsigned __int16 *)this + 2304)) < 0xB0u )
  {
    if ( *(this + *((unsigned __int16 *)this + 2304)) <= 0xA0u )
    {
      if ( *(this + *((unsigned __int16 *)this + 2304)) >= 0x80u )
      {
        switch ( *(this + *((unsigned __int16 *)this + 2304)) )
        {
          case 0x80u:
            *((_WORD *)this + *(this + *((unsigned __int16 *)this + 2304) + 1) + 2307) = v8;
            *((_WORD *)this + 2304) += 3;
            break;
          case 0x81u:
            *((_WORD *)this + *(this + *((unsigned __int16 *)this + 2304) + 1) + 2307) = (*(this
                                                                                          + *((unsigned __int16 *)this
                                                                                            + 2304)
                                                                                          + 3) << 8)
                                                                                       | *(this
                                                                                         + *((unsigned __int16 *)this
                                                                                           + 2304)
                                                                                         + 2);
            *((_WORD *)this + 2304) += 4;
            break;
          case 0x90u:
            *((_WORD *)this + *(this + *((unsigned __int16 *)this + 2304) + 1) + 2307) += v8;
            *((_WORD *)this + 2304) += 3;
            break;
          case 0x91u:
            *((_WORD *)this + *(this + *((unsigned __int16 *)this + 2304) + 1) + 2307) += *(this
                                                                                          + *((unsigned __int16 *)this
                                                                                            + 2304)
                                                                                          + 2)
                                                                                        | (*(this
                                                                                           + *((unsigned __int16 *)this
                                                                                             + 2304)
                                                                                           + 3) << 8);
            *((_WORD *)this + 2304) += 4;
            break;
          case 0xA0u:
            *((_WORD *)this + *(this + *((unsigned __int16 *)this + 2304) + 1) + 2307) -= v8;
            *((_WORD *)this + 2304) += 3;
            break;
          default:
            return 0;
        }
      }
      else if ( n0xFC == 0x4C )
      {
        v3 = *((_WORD *)this + 2305);
        *((_WORD *)this + 2305) = v3 - 1;
        *((_WORD *)this + *(this + *((unsigned __int16 *)this + 2304) + 1) + 2307) = *((_WORD *)this + v3 + 2048);
        *((_WORD *)this + 2304) += 2;
      }
      else if ( *(this + *((unsigned __int16 *)this + 2304)) <= 0x4Cu )
      {
        if ( *(this + *((unsigned __int16 *)this + 2304)) > 0x29u )
        {
          if ( n0xFC == 0x48 )
          {
            v2 = *((_WORD *)this + *(this + *((unsigned __int16 *)this + 2304) + 1) + 2307);
            *((_WORD *)this + (unsigned __int16)++*((_WORD *)this + 2305) + 2048) = v2;
            *((_WORD *)this + 2304) += 2;
          }
        }
        else if ( *(this + *((unsigned __int16 *)this + 2304)) >= 8u )
        {
          switch ( *(this + *((unsigned __int16 *)this + 2304)) )
          {
            case 8u:
              *((_WORD *)this + *(this + *((unsigned __int16 *)this + 2304) + 1) + 2307) = *(this + v8);
              *((_WORD *)this + 2304) += 3;
              break;
            case 0xAu:
              *((_WORD *)this + *(this + *((unsigned __int16 *)this + 2304) + 1) + 2307) = (*(this + v8 + 1) << 8)
                                                                                         | *(this + v8);
              *((_WORD *)this + 2304) += 3;
              break;
            case 0x18u:
              *(this + *((unsigned __int16 *)this + *(this + *((unsigned __int16 *)this + 2304) + 1) + 2307)) = v8;
              *((_WORD *)this + 2304) += 3;
              break;
            case 0x1Au:
              *(this + *((unsigned __int16 *)this + *(this + *((unsigned __int16 *)this + 2304) + 1) + 2307)) = v8;
              *(this + *((unsigned __int16 *)this + *(this + *((unsigned __int16 *)this + 2304) + 1) + 2307) + 1) = HIBYTE(v8);
              *((_WORD *)this + 2304) += 3;
              break;
            case 0x28u:
              *((_WORD *)this + 2306) = v8 == *((_WORD *)this + *(this + *((unsigned __int16 *)this + 2304) + 1) + 2307);
              *((_WORD *)this + 2304) += 3;
              break;
            case 0x29u:
              *((_WORD *)this + 2306) = *((unsigned __int16 *)this
                                        + *(this + *((unsigned __int16 *)this + 2304) + 1)
                                        + 2307) == (*(this + *((unsigned __int16 *)this + 2304) + 2)
                                                  | (*(this + *((unsigned __int16 *)this + 2304) + 3) << 8));
              *((_WORD *)this + 2304) += 4;
              break;
            default:
              return 0;
          }
        }
      }
    }
    return 0;
  }
  switch ( *(this + *((unsigned __int16 *)this + 2304)) )
  {
    case 0xB0u:
      *((_WORD *)this + *(this + *((unsigned __int16 *)this + 2304) + 1) + 2307) ^= v8;
      *((_WORD *)this + 2304) += 3;
      return 0;
    case 0xC1u:
      *((_WORD *)this + *(this + *((unsigned __int16 *)this + 2304) + 1) + 2307) *= *(this
                                                                                    + *((unsigned __int16 *)this + 2304)
                                                                                    + 2)
                                                                                  | (unsigned __int16)(*(this + *((unsigned __int16 *)this + 2304) + 3) << 8);
      *((_WORD *)this + 2304) += 4;
      return 0;
    case 0xE0u:
      *((_WORD *)this + 2304) = *(this + *((unsigned __int16 *)this + 2304) + 1)
                              | (*(this + *((unsigned __int16 *)this + 2304) + 2) << 8);
      return 0;
    case 0xE8u:
      if ( *((_WORD *)this + 2306) )
        *((_WORD *)this + 2304) = *(this + *((unsigned __int16 *)this + 2304) + 1)
                                | (*(this + *((unsigned __int16 *)this + 2304) + 2) << 8);
      else
        *((_WORD *)this + 2304) += 3;
      return 0;
    case 0xECu:
      if ( *((_WORD *)this + 2306) )
        *((_WORD *)this + 2304) += 3;
      else
        *((_WORD *)this + 2304) = *(this + *((unsigned __int16 *)this + 2304) + 1)
                                | (*(this + *((unsigned __int16 *)this + 2304) + 2) << 8);
      return 0;
    case 0xEEu:
      v4 = *((_WORD *)this + 2304) + 2;
      *((_WORD *)this + (unsigned __int16)++*((_WORD *)this + 2305) + 2048) = v4;
      *((_WORD *)this + 2304) = *((_WORD *)this + *(this + *((unsigned __int16 *)this + 2304) + 1) + 2307);
      return 0;
    case 0xEFu:
      v5 = *((_WORD *)this + 2304) + 3;
      *((_WORD *)this + (unsigned __int16)++*((_WORD *)this + 2305) + 2048) = v5;
      *((_WORD *)this + 2304) = *(this + *((unsigned __int16 *)this + 2304) + 1)
                              | (*(this + *((unsigned __int16 *)this + 2304) + 2) << 8);
      return 0;
    case 0xF0u:
      v6 = *((_WORD *)this + 2305);
      *((_WORD *)this + 2305) = v6 - 1;
      *((_WORD *)this + 2304) = *((_WORD *)this + v6 + 2048);
      return 0;
    case 0xF8u:
      result = -1;
      break;
    case 0xFCu:
      if ( *((_WORD *)this + 2307) )
        *((_WORD *)this + 2307) = read(0, this + *((unsigned __int16 *)this + 2308), *((unsigned __int16 *)this + 2309));
      else
        write(1, this + *((unsigned __int16 *)this + 2308), *((unsigned __int16 *)this + 2309));
      ++*((_WORD *)this + 2304);
      return 0;
    default:
      return 0;
  }
  return result;
}
```

通过分析

```c
  unsigned int n0xFC; // eax
  __int16 v2; // dx
  unsigned __int16 v3; // ax
  __int16 v4; // cx
  __int16 v5; // cx
  unsigned __int16 v6; // ax
  unsigned __int16 result; // ax
  unsigned __int16 v8; // [rsp+1Eh] [rbp-2h]
```

和结构体各个变量的交互发现
结构体VM中 存在**pc sp flags stack[] regs[] mem[]** 这6个成员

通过程序基传入指针**this** 进行指针运算发现 推测该this指针为 传入VM参数的基地址
先推出**pc sp flags** 所在相对于基地址的偏移量,

pc:

从程序

```c
this->pc += 2;
*((_WORD *)this + 2304) += 2;
```

发现 this->pc 等价于*((_WORD *)this + 2304)，所以可以知道 2034 为 pc 相当于基地址的偏移量

sp:
从程序

```c
v6 = this->sp;
v6 = *((_WORD *)this + 2305);
```

同理发现 sp 的偏移量为 2305

flags:

```c
this->flags = v8 == this->regs[this->mem[this->pc + 1]];
*((_WORD *)this + 2306) = v8 == *((_WORD *)this + *(this + *((unsigned __int16 *)this + 2304) + 1) + 2307);
```

同理发现 flags 的偏移量是 2306

stack:
程序

```c
this->stack[sp];
*((_WORD *)this + 2305 + 2048);

this->regs[this->mem[this->pc + 1]] = this->stack[sp];
*((_WORD *)this + *(this + *((unsigned __int16 *)this + 2304) + 1) + 2307) = *((_WORD *)this + 2305 + 2048);

...
```

发现所有 关于 stack的访问 存在2048这个数字，由此可以知道 2048 是 stack的基址, 而他到 最近的pc(2304) 相差 256， 也符合栈的
所以stack大小为 256,

mem:

通过

```c
n0xFC = *(this + *((unsigned __int16 *)this + 2304));
n0xFC = this->mem[this->pc];
```

发现 mem是从 0 开始的,到最近的 stack(2048) 结束,但是通过代码中reg 的赋值为

```c
this->regs[this->mem[this->pc + 1]] = (this->mem[this->pc + 3] << 8) | this->mem[this->pc + 2];
```

发现:
存放一个 regs\[i\] 的存放需要两个 mem 分别放在 高8，和低8位，而regs\[i\] 的类型是 **uint16(16位)**
所以合理猜测 mem 的类型 为**uint8(8位)**, 所以 0-2048 一共占用 4096 个字节, 而一个 **uint8** 占用一个字节
所以mem 的长度 4096

regs:

```c
*((_WORD *)this + 2307)
this->regs[0]
```

通过以上代码发现 regs\[0\] 的地址是 2307, 但是没有找到明显的边界,但是确定regs的长度一定大于等于3

汇总以上信息

```txt
0:      uint8  -> unsigned char mem[4096]   

2048:   uint16 -> unsigned short stack[256]   
2304:   uint16 -> unsigned short pc
2305:   uint16 -> unsigned short sp
2306:   uint16 -> unsigned short flags

2307:   uint16 -> unsigned short reg[3+]
```

由于最后一个reg 看不出所以需要自己补上
从表达式

```c
this->regs[this->mem[this->pc + 1]] ^= v8;
```

发现理论上 this->mem 的值为 0~255 一共256个值
所以 regs 在设计上应该也要能装下 这些索引，所以 reg的大小应该为 256

所以最终得到的结果是

```c
struct VM
{
  unsigned char mem[4096];
  
  unsigned short stack[256];
  unsigned short pc；
  unsigned short sp；
  unsigned short flags；

  unsigned short reg[256];
}

```

## 1. 定义一个字符串，实现一个功能要求输入字符串可以输出字符串的长度，不要使用strlen等函数

实现效果如下:

```c
input:10d2n821 ma&@&@ 93cdw1!
string Length:24

```

由于C语言中的字符串是以\\0 为结尾的所以通过这个特性我们可以实现字符串长度的获取

```c
size_t mystrlen(char* arr)
{
  int l = 0;
  char* ptr = arr;
  while(*(ptr++) != '\0')
  {
    l++;
  }
  return l;
}

int main(void)
{
  char* input_str = (char*)malloc(sizeof(char) * 1000);
  
  printf("input:");
  if(fgets(input_str, sizeof(char) * 1000, stdin))
  {
    printf("string Length: %d", mystrlen(input_str)); 
  }
  free(input_str);
  return 0;

}
```
:::important
注意： 这里不适用scanf 是因为scanf 在读取到 空白符 制表符 和换行符等，会直接截断
:::