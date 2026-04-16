---
title: c06
published: 2025-11-13
description: "一些常用的转义符表, 一些常用的函数"
tags: ["c"]
category: language
draft: false
---

# C/C++ 格式转义符完整参考手册

## 整数类型格式符

| 格式符 | 含义                     | 示例                           |
| ------ | ------------------------ | ------------------------------ |
| `%d`   | 有符号十进制整数         | `printf("%d", 123);` → "123"  |
| `%i`   | 有符号十进制整数（同%d） | `printf("%i", 123);` → "123"  |
| `%u`   | 无符号十进制整数         | `printf("%u", 123u);` → "123" |
| `%o`   | 无符号八进制整数         | `printf("%o", 10);` → "12"    |
| `%x`   | 无符号十六进制（小写）   | `printf("%x", 255);` → "ff"   |
| `%X`   | 无符号十六进制（大写）   | `printf("%X", 255);` → "FF"   |

## 浮点数类型格式符


| 格式符 | 含义                     | 示例                                      |
| ------ | ------------------------ | ----------------------------------------- |
| `%f`   | 小数形式浮点数           | `printf("%f", 3.14);` → "3.140000"       |
| `%e`   | 指数形式（小写e）        | `printf("%e", 123.45);` → "1.234500e+02" |
| `%E`   | 指数形式（大写E）        | `printf("%E", 123.45);` → "1.234500E+02" |
| `%g`   | 自动选择%f或%e（较短者） | `printf("%g", 123.45);` → "123.45"       |
| `%G`   | 自动选择%f或%E（较短者） | `printf("%G", 0.0001);` → "0.0001"       |
| `%a`   | 十六进制浮点数（C99）    | `printf("%a", 3.14);` → "0x1.91eb86p+1"  |
| `%A`   | 十六进制浮点数（大写）   | `printf("%A", 3.14);` → "0X1.91EB86P+1"  |

## 字符和字符串格式符


| 格式符 | 含义     | 示例                                    |
| ------ | -------- | --------------------------------------- |
| `%c`   | 单个字符 | `printf("%c", 'A');` → "A"             |
| `%s`   | 字符串   | `printf("%s", "hello");` → "hello"     |
| `%p`   | 指针地址 | `printf("%p", &x);` → "0x7ffeeb39a82c" |

## 特殊格式符


| 格式符 | 含义                   | 示例                                            |
| ------ | ---------------------- | ----------------------------------------------- |
| `%%`   | 输出%字符本身          | `printf("%%");` → "%"                          |
| `%n`   | 将已输出字符数存入指针 | `int count; printf("hi%n", &count);` → count=2 |

## 长度修饰符


| 修饰符 | 含义          | 示例                                          |
| ------ | ------------- | --------------------------------------------- |
| `%hd`  | short int     | `short s=100; printf("%hd", s);`              |
| `%ld`  | long int      | `long l=1000L; printf("%ld", l);`             |
| `%lld` | long long int | `long long ll=1000LL; printf("%lld", ll);`    |
| `%lu`  | unsigned long | `unsigned long ul=1000UL; printf("%lu", ul);` |
| `%lf`  | double        | `double d=3.14; printf("%lf", d);`            |
| `%Lf`  | long double   | `long double ld=3.14L; printf("%Lf", ld);`    |

## 完整示例代码

```c
#include <stdio.h>

int main() {
    int integer = 42;
    float floating = 3.14159;
    double double_val = 2.71828;
    char character = 'X';
    char string[] = "Hello World";
    void* pointer = &integer;
  
    // 整数类型
    printf("十进制: %d\n", integer);        // 42
    printf("八进制: %o\n", integer);        // 52
    printf("十六进制: %x\n", integer);      // 2a
    printf("十六进制: %X\n", integer);      // 2A
  
    // 浮点类型
    printf("浮点数: %f\n", floating);       // 3.141590
    printf("科学计数: %e\n", floating);     // 3.141590e+00
    printf("自动选择: %g\n", 123.45);       // 123.45
    printf("自动选择: %g\n", 0.000123);     // 1.23e-04
  
    // 字符和字符串
    printf("字符: %c\n", character);        // X
    printf("字符串: %s\n", string);         // Hello World
    printf("指针: %p\n", pointer);          // 0x7ffd42a1b2ac
  
    // 宽度和精度控制
    printf("宽度8: |%8d|\n", integer);      // |      42|
    printf("左对齐: |%-8d|\n", integer);    // |42      |
    printf("前导零: |%08d|\n", integer);    // |00000042|
    printf("浮点精度: %.2f\n", floating);   // 3.14
  
    return 0;
}


```



## C 语言中常用的函数速查表

| 函数名字 | 描述                                                                                                                                                                                                                                                                           | 函数原型                                               |
| :------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------------------------------------------- |
| printf   | 格式化输出                                                                                                                                                                                                                                                                     | int printf(const char *format, ...);                   |
| scanf    | 获取用户输入                                                                                                                                                                                                                                                                   | int scanf(const char *format, ...);                    |
| getch    | 从控制台获取一个字符但是不回显到屏幕上，不需要按回车键立刻读取(不是标准库函数，通常Windows)                                                                                                                                                                                    | int getch(void);                                       |
| memset   | 是 C 语言标准库中的一个内存操作函数，用于将内存块的每个字节设置为特定的值 memset 将 ptr 指向的内存块的前 num 个字节每个都设置为 value 的值。                                                                                                                                   | void *memset(void *ptr, int value, size_t num);        |
| strcpy   | strcpy 用于将 源字符串 复制到 目标字符串，包括字符串的结束符 \0。 返回指向目标字符串的指针                                                                                                                                                                                     | char *strcpy(char *destination, const char *source);   |
| strcmp   | 比较字符串 =0 相等 <0 1小于2 >0 1大于3                                                                                                                                                                                                                                         | int strcmp(const char *str1, const char *str2);        |
| strcat   | 将源字符串 src 连接到目标字符串 dest 的末尾目标字符串必须有足够的空间容纳连接后的结果 源字符串的 \0 会被覆盖，并在新字符串末尾添加 \0。                                                                                                                                        | char *strcat(char *dest, const char *src);             |
| malloc   | 申请内存，失败返回NULL，否则返回内存地址，内存中为垃圾值，需要清零， 需要free释放                                                                                                                                                                                              | void* malloc(unsigned int num_bytes))                  |
| calloc   | nelem: 元素个数，elsize 元素长度 分配的内存会被初始化为零， free释放                                                                                                                                                                                                           | void* calloc(size_t nelem, size_t elsize);             |
| realloc  | 先判断当前指针是否由足够的地址空间，如果有，扩大mem_address指向的地址，并将mem_address返回，如果<br />空间不够，先安装newsize指定大小分配空间，将原有数据从头到尾考培到新分配的内存区域，而后释放原来mem_address所指的内存区域，<br />同时返回新分配的内存区域首地址，free释放 | void* realloc(void* mem_address, unigned int newsize); |
| free     | 释放内存                                                                                                                                                                                        