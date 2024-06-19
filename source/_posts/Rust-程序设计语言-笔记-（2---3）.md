title: Rust 程序设计语言 笔记 （2 / 3）
categories: 日志
tags: [Rust]
date: 2024-04-17 12:35:00
---
[Rust 程序设计语言 中文版][1]

## vector

vector 允许我们在一个单独的数据结构中储存多个值，所有值在内存中彼此相邻排列。vector 只能储存相同类型的值。

``` rust
let v: Vec<i32> = Vec::new();
v.push(5);
v.push(6);
v.push(7);
v.push(8);

// 获取指定元素的引用
let third: &i32 = &v[2];

// 通过 get 获取元素，返回值为 Option<T>
match v.get(2) {
    Some(third) => println!("The third element is {}", third),
    None => println!("There is no third element."),
}
```

使用初始值来创建一个Vec：

``` rust
// Rust 可以推断出 v 的类型是 Vec<i32>，因此类型标注就不是必须的。
let v = vec![1, 2, 3];
```

遍历 vector 中的元素：

``` rust
let v = vec![100, 32, 57];

for i in &v {
    println!("{}", i);
}

for i in &mut v {
    *i += 50;
}
```

<!--more-->

使用枚举储存多种类型：

``` rust
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
```

## 字符串

Rust 的核心语言中只有一种字符串类型：str，字符串 slice，它通常以被借用的形式出现，&str。

称作 String 的类型是由标准库提供的，而没有写进核心语言部分，它是可增长的、可变的、有所有权的、UTF-8 编码的字符串类型。

``` rust
let mut s = String::new();

let data = "initial contents";
let s = data.to_string();
let s = "initial contents".to_string();
let s = String::from("initial contents");

s.push_str("bar");

let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // 注意 s1 被移动了，不能继续使用

let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");
let s = format!("{}-{}-{}", s1, s2, s3);  // 格式化字符串
```

字符串slice：

``` rust
// s 会是一个 &str，它包含字符串的头 4 个字节。
let hello = "helloworld";
let s = &hello[0..4];
```

遍历每个字符：

``` rust
for c in "hello".chars() {
    println!("{}", c);
}
```

遍历每个字节：

``` rust
for b in "hello".bytes() {
    println!("{}", b);
}
```

## HashMap

像 vector 一样，哈希 map 将它们的数据储存在堆上。

``` rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

// 只在键没有对应值时插入
// entry 函数的返回值是一个枚举，Entry，它代表了可能存在也可能不存在的值。
// Entry 的 or_insert 方法在键对应的值存在时就返回这个值的可变引用，
// 如果不存在则将参数作为新值插入并返回新值的可变引用。
scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);

// 获取元素，返回 Option<V>
let team_name = String::from("Blue");
let score = scores.get(&team_name);

// 遍历
for (key, value) in &scores {
    println!("{}: {}", key, value);
}

// 使用 collect 方法
let teams  = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];
let scores: HashMap<_, _> = teams.iter().zip(initial_scores.iter()).collect();

// 所有权
let field_name = String::from("Favorite color");
let field_value = String::from("Blue");
let mut map = HashMap::new();
map.insert(field_name, field_value);
// field_name 和 field_value 不再有效，所有权被转移
```

根据旧值更新一个值：

``` rust
use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();

for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}
```

## panic! 与不可恢复的错误

当出现 panic 时，程序默认会开始 展开（unwinding），这意味着 Rust 会回溯栈并清理它遇到的每一个函数的数据，不过这个回溯并清理的过程有很多工作。

另一种选择是直接 终止（abort），这会不清理数据就退出程序。那么程序所使用的内存需要由操作系统来清理。

如果你需要项目的最终二进制文件越小越好，panic 时通过在 Cargo.toml 的 [profile] 部分增加 panic = 'abort'，可以由展开切换为终止。例如，如果你想要在release模式中 panic 时直接终止：

``` toml
[profile.release]
panic = 'abort'
```

## Result 与可恢复的错误

标准库中很多函数返回 Result 类型：

``` rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

例如：

``` rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("Problem opening the file: {:?}", error)
        },
    };
}
```

与 Option 枚举一样，Result 枚举和其成员也被导入到了 prelude 中，所以就不需要在 match 分支中的 Ok 和 Err 之前指定 Result::。

匹配不同的错误：

``` rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => panic!("Problem opening the file: {:?}", other_error),
        },
    };
}
```

另一种写法（用到了闭包）：

``` rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt").unwrap_or_else(|error| {
        if error.kind() == ErrorKind::NotFound {
            File::create("hello.txt").unwrap_or_else(|error| {
                panic!("Problem creating the file: {:?}", error);
            })
        } else {
            panic!("Problem opening the file: {:?}", error);
        }
    });
}
```

## 失败时 panic 的简写：unwrap 和 expect

unwrap：如果 Result 值是成员 Ok，unwrap 会返回 Ok 中的值。如果 Result 是成员 Err，unwrap 会为我们调用 panic!。

``` rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
}
```

expect：类似 unwrap，允许我们选择 panic! 的错误信息，使用 expect 而不是 unwrap 并提供一个好的错误信息可以表明你的意图并更易于追踪 panic 的根源。

``` rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

## 传播错误

``` rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
```

以上代码的简写形式：

``` rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

进一步简写：

``` rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();
    File::open("hello.txt")?.read_to_string(&mut s)?;
    Ok(s)
}
```

将文件读取到一个字符串是相当常见的操作，所以 Rust 提供了名为 fs::read_to_string 的函数，它会打开文件、新建一个 String、读取文件的内容，并将内容放入 String，接着返回它。

``` rust
use std::io;
use std::fs;

fn read_username_from_file() -> Result<String, io::Error> {
    fs::read_to_string("hello.txt")
}
```

Result 值之后的 ? ：如果 Result 的值是 Ok，这个表达式将会返回 Ok 中的值而程序将继续执行。如果值是 Err，Err 将作为整个函数的返回值，就好像使用了 return 关键字一样，这样错误值就被传播给了调用者。

? 运算符可被用于返回值类型为 Result 的函数。

## 泛型

在函数定义中使用泛型：

``` rust
// <T: PartialOrd + Copy> 表示指定泛型实现特定的 trait
// trait 相当于接口
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```

结构体定义中的泛型：

``` rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
```

``` rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
```

## 枚举定义中的泛型

``` rust
enum Option<T> {
    Some(T),
    None,
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

## 方法定义中的泛型

``` rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };
    println!("p.x = {}", p.x());
}
```

注意必须在 impl 后面声明 T，这样就可以在 Point<T> 上实现的方法中使用它了。在 impl 之后声明泛型 T ，这样 Rust 就知道 Point 的尖括号中的类型是泛型而不是具体类型。

下面的代码意味着 Point<f32> 类型会有一个方法 distance_from_origin，而其他 T 不是 f32 类型的 Point<T> 实例则没有定义此方法。

``` rust
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

## trait：定义共享的行为

trait 告诉 Rust 编译器某个特定类型拥有可能与其他类型共享的功能。

trait 类似于其他语言中常被称为 接口（interfaces）的功能。

定义trait：

``` rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

为类型实现trait：

``` rust
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
```

调用 trait 方法：

``` rust
let tweet = Tweet {
    username: String::from("horse_ebooks"),
    content: String::from("of course, as you probably already know, people"),
    reply: false,
    retweet: false,
};

println!("1 new tweet: {}", tweet.summarize());
```

## trait 默认实现

有时为 trait 中的某些或全部方法提供默认的行为，而不是在每个类型的每个实现中都定义自己的行为是很有用的。这样当为某个特定类型实现 trait 时，可以选择保留或重载每个方法的默认行为。

``` rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
```

## trait 作为参数

``` rust
pub fn notify(item: impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

Trait Bound 语法：

``` rust
pub fn notify<T: Summary>(item: T) {
    println!("Breaking news! {}", item.summarize());
}
```

指定多个trait：

``` rust
pub fn notify(item: impl Summary + Display) {}
pub fn notify<T: Summary + Display>(item: T) {}
```

有多个泛型参数的函数在名称和参数列表之间会有很长的 trait bound 信息，这使得函数签名难以阅读。为此，Rust 有另一个在函数签名之后的 where 从句中指定 trait bound 的语法。

``` rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: T, u: U) -> i32 {}

fn some_function<T, U>(t: T, u: U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{}
```

## 返回实现了 trait 的类型

``` rust
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
        reply: false,
        retweet: false,
    }
}
```

## 使用 trait bound 有条件地实现方法

以下代码只有那些为 T 类型实现了 PartialOrd trait （来允许比较） 和 Display trait （来启用打印）的 Pair<T> 才会实现 cmp_display 方法：

``` rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self {
            x,
            y,
        }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

## 生命周期标注

Rust 中的每一个引用都有其 生命周期（lifetime），也就是引用保持有效的作用域。大部分时候生命周期是隐含并可以推断的，正如大部分时候类型也是可以推断的一样。

``` rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}

// 无法通过编译
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

以上代码中 longest 函数无法通过编译，因为 Rust 编译器在编译阶段需要推断每个引用的生命周期，以上代码中 longest 函数在运行时才能决定返回的是哪个引用，所以导致编译器无法推断生命周期。

为了修复这个错误，我们将增加泛型生命周期参数来定义引用间的关系以便借用检查器可以进行分析。

``` rust
&i32        // 引用
&'a i32     // 带有显式生命周期的引用
&'a mut i32 // 带有显式生命周期的可变引用
```

单个生命周期标注本身没有多少意义，因为生命周期标注告诉 Rust 多个引用的泛型生命周期参数如何相互联系的。

函数签名中的生命周期标注：

``` rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

现在函数签名表明对于某些生命周期 'a，函数会获取两个参数，他们都是与生命周期 'a 存在的一样长的字符串 slice。函数会返回一个同样也与生命周期 'a 存在的一样长的字符串 slice。它的实际含义是 longest 函数返回的引用的生命周期与传入该函数的引用的生命周期的较小者一致。

## 结构体定义中的生命周期标注

``` rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.')
        .next()
        .expect("Could not find a '.'");
    let i = ImportantExcerpt { part: first_sentence };
}
```

这个标注意味着 ImportantExcerpt 的实例不能比其 part 字段中的引用存在的更久。

## 方法定义中的生命周期标注

函数或方法的参数的生命周期被称为 输入生命周期（input lifetimes），而返回值的生命周期被称为 输出生命周期（output lifetimes）。

``` rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}
```

## 静态生命周期

'static，其生命周期能够存活于整个程序期间。所有的字符串字面量都拥有 'static 生命周期。

``` rust
let s: &'static str = "I have a static lifetime.";
```

这个字符串的文本被直接储存在程序的二进制文件中而这个文件总是可用的。因此所有的字符串字面量都是 'static 的。

## 编写自动化测试

创建库项目：

```
cargo new adder --lib
```

src/lib.rs：

``` rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

fn 行之前的 #[test]：这个属性表明这是一个测试函数。

cargo test 命令会运行项目中所有的测试。

宏断言：

- assert! 测试布尔值
- assert_eq! 测试相等
- assert_ne! 测试不相等

自定义失败信息：

``` rust
assert!(
    result.contains("Carol"),
    "Greeting did not contain name, value was `{}`", result
);
```

使用 should_panic 检查 panic ：

``` rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

可以给 should_panic 属性增加一个可选的 expected 参数。测试工具会确保错误信息中包含其提供的文本。

``` rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic(expected = "Guess value must be less than or equal to 100")]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

将 Result<T, E> 用于测试：

``` rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four"))
        }
    }
}
```

在函数体中，不同于调用 assert_eq! 宏，而是在测试通过时返回 Ok(())，在测试失败时返回带有 String 的 Err。

为了编写集成测试，需要在项目根目录创建一个 tests 目录，与 src 同级。Cargo 知道如何去寻找这个目录中的集成测试文件。

接着可以随意在这个目录中创建任意多的测试文件，Cargo 会将每一个文件当作单独的 crate 来编译。

## 读取命令行参数

``` rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();
    println!("{:?}", args);
}
```

## 读取文件

``` rust
let contents = fs::read_to_string(filename)
    .expect("Something went wrong reading the file");

println!("With text:\n{}", contents);
```

## 输出到标准错误

``` rust
eprintln!("Application error: {}", e);
```

## 闭包：可以捕获环境的匿名函数

不同于函数，闭包允许捕获调用者作用域中的值。

``` rust
fn  add_one_v1   (x: u32) -> u32 { x + 1 }   // 函数
let add_one_v2 = |x: u32| -> u32 { x + 1 };  // 闭包
let add_one_v3 = |x|             { x + 1 };  // 闭包
let add_one_v4 = |x|               x + 1  ;  // 闭包
```

在结构体中存放闭包：

``` rust
struct Cacher<T>
    where T: Fn(u32) -> u32
{
    calculation: T,
    value: Option<u32>,
}
```

Fn 系列 trait 由标准库提供。所有的闭包都实现了 trait Fn、FnMut 或 FnOnce 中的一个。

闭包会捕获其环境：

``` rust
fn main() {
    let x = 4;

    let equal_to_x = |z| z == x;

    let y = 4;

    assert!(equal_to_x(y));
}
```

这里，即便 x 并不是 equal_to_x 的一个参数，equal_to_x 闭包也被允许使用变量 x，因为它与 equal_to_x 定义于相同的作用域。

闭包可以通过三种方式捕获其环境，他们直接对应函数的三种获取参数的方式：获取所有权，可变借用和不可变借用。

- FnOnce 消费从周围作用域捕获的变量，闭包周围的作用域被称为其 环境，environment。为了消费捕获到的变量，闭包必须获取其所有权并在定义闭包时将其移动进闭包。其名称的 Once 部分代表了闭包不能多次获取相同变量的所有权的事实，所以它只能被调用一次。
- FnMut 获取可变的借用值所以可以改变其环境
- Fn 从其环境获取不可变的借用值

如果你希望强制闭包获取其使用的环境值的所有权，可以在参数列表前使用 move 关键字。这个技巧在将闭包传递给新线程以便将数据移动到新线程中时最为实用。

``` rust
fn main() {
    let x = vec![1, 2, 3];

    let equal_to_x = move |z| z == x;

    // 这里不能再使用x，因为x的所有权已经被转移
    //println!("can't use x here: {:?}", x);

    let y = vec![1, 2, 3];

    assert!(equal_to_x(y));
}
```

## 迭代器

``` rust
let v1 = vec![1, 2, 3];
let v1_iter = v1.iter();

for val in v1_iter {
    println!("Got: {}", val);
}
```

迭代器都实现了一个叫做 Iterator 的定义于标准库的 trait。这个 trait 的定义看起来像这样：

``` rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // 此处省略了方法的默认实现
}
```

type Item 语法参见关联类型。

``` rust
fn iterator_demonstration() {
    let v1 = vec![1, 2, 3];
    let mut v1_iter = v1.iter();

    assert_eq!(v1_iter.next(), Some(&1));
    assert_eq!(v1_iter.next(), Some(&2));
    assert_eq!(v1_iter.next(), Some(&3));
    assert_eq!(v1_iter.next(), None);
}
```

## 迭代器适配器

允许我们将当前迭代器变为不同类型的迭代器。可以链式调用多个迭代器适配器。

``` rust
let v1: Vec<i32> = vec![1, 2, 3];
let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();
assert_eq!(v2, vec![2, 3, 4]);
```

## 实现 Iterator trait 来创建自定义迭代器

``` rust
struct Counter {
    count: u32,
}

impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
}

impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;

        if self.count < 6 {
            Some(self.count)
        } else {
            None
        }
    }
}

fn calling_next_directly() {
    let mut counter = Counter::new();

    assert_eq!(counter.next(), Some(1));
    assert_eq!(counter.next(), Some(2));
    assert_eq!(counter.next(), Some(3));
    assert_eq!(counter.next(), Some(4));
    assert_eq!(counter.next(), Some(5));
    assert_eq!(counter.next(), None);
}

fn using_other_iterator_trait_methods() {
    let sum: u32 = Counter::new()
        .zip(Counter::new().skip(1))
        .map(|(a, b)| a * b)
        .filter(|x| x % 3 == 0)
        .sum();
    assert_eq!(18, sum);
}
```

## 迭代器的性能

闭包和迭代器是 Rust 受函数式编程语言观念所启发的功能。他们对 Rust 以底层的性能来明确的表达高级概念的能力有很大贡献。闭包和迭代器的实现达到了不影响运行时性能的程度。这正是 Rust 竭力提供零成本抽象的目标的一部分。

## 构建

```
cargo build            # dev 构建
cargo build --release  # release 构建
```

Cargo.toml 中指定构建配置：

``` toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

## 文档注释

``` rust
/// Adds one to the number given.
///
/// # Examples
///
/// ```
/// let arg = 5;
/// let answer = my_crate::add_one(arg);
///
/// assert_eq!(6, answer);
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

`cargo doc --open` 会构建当前 crate 文档（同时还有所有 crate 依赖的文档）的 HTML 并在浏览器中打开。

`cargo test` 也会像测试那样运行文档中的示例代码！

`//!` 为 crate 或模块整体提供文档：

``` rust
//! # My Crate
//!
//! `my_crate` is a collection of utilities to make performing certain
//! calculations more convenient.
```

## pub use

使用 pub use 语句来重导出项到顶层结构。

``` rust
pub use self::kinds::PrimaryColor;
pub use self::kinds::SecondaryColor;
pub use self::utils::mix;

pub mod kinds {
    // --snip--
}

pub mod utils {
    // --snip--
}
```

## 工作空间

工作空间 是一系列共享同样的 Cargo.lock 和输出目录的包。

比如 Cargo.toml：

``` toml
[workspace]

members = [
    "adder",
    "add-one",
]
```

目录结构：

```
├── Cargo.lock
├── Cargo.toml
├── add-one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

add-one/src/lib.rs：

``` rust
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

adder/src/main.rs：

``` rust
use add_one;

fn main() {
    let num = 10;
    println!("Hello, world! {} plus one is {}!", num, add_one::add_one(num));
}
```

adder/Cargo.toml：

``` toml
[dependencies]
add-one = { path = "../add-one" }
```

  [1]: https://rustwiki.org/zh-CN/book/title-page.html