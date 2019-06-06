# Chapter 2 - Basic Commands and Directory Hierarchy

Created by : Mr Dk.

2019 / 06 / 06 20:58

@Nanjing, Jiangsu, China

---

## 2.1 The Bourne Shell: /bin/sh

shell 是用于执行命令的程序

shell script - 包含一系列 shell 命令的文本文件

有很多类型的 Unix shell

但基本特征全部来源于 Bourne shell

* 于贝尔实验室开发的标准 shell，运行于 Unix 的早期版本

Linux 使用加强版的 Bourne shell - _bash_

* bash 是大部分 Linux 发行版的默认 shell

## 2.2 Using the Shell

### 2.2.1 The Shell Window

### 2.2.2 cat

输出一个或多个文件内容

当输出多个文件时，形成拼接 (concatenation)，因此得名

```bash
$ cat file1 file2
```

### 2.2.3 Standard Input and Standard Output

Unix 使用 I/O stream 读写数据

stream 是一个很灵活的概念

* 文件 / 设备 / 终端 / 其它 stream

如果使用 `cat` 时不加文件名，`cat` 将会继续执行

* 默认从 kernel 的标准输入获取参数
* 内核将标准输入连接到 terminal
* `CTRL-D` 用于停止目前的标准输入
* `CTRL-C` 用于终结程序

`cat` 默认输出到标准输出

* 内核将标准输出连接到 terminal

大部分程序：

* 如果不指定输入文件，则从标准输入读取
* 部分程序输出到标准输出，部分程序输出到文件

## 2.3 Basic Commands

### 2.3.1 ls

`ls` 列出目录中的内容

* 默认输出当前目录

```bash
$ ls -l # detailed(long) listing
$ ls -F # file type information
```

### 2.3.2 cp

拷贝文件

```bash
$ cp file1 file2 # copy file1 to file2
```

拷贝一些文件到一个目录

```bash
$ cp file1 ... fileN dir
```

### 2.3.3 mv

最简单的形式 - 重命名

```bash
$ mv file1 file2
```

将一些文件移动到一个目录

```bash
$ mv file1 ... fileN dir
```

### 2.3.4 touch

创建文件

* 如果文件已经存在，则不会改变该文件，但会更新文件的修改时间

```bash
$ touch file
```

### 2.3.5 rm

### 2.3.6 echo

## 2.4 Navigating Directories

Unix 的目录以 `/` 开头

### 2.4.1 cd

### 2.4.2 mkdir

### 2.4.3 rmdir

### 2.4.4 Shell Globbing (Wildcards)

* `*` 匹配任意字符
* `?` 匹配任意单个字符

在命令执行之前将字符进行扩展

## 2.5 Intermediate Commands

### 2.5.1 grep

`greap` 从文件或输入流中，打印符合表达式的行

```bash
$ grep root /etc/* # print lines contain 'root' in all files under /etc
```

* `-i` - for case-insensitive matches
* `-v` - inverts the search - print all lines __don't__ match

`grep` 的 pattern 支持正则表达式

### 2.5.2 less

当文件或输出较大时使用

每次只会看到一个屏幕的信息

### 2.5.3 pwd

### 2.5.4 diff

```bash
$ diff file1 file2
```

* `diff -u` 适用于一些自动化工具

### 2.5.5 file

不确定一个文件的格式

```bash
$ file file
```

### 2.5.6 find and locate

在指定目录中寻找指定文件

```bash
$ find dir -name file -print
```

### 2.5.7 head and tail

分别显示头 10 行和尾 10 行

* `head -<n>` - 显示 n 行
* `head +<n>` - 从第 n 行开始显示

### 2.5.8 sort

快速将文本文件的每行以字母序排序

* `sort -n` - 按数值排序
* `sort -r` - 逆序

## 2.6 Changing Your Password and Shell

```bash
$ passwd
```

## 2.7 Dot Files

显示 dot files

```bash
$ ls -a
```

部分程序默认不显示它们

## 2.8 Environment and Shell Variables

shell 能存储临时变量

```bash
$ STUFF=blah
$ echo $STUFF
```

Unix 系统中的所有进程都有环境变量存储区

系统会将环境变量传递给所有进程

```bash
$ STUFF=blah
$ export STUFF
```

## 2.9 The Command Path

`PATH` 是一个环境变量，其中包含 `command path`

shell 在执行命令时会从 path 中搜索

打印所有的环境变量，路径用 `:` 分隔：

```bash
$ echo $PATH
```

将某个路径加入到环境变量之前 / 之后：

```bash
$ PATH=dir:$PATH
$ PATH=$PATH:dir
```

* 小心操作，不然可能会覆盖 PATH
* 重新启动 terminal 即可

## 2.10 Special Characters

## 2.11 Command-Line Editing

## 2.12 Text Editors

## 2.13 Getting Online Help

## 2.14 Shell Input and Output

将一条命令的输出从 terminal 转移到文件中

```bash
$ command > file
```

* 如果文件不存在，将会被创建
* 如果文件存在，原有文件将会被覆盖

如果想追加而不是覆盖：

```bash
$ command >> file
```

将命令的标准输出作为另一条命令的标准输入：

```bash
$ head /proc/cpuinfo | tr a-z A-Z
```

### 2.14.1 Standard Error

如果重定向后 terminal 依旧有输出

那么它来自 standard error

将 stderr 重定向到文件，使用 `2e`：

```bash
$ ls /ffff > f 2> e
```

将 stderr 与输出至于 stdout 同样的位置，使用 `>&`：

```bash
$ ls /ffff > f 2>&1
```

文件描述符：

* stdin - 0
* stdout - 1
* stderr - 2

### 2.14.2 Standard Input Redirection

```bash
$ head < /proc/cpuinfo
```

但通常来说没有必要

因为大部分输入接收文件名作为参数

## 2.15 Understanding Error Messages

## 2.16 Listing and Manipulating Processes

### 2.16.1 Command Options

系统上的每一个进程都被分配了数值的 process ID (PID)

```bash
$ ps
$ ps x # running process of user
$ ps ax # all processes on the system
$ ps u # detailed information on processes
$ ps w # full command names
```

* PID - Process ID
* TTY - Terminal device where the process is running
* STAT - The process status
* TIME - The total amount of time running on CPU
* COMMAND

`$$` 是一个 shell 变量，存储了当前 shell 的 PID

### 2.16.2 Killing Processes

当执行 `kill` 时，请求内核发送信号给对应进程

```bash
$ kill [-TERM] pid # terninate
$ kill -STOP pid # stop
$ kill -CONT pid # continue
$ kill -INT pid # CTRL-C
```

kill 比较粗暴，OS 直接回收内存

内核通过数值表示不同的信号，因此也可以直接通过数值发送信号

### 2.16.3 Job Control

### 2.16.4 Background Processes

在 shell 执行命令时

下一个 shell 行会在程序执行停止后才显示

如果想要在后台执行这个程序，使用 `&`

```bash
$ gunzip file.gz &
```

shell 会通过打印新的后台进程的 PID 进行回应

如果后台进程想从 stdin 中读取内容，将导致 freeze 或 terminate

如果后台进程输出到 stdout 或 stderr

terminal 中可能会出现意料之外的输出信息

## 2.17 File Modes and Permissions

每个 Unix 的文件都有权限，决定了是否能够读、写、执行

```bash
$ ls -l
-rw-rw-rw- 1 mrdrivingduck mrdrivingduck 15 Jun  5 09:32 ddd.txt
-rw-rw-rw- 1 mrdrivingduck mrdrivingduck 12 Jun  5 09:31 mmm.txt
```

* 第一位为 Type
  * `-` 为文件
  * `d` 为目录
* 之后每三位分别代表：
  * User permissions
  * Group permissions
  * Other permissions (World permissions)

取值可以为：

* `r` - readable
* `w` - writable
* `x` - executable
* `-` - NO

### 2.17.1 Modifying Permissions

使用 `chmod` 命令

* `u` - user
* `g` - group
* `o` - other

添加权限 - `+`

收回权限 - `-`

```bash
$ chmod g+r file
$ chmod go+r file
$ chmod o-w file
```

也可以用数值一次性修改 permission bits

__目录__ 也有权限

* readable - 可以列出目录中的所有内容
* executable - 才可以访问目录中的文件

### 2.17.2 Symbolic Links

一个指向另一个文件或目录的文件：

```bash
$ ls -l
lrwxrwxrwx 1 mrdrivingduck mrdrivingduck   19 Jun  6 20:35 myuser -> /home/mrdrivingduck
```

### 2.17.3 Creating Symbolic Links

```bash
$ ln -s target linkname
```

`-s` 代表这是软链接，不是硬链接

## 2.18 Archiving and Compressing Files

归档和压缩是有区别的

* 归档是把文件归到一起
* 压缩是把文件进行压缩

### 2.18.1 gzip

GNU Zip 是 Unix 的标准压缩程序，文件后缀名为 `.gz`

```bash
$ gunzip file.gz # uncompress
$ gzip file # compress
```

### 2.18.2 tar

`gzip` 无法将文件进行归档，只能压缩单一的文件

`tar` 可以将多个文件和目录归档到一个文件

```bash
$ tar cvf archive.tar file1 file2 ...
$ tar xvf archive.tar
```

* `c`  - create mode
* `x` - extract mode (unpack)
* `v` - Verbose diagnostic output - 在归档时输出归档的文件和目录
* `f` - file option - 命令的下一个参数必须是对应的归档文件名

在 unpacking 前，最好检测 `.tar` 文件的内容 - `t`：table-of-contents mode

该模式检查了归档的完整性，并打印所有内部文件的文件名

### 2.18.3 Compressed Archives (.tar.gz)

对于这种类型的文件的处理分为两步：

1. 解压缩，去掉 `.gz` 后缀
2. 解归档，去掉 `.tar` 后缀

```bash
$ gunzip file.tar.gz
$ tar xvf file.tar
```

### 2.18.4 zcat

可以使用一条命令简化：

```bash
$ zcat file.tar.gz | tar xvf -
```

* `zcat` == `gunzip -dc`
* `-d` - decompress
* `-c` - send the result to stdout

再简化：

```bash
$ tar ztvf file.tar.gz
```

### 2.18.5 Other Compression Utilities

## 2.19 Linux Directory Hierarchy Essentials

`/bin` : reday-to-run programs / executables 包括很多 Unix 命令

`/dev` : device files

`/etc` : 系统配置目录

`/home` : 用户目录

`/lib` : 库文件，shared libraries only

`/proc` : 目前运行的进程信息和内核参数

`/sys` : 与 `/proc` 类似

`/sbin` : system executables

`/tmp` : 临时文件

### 2.19.1 Other Root Subdirectories

`/boot` : 内核引导文件

`/media`

`/opt`

### 2.19.2 The /usr Directory

### 2.19.3 Kernel Location

在 Linux 系统中，kernel 通常位于 `/vmlinuz` / `/boot/vmlinuz`

_boot loader_ 将在系统引导时该文件导入内存

内核可以按需 load / unload 模块

* loadable kernel modules
* 位于 `/lib/modules`

---

## Summary

有一个命令今天是第一次搞懂是啥意思

但是感觉用得不会太频繁

所以估计过几天又忘了

不过先记下来方便以后查吧

我还没吃晚饭...... 😩

---

