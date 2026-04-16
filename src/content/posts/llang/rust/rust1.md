---
title: rust1
published: 2026-04-16
description: "所有权,借用, 结构体,枚举类，组织管理，错误处理"
tags: ["rust"]
category: language
draft: false
---

Rust 是由 Mozilla 主导开发的高性能编译型编程语言，遵循"安全、并发、实用"的设计原则。

## 所有权规则
所有权存在以下三个规则
+ Rust 中的每一个值都只有一个变量
+ 一次只存在一个所有者
+ 当程序不在程序的运行范围内

实际在运行时候的表现是，编译器会在程序变量结束的时候自动将变量的内存释放掉。
```rust
{
  let v = 123;
}
```
等效于
```c
{
  int* v = (int*)malloc(sizeof(int));
  v = 123;
  free(v);
}
```
同时除了自动释放垃圾资源，rust 对于资源的管理主要在于对变量存在所有权规范

```rust
fn func(s3: String) { ... }

fn test() {
    let s1 = String::from("a str");
    let s2 = s1;
    func(s2);
}
```
当 `s2` 得到 `s1` 的所有权之后，`s1` 自动废弃,同理在调用函数 `func` 之后 字符串 `"a str"`的所有权被转移到了 `s3` 中在`func` 结束之后`"a str"`被自动释放,因此如下的调用是不被允许的

```rust
fn test() {
    let s1 = String::from("a str");
    func(s1);   
    func(s1);  // 这里s1 已经不存在字符串的所有权了，所以编译器报错
}
```
:::note
+ 堆上数据（如 String）：所有权转移后，原变量被"销毁"（编译器禁止使用）
+ 栈上数据（如整数、&str）：直接复制，原变量仍然有效
:::

但是每次传递数据都必须转移所有权，会导致代码十分难用而且效率低下，其中最重要的问题是无法多次使用同一个数据, `rust` 引入了 ***借用*** 这一个特性, 等效于引用，只是 rust 强调临时使用这一个特性

在看如下例子
```rust

fn func(s: &String)

fn test()
{
    let s1 = String::from("abc");
    func(&s1);
    func(&s1);
}
```

:::note
+ 借用 = 引用，只是 Rust 用"借用"强调临时使用的语义
+ `&str2` 就是对 `str2` 的引用
+ 引用的核心规则：不能悬垂、不能冲突、不能超过所有者
+ 在字符串拼接中，`&str2 `让你可以借用 `str2 `的内容，而不获取所有权
:::

这种特性使得 Rust 通过编译器检查，从根本上消除了 `UAF` 和 `Double Free`这两类的内存安全漏洞. 

## 结构体

Rust中的结构体类似于 `json` 中 键值对句法定义
```rust
struct Person
{
    value: i32,
    name: String,
    age: i32
}

let person1 = Person {
  value: 1,
  name: String::from("XiaoMing"),
  age: 18,
}

let person2 = Person {
  value: 3
  ..person1  // 继承person1的大部分数据
}

// 元组结构体
struct RGB(i32, i32, i32);
let red = RGB(0xff, 0xff, 0xff);

```

### 结构体的所有权
+ 结构体必须掌握字段值所有权，因为结构体失效的时候会释放所有字段。
+ rust 的结构体和C一样 同样也存在对齐的概念,但是编译器会自动优化 当然也可以强制使用 `#[repr(C)]， #[repr(C, align(16))], ...` 修改进制编译器对于结构体内存的设置原则

### 结构体方法
rust 并不是面向对象的语言，所以不存在 `class` 关键字，但是它允许定义结构体方法，允许结构体拥有方法成员

```rust
struct Rectangle {
  width: u32,
  height: u32,
}

impl Rectangle {
  fn area(&self) -> u32 {
    return self.width * self.height;
  }
}
```
## 枚举
结构体给予你将字段和数据聚合在一起的方法,而枚举给予你一个途径去声明某个值是一个集合中的一员解决

```rust
enum IpAddrKind {
    V4,
    V6,
}
struct IpAddr {
    kind: IpAddrKind,
    address: String,
}
let home = IpAddr {
    kind: IpAddrKind::V4,
    address: String::from("127.0.0.1"),
};
let loopback = IpAddr {
    kind: IpAddrKind::V6,
    address: String::from("::1"),
};


// or 
enum IpAddr {
    V4(String),
    V6(String), 
} 
let home = IpAddr::V4(String::from("127.0.0.1"));
let loopback = IpAddr::V6(String::from("::1"));

```

### `match`
`match` 类似于 `switch` 用于处理枚举类，是实现分支结构
```rust
fn main() {
    enum Book {
        Papery {index: u32},
        Electronic {url: String},
    }
    
    let book = Book::Papery{index: 1001};
    let ebook = Book::Electronic{url: String::from("url...")};
    
    match book {
        Book::Papery { index } => {
            println!("Papery book {}", index);
        },
        Book::Electronic { url } => {
            println!("E-book {}", url);
        }
    }
}
```
当然 `match` 块是可以当作函数表达式来对待，也就说`match`块可以存在返回值,但是所有得返回值需要一样


### null 空值
`Option` 是 Rust 标准库中的枚举类，这个类用于填补Rust 不支持 null 引用的空白
```rust
enum Option<T> {
  Some(T),
  None,
}
```
:::note
注意： 初始值为空的 Option 必须明确类型
:::

### if let
`if let` 相当于是只区分两种情况的match 语法糖
```rust
let i = 0;
match i {
    0 => println!("zero"),
    _ => {},
}
// equl

let i = 0;
if let 0 = i {
    println!("zero");
}
```


## 组织管理
1. **Crate** 编译器一次处理一个**Crate**, 每个**Crate**是一个完整的库或者可执行文件
2. **package(包)** 使用 Cargo执行new 创建rust 工程时候，会创建一个`Cargo.toml`文件，而工程的实质就是一个包，是一个或者多个 **Crate**的集合
3. **模块** 类似于 `C++`的 `namespace` 但是里面的成员默认是私有的

```txt
包 (Package)
├── Cargo.toml
│
├── crate1: 库箱 (lib.rs)
│   ├── 模块: network
│   ├── 模块: database
│   │   ├── 子模块: postgres
│   │   └── 子模块: mysql
│   └── 模块: utils
│
├── crate2: 二进制箱 (main.rs)
│   ├── 模块: config
│   └── 模块: cli
│
└── crate3: 工具箱 (bin/tool.rs)
    └── 模块: tool_logic
```

同时可以使用 `use` 来导入对应的`crate`
```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}
pub fn eat_at_restaurant() {
    // 绝对路径
    crate::front_of_house::hosting::add_to_waitlist();
    // 相对路径
    front_of_house::hosting::add_to_waitlist();
}
```

## 错误处理

rust 中存在一套区别于 `try` 的错误处理机制, 对于程序中出现的错误，一般存在两种错误:**可恢复错误**和**不可恢复错误**

+ 可恢复错误： 比如文件访问失败错误(一般是文件已经被占用所导致的)
+ 不可恢复错误: 逻辑错误, 比如数组越界访问

### 对于不可恢复错误
可以在终端中使用如下命令
```text

// windows powershell
$env:RUST_BACKTRACE=1 ; cargo run


// linux bash
RUST_BACKTRACE=1 cargo run
```
rust 会展开运行的栈并输出所有的信息，然后程序依然会退出。上面的省略号省略了大量的输出信息，我们可以找到我们编写的 panic! 宏触发的错误

### 对于可恢复错误
通过 `Result<T, E>` 作为返回值来进行异常表达
```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");
    match f {
        Ok(file) => {
            println!("File opened successfully.");
        },
        Err(err) => {
            println!("Failed to open the file.");
        }
    }
}

// or 

fn main() {
    let f = File::open("hello.txt");
    if let Ok(file) = f {
        println!("File opened successfully.");
    } else {
        println!("Failed to open the file.");
    }
}
```
同时为了实现对于不同类型的错误进行不同处理可以使用 `kind()` 函数获取`Err`类型
```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_text_from_file(path: &str) -> Result<String, io::Error> {
    let mut f = File::open(path)?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}

fn main() {
    let str_file = read_text_from_file("hello.txt");
    match str_file {
        Ok(s) => println!("{}", s),
        Err(e) => {
            match e.kind() {
                io::ErrorKind::NotFound => {
                    println!("No such file");
                },
                _ => {
                    println!("Cannot read the file");
                }
            }
        }
    }
}
```
