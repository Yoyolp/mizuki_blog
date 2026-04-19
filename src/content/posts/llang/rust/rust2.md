---
title: rust2
published: 2026-04-18
description: "泛型和特性，生命周期，文件IO，面向对象, 宏， 智能指针"
tags: ["rust"]
category: language
draft: false
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


以下是cpp的版本
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

### 泛型枚举类和结构体

Rust 中的结构体和枚举类都可以实现泛型机制

```rust
struct Point<T> {
    x: T,
    y: T
}
impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}
```

## 特性

类似于接口 `interface`

```rust
trait Descriptive {
    fn describe(&self) -> String;
}

```

定义的了结构体 `Descriptive`中必须要有 `describe`方法, 使用方法如下

```rust
struct Person {
    name: String,
    age: u8
}

impl Descriptive for Person {
    fn describe(&self) -> String {
        format!("{} {}", self.name, self.age)
    }
}
```

使用 `for` 关键字

### 作为参数

很多情况下我们需要往一个函数中传入函数作为参数，而特性统一也可以作为参数类型

```rust
fn output(object: impl Descriptive) {
    println!("{}", object.describe());
}
```

任何实现了 `Descriptive` 特性的参数都可以传入，同时如果特性作为类型可以使用加号连接，同时如果实现关系过于复杂可以使用 `where` 简化

```rust
fn func<T: A + B, U: C + D>(t: T, u: U) -> i32
    where T: A + B,
          U: C + D
{
    ...
}
```

### 作为返回值

```rust
fn person() -> impl Descriptive {
    Person {
        name: String::from("Cali"),
        age: 24
    }
}
```

## 生命周期

生命周期，是为了保证引用永远不会指向已经被释放的内存，但是生命周期标注本质上不能延长对象的生命周期，真正控制变量的存在周期的是程序中的作用域。
所以生命周期实质上是显示标注出对象的存在周期，方便编译器检查

> 生命周期 = **显式告诉编译器**："这个引用能活多久，和其他引用的关系是什么"

```rust
// 没有标注 - 编译器需要猜测
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() { x } else { y }
}

// 有标注 - 明确告诉编译器
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
// 意思：x 和 y 至少活 'a 这么久，返回值也活 'a 这么久
```

### 静态生命周期

生命周期注释有一个特别的：'static 。所有用双引号包括的字符串常量所代表的精确数据类型都是 &'static str ，'static 所表示的生命周期从程序运行开始到程序运行结束。

## 文件 IO

### 接受文件的命令行参数

在 rust 中传递给程序的参数可以使用 `std::env`取出

```rust
fn main() {
    let args = std::env::args();
    println!("{:?}", args);
}
```

### 命令行输入

```rust
use std::io::stdin;

fn main() {
let mut str_buf = String::new();

    stdin().read_line(&mut str_buf)
        .expect("Failed to read line.");

    println!("Your input line is \n{}", str_buf);
}
```

### 文件fs

1. 文本读取写入

```rust

// 文件读取
use std::fs;

fn main() {
    let text = fs::read_to_string("D:\\text.txt").unwrap();
    println!("{}", text);
}


// 文件流读取
use std::io::prelude::*;
use std::fs;

fn main() {
    let mut buffer = [0u8; 5];
    let mut file = fs::File::open("D:\\text.txt").unwrap();
    file.read(&mut buffer).unwrap();
    println!("{:?}", buffer);
    file.read(&mut buffer).unwrap();
    println!("{:?}", buffer);
}


// 一次性写入
use std::fs;

fn main() {
    fs::write("D:\\text.txt", "FROM RUST PROGRAM")
        .unwrap();
}

// 流式写入
use std::io::prelude::*;
use std::fs::File;

fn main() {
    let mut file = File::create("D:\\text.txt").unwrap();
    file.write(b"FROM RUST PROGRAM").unwrap();
}

// openOptions
use std::io::prelude::*;
use std::fs::OpenOptions;

fn main() -> std::io::Result<()> {
  
    let mut file = OpenOptions::new()
            .read(true).write(true).open("D:\\text.txt")?;

    file.write(b"COVER")?;

    Ok(())
}
```

## 宏

在 Rust 中，使用 `macro_rules!` 关键字来定义声明式宏。

```rust
macro_rules! my_macro {
    // 模式匹配和展开
    ($arg:expr) => {
        // 生成的代码
        // 使用 $arg 来代替匹配到的表达式
    };
}

// 即如下模式

macro_rules! my_macro {
    (匹配模式1) => { 展开代码1 };  
    (匹配模式2) => { 展开代码2 };  
    (匹配模式3) => { 展开代码3 };  
    ...
}

```

### 1. 匹配模式
| 语法              | 匹配内容      | 示例                        |
| ----------------- | ------------- | --------------------------- |
| `$name:expr`    | 表达式        | `1+2`,`x`,`"hello"`   |
| `$name:ident`   | 标识符        | `foo`,`x`,`my_var`    |
| `$name:ty`      | 类型          | `i32`,`Vec<String>`     |
| `$name:path`    | 路径          | `std::vec::Vec`           |
| `$name:stmt`    | 语句          | `let x = 1;`              |
| `$name:block`   | 代码块        | `{ foo(); bar() }`        |
| `$name:literal` | 字面量        | `42`,`"hello"`,`true` |
| `$name:tt`      | 单个 Token 树 | 任何单个 token              |

### 2. 重复模式（固定语法）

```rust
$( ... ) 分隔符 重复符号
//  ^^^^    ^^    ^^^
//  内容   可选   必选
```

| 符号  | 含义        |
| ----- | ----------- |
| `*` | 0 次或多次  |
| `+` | 1 次或多次  |
| `?` | 0 次或 1 次 |


```rust
$(...),*    // 逗号分隔
$(...);+    // 分号分隔
$(...)*     // 无分隔符（空格隐含）
$(...)?     // 无分隔符
```

### 在展开中使用捕获的变量

```rust
$($element:expr),+  =>  {
    let mut v = Vec::new();
    $(
        v.push($element);   // 为每个匹配的元素生成 push
    )*
    v
}
```

### 重复展开的三种形式

| 形式          | 含义                      |
| ------------- | ------------------------- |
| `$($var)*`  | 重复 0 次或多次           |
| `$($var)+`  | 重复 1 次或多次           |
| `$($var),*` | 重复 0 次或多次，带分隔符 |


```rust
macro_rules! my_vec {
    // 规则1：空
    () => {
        Vec::new()
    };

    // 规则2：带元素（支持尾部逗号）
    ($($elem:expr),+ $(,)?) => {{
        let mut v = Vec::new();
        $(v.push($elem);)+
        v
    }};
}

// 使用
let v1 = my_vec![];           // 规则1
let v2 = my_vec![1, 2, 3];    // 规则2
let v3 = my_vec![1, 2, 3,];   // 规则2（尾部逗号被 $(,)? 匹配）
```

### 常见模式速查表

| 需求               | 模式                           |
| ------------------ | ------------------------------ |
| 匹配空             | `()`                         |
| 至少一个表达式     | `($($x:expr),+)`             |
| 零个或多个表达式   | `($($x:expr),*)`             |
| 支持尾部逗号       | `($($x:expr),+ $(,)?)`       |
| 匹配 key: value 对 | `($($k:ident: $v:expr),*)`   |
| 匹配不同类型混合   | `($($name:ident: $ty:ty),*)` |

---


### 重复模式可以嵌套

```rust
$($($inner:expr),+);*  // 嵌套重复
```

### 可选尾部逗号的两种写法


```rust
// 写法1：两个重复模式
($($x:expr),+ $(,)?)

// 写法2：使用 * 允许零次
($($x:expr),*)
```
### 块表达式必须用 `{{ }}` 或 `{ }`

```rust
=> { ... }      // 单层花括号
=> {{ ... }}    // 双层花括号（避免解析歧义）
```

### 捕获多个片段

```rust
// 捕获 3 个不同类型的片段
($func:ident($($arg:expr),*)) => { ... }
```

### 与正则表达式的类比

| 概念     | 正则       | Rust 宏    |
| -------- | ---------- | ---------- |
| 重复 0+  | `*`      | `*`      |
| 重复 1+  | `+`      | `+`      |
| 重复 0-1 | `?`      | `?`      |
| 分组     | `( )`    | `$( )`   |
| 捕获变量 | `\1`     | `$var`   |
| 分隔符   | `,``;`等 | `,``;`等 |


### 调试技巧

```bash
# 查看宏展开
cargo expand          # 需要安装 cargo-expand

# 或使用 trace_macros
#![feature(trace_macros)]
trace_macros!(true);
my_vec![1, 2, 3];     # 打印展开过程
trace_macros!(false);
```

```text
macro_rules! 定规则
模式匹配用 $( )
* + ? 控次数
expr ident 定类型
逗号分号作分隔
展开代码用 $var
```

### 总结
> **`macro_rules!` = 模式匹配（`$var:类型` + `$(...)*/+/?`）+ 代码生成（`$var` 填入模板）**

## 智能指针
智能指针通常封装了一个原始指针，并提供了一些额外的功能，比如引用计数、所有权转移、生命周期管理等。

在 Rust 中，标准库提供了几种常见的智能指针类型，例如 Box、Rc、Arc 和 RefCell。

:::note
一般存在如下智能指针
+ 当需要在堆上分配内存时，使用 `Box<T>`
+ 当需要多处共享所有权时，使用 `Rc<T> 或 Arc<T>`。
+ 当需要内部可变性时，使用 `RefCell<T>`。
+ 当需要线程安全的共享所有权时，使用 `Arc<T>`。
+ 当需要互斥访问数据时，使用 `Mutex<T>`。
+ 当需要读取-写入访问数据时，使用 `RwLock<T>`。
+ 当需要解决循环引用问题时，使用 `Weak<T>`。
:::
