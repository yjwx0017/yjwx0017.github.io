title: Rust 程序设计语言 笔记 （1 / 3）
categories: 日志
tags: [Rust]
date: 2024-04-13 12:05:00
---
[Rust 程序设计语言 中文版][1]

## Hello World

Rust 文件通常以 .rs 扩展名结尾。

``` rust
fn main() {
    println!("Hello, world!");
}
```

编译并运行：

```
> rustc main.rs
> .\main.exe
Hello, world!
```

其他命令：

```
rustup update           # 更新
rustup self uninstall   # 卸载
rustc --version         # 查看版本号
rustup doc              # 查看文档
```

## Cargo

Cargo 是 Rust 的构建系统和包管理工具。

常用命令：

```
cargo new hello_cargo  # 创建项目
cargo build            # 构建项目
cargo run              # 构建并运行项目
cargo check            # 构建项目而无需生成二进制文件来检查错误
cargo build --release  # 发布构建
```

<!--more-->

## 猜字游戏

修改 Cargo.toml 文件，添加依赖库：

``` toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
rand = "0.9.0"
```

程序实现：

``` rust
use rand::Rng;
use std::cmp::Ordering;
use std::io;

fn main() {
    // 叹号结尾的是宏
    println!("Guess the number!");

    // 定义变量并赋值一个随机数
    let secret_number = rand::thread_rng().gen_range(1..101);

    // 循环
    loop {
        println!("Please input your guess.");

        // mut 表示该变量可被修改
        let mut guess = String::new();

        // 从标准输入读取一行
        io::stdin()
            .read_line(&mut guess)
            .expect("Failed to read line");

        // 定义一个 u32 类型的变量
        // match 类似于 C++ 中的 switch
        // parse() 函数将字符串转换为数字
        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        // 使用 {} 格式化字符串
        println!("You guessed: {}", guess);

        // 比较两个数字，根据比较结果执行相应的代码
        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}
```

## 变量可变性

默认情况下变量是不可变的（immutable）。这是 Rust 众多精妙之处的其中一个，这些特性让你充分利用 Rust 提供的安全性和简单并发性的方式来编写代码。不过你也可以选择让变量是可变的（mutable）。

我们可以通过在变量名前加上 mut 使得它们可变。

``` rust
fn main() {
    let mut x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

## 常量

常量（constant）是绑定到一个常量名且不允许更改的值。

常量只能设置为常量表达式，而不能是函数调用的结果或是只能在运行时计算得到的值。

``` rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

## 遮蔽

可以声明和前面变量具有相同名称的新变量。第一个变量被第二个变量遮蔽（shadow），这意味着当我们使用变量时我们看到的会是第二个变量的值。我们可以通过使用相同的变量名并重复使用 let 关键字来遮蔽变量。

``` rust
fn main() {
    let x = 5;

    let x = x + 1;

    {
        let x = x * 2;
        println!("The value of x in the inner scope is: {}", x);
    }

    println!("The value of x is: {}", x);
}

```

## 数据类型：标量

有符号整数类型：i8 i16 i32 i64 i128 isize

无符号整数类型：u8 u16 u32 u64 u128 usize

isize 和 usize 类型取决于程序运行的计算机体系结构，在表中表示为“arch”：若使用 64 位架构系统则为 64 位，若使用 32 位架构系统则为 32 位。

浮点类型：f32 f64

布尔类型：bool

字符类型：char

字符类型大小为 4 个字节，表示的是一个 Unicode 标量值。

## 数据类型：元组

元组是将多种类型的多个值组合到一个复合类型中的一种基本方式。元组的长度是固定的：声明后，它们就无法增长或缩小。

``` rust
let tup: (i32, f64, u8) = (500, 6.4, 1);
```

解构：

``` rust
let tup = (500, 6.4, 1);
let (x, y, z) = tup;
```

访问元组元素：

``` rust
let x: (i32, f64, u8) = (500, 6.4, 1);
let five_hundred = x.0;
let six_point_four = x.1;
let one = x.2;
```

## 数据类型：数组

将多个值组合在一起的另一种方式就是使用数组（array）。与元组不同，数组的每个元素必须具有相同的类型。与某些其他语言中的数组不同，Rust 中的数组具有固定长度。

``` rust
let a = [1, 2, 3, 4, 5];
// i32 是每个元素的类型。分号之后，数字 5 表明该数组包含 5 个元素
let b: [i32; 5] = [1, 2, 3, 4, 5];
// 包含 5 个元素，这些元素的值初始化为 3
let c = [3; 5];
```

访问数组元素：

``` rust
let a = [1, 2, 3, 4, 5];
let first = a[0];
let second = a[1];
```

## 函数

``` rust
fn main() {
    println!("Hello, world!");
    another_function();
}

fn another_function() {
    println!("Another function.");
}
```

注意，源码中 another_function 定义在 main 函数之后；也可以定义在之前。Rust 不关心函数定义于何处，只要定义了就行。

函数参数：

``` rust
fn main() {
    print_labeled_measurement(5, 'h');
}

fn print_labeled_measurement(value: i32, unit_label: char) {
    println!("The measurement is: {}{}", value, unit_label);
}
```

## 语句和表达式

语句（statement）是执行一些操作但不返回值的指令。表达式（expression）计算并产生一个值。

## 带有返回值的函数

在 Rust 中，函数的返回值等同于函数体最后一个表达式的值。使用 return 关键字和指定值，可以从函数中提前返回；但大部分函数隐式返回最后一个表达式。

``` rust
fn five() -> i32 {
    5
}

fn main() {
    let x = five();
    println!("The value of x is: {}", x);
}
```

## 注释

使用//。

## if 表达式

``` rust
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

值得注意的是代码中的条件必须是 bool 值。如果条件不是 bool 值，我们将得到一个错误。


## 在 let 语句中使用 if

因为 if 是一个表达式，我们可以在 let 语句的右侧使用它来将结果赋值给一个变量。

``` rust
fn main() {
    let condition = true;
    let number = if condition { 5 } else { 6 };

    println!("The value of number is: {}", number);
}
```

## loop 循环

``` rust
fn main() {
    loop {
        println!("again!");
    }
}
```

如果存在嵌套循环，break 和 continue 应用于此时最内层的循环。你可以选择在一个循环上指定一个循环标签（loop label），然后将标签与 break 或 continue 一起使用，使这些关键字应用于已标记的循环而不是最内层的循环。

``` rust
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {}", count);
        let mut remaining = 10;

        loop {
            println!("remaining = {}", remaining);
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up;
            }
            remaining -= 1;
        }

        count += 1;
    }
    println!("End count = {}", count);
}
```

可以在用于停止循环的 break 表达式添加你想要返回的值；该值将从循环中返回，以便您可以使用它：

``` rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {}", result);
}
```

## while 循环

``` rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{}!", number);

        number -= 1;
    }

    println!("LIFTOFF!!!");
}
```

## for 循环

``` rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a {
        println!("the value is: {}", element);
    }
}
```

``` rust
fn main() {
    for number in (1..4).rev() {
        println!("{}!", number);
    }
    println!("LIFTOFF!!!");
}
```

## 所有权

首先，让我们看一下所有权的规则。当我们通过举例说明时，请谨记这些规则：

- Rust 中的每一个值都有一个被称为其 所有者（owner）的变量。
- 值在任一时刻有且只有一个所有者。
- 当所有者（变量）离开作用域，这个值将被丢弃。

Rust 会在编译阶段分析所有权，保证对无效引用的使用无法通过编译。

``` rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;  // 执行这一句之后，字符串的所有权被移动给s2，s1不再有效

    // 以下语句无法通过编译
    println!("{}, world!", s1); // s1不再有效，因此无法使用s1
}
```

如果我们确实需要深度复制 String 中堆上的数据，可以使用一个叫做 clone 的通用函数。

``` rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone();

    println!("s1 = {}, s2 = {}", s1, s2);
}
```

像整型这样的在编译时已知大小的类型被整个存储在栈上，所以拷贝其实际的值是快速的。这意味着没有理由在创建变量 y 后使 x 无效。

函数参数和返回值也可以转移所有权。

## 引用与借用

如果不需要转移所有权，可以使用引用：

``` rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

我们将创建一个引用的行为称为 借用（borrowing）。正如现实生活中，如果一个人拥有某样东西，你可以从他那里借来。当你使用完毕，必须还回去。

可变引用：

``` rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

可变引用有一个很大的限制：在同一时间，只能有一个对某一特定数据的可变引用。尝试创建两个可变引用的代码将会失败：

``` rust
fn main() {
    let mut s = String::from("hello");

    // 无法通过编译
    let r1 = &mut s;
    let r2 = &mut s;

    println!("{}, {}", r1, r2);
}
```

这个限制的好处是 Rust 可以在编译时就避免数据竞争。

## 悬垂引用

在具有指针的语言中，很容易通过释放内存时保留指向它的指针而错误地生成一个 悬垂指针（dangling pointer），所谓悬垂指针是其指向的内存可能已经被分配给其它持有者。相比之下，在 Rust 中编译器确保引用永远也不会变成悬垂状态：当你拥有一些数据的引用，编译器确保数据不会在其引用之前离开作用域。

Rust 会在编译阶段通过编译错误来避免出现悬垂引用。

## 引用的规则

- 在任意给定时间，要么只能有一个可变引用，要么只能有多个不可变引用。
- 引用必须总是有效的。

## 切片 Slice 类型

另一个没有所有权的数据类型是 slice。slice 允许你引用集合中一段连续的元素序列，而不用引用整个集合。

## 字符串 slice

字符串 slice（string slice）是 String 中一部分值的引用：

``` rust
let s = String::from("hello world");
let hello = &s[0..5];
let world = &s[6..11];
```

对于 Rust 的 .. range 语法，如果想要从索引 0 开始，可以不写两个点号之前的值。

依此类推，如果 slice 包含 String 的最后一个字节，也可以舍弃尾部的数字。

也可以同时舍弃这两个值来获取整个字符串的 slice。

``` rust
let s = String::from("hello");
let slice = &s[..2];
let slice = &s[3..];
let slice = &s[..];
```

字符串字面量就是 slice。

``` rust
let s = "Hello, world!";
```

这里 s 的类型是 &str：它是一个指向二进制程序特定位置的 slice。这也就是为什么字符串字面量是不可变的；&str 是一个不可变引用。

函数返回字符串slice：

``` rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

## 数组 slice

``` rust
let a = [1, 2, 3, 4, 5];
let slice = &a[1..3];
```

这个 slice 的类型是 &[i32]。

## 结构体

``` rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

fn main() {
    let user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };

    user1.email = String::from("anotheremail@example.com");
}
```

变量与字段同名时的字段初始化简写语法：

``` rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}

fn main() {
    let user1 = build_user(
        String::from("someone@example.com"),
        String::from("someusername123"),
    );
}
```

使用结构体更新语法从其他实例创建实例：

``` rust
let user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};

let user2 = User {
    email: String::from("another@example.com"),
    ..user1
};
```

## 元组结构体

``` rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
}
```

## 类单元结构体

我们也可以定义一个没有任何字段的结构体！它们被称为类单元结构体（unit-like structs），因为它们类似于 ()，即“元组类型”一节中提到的 unit 类型。类单元结构体常常在你想要在某个类型上实现 trait 但不需要在类型中存储数据的时候发挥作用。

``` rust
struct AlwaysEqual;

fn main() {
    let subject = AlwaysEqual;
}
```

## dbg! 宏

与 println! 宏打印到标准输出控制流（stdout）不同，调用 dbg! 宏会打印到标准错误控制流（stderr）。

dbg! 宏接收一个表达式的所有权，打印出代码中调用 dbg! 宏时所在的文件和行号，以及该表达式的结果值，并返回该值的所有权。

在 {} 中加入 :? 指示符告诉 println! 我们想要使用叫做 Debug 的输出格式。Debug 是一个 trait，它允许我们以一种对开发者有帮助的方式打印结构体，以便当我们调试代码时能看到它的值。

``` rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let scale = 2;
    let rect1 = Rectangle {
        width: dbg!(30 * scale),
        height: 50,
    };

    dbg!(&rect1);
}
```

## 为结构体定义方法

``` rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };
    let rect2 = Rectangle {
        width: 10,
        height: 40,
    };
    let rect3 = Rectangle {
        width: 60,
        height: 45,
    };

    println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
    println!("Can rect1 hold rect3? {}", rect1.can_hold(&rect3));
}
```

## 枚举

``` rust
enum IpAddrKind {
    V4,
    V6,
}
```

枚举值可以关联数据：

``` rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);
let loopback = IpAddr::V6(String::from("::1"));
```

另一个枚举的例子，它的成员中内嵌了多种多样的类型：

``` rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

也可以在枚举上定义方法：

``` rust
impl Message {
    fn call(&self) {
        // 在这里定义方法体
    }
}

let m = Message::Write(String::from("hello"));
m.call();
```

## Option 枚举

Rust 并没有空值，不过它确实拥有一个可以编码存在或不存在概念的枚举。这个枚举是 Option<T>，定义如下：

``` rust
enum Option<T> {
    Some(T),
    None,
}
```

## match 运算符

``` rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

## 匹配 Option<T>

``` rust
fn main() {
    fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            None => None,
            Some(i) => Some(i + 1),
        }
    }

    let five = Some(5);
    let six = plus_one(five);
    let none = plus_one(None);
}
```

Rust 中的匹配是穷举式的（exhaustive）：必须穷举到最后的可能性来使代码有效。

_ 通配符：

``` rust
fn main() {
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => reroll(),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn reroll() {}
}
```

如果无操作可以使用单元值：

``` rust
fn main() {
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => (), // 如果无操作可以使用单元值
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
}
```

## if let 语法

if let 语法让我们以一种不那么冗长的方式结合 if 和 let，来处理只匹配一个模式的值而忽略其他模式的情况。

``` rust
let some_u8_value = Some(0u8);
if let Some(3) = some_u8_value {
    println!("three");
}
```

## 包和crate

crate 是一个二进制项或者库。

包（package）是提供一系列功能的一个或者多个 crate。一个包会包含有一个 Cargo.toml 文件，阐述如何去构建这些 crate。

一个包中至多 只能 包含一个库 crate（library crate）；包中可以包含任意多个二进制 crate（binary crate）；包中至少包含一个 crate，无论是库的还是二进制的。

如果一个包同时含有 src/main.rs 和 src/lib.rs，则它有两个 crate：一个库和一个二进制项，且名字都与包相同。通过将文件放在 src/bin 目录下，一个包可以拥有多个二进制 crate：每个 src/bin 下的文件都会被编译成一个独立的二进制 crate。

## 定义模块来控制作用域与私有性

例如通过执行 cargo new --lib restaurant，来创建一个新的名为 restaurant 的库。

``` rust
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

以上代码对应的模块树为：

```
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

如果我们想要调用一个函数，我们需要知道它的路径。

- 绝对路径（absolute path）从 crate 根部开始，以 crate 名或者字面量 crate 开头。
- 相对路径（relative path）从当前模块开始，以 self、super 或当前模块的标识符开头。

Rust 中默认所有项（函数、方法、结构体、枚举、模块和常量）都是私有的。父模块中的项不能使用子模块中的私有项，但是子模块中的项可以使用他们父模块中的项。

可以通过使用 pub 关键字来创建公共项，使子模块的内部部分暴露给上级模块。

``` rust
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

super 类似于文件系统中以 .. 开头的语法：

``` rust
fn serve_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::serve_order();
    }

    fn cook_order() {}
}
```

创建公有的结构体：

``` rust
pub struct Breakfast {
    pub toast: String,
    seasonal_fruit: String,  // 该成员为私有
}
```

枚举与结构不同，如果我们将枚举设为公有，则它的所有成员都将变为公有：

``` rust
// Soup 和 Salad 都为公有
pub enum Appetizer {
    Soup,
    Salad,
}
```

## use 和 as

``` rust
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

## 使用 pub use 重导出名称

``` rust
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

## 嵌套路径来消除大量的 use 行

``` rust
use std::cmp::Ordering;
use std::io;
```

等价于：

``` rust
use std::{cmp::Ordering, io};
```

通配符：

``` rust
use std::collections::*;
```

## 将模块放入单独的文件

``` rust
mod front_of_house;

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

src/front_of_house.rs：

``` rust
pub mod hosting {
    pub fn add_to_waitlist() {}
}
```

`mod front_of_house;` 告诉 Rust 在另一个与模块同名的文件中加载模块的内容。

  [1]: https://rustwiki.org/zh-CN/book/title-page.html