# Chapter 11 - Introduction to Shell Scripts

Created by : Mr Dk.

2019 / 07 / 10 17:03

@NUAA, Nanjing, Jiangsu, China

---

_Shell script_ 是位于文件中的一系列命令

shell 从文件中读取命令并执行，就好像在 terminal 中输入一样

---

## 11.1 Shell Script Basics

Shell 脚本的通常格式：

```shell
#!/bin/sh
#
# Print

echo a b c
```

`#` 表示注释，shell 将会忽略以 `#` 开头的行

在创建脚本后，需要对脚本设置权限：

```bash
$ chmod +rx <script_name>
```

### 11.1.1 Limitations of Shell Scripts

Shell 脚本适合文件批处理的工作

对于字符串操作、运算操作

使用 Python 等编程语言会更好

---

## 11.2 Quoting and Literals

Shell 会将 `$` 字符识别为 shell 变量

为了输出 `$` 字符，需要将 `$` 用 `''` 括起来

### 11.2.1 Literals

Literal 是一个字符串

用于将一个想要传入指定命令的字符串 __完整__ 地传进去

比如，想将 `$100` 传入 `echo` 命令中

Shell 处理命令的过程：

1. 在执行命令之前，shell 会寻找命令中的变量等，并进行替换
2. shell 将替换后的结果传递给对应命令

Literal 实际上跳过了第一步

### 11.2.2 Single Quotes

单引号中的所有字符（包括空格）被视为一个参数

保证了 shell 不会对单引号中的内容做任何替换

### 11.2.3 Double Quotes

双引号的功能与单引号大致相同

但除了 shell 会将所有的变量名 `$` 替换为对应变量值

### 11.2.4 Passing a Literal Single Quote

如果想要将单引号作为字符输出，或作为命令参数的一部分，需要使用 `\` 作为转义

简单粗暴的方法：

1. 将参数中所有的 `'` 替换为 `'\''`
2. 将整个参数字符串用 `''` 包围

---

## 11.3 Special Variables

### 11.3.1 Individual Arguments: $1, $2, ...

所有非零正整数的变量分别对应 shell 脚本的输入参数

Shell 内置的 `shift` 命令将会移除 `$1`，其余参数依次前移

### 11.3.2 Number of Arguments: $#

表示传入脚本的输出参数个数

### 11.3.3 All Arguments: $@

该参数代表脚本的所有参数

### 11.3.4 Script Name: $0

该参数代表脚本的名称

### 11.3.5 Process ID: $$

该参数代表了 shell 的进程 ID

### 11.3.6 Exit Code: $?

该参数代表了上一条命令的 exit code

---

## 11.4 Exit Codes

Unix 程序终结时，会向启动该程序的父进程返回 exit code

一般来说

* `0` 代表了程序结束时没有出现任何问题
* `1` 代表了程序异常退出

但也有一些程序例外，比如 `diff` 和 `grep`

* `1` 可能代表该程序找到了符合条件的信息
* `2` 才代表程序异常

需要看一下程序的说明文档

---

## 11.5 Conditionals

```shell
#!/bin/sh
if [ $1 = hi ]; then
    # ...
elif [ ... ]; then
    # ...
else
    # ...
fi
```

`[` 字符其实对应一个 Unix 程序 - `test`

* 当该程序的 exit code 为 0 时，该条件判断为真
* 由于 `[` 对应一个命令，因此同一行的 `then` 关键字之前必须加 `;`，否则就需要另起一行
* 条件分支在 `fi` 终止
* `elif` 可以少用，因为可以用 `case` 替代

### 11.5.1 Getting Around Empty Parameter Lists

一个很常见的错误：`$1` 为空

避免错误的方法：

```shell
if [ "$1" = hi ]; then
if [ x"$1" = x"hi" ]; then
```

### 11.5.4 && and || Logical Constructs

通常用在 `if` 条件分支中

对于 `&&` ：

* 如果第一条命令失败，那么 `if` 使用第一条命令的 exit code
* 否则使用第二条命令的 exit code

对于 `||` ：

* 如果第一条命令失败，那么 `if` 使用第二条命令的 exit code
* 否则使用第一条命令的 exit code

### 11.5.5 Testing Conditions

test 分三种类型：

#### File Tests

```bash
[ -f file ]
[ file1 -nt file2 ]
```

* `-e` - 如果文件存在返回 true
* `-s` - 如果文件非空返回 true
* ...

#### String Tests

```shell
[ -z "" ]
```

* `-z` - 如果字符串为空返回 true
* `-n` - 如果字符串非空返回 true

#### Arithmetic Tests

使用 `=` 是字符串比较，而不是数学比较

需要使用选项参数来进行数学比较：

```shell
[ 1 -eq 1 ]
```

### 11.5.6 Matching Strings with case

`case` 经常用于字符串比较

* 不包含任何 `test` 命令
* 不评估任何 exit code

```shell
#!/bin/sh
case $1 in
    bye)
        # ...
        ;;
    hi|hello)
        # ...
        ;;
    what*)
        # ...
        ;;
    *)
        # ...
        ;;
esac
```

* 将 `$1` 与 `)` 之前的值进行比较
* 如果匹配，则执行对应的脚本，直到 `;;`，并跳转到 `esac`
* `case` 结束于 `esac`
* 同时匹配多个字符串：使用 `|`，或 `*` 和 `?`
* 用 `*` 来包含所有的默认情况，相当于 C 语言中 switch 中的 `default`

---

## 11.6 Loops

### 11.6.1 for Loops

```shell
#!/bin/sh
for str in one two three four; do
    echo $str
done
```

### 11.6.2 while Loops

其中可以使用 `break` 跳出

---

## 11.7 Command Substitution

在 shell 中，可以将一条命令的输出作为另一条命令的某一参数

* 可以将某一条命令的输出存储在 shell 变量中
* 使用 `$()`

```shell
FLAGS = $(grep ...)
# $FLAGS ...
```

传统的命令替换方法：使用 ` 字符包围命令

---

## 11.8 Temporary File Management

在运行脚本时，需要暂时创建文件存放中间结果，并在脚本结束后删除

```shell
#!/bin/sh
TMPFILE=$(mktemp /tmp/im1.XXXXXX)
# ...
rm -f $TMPFILE
```

`mktemp` 的参数是一个模板，`XXXXXX` 会被转换为一个唯一的字符集，并作为文件名

如果脚本中途执行停止，那么临时文件将会被留下

可以使用 `trap` 命令捕捉中断信号，并删除临时文件：

```shell
#!/bin/sh
trap "rm -f $TMPFILE; exit 1" INT
```

* 在 `trap` 中一定要使用 `exit` 人为停止程序运行
* 否则在中断处理之后，shell 将会被继续执行

---

## 11.10 Important Shell Script Utilities

### 11.10.1 basename

```bash
$ basename example.html .html
$ basename /usr/local/bin/example
```

* 为文件名脱去后缀
* 为目录名脱去前缀

### 11.10.2 awk

类似一门编程语言

现在 Python 盛行，这玩意儿已经没人用了

### 11.10.3 sed

文本编辑

* 将一个输入流（文件或标准输入）作为输入
* 根据表达式做对应操作
* 将结果输出到 stdout

替换、增加、删除指定的字符串，还可以指定特定行

### 11.10.4 xargs

开启大量进程同时运行

### 11.10.5 expr

在 shell 中进行数学运算：

```bash
$ expr 1 + 2
3
```

如果有大量数学运算，就不要用 shell 了，直接用编程语言

### 11.10.6 exec

与 `exec()` 系统调用差不多

使用了之后，shell 进程将被对应进程替换

* 所有变量丢失
* 结束进程后，shell 也被关闭了

---

## 11.11 Subshells

如果想在 shell 中执行一些一次性环境变量的脚本

可以在 shell 中再启动子 shell，并设置其环境变量

也是用 `$()` 实现：

```shell
$ (PATH=/usr/confusing:$PATH; uglyprogram)
$ PATH=/usr/confusing:$PATH; uglyprogram
```

* 下一种已经成为 shell 的内置语法了
* 当子 shell 结束时，子 shell 中的环境变量将不会影响到父 shell 的环境变量

---

## 11.12 Including Other Files in Scripts

使用 `.` 来执行其它 shell：

```shell
. config.sh
```

这种语法不会开启一个子 shell

---

## 11.13 Reading User Input

从标准输入中读取一行，并存储在 shell 变量中：

```shell
$ read var
# $var ...
```

---

