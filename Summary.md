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
