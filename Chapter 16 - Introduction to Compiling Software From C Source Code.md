# Chapter 16 - Introduction to Compiling Software From C Source Code

Created by : Mr Dk.

2019 / 07 / 12 13:40

@NUAA, Nanjing, Jiangsu, China

---

大部分 Unix 第三方软件都是由源代码编译、安装的

* Unix 面向太多的体系结构，很难为所有的体系结构提供 binary 软件包
* 发布源代码能够鼓励更多的开发者进行贡献

---

## 16.1 Software Build Systems

对于 C 的编译和安装系统：从 GNU autotools suite 生成配置脚本

基于 `make` 工具

通常的步骤：

1. 解压缩源码归档
2. 对软件包进行配置
3. 执行 `make` 编译程序
4. 执行 `make install` 安装软件包

---

## 16.2 Unpacking C Source Packages

一个软件包的源码归档通常是压缩文件 - `.tar.gz`, `.tar.bz2`, `.tar.xz`

先将源码解压缩到一个目录中

### 16.2.1 Where to Start

首先阅读 `README` 或 `INSTALL` 文件

---

## 16.3 GNU Autoconf

由于 Unix 面向很多不同平台

很难只使用一个 makefile 编译整个软件

早期的解决方式：

* 为每个平台提供一个 makefile

现在：自动分析系统信息，并生成对应的 makefile

GNU autoconf 是一个用于自动生成 makefile 的系统

* `configure`
* `Makefile.in`
* `config.h.in`

`.in` 文件是模板

运行 `configure` 脚本，检测本机系统的特征，并在模板中进行填充、替换

从而生成最终的 makefile：

```bash
$ ./configure
```

如果一切顺利，将会得到一个或多个 `Makefile` 文件和 `config.h` 文件

也会得到一个缓存文件 `config.cache`

### 16.3.1 An Autoconf Example

### 16.3.2 Installing Using a Packaging Tool

```bash
$ checkinstall make install
```

### 16.3.3 configure Script Options

* `--prefix=</usr/local>` - 指定安装目录
  * 默认情况下 - `/usr/local`，即：
    * 二进制程序 - `/usr/local/bin`
    * 库 - `/usr/local/lib`
* `--bindir=<dir>` - 安装可执行文件的路径
* `--sbindir=<dir>` - 安装系统可执行文件的路径
* `--libdir=<dir>` - 安装库的路径
* `--disable-shared` - 禁止软件编译共享库
* `--with-package=<dir>` - 告诉 `configure` 某个不在默认位置的库的位置

#### Using Separate Build Directories

在另一个地方创建一个新目录用于编译

从新目录中执行 `configure` 脚本

`configure` 会从目录中创建符号链接

指向源目录的代码树

这样，原目录代码树不会改变

### 16.3.4 Environment Variables

可以在配置时加入一些编译选项

* `CPPFLAGS`
* `CFLAGS`
* `LDFLAGS`

### 16.3.5 Autoconf Targets

* `make clean` - 删除所有的目标文件、可执行文件和库文件
* `make distclean` - 删除所有自动生成的文件，包括 makefile 等，使目录回到解归档的状态
* `make check` - 测试编译后的程序是否正常
* `make install-strip` - 将可执行文件和库文件中的符号表等调试信息移除，这样的二进制文件占用的空间更少

---

