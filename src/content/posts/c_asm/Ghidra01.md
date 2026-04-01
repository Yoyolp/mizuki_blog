---
title: Ghidra
published: 2025-11-18
description: "Ghidra的简单使用"
tags: ["tools"]
category: ctf
draft: false
---


Ghidra 是一个由美国国家安全局（NSA）开发并开源，的逆向安全工具,是一个软件逆向工程（SRE）框架 包含:
 + 反汇编器：将机器代码转换为汇编代码
 + 反编译器：将机器代码转换为 C-like 伪代码
 + 图形化分析工具：数据可视化、控制流分析等


基本工作流程:
1. 创建项目
```txt
File → New Project → Non-Shared Project
→ 输入项目名称 → 选择存储位置
```

2. 导入二进制文件
 + 拖拽文件到项目窗口，或
 + File → Import File
 + 选择正确的文件格式（ELF、PE、Mach-O 等）

3. 反编译并分析代码

## 示例：
#### 环境: Window11, Ghidra12.1, llvm-mingw-20250613-msvcrt-i686

编写一个简单的C语言程序 **Hello.c**
```c
#include <stdio.h>

int main(void)
{
  printf("Hello World\n");
  return 0;
}

// gcc ./Hello.c -o ./Hello.exe -s
```
在反编译后的结果:

伪C代码
![fake_C](/images/Ghidra/01/fake_c.png)

汇编
![asm_c](/images/Ghidra/01/asm_c.png)


接下来尝试稍微复杂点的算法，以如下冒泡排序算法(**sort.c**)实现为例子:
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define MAX_SIZE 64

// 这是一个简易的冒泡排序实现
int* bsort_int(const int *list, int size) {
  int *arr = (int*)malloc(size * sizeof(int));

  if (arr == NULL) {
    return NULL;
  }

  for(int i = 0; i < size; i++) {
    arr[i] = list[i];
  }

  bool flag = true;
  while (flag) {
    flag = false;
    for(int i = 0; i < size - 1; i++) {
      if(arr[i] < arr[i + 1]) {
        int temp = arr[i];
        arr[i] = arr[i + 1];
        arr[i + 1] = temp;
        flag = true;
      }
    }
  }
  return arr;
}

int main(void)
{
  int numbers[MAX_SIZE];
  int n;

  printf("请输入数字最多 %d 个", MAX_SIZE);
  scanf("%d", &n);

  if (n > MAX_SIZE || n <= 0) {
    printf("输入数量无效\n");
    return 1;
  }

  printf("请输入 %d 个数字: \n", n);
  for(int i = 0; i < n; i++) {
    scanf("%d", &numbers[i]);
  }

  printf("接下来进行排序...\n");
  int* num_sorted = bsort_int(numbers, n);

  for(int i = 0; i < n; i++) {
    printf("%d\t", num_sorted[i]);
  }

  return 0;
}

// gcc ./sort.c -o ./sort.exe
```

反编译结果：
![fake_c](/images/Ghidra/01/fake_c2.png)
![fake_c](/images/Ghidra/01/fake_c3.png)

伪C代码中的各种类型：
 + **undefined** 为未知类型 1 Byte
 + **undefined1** 为1Byte的未知类型
 + **undefined2** 为2Byte的未知类型
 + **undefined4** 为4Byte的未知类型
 + ...
 + uint           无符号整型
 + ulong          无符号长整型
 + ...



