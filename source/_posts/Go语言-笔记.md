title: Go语言 笔记
categories: 日志
tags: [golang]
date: 2022-04-05 14:38:00
---
《Head First Go》笔记

## Hello World

``` go
package main

import "fmt"

func main() {
  fmt.Println("Hello, World!")
}
```

典型的Go文件布局：

 1. package子句
 2. 任何import语句
 3. 实际代码

每个Go文件都必须以package子句开头。

每个Go文件都必须导入它引用的每个包。

Go文件必须只导入它们引用的包。

Go查找名为main的函数并首先运行。

Go中的所有内容都区分大小写。

Println函数是fmt包的一部分，因此Go需要在函数调用之前使用报名。


<!--more-->


## 导入多个包

``` go
package main

import (
  "fmt"
  "math"
  "strings"
)

func main() {
  fmt.Println(math.Floor(2.75))
  fmt.Println(strings.Title("head first go"))
}
```

## 符文

字符串通常用于表示一系列文本字符，而Go的符文（rune）则用于表示单个字符。

字符串字面量由双引号（"）包围，但rune字面量由单引号（'）包围。

## 布尔值

布尔值只能是两个值中的一个：true或false。

## 类型

 - int
 - float64
 - bool
 - string

你可以通过将任何值传递给reflect包的TypeOf函数，来查看它们的类型。

``` go
package main

import (
  "fmt"
  "reflect"
)

func main() {
  fmt.Println(reflect.TypeOf(42))
  fmt.Println(reflect.TypeOf(3.14))
  fmt.Println(reflect.TypeOf(true))
  fmt.Println(reflect.TypeOf("Hello, Go!"))
}
```

## 声明变量

``` go
var quantity int
var length, width float64
var customerName string

quantity = 2
customerName = "Damon Cole"

// 同一语句中为多个变量赋值
length, width = 1.2, 2.4

// 声明的同时赋值
var quantity int = 4
var length, width float64 = 1.2, 2.4
var customerName string = "Damon Cole"

// 声明的同时赋值可省略变量类型
var quantity = 4
var length, width = 1.2, 2.4
var customerName = "Damon Cole"

// 声明的同时赋值可简写为短变量声明
quantity := 4
length, width := 1.2, 2.4
customerName := "Damon Cole"
```

如果声明一个变量而没有给它赋值，该变量将包含其类型的零值。对于数值类型，零值实际上就是0，字符串变量的零值是空字符串，布尔变量的零值是false。

一个变量只能声明一次。

只能给变量赋相同类型的值。

所有声明的变量都必须在程序中使用，如果删除了使用变量的代码，必须也要删除声明。


## 命名规则

语言强制规则：

名称必须以字母开头，并且可以有任意数量的额外的字母和数字。

如果变量、函数或类型的名称以大写字母开头，则认为它是导出的，可以从当前包之外的包访问它。

社区遵循的约定：

如果一个名称由多个单词组成，那么第一个单词之后的每个单词都应该首字母大写，并且它们应该连接在一起，中间没有空格，比如topPrice、RetryConnection，等等。（名称的第一个字母只有在你想从包中导出时才应大写。）

当名称的含义在上下文中很明显时，Go社区的惯例是缩写它：用i代替index，用max代替maximum，等等。

## 转换

``` go
var myInt int = 2
fmt.Println(reflect.TypeOf(myInt))
fmt.Println(reflect.TypeOf(float64(myInt)))
```

## 编译Go代码

``` bash
# 格式化代码，这一步不是必需的，但无论如何这是个好主意
go fmt hello.go
# 这将向当前目录添加一个可执行文件
go build hello.go
# 运行程序
./hello

# go run 命令编译并运行源文件，而不将可执行文件保存到当前目录
# 你将立即看到程序输出。如果对源代码进行更改，则不必执行单独的编译步骤，
# 只需使用go run运行代码，即可立即看到结果。当你在处理小程序时，
# go run是一个很方便的工具！
go run hello
```

## 调用方法

``` go
package main

import (
  "fmt"
  "time"
)

func main() {
  var now time.Time = time.Now()
  var year int = now.Year()
  fmt.Println(year)
}
```

``` go
package main

import (
  "fmt"
  "strings"
)

func main() {
  broken := "G# r#cks!"
  replacer := strings.NewReplacer("#", "o")
  fixed := replacer.Replace(broken)
  fmt.Println(fixed)
}
```

## 注释

单行注释和多行注释和C++一样。

## 条件语句

实例，评分程序：

``` go
package main

import (
  "bufio"
  "fmt"
  "log"
  "os"
  "strings"
  "strconv"
)

func main() {
  fmt.Print("Enter a grade: ")

  // 读取键盘输入
  reader := bufio.NewReader(os.Stdin)
  input, err := reader.ReadString('\n')
  if err != nil {
    // 如果有错误则打印信息并退出
    log.Fatal(err)
  }

  // 将换行符从输入中删除
  input = strings.TrimSpace(input)
  // 将字符串转换为float64
  grade, err := strconv.ParseFloat(input, 64)
  if err != nil {
    log.Fatal(err)
  }

  var status string
  if grade >= 60 {
    status = "passing"
  } else {
    status = "failing"
  }

  fmt.Println("A grade of", grade, "is", status)
}
```

## 循环

实例，猜数字游戏：

``` go
package main

import (
  "bufio"
  "fmt"
  "log"
  "math/rand"
  "os"
  "strconv"
  "strings"
  "time"
)

func main() {
  seconds := time.Now().Unix()
  rand.Seed(seconds)

  // 1 - 100 之间的随机数
  target := rand.Intn(100) + 1
  fmt.Println("I've chosen a random number between 1 and 100.")
  fmt.Println("Can you guess it?")

  reader := bufio.NewReader(os.Stdin)
  success := false

  for guesses := 0; guesses < 10; guesses++ {
    fmt.Println("You have", 10 - guesses, "guesses left.")
    fmt.Print("Make a guess: ")
    input, err := reader.ReadString('\n')
    if err != nil {
      log.Fatal(err)
    }

    input = strings.TrimSpace(input)
    // 字符串转为整数
    guess, err := strconv.Atoi(input)
    if err != nil {
      log.Fatal(err)
    }

    if guess < target {
      fmt.Println("Oops. Your guess was LOW.")
    } else if guess > target {
      fmt.Println("Oops. Your guess was HIGH.")
    } else {
      success = true
      fmt.Println("Good job! You guessed it!")
      break
    }
  }

  if !success {
    fmt.Println("Sorry, you didn't guess my number. It was:", target)
  }
}
```

## 定义函数

``` go
package main

import "fmt"

// 返回两个值
func paintNeeded(width float64, height float64) (float64, error) {
  if width < 0 {
    return 0, fmt.Errorf("a width of %0.2f is invalid", width)
  }

  if height < 0 {
    return 0, fmt.Errorf("a height of %0.2f is invalid", height)
  }

  area := width * height
  return area / 10.0, nil
}

func main() {
  amount, err := paintNeeded(4.2, 3)
  if err != nil {
    log.Fatal(err)
  }
  fmt.Printf("%0.2f liters needed\n", amount)
}
```

## 指针

``` go
func main() {
  amount := 6

  // 传递指针
  double(&amount)
  fmt.Println(amount)
}

// 参数为int型的指针
func double(number *int) {
  *number *= 2
}
```

与其他语言不同，在Go中，返回一个指向函数局部变量的指针是可以的。即使该变量不在作用域内，只要你仍然拥有指针，Go将确保你仍然可以访问该值。

## 工作区

Go工具在你的计算机上名为工作区的特殊目录（文件夹）中查找包代码。默认情况下，工作区是当前用户主目录中名为go的目录。

```
go     // 工作区目录
  bin  // 保存可执行程序
  pkg  // 保存已编译的包代码
  src  // 保存源代码
    doodad    // 包
    gizmo     // 包
      gizmo.go
      plug.go
```

## 创建一个包

```
go
  src
    greeting
      greeting.go
```

``` go
package greeting

import "fmt"

func Hello() {
  fmt.Println("Hello!")
}

func Hi() {
  fmt.Println("Hi!")
}
```

通常，包名应该与保存它的目录名相匹配，但是main包是该规则的一个例外。

使用greeting包：

``` go
package main

import "greeting"

func main() {
  greeting.Hello()
  greeting.Hi()
}
```

## 包命名规范

 - 包名应全部为小写。
 - 如果含义相当明显，名称应该缩写（如fmt）。
 - 如果可能的话，应该是一个词。如果需要两个词，不应该用下划线分隔，第二个词也不应该大写。（strconv包就是一个例子。）
 - 导入的包名可能与本地变量名冲突，所以不要使用包用户可能也想使用的名称。（例如，如果fmt包被命名为format，那么导入该包的任何人如果把一个局部变量也命名为format，则将面临冲突的风险）。

## 常量

 - 使用const关键字而不是var关键字。
 - 必须在声明常量时赋值；不能像变量那样以后赋值。
 - 变量有:=短变量声明语法，但是常量没有等效的语法。

``` go
const TriangleSides int = 3
const SquareSides = 4
```

## go install

当我们使用 go run 时，Go必须编译程序以及它所依赖的所有包，然后才能执行。当编译完成后，它会丢弃编译后的代码。

`go build` 命令进行编译并把可执行二进制文件（即使没有安装Go也可以执行的文件）保存在当前目录中。

`go install` 命令也保存编译后的可执行程序的二进制版本，但保存在定义良好、易于访问的位置：Go工作区中的bin目录。

请确保将“src”中的目录名传递给“go install”，而不是.go文件名！默认情况下，“go install”未设置成直接处理.go文件。

## GOPATH环境变量

`GOPATH`是一个环境变量，Go工具会参考它来查找工作区位置。大多数Go开发人员将他们所有代码都保存在一个工作区中，并且不会更改其默认位置。但是如果你愿意，可以使用`GOPATH`将你的工作区转移到其他目录。

## go get

使用 `go get` 下载和安装包。

``` bash
go get github.com/headfirstgo/greeting
```

go工具将连接到github.com，在/headfirstgo/greeting路径下载Git存储库，并将其保存在Go工作区的src目录中。

## go doc

使用 `go doc` 阅读包文档，文档来自于文档注释。

例如，我们可以通过运行 `go doc strconv` 获得关于strconv包的信息。

我们可以使用 `go doc strconv ParseFloat` 来查看包内函数的文档。

## 文档注释

``` go
// Package keyboard reads user input from the keyboard
package keyboard

import (
  "bufio"
  "os"
  "strconv"
  "strings"
)

// GetFloat reads a floating-point number from the keyboard.
// It returns the number read and any error encountered.
func GetFloat() (float64, error) {
  // ...
}
```

## 本地文档Web服务器

``` bash
godoc -http=:8080
```

除了来自Go标准库的包之外，godoc工具还为Go工作区中的任何包构建HTML文档。这些包可能是你安装的第三方的包，也可能是你自己编写的包。

## 数组

``` go
var notes [7]string
notes[0] = "do"
notes[1] = "re"
notes[2] = "mi"
fmt.Println(notes[0])
fmt.Println(notes[1])

// 使用数组字面量
var notes [7]string = [7]string{"do", "re", "mi", "fa", "so", "la", "ti"}
var primes [5]int = [5]int{2, 3, 5, 7, 11}

// 短变量声明
primes := [5]int{2, 3, 5, 7, 11}

// 遍历数组
for i := 0; i < len(notes); i++ {
  fmt.Println(i, notes[i])
}

// 遍历数组，使用 for range
for index, note := range notes {
  fmt.Println(index, note)
}

// 对不需要的变量使用空白标识符
for _, note := range notes {
  fmt.Println(index, note)
}
```

## 读取文件

``` go
package main

import (
  "bufio"
  "fmt"
  "log"
  "os"
)

func main() {
  file, err := os.Open("data.txt")
  if err != nil {
    log.Fatal(err)
  }

  scanner := bufio.NewScanner(file)
  for scanner.Scan() {
    // 读取一行
    fmt.Println(scanner.Text())
  }

  err = file.Close()
  if err != nil {
    log.Fatal(err)
  }

  if scanner.Err() != nil {
    log.Fatal(scanner.Err())
  }
}
```

## 切片

``` go
var myArray [5]int  // 数组
var mySlice []int   // 切片
```

不像数组变量，声明切片变量并不会自动创建一个切片。为此，你可以调用内建的make函数。

``` go
var notes []string
notes = make([]string, 7)

primes := make([]int, 5)

fmt.Println(len(notes))
fmt.Println(len(primes))

for i := 0; i < len(notes); i++ {
  fmt.Println(notes[i])
}

for _, note := range notes {
  fmt.Println(note)
}

// 切片字面量
notes := []string{"do", "re", "mi", "fa", "so", "la", "ti"}
```

每一个切片都构建于一个底层的数组之上。实际上是底层的数组存储了切片的数据；切片仅仅是数组中的一部分（或者所有）元素的视图。

``` go
underlyingArray := [5]string{"a", "b", "c", "d", "e"}
slice1 := underlyingArray[0:3]  // 切片，左开右闭区间
fmt.Println(slice1)
```

使用append函数向切片中追加数据，如果底层数组容量不够则会重新创建容量更大的底层数组然后把数据拷贝过去，类似C++中的vector。

``` go
slice := []string{"a", "b"}
fmt.Println(slice, len(slice))
slice = append(slice, "c")
fmt.Println(slice, len(slice))
slice = append(slice, "d", "e")
fmt.Println(slice, len(slice))
```

## 处理命令行参数

`os.Args`是一个切片，第一个元素是程序名。

``` go
package main

import (
  "fmt"
  "os"
)

func main() {
  fmt.Println(os.Args)
}
```

## 可变长参数

``` go
func serveralInts(numbers ...int) {
  // 参数被收集到一个切片中
  fmt.Println(numbers)
}

func main() {
  severalInts(1)
  severalInts(1, 2, 3)
}
```

``` go
package main

import (
  "fmt"
  "math"
)

func maximum(numbers ...float64) float64 {
  max := math.Inf(-1) // 很小的数
  for _, number := range numbers {
    if number > max {
      max = number
    }
  }
  return max
}

func main() {
  fmt.Println(maximum(71.8, 56.2, 89.5))
  fmt.Println(maximum(90.7, 89.7, 98.5, 92.3))
}
```

为可变长参数函数传入切片：

``` go
func severalInts(numbers ...int) {
  fmt.Println(numbers)
}

func mix(num int, flag bool, strings ...string) {
  fmt.Println(num, flag, strings)
}

func main() (
  intSlice := []int{1, 2, 3}
  severalInts(intSlice...)

  stringSlice := []string{"a", "b", "c", "d"}
  mix(1, true, stringSlice...)
)
```

## 映射

``` go
var ranks map[string]int      // 声明一个map变量
ranks = make(map[string]int)  // 实际创建一个map

// 短变量声明
ranks := make(map[string]int)

// map字面量
ranks := map[string]int{"bronze": 3, "silver": 2, "gold": 1}
fmt.Println(ranks["gold"])
fmt.Println(ranks["bronze"])
elements := map[string]string{
  "H": "Hydrogen",
  "Li": "Lithium"
}
fmt.Println(elements["H"])
fmt.Println(elements["Li"])
```

获取某个键的值的时候，如果不存在则返回零值。

取值的时候判断值是否已存在：

``` go
counters := map[string]int{"a": 3, "b": 0}
var value int
var ok bool
value, ok = counters["a"]

// 删除键值对
delete(counters, "b")

// 遍历map
grades := map[string]float64{"Alma": 74.2, "Rohit": 86.5, "Carl": 59.7}
for name, grade := range grades {
  fmt.Printf("%s has a grade of %0.1f%%\n", name, grade)
}
```

## struct

``` go
var subscriber struct {
  name   string
  rate   float64
  active bool
}

subscriber.name = "Aman Singh"
subscriber.rate = 4.99
subscriber.active = true
fmt.Println("Name:", subscriber.name)
fmt.Println("Monthly rate:", subscriber.rate)
fmt.Println("Active?", subscriber.active)
```

将结构定义成类型：

``` go
type car struct {
  name     string
  topSpeed float64
}

var porsche car
porsche.name = "Porsche 911 R"
porsche.topSpeed = 323
fmt.Println("Name:", porsche.name)
fmt.Println("Top speed:", porsche.topSpeed)
```

传递结构的指针作为参数：

``` go
package main

import "fmt"

type subscriber struct {
  name   string
  rate   float64
  active bool
}

func applyDiscount(s *subscriber) {
  s.rate = 4.99
}

func main() {
  var s subscriber
  applyDiscount(&s)
  fmt.Println(s.rate)
}
```

结构字面量：

``` go
s := subscriber{name: "Aman Singh", rate: 4.99, active: true}
```

匿名字段：

``` go
type Subscriber struct {
  Name string
  Rate float64
  Active bool
  HomeAddress Address
}

type Address struct {
  Name string
}

var s Subscriber
s.HomeAddress.Name = "abc"

// 使用匿名字段
type Subscriber struct {
  Name string
  Rate float64
  Active bool
  Address
}

type Address struct {
  Name string
}

var s Subscriber
s.Address.Name = "abc"  // 方式1
s.Name = "abc"          // 方式2
```

## 定义方法

``` go
package main

import "fmt"

type MyType string  // 定义一个新类型

// 按照惯例，Go开发者通常使用一个字母作为名称——小写的接收器类型名称的首字母。
// （这就是为什么我们将m来用作MyType接收器的参数名称。）
func (m MyType) sayHi() {
  fmt.Println("Hi from", m)
}

func main() {
  value := MyType("a MyType value")
  value.sayHi()
  anotherValue := MyType("another value")
  anotherValue.sayHi()
}
```

如果想要修改接收器的值，接收器参数需要使用指针类型：

``` go
type Number int

func (n *Number) Double {
  *n *= 2
}

func main() {
  number := Number(4)
  number.Double()
  // 现在number的值为8
}
```

为了一致性，你所有的类型函数接受值类型，或者都接受指针类型，但是你应该避免混用的情况。

## 封装

变量名、类型名或函数名小写字母开头表示不导出，大写字母开头表示导出。

``` go
package calendar

import "erros"

type Date struct {
  year  int
  month int
  day   int
}

func (d *Date) Year() int {
  return d.year
}

func (d *Date) Month() int {
  return d.month
}

func (d *Date) Day() int {
  return d.day
}

func (d *Date) SetYear(year int) error {
  if year < 1 {
    return erros.New("invalid year")
  }
  d.year = year
  return nil
}

func (d *Date) SetMonth(month int) error {
  if month < 1 || month > 12 {
    return errors.New("invalid month")
  }
  d.month = month
  return nil
}

func (d *Date) SetDay(day int) error {
  if day < 1 || day > 31 {
    return errors.New("invalid day")
  }
  d.day = day
  return nil
}
```

## 接口

在Go中，一个接口被定义为特定值预期具有的一组方法。你可以把接口看作需要类型实现的一组行为。

使用interface关键字定义一个接口类型，后面跟着一个花括号，内部含有一组方法，以及方法期望的参数和返回值。

任何拥有接口定义的所有方法的类型被称作满足那个接口。一个满足接口的类型可以用在任何需要接口的地方。

``` go
package main

import "fmt"

type Whistle string

func (w Whistle) MakeSound() {
  fmt.Println("Tweet!")
}

type Horn string

func (h Horn) MakeSound() {
  fmt.Println("Honk!")
}

type Robot string

func (r Robot) MakeSound() {
  fmt.Println("Beep Boop")
}

func (r Robot) Walk() {
  fmt.Println("Powering legs")
}

// 定义一个接口类型
type NoiseMaker interface {
  MakeSound()
}

func play(n NoiseMaker) {
  n.MakeSound()
}

func main() {
  play(Robot("Botco Ambler"))
}
```

## 类型断言

类似于C++中的向下转型，将接口类型转型为具体类型：

``` go
type Robot string

func (r Robot) MakeSound() {
  fmt.Println("Beep Boop")
}

func (r Robot) Walk() {
  fmt.Println("Powering legs")
}

type NoiseMaker interface {
  MakeSound()
}

func main() {
  var noiseMaker NoiseMaker = Robot("Botco Ambler")
  noiseMaker.MakeSound()

  var robot Robot = noiseMaker.(Robot)
  robot.Walk()
}
```

如果转型失败将产生异常。

``` go
// 避免产生异常
robot, ok := noiseMaker.(Robot)
if ok {
  robot.Walk()
}
```

## error 接口

``` go
type error interface {
  Error() string
}
```

## Stringer 接口

``` go
type Stringer interface {
  String() string
}
```

## 空接口

空接口 `interface{}` 可接收任何类型的值。

## 延迟函数调用

``` go
package main

import "fmt"

func Socialize() {
  defer fmt.Println("Goodbye!")  // 函数调用被推迟
  fmt.Println("Hello!")
  fmt.Println("Nice weather, eh?")
}

func main() {
  Socialize()
}

// 输出：
// Hello!
// Nice weather, eh?
// Goodbye!
```

``` go
package main

import (
  "fmt"
  "log"
)

func Socialize() {
  defer fmt.Println("Goodbye!")
  fmt.Println("Hello!")
  return fmt.Errorf("I don't want to talk.")
  fmt.Println("Nice weather, eh?")
  return nil
}

func main() {
  err := Socialize()
  if err != nil {
    log.Fatal(err)
  }
}

// 输出：
// Hello!
// Goodbye!
// 2018/04/08 19:24:48 I don't want to talk.
```

“defer”关键字确保函数调用发生，即使调用函数提前退出了。

实例，使用延迟函数调用确保文件关闭：

``` go
func OpenFile(fileName string) (*os.File, error) {
  fmt.Println("Opening", fileName)
  return os.Open(fileName)
}

func CloseFile(file *os.File) {
  fmt.Println("Closing file")
  file.Close()
}

func GetFloats(fileName string) ([]float64, error) {
  var numbers []float64
  file, err := OpenFile(fileName)
  if err != nil {
    return nil, err
  }

  defer CloseFile(file)

  scanner := bufio.NewScanner(file)
  for scanner.Scan() {
    number, err := strconv.ParseFloat(scanner.Text(), 64)
    if err != nil {
      return nil, err
    }

    number = append(numbers, number)
  }

  if scanner.Err() != nil {
    return nil, scanner.Err()
  }

  return numbers, nil
}
```

## 递归列出目录中的文件

``` go
package main

import (
  "fmt"
  "io/ioutil"
  "log"
  "path/filepath"
)

func scanDirectory(path string) error {
  fmt.Println(path)
  files, err := ioutil.ReadDir(path)
  if err != nil {
    return err
  }

  for _, file := range files {
    filePath := filepath.Join(path, file.Name())
    if file.IsDir() {
      err := scanDirectory(filePath)
      if err != nil {
        return err
      }
    } else {
      fmt.Println(filePath)
    }
  }
  return nil
}

func main() {
  err := scanDirectory("go")
  if err != nil {
    log.Fatal(err)
  }
}
```

## 发起一个panic

当程序出现panic时，当前函数停止运行，程序打印日志消息并崩溃。

``` go
package main

func main() {
  panic("oh, no, we're going down")
}
```

当程序发生panic时，panic输出中包含堆栈跟踪，即调用堆栈列表。这对于确定导致程序崩溃的原因很有用。

当程序出现panic时，所有延迟的函数调用仍然会被执行。如果有多个延迟调用，它们的执行顺序将与被延迟的顺序相反。

事实上，调用panic并不是处理错误的理想方法。

无法访问的文件、网络故障和错误的用户输入通常应该被认为是“正常的”，应该通过错误值来进行适当的处理。通常，调用panic应该留给“不可能的”情况：错误表示的是程序中的错误，而不是用户方面的错误。

恢复panic：

``` go
func calmDown() {
  recover()
}

func freakOut() {
  defer calmDown()
  panic("oh no")
  fmt.Println("I won't be run!")  // 永远不会被运行
}

func main() {
  freakOut()
  fmt.Println("Exiting normally")  // 会被运行
}
```

## 不鼓励使用panic

Go语言本身的设计不鼓励使用panic和recover。在2012年的一次主题会议上，Rob Pike（Go的创始人之一）把panic和recover描述为“故意笨拙”。这意味着，在设计Go时，创作者们没有试图使panic和recover被容易或愉快地使用，因此它们会很少使用。

这是Go设计者对exception的一个主要弱点的回应：它们可以使程序流程更加复杂。相反，Go开发人员被鼓励以处理程序其他部分的方式处理错误：使用if和return语句，以及error值。当然，直接在函数中处理错误会使函数的代码变长，但这比根本不处理错误要好得多。（Go的创始人发现，许多使用exception的开发人员只是抛出一个exception，之后并没有正确地处理它。）直接处理错误也使错误的处理方式一目了然——你不必查找程序的其他部分来查看错误处理代码。

## goroutine

goroutine可以让程序同时处理几个不同的任务。goroutine可以使用channel来协调它们的工作，channel允许goroutine互相发送数据并同步，这样一个goroutine就不会领先于另一个goroutine。goroutine让你充分利用具有多处理器的计算机，让程序运行得尽可能快！

### http.Get

``` go
package main

import (
  "fmt"
  "io/ioutil"
  "log"
  "net/http"
)

func main() {
  responseSize("https://example.com")
  responseSize("https://golang.org")
  responseSize("https://golang.org/doc")
}

func responseSize(url string) {
  fmt.Println("Getting", url)

  response, err := http.Get(url)
  if err != nil {
    log.Fatal(err)
  }

  defer response.Body.Close()

  body, err := ioutil.ReadAll(response.Body)
  if err != nil {
    log.Fatal(err)
  }

  fmt.Println(len(body))
}
```

### 并发

在Go中，并发任务称为goroutine。其他编程语言有一个类似的概念，叫作线程，但是goroutine比线程需要更少的计算机内存，启动和停止的时间更少，这意味着你可以同时运行更多的goroutine。

要启动另一个goroutine，可以使用go语句，它只是一个普通的函数或方法调用，前面有go关键字。

每个Go程序的main函数都是使用goroutine启动的，因此每个Go程序至少运行一个goroutine。

在正常情况下，Go不能保证何时在goroutine之间切换，或者切换多长时间。这允许goroutine以最有效的方式运行。但是，如果goroutine运行的顺序对你很重要，那么你需要使用channel来同步它们。

``` go
package main

import (
  "fmt"
  "io/ioutil"
  "log"
  "net/http"
)

func main() {
  sizes := make(chan int)  // 创建一个int值channel
  go responseSize("https://example.com", sizes)
  go responseSize("https://golang.org", sizes)
  go responseSize("https://golang.org/doc", sizes)
  fmt.Println(<-sizes) // 从channel接收值
  fmt.Println(<-sizes)
  fmt.Println(<-sizes)
}

func responseSize(url string, channel chan int) {
  fmt.Println("Getting", url)

  response, err := http.Get(url)
  if err != nil {
    log.Fatal(err)
  }

  defer response.Body.Close()

  body, err := ioutil.ReadAll(response.Body)
  if err != nil {
    log.Fatal(err)
  }

  channel <- len(body)  // 通过channel发送，会阻塞直到接收方接收完毕
}
```

## 自动化测试

Go包含一个testing包，你可以用来为代码编写自动化测试，还有一个go test命令，你可以用来运行这些测试。

测试文件中的代码由普通的Go函数组成，但需要遵循一定的约定才能使用go test工具：

 - 你不需要将测试代码与正在测试的代码放在同一个包中，但是如果你想从包中访问未导出的类型或函数，则需要这样做。
 - 测试需要使用testing包中的类型，所以需要在每个测试文件的顶部导入该包。
 - 测试函数名应该以Test开头。（名字的其余部分可以是你想要的任何内容，但它应该以大写字母开头。）
 - 测试函数应该接受单个参数：一个指向testing.T值的指针。
 - 你可以通过对testing.T值调用方法（例如Error）来报告测试失败。大多数方法都接受一个字符串，其中包含解释测试失败原因的信息。

要运行测试，可以使用go test命令。该命令采用一个或多个包的导入路径。它将在那些包目录中找到所有名字以_test.go结尾的文件，并运行名字以Test开头的文件中包含的每个函数。

测试驱动开发步骤：

 1. 编写测试
 2. 确保通过
 3. 重构代码

## net/http 包

``` go
package main

import (
  "log"
  "net/http"
)

func write(writer http.ResponseWriter, message string) {
  _, err := writer.Write([]byte(message))
  if err != nil {
    log.Fatal(err)
  }
}

func englishHandler(writer http.ResponseWriter, request *http.Request) {
  write(writer, "Hello, web!")
}

func frenchHandler(writer http.ResponseWriter, request *http.Request) {
  write(writer, "Salut web!")
}

func hindiHandler(writer http.ResponseWriter, request *http.Request) {
  write(writer, "Namaste, web!")
}

func main() {
  http.HandleFunc("/hello", englishHandler)
  http.HandleFunc("/salut", frenchHandler)
  http.HandleFunc("/namaste", hindiHandler)
  err := http.ListenAndServe("localhost:8080", nil)
  log.Fatal(err)
}
```