title: shell脚本编程 基础 笔记
categories: 日志
tags: [Linux,shell]
date: 2023-11-22 12:07:00
---
《Linux命令行与shell脚本编程大全》 笔记

## 执行脚本

添加执行权限：

``` bash
chmod +x test.sh
```

执行脚本时，使用相对路径或绝对路径：

``` bash
./test.sh
```

或将脚本所在路径添加到 PATH 环境变量：

``` bash
export PATH=$PATH:/your/path
```

## 执行命令

``` bash
#!/bin/bash
date
who
```

``` bash
#!/bin/bash
# 在一行中用分号分隔多个命令
date ; who
```

## 显示消息

``` bash
echo hello world
echo "Let's see if this'll work"
# 不输出换行
echo -n "The time and date are: "
```

<!--more-->

## 环境变量

``` bash
echo $PATH
echo ${PATH}
# 转义$
echo \$15
```

## 定义变量

``` bash
# 变量名、等号和值之间不能有空格
var1=10
var2=-57
var3=testing
var4="still more testing"
echo $var1 $var2 $var3 $var4
```

## 命令替换

``` bash
# 以下两种方式等价
testing1=`date`
testing2=$(date)
```

命令替换会创建一个子shell来运行对应的命令，因此子shell中执行的命令无法使用脚本中创建的变量。

## 输出重定向

``` bash
# 重定向到文件
date > test1
# 追加到文件
date >> test1
```

## 输入重定向

``` bash
wc < test1
# 内联输入重定向
wc << EOF
> test string 1
> test string 2
> EOF
```

## 管道

``` bash
rpm -qa | sort | more
```

Linux系统实际上会同时运行这几个命令，系统内部将它们连接起来，数据传输不会用到任何中间文件或缓冲区。

## 执行数学运算

``` bash
var1=100
var2=50
var3=45
var4=$[$var1 * ($var2 - $var3)]
echo The final result is $var4
```

这种方法只支持整数运算。

## 浮点解决方案

使用bc命令。

``` bash
# 浮点运算是由内建变量scale控制的，必须将这个值设置为你希望在计算结果中保留的小数位数
var1=`echo "scale=4; 3.44 / 5" | bc`
echo The answer is $var1
```

``` bash
var1=10.46
var2=43.67
var3=33.2
var4=71

var5=`bc << EOF
scale = 4
a1 = ($var1 * $var2)
b1 = ($var3 * $var4)
a1 + b1
EOF
`

echo The final answer for this mess is $var5
```

## 退出状态码

``` bash
date
echo $?
```

Linux提供了一个专门的变量 $? 来保存上个已执行命令的退出状态码。

按照惯例，一个成功结束的命令的退出状态码是0。如果一个命令结束时有错误，退出状态码就是一个正整数。

默认情况下，shell脚本会以脚本中的最后一个命令的退出状态码退出。

exit命令允许你在脚本结束时指定一个退出状态码。

``` bash
echo hello
exit 5
```

## if-then 语句

if语句会运行if后面的那个命令，如果该命令的退出状态码是0（该命令成功运行），位于then部分的命令就会执行。

``` bash
if pwd
then
  echo "It worked"
fi

# 另一种写法
if pwd; then
  echo "It worked"
fi
```

## if-then-else 语句

``` bash
testuser=NoSuchUser
if grep $testuser /etc/passwd; then
  echo "The bash files for user $testuser are:"
  ls -a /home/$testuser/.b*
  echo
else
  echo "The user $testuser does not exist on this system."
  echo
fi
```

``` bash
if command1; then
  commands
elif command2; then
  commands
else
  commands
fi
```

## test 命令

if-then 不能用来测试命令退出码之外的条件，需要借助test命令。

如果test命令中列出的条件成立，test命令就会退出并返回退出状态码0，否则就返回非零。

判断变量中是否有内容：

``` bash
var1="Full"
if test $var1; then
  echo "The $var1 expression returns a True"
else
  echo "The $var1 expression returns a False"
fi

# 另一种写法（第一个方括号之后和第二个方括号之前必须加空格）：
if [ $var1 ]; then
  echo "The $var1 expression returns a True"
else
  echo "The $var1 expression returns a False"
fi
```

test命令可以判断三类条件：数值比较，字符串比较，文件比较。

## test 数值比较

只支持整数，不支持浮点数。

| 比较       | C++等价描述 |
| ---       | ---      |
| n1 -eq n2 | n1 == n2 |
| n1 -ge n2 | n1 >= n2 |
| n1 -gt n2 | n1 > n2  |
| n1 -le n2 | n1 <= n2 |
| n1 -lt n2 | n1 < n2  |
| n1 -ne n2 | n1 != n2 |

``` bash
value1=10
value2=11
if [ $value1 -gt 5 ]; then
  echo "The test value $value1 is greater than 5"
fi
```

## test 字符串比较

| 比较          | C++等价描述    |
| ---          | ---           |
| str1 = str2  | str1 == str2  |
| str1 != str2 | str1 != str2  |
| str1 < str2  | str1 < str2   |
| str1 > str2  | str1 > str2   |
| -n str1      | !str1.empty() |
| -z str1      | str1.empty()  |

``` bash
testuser=christine
if [ $testuser = christine ]; then
  echo "The testuser variable contains: christine"
else
  echo "The testuser variable contains: $testuser"
fi
```

``` bash
string1=soccer
string2=zorbfootball
# 大于号和小于号需要转义
if [ $string1 \> $string2 ]
then
echo "$string1 is greater than $string2"
else
echo "$string1 is less than $string2"
fi
```

在比较测试中，大写字母被认为是小于小写字母的。但 sort 命令正好相反。当你将同样的字符串放进文件中并用 sort 命令排序时，小写字母会先出现。这是由于各个命令使用了不同的排序技术。

比较测试中使用的是标准的 Unicode 顺序，根据每个字符的 Unicode 编码值来决定排序结果。sort 命令使用的是系统的语言环境设置中定义的排序顺序。

如果你对数值使用了数学运算符号，那么 shell 会将它们当成字符串值，并可能产生错误结果。

空变量和未初始化的变量会对 shell 脚本测试造成灾难性的影响。如果不确定变量的内容，那么最好在将其用于数值或字符串比较之前先通过-n 或-z 来测试一下变量是否为空。

## test 文件比较

| 比较             | 描述    |
| ---             | ---     |
| -d file         | 检查 file 是否存在且为目录 |
| -e file         | 检查 file 是否存在 |
| -f file         | 检查 file 是否存在且为文件 |
| -r file         | 检查 file 是否存在且可读 |
| -s file         | 检查 file 是否存在且非空 |
| -w file         | 检查 file 是否存在且可写 |
| -x file         | 检查 file 是否存在且可执行 |
| -O file         | 检查 file 是否存在且属当前用户所有 |
| -G file         | 检查 file 是否存在且默认组与当前用户相同 |
| file1 -nt file2 | 检查 file1 是否比 file2 新 |
| file1 -ot file2 | 检查 file1 是否比 file2 旧 |

## test 复合条件测试

```
[ condition1 ] && [ condition2 ]
[ condition1 ] || [ condition2 ]
```

``` bash
if [ -d $HOME ] && [ -w $HOME/newfile ]; then
  echo "The file exists and you can write to it."
else
  echo "You cannot write to the file."
```

## if-then 的高级特性: 使用单括号

单括号允许在 if 语句中使用子 shell。

在执行单括号内的命令之前，会先创建一个子 shell，然后在其中执行命令。

``` bash
echo $BASH_SUBSHELL
if (echo $BASH_SUBSHELL); then
  echo "The subshell command operated successfully."
else
  echo "The subshell command was NOT successful."
fi
```

## if-then 的高级特性: 使用双括号

双括号命令允许在比较过程中使用高级数学表达式。

| 比较    | 描述    |
| ---    | ---    |
| val++  | 后增 | 
| val--  | 后减 | 
| ++val  | 先增 | 
| --val  | 先减 | 
| !      | 逻辑求反 | 
| ~      | 位求反 | 
| **     | 幂运算 | 
| <<     | 左位移 | 
| >>     | 右位移 | 
| &      | 位布尔 AND | 
| \|     | 位布尔 OR | 
| &&     | 逻辑 AND | 
| \|\|   | 逻辑 OR | 

``` bash
val1=10
if (( $val1 ** 2 > 90 )); then
  (( val2 = $val1 ** 2 ))
  echo "The square of $val1 is $val2,"
  echo "which is greater than 90."
fi
```

## if-then 的高级特性: 使用双方括号

双方括号命令提供了针对字符串比较的高级特性。

双方括号内可以使用 test 命令中的标准字符串比较。除此之外，它还提供了 test 命令
所不具备的另一个特性——模式匹配。

``` bash
if [[ $BASH_VERSION == 5.* ]]; then
  echo "You are using the Bash Shell version 5 series."
fi
```

双等号会将右侧的字符串（ 5.*）视为一个模式并应用模式匹配规则。双方括号命令会对$BASH_VERSION 环境变量进行匹配，看是否以字符串 5.起始。如果是，则测试通过， shell 会执行 then 部分的命令。

## case 命令

``` bash
case $USER in
rich | christine)
  echo "Welcome $USER"
  echo "Please enjoy your visit.";;
barbara | tim)
  echo "Hi there, $USER"
  echo "We're glad you could join us.";;
testing)
  echo "Please log out when done with test.";;
*)
  echo "Sorry, you are not allowed here.";;
esac
```

## for 命令

``` bash
for test in Alabama Alaska Arizona Arkansas California Colorado
do
  echo The next state is $test
done
```

处理引号：

``` bash
for test in I don\'t know if "this'll" work
do
  echo "word:$test"
done
# output
# word:I
# word:don't
# word:know
# word:if
# word:this'll
# word:work
```

for 命令使用空格来划分列表中的每个值。如果某个值含有空格，则必须将其放入双引号内。

从变量中读取值列表：

``` bash
list="Alabama Alaska Arizona Arkansas Colorado"
list=$list" Connecticut"
for state in $list
do
  echo "Have you ever visited $state?"
done
```

从命令中读取值列表：

``` bash
file="states.txt"
for state in $(cat $file)
do
  echo "Visit beautiful $state"
done
```

IFS 环境变量定义了 bash shell 用作字段分隔符的一系列字符。

在默认情况下， bash shell 会将下列字符视为字段分隔符：

- 空格
- 制表符
- 换行符

如果 bash shell 在数据中看到了这些字符中的任意一个，那么它就会认为这是列表中的一个新字段的开始。在处理可能含有空格的数据（比如文件名）时，这就很烦人了。

解决这个问题的办法是在 shell 脚本中临时更改 IFS 环境变量的值来限制被 bash shell 视为字段分隔符的字符。如果想修改 IFS 的值，使其只能识别换行符，可以这么做：

``` bash
IFS=$'\n'
```

遍历文件中的行：

``` bash
file="states.txt"
IFS=$'\n'
for state in $(cat $file)
do
  echo "Visit beautiful $state"
done
```

一种安全的做法是在修改 IFS 之前保存原来的 IFS 值，之后再恢复它：

``` bash
IFS.OLD=$IFS
IFS=$'\n'
  # <在代码中使用新的 IFS 值>
IFS=$IFS.OLD
```

使用通配符读取目录:

``` bash
for file in /home/rich/test/*
do
  if [ -d "$file" ]
  then
    echo "$file is a directory"
  elif [ -f "$file" ]
  then
    echo "$file is a file"
  fi
done
```

## C 语言风格的 for 命令

``` bash
for (( i=1; i <= 10; i++ ))
do
  echo "The next number is $i"
done
```

``` bash
for (( a=1, b=10; a <= 10; a++, b-- ))
do
  echo "$a - $b"
done
```

## while 命令

``` bash
var1=10
while [ $var1 -gt 0 ]
do
  echo $var1
  var1=$[ $var1 - 1 ]
done
```

while 命令允许在 while 语句行定义多个测试命令。只有最后一个测试命令的退出状态码会被用于决定是否结束循环。

``` bash
var1=10
while echo $var1
      [ $var1 -ge 0 ]
do
  echo "This is inside the loop"
  var1=$[ $var1 - 1 ]
done
```

## until 命令

与 while 命令工作的方式完全相反， until 命令要求指定一个返回非 0 退出状态码的测试命令。只要测试命令的退出状态码不为 0， bash shell 就会执行循环中列出的命令。一旦测试命令返回了退出状态码 0，循环就结束了。

``` bash
var1=100
until [ $var1 -eq 0 ]
do
  echo $var1
  var1=$[ $var1 - 25 ]
done
```

## break 和 continue

用法和其他编程语言相同。

## 处理循环的输出

在 shell 脚本中，可以对循环的输出使用管道或进行重定向。

``` bash
for file in /home/rich/*
do
  if [ -d "$file" ]
  then
    echo "$file is a directory"
  elif
    echo "$file is a file"
  fi
done > output.txt
```

``` bash
for state in "North Dakota" Connecticut Illinois Alabama Tennessee
do
  echo "$state is the next place to go"
done | sort
echo "This completes our travels"
```

## 传递参数

$0 对应脚本名， $1 对应第一个命令行参数， $2 对应第二个命令行参数，以此类推，直到$9。

如果脚本需要的命令行参数不止 9 个，则仍可以继续加入更多的参数，但是需要稍微修改一下位置变量名。在第 9 个位置变量之后，必须在变量名两侧加上花括号，比如${10}。

basename 命令可以返回不包含路径的脚本名：

``` bash
name=$(basename $0)
echo This script name is $name.
exit
```

在使用位置变量之前一定要检查是否为空：

``` bash
if [ -n "$1" ]
then
  factorial=1
  for (( number = 1; number <= $1; number++ ))
  do
    factorial=$[ $factorial * $number ]
  done
  echo The factorial of $1 is $factorial
else
  echo "You did not provide a parameter."
fi
exit
```

特殊变量$#含有脚本运行时携带的命令行参数的个数：

``` bash
if [ $# -eq 1 ]
then
  fragment="parameter was"
else
  fragment="parameters were"
fi
echo $# $fragment supplied.
exit
```

获取最后一个参数：

``` bash
echo The number of parameters is $#
echo The last parameter is ${!#}
exit
```

`$*`变量和`$@`变量可以轻松访问所有参数。

`$*`变量会将所有的命令行参数视为一个单词。这个单词含有命令行中出现的每一个参数。基本上， `$*`变量会将这些参数视为一个整体，而不是一系列个体。

`$@`变量会将所有的命令行参数视为同一字符串中的多个独立的单词，以便你能遍历并处理全部参数。这通常使用 for 命令完成。

## 移动参数

在使用 shift 命令时，默认情况下会将每个位置的变量值都向左移动一个位置。因此，变量$3 的值会移入$2，变量$2 的值会移入$1，而变量$1 的值则会被删除（注意，变量$0 的值，也就是脚本名，不会改变）。

这是遍历命令行参数的另一种好方法，尤其是在不知道到底有多少参数的时候。你可以只操作第一个位置变量，移动参数，然后继续处理该变量。

``` bash
echo
echo "Using the shift method:"
count=1
while [ -n "$1" ]
do
  echo "Parameter #$count = $1"
  count=$[ $count + 1 ]
  shift
done
echo
```

另外，也可以一次性移动多个位置。只需给 shift 命令提供一个参数，指明要移动的位置数即可。

## 处理选项

使用 shift 和 case 处理简单选项：

``` bash
echo
while [ -n "$1" ]
do
  case "$1" in
    -a) echo "Found the -a option" ;;
    -b) echo "Found the -b option" ;;
    -c) echo "Found the -c option" ;;
    *) echo "$1 is not an option" ;;
  esac
  shift
done
echo
exit
```

分离参数和选项：

``` bash
./extractoptionsparams.sh -a -b -c -- test1 test2 test3
```

在 Linux 中，处理这个问题的标准做法是使用特殊字符将两者分开，该字符会告诉脚本选项何时结束，普通参数何时开始。在 Linux 中，这个特殊字符是双连字符（ --）。 shell 会用双连字符表明选项部分结束。在双连字符之后，脚本就可以放心地将剩下的部分作为参数处理了。

``` bash
echo
while [ -n "$1" ]
do
  case "$1" in
    -a) echo "Found the -a option" ;;
    -b) echo "Found the -b option" ;;
    -c) echo "Found the -c option" ;;
    --) shift
        break;;
    *) echo "$1 is not an option" ;;
  esac
  shift
done

echo
count=1
for param in $@
do
  echo "Parameter #$count: $param"
  count=$[ $count + 1 ]
done
echo
exit
```

处理含值的选项：

有些选项需要一个额外的参数值。

``` bash
./testing.sh -a test1 -b -c -d test2
```

``` bash
echo
while [ -n "$1" ]
do
  case "$1" in
    -a) echo "Found the -a option" ;;
    -b) param=$2
        echo "Found the -b option with parameter value $param"
        shift;;
    -c) echo "Found the -c option" ;;
    --) shift
        break;;
    *) echo "$1 is not an option" ;;
  esac
  shift
done

echo
count=1
for param in $@
do
  echo "Parameter #$count: $param"
  count=$[ $count + 1 ]
done
exit
```

现在 shell 脚本已经拥有了处理命令行选项的基本能力，但还有一些局限。例如，当你想合并多个选项时，脚本就不管用了：

``` bash
./extractoptionsvalues.sh -ac
```

## 使用 getopt 命令处理选项

``` bash
getopt ab:cd -a -b BValue -cd test1 test2
# output
# -a -b BValue -c -d -- test1 test2
```

b后面的冒号表示b需要一个参数值。

getopt 会将合并的选项（比如-cd）处理成分离的选项。

如果 optstring 未包含你指定的选项，则在默认情况下， getopt 命令会产生一条错误消息。

如果想忽略这条错误消息，可以使用 getopt 的-q 选项。

set 命令有一个选项是双连字符（ --），可以将位置变量的值替换成 set 命令所指定的值。

具体做法是将脚本的命令行参数传给 getopt 命令，然后再将 getopt 命令的输出传给 set 命令，用 getopt 格式化后的命令行参数来替换原始的命令行参数，如下所示：

``` bash
set -- $(getopt -q ab:cd "$@")

echo
while [ -n "$1" ]
do
  case "$1" in
    -a) echo "Found the -a option" ;;
    -b) param=$2
        echo "Found the -b option with parameter value $param"
        shift;;
    -c) echo "Found the -c option" ;;
    --) shift
        break;;
    *) echo "$1 is not an option" ;;
  esac
  shift
done

echo
count=1
for param in $@
do
  echo "Parameter #$count: $param"
  count=$[ $count + 1 ]
done
exit
```

getopt 命令中仍然隐藏着一个小问题。getopt 命令并不擅长处理带空格和引号的参数值。它会将空格当作参数分隔符，而不是根据双引号将二者当作一个参数。

## 使用 getopts 命令处理选项

getopts 命令要用到两个环境变量。

OPTARG 环境变量保存选项的参数值。 

OPTIND 环境变量保存着参数列表中 getopts 正在处理的参数位置。

``` bash
echo
# 不想显示错误消息的话，可以在 optstring 之前加一个冒号。
while getopts :ab:c opt
do
  case "$opt" in
    a) echo "Found the -a option" ;;
    b) echo "Found the -b option with parameter value $OPTARG";;
    c) echo "Found the -c option" ;;
    *) echo "Unknown option: $opt" ;;
  esac
done
exit
```

``` bash
echo
while getopts :ab:cd opt
do
  case "$opt" in
    a) echo "Found the -a option" ;;
    b) echo "Found the -b option with parameter value $OPTARG";;
    c) echo "Found the -c option" ;;
    d) echo "Found the -d option" ;;
    *) echo "Unknown option: $opt" ;;
  esac
done

shift $[ $OPTIND - 1 ]

echo
count=1
for param in "$@"
do
  echo "Parameter $count: $param"
  count=$[ $count + 1 ]
done
exit
```

## 选项标准化

在 Linux 中，有些选项字母在某种程度上已经有了标准含义。如果能在 shell 脚本中支持这些选项，则你的脚本会对用户更友好。

| 选项 | 描述 |
| --- | --- |
| -a | 显示所有对象 |
| -c | 生成计数 |
| -d | 指定目录 |
| -e | 扩展对象 |
| -f | 指定读入数据的文件 |
| -h | 显示命令的帮助信息 |
| -i | 忽略文本大小写 |
| -l | 产生长格式输出 |
| -n | 使用非交互模式（批处理） |
| -o | 将所有输出重定向至指定的文件 |
| -q | 以静默模式运行 |
| -r | 递归处理目录和文件 |
| -s | 以静默模式运行 |
| -v | 生成详细输出 |
| -x | 排除某个对象 |
| -y | 对所有问题回答 yes |

## read 获取用户输入

``` bash
echo -n "Enter your name: "
read name
echo "Hello $name, welcome to my script."
exit
```

read 命令也提供了-p 选项，允许直接指定提示符：

``` bash
read -p "Please enter your age: " age
days=$[ $age * 365 ]
echo "That means you are over $days days old!"
exit
```

read 命令会将提示符后输入的所有数据分配给单个变量。如果指定多个变量，则输入的每个数据值都会分配给变量列表中的下一个变量。如果变量数量不够，那么剩下的数据就全部分配给最后一个变量：

``` bash
read -p "Enter your first and last name: " first last
echo "Checking data for $last, $first..."
exit
```

也可以在 read 命令中不指定任何变量，这样 read 命令便会将接收到的所有数据都放进特殊环境变量 REPLY 中：

``` bash
read -p "Enter your name: "
echo
echo "Hello $REPLY, welcome to my script."
exit
```

## read 超时

``` bash
if read -t 5 -p "Enter your name: " name
then
  echo "Hello $name, welcome to my script."
else
  echo
  echo "Sorry, no longer waiting for name."
fi
exit
```

当字符数达到预设值时，就自动退出，将已输入的数据赋给变量：

``` bash
read -n 1 -p "Do you want to continue [Y/N]? " answer
case $answer in
Y | y) echo
  echo "Okay. Continue on...";;
N | n) echo
  echo "Okay. Goodbye"
  exit;;
esac
echo "This is the end of the script."
exit
```

## read 无显示读取

-s 选项可以避免在 read 命令中输入的数据出现在屏幕上。

``` bash
read -s -p "Enter your password: " pass
echo
echo "Your password is $pass"
exit
```

## read 从文件中读取

每次调用 read 命令都会从指定文件中读取一行文本。当文件中没有内容可读时， read 命令会退出并返回非 0 退出状态码。

``` bash
count=1
cat $HOME/scripts/test.txt | while read line
do
  echo "Line $count: $line"
  count=$[ $count + 1 ]
done
echo "Finished processing the file."
exit
```

## 命令行中重定向错误

``` bash
# 只重定向错误
ls -al badfile 2> test4
# 重定向错误消息和正常输出
ls -al test test2 test3 badtest 2> test6 1> test7
ls -al test test2 test3 badtest &> test7
```

## 脚本中重定向

重定向输出：

``` bash
echo "This is an error" >&2
echo "This is normal output"
```

这种方法非常适合在脚本中生成错误消息。

如果脚本中有大量数据需要重定向，那么逐条重定向所有的 echo 语句会很烦琐。这时可以用 exec 命令，它会告诉 shell 在脚本执行期间重定向某个特定文件描述符：

``` bash
# 重定向所有输出到一个文件
exec 1>testout
echo "This is a test of redirecting all output"
echo "from a script to another file."
echo "without having to redirect every individual line"
```

重定向输入：

``` bash
exec 0< testfile
count=1
while read line
do
 echo "Line #$count: $line"
 count=$[ $count + 1 ]
done
```

创建自己的重定向：

``` bash
exec 3>test13out
echo "This should display on the monitor"
echo "and this should be stored in the file" >&3
echo "Then this should be back on the monitor"
```

有一个技巧能帮助你恢复已重定向的文件描述符。你可以将另一个文件描述符分配给标准文件描述符，反之亦可。这意味着可以将 STDOUT 的原先位置重定向到另一个文件描述符，然后再利用该文件描述符恢复 STDOUT。

``` bash
exec 3>&1
exec 1>test14out
echo "This should store in the output file"
echo "along with this line."
exec 1>&3
echo "Now things should be back to normal"
```

## 关闭文件描述符

``` bash
exec 3> test17file
echo "This is a test line of data" >&3
exec 3>&-
echo "This won't work" >&3
```

## 列出打开的文件描述符

``` bash
/usr/sbin/lsof -a -p $$ -d 0,1,2
```

## 抑制命令输出

``` bash
ls -al > /dev/null
```

## 创建本地临时文件

在默认情况下， mktemp 会在本地目录中创建一个文件。

mktemp 命令会任意地将 6 个 X 替换为同等数量的字符，以保证文件名在目录中是唯一的。

``` bash
tempfile=$(mktemp test19.XXXXXX)
exec 3>$tempfile
echo "This script writes to temp file $tempfile"
echo "This is the first line" >&3
echo "This is the second line." >&3
echo "This is the last line." >&3
exec 3>&-
echo "Done creating temp file. The contents are:"
cat $tempfile
rm -f $tempfile 2> /dev/null
```

## 在/tmp 目录中创建临时文件

``` bash
tempfile=$(mktemp -t tmp.XXXXXX)
echo "This is a test file." > $tempfile
echo "This is the second line of the test." >> $tempfile
echo "The temp file is located at: $tempfile"
cat $tempfile
rm -f $tempfile
```

## 创建临时目录

``` bash
tempdir=$(mktemp -d dir.XXXXXX)
cd $tempdir
tempfile1=$(mktemp temp.XXXXXX)
tempfile2=$(mktemp temp.XXXXXX)
exec 7> $tempfile1
exec 8> $tempfile2
echo "Sending data to directory $tempdir"
echo "This is a test line of data for $tempfile1" >&7
echo "This is a test line of data for $tempfile2" >&8
```

## tee 命令

tee 命令就像是连接管道的 T 型接头，它能将来自 STDIN 的数据同时送往两处。一处是STDOUT，另一处是 tee 命令行所指定的文件名：

``` bash
tempfile=test22file
echo "This is the start of the test" | tee $tempfile
echo "This is the second line of the test" | tee -a $tempfile
echo "This is the end of the test" | tee -a $tempfile
```

## 常见 Linux 信号

| 信号 | 值 | 描述 |
| --- | --- | --- |
| 1 | SIGHUP | 挂起（ hang up）进程 |
| 2 | SIGINT | 中断（ interrupt）进程 |
| 3 | SIGQUIT | 停止（ stop）进程 |
| 9 | SIGKILL | 无条件终止（ terminate）进程 |
| 15 | SIGTERM | 尽可能终止进程 |
| 18 | SIGCONT | 继续运行停止的进程 |
| 19 | SIGSTOP | 无条件停止，但不终止进程 |
| 20 | SIGTSTP | 停止或暂停（ pause），但不终止进程 |

在默认情况下， bash shell 会忽略收到的任何 SIGQUIT(3)信号和 SIGTERM(15)信号（因此交互式 shell 才不会被意外终止）。但是， bash shell 会处理收到的所有 SIGHUP(1)信号和SIGINT(2)信号。

如果收到了 SIGHUP 信号（比如在离开交互式 shell 时）， bash shell 就会退出。但在退出之前，它会将 SIGHUP 信号传给所有由该 shell 启动的进程，包括正在运行的 shell 脚本。随着收到 SIGINT 信号， shell 会被中断。 Linux 内核将不再为 shell 分配 CPU 处理时间。当出现这种情况时， shell 会将 SIGINT 信号传给由其启动的所有进程，以此告知出现的状况。

停止（ stopping）进程跟终止（ terminating）进程不同，前者让程序继续驻留在内存中，还能从上次停止的位置继续运行。 

## 产生信号

Ctrl+C - SIGINT 信号
Ctrl+Z - SIGTSTP 信号

## 捕获信号

``` bash
trap "echo ' Sorry! I have trapped Ctrl-C'" SIGINT

echo This is a test script.

count=1
while [ $count -le 5 ]
do
  echo "Loop #$count"
  sleep 1
  count=$[ $count + 1 ]
done

echo "This is the end of test script."
exit
```

## 捕获脚本退出

这是在 shell 完成任务时执行命令的一种简便方法。

``` bash
trap "echo Goodbye..." EXIT

count=1
while [ $count -le 5 ]
do
  echo "Loop #$count"
  sleep 1
  count=$[ $count + 1 ]
done
```

## 移除信号捕获

``` bash
trap "echo ' Sorry...Ctrl-C is trapped.'" SIGINT

count=1
while [ $count -le 3 ]
do
  echo "Loop #$count"
  sleep 1
  count=$[ $count + 1 ]
done

trap -- SIGINT
echo "The trap is now removed."

count=1
while [ $count -le 3 ]
do
  echo "Second Loop #$count"
  sleep 1
  count=$[ $count + 1 ]
done
exit
```

也可以在 trap 命令后使用单连字符来恢复信号的默认行为。单连字符和双连字符的效果一样。

## 后台运行脚本

``` bash
./backgroundscript.sh &
```

有时候，即便退出了终端会话，你也想在终端会话中启动 shell 脚本，让脚本一直以后台模式运行到结束。这可以用 nohup 命令来实现。

nohup 命令能阻断发给特定进程的 SIGHUP 信号。当退出终端会话时，这可以避免进程退出。

``` bash
nohup ./testAscript.sh &
```

由于 nohup 命令会解除终端与进程之间的关联，因此进程不再同 STDOUT 和 STDERR 绑定在一起。为了保存该命令产生的输出， nohup 命令会自动将 STDOUT 和 STDERR 产生的消息重定向到一个名为 nohup.out 的文件中。

当运行位于同一目录中的多个命令时，一定要当心，因为所有的命令输出都会发送到同一个 nohup.out 文件中，结果会让人摸不着头脑。

## 查看作业

jobs 命令

jobs 命令输出中的加号和减号。带有加号的作业为默认作业。如果作业控制命令没有指定作业号，则引用的就是该作业。

带有减号的作业会在默认作业结束之后成为下一个默认作业。任何时候，不管 shell 中运行着多少作业，带加号的作业只能有一个，带减号的作业也只能有一个。

查看作业的 PID：

``` bash
jobs -l
```

向指定 PID 发送 SIGKILL(9) 信号：

``` bash
kill -9 1580
```

## 重启已停止的作业

要以后台模式重启作业，可以使用 bg 命令。

如果存在多个作业，则需要在 bg 命令后加上作业号，以便于控制。

要以前台模式重启作业，可以使用带有作业号的 fg 命令。

## 调整谦让度

调度优先级是一个整数值，取值范围从-20（最高优先级）到+19（最低优先级）。在默认情况下， bash shell 以优先级 0 来启动所有进程。

nice 命令允许在启动命令时设置其调度优先级。要想让命令以更低的优先级运行，只需用nice 命令的-n 选项指定新的优先级即可：

``` bash
nice -n 10 ./jobcontrol.sh > jobcontrol.out &
```

只有 root 用户或者特权用户才能提高作业的优先级。

有时候，你想修改系统中已运行命令的优先级。 renice 命令可以帮你搞定。

``` bash
renice -n 10 -p 16642
```

## 定时运行作业

### 使用 at 命令调度作业

当在 Linux 系统中运行 at 命令时，显示器并不会关联到该作业。 Linux 系统反而会将提交该作业的用户 email 地址作为 STDOUT 和 STDERR。任何送往 STDOUT 或 STDERR 的输出都会通过邮件系统传给该用户。

``` bash
at -f tryat.sh now
```

使用 email 作为 at 命令的输出极不方便。 at 命令通过 sendmail 应用程序发送 email。如果系统中没有安装 sendmail，那就无法获得任何输出。因此在使用 at 命令时，最好在脚本中对STDOUT 和 STDERR 进行重定向。

``` bash
outfile=$HOME/scripts/tryat.out
#
echo "This script ran at $(date +%B%d,%T)" > $outfile
echo >> $outfile
echo "This script is using the $SHELL shell." >> $outfile
echo >> $outfile
sleep 5
echo "This is the script's end." >> $outfile
#
exit
```

``` bash
at -M -f tryatout.sh now
```

如果不想在 at 命令中使用 email 或者重定向，则最好加上-M 选项，以禁止作业产生的输出信息。

atq 命令列出等待的作业。

``` bash
at -M -f tryatout.sh teatime
at -M -f tryatout.sh tomorrow
at -M -f tryatout.sh 20:30
at -M -f tryatout.sh now+1hour
atq
```

atrm 命令删除等待中的作业。

### cron 调度需要定期运行的脚本

cron 时间表通过一种特别的格式指定作业何时运行，其格式如下：

minutepasthour hourofday dayofmonth month dayofweek command

如果想在每天的 10:15 运行一个命令，可以使用如下 cron 时间表字段：

```
15 10 * * * command
```

要指定一条在每周一的下午 4:15（ 4:15 p.m.）执行的命令：

```
15 16 * * 1 command
```

每月第一天的中午 12 点执行命令：

```
00 12 1 * * command
```

聪明的读者可能会思考，如何设置才能让命令在每月的最后一天执行，因为无法设置一个 dayofmonth 值，涵盖所有月份的最后一天。常用的解决方法是加一个 if-then 语句，在其中使用 date 命令检查明天的日期是不是某个月份的第一天（ 01）：

```
00 12 28-31 * * if [ "$(date +%d -d tomorrow)" = 01 ] ; then command ; fi
```

另一种方法是将 command 替换成一个控制脚本（ controlling script），在可能是每月最后一天的时候运行。控制脚本包含 if-then 语句，用于检查第二天是否为某个月的第一天。如果是，则由控制脚本发出命令，执行必须在当月最后一天执行的内容。

列出已有的 cron 时间表：

``` bash
crontab -l
```

如果创建的脚本对于执行时间的精确性要求不高，则用预配置的 cron 脚本目录会更方便。预配置的基础目录共有 4 个： hourly、 daily、 monthly 和 weekly。

``` bash
ls /etc/cron.*ly
```

如果你的脚本需要每天运行一次，那么将脚本复制到 daily 目录， cron 就会每天运行它。

cron 程序唯一的问题是它假定 Linux 系统是 7×24 小时运行的。除非你的 Linux 运行在服务器环境，否则这种假设未必成立。

如果某个作业在 cron 时间表中设置的运行时间已到，但这时候 Linux 系统处于关闭状态，那么该作业就不会运行。当再次启动系统时， cron 程序不会再去运行那些错过的作业。为了解决这个问题，许多 Linux 发行版提供了 anacron 程序。

如果 anacron 判断出某个作业错过了设置的运行时间，它会尽快运行该作业。这意味着如果Linux 系统关闭了几天，等到再次启动时，原计划在关机期间运行的作业会自动运行。有了anacron，就能确保作业一定能运行，这正是通常使用 anacron 代替 cron 调度作业的原因。

## 使用新 shell 启动脚本

当用户登录 bash shell 时要运行的启动文件：

- $HOME/.bash_profile
- $HOME/.bash_login
- $HOME/.profile

基本上，以下所列文件中的第一个文件会被运行，其余的则会被忽略。

因此，应该将需要在登录时运行的脚本放在上述第一个文件中。

每次启动新 shell， bash shell 都会运行$HOME/.bashrc 文件。 
