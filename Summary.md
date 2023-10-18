# Rust 第一阶段学习总结

我于 2023 年 10 月 13 日完成全部练习，在这里总结一下 Rust 的学习经历。本次总结主要借助于[Rust 程序设计语言（简体中文版）](https://kaisery.github.io/trpl-zh-cn/title-page.html)的组织顺序。

# 一、安装与 Cargo 包

第一节主要介绍 Rust 的安装以及包管理工具 cargo。

## 安装·

IDE 采用 VS Code，这里的重点是下载 rust。简单的说就是执行一个语句：

```bash
curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

等待安装结束即可。执行

```bash
rustc --version
```

如果能够看到版本号，证明安装没有问题。

## Cargo 包管理工具

直接使用

```bash
cargo new hello_cargo
```

就可以创建一个名称为 hello_cargo 的项目。配置文件为 Cargo.toml. Rustlings 练习已经采用 cargo 进行管理，查看其 toml 文件，可以发现如下几个部分

```toml
[package]
name = "rustlings"
description = "Small exercises to get you used to reading and writing Rust code!"
version = "5.5.1"
......

[dependencies]
argh = "0.1"
indicatif = "0.16"
console = "0.15"
notify = "4.0"
......

[[bin]]
name = "rustlings"
path = "src/main.rs"

[dev-dependencies]
assert_cmd = "0.11.0"
predicates = "1.0.1"
glob = "0.3.0"


```

这个 toml 文件依次介绍了本项目的若干信息（package）、依赖（deoendencies）、执行命令（bin）、开发依赖（dev-dependencies）。与其他包管理工具（如 Maven）类似，需要什么依赖直接往里写就行了，它会自动下载。

下面介绍下 cargo 的常见命令：

```bash
cargo new #创建项目
cargo build # 编译本项目
cargo run  # 一步构建并运行项目
cargo check # 在不生成二进制文件的情况下构建项目来检查错误
```

# 二、Rust 基本语法(常见编程概念)

这一部分没有什么难的，熟悉一下 Rust 最最基本的语法就行。

## 变量 与 可变性

```rust
let a  = 10; // 不可变变量
let mut b = 10; // 可变变量

// 常量，用const声明，必须注明值的类型，常量在整个程序生命周期中都有效
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;

//  隐藏（Shadowing）， 此时任何使用该变量名的行为中都会视为是在使用第二个变量，直到作用域结束。
let spaces = "   ";
let spaces = spaces.len();
```

## 数据类型

一共有两类数据类型子集：标量（scalar）和复合（compound）。

### 标量

标量（scalar）类型代表一个单独的值。Rust 有四种基本的标量类型：整型、浮点型、布尔类型和字符类型。

需要指出的是，我们用单引号声明 char 字面量，使用双引号声明字符串字面量。Rust 的 char 类型的大小为四个字节，并代表了一个 Unicode 标量值。

```rust
// 整型
let a: u32 = 19;
let b: isize = 1000;

// 浮点型
//1.默认为float64类型
let x = 2.0;
//2.还有float32类型可供选择
let y: f32 = 3.0;

// 布尔型
let t = true;
let f: bool = false;

// 字符型
let c = 'z';
let z: char = 'ℤ';
let heart_eyed_cat = '😻';
```

### 复合

复合类型主要有两种：元组和数组。

```rust
// 元组类型
let tup: (i32, f64, u8) = (500, 6.4, 1);

//元组访问
let five_hundred = tup.0;

// 数组类型
let a = [1, 2, 3, 4, 5];
let b: [i32; 5] = [1, 2, 3, 4, 5];
let c = [3; 5];

// 数组访问
let first = a[0];
```

## 函数

基本模式如下：

```rust
fn main() {
    let x = plus_one(5);

    println!("The value of x is: {x}");
}

fn plus_one(x: i32) -> i32 {
    x + 1
}

```

## 控制流

### if 条件判断

条件判断基本模式如下：

```rust
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```

<b>if</b> 还可以用于三元表达式：

```rust
let condition = true;
let number = if condition { 5 } else { 6 };
```

### 循环

使用 loop 重复执行代码

```rust
loop {
        println!("again!");
    }
```

从循环返回值

```rust
let mut counter = 0;

let result = loop {
    counter += 1;

    if counter == 10 {
        break counter * 2;
    }
};
```

while 条件循环

```rust
let mut number = 3;

while number != 0 {
    println!("{number}!");

    number -= 1;
}
```

使用 for 遍历集合

```rust
let a = [10, 20, 30, 40, 50];

for element in a {
    println!("the value is: {element}");
}
```

# 二、所有权

可以说是 rust 的第一个难点了。

首先介绍一下所有权的规则：

<li>Rust 中的每一个值都有一个 所有者（owner）。</li>
<li>值在任一时刻有且只有一个所有者。</li>
<li>当所有者（变量）离开作用域，这个值将被丢弃。</li>

&nbsp;

前面介绍的类型都是已知大小的，可以存储在栈中，并且当离开作用域时被移出栈，如果代码的另一部分需要在不同的作用域中使用相同的值，可以快速简单地复制它们来创建一个新的独立实例。不过我们需要寻找一个存储在堆上的数据来探索 Rust 是如何知道该在何时清理数据的。

字符串字面值来说，我们在编译时就知道其内容，所以文本被直接硬编码进最终的可执行文件中。  但是，String 类型管理被分配到堆上的数据，所以能够存储在编译时未知大小的文本。

首先看一下变量与数据交互的方式。

## 移动

```rust
fn main() {
    let x = 5;
    let y = x;
}
```

这两个变量都能够被访问，因为基本类型存储在栈中。

```rust
let s1 = String::from("hello");
let s2 = s1;
```

这两个变量只有 s2 能够被访问，这里相当于在栈内存进行浅复制（在 Rust 里叫移动）,为了避免二次释放等问题，Rust 认为 s1 不再有效，之后不能再次访问 s1，因此 Rust 不需要在 s1 离开作用域后清理任何东西。如果访问 s1 那就发生了所有权的错误。

## 克隆

克隆相当于深拷贝。

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone();

    println!("s1 = {}, s2 = {}", s1, s2);
}

```

这里两个变量 s1, s2 都能够被访问。

在栈中存储的数据，都可以看作克隆。

## Copy Trait

Rust 有一个叫做 Copy trait 的特殊注解，可以用在类似整型这样的存储在栈上的类型上（第十章将会详细讲解 trait）。如果一个类型实现了 Copy trait，那么一个旧的变量在将其赋值给其他变量后仍然可用。

那么哪些类型实现了 Copy trait 呢？任何一组简单标量值的组合都可以实现 Copy，任何不需要分配内存或某种形式资源的类型都可以实现 Copy 。如下是一些 Copy 的类型：

<li>所有整数类型</li>
<li>布尔类型</li>
<li>所有浮点数类型</li>
<li>字符类型，char</lu>
<li>元组，当且仅当其包含的类型也都实现 Copy。比如，(i32, i32) 实现了 Copy，但 (i32, String) 就没有。</li>

## 引用与借用

我们有时希望在函数调用之后继续使用函数参数，但由于所有权的存在，这这种操作变得复杂了很多。

引用（reference）像一个指针，因为它是一个地址，我们可以由此访问储存于该地址的属于其他变量的数据。 与指针不同，引用确保指向某个特定类型的有效值。

看下面一个例子：

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

`&s1` 语法让我们创建一个 指向 值 `s1` 的引用，但是并不拥有它。因为并不拥有这个值，所以当引用停止使用时，它所指向的值也不会被丢弃。同理，函数签名使用 & 来表明参数 s 的类型是一个引用。

我们将创建一个引用的行为称为 借用（borrowing）。不可以通过借用的方式修改不可变变量。同样，一个可变变量不能拥有两个可变引用，如下面的程序是错的。这样是为了避免数据竞争。

```rust
// 这是一个错误的程序
fn main() {
    let mut s = String::from("hello");

    let r1 = &mut s;
    let r2 = &mut s;

    println!("{}, {}", r1, r2);
}

```

同样的，在同一作用域内也不能在拥有不可变引用的同时拥有可变引用。

```rust
// 这也是一个错误的程序
fn main() {
    let mut s = String::from("hello");

    let r1 = &s; // 没问题
    let r2 = &s; // 没问题
    let r3 = &mut s; // 大问题

    println!("{}, {}, and {}", r1, r2, r3);
}
```

## slice

slice 允许你引用集合中一段连续的元素序列，而不用引用整个集合。slice 是一类引用，所以它没有所有权。

### 字符串 slice

字符串 slice 是 String 中一部分值的引用。下面是一个例子：

```rust
fn main() {
    let s = String::from("hello world");

    let hello = &s[0..5];
    let world = &s[6..11];
}

```

需要指出一点的是，字符串字面值的类型就是字符串 slice，这也解释了为什么 str 类型是不可变的。

### 其他类型 slice

下面是 i32 数组的一个例子：

```rust
#![allow(unused)]
fn main() {
    let a = [1, 2, 3, 4, 5];

    let slice = &a[1..3];

    assert_eq!(slice, &[2, 3]);
}

```
