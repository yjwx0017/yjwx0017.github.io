title: shell脚本编程 进阶 笔记
categories: 日志
tags: [Linux,shell]
date: 2023-12-16 15:07:00
---
《Linux命令行与shell脚本编程大全》 笔记

## 自定义函数

``` bash
#!/bin/bash
# using a function in a script

function func1 {
  echo "This is an example of a function"
}

# 另一种定义函数的方式
func2() {
  echo "This is an example of a function"
}

count=1
while [ $count -le 5 ]
do
  func1
  count=$[ $count + 1 ]
done
echo "This is the end of the loop"
func1
echo "Now this is the end of the script"
```

函数名必须是唯一的，否则就会出问题。如果定义了同名函数，那么新定义就会覆盖函数原先的定义，而这一切不会有任何错误消息。

<!--more-->

## 函数返回值

在默认情况下，函数的退出状态码是函数中最后一个命令返回的退出状态码。函数执行结束后，可以使用标准变量`$?`来确定函数的退出状态码。

使用return：

``` bash
#!/bin/bash
# using the return command in a function

function dbl {
  read -p "Enter a value: " value
  echo "doubling the value"
  return $[ $value * 2 ]
}

dbl
echo "The new value is $?"
```

当用return从函数中返回值时，一定要小心。为了避免出问题，牢记以下两个技巧。

- 函数执行一结束就立刻读取返回值
- 退出状态码必须介于 0~255

使用echo返回结果：

``` bash
#!/bin/bash
# using the echo to return a value

function dbl {
  read -p "Enter a value: " value
  echo $[ $value * 2 ]
}

result=$(dbl)
echo "The new value is $result"
```

这种方法还可以返回浮点值和字符串，这使其成为一种获取函数返回值的强大方法。

## 函数参数

函数可以使用标准的位置变量来表示在命令行中传给函数的任何参数。例如，函数名保存在$0 变量中，函数参数依次保存在$1、 $2 等变量中。也可以用特殊变量$#来确定传给函数的参数数量。

``` bash
#!/bin/bash
# passing parameters to a function

function addem {
  if [ $# -eq 0 ] || [ $# -gt 2 ]
  then
    echo -1
  elif [ $# -eq 1 ]
  then
    echo $[ $1 + $1 ]
  else
    echo $[ $1 + $2 ]
  fi
}

echo -n "Adding 10 and 15: "
value=$(addem 10 15)
echo $value
echo -n "Let's try adding just one number: "
value=$(addem 10)
echo $value
echo -n "Now try adding no numbers: "
value=$(addem)
echo $value
echo -n "Finally, try adding three numbers: "
value=$(addem 10 15 20)
echo $value
```

尽管函数使用了$1 变量和$2 变量，但它们和脚本主体中的$1 变量和$2 变量不是一回事。

要在函数中使用脚本的命令行参数，必须在调用函数时手动将其传入。

## 变量作用域

全局变量是在 shell 脚本内任何地方都有效的变量。如果在脚本的主体部分定义了一个全局变量，那么就可以在函数内读取它的值。类似地，如果在函数内定义了一个全局变量，那么也可以在脚本的主体部分读取它的值。

在默认情况下，在脚本中定义的任何变量都是全局变量。在函数外定义的变量可在函数内正常访问。

任何在函数内部使用的变量都可以被声明为局部变量。为此，只需在变量声明之前加上 local 关键字即可。

``` bash
#!/bin/bash
# demonstrating the local keyword

function func1 {
  local temp=$[ $value + 5 ]
  result=$[ $temp * 2 ]
}

temp=4
value=6

func1
echo "The result is $result"
if [ $temp -gt $value ]
then
  echo "temp is larger"
else
  echo "temp is smaller"
fi
```

## 向函数传递数组

必须先将数组变量拆解成多个数组元素，然后将这些数组元素作为函数参数传递。最后在函数内部，将所有的参数重新组合成一个新的数组变量。

``` bash
#!/bin/bash
# array variable to function test

function testit {
  local newarray
  newarray=(`echo "$@"`)
  echo "The new array value is: ${newarray[*]}"
}

myarray=(1 2 3 4 5)
echo "The original array is ${myarray[*]}"
testit ${myarray[*]}
```

## 从函数返回数组

函数先用 echo 语句按正确顺序输出数组的各个元素，然后脚本再将数组元素重组成一个新的数组变量。

``` bash
#!/bin/bash
# returning an array value

function arraydblr {
  local origarray
  local newarray
  local elements
  local i
  origarray=($(echo "$@"))
  newarray=($(echo "$@"))
  elements=$[ $# - 1 ]
  for (( i = 0; i <= $elements; i++ ))
  {
    newarray[$i]=$[ ${origarray[$i]} * 2 ]
  }
  echo ${newarray[*]}
}

myarray=(1 2 3 4 5)
echo "The original array is: ${myarray[*]}"
arg1=$(echo ${myarray[*]})
result=($(arraydblr $arg1))
echo "The new array is: ${result[*]}"
```

## 递归函数

``` bash
#!/bin/bash
# using recursion

function factorial {
  if [ $1 -eq 1 ]
  then
    echo 1
  else
    local temp=$[ $1 - 1 ]
    local result=$(factorial $temp)
    echo $[ $result * $1 ]
  fi
}

read -p "Enter value: " value
result=$(factorial $value)
echo "The factorial of $value is: $result"
```

## 函数库

创建一个包含脚本中所需函数的公用库文件：

``` bash
# filename: myfuncs
# my script functions

function addem {
  echo $[ $1 + $2 ]
}

function multem {
  echo $[ $1 * $2 ]
}

function divem {
  if [ $2 -ne 0 ]
  then
    echo $[ $1 / $2 ]
  else
    echo -1
  fi
}
```

使用函数库文件：

``` bash
#!/bin/bash
# using functions defined in a library file

# 和环境变量一样， shell 函数仅在定义它的 shell 会话内有效。
# 如果在 shell 命令行界面运行 myfuncs 脚本，那么 shell 会创建一个新的 shell 
# 并在其中运行这个脚本。在这种情况下，以上 3 个函数会定义在新 shell 中，
# 当你运行另一个要用到这些函数的脚本时，它们是无法使用的。
# 使用函数库的关键在于 source 命令。 
# source 命令会在当前 shell 的上下文中执行命令，
# 而不是创建新的 shell 并在其中执行命令。
# source 命令有个别名，称作点号操作符。
. ./myfuncs # 等价于 source ./myfuncs

value1=10
value2=5
result1=$(addem $value1 $value2)
result2=$(multem $value1 $value2)
result3=$(divem $value1 $value2)
echo "The result of adding them is: $result1"
echo "The result of multiplying them is: $result2"
echo "The result of dividing them is: $result3"
```

## .bashrc

可以在用户主目录的 .bashrc 文件中定义函数。 

## sed 命令 文本替换

``` bash
# s: 替换命令
# test 替换为 big test
echo "This is a test" | sed 's/test/big test/'
```

对文件中的每一行进行替换：

``` bash
# 注意：并不会修改文本文件的数据。它只是将修改后的数据发送到STDOUT。
sed 's/dog/cat/' data1.txt
```

sed 命令行中执行多个命令，可以使用-e 选项：

``` bash
sed -e 's/brown/red/; s/dog/cat/' data1.txt
```

```
$ sed -e '
> s/brown/green/
> s/fox/toad/
> s/dog/cat/' data1.txt
```

执行文件中的 sed 命令：

``` bash
# script1.sed
# s/brown/green/
# s/fox/toad/
# s/dog/cat/
sed -f script1.sed data1.txt
```

## gawk 命令

### 初识

gawk 是 Unix 中最初的 awk 的 GNU 版本。 

gawk 的主要特性之一是处理文本文件中的数据。它会自动为每一行的各个数据元素分配一个变量。在默认情况下， gawk 会将下列变量分配给文本行中的数据字段。

- $0 代表整个文本行。
- $1 代表文本行中的第一个数据字段。
- $2 代表文本行中的第二个数据字段。
- $n 代表文本行中的第 n 个数据字段。

在默认情况下，字段分隔符是任意的空白字符（比如空格或制表符）。

``` bash
# data2.txt
# One line of test text.
# Two lines of test text.
# Three lines of test text.
gawk '{print $1}' data2.txt
```

如果要读取的文件采用了其他的字段分隔符，可以通过-F 选项指定：

``` bash
gawk -F: '{print $1}' /etc/passwd
```

使用多条命令：

``` bash
echo "My name is Rich" | gawk '{$4="Christine"; print $0}'
# output: My name is Christine
```

```
$ gawk '{
> $4="Christine "
> print $0 }'
My name is Rich
My name is Christine
```

从文件中读取脚本：

``` bash
# script3.gawk
# {
# text = "'s home directory is "
# print $1 text $6
# }
gawk -F: -f script3.gawk /etc/passwd
```

BEGIN 关键字强制 gawk 在读取数据前执行 BEGIN 关键字之后指定的脚本。

END 关键字允许指定一段脚本， gawk 会在处理完数据后执行这段脚本。

``` bash
gawk 'BEGIN {print "The data3 File Contents:"} \
{print $0} \
END {print "End of File"}' data3.txt
```


