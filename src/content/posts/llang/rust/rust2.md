---
title: rust2
published: 2026-04-16
description: "泛型和特性，生命周期，文件IO，集合&字符串，面向对象, 宏， 智能指针， 异步和并发"
tags: ["rust"]
category: language
draft: true
---

## 泛型

### 泛型函数
相较于C++的 `template` rust 使用尖括号来表示泛型
和C++类似 rust 在底层会自动生成对应类型的函数的实现，即在底层将抽象化的泛型，退化回不同类型的函数实例实现

如下代码,分别是C++ 和 rust在泛型方面的底层实现
```rust
use std::ops::Add;

fn add<T: Add<Output = T> + Copy>(a: &T, b: &T) -> T {
    *a + *b
}

pub fn main() {
    let c = add(&1, &1);
    let d = add(&1.1, &1.2);
    println!("{c}, {d}");
}
```

在底层会生成处理两个对象的 `add` 函数
```text
add_i32();
add_f64();
```
即同一个泛型函数add实际生成了两个独立的机械码函数
```asm
; f64 版本的 add
example::add::h4d56ee231db6c54f:
    movsd   xmm0, qword ptr [rdi]    ; 从指针加载 f64
    movsd   xmm1, qword ptr [rsi]    
    call    <f64 as core::ops::arith::Add>::add::h480a25c990bb15c3
    ret

; i32 版本的 add  
example::add::h4d6c5e6f2e6494f1:
    mov     edi, dword ptr [rdi]     ; 从指针加载 i32
    mov     esi, dword ptr [rsi]
    call    <i32 as core::ops::arith::Add>::add::h2c7527d51bc18883
    ret
...
```

```cpp
template <typename T>
auto add(T a, T b) -> decltype(a + b)
{
  return a + b;
}

int main() {
    int a = add(1, 1);
    float b = add(1.1, 1.1);

    return 0;
}
```

```asm
main:
        push    rbp
        mov     rbp, rsp
        sub     rsp, 16
        mov     esi, 1
        mov     edi, 1
        call    decltype(fp + fp0) add<int>(int, int)
        mov     DWORD PTR [rbp-4], eax
        movsd   xmm0, QWORD PTR .LC0[rip]
        mov     rax, QWORD PTR .LC0[rip]
        movapd  xmm1, xmm0
        movq    xmm0, rax
        call    decltype(fp + fp0) add<double>(double, double)
        cvtsd2ss        xmm0, xmm0
        movss   DWORD PTR [rbp-8], xmm0
        mov     eax, 0
        leave
        ret
decltype(fp + fp0) add<int>(int, int):
        push    rbp
        mov     rbp, rsp
        mov     DWORD PTR [rbp-4], edi
        mov     DWORD PTR [rbp-8], esi
        mov     edx, DWORD PTR [rbp-4]
        mov     eax, DWORD PTR [rbp-8]
        add     eax, edx
        pop     rbp
        ret
decltype(fp + fp0) add<double>(double, double):
        push    rbp
        mov     rbp, rsp
        movsd   QWORD PTR [rbp-8], xmm0
        movsd   QWORD PTR [rbp-16], xmm1
        movsd   xmm0, QWORD PTR [rbp-8]
        addsd   xmm0, QWORD PTR [rbp-16]
        pop     rbp
        ret
.LC0:
        .long   -1717986918
        .long   1072798105
```








