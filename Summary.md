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

# 三、结构体与枚举

结构体和枚举变化不大，但是需要知道如何用枚举处理数据。

## 结构体

首先给出结构体的定义方式

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```

为了从结构体中获取某个特定的值，可以使用点号。举个例子，想要用户的邮箱地址，可以用 user1.email。如果结构体的实例是可变的，我们可以使用点号并为对应的字段赋值，但是 Rust 并不允许只将某个字段标记为可变。下面是一个例子。

```rust
 let mut user1 = User {
        active: true,
        username: String::from("someusername123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };

    user1.email = String::from("anotheremail@example.com");
```

也可以用一个结构体给另一结构体赋值：

```rust
 let user2 = User {
        email: String::from("another@example.com"),
        ..user1
    };
```

需要注意的是，这里 u1 字段的所有权发生了变化，因此不能再使用 u1。

### 元组结构体

元组结构体有着结构体名称提供的含义，但没有具体的字段名，只有字段的类型。

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
}
```

元组结构体实例类似于元组，可以将它们解构为单独的部分，也可以使用 . 后跟索引来访问单独的值，等等。

### 类单元结构体（unit-like structs）

目前我不知道这个东西有什么用，所以先放在这里。

```rust
struct AlwaysEqual;

fn main() {
    let subject = AlwaysEqual;
}

```

## 结构体方法

结构体相关方法需要放在 impl 块中，具体如下所示。self 前面的&表示只获得值不获得所有权。

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn width(&self) -> bool {
        self.width > 0
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    if rect1.width() {
        println!("The rectangle has a nonzero width; it is {}", rect1.width);
    }
}

```

Rust 语言不像 C++等有专门的->符号，它可以自动引用和解引用。当使用方法时，我们不需要认为加上&、&mut 或 \* 来使类型匹配。

## 枚举

Rust 的枚举功能格外强大，它可以很好的处理不同返回值，无论返回的是空，是数值，还是函数。

枚举功能的定义如下：

```rust
enum IpAddrKind {
    V4,
    V6,
}
```

我们可以将任意类型的数据放入枚举成员中，这也是与其他语言枚举类型不同的地方。

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

### match

我们采用 match 语句进行枚举的匹配，下面是一个例子：

```rust
#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        }
    }
}

fn main() {
    value_in_cents(Coin::Quarter(UsState::Alaska));
}

```

这里 match 需要穷举所有可能值，但有时我们只需要部分结果。我们可以使用统配模式或者占位符来给出默认结果。

```rust
fn main() {
    let dice_roll = 9;
    match dice_roll {
        3 => 3,
        7 => 7,
        _ => (),
    }

}
```

### if let

if let 语句可以让我们只专注于某一特定类型。它的使用方法如下所示。

```rust

fn main() {
    let coin = Coin::Penny;
    let mut count = 0;
    if let Coin::Quarter(state) = coin {
        println!("State quarter from {:?}!", state);
    } else {
        count += 1;
    }
}
```

这个程序只有当类型匹配时才会打印语句。

# 四、包管理

Rust 有许多功能可以让我们管理代码的组织，包括哪些内容可以被公开，哪些内容作为私有部分，以及程序每个作用域中的名字。这些功能。这有时被称为 “模块系统（the module system）”，包括：

<li>包（Packages）：允许我们构建、测试和分享 crate。</li>
<li>Crates ：一个模块的树形结构，它形成了库或二进制项目。</li>
<li>模块（Modules）和 use：允许你控制作用域和路径的私有性。</li>
<li>路径（path）：一个命名例如结构体、函数或模块等项的方式.</li>

### 包和 Carte

首先介绍一下 crate。

crate 是 Rust 在编译时最小的代码单位。如果用 rustc 而不是 cargo 来编译一个文件，编译器还是会将那个文件认作一个 crate。crate 可以包含模块，模块可以定义在其他文件，然后和 crate 一起编译。

crate 有两种形式：二进制项和库。二进制项可以被编译为可执行程序，比如一个命令行程序或者一个服务器。它们必须有一个 main 函数来定义当程序被执行的时候所需要做的事情。目前我们所创建的 crate 都是二进制项。

crate root 是一个源文件，Rust 编译器以它为起始点。当编译一个 crate, 编译器首先在 crate 根文件（通常，对于一个库 crate 而言是 src/lib.rs，对于一个二进制 crate 而言是 src/main.rs）中寻找需要被编译的代码。

下面看一下包。

包（package）是提供一系列功能的一个或者多个 crate。一个包会包含一个 Cargo.toml 文件，阐述如何去构建这些 crate。Cargo 就是一个包含构建你代码的二进制项的包。Cargo 也包含这些二进制项所依赖的库。其他项目也能用 Cargo 库来实现与 Cargo 命令行程序一样的逻辑。

包中可以包含至多一个库 crate(library crate)。包中可以包含任意多个二进制 crate(binary crate)，但是必须至少包含一个 crate（无论是库的还是二进制的）。

### 包管理流程

直接照抄原文来概述包管理工作的流程。

<li>从 crate 根节点开始: 当编译一个 crate, 编译器首先在 crate 根文件（通常，对于一个库 crate 而言是src/lib.rs，对于一个二进制 crate 而言是src/main.rs）中寻找需要被编译的代码。</li>

<li>声明模块: 在 crate 根文件中，你可以声明一个新模块；比如，你用mod garden声明了一个叫做garden的模块。编译器会在下列路径中寻找模块代码：

1. 内联，在大括号中，当 mod garden 后方不是一个分号而是一个大括号。
2. 在文件 src/garden.rs
3. 在文件 src/garden/mod.rs
</li>

<li>声明子模块: 在除了 crate 根节点以外的其他文件中，你可以定义子模块。比如，你可能在src/garden.rs中定义了mod vegetables;。编译器会在以父模块命名的目录中寻找子模块代码：

1. 内联，在大括号中，当 mod vegetables 后方不是一个分号而是一个大括号。
2. 在文件 src/garden/vegetables.rs
3. 在文件 src/garden/vegetables/mod.rs
</li>

<li>模块中的代码路径: 一旦一个模块是你 crate 的一部分，你可以在隐私规则允许的前提下，从同一个 crate 内的任意地方，通过代码路径引用该模块的代码。举例而言，一个 garden vegetables 模块下的Asparagus类型可以在crate::garden::vegetables::Asparagus被找到。
</li>

<li>私有 vs 公用: 一个模块里的代码默认对其父模块私有。为了使一个模块公用，应当在声明时使用pub mod替代mod。为了使一个公用模块内部的成员公用，应当在声明前使用pub。
</li>

<li>use 关键字: 在一个作用域内，use关键字创建了一个成员的快捷方式，用来减少长路径的重复。在任何可以引用crate::garden::vegetables::Asparagus的作用域，你可以通过 use crate::garden::vegetables::Asparagus;创建一个快捷方式，然后你就可以在作用域中只写Asparagus来使用该类型。</li>

在冗长的文字叙述之后，下面是一个简单的包管理例子。

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}
        fn serve_order() {}
        fn take_payment() {}
    }
}
```

它的包管理树形结构如下所示：

```text
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment

```

### 包管理路径

与文件系统相同，Rust 包管理路径也分为绝对路径和相对路径。

<li>绝对路径（absolute path）是以 crate 根（root）开头的全路径；对于外部 crate 的代码，是以 crate 名开头的绝对路径，对于当前 crate 的代码，则以字面值 crate 开头。</li>
<li>相对路径（relative path）从当前模块开始，以 self、super 或当前模块的标识符开头。</li>

下面是一个例子：

```rust
pub fn eat_at_restaurant() {
    // 绝对路径
    crate::front_of_house::hosting::add_to_waitlist();

    // 相对路径
    front_of_house::hosting::add_to_waitlist();
}
```

### pub 属性

当然。在不加 pub 属性的情况下上面的代码编译不了，能通过编译的版本如下所示。

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 绝对路径
    crate::front_of_house::hosting::add_to_waitlist();

    // 相对路径
    front_of_house::hosting::add_to_waitlist();
}

```

需要注意的是，模块公有并不使其内容也是公有的。模块上的 pub 关键字只允许其父模块引用它，而不允许访问内部代码。因为模块是一个容器，只是将模块变为公有能做的其实并不太多；同时需要更深入地选择将一个或多个项变为公有。

对于结构体，需要对内部字段也设置 pub 属性；对于枚举类型则不需要，他们默认都是 pub 类型。

可以使用 super 关键字直接访问到上一级的包。

### use 关键字

使用 use 关键字可以简化路径的书写：

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}

```

在作用域中增加 use 和路径类似于在文件系统中创建软连接（符号连接，symbolic link）。通过在 crate 根增加 use crate::front_of_house::hosting，现在 hosting 在作用域中就是有效的名称了，如同 hosting 模块被定义于 crate 根一样。通过 use 引入作用域的路径也会检查私有性，同其它路径一样。

#### 使用 as 关键字重命名

直接看例子：

```rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
    // --snip--
    Ok(())
}

fn function2() -> IoResult<()> {
    // --snip--
    Ok(())
}
```

#### pub use 混用

使用 use 关键字，将某个名称导入当前作用域后，这个名称在此作用域中就可以使用了，但它对此作用域之外还是私有的。如果想让其他人调用我们的代码时，也能够正常使用这个名称，就好像它本来就在当前作用域一样，那我们可以将 pub 和 use 合起来使用。这种技术被称为 “重导出（re-exporting）”：我们不仅将一个名称导入了当前作用域，还允许别人把它导入他们自己的作用域。

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

#### 嵌套路径

只是一种减少代码行数的办法。

```rust
use std::{cmp::Ordering, io};
// 等价于下面这两句:
// use std::cmp::Ordering;
// use std::cmp::io;

use std::io::{self, Write};
//  等价于下面这两句:
// use std::io;
// use std::io::Write;
```

## 五、常见集合

这一部分主要介绍了 Vector, String, Hashmap 的相关使用。

### Vector

直接用代码介绍下 API：

```rust
fn main() {
    // 1.新建vector
    let v: Vec<i32> = Vec::new();
    let v = vec![1, 2, 3];

    // 2.添加元素
    v.push(5);
    v.push(6);
    v.push(7);
    v.push(8);

    // 3.获取元素
    // 3.1 使用索引直接获取，注意这里不获得所有权。
    let third: &i32 = &v[2];
    println!("The third element is {third}");

    //3.2 get() 方法获取，返回值是Option<&T>枚举类型。
    let third: Option<&i32> = v.get(2);
    match third {
        Some(third) => println!("The third element is {third}"),
        None => println!("There is no third element."),
    }

    //4. 遍历Vector, 同样，需要使用引用来避免所有权的变化。

    let v = vec![100, 32, 57];
    for i in &mut v {
        println!("{i}");
        *i += 10;
    }


}

```

vector 只能储存相同类型的值。这是很不方便的；绝对会有需要储存一系列不同类型的值的用例。幸运的是，枚举的成员都被定义为相同的枚举类型，所以当需要在 vector 中储存不同类型值时，我们可以定义并使用一个枚举。

```rust
fn main() {
    enum SpreadsheetCell {
        Int(i32),
        Float(f64),
        Text(String),
    }

    let row = vec![
        SpreadsheetCell::Int(3),
        SpreadsheetCell::Text(String::from("blue")),
        SpreadsheetCell::Float(10.12),
    ];
}

```

### String

由于 Rust 中存储的字符串是 UTF-8 字符集，因此从字节存储角度来看和其他语言有很大不同。但基本 API 还是类似的。 依然用代码直接示例。

```rust
fn main() {
    // 创建一个空字符串
    let mut s = String::new();

    // str类型转String
    let mut s = "initial contents".to_string();

    // 更新字符串
    s.push_str("bar");
    s.push('l');

    // 如果 push_str 方法不获得s2的所有权
    let mut s1 = String::from("foo");
    let s2 = "bar";
    s1.push_str(s2);
    println!("s2 is {s2}");

    // String类型不支持索引，因为它采用UTF-8编码，
    // 不知道字节是怎么分配给字符的

    // s 包含字符串的头四个字节
    let hello = "Здравствуйте";
    let s = &hello[0..4];

    // 遍历字符串
    for c in "Зд".chars() {
        println!("{c}");
    }


}
```

### Hashmap

有了所有权的限制，Hashmap 的使用比其他语言复杂一些。同样，直接看下面的代码示例：

```rust
fn main() {
    use std::collections::HashMap;

    // 创建空hashmap
    let mut scores = HashMap::new();

    // 添加键值对
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    // get() 方法获得键对应的值。Option<&V>，
    // 如果某个键在哈希 map 中没有对应的值,get 会返回 None。
    // 程序调用 copied 方法来获取Option<i32>，而不是 Option<&i32>,
    // 接着调用 unwrap_or 在 scores 中没有该键所对应的项时将其设置为零。
    let team_name = String::from("Blue");
    let score = scores.get(&team_name).copied().unwrap_or(0);

    // 遍历hashmap
    for (key, value) in &scores {
        println!("{key}: {value}");
    }

    // entry使用我们想要检查的键作为参数，返回值是一个枚举Entry，
    // 它代表了可能存在也可能不存在的值。
    // Entry 的 or_insert 方法在键对应的值存在时就返回这个值的可变引用，
    // 如果不存在则将参数作为新值插入并返回新值的可变引用。
    scores.entry(String::from("Red")).or_insert(100);


    // 一个例子：单词计数，统计一些文本中每一个单词分别出现了多少次。
    let text = "hello world wonderful world";

    let mut map = HashMap::new();

    for word in text.split_whitespace() {
        let count = map.entry(word).or_insert(0);
        *count += 1;
    }

}

```

# 六、泛型、trait 与生命周期

先跳过 panic 把基本语法说完

## 泛型

泛型的含义与其他语言相同。首先看下结构体里泛型的定义：

```rust

// 相同类型
struct Point<T> {
    x: T,
    y: T,
}

// 结构体方法的定义
impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}


fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}


// 不同类型
struct Point2<T, U> {
    x: T,
    y: U,
}

// 结构体方法定义
impl<T, U> Point2<T, U> {
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let both_integer = Point2 { x: 5, y: 10 };
    let both_float = Point2 { x: 1.0, y: 4.0 };
    let integer_and_float = Point2 { x: 5, y: 4.0 };
}


```

再看下常见的枚举类型：

```rust

// 相同类型
enum Option<T> {
    Some(T),
    None,
}

// 不同类型
enum Result<T, E> {
    Ok(T),
    Err(E),
}


```

普通泛型方法定义如下：

```rust
fn largest<T>(list: &[T]) -> T {
    let largest = list[0];
    largest
}

```

## Trait

Trait 在其他语言中可以理解为接口。一个类型的行为由其可供调用的方法构成。如果可以对不同类型调用相同的方法的话，这些类型就可以共享相同的行为了。trait 定义是一种将方法签名组合起来的方法，目的是定义一个实现某些目的所必需的行为的集合。

下面是一个示例：

```rust

#![allow(unused)]
fn main() {
pub trait Summary {
    fn summarize(&self) -> String;
}

pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
}

```

实现 trait 时需要注意的一个限制是，只有当 trait 或者要实现 trait 的类型位于 crate 的本地作用域时，才能为该类型实现 trait，但是我们不能为外部类型实现外部 trait。这个限制是被称为 相干性（coherence） 的程序属性的一部分，或者更具体的说是 孤儿规则（orphan rule），其得名于不存在父类型。这条规则确保了其他人编写的代码不会破坏你代码，反之亦然。没有这条规则的话，两个 crate 可以分别对相同类型实现相同的 trait，而 Rust 将无从得知应该使用哪一个实现。

### trait 作为参数

```rust
pub fn notify(item: impl Summary) {
    println!("Breaking news! {}", item.summarize());
}

```

### trait bound 语法

impl 形式是 trait bound 的语法糖。

```rust
pub fn notify<T: Summary>(item: T) {
    println!("Breaking news! {}", item.summarize());
}

```

### 多个 trait 参数

```rust

// impl 类型
pub fn notify(item: impl Summary + Display) {
}

// trait bound 类型
pub fn notify<T: Summary + Display>(item: T) {
}
```

另外，可以用 where 语句简化 trait bound

```rust
fn some_function<T, U>(t: T, u: U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{}

```

### 返回实现了某个 trait 的类型

直接看例子：

```rust
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
        reply: false,
        retweet: false,
    }
}

```

## 生命周期

Rust 中的每一个引用都有其 生命周期（lifetime），也就是引用保持有效的作用域。生命周期的主要目标是避免悬垂引用。大部分时候生命周期是隐含并可以推断的，正如大部分时候类型也是可以推断的一样。类似于当因为有多种可能类型的时候必须注明类型，也会出现引用的生命周期以一些不同方式相关联的情况，所以 Rust 需要我们使用泛型生命周期参数来注明他们的关系，这样就能确保运行时实际使用的引用绝对是有效的。

下面看一个需要使用生命周期的例子：

```rust
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

```

当我们定义这个函数的时候，并不知道传递给函数的具体值，所以也不知道到底是 if 还是 else 会被执行。借用检查器自身同样也无法确定，因为它不知道 x 和 y 的生命周期是如何与返回值的生命周期相关联的(不知道返回的是什么，因此无法通过分析作用域得到生命周期)。为了修复这个错误，我们将增加泛型生命周期参数来定义引用间的关系以便借用检查器可以进行分析。

应该修改成这个样子：

```rust

fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

```

这里我们想要告诉 Rust 关于参数中的引用和返回值之间的限制是他们都必须拥有相同的生命周期.

当在函数中使用生命周期标注时，这些标注出现在函数签名中，而不存在于函数体中的任何代码中。这是因为 Rust 能够分析函数中代码而不需要任何协助，不过当函数引用或被函数之外的代码引用时，让 Rust 自身分析出参数或返回值的生命周期几乎是不可能的。这些生命周期在每次函数被调用时都可能不同。这也就是为什么我们需要手动标记生命周期。

让我们看看如何通过传递拥有不同具体生命周期的引用来限制 longest 函数的使用。

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let string1 = String::from("long string is long");

    {
        let string2 = String::from("xyz");
        let result = longest(string1.as_str(), string2.as_str());
        println!("The longest string is {}", result);
    }
}

```

在这个例子中，string1 直到外部作用域结束都是有效的，string2 则在内部作用域中是有效的，而 result 则引用了一些直到内部作用域结束都是有效的值。借用检查器认可这些代码；它能够编译和运行，并打印出 The longest string is long string is long。

但是下面的例子就无法运行了：

```rust

fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }
    println!("The longest string is {}", result);
}

```

如果从人的角度读上述代码，我们可能会觉得这个代码是正确的。 string1 更长，因此 result 会包含指向 string1 的引用。因为 string1 尚未离开作用域，对于 println! 来说 string1 的引用仍然是有效的。然而，我们通过生命周期参数告诉 Rust 的是： <b>longest 函数返回的引用的生命周期应该与传入参数的生命周期中较短那个保持一致。</b>因此，借用检查器这一代码，因为它可能会存在无效的引用。

### 更深入的理解生命周期

指定生命周期参数的正确方式依赖函数实现的具体功能。例如，如果将 longest 函数的实现修改为总是返回第一个参数而不是最长的字符串 slice，就不需要为参数 y 指定一个生命周期。如下例所示。

```rust

fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
```

<b>当从函数返回一个引用，返回值的生命周期参数需要与一个参数的生命周期参数相匹配。</b>如果返回的引用 没有 指向任何一个参数，那么唯一的可能就是它指向一个函数内部创建的值，它将会是一个悬垂引用，因为它将会在函数结束时离开作用域。

下面是另一个报错的例子：

```rust
fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from("really long string");
    result.as_str()
}
```

综上，生命周期语法是用于将函数的多个参数与其返回值的生命周期进行关联的。一旦他们形成了某种关联，Rust 就有了足够的信息来允许内存安全的操作并阻止会产生悬垂指针亦或是违反内存安全的行为。

### 结构体定义中的生命周期标注

接下来，我们将定义包含引用的结构体，不过这需要为结构体定义中的每一个引用添加生命周期标注。看下面的例子：

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}
```

### 生命周期的省略

但是在之前的代码中，我们从来没有考虑过生命周期的问题，程序也能通过编译，这是为什么呢？下面的三条规则可以帮助编译器自动的推断出引用的生命周期。

<li>每一个是引用的参数都有它自己的生命周期参数。换句话说就是，有一个引用参数的函数有一个生命周期参数：fn foo<'a>(x: &'a i32)，有两个引用参数的函数有两个不同的生命周期参数，fn foo<'a, 'b>(x: &'a i32, y: &'b i32)，依此类推。</li>

<li>如果只有一个输入生命周期参数，那么它被赋予所有输出生命周期参数：fn foo<'a>(x: &'a i32) -> &'a i32。</li>

<li>如果方法有多个输入生命周期参数并且其中一个参数是 &self 或 &mut self，说明是个对象的方法(method)(译者注： 这里涉及 Rust 的面向对象，参见第 17 章), 那么所有输出生命周期参数被赋予 self 的生命周期。第三条规则使得方法更容易读写，因为只需更少的符号。</li>

### 结构体方法中的生命周期标注

结构体字段的生命周期必须总是在 impl 关键字之后声明并在结构体名称之后被使用，因为这些生命周期是结构体类型的一部分。impl 块里的方法签名中，引用可能与结构体字段中的引用相关联，也可能是独立的。

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}

```

### 静态生命周期

'static，其生命周期能够存活于整个程序期间。所有的字符串字面量都拥有 'static 生命周期。下面是一个例子：

```rust
let s: &'static str = "I have a static lifetime.";
```

## 七、智能指针

智能指针（smart pointers）是一类数据结构，它们的表现类似指针，但是也拥有额外的元数据和功能。在 Rust 中，普通引用和智能指针的一个额外的区别是引用是一类只借用数据的指针；相反，在大部分情况下，智能指针 拥有 它们指向的数据实际上，String 和 Vec\<T>就是智能指针！

智能指针通常使用结构体实现。智能指针区别于常规结构体的显著特性在于其实现了 Deref 和 Drop trait。Deref trait 允许智能指针结构体实例表现的像引用一样，这样就可以编写既用于引用、又用于智能指针的代码。Drop trait 允许我们自定义当智能指针离开作用域时运行的代码。本章会讨论这些 trait 以及为什么对于智能指针来说它们很重要。

### Box\<T>

最简单直接的智能指针是 box，其类型是 Box\<T>。 box 允许你将一个值放在堆上而不是栈上。留在栈上的则是指向堆数据的指针。使用 Box\<T> 有如下几种情况：

<li>当有一个在编译时未知大小的类型，而又想要在需要确切大小的上下文中使用这个类型值的时候。</li>
<li>当有大量数据并希望在确保数据不被拷贝的情况下转移所有权的时候。</li>
<li>当希望拥有一个值并只关心它的类型是否实现了特定 trait 而不是其具体类型的时候。</li>

一个例子如下：

```rust
fn main() {
    let b = Box::new(5);
    println!("b = {}", b);
}

```

#### Box 允许创建递归类型

经典例子：链表。

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let list = Cons(1,
        Box::new(Cons(2,
            Box::new(Cons(3,
                Box::new(Nil))))));
}

```

### Deref trait

为了体会默认情况下智能指针与引用的不同，让我们创建一个类似于标准库提供的 Box\<T> 类型的智能指针。接着学习如何增加使用解引用运算符的功能。

从根本上说，Box\<T> 被定义为包含一个元素的元组结构体，所以我们以相同的方式定义了 MyBox\<T> 类型。我们还定义了 new 函数来对应定义于 Box\<T> 的 new 函数：

```rust

struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}
```

直接解引用会报错：

```rust
fn main() {
    let x = 5;
    let y = MyBox::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

需要我们自己实现 Deref trait。

```rust

use std::ops::Deref;

struct MyBox<T>(T);

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.0
    }
}
```

当我们使用\*y 时，Rust 底层将其修改为

```rust
*(y.deref())
```

Rust 将 _ 运算符替换为先调用 deref 方法再进行普通解引用的操作，如此我们便不用担心是否还需手动调用 deref 方法了。Rust 的这个特性可以让我们写出行为一致的代码，无论是面对的是常规引用还是实现了 Deref 的类型。注意，每次当我们在代码中使用 _ 时， _ 运算符都被替换成了先调用 deref 方法再接着使用 _ 解引用的操作，且只会发生一次，不会对 \* 操作符无限递归替换，解引用出上面 i32 类型的值就停止了.

### 函数和方法的隐式解引用强制转换

解引用强制转换（deref coercions）是 Rust 在函数或方法传参上的一种便利。解引用强制转换只能工作在实现了 Deref trait 的类型上。解引用强制转换将一种类型（A）隐式转换为另外一种类型（B）的引用，因为 A 类型实现了 Deref trait，并且其关联类型是 B 类型。比如，解引用强制转换可以将 &String 转换为 &str，因为类型 String 实现了 Deref trait 并且其关联类型是 str。下面是一个例子：

```rust
use std::ops::Deref;

struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.0
    }
}

fn hello(name: &str) {
    println!("Hello, {}!", name);
}

fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&(*m)[..]);
}

```

(\*m) 将 MyBox\<String> 解引用为 String。接着 & 和 [..] 获取了整个 String 的字符串 slice 来匹配 hello 的签名。没有解引用强制转换所有这些符号混在一起将更难以读写和理解。解引用强制转换使得 Rust 自动的帮我们处理这些转换。

### Drop trait

对于智能指针模式来说第二个重要的 trait 是 Drop，其允许我们在值要离开作用域时执行一些代码。可以为任何类型提供 Drop trait 的实现，同时所指定的代码被用于释放类似于文件或网络连接的资源。我们在智能指针上下文中讨论 Drop 是因为其功能几乎总是用于实现智能指针。例如，Box\<T> 自定义了 Drop 用来释放 box 所指向的堆空间。

看下面的一个例子：

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer { data: String::from("my stuff") };
    let d = CustomSmartPointer { data: String::from("other stuff") };
    println!("CustomSmartPointers created.");
}
```

### Rf\<T> 引用计数指针

Rc<T> 用于当我们希望在堆上分配一些内存供程序的多个部分读取，而且无法在编译时确定程序的哪一部分会最后结束使用它的时候。如果确实知道哪部分是最后一个结束使用的话，就可以令其成为数据的所有者，正常的所有权规则就可以在编译时生效。注意 Rc<T> 只能用于单线程场景。

看下面的一个例子：

```rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    let b = Cons(3, Rc::clone(&a));
    let c = Cons(4, Rc::clone(&a));
}

```

可以调用 a.clone() 而不是 Rc::clone(&a)，不过在这里 Rust 的习惯是使用 Rc::clone。Rc::clone 的实现并不像大部分类型的 clone 实现那样对所有数据进行深拷贝。Rc::clone 只会增加引用计数，这并不会花费多少时间。深拷贝可能会花费很长时间。通过使用 Rc::clone 进行引用计数，可以明显的区别深拷贝类的克隆和增加引用计数类的克隆。当查找代码中的性能问题时，只需考虑深拷贝类的克隆而无需考虑 Rc::clone 调用。

下面一个例子可以用于查看引用计数：

```rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    println!("count after creating a = {}", Rc::strong_count(&a));
    let b = Cons(3, Rc::clone(&a));
    println!("count after creating b = {}", Rc::strong_count(&a));
    {
        let c = Cons(4, Rc::clone(&a));
        println!("count after creating c = {}", Rc::strong_count(&a));
    }
    println!("count after c goes out of scope = {}", Rc::strong_count(&a));
}
```

我们能够看到 a 中 Rc\<List> 的初始引用计数为 1，接着每次调用 clone，计数会增加 1。当 c 离开作用域时，计数减 1。不必像调用 Rc::clone 增加引用计数那样调用一个函数来减少计数；Drop trait 的实现当 Rc\<T> 值离开作用域时自动减少引用计数。

从这个例子我们所不能看到的是，在 main 的结尾当 b 然后是 a 离开作用域时，此处计数会是 0，同时 Rc\<List> 被完全清理。使用 Rc\<T> 允许一个值有多个所有者，引用计数则确保只要任何所有者依然存在其值也保持有效。

<b>通过不可变引用， Rc\<T> 允许在程序的多个部分之间只读地共享数据。如果 Rc\<T> 也允许多个可变引用，则会违反第 4 章讨论的借用规则之一：相同位置的多个可变借用可能造成数据竞争和不一致。</b>不过可以修改数据是非常有用的！这就是下一节 RefCall\<T>的相关内容。

## RefCall\<T>与内部可变性模式

不同于 Rc<T>，RefCell\<T> 代表其数据的唯一的所有权。那么是什么让 RefCell\<T> 不同于像 Box\<T> 这样的类型呢？回忆一下借用规则：

<li>在任意给定时刻，只能拥有一个可变引用或任意数量的不可变引用 之一（而不是两者）。</li>
<li>引用必须总是有效的。</li>
对于引用和 Box<T>，借用规则的不可变性作用于编译时。对于 RefCell<T>，这些不可变性作用于 运行时。对于引用，如果违反这些规则，会得到一个编译错误。而对于 RefCell<T>，如果违反这些规则程序会 panic 并退出。因为一些分析是不可能的，如果 Rust 编译器不能通过所有权规则编译，它可能会拒绝一个正确的程序；从这种角度考虑它是保守的。如果 Rust 接受不正确的程序，那么用户也就不会相信 Rust 所做的保证了。然而，如果 Rust 拒绝正确的程序，虽然会给开发者带来不便，但不会带来灾难。RefCell<T> 正是用于当你确信代码遵守借用规则，而编译器不能理解和确定的时候。

请看下面的一个示例：

```rust

#![allow(unused)]
fn main() {
pub trait Messenger {
    fn send(&self, msg: &str);
}

pub struct LimitTracker<'a, T: Messenger> {
    messenger: &'a T,
    value: usize,
    max: usize,
}

impl<'a, T> LimitTracker<'a, T>
    where T: Messenger {
    pub fn new(messenger: &T, max: usize) -> LimitTracker<T> {
        LimitTracker {
            messenger,
            value: 0,
            max,
        }
    }

    pub fn set_value(&mut self, value: usize) {
        self.value = value;

        let percentage_of_max = self.value as f64 / self.max as f64;

        if percentage_of_max >= 1.0 {
            self.messenger.send("Error: You are over your quota!");
        } else if percentage_of_max >= 0.9 {
             self.messenger.send("Urgent warning: You've used up over 90% of your quota!");
        } else if percentage_of_max >= 0.75 {
            self.messenger.send("Warning: You've used up over 75% of your quota!");
        }
    }
}
}


pub trait Messenger {
    fn send(&self, msg: &str);
}

pub struct LimitTracker<'a, T: Messenger> {
    messenger: &'a T,
    value: usize,
    max: usize,
}

impl<'a, T> LimitTracker<'a, T>
    where T: Messenger {
    pub fn new(messenger: &T, max: usize) -> LimitTracker<T> {
        LimitTracker {
            messenger,
            value: 0,
            max,
        }
    }

    pub fn set_value(&mut self, value: usize) {
        self.value = value;

        let percentage_of_max = self.value as f64 / self.max as f64;

        if percentage_of_max >= 1.0 {
            self.messenger.send("Error: You are over your quota!");
        } else if percentage_of_max >= 0.9 {
             self.messenger.send("Urgent warning: You've used up over 90% of your quota!");
        } else if percentage_of_max >= 0.75 {
            self.messenger.send("Warning: You've used up over 75% of your quota!");
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    use std::cell::RefCell;

    struct MockMessenger {
        sent_messages: RefCell<Vec<String>>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger { sent_messages: RefCell::new(vec![]) }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.borrow_mut().push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        // --snip--
        let mock_messenger = MockMessenger::new();
        let mut limit_tracker = LimitTracker::new(&mock_messenger, 100);
        limit_tracker.set_value(75);

        assert_eq!(mock_messenger.sent_messages.borrow().len(), 1);
    }
}
fn main() {}

```

## RefCell\<T> 在运行时记录借用

当创建不可变和可变引用时，我们分别使用 & 和 &mut 语法。对于 RefCell\<T> 来说，则是 borrow 和 borrow_mut 方法，这属于 RefCell\<T> 安全 API 的一部分。borrow 方法返回 Ref\<T> 类型的智能指针，borrow_mut 方法返回 RefMut 类型的智能指针。这两个类型都实现了 Deref，所以可以当作常规引用对待。

RefCell\<T> 记录当前有多少个活动的 Ref\<T> 和 RefMut\<T> 智能指针。每次调用 borrow，RefCell\<T> 将活动的不可变借用计数加一。当 Ref\<T> 值离开作用域时，不可变借用计数减一。就像编译时借用规则一样，RefCell\<T> 在任何时候只允许有多个不可变借用或一个可变借用。

如果我们尝试违反这些规则，相比引用时的编译时错误，RefCell\<T> 的实现会在运行时出现 panic。

## 引用循环

需要使用 Weak\<T>来缓解。使用 RefCell\<T>和 Rc\<T>可能会造成引用循环，下面是一个例子：

```rust
use crate::List::{Cons, Nil};
use std::cell::RefCell;
use std::rc::Rc;

#[derive(Debug)]
enum List {
    Cons(i32, RefCell<Rc<List>>),
    Nil,
}

impl List {
    fn tail(&self) -> Option<&RefCell<Rc<List>>> {
        match self {
            Cons(_, item) => Some(item),
            Nil => None,
        }
    }
}

fn main() {
    let a = Rc::new(Cons(5, RefCell::new(Rc::new(Nil))));

    println!("a initial rc count = {}", Rc::strong_count(&a));
    println!("a next item = {:?}", a.tail());

    let b = Rc::new(Cons(10, RefCell::new(Rc::clone(&a))));

    println!("a rc count after b creation = {}", Rc::strong_count(&a));
    println!("b initial rc count = {}", Rc::strong_count(&b));
    println!("b next item = {:?}", b.tail());

    if let Some(link) = a.tail() {
        *link.borrow_mut() = Rc::clone(&b);
    }

    println!("b rc count after changing a = {}", Rc::strong_count(&b));
    println!("a rc count after changing a = {}", Rc::strong_count(&a));

    // Uncomment the next line to see that we have a cycle;
    // it will overflow the stack
    // println!("a next item = {:?}", a.tail());
}
```

这里创建了两个链表 a，b，并且 a,b 互相作为对方的尾部。由于我们采用了 Rc\<T>，这两个引用计数均为 1。这样编译器永远也不会主动清除 a,b，除非程序终止。因此，我们需要 weak\<T>来使得 weak_count 加 1，而不是 strong_count。

强引用代表如何共享 Rc\<T> 实例的所有权，但弱引用并不属于所有权关系。它们不会造成引用循环，因为任何弱引用的循环会在其相关的强引用计数为 0 时被打断。

因为 Weak\<T> 引用的值可能已经被丢弃了，为了使用 Weak\<T> 所指向的值，我们必须确保其值仍然有效。为此可以调用 Weak\<T> 实例的 upgrade 方法，这会返回 Option\<Rc\<T>>。如果 Rc\<T> 值还未被丢弃，则结果是 Some；如果 Rc\<T> 已被丢弃，则结果是 None。因为 upgrade 返回一个 Option\<Rc\<T>>，Rust 会确保处理 Some 和 None 的情况，所以它不会返回非法指针。

树形结构是典型的需要用到 weak\<T>的例子。这里我们同时使用父子指针。为来避免造成环路，父指针使用弱引用。请看下面的例子：

```rust
use std::cell::RefCell;
use std::rc::{Rc, Weak};

#[derive(Debug)]
struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}

fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());

    let branch = Rc::new(Node {
        value: 5,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });

    *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
}

```

这样，一个节点就能够引用其父节点，但不拥有其父节点。这个示例中，我们更新 main 来使用新定义以便 leaf 节点可以通过 branch 引用其父节点。

# 八、并发

### 创建线程

为了创建一个新线程，需要调用 thread::spawn 函数并传递一个闭包，并在其中包含希望在新线程运行的代码。一个简单的例子如下所示。

```rust
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}

```

由于主线程结束，示例中的代码大部分时候会提早结束新建线程，甚至不会开始。可以通过将 thread::spawn 的返回值储存在变量中来修复新建线程部分没有执行或者完全没有执行的问题。thread::spawn 的返回值类型是 JoinHandle。JoinHandle 是一个拥有所有权的值，当对其调用 join 方法时，它会等待其线程结束。

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}

```

### 使用 move 转移所有权

下面是一段无法执行的代码：线程中使用了 v（只是借用，不取得所有权）。因此 v 很可能在线程还没有结束时就被主线程释放掉，造成安全问题。

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

move 可以将所有权转移至闭包内部，这样线程就可以安全的使用 v 了。

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

## 线程通信

### 信道

为了实现消息传递并发，Rust 标准库提供了一个 信道（channel）实现。信道是一个通用编程概念，表示数据从一个线程发送到另一个线程。编程中的信息渠道（信道）有两部分组成，一个发送者（transmitter）和一个接收者（receiver）。代码中的一部分调用发送者的方法以及希望发送的数据，另一部分则检查接收端收到的消息。当发送者或接收者任一被丢弃时可以认为信道被 关闭（closed）了。

下面的代码描述了信道的创建。

```rust
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();
}

```

这里使用 mpsc::channel 函数创建一个新的信道；mpsc 是 多个生产者，单个消费者的缩写。mpsc::channel 函数返回一个元组：第一个元素是发送端 -- 发送者，而第二个元素是接收端 -- 接收者。

下面的代码描述了发送方如何发送信息：

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });
}
```

这里再次使用 thread::spawn 来创建一个新线程并使用 move 将 tx 移动到闭包中这样新建线程就拥有 tx 了。新建线程需要拥有信道的发送端以便能向信道发送消息。信道的发送端有一个 send 方法用来获取需要放入信道的值。send 方法返回一个 Result\<T, E> 类型，所以如果接收端已经被丢弃了，将没有发送值的目标，所以发送操作会返回错误。在这个例子中，出错的时候调用 unwrap 产生 panic。

接下来在主线程中实现接收信息。

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

信道的接收者有两个有用的方法：recv 和 try_recv。这里，我们使用了 recv，它是 receive 的缩写。这个方法会阻塞主线程执行直到从信道中接收一个值。一旦发送了一个值，recv 会在一个 Result<T, E> 中返回它。当信道发送端关闭，recv 会返回一个错误表明不会再有新的值到来了。

try_recv 不会阻塞，相反它立刻返回一个 Result<T, E>：Ok 值包含可用的信息，而 Err 值代表此时没有任何消息。如果线程在等待消息过程中还有其他工作时使用 try_recv 很有用：可以编写一个循环来频繁调用 try_recv，在有可用消息时进行处理，其余时候则处理一会其他工作直到再次检查。

#### 信道与所有权转移

Rust 语言在任何情况都要考虑所有权。简单的说：<b>send 函数获取其参数的所有权并移动这个值归接收者所有。</b>

#### 多个生产者

通过对最初生产者的克隆得到（合情合理！）.

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {

    let (tx, rx) = mpsc::channel();

    let tx1 = tx.clone();
    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx1.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    thread::spawn(move || {
        let vals = vec![
            String::from("more"),
            String::from("messages"),
            String::from("for"),
            String::from("you"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }

}

```

### 共享状态并发

在某种程度上，任何编程语言中的信道都类似于单所有权，因为一旦将一个值传送到信道中，将无法再使用这个值。共享内存类似于多所有权：多个线程可以同时访问相同的内存位置。第十五章介绍了智能指针如何使得多所有权成为可能，然而这会增加额外的复杂性，因为需要以某种方式管理这些不同的所有者。Rust 的类型系统和所有权规则极大的协助了正确地管理这些所有权。作为一个例子，让我们看看互斥器，一个更为常见的共享内存并发原语。（就是 Mutex 系列语法）

#### Mutex\<T>及其相关 API

```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        let mut num = m.lock().unwrap();
        *num = 6;
    }

    println!("m = {:?}", m);
}
```

像很多类型一样，我们使用关联函数 new 来创建一个 Mutex<T>。使用 lock 方法获取锁，以访问互斥器中的数据。这个调用会阻塞当前线程，直到我们拥有锁为止。

如果另一个线程拥有锁，并且那个线程 panic 了，则 lock 调用会失败。在这种情况下，没人能够再获取锁，所以这里选择 unwrap 并在遇到这种情况时使线程 panic。

一旦获取了锁，就可以将返回值（在这里是 num）视为一个其内部数据的可变引用了。类型系统确保了我们在使用 m 中的值之前获取锁。m 的类型是 Mutex\<i32> 而不是 i32，所以 必须 获取锁才能使用这个 i32 值。

Mutex\<T> 是一个智能指针。更准确的说，lock 调用 返回 一个叫做 MutexGuard 的智能指针。这个智能指针实现了 Deref 来指向其内部数据；其也提供了一个 Drop 实现当 MutexGuard 离开作用域时自动释放锁，这正发生于示例内部作用域的结尾。为此，我们不会忘记释放锁并阻塞互斥器为其它线程所用的风险，因为锁的释放是自动发生的。

#### 在线程间共享 Mutex\<T>

下面是一段错误代码：

```rust
use std::sync::Mutex;
use std::thread;

fn main() {

    // 注意counter所有权的移动
    let counter = Mutex::new(0);
    let mut handles = vec![];

    for _ in 0..10 {
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}

```

这个例子不能完成编译，因为所有权被移动到多个线程中，这是 Rust 所不允许的。

```rust
use std::rc::Rc;
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Rc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Rc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}

```

使用 Rc\<T>智能指针呢？因为它允许多所有权。

```rust
se std::rc::Rc;
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Rc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Rc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}

```

编译结果告诉我们，这也不行。因为 Rc\<T>只能用于单线程环境中，这也是最初讲 Rc\<T>所提到的。多线程环境中，需要用 Arc\<T>。下面的代码就可以了。

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}

```

## 使用 Send 和 Sync trait 的可扩展并发

接下来考虑两个 trait：std::marker 中的 Sync 和 Send。

### 通过 Send 允许在线程间转移所有权

Send 标记 trait 表明实现了 Send 的类型值的所有权可以在线程间传送。几乎所有的 Rust 类型都是 Send 的，不过有一些例外，包括 Rc\<T>：这是不能 Send 的，因为如果克隆了 Rc\<T> 的值并尝试将克隆的所有权转移到另一个线程，这两个线程都可能同时更新引用计数。为此，Rc<T> 被实现为用于单线程场景.
任何完全由 Send 的类型组成的类型也会自动被标记为 Send。几乎所有基本类型都是 Send 的，除了裸指针（raw pointer）。

### Sync 允许多线程访问

实现了 Sync 的类型可以安全的在多个线程中拥有其值的引用。换一种方式来说，对于任意类型 T，如果 &T（T 的不可变引用）是 Send 的话, T 就是 Sync 的，这意味着其引用就可以安全的发送到另一个线程。类似于 Send 的情况，基本类型是 Sync 的，完全由 Sync 的类型组成的类型也是 Sync 的。

智能指针 Rc\<T> 也不是 Sync 的，RefCell\<T>和 Cell\<T> 系列类型也不是 Sync 的。RefCell\<T> 在运行时所进行的借用检查也不是线程安全的。但是 Mutex\<T> 是 Sync 的。

本并发部分的最后一句话：<b>手动实现 Send 和 Sync 是不安全的</b>。
