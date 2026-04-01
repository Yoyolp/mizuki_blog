---
title: c06
published: 2025-11-13
description: "一些常用的转义符表"
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
